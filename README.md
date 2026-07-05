# home-server

Repositório para registrar minha experiência montando e operando um **home server self-hosted**. Aqui documento o que instalei, por que escolhi cada ferramenta e como as peças se encaixam — não é um repositório para armazenar configurações de produção.

## Sobre este repositório

Este projeto tem caráter **educativo**, de **contribuição** e **diversão** para entusiastas de automação residencial, mídia, jogos retro, containers e IoT. Os arquivos `docker-compose` e `env.example` presentes aqui são **exemplos ilustrativos** da minha stack em funcionamento — servem como ponto de partida para estudo, não como receita pronta para copiar e colar somente.

Cada ambiente é único: hardware, rede, storage e necessidades diferem. O que funciona aqui pode precisar de adaptação no seu setup.

## Antes de começar

A documentação deste repositório é um **guia rápido**. Para configurar qualquer serviço corretamente, é **imprescindível** que você:

1. **Leia a documentação oficial** de cada aplicação — links incluídos no final de cada documento.
2. **Busque bons tutoriais** no Google e no YouTube.
3. **Entenda os conceitos** por trás de cada ferramenta antes de expor serviços na internet.

O universo de home server é **muito vasto**. Além do que está documentado aqui, existem dezenas de alternativas, integrações e projetos open source igualmente interessantes e úteis — vale explorar, experimentar e compartilhar o que aprender.

## Hardware

A infraestrutura de software documentada neste repositório roda em dois dispositivos separados.

### Raspberry Pi 5 — containers e stacks

Responsável pelo [Portainer](apps/portainer/README.md) e por todas as [stacks Docker](apps/stacks/admin/README.md). Sistema operacional [Ubuntu Trixie 13](https://ubuntu.com/) com gerenciamento via [Webmin](https://webmin.com/) — interface web para administração do SO (usuários, serviços, arquivos, rede, entre outros).

**Referências:** [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/) · [Webmin](https://webmin.com/)

### Raspberry Pi 4 — automação residencial

Dedicado ao [Home Assistant](apps/home_assistant/README.md). O HA roda **diretamente no dispositivo como sistema operacional**, no modo **Supervisor** (instalação gerenciada com suporte nativo a apps/add-ons).

**Referência:** [Raspberry Pi 4 Model B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)

## Visão geral

O home server combina serviços de infraestrutura, mídia, jogos, automação residencial e IoT, organizados da seguinte forma:

```
Portainer (gerenciador)
    ├── Stack Admin      → backup + túnel Cloudflare
    ├── Stack Stream     → pipeline *Arr + Jellyfin
    └── Stack Games      → RomM

Home Assistant (apps)    → DNS, monitoramento, fotos, HACS
Automations ESP32        → proxies Bluetooth para o HA
```


| Área               | Função                                                    |
| ------------------ | --------------------------------------------------------- |
| **Portainer**      | Gerenciamento de containers Docker (Raspberry Pi 5)       |
| **Stacks**         | Serviços agrupados por contexto, deploy via Portainer     |
| **Home Assistant** | Automação residencial e apps integrados (Raspberry Pi 4)  |
| **ESP32**          | Extensão de alcance BLE para rastreamento de dispositivos |




## Documentação



### Infraestrutura


| Documento                             | Descrição                                                      |
| ------------------------------------- | -------------------------------------------------------------- |
| [Portainer](apps/portainer/README.md) | Gerenciador de containers — ponto de partida da infraestrutura |




### Stacks Docker


| Stack                                  | Descrição                                                            |
| -------------------------------------- | -------------------------------------------------------------------- |
| [admin](apps/stacks/admin/README.md)   | Backup do Portainer e Cloudflare Tunnel                              |
| [stream](apps/stacks/stream/README.md) | Pipeline *Arr (Sonarr, Radarr, Bazarr, Jellyfin, Seerr, qBittorrent) |
| [games](apps/stacks/games/README.md)   | RomM — gerenciador de ROMs no browser                                |




### Automação e IoT


| Documento                                                | Descrição                                                |
| -------------------------------------------------------- | -------------------------------------------------------- |
| [Home Assistant](apps/home_assistant/README.md)          | Apps instalados: HACS, AdGuard Home, Uptime Kuma, Immich |
| [ESP32 — Bluetooth Tracker](automations/esp32/README.md) | Proxies BLE integrados ao Home Assistant via ESPHome     |




## Estrutura do repositório

```
home-server/
├── apps/
│   ├── portainer/           # Gerenciador Docker (iniciado via terminal)
│   ├── home_assistant/      # Documentação dos apps do HA
│   └── stacks/
│       ├── admin/           # Ferramentas administrativas
│       ├── games/           # Jogos retro (RomM)
│       └── stream/          # Mídia e streaming (*Arr + Jellyfin)
├── automations/
│   └── esp32/               # Proxies Bluetooth (ESPHome)
└── README.md
```



## Contribuições

Sugestões, correções e ideias são bem-vindas via issues e pull requests. Se este repositório ajudar alguém a dar os primeiros passos no mundo self-hosted, já cumpriu o objetivo.

---

*Lembre-se: a melhor documentação sempre será a oficial de cada projeto. Este repositório é um mapa da minha jornada — o caminho detalhado, você constrói com as fontes primárias.*