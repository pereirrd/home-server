# Telemetria

Stack de observabilidade do home server: métricas (Glances → MQTT → Telegraf → InfluxDB) e dashboards no Grafana. Roda no **Raspberry Pi 5** (`raspberrypi`).

> Exemplo ilustrativo alinhado à produção. Configs de Mosquitto, Telegraf e Grafana ficam em `STACK_DIR` no host — não versionadas aqui. Consulte a documentação oficial de cada componente.

## Onde roda

| Host | Hostname | Stack Compose |
| ---- | -------- | ------------- |
| Raspberry Pi 5 | `raspberrypi` | `telemetria` |

Inventário: [hardware/raspberry_pi/pi5/v2](../../../../hardware/raspberry_pi/pi5/v2/README.md).

## Arquivos de exemplo

```
apps/stacks/raspberrypi/telemetria/
├── README.md
├── docker-compose.yaml
└── env.example
```

No host, tipicamente:

```
$STACK_DIR/
├── mosquitto/config/   # mosquitto.conf, acl, passwd
├── telegraf/           # telegraf.conf, glances.star
└── grafana/
    ├── provisioning/
    └── dashboards/
```

## Arquitetura

```
cada servidor (Glances no SO, como serviço)
  CPU, memória, disco, temperaturas
              ↓ MQTT
          Mosquitto
              ↓
          Telegraf  →  InfluxDB 2.7
                           ↓
                       Grafana (dashboard)
```

Todos os serviços entram na `proxy_network` do Pi 5, com aliases estáveis (`telemetria-mosquitto`, `telemetria-influxdb`, `telemetria-grafana`) para o NPM e entre containers. Nomes de container gerados pelo Compose costumam ser `telemetria-*-1`.

**MQTT na LAN:** a porta `1883` é publicada pelo **NPM** da stack [admin do Pi 5](../admin/) (`1883:1883`), não por este compose.

**Grafana:** Proxy Host no NPM de entrada apontando para `telemetria-grafana:3000` (ou o nome do container), com `GF_SERVER_ROOT_URL` igual ao hostname público/local.

## Coleta com Glances

Os dados de telemetria são gerados pelo [Glances](https://github.com/nicolargo/glances). Ele está instalado em **cada servidor** da infraestrutura, diretamente no sistema operacional, e roda como serviço — fora desta stack Docker. Cada instância publica no Mosquitto as métricas do hardware em que está:

- consumo de CPU
- uso de memória
- uso de disco
- temperaturas registradas pelos sensores daquele hardware

O Telegraf consome esses tópicos e grava as séries no bucket do InfluxDB. No Grafana há um dashboard com gráficos para exibir essas métricas.

## Containers

| Alias / serviço | Imagem | Função |
| --------------- | ------ | ------ |
| `mosquitto` → `telemetria-mosquitto` | `eclipse-mosquitto:2` | Broker MQTT |
| `influxdb` → `telemetria-influxdb` | `influxdb:2.7` | Séries temporais (retention init `30d`) |
| `telegraf` | `telegraf:1.32` | Consome MQTT e grava no Influx |
| `grafana` → `telemetria-grafana` | `grafana/grafana:11.6.16` | Dashboards |
| `portainer-collector` | imagem local (`STACK_DIR/portainer`) | Lê a API do [Komodo](../komodo/README.md) e publica o inventário Docker |

## Coletor do Komodo (`portainer-collector`)

O nome ficou da época do Portainer — ver [DESCONTINUADOS.md](../../../../DESCONTINUADOS.md). O container **não** fala mais com o Portainer. A cada intervalo ele lê a API do Komodo (`KOMODO_URL`, em geral `http://komodo-core:9120` na `proxy_network`), garante o bucket Docker no InfluxDB e publica em `portainer/<servidor>/<medição>`.

O prefixo do tópico continua `portainer/` e o usuário MQTT continua `portainer`. O Telegraf e os dashboards já filtram esses nomes; trocar o prefixo obrigaria a reescrever os dois. A tag de endpoint é a do ambiente que o dashboard já usava.

Segredos (`KOMODO_API_KEY`, `KOMODO_API_SECRET`, `MQTT_PORTAINER_PASSWORD`) ficam só no `.env`.

## Configuração

Copie `env.example` para `.env` e preencha:

| Variável | Descrição |
| --- | --- |
| `STACK_DIR` | Path no host com configs montadas nos containers |
| `MQTT_TELEGRAF_PASSWORD` | Senha do usuário Telegraf no Mosquitto |
| `MQTT_GLANCES_PASSWORD` | Senha usada pelos publishers Glances (arquivo `passwd` / ACL) |
| `INFLUX_USERNAME` / `INFLUX_PASSWORD` | Admin do setup inicial do InfluxDB |
| `INFLUX_TOKEN` | Token admin (init + Telegraf + Grafana) |
| `INFLUX_ORG` / `INFLUX_BUCKET` | Org e bucket (ex.: `home` / `glances`) |
| `GRAFANA_ADMIN_*` | Usuário/senha do admin do Grafana |
| `GRAFANA_ROOT_URL` | URL canônica atrás do NPM |
| `INFLUX_BUCKET_DOCKER` | Bucket do inventário Docker publicado pelo coletor |
| `KOMODO_URL` | API do Core na rede Docker (`http://komodo-core:9120`) |
| `KOMODO_API_KEY` / `KOMODO_API_SECRET` | Chave de API do Komodo usada pelo coletor |
| `MQTT_PORTAINER_PASSWORD` | Senha do usuário MQTT `portainer` (escrita em `portainer/#`) |

Antes do deploy: `docker network create proxy_network` (se ainda não existir). Os arquivos em `STACK_DIR` precisam existir antes do `compose up`.

## Referências

- [Glances](https://nicolargo.github.io/glances/)
- [Grafana](https://grafana.com/docs/)
- [InfluxDB](https://docs.influxdata.com/influxdb/)
- [Eclipse Mosquitto](https://mosquitto.org/documentation/)
- [Telegraf](https://docs.influxdata.com/telegraf/)
- [Admin Pi 5 — porta MQTT](../admin/)
- [Raspberry Pi 5 — v2](../../../../hardware/raspberry_pi/pi5/v2/README.md)
