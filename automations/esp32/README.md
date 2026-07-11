# ESP32 — Automations (ESPHome)

Este diretório contém configurações ESPHome para dispositivos ESP32 usados com o [Home Assistant](https://www.home-assistant.io/). Cada projeto fica em `projects/<nome>/` com o YAML do dispositivo e um `secrets.yaml.example` (credenciais locais em `secrets.yaml` — **não versionar**).

## Projetos

| Projeto | Função |
|---|---|
| [`bluetooth_tracker`](projects/bluetooth_tracker/) | Bluetooth proxy — estende o alcance BLE do Home Assistant |
| [`irk-capture`](projects/irk-capture/) | Captura IRKs de dispositivos Apple/Android para presence detection |

---

## Bluetooth Tracker

Transforma um ESP32 em um **Bluetooth proxy** integrado ao Home Assistant. O dispositivo escuta anúncios BLE na área onde está instalado e repassa os dados para o Home Assistant via Wi-Fi.

```
[ Dispositivo BLE ]  ~~BLE~~>  [ ESP32 proxy ]  ~~Wi-Fi~~>  [ Home Assistant ]
```

O arquivo principal é [`projects/bluetooth_tracker/bluetooth_tracker.yaml`](projects/bluetooth_tracker/bluetooth_tracker.yaml). Ele define:

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

### Configuração

1. Copie o template e preencha com os valores do seu ambiente:

   ```bash
   cp projects/bluetooth_tracker/secrets.yaml.example projects/bluetooth_tracker/secrets.yaml
   ```

2. Confira `esp32.variant` em `bluetooth_tracker.yaml` e ajuste conforme o chip da sua placa. O valor padrão é `esp32` (ESP32 clássico). Para S2, S3, C3 ou outras variantes, consulte a lista completa em [ESPHome — ESP32 configuration variables](https://esphome.io/components/esp32/#configuration-variables).

3. (Opcional) Descomente o bloco `manual_ip` em `bluetooth_tracker.yaml` e use os valores de `static_ip`, `gateway` e `subnet` do `secrets.yaml` se quiser IP fixo.

### Integração com o Home Assistant

Depois que o ESP32 estiver online:

1. Vá em **Configurações → Dispositivos e serviços → ESPHome**
2. Confirme que o dispositivo foi descoberto (ou adicione manualmente pelo host/IP)
3. Em **Configurações → Dispositivos e serviços → Bluetooth**, verifique se o proxy aparece como **Ativo**
4. Dispositivos BLE próximos ao ESP32 passam a ser detectados pelo Home Assistant

Para múltiplos cômodos, crie uma cópia de `bluetooth_tracker.yaml` com outro `device_name` no `secrets.yaml` e grave um ESP32 por área.

---

## IRK Capture

Captura **Identity Resolving Keys (IRK)** de dispositivos Apple e Android via Bluetooth. Telefones e relógios modernos randomizam o MAC BLE periodicamente; com o IRK, o Home Assistant consegue resolver esses endereços e fazer presence detection confiável (ex.: [Private BLE Device](https://www.home-assistant.io/integrations/private_ble_device/) + [Bermuda](https://github.com/agittins/bermuda)).

Este projeto **não** funciona como Bluetooth proxy ao mesmo tempo. Use um ESP32 sobressalente ou grave temporariamente no lugar de um proxy e depois restaure o firmware original.

Pacote remoto: [DerekSeaman/irk-capture](https://github.com/DerekSeaman/irk-capture) (requer framework **ESP-IDF**).

```
[ iPhone / Android ]  ~~BLE pair~~>  [ ESP32 IRK Capture ]  ~~Wi-Fi~~>  [ Home Assistant ]
                                              |
                                         IRK capturado
                                              v
                              Private BLE Device / Bermuda
```

O arquivo principal é [`projects/irk-capture/irk-capture.yaml`](projects/irk-capture/irk-capture.yaml). Ele puxa o pacote base do GitHub e sobrescreve Wi-Fi/credenciais locais:

| Componente | Função |
|---|---|
| `packages` → `irk-capture-base.yaml` | Componente `irk_capture`, NimBLE, sensores e controles no HA |
| `substitutions` | Nome do dispositivo, board/variant, chaves API/OTA e nome BLE |
| `wifi` + `ap` | Rede local e AP de fallback (`wifi_captive`) |

### Configuração

1. Copie o template e preencha com os valores do seu ambiente:

   ```bash
   cp projects/irk-capture/secrets.yaml.example projects/irk-capture/secrets.yaml
   ```

2. Ajuste `esp32_variant` e `esp32_board` em `irk-capture.yaml` conforme a sua placa (padrão: `esp32c3` / `esp32-c3-devkitm-1`).

3. Gere `api_key` e `ota_password` no assistente de novo dispositivo do ESPHome e coloque-os no `secrets.yaml`.

### Uso (resumo)

1. Grave o firmware e reinicie o ESP32 (randomiza o MAC BLE).
2. No Home Assistant, escolha o **BLE Profile**:
   - **Heart Sensor** — Apple (iPhone, Watch, iPad) e relógios Android
   - **Keyboard** — telefones Android (Samsung, Pixel, etc.)
3. Pareie o telefone/relógio com o dispositivo anunciado (nome padrão: `IRK Capture` ou `Logitech K380` no perfil Keyboard).
4. Leia o IRK no sensor de texto **IRK** (ou nos logs do ESPHome).
5. Esqueça o pareamento no telefone/relógio e cole o IRK na integração **Private BLE Device**.

Guia completo, troubleshooting e dispositivos testados: [DerekSeaman/irk-capture](https://github.com/DerekSeaman/irk-capture).

---

## Pré-requisitos

- ESP32 compatível com o projeto escolhido (Bluetooth; confira `variant`/`board` na [documentação do componente ESP32](https://esphome.io/components/esp32/#configuration-variables))
- Cabo USB para a primeira gravação. É necessário um cabo de qualidade, próprio para transferência de dados. Cabos de carregador de celular não irão funcionar para a gravação.
- Home Assistant com a integração [ESPHome](https://www.home-assistant.io/integrations/esphome/) habilitada
- Docker e Docker Compose (para compilar e gravar via container)

## Gravação com Docker Compose

A forma mais simples de compilar e gravar o firmware é subir o ESPHome em um container. Não é necessário instalar o ESPHome localmente — o Docker cuida do ambiente de build.

```bash
cd automations/esp32
docker compose up -d
```

O serviço fica disponível em **http://localhost:6052**. A partir da interface web:

1. Abra o dashboard do ESPHome
2. Selecione o dispositivo (`bluetooth_tracker.yaml` ou `irk-capture.yaml`)
3. Clique em **Install** e escolha a porta serial do ESP32 conectado via USB

Após a primeira gravação, o dispositivo conecta ao Wi-Fi e aparece automaticamente no Home Assistant. Gravações e atualizações seguintes podem ser feitas **OTA** (sem cabo), direto pelo dashboard ou pelo Home Assistant.

### Por que `privileged: true` e `network_mode: host`?

- **`privileged`** — permite acesso à porta serial USB do ESP32 a partir do container
- **`network_mode: host`** — o ESPHome descobre dispositivos na rede local e se comunica corretamente com o Home Assistant

### Comandos úteis

Execute a partir de `automations/esp32`:

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
home-server/
└── automations/
    └── esp32/
        ├── docker-compose.yaml
        ├── README.md
        └── projects/
            ├── bluetooth_tracker/
            │   ├── bluetooth_tracker.yaml
            │   ├── secrets.yaml.example
            │   └── secrets.yaml             # local (não versionar)
            └── irk-capture/
                ├── irk-capture.yaml
                ├── secrets.yaml.example
                └── secrets.yaml             # local (não versionar)
```

## Referências

| Recurso | Descrição |
|---|---|
| [ESPHome — ESP32](https://esphome.io/components/esp32/#configuration-variables) | Variantes de chip (`variant`), framework e demais opções de hardware |
| [ESPHome](https://esphome.io/) | Documentação principal — instalação, componentes, guias e referência YAML |
| [ESPHome Projects](https://esphome.io/projects/) | Projetos prontos e exemplos de configuração para casos de uso comuns |
| [Home Assistant — ESPHome](https://www.home-assistant.io/integrations/esphome) | Integração oficial do ESPHome com o Home Assistant |
| [Home Assistant — Private BLE Device](https://www.home-assistant.io/integrations/private_ble_device/) | Tracking de dispositivos com MAC aleatório via IRK |
| [DerekSeaman/irk-capture](https://github.com/DerekSeaman/irk-capture) | Pacote ESPHome para captura de IRKs Apple/Android |
| [Bermuda](https://github.com/agittins/bermuda) | Integração HACS para triangulação/trilateração BLE usando proxies ESPHome |
