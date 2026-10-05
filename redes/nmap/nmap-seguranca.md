---
layout: default
title: Nmap — Segurança
---

# Nmap: auditoria, vulnerabilidades e hardening
{:.no_toc}

Use Nmap como fonte de evidência para reduzir superfície de ataque e validar controles. Uma varredura não substitui gestão de vulnerabilidades, inventário, correção de patches, análise de configuração ou monitoramento.

> **Uso autorizado:** execute varreduras apenas em ativos próprios ou para os quais exista autorização formal e escopo definido. Este material é voltado a inventário, validação de exposição, troubleshooting e hardening. Não inclui instruções de exploração, negação de serviço, evasão ou força bruta.

---

<div class="toc-title">Sumário</div>
* Sumário:
{:toc}

---

## 1. Processo de auditoria autorizado

| Fase | Entrega |
| --- | --- |
| Planejamento | Escopo, autorização, janela, contatos e critérios de parada |
| Descoberta | Inventário de hosts e serviços aprovados |
| Validação | Comparação com CMDB, firewall e configuração do host |
| Priorização | Risco, exposição, criticidade do ativo e necessidade de negócio |
| Correção | Hardening, patch, ACL, remoção ou segmentação |
| Reteste | Evidência de que a exposição foi removida ou reduzida |

## 2. Linhas de base

Crie uma linha de base por perfil. Exemplo:

| Perfil | Portas permitidas externamente | Observação |
| --- | --- | --- |
| Bastion | `22/tcp` via VPN ou rede administrativa | MFA, chaves, logs e origem restrita |
| Web | `80/tcp`, `443/tcp` | Preferir redirecionamento de 80 para 443 |
| DNS recursivo interno | `53/tcp`, `53/udp` apenas nas redes internas | Nunca expor recursão indiscriminadamente |
| Banco de dados | Nenhuma porta pública | Acesso apenas de sub-redes de aplicação |

Validação simples pós-hardening:

```bash
nmap -sT -p 22,80,443,3306,5432 192.168.56.20
```

## 3. Exemplos de achados e ações

| Achado | Risco | Validação | Ação recomendada |
| --- | --- | --- | --- |
| `3306/tcp` aberto em servidor web | Exposição indevida de banco | Confirmar processo, origem e regra de firewall | Restringir à rede da aplicação ou desabilitar |
| SSH exposto a rede ampla | Tentativas não autorizadas | Conferir ACL, logs e necessidade | Limitar origem, usar chaves e MFA/VPN |
| HTTP sem HTTPS | Tráfego sem proteção | Verificar redirecionamento e certificados | Habilitar TLS e redirecionar para 443 |
| Serviço legado | Patch e configuração incertos | Confirmar versão real e suporte | Atualizar, isolar ou descontinuar |
| Porta inesperada | Shadow IT ou configuração residual | Identificar dono e processo | Formalizar ou remover |

## 4. Verificação de vulnerabilidades

A identificação de versão pode orientar a consulta de boletins do fornecedor, CVEs e base de patches. Isso não confirma exploração possível: versão pode estar mascarada, corrigida por backport ou protegida por configuração compensatória.

Fluxo seguro:

```text
Serviço e versão observados
          |
          v
Confirmar versão no ativo e pacote instalado
          |
          v
Consultar fornecedor e política de patches
          |
          v
Avaliar exposição, autenticação e segmentação
          |
          v
Corrigir e retestar portas e serviço
```

Evite converter automaticamente resultado de scanner em incidente ou vulnerabilidade confirmada.

## 5. Hardening de rede

- Exponha somente serviços necessários e documentados.
- Restrinja origem por firewall, security groups, ACLs e segmentação.
- Separe administração, aplicação, banco e usuários em redes distintas.
- Desabilite serviços não utilizados e remova regras antigas.
- Aplique patches de sistema, middleware e bibliotecas dentro da política de mudança.
- Use TLS atualizado, certificados válidos e desative protocolos legados.
- Centralize logs e monitore tentativas de conexão, mudanças de firewall e falhas de serviço.

## 6. Hardening do host Linux

Verificar escuta e serviços:

```bash
sudo ss -lntup
sudo systemctl --type=service --state=running
sudo firewall-cmd --list-all
```

Exemplo de política: liberar HTTPS somente quando o serviço for necessário e houver regra de mudança aprovada:

```bash
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

Depois, valide a partir de uma origem autorizada:

```bash
nmap -sT -p 443 192.168.56.20
```

## 7. Relatório mínimo

| Campo | Exemplo |
| --- | --- |
| Identificador | NMAP-2026-001 |
| Ativo | `srv-app-01` / `192.168.56.20` |
| Evidência | Saída Nmap, data, origem e comando aprovado |
| Achado | `3306/tcp` exposta fora da rede de aplicação |
| Impacto | Aumenta superfície de ataque e viola baseline |
| Recomendação | Restringir regra ao CIDR da aplicação |
| Responsável e prazo | Time de plataforma / data acordada |
| Reteste | Saída comparativa pós-correção |

## 8. Limites e ética

Não utilize este processo para enumerar ou testar redes sem autorização. Nmap é uma ferramenta de exploração de rede e auditoria de segurança; uso responsável exige escopo, registro e respeito às regras do ambiente. [cite:84]
