# 0005. WireGuard para a private mesh

- Status: Aceito
- Data: 2026-05-06

## Contexto

Quatro nodes precisam se comunicar de forma privada e segura. Peering no nível de VCN da OCI existe (VCN peering, DRG interconnect) mas é region-specific e prende o modelo de rede a um único provedor — conflita com objetivo de longo prazo de eventualmente estender a laboratório para outro provedor ou hardware local.

Rede overlay criptografada sobre a internet pública é mais flexível. Candidatos principais:

- **WireGuard puro:** Kernel module, configuração manual, sem control plane de terceiros
- **Tailscale:** WireGuard por baixo, control plane SaaS gerenciado, operação simplificada
- **OpenVPN:** Mais antigo, mais pesado, mais lento

## Decisão

WireGuard puro, hub-and-spoke com gateway como hub. Configuração de peers manual em `/etc/wireguard/wg0.conf`. Reconhecer que isso não escala para dezenas de peers, mas para poucos peers é gerenciável e educativo. Revisitar control plane (Tailscale, Headscale, Netbird) conforme a topologia cresça.

## Consequências

### Positivas

- Dependências mínimas (módulo de kernel + userspace tools, sem SaaS)
- Performance excelente, baixo overhead de CPU, criptografia moderna (ChaCha20)
- Conhecimento hands-on do protocolo é transferível independentemente de qual produto de mesh apareça em trabalhos futuros
- Configuração em texto puro facilmente versionada (chaves privadas excluídas)
- Auditar segurança é simples — código e configuração são claros

### Negativas

- Gestão manual não escala além de uns poucos peers — adicionar peer mexe em múltiplas configs
- Sem ACLs nativas — controle de acesso entre peers via firewall do host
- Sem rotação automática de chave — procedimento manual no runbook
- Sem DNS nativo — resolução interna baseada em `/etc/hosts`
- Sem peer discovery — descoberta de peer é manual

## Alternativas Consideradas

- **Tailscale:** Muito mais fácil de operar em escala, gestão de chaves automática, magic DNS. Downside: introduz dependência de control plane SaaS, abstrai detalhes do protocolo valiosos de aprender. Viável para revisitar quando o número de peers crescer além do gerenciável manualmente.
- **Headscale (Tailscale self-hosted):** Interesting middle-ground, mas adiciona um serviço para operar sem ter nodes suficientes para justificar. Rejeitado por enquanto.
- **OpenVPN:** Mais pesado, mais lento, mais antigo — descartado em todo eixo relevante para este laboratório.
