# Immich

Stack de gerenciamento de fotos e vídeos do home server. Hospeda o [Immich](https://immich.app/), alternativa self-hosted de alto desempenho a serviços de nuvem como Google Photos.

## Arquitetura de rede

O Immich **não publica porta no host**. O acesso à interface web passa pelo [Nginx Proxy Manager](../admin/README.md) na rede externa `proxy_network`. Redis e PostgreSQL ficam apenas na rede local da stack (`default`).

```
proxy_network  →  Immich Server (WebUI / API) via NPM
default        →  immich-server ↔ redis ↔ database ↔ machine-learning
```

No NPM, aponte o Proxy Host para o container `immich_server` na porta interna `2283`.

## Containers

| Container | Função |
| --- | --- |
| `immich_server` | API, WebUI e orquestração da biblioteca (uploads, álbuns, usuários, jobs). Único serviço na `proxy_network`. |
| `immich_machine_learning` | Inferência de ML (reconhecimento facial, smart search, tags). Modelos em cache no volume `model-cache`. |
| `immich_redis` | Valkey (Redis) — filas, cache e coordenação de jobs em background. |
| `immich_postgres` | PostgreSQL com extensões vetoriais (VectorChord / pgvector) para metadados e busca semântica. |

Aceleração de hardware (transcoding e ML) fica comentada no `docker-compose.yaml`; veja a documentação oficial se for habilitar.

## Configuração

Copie `env.example` para `.env` e preencha as variáveis antes de subir a stack:

```bash
cp env.example .env
```

| Variável | Descrição |
| --- | --- |
| `TZ` | Fuso horário do servidor |
| `IMMICH_VERSION` | Tag da imagem (`release`, `v3` ou versão pinada) |
| `UPLOAD_LOCATION` | Diretório no host para a biblioteca de mídia |
| `DB_DATA_LOCATION` | Diretório no host para os dados do PostgreSQL (SSD local recomendado) |
| `DB_USERNAME` / `DB_DATABASE_NAME` | Usuário e nome do banco |
| `DB_PASSWORD` | Senha do Postgres (apenas `A-Za-z0-9`) |

Documentação das variáveis: [Environment Variables](https://docs.immich.app/install/environment-variables).

## Deploy

```bash
docker compose up -d
```

Após o deploy, configure o Proxy Host no Nginx Proxy Manager e acesse o hostname definido para criar o usuário administrador.

## Referências

- [Immich — Site oficial](https://immich.app/)
- [Immich — Documentação](https://docs.immich.app/)
- [Instalação com Docker Compose](https://docs.immich.app/install/docker-compose)
- [Variáveis de ambiente](https://docs.immich.app/install/environment-variables)
- [Aceleração de hardware (ML)](https://docs.immich.app/features/ml-hardware-acceleration)
- [Releases / compose oficial](https://github.com/immich-app/immich/releases/latest)
