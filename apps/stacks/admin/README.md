# Admin

Stack de ferramentas administrativas do home server. Reúne serviços de suporte à operação da infraestrutura: backup do gerenciador de containers e túnel seguro para expor serviços internos à internet.

## Aplicações

### Portainer Backup

Utilitário para automatizar backups do [Portainer](https://www.portainer.io) via API. Conecta-se ao servidor Portainer com um access token e exporta periodicamente o banco de dados da instância em um arquivo `.tar.gz`. Opcionalmente, inclui os arquivos `docker-compose` das stacks criadas na interface web do Portainer.

Nesta stack, o container roda no modo `schedule`, executando backups de forma contínua conforme a expressão cron definida em `PORTAINER_BACKUP_SCHEDULE`. Os arquivos gerados são gravados no volume montado em `/backup` (mapeado para `PORTAINER_BACKUP_HOST_PATH` no host).

**Referência:** [SavageSoftware/portainer-backup](https://github.com/SavageSoftware/portainer-backup)

### Cloudflare Tunnel

O [Cloudflare Tunnel](https://developers.cloudflare.com/tunnel/) conecta serviços locais à rede global da Cloudflare por meio de conexões criptografadas apenas de saída, sem expor IP público nem abrir portas de entrada no firewall. O daemon `cloudflared` mantém um túnel persistente com a Cloudflare, permitindo mapear hostnames públicos para serviços internos (por exemplo, `app.exemplo.com` → `http://localhost:8080`). O tráfego passa pela CDN da Cloudflare, com proteção contra DDoS, WAF e demais recursos de segurança aplicados na borda.

Nesta stack, o `cloudflared` é iniciado com um token de túnel (`CLOUDFLARED_TUNNEL_TOKEN`), que carrega a configuração definida no dashboard da Cloudflare (rotas, hostnames e serviços de destino).

**Referência:** [Cloudflare Tunnel — Documentação oficial](https://developers.cloudflare.com/tunnel/)

## Configuração

Copie `env.example` para `.env` e preencha as variáveis antes de subir a stack:


| Variável | Descrição |
| --- | --- |
| `PORTAINER_BACKUP_URL` | URL base do Portainer |
| `PORTAINER_BACKUP_TOKEN` | Access token de um usuário administrador |
| `PORTAINER_BACKUP_SCHEDULE` | Expressão cron do agendamento (6 campos) |
| `PORTAINER_BACKUP_HOST_PATH` | Diretório no host onde os backups são salvos |
| `CLOUDFLARED_TUNNEL_TOKEN` | Token do túnel criado no dashboard Cloudflare |

