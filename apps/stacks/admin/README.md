# Admin

Stack de ferramentas administrativas do home server. Reúne o proxy reverso de entrada, o backup do gerenciador de containers e o túnel seguro para expor serviços internos à internet.

## Arquitetura de rede

Os serviços desta stack (e das demais stacks no Raspberry Pi 5) deixam de publicar portas no host. O tráfego HTTP/HTTPS chega pelo **Nginx Proxy Manager**, que roteia pelo nome do container na rede Docker compartilhada `proxy_network`.

```
Internet / LAN
      │
      ▼
Nginx Proxy Manager  (portas 80, 443 e painel 81 no host)
      │
      │  proxy_network (externa, compartilhada)
      ├── jellyfin, seerr, sonarr, …   (stack stream)
      ├── romm                         (stack games)
      ├── portainer
      ├── cloudflared
      └── portainer-backup
```

Antes do primeiro deploy, a rede externa precisa existir:

```bash
docker network create proxy_network
```

Cada stack também mantém uma rede local (`admin_default`, `stream_network`, etc.) para comunicação interna entre containers da mesma stack, sem passar pelo proxy.

## Aplicações

### Nginx Proxy Manager

O [Nginx Proxy Manager](https://nginxproxymanager.com/) (NPM) é a **única** aplicação desta infraestrutura que publica portas no host (`80`, `443` e `81`). Ele assume o roteamento HTTP/HTTPS e o gerenciamento de certificados (Let's Encrypt), apontando cada hostname para o serviço correspondente via nome DNS interno do Docker (por exemplo, `http://jellyfin:8096`).

No painel (`http://<host>:81`), cada Proxy Host deve usar o **nome do container** como Forward Hostname e a porta **interna** do serviço (não a antiga porta mapeada no host).

| Porta no host | Uso |
| --- | --- |
| `80` | HTTP público |
| `443` | HTTPS público |
| `81` | Painel de administração do NPM |

**Referência:** [Nginx Proxy Manager — Documentação](https://nginxproxymanager.com/guide/)

### Portainer Backup

Utilitário para automatizar backups do [Portainer](https://www.portainer.io) via API. Conecta-se ao servidor Portainer com um access token e exporta periodicamente o banco de dados da instância em um arquivo `.tar.gz`. Opcionalmente, inclui os arquivos `docker-compose` das stacks criadas na interface web do Portainer.

Nesta stack, o container roda no modo `schedule`, executando backups de forma contínua conforme a expressão cron definida em `PORTAINER_BACKUP_SCHEDULE`. Os arquivos gerados são gravados no volume montado em `/backup` (mapeado para `PORTAINER_BACKUP_HOST_PATH` no host). Como está na `proxy_network`, a URL do Portainer pode usar o hostname do container (por exemplo, `http://portainer:9000`).

**Referência:** [SavageSoftware/portainer-backup](https://github.com/SavageSoftware/portainer-backup)

### Cloudflare Tunnel

O [Cloudflare Tunnel](https://developers.cloudflare.com/tunnel/) conecta serviços locais à rede global da Cloudflare por meio de conexões criptografadas apenas de saída, sem expor IP público nem abrir portas de entrada no firewall. O daemon `cloudflared` mantém um túnel persistente com a Cloudflare, permitindo mapear hostnames públicos para serviços internos.

Nesta stack, o `cloudflared` também participa da `proxy_network`. O destino típico das rotas no dashboard da Cloudflare é o próprio Nginx Proxy Manager (`http://nginx-proxy-manager:80`), de modo que o roteamento por hostname continue centralizado no NPM. Alternativamente, o túnel pode apontar direto para um container na mesma rede.

**Referência:** [Cloudflare Tunnel — Documentação oficial](https://developers.cloudflare.com/tunnel/)

## Configuração

Copie `env.example` para `.env` e preencha as variáveis antes de subir a stack:

| Variável | Descrição |
| --- | --- |
| `PORTAINER_BACKUP_URL` | URL base do Portainer (ex.: `http://portainer:9000`) |
| `PORTAINER_BACKUP_TOKEN` | Access token de um usuário administrador |
| `PORTAINER_BACKUP_SCHEDULE` | Expressão cron do agendamento (6 campos) |
| `PORTAINER_BACKUP_HOST_PATH` | Diretório no host onde os backups são salvos |
| `CLOUDFLARED_TUNNEL_TOKEN` | Token do túnel criado no dashboard Cloudflare |
| `NGINX_PROXY_DATA_PATH` | Diretório no host para dados do NPM (`/data`) |
| `NGINX_PROXY_LETSENCRYPT_PATH` | Diretório no host para certificados Let's Encrypt |
