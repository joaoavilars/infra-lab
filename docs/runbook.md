# Runbook

Este documento é feito para ser skimável durante incidentes, não lido de ponta a ponta. Navegue pela seção relevante.

## Adicionar um novo peer no WireGuard

1. **Gerar par de chaves no novo node:**
   ```
   wg genkey | tee privatekey | wg pubkey > publickey
   ```
   Armazene `privatekey` com segurança (nunca commit no repositório).

2. **No gateway, adicionar o peer em `/etc/wireguard/wg0.conf`:**
   Encontre o próximo IP disponível em `10.13.13.0/24` (provavelmente `.8`, `.9`, etc.). Adicione uma seção `[Peer]` com a chave pública do novo node e o IP alocado.

3. **Recarregar o WireGuard no gateway:**
   ```
   sudo wg-quick down wg0 && sudo wg-quick up wg0
   ```
   Ou, para hot reload sem desconectar peers existentes:
   ```
   sudo wg syncconf wg0 <(wg-quick strip /etc/wireguard/wg0.conf)
   ```

4. **No novo node, configurar `/etc/wireguard/wg0.conf`:**
   Apontando o gateway como endpoint (`<gateway-public-ip>:51820`), configurar chave privada, IP de mesh e adicionar peer block para o gateway.

5. **Subir a interface no novo node:**
   ```
   sudo wg-quick up wg0
   ```

6. **Verificar conectividade:**
   ```
   ping 10.13.13.1
   ```
   A partir do novo node, ping no IP do gateway. Se passar, o peer está ativo.

7. **Atualizar documentação:**
   Adicionar a entrada do novo peer a [`./network.md`](./network.md) na tabela de atribuição de IPs.

## Rotacionar chaves do WireGuard

Rotacione per-peer, não a mesh inteira. Regenerar chaves em lote cria risco de indisponibilidade se algo falhar.

1. **No node afetado, gerar novo par de chaves:**
   ```
   wg genkey | tee privatekey.new | wg pubkey > publickey.new
   ```

2. **Atualizar o bloco de peer no gateway:**
   Editar `/etc/wireguard/wg0.conf` substituindo a chave pública antiga do peer pela nova.

3. **Atualizar qualquer peer que liste este node como peer direto:**
   Se este node faz peering direto com outros workload nodes, aqueles também precisam da chave pública atualizada.

4. **Recarregar WireGuard nos hosts afetados:**
   ```
   sudo wg-quick down wg0 && sudo wg-quick up wg0
   ```

5. **Verificar ping na mesh:**
   ```
   ping 10.13.13.5  # ou qualquer outro IP de peer
   ```

## Reiniciar o Cloudflare Tunnel

```
sudo systemctl restart cloudflared
sudo systemctl status cloudflared
journalctl -u cloudflared -f  # tail dos logs
```

Se o tunnel não voltar:

- Verificar dashboard Cloudflare (Zero Trust → Networks → Tunnels) — o status deve estar Green.
- Verificar conectividade outbound do gateway: `curl https://www.cloudflare.com`
- Verificar token em `/etc/cloudflared/config.yml` (nunca commit no repositório, armazenar externamente).

## Diagnósticos comuns

### A mesh está saudável?

A partir de qualquer node, fazer ping em todos os IPs `10.13.13.X` ativos:

```
ping 10.13.13.1
ping 10.13.13.5
ping 10.13.13.6
ping 10.13.13.7
```

Se um ping falhar, fazer SSH direto pelo IP público naquele node e verificar:

```
sudo wg show
```

Procurar por erros de sintaxe em `/etc/wireguard/wg0.conf` ou chaves desatualizadas.

### Um serviço público está acessível?

```
curl -I https://service.example.com
```

Ordem de debug por camadas:

1. **DNS:** `nslookup service.example.com` — se falhar, o problema é com DNS na Cloudflare.
2. **Tunnel:** Verificar status em Cloudflare Zero Trust dashboard. Se estiver Red, o tunnel caiu.
3. **cloudflared no gateway:** `sudo systemctl status cloudflared` e `journalctl -u cloudflared -n 50`.
4. **Alcançabilidade WireGuard:** Ping no IP WireGuard do workload node que rodeia o serviço.
5. **Saúde do serviço:** `ssh` direto no workload node e verificar status do container/processo.

### Um node está sem recursos?

```
htop  # CPU e memória
df -h  # Espaço em disco
docker stats  # Uso de container (se aplicável)
```

Se memória/disco cheios, remover containers parados, imagens sem tag, volumes órfãos. Se CPU saturada, revisar `docker ps` e avaliar qual workload está consumindo.

## Provisionar um novo node

Manual até playbooks Ansible existirem (estão em construção).

1. **Provisionar VM na OCI:**
   VM.Standard.A1.Flex (ARM Ampere), Ubuntu LTS (22.04 ou posterior), 4 vCPU / 24 GB RAM.

2. **Configurar SSH:**
   - Autenticação por chave apenas (desabilitar password auth)
   - Porta não-default (não 22)
   - Instalar `fail2ban` para proteção contra brute force

3. **Atualizar sistema:**
   ```
   sudo apt update && sudo apt full-upgrade -y && sudo reboot
   ```

4. **Instalar Docker e Compose:**
   ```
   curl -fsSL https://get.docker.com | sudo bash
   sudo usermod -aG docker $USER
   sudo apt install -y docker-compose-plugin
   ```

5. **Instalar WireGuard e juntar à mesh:**
   ```
   sudo apt install -y wireguard wireguard-tools
   ```
   Configurar `/etc/wireguard/wg0.conf` conforme seção "Adicionar um novo peer", depois `sudo wg-quick up wg0`.

6. **Configurar firewall do host:**
   ```
   sudo apt install -y ufw
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   sudo ufw allow <ssh-port>/tcp
   sudo ufw allow 51820/udp
   sudo ufw enable
   ```

7. **Atualizar documentação:**
   - Adicionar entrada em `README.md` (tabela de inventário)
   - Adicionar entrada em [`./network.md`](./network.md) (tabela de atribuição de IPs)

Quando playbooks Ansible estiverem prontos, este fluxo inteiro vira uma única execução: `ansible-playbook ansible/playbooks/bootstrap.yml -i inventory/hosts -e new_node=<hostname>`.

## Backups (Placeholder)

Ainda não implementado. Consulte [`./architecture.md`](./architecture.md) para a abordagem planejada.
