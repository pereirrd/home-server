# Home Assistant

Documentação das aplicações instaladas no [Home Assistant](https://www.home-assistant.io/) do home server. Todas rodam no formato **Apps** (anteriormente chamados de *add-ons*), gerenciadas pela interface do próprio HA — Settings → Apps.

> **Importante:** este diretório documenta o que está instalado na instância. A configuração detalhada de cada app deve seguir a documentação oficial referenciada abaixo.

## Repositórios de origem


| Origem                                   | Apps instalados           |
| ---------------------------------------- | ------------------------- |
| Repositório oficial de Apps do HA        | AdGuard Home, Uptime Kuma |
| Comunidade (via repositório customizado) | HACS, Immich              |


Para instalar apps de repositórios da comunidade, é necessário adicionar a URL do repositório em **Settings → Apps → Repositories** no Home Assistant. O **HACS** estende essa capacidade para integrações, temas e componentes customizados além dos apps em si.

## Aplicações



### HACS

O [Home Assistant Community Store (HACS)](https://hacs.xyz/) é a loja da comunidade para o Home Assistant. Permite descobrir, instalar e atualizar integrações, dashboards, temas e templates mantidos por terceiros — conteúdo que não faz parte do repositório oficial de Apps.

Nesta instalação, o HACS é a **base para expandir o HA com repositórios da comunidade**, incluindo o repositório que disponibiliza o app Immich.

**Referência:** [HACS — Configuração inicial](https://hacs.xyz/docs/use/configuration/basic/)

### AdGuard Home

[AdGuard Home](https://adguard.com/adguard-home/overview.html) é um servidor DNS de bloqueio de anúncios e rastreadores em nível de rede, com controle parental e interface web para gerenciar filtros. Atua como DNS da rede local, protegendo todos os dispositivos conectados sem instalar software em cada um.

Instalado a partir do **repositório oficial de Apps do Home Assistant** (mantido por [hassio-addons](https://github.com/hassio-addons)).

**Referência:** [AdGuard Home — Home Assistant Community App](https://github.com/hassio-addons/app-adguard-home)

### Uptime Kuma

[Uptime Kuma](https://uptime.kuma.pet/) é uma ferramenta de monitoramento self-hosted, comparável a serviços comerciais como Uptime Robot. Monitora disponibilidade de serviços via HTTP/S, TCP, DNS e outros protocolos, com dashboard visual e notificações em caso de indisponibilidade.

Útil para acompanhar a saúde dos serviços do home server (stacks Docker, apps do HA, endpoints expostos via túnel). Também pode disparar webhooks de automação no Home Assistant.

Instalado a partir do **repositório oficial de Apps do Home Assistant**.

**Referência:** [Uptime Kuma — Home Assistant Community App](https://github.com/hassio-addons/app-uptime-kuma)

### Immich

[Immich](https://immich.app/) é uma solução self-hosted de alto desempenho para gerenciamento de fotos e vídeos — alternativa open source a serviços de nuvem como Google Photos. Oferece backup automático de dispositivos móveis, reconhecimento facial, álbuns compartilhados, busca por metadados e timeline de mídia.

Instalado a partir do repositório da comunidade [fabio-garavini/hassio-addons](https://github.com/fabio-garavini/hassio-addons), que empacota diversos serviços populares como apps do Home Assistant.

**Referências:**

- [fabio-garavini/hassio-addons](https://github.com/fabio-garavini/hassio-addons)
- [Immich — Documentação](https://immich.app/docs)



## Integração com o home server

O Home Assistant centraliza automações e monitoramento da casa. Os proxies ESP32 documentados em `[automations/esp32](../../automations/esp32/README.md)` integram-se ao HA via ESPHome para estender o alcance Bluetooth (BLE) e rastrear dispositivos.

Para adicionar um novo app de repositório da comunidade:

1. **Settings → Apps → Repositories → Add**
2. Informe a URL do repositório GitHub (ex.: `https://github.com/fabio-garavini/hassio-addons`)
3. Instale o app desejado em **Settings → Apps → Install**

Consulte sempre a documentação oficial de cada app antes de alterar configurações de rede, DNS ou volumes de armazenamento.

## Referências

- [Home Assistant](https://www.home-assistant.io/)
- [HACS](https://hacs.xyz/)
- [HACS — Configuração inicial](https://hacs.xyz/docs/use/configuration/basic/)
- [AdGuard Home](https://adguard.com/adguard-home/overview.html)
- [hassio-addons](https://github.com/hassio-addons)
- [AdGuard Home — Home Assistant Community App](https://github.com/hassio-addons/app-adguard-home)
- [Uptime Kuma](https://uptime.kuma.pet/)
- [Uptime Kuma — Home Assistant Community App](https://github.com/hassio-addons/app-uptime-kuma)
- [fabio-garavini/hassio-addons](https://github.com/fabio-garavini/hassio-addons)
- [Immich](https://immich.app/)
- [Immich — Documentação](https://immich.app/docs)
- [ESP32 Bluetooth Tracker](../../automations/esp32/README.md)

