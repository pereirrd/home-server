# Hardware — Dell

Máquinas x86 adicionadas à infraestrutura do home server. A documentação de software (stacks, Portainer, Home Assistant) está no [README principal](../../README.md). Os Raspberry Pi continuam em [hardware/raspberry_pi](../raspberry_pi/README.md).

## Visão geral

| Dispositivo | CPU | RAM | Armazenamento | Função |
| ----------- | --- | --- | ------------- | ------ |
| **Dell OptiPlex 3050 Micro** | Intel Core i5 (6ª geração) | 8 GB | HDD 1 TB | Novo nó da infra — armazenamento em massa e capacidade x86 |

---

## Dell OptiPlex 3050 Micro

Mini PC (form factor **Micro** / mini) da linha OptiPlex, adicionado ao home server para ampliar capacidade de processamento e armazenamento em relação ao setup só com Raspberry Pi.

Com a chegada desta máquina, o **HDD de 1 TB** que antes ficava ligado ao Raspberry Pi 5 via hub USB passou a fazer parte deste nó. O Pi 5 ficou apenas com o **NVMe M.2 de 256 GB** (sistema operacional e alguns containers Docker) — detalhes em [pi5/v2](../raspberry_pi/pi5/v2/README.md).

### Especificações neste setup

| Item | Detalhe |
| ---- | ------- |
| **Modelo** | Dell OptiPlex 3050 Micro (mini) |
| **CPU** | Intel Core i5 — 6ª geração (Skylake) |
| **RAM** | 8 GB DDR4 |
| **Armazenamento** | HDD 1 TB |
| **Form factor** | Micro / mini desktop |

A linha OptiPlex 3050 Micro usa chipset Intel B250, gráficos integrados Intel HD Graphics 530 (6ª geração), Ethernet Gigabit e suporte a até 32 GB de RAM (2× SODIMM). A unidade deste lab vem com a configuração da tabela acima.

### Papel na infraestrutura

Nó x86 compacto com disco de massa de 1 TB, complementando o Raspberry Pi 5 (containers leves no NVMe) e o Raspberry Pi 4 (Home Assistant). O form factor Micro facilita manter tudo no mesmo rack/armário do homelab sem ocupar espaço de um desktop torre.

Hostname na rede: **`dell3050`**.

**Portainer:** neste host **não** roda o Portainer CE — só o **`portainer-agent`**. O gerenciamento das stacks do Dell (criar/editar stacks, containers, etc.) é feito pela **única** UI do Portainer no Pi 5, apontando para este agent. Detalhes: [Portainer — CE + Agent](../../apps/portainer/README.md#uma-única-instância--ce-no-pi-5-agent-no-dell).

Acesso local às apps deste nó: o cliente fala com o **NPM do Pi 5**, que **repassa** ao NPM deste host. Exposição à internet: túnel Cloudflare **deste** nó. Ver [estratégia entre NPMs](../../apps/stacks/admin/README.md#estratégia-de-comunicação-entre-os-npms).

### Resumo do armazenamento

| Disco | Capacidade | Função |
| ----- | ---------- | ------ |
| HDD | 1 TB | Armazenamento em massa (mídia *Arr*, Seafile, etc.) |

### Containers em execução

Inventário a partir de `docker compose ls` / `docker ps` neste host. O Pi 4 (Home Assistant) não entra aqui.

| Stack / origem | Containers | Papel |
| -------------- | ---------- | ----- |
| **admin** | `nginx-proxy-manager`, `cloudflared_tunnel`, `filebrowser`, `portainer-backup` | Proxy, túnel Cloudflare, navegação de arquivos e backup do Portainer |
| **stream** | `jellyfin`, `seerr`, `sonarr`, `radarr`, `bazarr`, `jackett`, `prowlarr`, `qbittorrent` | Pipeline *Arr + streaming (usa o HDD) |
| **seafile** | `seafile`, `seadoc`, `seafile-mysql`, `seafile-redis` | Arquivos sync / colaboração (Seafile) |
| *(sem stack Compose)* | `portainer-agent` | Agent remoto — **não** é um segundo Portainer; a UI fica no Pi 5 |

```
dell3050
├── portainer-agent          ← só Agent (UI do Portainer fica no raspberrypi)
├── admin                    → NPM, Cloudflare Tunnel, Filebrowser, portainer-backup
├── stream                   → *Arr + Jellyfin + Seerr + qBittorrent
└── seafile                  → Seafile + Seadoc + MariaDB + Redis
```

Documentação das stacks: [admin](../../apps/stacks/admin/README.md) · [compose admin no Dell](../../apps/stacks/admin/dell/) · [stream](../../apps/stacks/stream/README.md) · [seafile](../../apps/stacks/seafile/) (compose + env). Inventário do Pi 5: [pi5/v2](../raspberry_pi/pi5/v2/README.md).

---

## Referências

- [OptiPlex 3050 Tower, SFF and Micro — technical specifications (PDF Dell)](https://i.dell.com/sites/csdocuments/shared-content_data-sheets_documents/en/optiplex-3050-towers-technical-specifications.pdf)
- [Dell OptiPlex 3050 Micro — ServeTheHome (TinyMiniMicro)](https://www.servethehome.com/dell-optiplex-3050-micro-project-tinyminimicro-guide-review/)
- [Raspberry Pi 5 — configuração atual (v2)](../raspberry_pi/pi5/v2/README.md)
