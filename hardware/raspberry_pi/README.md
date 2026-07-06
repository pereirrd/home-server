# Hardware — Raspberry Pi

Apresentando a infraestrutura física do home server. A documentação de software (stacks, Portainer, Home Assistant) está no [README principal](../../README.md).

## Visão geral

| Dispositivo | RAM | Função | SO / boot |
| ----------- | --- | ------ | --------- |
| **Raspberry Pi 5** | 8 GB | Containers Docker, Portainer e stacks de mídia/jogos/admin | NVMe M.2 256 GB (boot) |
| **Raspberry Pi 4** | 8 GB | Automação residencial (Home Assistant) | SD card 32 GB (boot) + SSD 224 GB (USB 3.0) |

---

## Raspberry Pi 5

Placa principal da infraestrutura. Roda [Debian Trixie 13](https://www.raspberrypi.com/news/trixie-the-new-version-of-raspberry-pi-os/) com administração via [Webmin](https://webmin.com/), hospedando o [Portainer](../../apps/portainer/README.md) e as [stacks Docker](../../apps/stacks/admin/README.md).

### Especificações da placa

O [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/) traz: CPU Cortex-A76 quad-core a 2,4 GHz, GPU VideoCore VII, 2× USB 3.0 a 5 Gbps, Ethernet Gigabit, Wi-Fi 802.11ac e, principalmente para este setup, **interface PCIe 2.0 x1** para periféricos de alta velocidade. Mais detalhes no anúncio oficial: [Introducing: Raspberry Pi 5](https://www.raspberrypi.com/news/introducing-raspberry-pi-5/).

### Expansão PCIe — topologia

O único slot PCIe do Pi 5 é dividido em **dois canais** por um HAT hub, permitindo conectar simultaneamente o disco de boot (NVMe) e a placa de armazenamento em massa (SATA).

```
Raspberry Pi 5 (PCIe 2.0 x1)
    │
    └── Waveshare PCIe TO 2-CH PCIe HAT
            ├── Canal 1 → Geekworm X1001 (NVMe)     → M.2 NVMe 256 GB  [sistema operacional]
            └── Canal 2 → Geekworm X1006 (SATA)     → HDD 2.5" 1 TB
                                                    → M.2 SATA 256 GB
```

#### Waveshare PCIe TO 2-CH PCIe HAT

Adaptador que divide o conector PCIe do Pi 5 em **dois canais PCIe FFC**, operando em **PCIe Gen2 x1** (5 GT/s). Permite empilhar módulos de expansão sem perder o acesso ao GPIO.

- **Wiki:** [PCIe TO 2-CH PCIe HAT](https://www.waveshare.com/wiki/PCIe_TO_2-CH_PCIe_HAT)
- **Configuração:** habilitar PCIe em `/boot/firmware/config.txt` com `dtparam=pciex1` (habilitado por padrão no Pi 5)

#### Geekworm X1001 — NVMe (boot)

Shield PIP (PCIe Peripheral Board) montado na **parte superior** do Pi 5. Converte o canal PCIe em slot **M.2 NVMe** (Key-M), usado como disco de sistema.

| Item | Detalhe |
| ---- | ------- |
| **Disco** | M.2 NVMe 256 GB |
| **Função** | Sistema operacional (boot) |
| **Formatos suportados** | 2230 / 2242 / 2260 / 2280 |
| **Velocidade** | PCIe Gen2 (5 GT/s); Gen3 opcional via `dtparam=pciex1_gen=3` |
| **Boot NVMe** | Suportado |

- **Wiki:** [X1001](https://wiki.geekworm.com/X1001)

> O X1001 **não** aceita SSDs M.2 SATA — apenas NVMe. A alimentação via cabo FFC suporta até 5 W contínuos; SSDs de maior consumo podem precisar de alimentação auxiliar pelo conector XH2.54.

#### Geekworm X1006 — SATA (armazenamento)

Shield montado na **parte inferior** do Pi 5 (via pogo pins). Expande o segundo canal PCIe em **duas interfaces SATA 3.0** simultâneas.

| Item | Detalhe |
| ---- | ------- |
| **HDD 2.5"** | 1 TB — armazenamento em massa |
| **M.2 SATA** | 256 GB — SSD SATA Key-B 2280 |
| **Velocidade SATA** | Até 5 Gbps (SATA 3.0) |
| **Alimentação** | 5 V ≥ 3 A via FFC e pogo pins |
| **Boot SATA** | **Não suportado** com firmware atual |

- **Wiki:** [X1006](https://wiki.geekworm.com/X1006)

> O X1006 **não** é compatível com SSDs M.2 NVMe ou PCIe AHCI — apenas **M.2 SATA Key-B** e **HDD/SSD 2.5" SATA**. Discos novos precisam ser particionados e formatados antes do uso.

### Armazenamento — resumo (Pi 5)

| Disco | Interface | Capacidade | Função |
| ----- | --------- | ---------- | ------ |
| M.2 NVMe | PCIe (X1001) | 256 GB | Sistema operacional |
| M.2 SATA | SATA (X1006) | 256 GB | Armazenamento |
| HDD 2.5" | SATA (X1006) | 1 TB | Armazenamento em massa |

### Alimentação

Com três discos ativos (NVMe + SATA + HDD), a fonte recomendada é **5 V ≥ 3 A** via USB-C — alinhada às especificações do X1006 e do Pi 5. Fontes GaN de 27 W (ex.: Geekworm XS-GaN-27W) são referência comum para esse tipo de montagem.

---

## Raspberry Pi 4

Nó dedicado à [automação residencial](../../apps/home_assistant/README.md). O Home Assistant roda **diretamente no dispositivo** no modo **Supervisor** (instalação gerenciada com suporte nativo a add-ons).

### Especificações da placa

O [Raspberry Pi 4 Model B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/) oferece CPU Cortex-A72 quad-core a 1,5 GHz (até 1,8 GHz em revisões recentes), até 8 GB de RAM LPDDR4, 2× USB 3.0, 2× USB 2.0, Ethernet Gigabit e Wi-Fi 802.11ac — plataforma madura e amplamente suportada pelo ecossistema Home Assistant.

### Armazenamento (Pi 4)

| Disco | Interface | Capacidade | Função |
| ----- | --------- | ---------- | ------ |
| SD card | microSD (slot interno) | 32 GB | Boot / sistema Home Assistant OS |
| SSD | USB 3.0 | 224 GB | Armazenamento adicional |

O boot parte do **cartão SD de 32 GB**; o **SSD de 224 GB** na porta USB 3.0 complementa a capacidade de armazenamento (dados, snapshots, mídia do HA, etc.).

---

## Referências

### Placas

- [Raspberry Pi 5 — produto oficial](https://www.raspberrypi.com/products/raspberry-pi-5/)
- [Introducing: Raspberry Pi 5](https://www.raspberrypi.com/news/introducing-raspberry-pi-5/)
- [Raspberry Pi 4 Model B — produto oficial](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)

### Expansões PCIe (Pi 5)

- [Waveshare PCIe TO 2-CH PCIe HAT](https://www.waveshare.com/wiki/PCIe_TO_2-CH_PCIe_HAT)
- [Geekworm X1001 — NVMe SSD Shield](https://wiki.geekworm.com/X1001)
- [Geekworm X1006 — PCIe to SATA](https://wiki.geekworm.com/X1006)
