# Stream

Stack de mídia e streaming do home server, baseada no ecossistema **\*Arr** — conjunto popular de aplicações open source e self-hosted para automatizar a aquisição, organização e reprodução de conteúdo audiovisual.

**Onde roda:** [Dell OptiPlex 3050 Micro](../../../../hardware/dell3050/README.md) (`dell3050`), no HDD de 1 TB. **Migrada do Raspberry Pi 5** — configs em `~/Stream`, biblioteca em `/media/.../HDD/stream`.

> **Importante:** este repositório contém apenas um **exemplo funcional** da stack. A integração correta entre os serviços (indexadores, clientes de download, paths de volume, perfis de qualidade, nomenclatura de arquivos etc.) depende de configuração detalhada. **Consulte sempre a documentação oficial** de cada aplicação para montar a pipeline de ponta a ponta.

Compose alinhado ao Dell: **Jackett + Prowlarr**, Jellyfin com **`/dev/dri`** (aceleração de hardware) e `group_add` dos grupos `render`/`video` do host.

## Arquitetura de rede

Os containers desta stack **não publicam portas no host**. O acesso às WebUIs passam pelo [Nginx Proxy Manager](../admin/README.md) na rede externa `proxy_network`. A comunicação entre os serviços da pipeline (Sonarr ↔ qBittorrent, Seerr ↔ Radarr, etc.) usa a rede local `stream_network`.

```
proxy_network  →  acesso HTTP via NPM (jellyfin, seerr, sonarr, …)
stream_network →  tráfego interno entre containers da stack
```

Crie a rede externa uma vez, se ainda não existir:

```bash
docker network create proxy_network
```

No NPM, configure um Proxy Host por serviço usando o **nome do container** e a porta interna:

| Container | Porta interna |
| --- | --- |
| `jellyfin` | `8096` |
| `seerr` | `5055` |
| `qbittorrent` | valor de `WEBUI_PORT` (padrão `8080`) |
| `sonarr` | `8989` |
| `radarr` | `7878` |
| `bazarr` | `6767` |
| `jackett` | `9117` |
| `prowlarr` | `9696` |

No Dell, o NPM local recebe o repasse do NPM do Pi 5 ([estratégia entre NPMs](../../raspberrypi/admin/README.md#estratégia-de-comunicação-entre-os-npms)).


## Pipeline automatizada

Quando configurada corretamente, o fluxo típico funciona assim:

```
Seerr → Sonarr / Radarr → Indexadores → qBittorrent → importação → Bazarr → Jellyfin
```

1. **Seerr** — usuários solicitam filmes ou séries pela interface web.
2. **Sonarr / Radarr** — recebem o pedido, monitoram o título e buscam releases nos indexadores.
3. **Prowlarr** e/ou **Jackett** — indexadores; no Dell os dois rodam (Prowlarr é o caminho Servarr “nativo”; Jackett permanece como fallback/legado).
4. **qBittorrent** — recebe o torrent selecionado, faz o download e reporta o progresso.
5. **Sonarr / Radarr** — importam o arquivo baixado, renomeiam, organizam pastas e enriquecem com metadados (posters, sinopses, IDs).
6. **Bazarr** — busca e associa legendas no idioma desejado aos arquivos de vídeo.
7. **Jellyfin** — escaneia a biblioteca e disponibiliza o catálogo para streaming em qualquer dispositivo.

Todos os serviços de download e organização compartilham o mesmo volume de mídia (`STREAM_MEDIA_PATH`), o que facilita hard links, moves rápidos e permissões consistentes entre containers. Nas integrações entre apps, use o hostname do container (ex.: `http://qbittorrent:8080`) em vez de `localhost` ou do IP do host.

## Aplicações

### Seerr

Interface de **pedidos de mídia** para usuários finais. Integra com Jellyfin (também Plex e Emby) para exibir o que já existe na biblioteca e encaminha solicitações de filmes e séries para Radarr e Sonarr. Oferece controle de permissões, aprovação de pedidos e notificações.

Nesta stack, a configuração fica em `SEERR_CONFIG_PATH`; a WebUI interna escuta na porta `5055`.

### Sonarr

**PVR de séries de TV** para Usenet e torrents. Monitora séries e temporadas, busca episódios ausentes, envia downloads ao cliente configurado e gerencia falhas automaticamente (blacklist de releases problemáticos, nova tentativa). Inclui calendário de estreias, busca manual, perfis de qualidade customizáveis e notificações.

Nesta stack, configuração persistida em `SONARR_CONFIG_PATH`, com acesso à mídia em `/stream`.

### Radarr

Equivalente ao Sonarr, focado em **filmes**. Monitora filmes desejados, busca releases, gerencia qualidade via Custom Formats e importa automaticamente para a biblioteca. Suporta coleções, atores e diretores favoritos.

Nesta stack, configuração persistida em `RADARR_CONFIG_PATH`, com acesso à mídia em `/stream`.

### Prowlarr

**Gerenciador central de indexadores** do ecossistema Servarr. Configura trackers e sites de busca uma única vez e sincroniza com Sonarr, Radarr, Lidarr e demais apps *Arr — eliminando a necessidade de repetir credenciais e URLs em cada serviço.

Nesta stack (Dell), o container `prowlarr` usa `PROWLARR_CONFIG_PATH`. **Jackett** continua na stack em paralelo (`JACKETT_CONFIG_PATH`).

### Bazarr

Companion do Sonarr e Radarr para **gerenciamento de legendas**. Busca automaticamente subtítulos ausentes, permite busca manual entre múltiplos provedores e faz upgrade periódico quando legendas de melhor qualidade ficam disponíveis.

Nesta stack, monitora os mesmos diretórios de mídia montados em `/stream`.

### Jellyfin

**Servidor de mídia** free software (GPL). Organiza e transmite filmes, séries, música, TV ao vivo, livros e fotos a partir do seu próprio servidor — sem taxas, sem tracking e sem dependência de serviços externos. Suporta múltiplos clientes (web, mobile, TV, Kodi) e recursos como SyncPlay, DVR e gerenciamento multi-usuário.

Nesta stack, a mídia é montada em `/media` a partir de `STREAM_MEDIA_PATH`. O acesso à WebUI/streaming passa pelo NPM (porta interna `8096`).

No Dell, o Jellyfin monta **`/dev/dri`** e entra nos grupos do host (`JELLYFIN_RENDER_GID` / `JELLYFIN_VIDEO_GID`, tipicamente `render` e `video`) para transcodificação por hardware (Intel iGPU do OptiPlex). Confirme os GIDs com `getent group render video` no host — não copie os valores do `env.example` sem validar.

### qBittorrent

**Cliente BitTorrent** open source, alternativa ao µTorrent. Interface web para controle remoto, suporte a magnet links, DHT, PEX, filas de prioridade e seleção de arquivos dentro do torrent. É o ponto de download da pipeline: Sonarr e Radarr enviam torrents selecionados e monitoram o progresso via API.

Nesta stack, a WebUI usa a porta definida em `WEBUI_PORT` (padrão `8080`), acessível via NPM e pelos demais containers na `stream_network`.

### Jackett

Proxy de indexadores legado/alternativo. Continua útil para trackers que o Prowlarr ainda não cobre bem. Config em `JACKETT_CONFIG_PATH`; `AUTO_UPDATE` controla atualização automática do Jackett.

## Configuração

Copie `env.example` para `.env` e preencha as variáveis antes de subir a stack:

```bash
cp env.example .env
```

| Variável | Descrição |
| --- | --- |
| `STREAM_MEDIA_PATH` | Volume compartilhado de mídia no HDD (downloads + biblioteca) |
| `PUID` / `PGID` | UID/GID do usuário dono dos arquivos no host |
| `WEBUI_PORT` / `UMASK` | Porta WebUI e umask do qBittorrent |
| `AUTO_UPDATE` | Auto-update do Jackett |
| `JELLYFIN_*_PATH` | Config e cache do Jellyfin |
| `JELLYFIN_RENDER_GID` / `JELLYFIN_VIDEO_GID` | GIDs do host para `/dev/dri` |
| `SEERR_CONFIG_PATH` | Config do Seerr (em produção o dir pode se chamar `jelyserr`) |
| `PROWLARR_CONFIG_PATH` | Config do Prowlarr |
| `JACKETT_CONFIG_PATH` | Config do Jackett |
| `QBITTORRENT_CONFIG_PATH` / `SONARR_*` / `RADARR_*` / `BAZARR_*` | Demais configs |

A configuração da pipeline (conectar indexadores, definir root folders, mapear paths entre qBittorrent e *Arr, integrar Bazarr com Sonarr/Radarr, vincular Seerr ao Jellyfin) **não está no docker-compose** — deve ser feita nas interfaces web de cada serviço, seguindo os guias oficiais.

## Deploy

```bash
docker compose up -d
```

Após subir os containers, configure os Proxy Hosts no Nginx Proxy Manager e complete a configuração inicial de cada WebUI antes de esperar automação end-to-end.

## Sugestão de conteúdo em português

Existem diversos tutoriais na internet sobre a configuração da pipeline *Arr. Abaixo, uma sugestão em português — **não é a única referência válida**, mas é uma excelente opção para quem prefere conteúdo nesse idioma.

Canal: [Danilo TI](https://www.youtube.com/@Danilo_TI)

- [Configuração da pipeline (parte 1)](https://www.youtube.com/watch?v=OcWetMkxKfI)
- [Configuração da pipeline (parte 2)](https://www.youtube.com/watch?v=SVoMhL0PxJM)
- [Configuração da pipeline (parte 3)](https://www.youtube.com/watch?v=FwSxjpVRdWs)

## Referências oficiais

| Aplicação | Site |
| --- | --- |
| Sonarr | [sonarr.tv](https://sonarr.tv/) |
| Radarr | [radarr.video](https://radarr.video/) |
| Prowlarr | [prowlarr.com](https://prowlarr.com/) |
| Bazarr | [bazarr.media](https://www.bazarr.media/) |
| Jellyfin | [jellyfin.org](https://jellyfin.org/) |
| Seerr | [docs.seerr.dev](https://docs.seerr.dev/) |
| qBittorrent | [qbittorrent.org](https://www.qbittorrent.org/) |

Documentação complementar do ecossistema Servarr: [wiki.servarr.com](https://wiki.servarr.com/)
