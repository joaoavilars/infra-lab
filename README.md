# infra-lab

Laboratório pessoal de infraestrutura na nuvem rodando no Oracle Cloud Free Tier. O projeto documenta e operacionaliza um ambiente VPS com 4 nodes, utilizado para self-hosted services, experimentação contínua e aprendizado prático em DevOps, redes e platform engineering.

> **Status:** Documento vivo. O laboratório evolui conforme estudo novos tópicos — security hardening, CI/CD, observabilidade e IaC estão sendo adicionados progressivamente.

## Visão geral

O modelo arquitetural segue três princípios fundamentais: **zero exposed ports** (nenhuma VM tem HTTP/HTTPS público; todo ingress flui pelo Cloudflare Tunnel), **private mesh by default** (comunicação entre nodes via WireGuard em criptografia de ponta a ponta) e **custo zero** (totalmente dentro do Always Free da OCI).

```mermaid
flowchart TB
    Internet([Usuários da Internet])
    CF[Cloudflare Edge]
    Internet -->|HTTPS| CF

    subgraph OCI["Oracle Cloud"]
        subgraph Gateway["gateway (1 vCPU / 1GB RAM - x86)"]
            CFT["cloudflared tunnel"]
            WG0["WireGuard hub<br/>10.13.13.1"]
            CFT --> WG0
        end

        subgraph Scarabeus["scarabeus (4 vCPU / 24GB ARM)"]
            WG1["WireGuard 10.13.13.5"]
            APP1["Web apps / APIs<br/>K8s / Swarm"]
            WG1 --- APP1
        end

        subgraph Libellula["libellula (4 vCPU / 24GB ARM)"]
            WG2["WireGuard 10.13.13.6"]
            DOCKER["Containers Docker"]
            WG2 --- DOCKER
        end

        subgraph Lucanus["lucanus (4 vCPU / 24GB ARM)"]
            WG3["WireGuard 10.13.13.7"]
            SVC["Self-hosted<br/>Ollama"]
            WG3 --- SVC
        end
    end

    CF -.->|Tunnel| CFT
    WG0 <-->|encrypted mesh| WG1
    WG0 <-->|encrypted mesh| WG2
    WG0 <-->|encrypted mesh| WG3
    WG1 <-->|encrypted mesh| WG2
    WG2 <-->|encrypted mesh| WG3
    WG1 <-->|encrypted mesh| WG3

    classDef gateway fill:#fef3c7,stroke:#d97706,stroke-width:2px
    classDef workload fill:#dbeafe,stroke:#2563eb,stroke-width:2px

    class Gateway gateway
    class Scarabeus workload
    class Libellula workload
    class Lucanus workload
```

## Inventário

| Host | Papel | Specs | Arquitetura |
|------|-------|-------|-------------|
| `gateway` | Hub WireGuard + Cloudflare Tunnel | 1 vCPU / 1 GB | x86_64 (E2.1.Micro) |
| `scarabeus` | Web apps, APIs, K8s/Swarm | 4 vCPU / 24 GB | ARM64 (A1.Flex) |
| `libellula` | Containers Docker | 4 vCPU / 24 GB | ARM64 (A1.Flex) |
| `lucanus` | Self-hosted services (Ollama) | 4 vCPU / 24 GB | ARM64 (A1.Flex) |

> As atribuições de papel são ilustrativas. O laboratório é intencionalmente flexível para experimentos com k3s, Swarm, e outras topologias de orquestração.

## Estrutura do repositório

```
infra-lab/
├── README.md                    ← Este arquivo
├── CHANGELOG.md
├── AGENTS.md
├── .gitignore
├── docs/
│   ├── architecture.md
│   ├── network.md
│   ├── runbook.md
│   └── diagrams/
│       └── overview.mmd
├── adr/
│   ├── README.md
│   ├── template.md
│   ├── 0001-oracle-cloud-free-tier.md
│   ├── 0002-gateway-node-separado.md
│   ├── 0003-arm-ampere-como-arquitetura-primaria.md
│   ├── 0004-cloudflare-tunnel.md
│   └── 0005-wireguard-mesh.md
├── ansible/
│   └── README.md
├── terraform/
│   └── README.md
└── compose/
    └── README.md
```

## Roadmap

- [x] Documentação de arquitetura e diagramas
- [x] Setup do WireGuard mesh
- [ ] Integração com Cloudflare Tunnel
- [ ] Playbooks Ansible para hardening básico
- [ ] Módulos Terraform para recursos do Oracle Cloud
- [ ] Stack de observabilidade centralizada (Prometheus + Grafana + Loki)
- [ ] Pipeline de CI/CD para deployments declarativos
- [ ] Estratégia de backup automatizado
- [ ] Runbook de disaster recovery

## Decision Records

Escolhas arquiteturais relevantes são documentadas como Architecture Decision Records no diretório [`./adr/`](./adr/). Consulte [`./adr/README.md`](./adr/README.md) para o índice e formato.

## Licença

Documentação sob licença MIT. Arquivo de licença será adicionado no futuro.
