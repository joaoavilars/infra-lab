# Terraform

> 🚧 **Em construção.** Este diretório vai conter as definições de Infrastructure-as-Code para os recursos da OCI usados pelo laboratório.

## Módulos planejados

- `network/` — VCN, subnets, security lists, network security groups
- `compute/` — Instâncias de VM (gateway e workload nodes)
- `dns/` — Registros DNS na Cloudflare e configuração de tunnel

## Gestão de estado

State remoto em OCI Object Storage com versionamento habilitado. State files (`.tfstate`) nunca commitados no git — armazenados remotamente e sincronizados apenas durante `terraform apply/plan`.
