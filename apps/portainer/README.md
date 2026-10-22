# Portainer

Gerenciador de containers Docker do home server. Diferente das stacks em `apps/stacks/`, este serviço **não é implantado pelo Portainer** — é um `docker-compose` standalone iniciado manualmente pelo terminal, pois o Portainer é justamente a ferramenta que passará a gerenciar todas as demais stacks documentadas neste repositório.

## Sobre o Portainer

O [Portainer](https://www.portainer.io/) é uma plataforma leve para gerenciar ambientes containerizados por meio de interface gráfica e API. Permite administrar containers, imagens, volumes, redes e stacks (Docker Compose) sem depender exclusivamente da linha de comando.

Nesta instalação utiliza-se a **Community Edition (CE)** — versão gratuita, open source (licença zlib), sem chave de licença e sem telemetria. É adequada para uso pessoal e pequenos ambientes, oferecendo gestão completa de Docker, deploy de stacks, acesso a logs e console de containers.

## Uma única instância — CE no Pi 5, Agent no Dell

Existe **somente um Portainer CE** na infraestrutura, rodando no **Raspberry Pi 5** (`raspberrypi`). Não há segunda interface/web do Portainer no Dell.

O **Dell OptiPlex 3050 Micro** (`dell3050`) não hospeda o Portainer CE. Nele roda apenas o **[Portainer Agent](https://docs.portainer.io/admin/environments/add/docker/agent)** (`portainer-agent`): um endpoint leve que expõe o Docker Engine local ao Portainer do Pi 5. Pela UI do CE (no Pi 5), seleciona-se o ambiente `dell3050` e gerenciam-se stacks, containers, volumes e redes **desse** host — tudo pelo mesmo painel.

```
raspberrypi (Pi 5)                         dell3050
┌─────────────────────┐                    ┌─────────────────────┐
│  Portainer CE       │ ──── Agent API ──► │  portainer-agent    │
│  (única UI / API)   │                    │  (sem UI própria)   │
│  ambiente local     │                    │  → Docker do Dell   │
│  + ambiente Dell    │                    └─────────────────────┘
└─────────────────────┘
```

| Componente | Host | Papel |
| ---------- | ---- | ----- |
| `portainer` (CE) | `raspberrypi` | Única interface web e API; gerencia os dois Docker Engines |
| `portainer-agent` | `dell3050` | Bridge remota — **não** é um segundo Portainer |

O agent fica **fora** das stacks Compose do Portainer (container standalone / deploy próprio). O backup automatizado (`portainer-backup` na stack admin) cobre a instância CE do Pi 5.

## Arquitetura de rede

O Portainer CE **não publica portas no host**. Ele participa da rede externa `proxy_network` do Pi 5 e o acesso à interface web é feito pelo [Nginx Proxy Manager](../stacks/raspberrypi/admin/README.md) de entrada.

```bash
docker network create proxy_network   # se ainda não existir (em cada host, se aplicável)
```

No NPM do Pi 5, aponte o Proxy Host para o container `portainer`. Portas internas da imagem:

| Porta interna | Uso |
| --- | --- |
| `9443` | Interface web HTTPS |
| `9000` | Interface web HTTP |

Para o Proxy Host, o destino mais simples é `http://portainer:9000` (ou `https://portainer:9443` se preferir TLS ponta a ponta até o container).

A comunicação **CE → Agent** ocorre na LAN (Pi 5 fala com o agent no `dell3050`, tipicamente na porta do agent). Não passa pela `proxy_network` Docker — são hosts distintos.

## Papel no home server

Stacks gerenciadas a partir **desta** única UI do Portainer:

| Ambiente no Portainer | Host | Stacks |
| --------------------- | ---- | ------ |
| Local (Docker do Pi 5) | `raspberrypi` | [admin](../stacks/raspberrypi/admin/README.md), [games](../stacks/raspberrypi/games/README.md), [immich](../stacks/raspberrypi/immich/README.md), [telemetria](../stacks/raspberrypi/telemetria/README.md) |
| Remoto (via Agent) | `dell3050` | [admin](../stacks/dell3050/admin/README.md), [stream](../stacks/dell3050/stream/README.md), [seafile](../stacks/dell3050/seafile/README.md) |

Após a instalação inicial, as stacks dos dois hosts são criadas e mantidas pela interface web do Portainer CE. O backup automatizado da instância é feito pela stack admin ([portainer-backup](../stacks/raspberrypi/admin/README.md)).

## Configuração

Antes do primeiro deploy, ajuste o volume de dados no `docker-compose.yaml`:

```yaml
- /home/username/yourpath/portainer/data:/data
```

Substitua pelo caminho real no host onde a configuração e o banco de dados do Portainer serão persistidos.

O container monta o socket Docker do host (`/var/run/docker.sock`) em modo somente leitura, permitindo que o Portainer controle o daemon local.

## Deploy

Este serviço deve ser iniciado **pelo terminal**. A ordem recomendada é criar a `proxy_network`, subir o Portainer e em seguida a [stack admin](../stacks/raspberrypi/admin/README.md) (que inclui o NPM):

```bash
docker network create proxy_network
cd apps/portainer
docker compose up -d
```

Na primeira execução, acesse o hostname configurado no NPM (ou temporariamente o IP do host na porta do painel do NPM, se ainda estiver criando o Proxy Host) e crie o usuário administrador.

## Referências

- [Portainer — Site oficial](https://www.portainer.io/)
- [Documentação](https://docs.portainer.io/)
- [Add a Docker Standalone environment via Agent](https://docs.portainer.io/admin/environments/add/docker/agent)
- [Repositório GitHub (CE)](https://github.com/portainer/portainer)
