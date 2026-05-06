# Architecture Decision Records

Architecture Decision Records documentam o *porquê* de escolhas não-óbvias, não o resultado. Cada ADR registra o contexto, a decisão tomada e as consequências — permitindo que decisões sejam revisitadas no futuro com informação histórica completa.

O formato segue a abordagem lightweight de Michael Nygard, adaptada para este repositório.

## Índice

| # | Título | Status |
|---|--------|--------|
| 0001 | [Usar Oracle Cloud Free Tier como plataforma do laboratório](./0001-oracle-cloud-free-tier.md) | Aceito |
| 0002 | [Gateway node separado para isolar ingress](./0002-gateway-node-separado.md) | Aceito |
| 0003 | [ARM Ampere como arquitetura primária](./0003-arm-ampere-como-arquitetura-primaria.md) | Aceito |
| 0004 | [Cloudflare Tunnel para zero exposed ports](./0004-cloudflare-tunnel.md) | Aceito |
| 0005 | [WireGuard para a private mesh](./0005-wireguard-mesh.md) | Aceito |

## Escrevendo um novo ADR

1. Copiar `template.md` para `NNNN-titulo-curto.md`, preenchendo o próximo número sequencial.
2. Preencher as seções obrigatórias: Contexto, Decisão, Consequências, Alternativas Consideradas. Incluir sempre consequências negativas — ADR só com positivas é suspeita.
3. Adicionar a nova ADR ao índice acima em ordem numérica.
4. Commitar com mensagem: `adr: NNNN <título-curto>` (ex: `adr: 0006 stack de observabilidade`).
