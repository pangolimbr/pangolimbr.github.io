---
layout: default
title: Nmap — Comandos
---

# Nmap: referência rápida de parâmetros
{:.no_toc}

Referência operacional para inventário e validação defensiva em redes autorizadas.

> **Uso autorizado:** execute varreduras apenas em ativos próprios ou para os quais exista autorização formal e escopo definido. Este material é voltado a inventário, validação de exposição, troubleshooting e hardening. Não inclui instruções de exploração, negação de serviço, evasão ou força bruta.

---

<div class="toc-title">Sumário</div>
* Sumário:
{:toc}

---

## 1. Estrutura do comando

```bash
nmap [tipo-de-scan] [descoberta] [portas] [detecção] [saída] ALVO
```

Exemplo de baixo impacto para um servidor autorizado:

```bash
nmap -sT -sV -p 22,80,443 -oN servidor-web.txt 192.168.56.20
```

## 2. Seleção de alvos

| Sintaxe | Uso |
| --- | --- |
| `192.168.56.20` | Um host explícito |
| `192.168.56.20-30` | Intervalo controlado |
| `192.168.56.0/28` | CIDR pequeno em laboratório |
| `-iL alvos.txt` | Arquivo de alvos aprovados |
| `--exclude 192.168.56.1` | Exclusão documentada |

```bash
nmap -sT -p 22,443 -iL alvos-aprovados.txt
```

## 3. Descoberta de hosts

| Opção | Finalidade |
| --- | --- |
| `-sn` | Descoberta sem varredura de portas |
| `-Pn` | Não executar descoberta; use apenas quando houver justificativa |
| `-n` | Não resolver DNS, tornando a execução mais previsível |
|

```bash
nmap -sn -n 192.168.56.0/28
```

`-Pn` pode aumentar muito o tempo e o volume de tráfego porque faz o Nmap tratar os alvos como ativos; não o use como padrão. [cite:88]

## 4. Portas e protocolos

| Opção | Exemplo | Finalidade |
| --- | --- | --- |
| `-p 22` | `-p 22` | Uma porta |
| `-p 22,80,443` | `-p 22,80,443` | Lista curta |
| `-p 8000-8100` | `-p 8000-8100` | Intervalo justificado |
| `--top-ports 20` | `--top-ports 20` | Portas frequentes, para triagem |
| `-sT` | `-sT` | TCP connect; útil sem privilégios especiais |
| `-sU` | `-sU -p 53,123` | UDP; execute de forma limitada e com cuidado |

Para ambientes críticos, comece por portas conhecidas do serviço. Uma varredura UDP pode ser mais lenta e gerar resultados ambíguos por filtragem ou ausência de resposta.

## 5. Detecção de serviço

| Opção | Finalidade |
| --- | --- |
| `-sV` | Tenta identificar serviço e versão |
| `--version-light` | Menos sondas; útil em triagem com menor custo |
| `--version-all` | Mais intensidade; exige autorização e janela adequada |

```bash
nmap -sT -sV --version-light -p 22,443 192.168.56.20
```

## 6. Saída e evidência

| Opção | Resultado |
| --- | --- |
| `-oN arquivo.txt` | Saída normal legível |
| `-oX arquivo.xml` | XML para ferramenta ou processamento |
| `-oG arquivo.gnmap` | Formato grepável legado |
| `-oA prefixo` | Gera normal, XML e grepável |
| `-v` / `-vv` | Mais detalhes na tela |

```bash
mkdir -p evidencias
nmap -sT -sV -p 22,443 -oA evidencias/web-2026-10-05 192.168.56.20
```

## 7. Desempenho e cuidado

| Opção | Orientação |
| --- | --- |
| `-T2` | Mais conservador; útil para redes sensíveis |
| `-T3` | Padrão equilibrado para muitos casos |
| `--max-retries 2` | Limita repetição; documente a perda potencial de detecção |
| `--host-timeout 2m` | Evita host problemático consumir a janela toda |

Evite acelerar agressivamente scans de produção. Se houver degradação, pare a execução, registre o horário e envolva a equipe responsável.

## 8. Perfis defensivos

### Inventário de SSH e HTTPS

```bash
nmap -sT -sV --version-light -p 22,443 -oN inventario.txt -iL alvos-aprovados.txt
```

### Verificação de DNS autorizado

```bash
nmap -sU -sV -p 53 -oN dns-udp.txt 192.168.56.53
```

### Confirmação pós-hardening

```bash
nmap -sT -p 22,80,443,3306,5432 192.168.56.20
```

O último exemplo confirma que apenas portas necessárias permanecem expostas; não substitui revisão de firewall, autenticação ou logs.
