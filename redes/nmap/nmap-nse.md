---
layout: default
title: Nmap — NSE
---

# Nmap NSE: scripts e automação defensiva
{:.no_toc}

O Nmap Scripting Engine (NSE) amplia a coleta de informações e verificações de serviços. Use scripts apenas dentro de autorização explícita e priorize categorias e scripts classificados como seguros.

> **Uso autorizado:** execute varreduras apenas em ativos próprios ou para os quais exista autorização formal e escopo definido. Este material é voltado a inventário, validação de exposição, troubleshooting e hardening. Não inclui instruções de exploração, negação de serviço, evasão ou força bruta.

---

<div class="toc-title">Sumário</div>
* Sumário:
{:toc}

---

## 1. Como funciona

Em geral, scripts NSE são executados junto com uma varredura de portas; a execução pode depender dos estados e serviços encontrados. Também é possível usar `-sn` para scripts de descoberta de host. [cite:75]

```text
Host e portas autorizados
          |
          v
Nmap identifica serviço elegível
          |
          v
NSE executa script selecionado
          |
          v
Resultado é revisado e validado
```

## 2. Categorias e política de uso

O NSE classifica scripts em categorias como `safe`, `default`, `discovery`, `version`, `auth`, `vuln`, `intrusive`, `brute`, `dos`, `exploit` e outras. Algumas categorias podem ser intrusivas; não use scripts de brute force, DoS, exploração, fuzzing ou intrusivos em produção. [cite:75][cite:77]

| Categoria | Política recomendada |
| --- | --- |
| `safe` | Permitida após validação do escopo |
| `discovery` | Permitida com cuidado e limite de alvo |
| `version` | Permitida para inventário autorizado |
| `default` | Revisar antes; não assumir baixo impacto |
| `vuln` | Só com aprovação explícita e janela de manutenção |
| `intrusive`, `brute`, `dos`, `exploit`, `fuzzer` | Fora deste guia; não executar em ativos reais |

## 3. Listar e documentar scripts disponíveis

```bash
ls /usr/share/nmap/scripts | less
nmap --script-help safe
nmap --script-help ssh-hostkey
nmap --script-help http-title
```

A documentação NSE lista scripts e descrições; leia `--script-help` antes de executar qualquer script. [cite:76][cite:86]

## 4. Exemplos seguros

### Título HTTP

```bash
nmap -sT -p 80,443 --script http-title 192.168.56.20
```

### Chave pública SSH

```bash
nmap -sT -p 22 --script ssh-hostkey 192.168.56.20
```

### Categoria segura em portas limitadas

```bash
nmap -sT -sV -p 22,80,443 --script safe -oN nse-safe.txt 192.168.56.20
```

Scripts seguros são destinados a não derrubar serviços, consumir recursos excessivos ou explorar falhas; muitos são voltados a descoberta. Ainda assim, execute primeiro em laboratório ou em um único ativo. [cite:75]

## 5. Automatizar inventário com Bash

Crie `nmap-inventario.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

ALVOS="${1:?Informe o arquivo de alvos aprovados}"
DATA="$(date +%F)"
SAIDA="evidencias/nmap-${DATA}"

mkdir -p "$SAIDA"
nmap -sT -sV --version-light   -p 22,80,443   --script safe   -iL "$ALVOS"   -oA "$SAIDA/inventario"
```

Execute somente com uma lista revisada:

```bash
chmod 750 nmap-inventario.sh
./nmap-inventario.sh alvos-aprovados.txt
```

## 6. Automatizar com Ansible

Exemplo de coleta controlada no nó de administração:

```yaml
---
- name: Inventário Nmap autorizado
  hosts: localhost
  gather_facts: false
  vars:
    alvo: 192.168.56.20
  tasks:
    - name: Verificar SSH e HTTPS
      ansible.builtin.command:
        argv:
          - nmap
          - -sT
          - -sV
          - --version-light
          - -p
          - 22,443
          - --script
          - safe
          - '{{ alvo }}'
      register: nmap_result
      changed_when: false

    - name: Mostrar resultado
      ansible.builtin.debug:
        var: nmap_result.stdout_lines
```

Não grave saídas de scans em repositórios públicos. Use diretório de evidências protegido, retenção definida e controle de acesso.

## 7. Interpretação e validação

- Script retornou informação não equivale a vulnerabilidade confirmada.
- Um resultado sem saída não prova ausência de risco.
- Correlacione com CMDB, logs, configuração, patch level e dono do serviço.
- Transforme cada achado em ticket com ativo, evidência, criticidade, prazo e responsável.
