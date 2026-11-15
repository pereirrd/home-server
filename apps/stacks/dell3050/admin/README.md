# Admin — Dell OptiPlex 3050 Micro

Stack administrativa do host **`dell3050`**. Reúne o proxy reverso deste nó, o túnel Cloudflare e o [Filebrowser](https://filebrowser.org/). O deploy é feito pelo [Komodo](../../../komodo/README.md).

A mesma stack existe no Raspberry Pi 5, que publica a porta MQTT `1883` a mais. O desenho dos dois NPMs, o repasse Pi 5 → Dell e os túneis estão em [admin do Pi 5](../../raspberrypi/admin/README.md).

| Host | Hostname | Apps | Compose |
| ---- | -------- | ---- | ------- |
| Dell OptiPlex 3050 Micro | `dell3050` | NPM, Cloudflare Tunnel, Filebrowser | este diretório |
| Raspberry Pi 5 | `raspberrypi` | NPM (entrada + MQTT `1883`), Tunnel, Filebrowser | [`../../raspberrypi/admin/`](../../raspberrypi/admin/) |

## Arquivos de exemplo

```
apps/stacks/dell3050/admin/
├── README.md
├── docker-compose.yaml
└── env.example
```

## Estratégia de comunicação entre os NPMs

O NPM deste host é o **nó remoto**: recebe o repasse do NPM de entrada do Pi 5 e entrega a requisição aos containers da `proxy_network` local (games, immich, stream, seafile). O túnel Cloudflare deste nó publica na internet o que sai daqui. O acesso local (`*.local`) continua centralizado no Pi 5.

Detalhe do fluxo, papéis e diagrama: [estratégia entre os NPMs](../../raspberrypi/admin/README.md#estratégia-de-comunicação-entre-os-npms).

Antes do primeiro deploy:

```bash
docker network create proxy_network
```

## Configuração

Sem porta MQTT no NPM. Os servidores MCP ficam no Pi 5, na stack [mcps](../../../mcps/README.md); o deste host só é alcançado pela LAN, a partir de `npm-mcp-dell3050`. Copie `env.example` para `.env`.

| Variável | Descrição |
| --- | --- |
| `CLOUDFLARED_TUNNEL_TOKEN` | Token do túnel Cloudflare **deste** host |
| `FILEBROWSER_*` | Raiz `/srv` e paths de database/config no host |
| `NGINX_PROXY_*` | Dados e Let's Encrypt do NPM |

No painel do NPM (`http://<dell3050>:81`), os Proxy Hosts apontam para o **nome do container** na `proxy_network` local (ex.: `http://jellyfin:8096`, `http://seafile:80`).

## Referências

- [Admin do Pi 5 — NPM, túnel e estratégia entre hosts](../../raspberrypi/admin/README.md)
- [Komodo](../../../komodo/README.md)
- [Nginx Proxy Manager](https://nginxproxymanager.com/guide/)
- [Filebrowser](https://filebrowser.org/)
- [Cloudflare Tunnel](https://developers.cloudflare.com/tunnel/)
