# Stacks Docker Compose

> 🚧 **Em construção.** Este diretório vai conter um arquivo Docker Compose por serviço ou grupo de serviços, organizado pelo papel do node.

## Layout planejado

```
compose/
├── monitoring/          ← Prometheus, Grafana, Loki, Promtail
├── selfhosted/          ← Serviços pessoais (Ollama, etc.)
├── databases/           ← Postgres, Redis, etc.
└── ingress/             ← Reverse proxy, load balancer
```

## Convenções

- `.env.example` por stack documentando todas as variáveis (nunca commit `.env` reais)
- Arquivos `.env` reais no `.gitignore`
- Volumes nomeados (não bind mounts) por padrão, exceto onde justificado
- `healthcheck` definido para todo serviço
- Versões de imagem pinadas (`postgres:16.4`, não `postgres:latest`)
- Para imagens multi-arch, validar que existe build ARM64 antes de incluir (nem todo software popular oferece ARM64)
