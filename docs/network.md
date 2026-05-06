# Rede

## Visão geral

WireGuard mesh hub-and-spoke com peering full-mesh entre workload nodes. O gateway atua como hub — todos os peers listam o gateway como endpoint inicial — mas os três workload nodes também fazem peering direto entre si, evitando round-trip desnecessário e criando redundância. O tráfego encriptado flui sobre a internet pública; todo node possui chave privada e participa da mesh em igualdade.

## Plano de endereçamento

### Subnets

| Subnet | Propósito |
|--------|-----------|
| `10.13.13.0/24` | WireGuard private mesh |
| `<oci-vcn-cidr>` | Oracle Cloud VCN (por node, não relevante para tráfego entre VMs) |

### Atribuição de IPs

| Hostname | IP WireGuard | IP Público | Notas |
|----------|--------------|------------|-------|
| `gateway` | `10.13.13.1` | Estático | Hub do WireGuard, endpoint do Cloudflare Tunnel |
| `scarabeus` | `10.13.13.5` | Estático | Workload node — web apps, APIs, K8s/Swarm |
| `libellula` | `10.13.13.6` | Estático | Workload node — containers Docker |
| `lucanus` | `10.13.13.7` | Estático | Workload node — self-hosted services (Ollama) |

> IPs públicos reais não estão neste repositório por questão de segurança e ficam armazenados externamente.

## Fluxo de ingress

```
Usuário → Cloudflare Edge → Cloudflare Tunnel → gateway (cloudflared)
→ WireGuard mesh → workload node → aplicação
```

O Cloudflare Tunnel é outbound-initiated a partir do gateway. O daemon `cloudflared` abre uma conexão TCP/HTTPS contínua para o edge da Cloudflare, que depois roteia requisições inbound (HTTP/HTTPS de usuários) de volta por esse túnel. Dessa forma, não é necessário abrir nenhuma porta inbound de HTTP/HTTPS no gateway nem em nenhuma VM — todo tráfego público é mediado pelo Cloudflare, que fornece terminação TLS, WAF e proteção contra DDoS.

## Fluxo de egress

Workload nodes fazem egress diretamente pelos próprios IPs públicos da OCI (atualizações de pacotes, pulls de containers Docker, requisições HTTPS para APIs externas). Egress não é roteado pelo gateway — criar tal engarrafamento criaria um SPOF e saturaria a VM pequena rapidamente.

## Regras de firewall

Duas camadas: OCI Security Lists/Network Security Groups no nível cloud e `iptables`/`nftables` no nível host como defense in depth.

### Inbound

**Gateway:**
- UDP porta 51820 (WireGuard)
- TCP porta <ssh-port-nao-default> (SSH, fonte restrita a IPs administrativos)
- Sem HTTP/HTTPS (Cloudflare Tunnel é outbound-initiated)

**Workload nodes (scarabeus, libellula, lucanus):**
- UDP porta 51820 (WireGuard)
- TCP porta <ssh-port-nao-default> (SSH, fonte restrita a IPs administrativos)
- Sem HTTP/HTTPS público

### Outbound

Sem restrições (necessary para atualizações, pulls de container, requisições HTTP/HTTPS).

## Estratégia de DNS

DNS externo aponta para o edge da Cloudflare. Não há IP público de nenhuma VM exposto diretamente em registros — o ponto de entrada é sempre o Cloudflare Tunnel.

DNS interno atualmente usa `/etc/hosts` com entradas estáticas dos IPs WireGuard de cada node. Médio prazo: CoreDNS ou Pi-hole para service discovery automático.

## Modos de falha

**Queda do gateway:** O serviço público fica inacessível (Cloudflare Tunnel quebrado). A mesh WireGuard continua funcionando entre workload nodes — eles conseguem se comunicar diretamente. O gateway é reconstruível em minutos.

**Comprometimento de chave WireGuard:** Rotação per-peer é possível sem reconstruir o gateway. A chave privada de um node pode ser regenerada, a chave pública é atualizada no gateway e nos peers diretos, e peering continua.

**Queda da Cloudflare:** Sem fallback alternativo de ingress hoje — serviços públicos ficam inacessíveis. O risco é aceitável para um laboratório não-produção. Médio prazo: segunda rota de ingress (p. ex., Tailscale Funnel ou SSH reverso) como backup.
