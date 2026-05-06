# AGENTS.md

Instruções para agentes de IA (OpenCode, Claude Code, Codex, etc.) que trabalham neste repositório.

## Sobre o projeto

`infra-lab` é a documentação viva da minha infraestrutura pessoal no Oracle Cloud Free Tier. É um repositório público no GitHub que serve três audiências: recrutadores avaliando portfólio técnico, comunidade dev e meu eu do futuro. Não é um projeto de código com build/test — é um projeto de documentação e Infrastructure-as-Code que evolui conforme estudo novos tópicos.

## Ambiente real (não invente nada além disso)

Quatro VMs no Oracle Cloud Free Tier, conectadas por WireGuard mesh em `10.13.13.0/24`:

| Host | Papel | Specs | Arquitetura | WireGuard IP |
|------|-------|-------|-------------|--------------|
| `gateway` | Hub WireGuard + Cloudflare Tunnel | 1 vCPU / 1 GB | x86_64 (E2.1.Micro) | `10.13.13.1` |
| `scarabeus` | Web apps, APIs, K8s/Swarm | 4 vCPU / 24 GB | ARM64 (A1.Flex) | `10.13.13.5` |
| `libellula` | Containers Docker | 4 vCPU / 24 GB | ARM64 (A1.Flex) | `10.13.13.6` |
| `lucanus` | Self-hosted services (Ollama) | 4 vCPU / 24 GB | ARM64 (A1.Flex) | `10.13.13.7` |

Princípios arquiteturais que devem ser refletidos em qualquer documentação ou código novo:

1. **Zero exposed ports** — nenhuma VM com HTTP/HTTPS público. Ingress sempre via Cloudflare Tunnel.
2. **Private mesh by default** — tráfego entre nodes via WireGuard.
3. **Custo zero** — tudo dentro do Oracle Cloud Free Tier.
4. **Decisões documentadas** — toda escolha não-óbvia vira um ADR.

Para qualquer valor desconhecido (IPs públicos reais, domínios, region codes), use placeholder `<TODO: descrição>`. Nunca invente.

## Idioma

A documentação é escrita em **português brasileiro**. Eu (autor) faço a tradução para inglês manualmente, de forma progressiva, como prática do idioma.

Regras de idioma:

- Texto, narrativa, explicações, conectivos: **português**.
- Títulos de seções: **português** (`## Visão geral`, não `## Overview`), exceto quando o título é um termo técnico consagrado.
- **Termos técnicos consagrados ficam em inglês**, mesmo dentro do texto português. Não traduzir:
  - Tecnologias e ferramentas: WireGuard, Cloudflare Tunnel, Ansible, Terraform, Docker Compose, Kubernetes, etc.
  - Conceitos de arquitetura: hub-and-spoke, mesh, zero exposed ports, ingress, egress, workload, deployment, runbook, healthcheck, multi-arch, ADR, failover, blast radius, attack surface, single point of failure (SPOF), defense in depth.
  - Comandos, paths e nomes de arquivos: `/etc/wireguard/wg0.conf`, `systemctl`, `iptables`, etc.

Quando eu pedir explicitamente para traduzir um arquivo para inglês, traduza o texto narrativo mantendo os mesmos termos técnicos (eles já estão no idioma certo).

## Tom e estilo

- Técnico-profissional, como um engenheiro sênior documentando o próprio trabalho para colegas.
- Frases curtas. Parágrafos curtos. Sem enchimento.
- Sem linguagem de marketing ("solução robusta", "estado da arte", "stack completa").
- Sem emoji, exceto 🚧 nos placeholders WIP de diretórios.
- Não use voz passiva quando a ativa serve ("o ADR documenta X", não "X é documentado pelo ADR").
- Listas com bullets de uma linha precisam de paralelismo gramatical (todos começando com verbo, ou todos com substantivo).

## Convenções do repositório

### Estrutura de pastas

```
infra-lab/
├── AGENTS.md           ← este arquivo
├── README.md           ← entrada principal com diagrama Mermaid
├── CHANGELOG.md        ← formato Keep a Changelog
├── .gitignore
├── docs/
│   ├── architecture.md
│   ├── network.md
│   ├── runbook.md
│   └── diagrams/       ← .mmd, .drawio, .svg versionados
├── adr/                ← Architecture Decision Records
├── ansible/            ← playbooks (em construção)
├── terraform/          ← módulos IaC (em construção)
└── compose/            ← docker-compose por serviço (em construção)
```

Não crie diretórios ou arquivos fora dessa estrutura sem antes me perguntar.

### ADRs (Architecture Decision Records)

- Formato lightweight do Michael Nygard.
- Numeração sequencial com 4 dígitos: `0001-titulo-curto.md`, `0002-...`.
- Filenames em kebab-case e em português (`0006-stack-de-observabilidade.md`).
- Toda nova ADR deve ser adicionada ao índice em `adr/README.md` na ordem.
- Seções obrigatórias: Contexto, Decisão, Consequências, Alternativas Consideradas.
- Sempre incluir consequências negativas — ADR só com positivas é suspeita e perde valor profissional.
- Status inicial é "Aceito"; "Proposto" só se for explicitamente para discussão.

### Mermaid e diagramas

- Diagramas no `README.md` ficam embutidos em bloco ` ```mermaid `.
- O mesmo diagrama deve existir como arquivo `.mmd` standalone em `docs/diagrams/` para versionamento limpo.
- Paleta consistente: gateway com fill `#fef3c7` stroke `#d97706` (amber); workload nodes com fill `#dbeafe` stroke `#2563eb` (blue).
- Use `flowchart TB` (top-bottom) para topologias gerais, `sequenceDiagram` para fluxos de protocolo.

### Commits

- Conventional Commits, em português, em letras minúsculas: `feat:`, `fix:`, `docs:`, `chore:`, `adr:`.
- ADRs novos: `adr: NNNN <título-curto>` (ex: `adr: 0006 stack de observabilidade`).
- Atualizações de doc: `docs: <escopo> <descrição-curta>` (ex: `docs: runbook procedimento de rotação de chaves`).
- Tradução para inglês: `docs(en): translate <arquivo>` (ex: `docs(en): translate adr/0001`).
- Não commite por mim a menos que eu peça explicitamente.

### Markdown

- Links sempre relativos dentro do repo: `./adr/README.md`, `../docs/network.md`. Nunca URLs absolutas para arquivos do próprio repo.
- Tabelas com alinhamento de coluna explícito quando misturar texto e códigos.
- Code blocks sempre com linguagem declarada (` ```bash `, ` ```yaml `, ` ```mermaid `).
- Blockquotes só para callouts de status ou avisos importantes, não para citações comuns.

## Segurança e segredos

Nunca commite, nunca exiba em output, nunca peça em prompts:

- IPs públicos reais das VMs
- Chaves privadas WireGuard (qualquer arquivo `privatekey`, `*.privatekey`)
- Tokens de API (Cloudflare, OCI, GitHub)
- Conteúdo de `.env`, exceto exemplos `.env.example`
- Senhas, secrets, certificados, chaves SSH privadas

Se eu colar acidentalmente algum desses, alerte e recuse continuar até eu confirmar que removi.

Use placeholders explícitos em todos os exemplos: `<TODO: token>`, `<your-domain.com>`, `<wireguard-public-key>`.

## Workflow esperado para tarefas comuns

### Quando eu pedir uma ADR nova

1. Sugira o próximo número disponível olhando `adr/README.md`.
2. Confirme o título comigo antes de criar o arquivo.
3. Pergunte sobre alternativas consideradas se eu não tiver mencionado nenhuma — ADR sem alternativa é fraca.
4. Após criar o arquivo, atualize o índice em `adr/README.md`.
5. Se a decisão afeta `architecture.md` ou `network.md`, sugira a edição correspondente, mas só execute se eu confirmar.

### Quando eu pedir um playbook Ansible

1. Coloque em `ansible/playbooks/<nome>.yml` (crie a subpasta se não existir).
2. Variáveis em `ansible/group_vars/` ou `ansible/host_vars/`, segredos sempre via `ansible-vault`.
3. Adicione um README curto na primeira execução em `ansible/` se ainda for placeholder.
4. Use módulos idempotentes; nada de `command:` ou `shell:` quando existe módulo nativo.
5. Comente em português dentro do playbook quando a intenção não for óbvia.

### Quando eu pedir um docker-compose

1. Coloque em `compose/<categoria>/<servico>/docker-compose.yml`.
2. Sempre crie `.env.example` documentando todas as variáveis (nunca o `.env` real).
3. Defina `healthcheck` para todo serviço.
4. Use volumes nomeados, não bind mounts, salvo motivo justificado.
5. Pin versões de imagem (`postgres:16.4`, não `postgres:latest`).
6. Para imagens multi-arch, valide que existe build ARM64 antes de sugerir — vários self-hosted populares só têm x86.

### Quando eu pedir tradução para inglês

1. Traduza o **texto narrativo** (parágrafos, explicações, conectivos).
2. **Não traduza** termos técnicos consagrados — eles já estão no idioma certo.
3. Mantenha a estrutura idêntica: mesmas seções, mesmos níveis de heading, mesmas tabelas.
4. Crie o arquivo traduzido em paralelo (`docs/architecture.en.md`) ou substitua o original — me pergunte qual workflow seguir antes de começar.
5. Não traduza ADRs já aceitas sem eu pedir explicitamente — elas registram um momento e não devem ser editadas casualmente.

## O que NÃO fazer

- **Não execute `git push`** em nenhuma circunstância. Sempre eu mesmo.
- **Não crie `LICENSE`** — vou adicionar pela web UI do GitHub.
- **Não rode comandos contra a infra real** (ssh, ansible-playbook, terraform apply, kubectl). Este repo é só documentação e definições; a execução é responsabilidade minha.
- **Não invente versões, IPs, hostnames ou regiões** que não estão neste arquivo.
- **Não adicione dependências** (módulos Ansible de coleções externas, providers Terraform de terceiros, imagens Docker obscuras) sem antes me explicar a alternativa "vanilla" e por que ela não basta.
- **Não substitua texto existente** silenciosamente quando eu pedir adição. Se vai mexer em algo já escrito, mostre o diff e pergunte.

## Quando estiver em dúvida

Pergunte antes de gerar. Erros de hostname, IP, versão ou idioma são mais caros de corrigir depois do que a frição de uma pergunta agora.