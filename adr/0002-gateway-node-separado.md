# 0002. Gateway node separado para isolar ingress

- Status: Aceito
- Data: 2026-05-06

## Contexto

Quatro VMs disponíveis. O padrão comum para publicar serviços é dar IP público a cada node e rodar um reverse proxy localmente — simples em escala pequena, mas:

- Attack surface espalhado: qualquer node com HTTP/HTTPS público é alvo permanente de scanners e bots
- Workloads experimentais rodam em produção: clusters k3s instáveis, serviços com credenciais default durante prototipagem, software não-auditado — expô-los diretamente à internet é risco desnecessário
- Cada node precisa de gestão TLS: renovação de certificados, configuração de proxy, segurança de proxy — dispersa responsabilidade

## Decisão

Dedicar a menor VM (1 vCPU / 1 GB, x86) como gateway. É a única VM em qualquer fluxo de ingress público. Os três workload nodes são acessíveis apenas pela WireGuard private mesh; não recebem tráfego inbound da internet pública.

## Consequências

### Positivas

- Attack surface concentrado num único node pequeno, reconstruível em minutos
- Workloads podem ser experimentados livremente sem exposição ao público
- Gateway é reconstruível sem impacto nos workloads
- Modelo mental claro: "se não está no gateway, não está na internet"
- Separação de responsabilidade bem definida

### Negativas

- Gateway é SPOF para tráfego público (se cair, serviços públicos ficam inacessíveis)
- 1 GB de RAM limita configurações pesadas de reverse proxy, WAF local ou caching
- Leve overhead de latência inter-node vs ingress direto em cada workload
- Requer gestão de roteamento entre gateway e workloads (resolvido via WireGuard)

## Alternativas Consideradas

- **Ingress público em todo node:** Simples operacionalmente, mas espalha attack surface. Conflita com objetivo de experimentar livremente. Rejeitado.
- **Cloudflare Tunnel em múltiplos nodes:** Distribui o tráfego por múltiplos nodes, mas cada um carrega credenciais Cloudflare — multiplica blast radius se uma credencial foi comprometida. Rejeitado.
- **Par de gateways em HA:** Adiciona complexidade de failover e sincronização de estado. Para um laboratório onde breve downtime é aceitável durante aprendizado, custo não compensa. Rejeitado.
