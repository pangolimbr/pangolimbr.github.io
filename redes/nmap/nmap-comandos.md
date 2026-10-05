---
layout: default
title: Nmap — Comandos
description: Referência rápida de comandos Nmap para inventário, diagnóstico e validação defensiva.
---

# Nmap: Comandos e Referência Operacional
{:.no_toc}

Referência para executar varreduras previsíveis, limitadas e úteis em ativos autorizados.

> **Uso autorizado:** execute varreduras apenas em redes, hosts e serviços próprios ou formalmente autorizados. Defina escopo, janela, responsáveis e critério de parada. Este material prioriza inventário, diagnóstico e hardening; não cobre exploração, força bruta, evasão ou negação de serviço.

---

<div class="toc-title">Sumário</div>
* Sumário:
{:toc}

---

## 1. Estrutura

```bash
nmap [descoberta] [tipo-de-scan] [portas] [detecção] [NSE] [saída] ALVO
```

Exemplo:

```bash
nmap -n -sT -sV --version-light -p 22,443   -oA evidencias/servidor-01   192.168.56.20
```

## 2. Alvos

| Opção | Exemplo | Uso |
| --- | --- | --- |
| Host único | `192.168.56.20` | Diagnóstico individual |
| Intervalo | `192.168.56.20-30` | Grupo pequeno e aprovado |
| CIDR | `192.168.56.0/28` | Laboratório ou segmento limitado |
| Arquivo | `-iL alvos.txt` | Lista revisada e auditável |
| Exclusão | `--exclude 192.168.56.1` | Retirar gateway ou ativo sensível |
| Sem DNS | `-n` | Evitar resolução reversa e atrasos |

```bash
nmap -n -sT -p 22,443 -iL alvos-aprovados.txt
```

## 3. Descoberta de hosts

| Opção | Uso | Quando usar |
| --- | --- | --- |
| `-sn` | Descoberta sem portas | Inventário inicial |
| `-Pn` | Considera hosts ativos | Somente quando descoberta é bloqueada e há justificativa |
| `-n` | Desativa DNS | Execução previsível e mais rápida |

```bash
nmap -sn -n 192.168.56.0/28
```

Evite `-Pn` em faixas grandes: ele pode gerar tráfego contra todos os endereços e aumentar muito o tempo de scan.

## 4. TCP e UDP

| Opção | Descrição | Observação |
| --- | --- | --- |
| `-sT` | TCP connect | Boa escolha para diagnóstico simples |
| `-sU` | Varredura UDP | Limite portas e espere resultados mais lentos |
| `-p` | Define portas | Prefira lista explícita |
| `--top-ports` | Portas frequentes | Útil para triagem, não para baseline completo |

```bash
# SSH e HTTPS
nmap -sT -p 22,443 192.168.56.20

# DNS UDP controlado
nmap -sU -p 53 192.168.56.53

# Intervalo justificado
nmap -sT -p 8000-8010 192.168.56.20
```

## 5. Detecção de serviço

| Opção | Efeito | Uso recomendado |
| --- | --- | --- |
| `-sV` | Detecta serviço e versão | Inventário e troubleshooting |
| `--version-light` | Menos sondas | Primeira coleta em produção |
| `--version-all` | Mais sondas | Laboratório ou janela autorizada |
| `-O` | Tenta inferir sistema operacional | Validar em escopo restrito; resultado pode ser impreciso |

```bash
nmap -sT -sV --version-light -p 22,80,443 192.168.56.20
```

## 6. NSE seguro

Veja a página [NSE](nse.html) para documentação completa. Para uma coleta inicial, prefira scripts explícitos:

```bash
nmap -sT -sV --version-light   -p 22,80,443   --script ssh-hostkey,http-title,http-headers,ssl-cert   192.168.56.20
```

```bash
nmap --script-help ssh-hostkey
```

Não use `--script all`, categorias agressivas ou scripts de terceiros sem revisão e autorização.

## 7. Tempo e estabilidade

| Opção | Aplicação | Observação |
| --- | --- | --- |
| `-T2` | Ambiente sensível | Mais conservador |
| `-T3` | Padrão equilibrado | Recomendação inicial |
| `--max-retries 2` | Limita tentativas | Pode reduzir detecção em rede instável |
| `--host-timeout 2m` | Limita tempo por host | Evita travar toda a execução |
| `--reason` | Explica o motivo do estado | Excelente para troubleshooting |

```bash
nmap -n -sT -T2 --reason --max-retries 2   -p 22,443 192.168.56.20
```

Não acelere scans de produção por conveniência. Primeiro reduza escopo, portas e intensidade.

## 8. Saída e evidências

| Opção | Resultado |
| --- | --- |
| `-oN arquivo.txt` | Texto legível |
| `-oX arquivo.xml` | XML para integração |
| `-oG arquivo.gnmap` | Formato legado grepável |
| `-oA prefixo` | Gera os três formatos |
| `-v` / `-vv` | Mais detalhes no terminal |

```bash
mkdir -p evidencias
nmap -n -sT -sV --version-light -p 22,443   -oA evidencias/ssh-https-$(date +%F)   192.168.56.20
```

## 9. Perfis prontos

### Descoberta em laboratório

```bash
nmap -sn -n 192.168.56.0/24
```

### Inventário de Linux

```bash
nmap -n -sT -sV --version-light   -p 22,80,443   -iL linux-aprovados.txt
```

### Validação de exposição pós-mudança

```bash
nmap -n -sT --reason   -p 22,80,443,3306,5432   192.168.56.20
```

### Servidor DNS interno

```bash
nmap -n -sT -sU -sV --version-light   -p T:53,U:53   192.168.56.53
```

### HTTPS com certificado e cabeçalhos

```bash
nmap -n -sT -p 443   --script ssl-cert,http-title,http-headers,http-security-headers   192.168.56.20
```

## 10. Quando evitar opções

| Opção ou padrão | Por que evitar como padrão |
| --- | --- |
| `-Pn` em rede ampla | Trata todos os IPs como ativos |
| `-p-` em produção | Amplia tráfego, tempo e superfície de impacto |
| `-sU` sem limitar portas | Pode ser lento e ambíguo |
| `-T4` ou `-T5` | Pode aumentar perda, alertas e impacto operacional |
| `-A` sem revisão | Combina múltiplas técnicas; prefira parâmetros explícitos |
| `--script all` | Pode incluir scripts inadequados ao escopo |

## 11. Consulta rápida

```bash
# Ajuda geral
nmap --help

# Ajuda de parâmetro
man nmap

# Ajuda de script NSE
nmap --script-help ssl-cert

# Detalhar por que cada estado foi atribuído
nmap --reason -sT -p 22,443 192.168.56.20

# Sem resolução DNS
nmap -n -sT -p 22,443 192.168.56.20
```
