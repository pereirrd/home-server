# Hardware — Dell

Máquinas x86 adicionadas à infraestrutura do home server. A documentação de software (stacks, Komodo, Home Assistant) está no [README principal](../../README.md). Os Raspberry Pi continuam em [hardware/raspberry_pi](../raspberry_pi/README.md).

## Visão geral

| Dispositivo | CPU | RAM | Armazenamento | Função |
| ----------- | --- | --- | ------------- | ------ |
| **Dell OptiPlex 3050 Micro** | Intel Core i5 (6ª geração) | 8 GB | HDD 1 TB | Novo nó da infra — armazenamento em massa e capacidade x86 |

---

## Dell OptiPlex 3050 Micro

Mini PC (form factor **Micro** / mini) da linha OptiPlex, adicionado ao home server para ampliar capacidade de processamento e armazenamento em relação ao setup só com Raspberry Pi.

Com a chegada desta máquina, o **HDD de 1 TB** que antes ficava ligado ao Raspberry Pi 5 via hub USB passou a fazer parte deste nó. O Pi 5 ficou apenas com o **NVMe M.2 de 256 GB** (sistema operacional e alguns containers Docker) — detalhes em [pi5/v2](../raspberry_pi/pi5/v2/README.md).

### Especificações neste setup

A linha OptiPlex 3050 Micro usa chipset Intel B250, gráficos integrados Intel HD Graphics 530 (6ª geração), Ethernet Gigabit e suporte a até 32 GB de RAM (2× SODIMM). A unidade deste lab vem com a configuração:

| Item | Detalhe |
| ---- | ------- |
| **Modelo** | Dell OptiPlex 3050 Micro (mini) |
| **CPU** | Intel Core i5 — 6ª geração (Skylake) |
| **RAM** | 8 GB DDR4 |
| **Armazenamento** | HDD 1 TB |
| **Form factor** | Micro / mini desktop |

### Papel na infraestrutura

Nó x86 compacto com disco de massa de 1 TB, complementando o Raspberry Pi 5 (containers leves no NVMe) e o Raspberry Pi 4 (Home Assistant). O form factor Micro facilita manter tudo no mesmo rack/armário do homelab sem ocupar espaço de um desktop torre.

Hostname na rede: **`dell3050`**.

**Komodo:** neste host **não** roda o Core — só o **Periphery**. Criar e atualizar as stacks do Dell é feito pela UI do Core no Pi 5; o Periphery daqui executa o compose que está no disco desta máquina. Detalhes: [Komodo](../../apps/komodo/README.md).

Acesso local às apps deste nó: o cliente fala com o **NPM do Pi 5**, que **repassa** ao NPM deste host. Exposição à internet: túnel Cloudflare **deste** nó. Ver [estratégia entre NPMs](../../apps/stacks/raspberrypi/admin/README.md#estratégia-de-comunicação-entre-os-npms).

### Resumo do armazenamento

| Disco | Capacidade | Função |
| ----- | ---------- | ------ |
| HDD | 1 TB | Armazenamento em massa (mídia *Arr*, Seafile, etc.) |

### Containers em execução

Inventário a partir de `docker compose ls` / `docker ps` neste host. O Pi 4 (Home Assistant) não entra aqui.

| Stack / origem | Containers | Papel |
| -------------- | ---------- | ----- |
| **komodo** (Periphery) | `komodo-periphery` | Agente local. Sem UI — o Core fica no Pi 5 |
| **admin** | `nginx-proxy-manager`, `cloudflared_tunnel`, `filebrowser` | Proxy, túnel Cloudflare e navegação de arquivos |
| **games** | `romm`, `romm-db` | Gerenciador de ROMs |
| **immich** | `immich_server`, `immich_machine_learning`, `immich_postgres`, `immich_redis` | Fotos e vídeos |
| **stream** | `jellyfin`, `seerr`, `sonarr`, `radarr`, `bazarr`, `jackett`, `prowlarr`, `qbittorrent` | Pipeline *Arr + streaming (usa o HDD) |
| **seafile** | `seafile`, `seadoc`, `seafile-mysql`, `seafile-redis` | Arquivos sync / colaboração (Seafile) |

```
dell3050
├── komodo-periphery         ← só Periphery (UI do Komodo fica no raspberrypi)
├── admin                    → NPM, Cloudflare Tunnel, Filebrowser
├── games                    → RomM + MariaDB
├── immich                   → Immich (server, ML, Postgres, Redis/Valkey)
├── stream                   → *Arr + Jellyfin + Seerr + qBittorrent
└── seafile                  → Seafile + Seadoc + MariaDB + Redis
```

Documentação: [Komodo](../../apps/komodo/README.md) · [admin](../../apps/stacks/dell3050/admin/README.md) · [games](../../apps/stacks/dell3050/games/README.md) · [immich](../../apps/stacks/dell3050/immich/README.md) · [stream](../../apps/stacks/dell3050/stream/README.md) · [seafile](../../apps/stacks/dell3050/seafile/README.md). Inventário do Pi 5: [pi5/v2](../raspberry_pi/pi5/v2/README.md).

---

## Referências

- [OptiPlex 3050 Tower, SFF and Micro — technical specifications (PDF Dell)](https://i.dell.com/sites/csdocuments/shared-content_data-sheets_documents/en/optiplex-3050-towers-technical-specifications.pdf)
- [Dell OptiPlex 3050 Micro — ServeTheHome (TinyMiniMicro)](https://www.servethehome.com/dell-optiplex-3050-micro-project-tinyminimicro-guide-review/)
- [Raspberry Pi 5 — configuração atual (v2)](../raspberry_pi/pi5/v2/README.md)
