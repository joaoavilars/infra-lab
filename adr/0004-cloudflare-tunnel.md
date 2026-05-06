# 0004. Cloudflare Tunnel para zero exposed ports

- Status: Aceito
- Data: 2026-05-06

## Contexto

O laboratório precisa publicar HTTP/HTTPS. Abordagem tradicional: abrir portas 80/443 na OCI Security List e firewall do host, rodar reverse proxy (Nginx/Caddy/HAProxy).

Funciona, mas o gateway fica constantemente sob ruído de fundo da internet — scanners automatizados, tentativas de exploit, brute force no SSH, etc. Qualquer zero-day no reverse proxy se torna problema imediato. Gestão manual de certificados TLS (renovação, armazenamento seguro, rollout) é overhead operacional.

Cloudflare Tunnel oferece modelo alternativo: conexão outbound-initiated do gateway até o edge da Cloudflare, que depois roteia o tráfego inbound — sem precisar abrir nenhuma porta HTTP/HTTPS inbound.

## Decisão

Usar Cloudflare Tunnel (`cloudflared`) no gateway como único mecanismo de ingress público. Manter portas 80 e 443 fechadas tanto na OCI Security List quanto no firewall do host.

## Consequências

### Positivas

- Zero attack surface pública em HTTP/HTTPS — scanners não veem nada
- Terminação TLS gratuita no edge da Cloudflare com gestão automática de certificado
- WAF, bot mitigation e rate limiting oferecidos pelo Cloudflare
- DNS, TLS, ingress gerenciados num único dashboard
- Túnel outbound é bem mais seguro que abrir porta inbound

### Negativas

- Dependência forte da Cloudflare — se cair, serviços públicos ficam inacessíveis
- Sem fallback alternativo de ingress hoje (risco aceitável para laboratório, não para produção crítica)
- Alguns protocolos não-HTTP exigem configuração extra ou não estão no free tier
- Debug agora requer entender internals do tunnel além do stack do host

## Alternativas Consideradas

- **Exposição direta com Nginx/Caddy:** Operacionalmente familiar, mas aceita attack surface permanente e manutenção de TLS. Rejeitado — conflita com objetivo de zero exposed ports.
- **Tailscale Funnel:** Conceito similar a Cloudflare Tunnel, atraente para projetos pessoais, porém menos maduro e atrelado ao modelo de conta Tailscale. Ainda viável; foi considerado mas Cloudflare oferece feature set maior no free tier.
- **Acesso só por VPN:** Viável para serviços puramente pessoais, mas inviabiliza compartilhar demos com colaboradores ou recrutadores — conflita com objetivo de usar como portfólio técnico público.
