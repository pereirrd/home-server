# MCPs

Stack de servidores [MCP](https://modelcontextprotocol.io/) (Model Context Protocol) do home server. Roda no **Raspberry Pi 5** (`raspberrypi`), no projeto Compose `mcps`, e é implantada pelo [Komodo](../komodo/README.md). O ponto de entrada dos clientes é um **gateway**: o [MCPJungle](https://github.com/mcpjungle/mcpjungle).

Nenhum serviço publica porta no host. O [Nginx Proxy Manager](../admin/README.md) de entrada alcança o gateway em `http://mcpjungle-server:8080` na `proxy_network`. O hostname local é `mcp.local.meudominio.me`.

## Serviços

| Serviço | Contêiner | Função |
| ------- | --------- | ------ |
| `db` | `mcpjungle-db` | Postgres 17. Só na rede do projeto. Não entra na `proxy_network`. |
| `mcpjungle` | `mcpjungle-server` | Gateway. Escuta `8080` dentro da rede. A imagem `latest-stdio` traz `npx` e `uvx` para servidores STDIO. |
| `komodo-mcp-server` | `komodo-mcp-server` | MCP do [Komodo](../komodo/README.md). Fala com `http://komodo-core:9120`. |
| `npm-mcp-raspberrypi` | `npm-mcp-raspberrypi` | MCP do Nginx Proxy Manager deste Pi. Fala com `http://nginx-proxy-manager:81/api`. |
| `npm-mcp-dell3050` | `npm-mcp-dell3050` | MCP do Nginx Proxy Manager do Dell. Fala com a API do NPM daquele host, pela LAN (`http://<ip-dell3050>:81/api`). |

O endpoint MCP de cada servidor é `/mcp`. `/health` responde que o processo subiu. No MCP do Komodo, `/ready` responde 200 quando a conexão com o Core fecha.

Os dados do Postgres ficam num diretório do host (`MCPJUNGLE_DATA_PATH`), não num volume nomeado. `docker compose down -v` não apaga essa pasta.

## Somente leitura de propósito

O MCP do Komodo sobe com autenticação desligada (`MCP_AUTH_ENABLED=false`) porque o único cliente é o gateway, na rede Docker, sem login de navegador. Nesse modo o servidor HTTP fica **somente leitura**: ferramentas de escrita, exclusão e terminal não aparecem.

Os MCP do Nginx Proxy Manager vêm de `ghcr.io/b3nw/nginx-proxy-manager-mcp`. A escrita fica desligada: a variável que habilita criar, alterar, apagar, ligar e desligar host **não** entra no compose. Também não há bearer token — o cliente é o gateway, na mesma rede.

`SERVER_MODE=development` no MCPJungle não exige autenticação. O hostname do gateway fica na rede de casa. O modo `enterprise` passa a exigir `mcpjungle init-server` depois do primeiro boot.

## Gateway e grupos

O endpoint `/mcp` do gateway expõe **todos** os servidores registrados. Os clientes deste homelab não usam esse endpoint. Usam um **grupo**: um recorte nomeado de servidores, com URL própria.

```
https://mcp.local.meudominio.me/v0/groups/<grupo>/mcp
```

| Grupo | Servidores | Quando usar |
| ----- | ---------- | ----------- |
| `raspberry-pi` | `komodo-mcp`, `npm-mcp-raspberrypi` | Stacks, containers e proxy **deste Pi**. O Komodo enxerga os dois hosts — filtre pelo servidor `raspberrypi`. |
| `dell-3050` | `komodo-mcp`, `npm-mcp-dell3050` | O mesmo, filtrando o servidor `dell3050` e o painel de proxy do Dell. |
| `telemetria` | `grafana-mcp` | Consulta de dashboard e métrica. Sem Komodo e sem proxy. |
| `automacao-residencial` | `home-assistant` | Home Assistant no Raspberry Pi 4. |
| `repositorio` | `GitHub` | Repositórios. O nome do servidor no gateway leva G maiúsculo. |
| `proxy-manager` | `npm-mcp-raspberrypi`, `npm-mcp-dell3050` | Os dois painéis de proxy, sem o Komodo. |

`grafana-mcp`, `home-assistant` e `GitHub` são servidores registrados no gateway (STDIO ou remoto). Não são containers deste compose: entram pelo cliente `mcpjungle` depois que o gateway está no ar. A tool aparece para o cliente como `servidor__tool`.

Um grupo é a fonte do recorte. Criar ou atualizar o grupo no gateway é o que passa a valer. Mudar só o arquivo local não altera o endpoint.

### Cliente (Cursor e OpenCode)

Os dois leem a mesma lista de grupos. Exemplo do bloco que o Cursor espera em `mcp.json` — sem token, porque o gateway desta rede não exige bearer:

```json
{
  "mcpServers": {
    "raspberry-pi": {
      "url": "https://mcp.local.meudominio.me/v0/groups/raspberry-pi/mcp"
    },
    "dell-3050": {
      "url": "https://mcp.local.meudominio.me/v0/groups/dell-3050/mcp"
    },
    "telemetria": {
      "url": "https://mcp.local.meudominio.me/v0/groups/telemetria/mcp"
    },
    "automacao-residencial": {
      "url": "https://mcp.local.meudominio.me/v0/groups/automacao-residencial/mcp"
    },
    "repositorio": {
      "url": "https://mcp.local.meudominio.me/v0/groups/repositorio/mcp"
    },
    "proxy-manager": {
      "url": "https://mcp.local.meudominio.me/v0/groups/proxy-manager/mcp"
    }
  }
}
```

O OpenCode usa as mesmas URLs, com `type: remote`.

## Configuração

Copie `env.example` para `.env`. Segredo (senha do Postgres, chave de API do Komodo, senha dos painéis NPM) fica só nesse arquivo.

| Variável | Descrição |
| --- | --- |
| `POSTGRES_*` | Usuário, senha e banco do MCPJungle. A senha também entra na `DATABASE_URL` do gateway. |
| `MCPJUNGLE_IMAGE_TAG` | Tag da imagem. `latest-stdio` inclui `npx` e `uvx`. |
| `SERVER_MODE` | `development` nesta rede de casa. |
| `KOMODO_URL` | `http://komodo-core:9120` dentro da `proxy_network`. |
| `KOMODO_API_KEY` / `KOMODO_API_SECRET` | Chave de API do Core. A mesma usada por outros clientes do Komodo neste host. |
| `NPM_RPI_API_URL` | `http://nginx-proxy-manager:81/api`. |
| `NPM_DELL3050_API_URL` | API do NPM do Dell, pela LAN. |
| `NPM_*_IDENTITY` / `NPM_*_SECRET` | Usuário e senha de cada painel. |
| `MCPJUNGLE_DATA_PATH` | Dados do Postgres no host. |
| `MCPJUNGLE_HOST_PATH` | Pasta somente leitura montada em `/host` para servidores de filesystem. |

Antes do deploy, a rede externa precisa existir neste Docker:

```bash
docker network create proxy_network
```

No Nginx Proxy Manager, o Proxy Host de `mcp.local.meudominio.me` aponta para `mcpjungle-server:8080`.

## Deploy

Pelo Komodo, na stack `mcps`, depois de validar a interpolação:

```bash
docker compose --env-file .env config --quiet
```

Recriar o Postgres sem o diretório `MCPJUNGLE_DATA_PATH` perde os servidores registrados no gateway.

## Referências

- [Model Context Protocol](https://modelcontextprotocol.io/)
- [MCPJungle](https://github.com/mcpjungle/mcpjungle)
- [nginx-proxy-manager-mcp](https://github.com/b3nw/nginx-proxy-manager-mcp)
- [Komodo](../komodo/README.md)
- [Admin do Pi 5 — NPM de entrada](../admin/README.md)
