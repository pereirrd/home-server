# Home Assistant

Documentação das aplicações e integrações instaladas no [Home Assistant](https://www.home-assistant.io/) do home server. Apps (anteriormente *add-ons*) são gerenciados em **Settings → Apps**; integrações customizadas como Bermuda e Homelable são instaladas via **HACS**.

> **Importante:** este diretório documenta o que está instalado na instância. A configuração detalhada de cada app deve seguir a documentação oficial referenciada abaixo.

## Repositórios de origem

| Origem                                   | Apps / integrações instalados |
| ---------------------------------------- | ----------------------------- |
| Repositório oficial de Apps do HA        | AdGuard Home, Uptime Kuma     |
| Comunidade (via repositório customizado) | HACS, Immich                  |
| HACS (integrações customizadas)          | Bermuda, Homelable            |

Para instalar apps de repositórios da comunidade, é necessário adicionar a URL do repositório em **Settings → Apps → Repositories** no Home Assistant. O **HACS** estende essa capacidade para integrações, temas e componentes customizados além dos apps em si.

## Aplicações

### HACS

O [Home Assistant Community Store (HACS)](https://hacs.xyz/) é a loja da comunidade para o Home Assistant. Permite descobrir, instalar e atualizar integrações, dashboards, temas e templates mantidos por terceiros — conteúdo que não faz parte do repositório oficial de Apps.

Nesta instalação, o HACS é a **base para expandir o HA com repositórios da comunidade**, incluindo o repositório que disponibiliza o app Immich e as integrações Bermuda e Homelable.

**Referência:** [HACS — Configuração inicial](https://hacs.xyz/docs/use/configuration/basic/)

### AdGuard Home

[AdGuard Home](https://adguard.com/adguard-home/overview.html) é um servidor DNS de bloqueio de anúncios e rastreadores em nível de rede, com controle parental e interface web para gerenciar filtros. Atua como DNS da rede local, protegendo todos os dispositivos conectados sem instalar software em cada um.

Instalado a partir do **repositório oficial de Apps do Home Assistant** (mantido por [hassio-addons](https://github.com/hassio-addons)).

**Referência:** [AdGuard Home — Home Assistant Community App](https://github.com/hassio-addons/app-adguard-home)

### Uptime Kuma

[Uptime Kuma](https://uptime.kuma.pet/) é uma ferramenta de monitoramento self-hosted, comparável a serviços comerciais como Uptime Robot. Monitora disponibilidade de serviços via HTTP/S, TCP, DNS e outros protocolos, com dashboard visual e notificações em caso de indisponibilidade.

Útil para acompanhar a saúde dos serviços do home server (stacks Docker, apps do HA, endpoints expostos via túnel). Também pode disparar webhooks de automação no Home Assistant.

Instalado a partir do **repositório oficial de Apps do Home Assistant**.

**Referência:** [Uptime Kuma — Home Assistant Community App](https://github.com/hassio-addons/app-uptime-kuma)

### Immich

[Immich](https://immich.app/) é uma solução self-hosted de alto desempenho para gerenciamento de fotos e vídeos — alternativa open source a serviços de nuvem como Google Photos. Oferece backup automático de dispositivos móveis, reconhecimento facial, álbuns compartilhados, busca por metadados e timeline de mídia.

Instalado a partir do repositório da comunidade [fabio-garavini/hassio-addons](https://github.com/fabio-garavini/hassio-addons), que empacota diversos serviços populares como apps do Home Assistant.

**Referências:**

- [fabio-garavini/hassio-addons](https://github.com/fabio-garavini/hassio-addons)
- [Immich — Documentação](https://immich.app/docs)

### Bermuda

[Bermuda](https://github.com/agittins/bermuda) é uma integração customizada para o Home Assistant que rastreia a presença e a localização de dispositivos Bluetooth Low Energy (BLE) dentro e perto da casa. Processa os anúncios BLE coletados pelo HA — vindos de proxies ESPHome, integração Bluetooth ou dispositivos Shelly Plus — e estima em qual **Área** (cômodo) cada dispositivo está, com base na intensidade do sinal (RSSI) recebido por cada scanner.

Para cada dispositivo rastreado, a integração expõe:

| Entidade | Função |
|---|---|
| `device_tracker` | Estado home/away |
| Sensor de Área | Nome da área mais próxima (proxy associado a um cômodo no HA) |
| Sensor de Distância | Estimativa de distância em relação ao proxy mais próximo |

Funciona bem com os proxies ESP32 documentados em [`automations/esp32`](../../automations/esp32/README.md): quanto mais scanners distribuídos pela casa, melhor a precisão. Para telefones Android/iOS com MAC randomizado, é necessário configurar o componente **Private BLE Device** do Home Assistant.

Instalado via **HACS** (integração customizada, não é um App/add-on).

**Referências:**

- [Bermuda — Wiki oficial](https://github.com/agittins/bermuda/wiki/)
- [agittins/bermuda](https://github.com/agittins/bermuda)

### Homelable

[Homelable](https://github.com/Pouzor/homelable) é um visualizador self-hosted de infraestrutura de homelab. Permite montar um **diagrama interativo da rede** com monitoramento de status em tempo real (online/offline) dos serviços e dispositivos do home server.

Principais recursos:

- **Scanner de rede** — descobre hosts e serviços via `nmap` nos ranges CIDR configurados e popula uma fila de dispositivos pendentes para aprovação no canvas
- **Health checks** — ping, HTTP/S, TCP, SSH, Prometheus e `/health` por nó
- **Importação Zigbee/Z-Wave** — topologia a partir de Zigbee2MQTT e Z-Wave JS UI via MQTT
- **Estilos personalizáveis** — diagrama exportável como PNG

Nesta instalação, roda como **integração/app do Home Assistant via HACS** (versão HA do projeto), complementando o mapa visual da infraestrutura documentada nas [stacks Docker](../stacks/admin/README.md) e demais serviços do home server.

**Referências:**

- [Pouzor/homelable](https://github.com/Pouzor/homelable)
- [Homelable — FEATURES.md](https://github.com/Pouzor/homelable/blob/main/FEATURES.md)
- [Homelable — INSTALLATION.md](https://github.com/Pouzor/homelable/blob/main/INSTALLATION.md)

## Integração com o home server

O Home Assistant centraliza automações e monitoramento da casa. Os proxies ESP32 documentados em [`automations/esp32`](../../automations/esp32/README.md) integram-se ao HA via ESPHome para estender o alcance Bluetooth (BLE); a integração **Bermuda** usa esses dados para estimar presença e cômodo de cada dispositivo BLE. O **Homelable** complementa essa visão com um mapa interativo dos dispositivos e serviços de rede do home server.

Para adicionar um novo app de repositório da comunidade:

1. **Settings → Apps → Repositories → Add**
2. Informe a URL do repositório GitHub (ex.: `https://github.com/fabio-garavini/hassio-addons`)
3. Instale o app desejado em **Settings → Apps → Install**

Consulte sempre a documentação oficial de cada app antes de alterar configurações de rede, DNS ou volumes de armazenamento.

## Referências

- [Home Assistant](https://www.home-assistant.io/)
- [HACS](https://hacs.xyz/)
- [HACS — Configuração inicial](https://hacs.xyz/docs/use/configuration/basic/)
- [AdGuard Home](https://adguard.com/adguard-home/overview.html)
- [hassio-addons](https://github.com/hassio-addons)
- [AdGuard Home — Home Assistant Community App](https://github.com/hassio-addons/app-adguard-home)
- [Uptime Kuma](https://uptime.kuma.pet/)
- [Uptime Kuma — Home Assistant Community App](https://github.com/hassio-addons/app-uptime-kuma)
- [fabio-garavini/hassio-addons](https://github.com/fabio-garavini/hassio-addons)
- [Immich](https://immich.app/)
- [Immich — Documentação](https://immich.app/docs)
- [Bermuda — Wiki oficial](https://github.com/agittins/bermuda/wiki/)
- [agittins/bermuda](https://github.com/agittins/bermuda)
- [Pouzor/homelable](https://github.com/Pouzor/homelable)
- [Homelable — FEATURES.md](https://github.com/Pouzor/homelable/blob/main/FEATURES.md)
- [Homelable — INSTALLATION.md](https://github.com/Pouzor/homelable/blob/main/INSTALLATION.md)
- [ESP32 Bluetooth Tracker](../../automations/esp32/README.md)
