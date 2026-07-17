# Hardware — ESP32

Placas ESP32 usadas na automatização residencial, integradas ao [Home Assistant](../../apps/home_assistant/README.md) via [ESPHome](../../automations/esp32/README.md). A documentação de firmware e configuração fica em [`automations/esp32`](../../automations/esp32/README.md). A escolha de modelos diferentes foi só por curiosidade e aprendizado — qualquer um deles resolve as tarefas básicas deste setup (proxy BLE e captura de IRK).

## Inventário

| Modelo | Qtd. | Função neste setup |
| ------ | ---- | ------------------ |
| **ESP32 DevKit V1** | 3 | Bluetooth tracker (proxy BLE) |
| **ESP32-DevKitC V4** | 1 | Bluetooth tracker (proxy BLE) |
| **Mini ESP32-C3** | 1 | Captura de IRK (presence detection) |

Os quatro DevKits clássicos (V1 + DevKitC V4) atuam como **gatilhos de rotinas**, rastreando os dispositivos mobile da casa. O Mini ESP32-C3 captura IRKs para que novos telefones/relógios com MAC randomizado possam ser monitorados pelo Home Assistant.

---

## ESP32 DevKit V1

Placa de desenvolvimento popular (geralmente baseada no módulo **ESP32-WROOM-32**). Não é um kit oficial Espressif, mas usa o SoC **ESP32** da Espressif — a referência de hardware é o datasheet da série ESP32.

### Especificações básicas

| Item | Detalhe |
| ---- | ------- |
| **SoC** | ESP32 (Xtensa dual-core LX6, até 240 MHz) |
| **Módulo típico** | ESP32-WROOM-32 |
| **Wi-Fi** | 802.11 b/g/n (2,4 GHz) |
| **Bluetooth** | Classic + BLE 4.2 |
| **SRAM** | 520 KB |
| **Flash** | tipicamente 4 MB (módulo) |
| **Alimentação** | Micro-USB / 3,3 V |

Neste setup: **3 unidades** como Bluetooth trackers distribuídos pela casa.

**Referências oficiais (chip / módulo):**

- [ESP32 — overview (Espressif)](https://www.espressif.com/en/products/socs/esp32)
- [ESP32 Series Datasheet (PDF)](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)
- [ESP32-WROOM-32 — datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32_datasheet_en.pdf)

---

## ESP32-DevKitC V4

Kit oficial Espressif baseado em módulos da família ESP32 (ex.: ESP32-WROOM-32E). Formato compacto com a maioria dos GPIOs expostos em headers.

### Especificações básicas

| Item | Detalhe |
| ---- | ------- |
| **SoC** | ESP32 (Xtensa dual-core LX6, até 240 MHz) |
| **Módulos suportados** | ESP32-WROOM-32E / 32UE, WROVER-E, entre outros |
| **Wi-Fi** | 802.11 b/g/n (2,4 GHz) |
| **Bluetooth** | Classic + BLE 4.2 |
| **Flash** | tipicamente 4 MB (depende do módulo) |
| **Interface** | Micro-USB (alimentação e programação) |

Neste setup: **1 unidade** como Bluetooth tracker.

**Referências oficiais:**

- [ESP32-DevKitC — overview](https://www.espressif.com/en/products/devkits/esp32-devkitc/overview)
- [ESP32-DevKitC V4 — user guide](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32/esp32-devkitc/user_guide.html)
- [ESP32 Series Datasheet (PDF)](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)

---

## Mini ESP32-C3

Placa compacta da família **ESP32-C3** (módulo **ESP32-C3-MINI-1** / placa de referência **ESP32-C3-DevKitM-1**). Single-core RISC-V, com Wi-Fi e **apenas BLE** (sem Bluetooth Classic).

### Especificações básicas

| Item | Detalhe |
| ---- | ------- |
| **SoC** | ESP32-C3 (RISC-V single-core, até 160 MHz) |
| **Módulo** | ESP32-C3-MINI-1 (ou MINI-1U) |
| **Wi-Fi** | 802.11 b/g/n (2,4 GHz) |
| **Bluetooth** | BLE 5.0 |
| **Flash** | 4 MB (embutida no chip/módulo) |
| **Formato** | placa mini / DevKitM-1 |

Neste setup: **1 unidade** dedicada à captura de IRK — permite adicionar dispositivos mobile (Apple/Android) ao monitoramento de presença, mesmo com MAC BLE randomizado.

**Referências oficiais:**

- [ESP32-C3 — overview](https://www.espressif.com/en/products/socs/esp32-c3)
- [ESP32-C3-DevKitM-1 — user guide](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32c3/esp32-c3-devkitm-1/user_guide.html)
- [ESP32-C3 Series Datasheet (PDF)](https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_en.pdf)

---

## Bluetooth Tracker (uso no home server)

Os ESP32 DevKit V1 e o DevKitC V4 rodam o projeto **Bluetooth Tracker**: cada placa vira um **proxy BLE** via ESPHome. Eles escutam anúncios Bluetooth próximos e encaminham os dados ao Home Assistant pela Wi-Fi, estendendo o alcance de rastreamento pela casa.

Esses proxies alimentam rotinas de presença (ex.: com [Bermuda](../../apps/home_assistant/README.md#bermuda)): quando o telefone entra ou sai do alcance de um scanner, o HA dispara automações.

```
[ Mobile / BLE ]  ~~BLE~~>  [ ESP32 tracker ]  ~~Wi-Fi~~>  [ Home Assistant ]
```

Documentação completa (YAML, flash, integração): **[ESP32 — Automations (ESPHome)](../../automations/esp32/README.md#bluetooth-tracker)**.

### IRK Capture (Mini ESP32-C3)

O Mini ESP32-C3 roda o projeto **IRK Capture** para obter a *Identity Resolving Key* de telefones e relógios. Com o IRK no [Private BLE Device](https://www.home-assistant.io/integrations/private_ble_device/), o Home Assistant consegue rastrear dispositivos que mudam o endereço MAC periodicamente.

Documentação completa: **[ESP32 — IRK Capture](../../automations/esp32/README.md#irk-capture)**.

---

## Referências

### Placas e chips (Espressif)

- [ESP32 — SoC](https://www.espressif.com/en/products/socs/esp32)
- [ESP32 Series Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)
- [ESP32-DevKitC V4 — user guide](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32/esp32-devkitc/user_guide.html)
- [ESP32-C3 — SoC](https://www.espressif.com/en/products/socs/esp32-c3)
- [ESP32-C3-DevKitM-1 — user guide](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32c3/esp32-c3-devkitm-1/user_guide.html)

### Neste repositório

- [ESP32 — Automations (ESPHome)](../../automations/esp32/README.md)
- [Home Assistant](../../apps/home_assistant/README.md)
