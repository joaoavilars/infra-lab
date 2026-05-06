# Arquitetura do laboratório

## Princípios de design

1. **Zero exposed ports** — Nenhuma VM tem HTTP/HTTPS público. Todo ingress passa pelo Cloudflare Tunnel, eliminando attack surface em HTTP/HTTPS.
2. **Private mesh by default** — Tráfego entre nodes flui através de WireGuard criptografado. Comunicação sobre internet pública fica protegida por padrão.
3. **Custo zero** — A infraestrutura se encaixa totalmente no Oracle Cloud Always Free, sem ativação de tier pago.
4. **Decisões documentadas, não conhecimento tribal** — Toda escolha não-óbvia vira um ADR, tornando a infraestrutura compreensível e transferível.

## Topologia lógica

O laboratório separa ingress de workload. O gateway (`10.13.13.1`) é a única VM no caminho de tráfego público — funciona como edge que termina Cloudflare Tunnel e roteia para a mesh interna. Os três workload nodes (`10.13.13.5`, `.6`, `.7`) habitam a rede privada e não recebem tráfego inbound da internet.

Essa separação minimiza attack surface e liberta os workload nodes para experimentação livre com topologias instáveis, credenciais padrão em teste e software não-auditado — nada disso fica exposto ao público. O gateway, sendo especializado e reduzido, é reconstruível em minutos.

## Compute

O Oracle Cloud Free Tier oferece alocações Always Free: duas VMs micro x86 (VM.Standard.E2.1.Micro: 1 OCPU / 1 GB) e até 4 OCPU / 24 GB ARM Ampere divisíveis entre instâncias.

A arquitetura do laboratório utiliza:
- **Gateway:** 1 × VM.Standard.E2.1.Micro (x86_64, 1 vCPU, 1 GB RAM)
- **Workload nodes:** 3 × VM.Standard.A1.Flex ARM64 com 4 vCPU e 24 GB RAM cada, totalizando 12 vCPU e 72 GB

A escolha de ARM Ampere como arquitetura primária dos workload nodes está documentada em [ADR-0003](../adr/0003-arm-ampere-como-arquitetura-primaria.md).

## Rede

**Rede pública:** Apenas o gateway tem conectividade pública. Portas 80 e 443 ficam fechadas no nível de OCI Security List e firewall do host — o Cloudflare Tunnel usa conexão outbound-initiated, não requer portas inbound abertas.

**Rede privada:** WireGuard mesh em `10.13.13.0/24`. O gateway assume IP `.1` e funciona como hub; os três workload nodes recebem `.5`, `.6` e `.7`. O peering entre workload nodes é direto (full mesh), não roteado pelo hub, para evitar latência desnecessária e criar um SPOF.

**DNS e TLS:** DNS externo aponta para o Cloudflare Edge. TLS é terminado no Cloudflare. DNS interno atualmente usa `/etc/hosts` com entradas estáticas; CoreDNS ou Pi-hole estão no roadmap de médio prazo.

Detalhes de plano de endereçamento, firewall e fluxos de tráfego estão em [`./network.md`](./network.md).

## Padrões de workload

Os três workload nodes são propositalmente generalistas para acomodar experimentos paralelos. Atualmente e no roadmap próximo:

- **scarabeus** — Docker Compose para web apps e APIs, experimentos com k3s (Kubernetes lightweight), prototipagem de Docker Swarm
- **libellula** — Containers Docker para serviços auxiliares (bancos de dados, cache, monitoring, logging)
- **lucanus** — Self-hosted services (Ollama para LLM local, e outros serviços pessoais conforme surgem)

A atribuição não é rígida. Qualquer node pode rodar qualquer workload; a separação é conceitual. Padrões de deployment continuarão intencionalmente variados porque o laboratório também é ambiente de aprendizado — a inflexibilidade de padronizar tudo num único padrão perderia valor pedagógico.

## Postura de segurança

### Medidas atuais

- Sem HTTP/HTTPS público (zero exposed ports via Cloudflare Tunnel)
- SSH hardening: autenticação por chave apenas, porta não-default, fail2ban
- Criptografia de rede entre nodes via WireGuard
- WAF e bot mitigation oferecidos pelo Cloudflare

### Medidas planejadas

- Gestão centralizada de segredos (Vault ou SOPS)
- Baseline de segurança aplicada via Ansible
- Audit logging centralizado
- Scans periódicos de vulnerabilidades

## Observabilidade

Atualmente observabilidade é mínima — htop/btop para CPU/memória, logs por serviço.

O plano de médio prazo adiciona stack centralizado:
- **Métricas:** Prometheus + node_exporter + Grafana
- **Logs:** Loki + Promtail + Grafana
- **Uptime:** Uptime Kuma para blackbox monitoring

## Backup e recovery

Ainda não implementado. A abordagem planejada combina:
- Snapshots de volume da OCI
- Cópias criptografadas off-cloud de bancos, segredos e configs em object store de terceiro
- Drills de recovery trimestrais documentados no runbook

Uma ADR futura detalha a estratégia de backup completa.
