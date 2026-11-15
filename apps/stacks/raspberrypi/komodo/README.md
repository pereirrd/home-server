# Komodo

Gerenciador de containers do home server. Substituiu o Portainer — o motivo está em [DESCONTINUADOS.md](../../../../DESCONTINUADOS.md). Esta stack fica junto das outras do Raspberry Pi 5, mas **não é implantada pelo próprio Komodo**: sobe por `docker compose` no host, porque é a ferramenta que passa a gerenciar as demais.

## Sobre o Komodo

O [Komodo](https://komo.do/) é uma plataforma open source para operar Docker (e outros alvos) por interface web e API. Nesta instalação o modo usado é **Files on Server**: o `docker-compose.yaml` de cada stack fica **no disco do host**, e o Komodo só aponta para esse arquivo. Um agente de IA edita o arquivo no repositório de configuração da máquina; o deploy no Komodo lê esse arquivo. Não há uma segunda cópia do compose dentro de um banco.

A interface web foi o critério de escolha depois de comparar outras opções de gestão de containers.

## Core no Pi 5, Periphery em cada host

Existe **um** Komodo Core, no **Raspberry Pi 5** (`raspberrypi`). O banco é MongoDB 7, no mesmo compose. Cada máquina Docker tem um **Periphery**: o do Pi 5 sobe neste compose; o do Dell OptiPlex 3050 Micro sobe no próprio Dell e se registra no Core pela LAN.

O Core **não clona** o repositório de configuração. O Periphery executa o compose no diretório que já está no disco.

```
raspberrypi (Pi 5)                         dell3050
┌─────────────────────┐                    ┌─────────────────────┐
│  Komodo Core + Mongo│ ◄── Periphery ───► │  Komodo Periphery   │
│  (única UI / API)   │                    │  (sem UI)           │
│  Periphery local    │                    │  → Docker do Dell   │
└─────────────────────┘                    └─────────────────────┘
```

| Componente | Host | Papel |
| ---------- | ---- | ----- |
| `komodo-core` | `raspberrypi` | Interface web e API. Responde em `komodo-core:9120` na `proxy_network`. |
| `komodo-mongo` | `raspberrypi` | Banco do Core. Imagem `mongo:7`. |
| `komodo-periphery` | `raspberrypi` e `dell3050` | Agente local. Usa o socket do Docker e lê o diretório de stacks daquela máquina. |

O rótulo `komodo.skip` nos três serviços do Pi 5 impede que um "parar tudo" do próprio Komodo derrube o Core, o banco e o Periphery local.

Nesta placa o Mongo fica em `mongo:7`, com o cache do WiredTiger limitado. A imagem `mongo:8` não sobe no kernel deste Pi.

## Files on Server

Cada stack gerenciada tem um bloco num `stacks.toml` lido por um **Resource Sync** do Core (modo *Files on Server*). O sync **registra** a stack. Ele não faz o deploy: `deploy = false`. O deploy é uma ação separada na interface, e `destroy_before_deploy` fica `false` para não apagar volumes.

O campo de environment do recurso fica vazio. O `.env` da pasta da stack entra só como `--env-file` no host. O Komodo não copia o conteúdo desse arquivo para o banco dele.

Exemplo ilustrativo do que o sync do Pi 5 declara (caminhos genéricos — no servidor o diretório é o clone do repositório de configuração daquela máquina):

```toml
[[stack]]
name = "admin"
deploy = false

[stack.config]
server = "raspberrypi"
files_on_host = true
project_name = "admin"
run_directory = "/home/username/servidor/docker/admin"
file_paths = ["docker-compose.yaml"]
env_file_path = ".env"
destroy_before_deploy = false
```

O nome do recurso é único em todo o Core. O `project_name` é o projeto Docker local e pode ser diferente do nome do recurso — no Dell a stack de administração se chama `dell3050-admin` no Core e continua com `project_name = "admin"`, porque esse já é o projeto em execução.

A stack `komodo` **não** entra no `stacks.toml`. Ela sobe por `docker compose` no Pi 5.

## O que cada host executa

| Servidor no Core | Stacks gerenciadas pelo Komodo | Fora do sync |
| ---------------- | ------------------------------ | ------------ |
| `raspberrypi` | [admin](../admin/README.md), [telemetria](../telemetria/README.md), [mcps](../mcps/README.md) | este compose (`komodo`) |
| `dell3050` | [admin](../../dell3050/admin/README.md), [games](../../dell3050/games/README.md), [immich](../../dell3050/immich/README.md), [stream](../../dell3050/stream/README.md), [seafile](../../dell3050/seafile/README.md) | Periphery local |

## Arquitetura de rede

O Core publica `9120` e entra na rede externa `proxy_network` do Pi 5. O acesso à interface passa pelo [Nginx Proxy Manager](../admin/README.md) de entrada, no host `komodo.local.meudominio.me`, com destino `http://komodo-core:9120`.

O Mongo **não** entra na `proxy_network` e não publica porta no host.

```bash
docker network create proxy_network   # se ainda não existir
```

## Configuração

Copie `env.example` para `.env` e preencha antes do primeiro `up`. O arquivo real fica só no disco do Pi 5.

| Variável | Descrição |
| --- | --- |
| `COMPOSE_KOMODO_IMAGE_TAG` | Tag das imagens `komodo-core` e `komodo-periphery` |
| `COMPOSE_KOMODO_BACKUPS_PATH` | Diretório no host montado em `/backups` no Core |
| `KOMODO_DATABASE_USERNAME` / `KOMODO_DATABASE_PASSWORD` | Usuário root do Mongo criado na primeira subida |
| `KOMODO_INIT_ADMIN_USERNAME` / `KOMODO_INIT_ADMIN_PASSWORD` | Admin inicial da interface. A senha depois vive só no Komodo |
| `PERIPHERY_ROOT_DIRECTORY` | Diretório do host que o Periphery enxerga no mesmo caminho |
| `PERIPHERY_CORE_ADDRESS` | Endereço do Core visto pelo Periphery (no Pi 5, o serviço `core`) |

Segredo (senha do banco, JWT, chave de API, senha do admin) fica só no `.env`. Trocar o valor no arquivo **não** migra o que já foi gravado no volume `mongo-data`.

## Deploy

No Raspberry Pi 5, pelo terminal:

```bash
docker network create proxy_network
cd apps/stacks/raspberrypi/komodo
cp env.example .env
docker compose up -d
```

Na primeira execução, abra `http://komodo.local.meudominio.me` (depois de criar o Proxy Host no NPM), entre com o admin inicial e cadastre o Resource Sync de cada máquina. Executar o sync registra as stacks. O deploy de cada uma é um passo separado — no Pi 5, telemetria antes de admin, porque admin publica `80`, `443`, `81` e `1883`.

No Dell, o Periphery é um compose à parte, com o mesmo par de chaves do Core, apontando para o diretório de stacks daquela máquina.

## Referências

- [Komodo](https://komo.do/)
- [mbecker20/komodo](https://github.com/mbecker20/komodo)
- [Stacks Docker deste repositório](../admin/README.md)
- [MCPs — inclusive o servidor MCP do Komodo](../mcps/README.md)
- [Aplicações descontinuadas — Portainer](../../../../DESCONTINUADOS.md)
