# ESP32 — Bluetooth Tracker (ESPHome)

Este diretório contém a configuração para transformar um ESP32 em um **Bluetooth proxy** integrado ao [Home Assistant](https://www.home-assistant.io/). O dispositivo estende o alcance do Bluetooth Low Energy (BLE) da sua instância do Home Assistant, permitindo rastrear tags, sensores e outros aparelhos BLE que estão longe do servidor principal.

## Como funciona

O Home Assistant centraliza a descoberta de dispositivos BLE, mas o alcance do rádio Bluetooth do servidor (Raspberry Pi, NUC, etc.) costuma ser limitado. Um ESP32 com firmware ESPHome atua como **proxy**: escuta anúncios BLE na área onde está instalado e repassa os dados para o Home Assistant via Wi-Fi.

```
[ Dispositivo BLE ]  ~~BLE~~>  [ ESP32 proxy ]  ~~Wi-Fi~~>  [ Home Assistant ]
```

O arquivo principal da configuração é [`projects/bluetooth_tracker.yaml`](projects/bluetooth_tracker.yaml). Ele define:

| Componente | Função |
|---|---|
| `bluetooth_proxy` | Ativa o proxy BLE (função principal do projeto) |
| `esp32` + `esp-idf` | Hardware e framework; `variant` define o chip (ex.: `esp32`, `esp32s3`) — ver [configuration variables](https://esphome.io/components/esp32/#configuration-variables) |
| `wifi` | Conexão à rede local (`power_save_mode: none` mantém o proxy responsivo) |
| `api` | Integração nativa com o Home Assistant |
| `ota` | Atualizações de firmware via Wi-Fi, sem cabo USB |
| `improv_serial` | Configuração inicial de Wi-Fi via USB (protocolo Improv) |
| `web_server` | Interface web local para logs e status |
| `captive_portal` | Portal de fallback quando o Wi-Fi falha |

Credenciais e identificação do dispositivo ficam em [`projects/secrets.yaml`](projects/secrets.yaml) — **não commite este arquivo** com dados reais.

Use o arquivo  [`projects/bluetooth_tracker.yaml`](projects/bluetooth_tracker.yaml) como template para gravação em seus dispositivos ESP32. Veja as orientações à baixo sobre a gravação. Após instalado o gravador via Docker Compose acesse a aplicação via browser e crie um novo dispositivo colando o arquivo template com os dados do seu projeto.

## Pré-requisitos

- ESP32 compatível com Bluetooth proxy (chip padrão `esp32`; confira outras variantes na [documentação do componente ESP32](https://esphome.io/components/esp32/#configuration-variables))
- Cabo USB para a primeira gravação. É necessário um cado de qualidade, próprio para transferência de dados. Cabos de carregador de celular não irão funcionar para a gravação.
- Home Assistant com a integração [ESPHome](https://www.home-assistant.io/integrations/esphome/) habilitada
- Docker e Docker Compose (para compilar e gravar via container)

## Configuração

1. Copie ou edite `projects/secrets.yaml` com os valores do seu ambiente:

   ```yaml
   device_name: "esp32-ble-tracker-example"
   device_friendly_name: "ESP32 BLE Tracker Example"
   wifi_ssid: "your_wifi_ssid"
   wifi_password: "your_wifi_password"
   ```

2. Confira `esp32.variant` em `bluetooth_tracker.yaml` e ajuste conforme o chip da sua placa. O valor padrão é `esp32` (ESP32 clássico). Para S2, S3, C3 ou outras variantes, consulte a lista completa em [ESPHome — ESP32 configuration variables](https://esphome.io/components/esp32/#configuration-variables).

3. (Opcional) Descomente o bloco `manual_ip` em `bluetooth_tracker.yaml` e use os valores de `static_ip`, `gateway` e `subnet` do `secrets.yaml` se quiser IP fixo.

## Gravação com Docker Compose

A forma mais simples de compilar e gravar o firmware é subir o ESPHome em um container. Não é necessário instalar o ESPHome localmente — o Docker cuida do ambiente de build.

```bash
cd esp32
docker compose up -d
```

O serviço fica disponível em **http://localhost:6052**. A partir da interface web:

1. Abra o dashboard do ESPHome
2. Selecione o dispositivo (`bluetooth_tracker.yaml`)
3. Clique em **Install** e escolha a porta serial do ESP32 conectado via USB

Após a primeira gravação, o dispositivo conecta ao Wi-Fi e aparece automaticamente no Home Assistant. Gravações e atualizações seguintes podem ser feitas **OTA** (sem cabo), direto pelo dashboard ou pelo Home Assistant.

### Por que `privileged: true` e `network_mode: host`?

- **`privileged`** — permite acesso à porta serial USB do ESP32 a partir do container
- **`network_mode: host`** — o ESPHome descobre dispositivos na rede local e se comunica corretamente com o Home Assistant

### Comandos úteis

```bash
# Subir o ESPHome
docker compose up -d

# Ver logs
docker compose logs -f esphome

# Parar
docker compose down
```

## Estrutura do diretório

```
esp32/
├── docker-compose.yaml          # ESPHome via Docker (build + flash + OTA)
├── README.md
└── projects/
    ├── bluetooth_tracker.yaml   # Configuração do dispositivo
    └── secrets.yaml             # Credenciais e nomes (local, não versionar)
```

## Integração com o Home Assistant

Depois que o ESP32 estiver online:

1. Vá em **Configurações → Dispositivos e serviços → ESPHome**
2. Confirme que o dispositivo foi descoberto (ou adicione manualmente pelo host/IP)
3. Em **Configurações → Dispositivos e serviços → Bluetooth**, verifique se o proxy aparece como **Ativo**
4. Dispositivos BLE próximos ao ESP32 passam a ser detectados pelo Home Assistant

Para múltiplos cômodos, crie uma cópia de `bluetooth_tracker.yaml` com outro `device_name` no `secrets.yaml` e grave um ESP32 por área.

## Referências

Consulte a documentação oficial abaixo para detalhes atualizados sobre componentes, integração e rastreamento BLE:

| Recurso | Descrição |
|---|---|
| [ESPHome — ESP32](https://esphome.io/components/esp32/#configuration-variables) | Variantes de chip (`variant`), framework e demais opções de hardware |
| [ESPHome](https://esphome.io/) | Documentação principal — instalação, componentes, guias e referência YAML |
| [ESPHome Projects](https://esphome.io/projects/) | Projetos prontos e exemplos de configuração para casos de uso comuns |
| [Home Assistant — ESPHome](https://www.home-assistant.io/integrations/esphome) | Integração oficial do ESPHome com o Home Assistant |
| [Bermuda](https://github.com/agittins/bermuda) | Integração HACS para triangulação/trilateração BLE usando proxies ESPHome |
