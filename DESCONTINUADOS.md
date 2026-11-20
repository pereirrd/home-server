# Aplicações descontinuadas

Registro do que já fez parte deste home server e **saiu de uso**. O restante do repositório descreve só o que está em execução.

Cada entrada diz o que a aplicação fazia, por que foi descartada e o que ficou no lugar.

## Portainer

O [Portainer](https://www.portainer.io/) Community Edition era o gerenciador de containers. Havia **uma** interface no Raspberry Pi 5 e um **Agent** no Dell OptiPlex 3050 Micro: a UI do Pi enxergava os dois Docker Engines e era por ela que as stacks eram criadas e atualizadas. O backup da instância rodava na stack admin (`portainer-backup`) e havia um MCP (`portainer-mcp`) em cima da API do Portainer.

**Por que saiu.** O Portainer guarda o `docker-compose.yaml` de cada stack em um **banco interno**. Mudar uma stack significava editar esse banco pela interface, não um arquivo no disco. Isso dificulta a manutenção e, principalmente, mudanças feitas por **agentes de IA**, que trabalham em arquivos versionados. Foram avaliadas outras aplicações de gestão de containers. O [Komodo](apps/stacks/raspberrypi/komodo/README.md) foi o escolhido pela **interface amigável** e porque, no modo *Files on Server*, o compose fica **no disco** do host: a IA edita o arquivo e o deploy lê esse arquivo.

**No lugar.** [Komodo](apps/stacks/raspberrypi/komodo/README.md) — Core no Pi 5, Periphery em cada host Docker. O coletor da [telemetria](apps/stacks/raspberrypi/telemetria/README.md) é o `container-collector`: lê a API do Komodo e publica no prefixo MQTT `container/`. As medições no InfluxDB e a tag `endpoint` dos dashboards não mudaram com o nome.

## Webmin

O [Webmin](https://webmin.com/) administrava o sistema operacional do Raspberry Pi 5 pela web, na montagem antiga da placa (HAT PCIe com HDD 2.5" — ver [pi5/v1](hardware/raspberry_pi/pi5/v1/README.md)).

**Por que saiu.** Um bug aumentava o **iowait** do HDD 2.5" por um motivo que não ficou claro. Em paralelo, a configuração dos hosts passou a ser feita por **IA em arquivos no disco**. O painel web não ajudava nesse fluxo: a alteração útil é a que a IA grava no repositório de configuração do host, não um clique no Webmin.

**No lugar.** Nada equivalente. Sistema, units e stacks são editados nos arquivos de cada máquina e aplicados pelo serviço correspondente (systemd, Compose, Komodo).

## AdGuard Home

O [AdGuard Home](https://adguard.com/adguard-home/overview.html) foi o servidor DNS de bloqueio de anúncios e rastreadores, instalado como app do Home Assistant (repositório [hassio-addons](https://github.com/hassio-addons/app-adguard-home)).

**Por que saiu.** As listas bloqueavam **recursos úteis** junto com anúncio. Na rede com AdGuard, o **Duolingo** travava com frequência e o app da **Globoplay** também falhava. Ajustar exceções era difícil: a documentação não ajudava a afinar o filtro sem quebrar outro serviço, e a interface de gerenciamento era ruim para esse trabalho do dia a dia.

**No lugar.** Nenhum DNS de bloqueio. O [Home Assistant](apps/home_assistant/README.md) segue com HACS, Uptime Kuma, Bermuda e Homelable.

## Referências

- [Komodo — o que está no lugar do Portainer](apps/stacks/raspberrypi/komodo/README.md)
- [Home Assistant](apps/home_assistant/README.md)
- [Raspberry Pi 5 — montagem v1 (época do Webmin)](hardware/raspberry_pi/pi5/v1/README.md)
- [Portainer](https://www.portainer.io/) *(histórico)*
- [Webmin](https://webmin.com/) *(histórico)*
- [AdGuard Home](https://adguard.com/adguard-home/overview.html) *(histórico)*
- [AdGuard Home — Home Assistant Community App](https://github.com/hassio-addons/app-adguard-home) *(histórico)*
