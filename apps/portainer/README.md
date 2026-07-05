# Portainer

Gerenciador de containers Docker do home server. Diferente das stacks em `apps/stacks/`, este serviço **não é implantado pelo Portainer** — é um `docker-compose` standalone iniciado manualmente pelo terminal, pois o Portainer é justamente a ferramenta que passará a gerenciar todas as demais stacks documentadas neste repositório.

## Sobre o Portainer

O [Portainer](https://www.portainer.io/) é uma plataforma leve para gerenciar ambientes containerizados por meio de interface gráfica e API. Permite administrar containers, imagens, volumes, redes e stacks (Docker Compose) sem depender exclusivamente da linha de comando.

Nesta instalação utiliza-se a **Community Edition (CE)** — versão gratuita, open source (licença zlib), sem chave de licença e sem telemetria. É adequada para uso pessoal e pequenos ambientes, oferecendo gestão completa de Docker, deploy de stacks, acesso a logs e console de containers.

## Papel no home server

O Portainer roda em um **Raspberry Pi 5** (ARM64) e centraliza a operação das stacks:

- [admin](../stacks/admin/README.md) — backup do Portainer e Cloudflare Tunnel
- [games](../stacks/games/README.md) — RomM
- [stream](../stacks/stream/README.md) — ecossistema *Arr e Jellyfin

Após a instalação inicial, as stacks são criadas e mantidas pela interface web do Portainer. O backup automatizado da instância Portainer é feito pela stack admin ([portainer-backup](../stacks/admin/README.md)).

## Configuração

Antes do primeiro deploy, ajuste o volume de dados no `docker-compose.yaml`:

```yaml
- /home/username/yourpath/portainer/data:/data
```

Substitua pelo caminho real no host onde a configuração e o banco de dados do Portainer serão persistidos.

O container monta o socket Docker do host (`/var/run/docker.sock`) em modo somente leitura, permitindo que o Portainer controle o daemon local.

| Porta | Uso |
| --- | --- |
| `9443` | Interface web HTTPS (recomendado) |
| `9000` | Interface web HTTP |

## Deploy

Este serviço deve ser iniciado **uma única vez pelo terminal**, antes de configurar as demais stacks no Portainer:

```bash
cd apps/portainer
docker compose up -d
```

Na primeira execução, acesse `https://<host>:9443` (ou `http://<host>:9000`) e crie o usuário administrador.

## Referências

- [Portainer — Site oficial](https://www.portainer.io/)
- [Documentação](https://docs.portainer.io/)
- [Repositório GitHub (CE)](https://github.com/portainer/portainer)
