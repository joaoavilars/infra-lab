# Ansible

> 🚧 **Em construção.** Este diretório vai conter playbooks para configuration management dos nodes do laboratório.

## Playbooks planejados

- `bootstrap.yml` — Setup inicial (usuário, SSH hardening, firewall, pacotes base)
- `wireguard.yml` — Configurar a participação na WireGuard mesh
- `docker.yml` — Instalar Docker e o plugin Compose
- `cloudflared.yml` — Instalar e configurar Cloudflare Tunnel (só no gateway)

## Estratégia de inventário

Hosts agrupados por papel (`gateway`, `workload`) e por arquitetura (`x86`, `arm64`). Segredos criptografados com `ansible-vault`, nunca commitados em texto plano. Exemplo:

```
inventory/
├── hosts            # Inventário com variáveis públicas
└── group_vars/
    ├── gateway/
    │   ├── vars.yml
    │   └── vault.yml (criptografado)
    └── workload/
        ├── vars.yml
        └── vault.yml (criptografado)
```
