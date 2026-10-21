# Seafile

Stack de sincronização e compartilhamento de arquivos do home server, baseada no [Seafile](https://www.seafile.com/). Roda no **Dell OptiPlex 3050 Micro** (`dell3050`), junto ao armazenamento em massa (HDD 1 TB).

> Este repositório traz um **exemplo ilustrativo** do compose em produção. Ajuste volumes, hostname e secrets; consulte sempre a [documentação oficial](https://manual.seafile.com/docker/deploy_seafile_with_docker/).

## Onde roda

| Host | Hostname | Stack Compose |
| ---- | -------- | ------------- |
| Dell OptiPlex 3050 Micro | `dell3050` | `seafile` |

Inventário completo do nó: [hardware/dell](../../../hardware/dell/README.md).

Acesso local típico: NPM do Pi 5 → NPM do Dell → container `seafile` na `proxy_network` do x86 ([estratégia entre NPMs](../admin/README.md#estratégia-de-comunicação-entre-os-npms)).

## Arquivos de exemplo

```
apps/stacks/seafile/
├── README.md
├── docker-compose.yaml
└── env.example
```

## Arquitetura de rede

```
proxy_network (Dell)  →  seafile, seadoc (HTTP via NPM)
seafile-net           →  seafile ↔ seadoc ↔ db ↔ redis
```

Antes do deploy no Dell:

```bash
docker network create proxy_network
```

No NPM do Dell, aponte o Proxy Host do hostname do Seafile para o container `seafile` (porta interna `80`). O Seadoc usa paths `/sdoc-server` e `/socket.io` no mesmo host.

## Containers

| Container | Serviço Compose | Função |
| --------- | --------------- | ------ |
| `seafile` | `seafile` | Servidor Seafile (WebUI / sync) |
| `seadoc` | `seadoc` | Edição colaborativa (Seadoc) |
| `seafile-mysql` | `db` | MariaDB |
| `seafile-redis` | `redis` | Cache |

## Configuração

Copie `env.example` para `.env` e preencha. Variáveis críticas:

| Variável | Descrição |
| --- | --- |
| `BASIC_STORAGE_PATH` | Raiz dos volumes no host (HDD) |
| `SEAFILE_VOLUME` / `SEAFILE_MYSQL_VOLUME` / `SEADOC_VOLUME` | Paths derivados (ou absolutos) |
| `SEAFILE_SERVER_HOSTNAME` | Hostname público/local (ex.: `seafile.local.meudominio.me`) |
| `SEAFILE_SERVER_PROTOCOL` | `https` quando atrás do NPM com TLS |
| `JWT_PRIVATE_KEY` | Chave compartilhada Seafile ↔ Seadoc |
| `SEAFILE_MYSQL_DB_PASSWORD` | Senha do usuário `seafile` no MariaDB |
| `INIT_SEAFILE_MYSQL_ROOT_PASSWORD` | Root do MariaDB (init / healthcheck) |
| `INIT_SEAFILE_ADMIN_EMAIL` / `INIT_SEAFILE_ADMIN_PASSWORD` | Admin criado na primeira subida |
| `ENABLE_SEADOC` | `true` para edição colaborativa |

Labels `caddy.*` no compose vêm do template oficial do Seafile; neste home server o proxy de entrada é o **Nginx Proxy Manager**, não o Caddy — as labels podem ser ignoradas pelo runtime atual.

## Referências

- [Seafile](https://www.seafile.com/)
- [Seafile Docker](https://manual.seafile.com/docker/deploy_seafile_with_docker/)
- [Dell OptiPlex 3050 Micro](../../../hardware/dell/README.md)
- [Admin — NPMs](../admin/README.md)
