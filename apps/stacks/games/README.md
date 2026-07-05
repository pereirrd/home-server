# Games

Stack de serviços de jogos do home server. Hospeda o [RomM](https://romm.app/), um gerenciador de ROMs self-hosted acessado pelo navegador.

## Aplicações

### RomM

O RomM (ROM Manager) é um gerenciador de coleção de jogos retro e modernos, open source e self-hosted. Toda a operação acontece por uma interface web responsiva: escanear pastas de ROMs, enriquecer os títulos com metadados, organizar a biblioteca e jogar direto no browser — sem instalar emuladores no cliente.

**Principais vantagens**

- **Self-hosted e privado** — os dados ficam no seu servidor; sem tracking, sem upsells e controle total sobre a biblioteca.
- **Metadados automáticos** — capas, descrições e informações de plataforma obtidas de provedores como IGDB, ScreenScraper, MobyGames, RetroAchievements, Hasheous e SteamGridDB.
- **Ampla compatibilidade** — suporte a mais de 400 sistemas e plataformas, com reconhecimento de múltiplos esquemas de nomenclatura, tags customizadas, multi-disc, DLCs, mods, hacks e manuais.
- **Jogo no navegador** — execução integrada via [EmulatorJS](https://github.com/EmulatorJS/EmulatorJS) e [Ruffle](https://ruffle.rs/) (Flash), sem configuração extra no dispositivo de acesso.
- **Multi-usuário** — compartilhamento da biblioteca com permissões granulares; suporte a OIDC/SSO para ambientes com vários usuários.
- **Ecossistema de integrações** — apps e plugins oficiais para Playnite (Windows), Argosy (Android/handhelds) e Grout (CFWs como muOS, Knulli, ROCKNIX), com sync de saves e configurações entre dispositivos.
- **Open source** — licenciado sob AGPL-3.0, mantido pela comunidade com releases frequentes.

**Facilidade de uso**

O fluxo típico é simples: organizar as ROMs em pastas por plataforma, apontar o RomM para a biblioteca, configurar as chaves de API dos provedores de metadados e deixar o scan enriquecer a coleção automaticamente. A partir daí, a biblioteca pode ser navegada, filtrada por tags e jogada pelo browser ou sincronizada com dispositivos compatíveis.

**Tecnologias**

| Camada | Tecnologia |
| --- | --- |
| Backend | Python, [FastAPI](https://fastapi.tiangolo.com/) |
| Frontend | [Vue.js](https://vuejs.org/) |
| Servidor web | nginx |
| Banco de dados | MariaDB |
| Cache / filas | Valkey (Redis) |
| Emulação no browser | EmulatorJS, Ruffle |
| Deploy | Docker / Docker Compose |

Nesta stack, o RomM roda junto com um container MariaDB (`romm-db`). A biblioteca de ROMs, saves, configurações e recursos são persistidos em volumes montados no host (`ROMM_LIBRARY_PATH`, `ROMM_ASSETS_PATH`, `ROMM_CONFIG_PATH`). A aplicação fica exposta na porta `ROMM_PORT` (padrão `9080`).

**Referência:** [RomM — Site oficial](https://romm.app/)

## Configuração

Copie `env.example` para `.env` e preencha as variáveis antes de subir a stack:

```bash
cp env.example .env
```

| Variável | Descrição |
| --- | --- |
| `DB_PASSWORD` | Senha do usuário do banco (usada por RomM e MariaDB) |
| `MARIADB_ROOT_PASSWORD` | Senha root do MariaDB |
| `ROMM_AUTH_SECRET_KEY` | Chave de sessão (gerar com `openssl rand -hex 32`) |
| `SCREENSCRAPER_USER` / `SCREENSCRAPER_PASSWORD` | Credenciais do [ScreenScraper](https://www.screenscraper.fr/) |
| `RETROACHIEVEMENTS_API_KEY` | Chave da API do [RetroAchievements](https://retroachievements.org/) |
| `STEAMGRIDDB_API_KEY` | Chave da API do [SteamGridDB](https://www.steamgriddb.com/) |
| `ROMM_LIBRARY_PATH` | Diretório no host com as ROMs organizadas por plataforma |
| `ROMM_ASSETS_PATH` | Diretório de saves, states e assets |
| `ROMM_CONFIG_PATH` | Diretório de configuração (`config.yml`) |
| `ROMM_PORT` | Porta exposta no host |

Documentação complementar: [Metadata Providers](https://docs.romm.app/latest/Getting-Started/Metadata-Providers/) · [Folder Structure](https://docs.romm.app/latest/Getting-Started/Folder-Structure/)

## Deploy

```bash
docker compose up -d
```

Após o deploy, acesse `http://<host>:<ROMM_PORT>` no navegador para configurar o usuário administrador e iniciar o scan da biblioteca.
