# 0001. Usar Oracle Cloud Free Tier como plataforma do laboratório

- Status: Aceito
- Data: 2026-05-06

## Contexto

O laboratório necessita de múltiplas VMs com compute substancial para experimentação com Kubernetes, bancos de dados, self-hosted services e orquestração de containers. A meta financeira é custo zero. Comparação dos free tiers principais:

- **AWS:** 1 VM t2/t3.micro (elegível por 12 meses)
- **GCP:** 1 VM e2-micro (Always Free)
- **Azure:** 1 VM B1S (elegível por 12 meses)
- **OCI:** Até 4 VMs Always Free — especificamente 2 instâncias micro x86 (VM.Standard.E2.1.Micro: 1 OCPU / 1 GB) + até 4 OCPU / 24 GB ARM Ampere (VM.Standard.A1.Flex) divisíveis entre múltiplas instâncias

Apenas OCI oferece alocação generosa o suficiente para um cluster multi-node com compute real, sem expiração.

## Decisão

Usar Oracle Cloud Free Tier como único provedor. Provisionar quatro VMs: 1 micro x86 como edge/gateway + 3 instâncias ARM Ampere (4 OCPU / 24 GB cada) como workload nodes, maximizando a alocação Always Free.

## Consequências

### Positivas

- Compute substancial gratuito: total de 13 vCPU (1 + 12) e 73 GB RAM (1 + 72)
- Experiência prática com topologia multi-node distribuída
- Exposição a ARM (Ampere), tecnologia relevante na indústria (Graviton em AWS, Apple Silicon em dev, etc.)
- Always Free em vez de trial de 12 meses — risco muito menor de surpresas na conta

### Negativas

- OCI é menos comum no mercado de trabalho que AWS/GCP — conhecimento cloud pode ser mais especializado
- Capabilidade ARM às vezes indisponível em regiões populares
- Risco de reclaim por inatividade — OCI cobra se máquinas ficarem idle além de períodos, exigindo atividade ocasional
- Ecossistema de tutoriais e documentação comunitária menor que AWS

## Alternativas Consideradas

- **AWS/GCP/Azure free tier simples:** Uma VM pequena insuficiente para experimentos realistas (k3s, bancos, múltiplos containers). Qualquer compute adicional sai do free tier. Rejeitado.
- **Homelab com hardware próprio:** Custo inicial de hardware + eletricidade contínua. Não oferece exposição a padrões operacionais cloud-native (VPC, security groups, snapshots, IPs públicos/privados). Rejeitado.
- **Múltiplos free tiers combinados:** Usar a quota de cada provedor (AWS + GCP + Azure). Complexidade operacional supera o benefício neste estágio. Rejeitado.
