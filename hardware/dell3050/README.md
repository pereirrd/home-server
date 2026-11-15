# Hardware — Dell

Máquinas x86 adicionadas à infraestrutura do home server. A documentação de software (stacks, Komodo, Home Assistant) está no [README principal](../../README.md). Os Raspberry Pi continuam em [hardware/raspberry_pi](../raspberry_pi/README.md).

## Visão geral

| Dispositivo | CPU | RAM | Armazenamento | Função |
| ----------- | --- | --- | ------------- | ------ |
| **Dell OptiPlex 3050 Micro** | Intel Core i5 (6ª geração) | 20 GB (16+4), até 32 GB | M.2 256 GB + SSD SATA 256 GB + HDD 1 TB (USB) | Entretenimento e utilidades |

---

## Dell OptiPlex 3050 Micro

Mini PC (form factor **Micro** / mini) da linha OptiPlex, adicionado ao home server para ampliar capacidade de processamento e armazenamento em relação ao setup só com Raspberry Pi.

O **HDD de 2,5" e 1 TB** que antes ficava no Raspberry Pi 5 passou para este nó, agora numa gaveta USB. O Pi 5 ficou apenas com o **NVMe M.2 de 256 GB** — detalhes em [pi5/v2](../raspberry_pi/pi5/v2/README.md).

### Especificações neste setup

A linha OptiPlex 3050 Micro usa chipset Intel B250, gráficos integrados Intel HD Graphics 530 (6ª geração), Ethernet Gigabit e suporte a até 32 GB de RAM (2× SODIMM). A unidade deste lab vem com a configuração:

| Item | Detalhe |
| ---- | ------- |
| **Modelo** | Dell OptiPlex 3050 Micro (mini) |
| **CPU** | Intel Core i5 — 6ª geração (Skylake) |
| **RAM** | 20 GB DDR4 (módulo de 16 GB + módulo de 4 GB). O teto da placa é 32 GB, em 2× SODIMM |
| **Disco do sistema** | M.2 NVMe 256 GB |
| **Disco de dados** | SSD 256 GB na baia SATA interna |
| **Disco externo** | HDD 2,5" 1 TB em gaveta USB 3.0 |
| **Form factor** | Micro / mini desktop |

### Papel na infraestrutura

Hospeda as aplicações de entretenimento e as utilidades: jogos, fotos, streaming e arquivos. O form factor Micro facilita manter a máquina no mesmo rack do homelab; o volume grande fica fora do chassi, na gaveta USB.

Hostname na rede: **`dell3050`**.

**Komodo:** neste host **não** roda o Core — só o **Periphery**. Criar e atualizar as stacks do Dell é feito pela UI do Core no Pi 5; o Periphery daqui executa o compose que está no disco desta máquina. Detalhes: [Komodo](../../apps/stacks/raspberrypi/komodo/README.md).

Acesso local às apps deste nó: o cliente fala com o **NPM do Pi 5**, que **repassa** ao NPM deste host. Exposição à internet: túnel Cloudflare **deste** nó. Ver [estratégia entre NPMs](../../apps/stacks/raspberrypi/admin/README.md#estratégia-de-comunicação-entre-os-npms).

### Armazenamento

Cada disco tem um papel fixo. O SSD SATA guarda só o Seafile e o Immich. A biblioteca de vídeo e os backups ficam no HDD.

| Disco | Interface | Capacidade | Função |
| ----- | --------- | ---------- | ------ |
| M.2 NVMe | Slot M.2 da placa | 256 GB | Sistema operacional e os arquivos deste nó (compose, configs das stacks) |
| SSD | SATA interna (baia 2,5") | 256 GB | **Só** [Seafile](../../apps/stacks/dell3050/seafile/README.md) e [Immich](../../apps/stacks/dell3050/immich/README.md): biblioteca, bancos e dados dessas duas stacks |
| HDD 2,5" | USB 3.0 | 1 TB | Backup do nó e mídia da stack [stream](../../apps/stacks/dell3050/stream/README.md) (*Arr* + Jellyfin) |

O HDD fica numa **gaveta Ugreen para HD de 3,5"**, com fonte própria, que alimenta o disco. A gaveta está preparada para um disco de grande volume no futuro; hoje ela carrega este HDD de 2,5" e 1 TB.

### Containers em execução

Inventário a partir de `docker compose ls` / `docker ps` neste host. O Pi 4 (Home Assistant) não entra aqui.

| Stack / origem | Containers | Papel |
| -------------- | ---------- | ----- |
| **komodo** (Periphery) | `komodo-periphery` | Agente local. Sem UI — o Core fica no Pi 5 |
| **admin** | `nginx-proxy-manager`, `cloudflared_tunnel`, `filebrowser` | Proxy, túnel Cloudflare e navegação de arquivos |
| **games** | `romm`, `romm-db` | Gerenciador de ROMs |
| **immich** | `immich_server`, `immich_machine_learning`, `immich_postgres`, `immich_redis` | Fotos e vídeos — dados no SSD SATA |
| **stream** | `jellyfin`, `seerr`, `sonarr`, `radarr`, `bazarr`, `jackett`, `prowlarr`, `qbittorrent` | Pipeline *Arr + streaming — biblioteca no HDD USB |
| **seafile** | `seafile`, `seadoc`, `seafile-mysql`, `seafile-redis` | Arquivos sync — dados no SSD SATA, junto com o Immich |

```
dell3050
├── komodo-periphery         ← só Periphery (UI do Komodo fica no raspberrypi)
├── admin                    → NPM, Cloudflare Tunnel, Filebrowser
├── games                    → RomM + MariaDB
├── immich                   → Immich (server, ML, Postgres, Redis/Valkey)
├── stream                   → *Arr + Jellyfin + Seerr + qBittorrent
└── seafile                  → Seafile + Seadoc + MariaDB + Redis
```

Documentação: [Komodo](../../apps/stacks/raspberrypi/komodo/README.md) · [admin](../../apps/stacks/dell3050/admin/README.md) · [games](../../apps/stacks/dell3050/games/README.md) · [immich](../../apps/stacks/dell3050/immich/README.md) · [stream](../../apps/stacks/dell3050/stream/README.md) · [seafile](../../apps/stacks/dell3050/seafile/README.md). Inventário do Pi 5: [pi5/v2](../raspberry_pi/pi5/v2/README.md).

---

## Referências

- [OptiPlex 3050 Tower, SFF and Micro — technical specifications (PDF Dell)](https://i.dell.com/sites/csdocuments/shared-content_data-sheets_documents/en/optiplex-3050-towers-technical-specifications.pdf)
- [Dell OptiPlex 3050 Micro — ServeTheHome (TinyMiniMicro)](https://www.servethehome.com/dell-optiplex-3050-micro-project-tinyminimicro-guide-review/)
- [Raspberry Pi 5 — configuração atual (v2)](../raspberry_pi/pi5/v2/README.md)
