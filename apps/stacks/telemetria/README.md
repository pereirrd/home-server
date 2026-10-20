# Telemetria

Stack de observabilidade do home server: métricas, dashboards e mensageria MQTT. Roda no **Raspberry Pi 5** (`raspberrypi`).

> Documentação inicial do inventário em produção. Exemplos de `docker-compose` / `env` podem ser adicionados depois — configure cada componente pela documentação oficial.

## Onde roda

| Host | Hostname | Stack Compose |
| ---- | -------- | ------------- |
| Raspberry Pi 5 | `raspberrypi` | `telemetria` |

Inventário completo do nó: [hardware/raspberry_pi/pi5/v2](../../../hardware/raspberry_pi/pi5/v2/README.md).

## Containers

| Container | Imagem (produção) | Função |
| --------- | ----------------- | ------ |
| `telemetria-grafana-1` | `grafana/grafana:11.6.16` | Dashboards e visualização |
| `telemetria-influxdb-1` | `influxdb:2.7` | Banco de séries temporais |
| `telemetria-mosquitto-1` | `eclipse-mosquitto:2` | Broker MQTT |
| `telemetria-telegraf-1` | `telegraf:1.32` | Coleta de métricas / agentes |

## Referências

- [Grafana](https://grafana.com/docs/)
- [InfluxDB](https://docs.influxdata.com/influxdb/)
- [Eclipse Mosquitto](https://mosquitto.org/documentation/)
- [Telegraf](https://docs.influxdata.com/telegraf/)
- [Raspberry Pi 5 — v2](../../../hardware/raspberry_pi/pi5/v2/README.md)
