# Changelog

Mudanças relevantes no laboratório são documentadas aqui. O formato segue [Keep a Changelog](https://keepachangelog.com/), adaptado para este projeto de documentação.

## [Unreleased]

### Planejado

- Integração com Cloudflare Tunnel no `gateway`
- Playbooks Ansible para hardening básico (SSH, firewall, fail2ban)
- Stack de observabilidade centralizada (Prometheus + Grafana + Loki + Promtail)
- Módulos Terraform para recursos da OCI
- Docker Compose stacks para serviços iniciais

## [0.1.0] - 2026-05-06

### Adicionado

- Estrutura inicial do repositório (`docs/`, `adr/`, `ansible/`, `terraform/`, `compose/`)
- Documentação de arquitetura (`docs/architecture.md`)
- Documentação de rede (`docs/network.md`)
- Runbook de operações (`docs/runbook.md`)
- Diagrama de topologia em Mermaid (`docs/diagrams/overview.mmd`)
- Architecture Decision Records 0001–0005:
  - ADR-0001: Usar Oracle Cloud Free Tier como plataforma
  - ADR-0002: Gateway node separado para isolar ingress
  - ADR-0003: ARM Ampere como arquitetura primária
  - ADR-0004: Cloudflare Tunnel para zero exposed ports
  - ADR-0005: WireGuard para a private mesh
- WireGuard mesh entre gateway e 3 workload nodes (`10.13.13.0/24`)
