# Hardware — Raspberry Pi

Apresentando a infraestrutura física baseada em Raspberry Pi. A documentação de software (stacks, Portainer, Home Assistant) está no [README principal](../../README.md). O nó x86 Dell está em [hardware/dell](../dell/README.md).

## Visão geral

| Dispositivo | RAM | Função | SO / boot |
| ----------- | --- | ------ | --------- |
| **Raspberry Pi 5** (`raspberrypi`) | 8 GB | Portainer CE, admin, games, immich, telemetria (só M.2) | NVMe M.2 256 GB (boot + containers) |
| **Raspberry Pi 4** | 8 GB | Automação residencial (Home Assistant) — sem inventário Docker aqui | SD card 32 GB (boot) + SSD 224 GB (USB 3.0) |

> O HDD de 1 TB não fica mais no Pi 5 — ver [Dell OptiPlex 3050 Micro](../dell/README.md).

---

## Raspberry Pi 5
Para detalhes sobre a infraestrutura do Raspberry Pi 5, consulte o [README V1](./pi5/v1/README.md) e a configuração atual no [README V2](./pi5/v2/README.md). O Raspberry Pi 5 é o mesmo; só os periféricos mudaram. Na **v2 atual**, o nó usa **apenas o NVMe M.2 de 256 GB** (SO + containers).

## Raspberry Pi 4
Para detalhes sobre a infraestrutura do Raspberry Pi 4, consulte o [README do pi4](./pi4/README.md).


## Referências

### Placas

- [Raspberry Pi 5 — produto oficial](https://www.raspberrypi.com/products/raspberry-pi-5/)
- [Introducing: Raspberry Pi 5](https://www.raspberrypi.com/news/introducing-raspberry-pi-5/)
- [Raspberry Pi 4 Model B — produto oficial](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- [Dell OptiPlex 3050 Micro](../dell/README.md)

