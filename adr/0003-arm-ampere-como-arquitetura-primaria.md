# 0003. ARM Ampere como arquitetura primária

- Status: Aceito
- Data: 2026-05-06

## Contexto

O Always Free da OCI oferece uma alocação generosa de ARM Ampere (4 OCPU / 24 GB RAM por instância, até 4 OCPU / 24 GB no total divisível entre instâncias). Para usar o free tier por completo, a maior parte do compute precisa ser ARM.

ARM em servidores não é mais nicho:
- AWS Graviton é produção-ready em larga escala
- Apple Silicon popularizou ARM em desenvolvimento
- Ecossistema de software madurou — imagens multi-arch Docker são padrão
- Pontos de atrito históricos (compiladores, bibliotecas, drivers) foram amplamente resolvidos

Pontos de atrito remanescentes:
- Subset pequeno de software legado continua x86-only
- Performance absoluta difere — benchmarks local não são diretamente transferíveis para produção x86
- Alguns provadores proprietários oferecem suporte limitado a ARM

## Decisão

Padronizar os três workload nodes em ARM64 (Ampere). Reservar a x86 micro para o gateway, onde o workload é leve (reverse proxy, Cloudflare Tunnel) e compatibilidade ampla de software é mais importante que poder computacional.

## Consequências

### Positivas

- Uso máximo do free tier: 3 workload nodes com 4 vCPU / 24 GB cada (vs seria impossível com x86 dentro do orçamento)
- Experiência hands-on com ARM em servidor, acompanhando tendências da indústria
- Força hábitos bons como explicitar arquitetura em docker images e verificar multi-arch support
- Maior eficiência de poder (ARM é tipicamente mais power-efficient que x86)

### Negativas

- Subset de containers é x86-only — requer emulação QEMU (lento) ou alternativas
- Alguns softwares proprietários/legados não rodam em ARM
- Performance diferente de x86 — benchmarks locais menos transferíveis
- Complexidade operacional aumenta se experimentar com deployment misto de arquiteturas

## Alternativas Consideradas

- **Laboratório all-x86:** Usar apenas as duas VMs micro x86 gratuitas + pagar por compute adicional. Derrota o objetivo de custo zero. Rejeitado.
- **Deployment misto (x86 + ARM nos workload nodes):** Cada workload precisaria de variantes de imagem por arquitetura, pinning de deployment por node, gestão manual de compatibilidade. Complexidade supera o benefício neste estágio. Rejeitado.
