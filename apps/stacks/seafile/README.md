# Seafile

Stack de sincronização e compartilhamento de arquivos do home server, baseada no [Seafile](https://www.seafile.com/). Roda no **Dell OptiPlex 3050 Micro** (`dell3050`), junto ao armazenamento em massa (HDD 1 TB).

> Documentação inicial do inventário em produção. Exemplos de `docker-compose` / `env` podem ser adicionados depois — a referência operacional continua sendo a documentação oficial do Seafile.

## Onde roda

| Host | Hostname | Stack Compose |
| ---- | -------- | ------------- |
| Dell OptiPlex 3050 Micro | `dell3050` | `seafile` |

Inventário completo do nó: [hardware/dell](../../../hardware/dell/README.md).

## Containers

| Container | Imagem (produção) | Função |
| --------- | ----------------- | ------ |
| `seafile` | `seafileltd/seafile-mc:13.0-latest` | Servidor Seafile (WebUI / sync) |
| `seadoc` | `seafileltd/sdoc-server:2.0-latest` | Edição colaborativa de documentos (Seadoc) |
| `seafile-mysql` | `mariadb:10.11` | Banco de dados |
| `seafile-redis` | `redis` | Cache / filas |

## Referências

- [Seafile](https://www.seafile.com/)
- [Seafile Docker](https://manual.seafile.com/docker/deploy_seafile_with_docker/)
- [Dell OptiPlex 3050 Micro](../../../hardware/dell/README.md)
