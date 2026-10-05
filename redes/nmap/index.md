---
layout: default
title: Nmap
---

# Nmap: conceitos, instalação e funcionamento
{:.no_toc}

O **Nmap** (Network Mapper) é uma ferramenta de descoberta de rede e auditoria de segurança. Em um escopo autorizado, ele ajuda a identificar hosts alcançáveis, portas expostas, serviços, versões e evidências de controles de filtragem.

> **Uso autorizado:** execute varreduras apenas em ativos próprios ou para os quais exista autorização formal e escopo definido. Este material é voltado a inventário, validação de exposição, troubleshooting e hardening. Não inclui instruções de exploração, negação de serviço, evasão ou força bruta.

---

<div class="toc-title">Sumário</div>
* Sumário:
{:toc}

---

## 1. Conceitos

| Termo | Significado prático |
| --- | --- |
| Host discovery | Descoberta de endereços que respondem às sondas configuradas |
| Porta | Identificador TCP ou UDP de um serviço de rede |
| Serviço | Processo que escuta uma porta, como SSH, DNS ou HTTPS |
| Estado da porta | Resultado observado: `open`, `closed`, `filtered` ou outros |
| Fingerprint | Evidência usada para inferir produto, versão ou sistema operacional |
| Escopo | Lista aprovada de IPs, redes, janelas e técnicas permitidas |

Uma porta `open` sugere que há serviço aceitando conexões. `closed` indica que o host respondeu, mas nenhum serviço aceita naquela porta. `filtered` normalmente indica que firewall, ACL ou outro filtro impediu uma conclusão. Não trate a saída como prova absoluta: rota, NAT, proxy, IDS/IPS e política de firewall alteram o resultado.

## 2. Fluxo de uma varredura

```text
Definir autorização e escopo
          |
          v
Descobrir hosts permitidos
          |
          v
Identificar portas e protocolos necessários
          |
          v
Identificar serviço e versão, quando autorizado
          |
          v
Validar achados manualmente e registrar evidências
          |
          v
Corrigir, testar novamente e fechar o relatório
```

Por padrão, o Nmap faz descoberta de hosts antes da varredura de portas. A opção `-sn` limita a execução à descoberta, sem varredura de portas. [cite:79]

## 3. Instalação

### Rocky Linux, RHEL, AlmaLinux e Fedora

```bash
dnf install -y nmap
nmap --version
rpm -q nmap
```

### Debian e Ubuntu

```bash
sudo apt update
sudo apt install -y nmap
nmap --version
```

### Verificar binários e documentação local

```bash
command -v nmap
man nmap
nmap --help | less
```

Prefira o repositório da distribuição. A página oficial também oferece binários e código-fonte para os principais sistemas. [cite:78][cite:82]

## 4. Laboratório seguro

Crie uma rede isolada com uma VM de administração e uma VM de teste sob seu controle. Exemplo:

| Ativo | Endereço | Papel |
| --- | --- | --- |
| `admin-lab` | `192.168.56.10` | Máquina que executa Nmap |
| `srv-lab` | `192.168.56.20` | Alvo autorizado |
| Rede | `192.168.56.0/24` | Rede isolada de laboratório |

Antes de começar, registre: responsável, CIDRs permitidos, data/hora, técnicas autorizadas, limite de taxa, portas prioritárias e procedimento de parada.

## 5. Primeira execução

Descobrir apenas hosts em uma sub-rede de laboratório:

```bash
nmap -sn 192.168.56.0/24
```

Checar portas TCP comuns de um único ativo autorizado:

```bash
nmap -sT -p 22,80,443 192.168.56.20
```

Identificar serviço e versão nas portas permitidas:

```bash
nmap -sV -p 22,80,443 192.168.56.20
```

Salvar saída em formato normal e XML para evidência:

```bash
mkdir -p evidencias
nmap -sV -p 22,80,443 -oN evidencias/srv-lab.txt -oX evidencias/srv-lab.xml 192.168.56.20
```

## 6. Leitura responsável dos resultados

Para cada porta aberta, registre:

- IP, hostname e proprietário do ativo
- Protocolo, porta, estado e serviço detectado
- Evidência de versão, quando disponível
- Necessidade de negócio e responsável pelo serviço
- Controle de acesso atual: firewall, ACL, VPN ou segmentação
- Ação: manter, restringir, atualizar, desabilitar ou investigar

Uma versão identificada é indício, não confirmação definitiva. Valide com inventário, logs, `systemctl`, configuração do serviço e responsável do sistema.

## 7. Boas práticas

- Comece por um host e poucas portas; aumente o escopo apenas após validar impacto e resultado.
- Prefira janelas de mudança para ambientes produtivos.
- Salve evidências fora do repositório público; elas podem expor topologia e versões.
- Use alvos explícitos, como `192.168.56.20`, em vez de faixas amplas sem necessidade.
- Trate o resultado como dado sensível de segurança.
