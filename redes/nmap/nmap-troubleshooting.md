---
layout: default
title: Nmap — Troubleshooting
---

# Nmap: diagnóstico de conectividade e resultados
{:.no_toc}

Guia de diagnóstico para interpretar falhas de resolução, rota, filtragem e estados de porta em ativos autorizados.

> **Uso autorizado:** execute varreduras apenas em ativos próprios ou para os quais exista autorização formal e escopo definido. Este material é voltado a inventário, validação de exposição, troubleshooting e hardening. Não inclui instruções de exploração, negação de serviço, evasão ou força bruta.

---

<div class="toc-title">Sumário</div>
* Sumário:
{:toc}

---

## 1. Método de diagnóstico

```text
Confirmar IP e escopo
        |
        v
Validar DNS, rota e interface local
        |
        v
Testar conectividade básica permitida
        |
        v
Executar Nmap em poucas portas
        |
        v
Comparar com firewall, serviço e logs do alvo
```

Nunca conclua que um serviço está indisponível somente porque um scan falhou. Valide pelo menos rede, ACL/firewall e processo no host.

## 2. Host parece indisponível

Sintoma:

```text
Note: Host seems down.
```

Verificações no cliente:

```bash
getent hosts srv-lab.example.internal
ip route get 192.168.56.20
ping -c 2 192.168.56.20
```

Descoberta sem portas:

```bash
nmap -sn -n 192.168.56.20
```

Se há evidência de que o host está ativo, mas ele bloqueia as sondas de descoberta, documente a autorização e teste somente portas de serviço necessárias:

```bash
nmap -Pn -sT -p 22,443 192.168.56.20
```

## 3. Porta `filtered`

Significa que o Nmap não conseguiu determinar se a porta está aberta, geralmente por filtragem de firewall ou ausência de resposta.

No alvo Linux, valide:

```bash
sudo ss -lntup
sudo systemctl status nome-do-servico
sudo firewall-cmd --list-all
sudo firewall-cmd --list-ports
```

Em sistemas com nftables:

```bash
sudo nft list ruleset
```

Compare a porta, o protocolo, a origem autorizada e a interface. Não abra portas como “teste” sem ticket ou mudança aprovada.

## 4. Porta `closed`

O host respondeu, mas não há serviço aceitando conexão naquela porta. Confirme:

```bash
sudo ss -lntp '( sport = :443 )'
sudo systemctl status nginx
sudo journalctl -u nginx --since '30 minutes ago'
```

Causas comuns:

- Serviço desligado, falhou no boot ou escuta em outra porta
- Serviço limitado a `127.0.0.1` ou interface diferente
- Balanceador, NAT ou proxy aponta para destino incorreto
- Configuração mudou sem atualização de inventário

## 5. Serviço inesperado ou versão imprecisa

Use poucos testes adicionais e compare com o host:

```bash
nmap -sT -sV --version-light -p 22,443 192.168.56.20
curl -vkI https://192.168.56.20/
ssh -v usuario@192.168.56.20
```

Versões podem estar ocultas, alteradas pelo fornecedor, intermediadas por proxy ou inferidas de forma incompleta. A configuração do ativo e o inventário têm precedência sobre a inferência automática.

## 6. DNS não resolve

```bash
getent hosts srv-lab.example.internal
resolvectl query srv-lab.example.internal
cat /etc/resolv.conf
```

Em RHEL/Rocky, confirme também a conexão ativa:

```bash
nmcli device show | grep -E 'IP4.DNS|IP4.GATEWAY'
```

Após corrigir DNS, repita a varredura usando hostname e IP. Se os resultados diferirem, investigue registros desatualizados, split DNS ou IPv4/IPv6.

## 7. Erros de permissão

Algumas técnicas exigem privilégios administrativos ou capacidades de rede. Para um diagnóstico simples sem privilégios, prefira `-sT`:

```bash
nmap -sT -p 22,443 192.168.56.20
```

Não execute como root por hábito. Use o menor privilégio necessário e registre quando privilégios forem indispensáveis.

## 8. Checklist de evidências

- Comando completo, data, fuso e origem do scan
- Alvo, CIDR e autorização associada
- Saída normal e XML, quando permitido
- Rota e resolvedor DNS usados
- Estado do serviço no host
- Regras relevantes de firewall, sem expor segredos
- Conclusão, impacto e ação corretiva
