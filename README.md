# 🛡️ Laboratório de Auditoria de Autenticação: Prevenção contra Força Bruta com Medusa e Kali Linux

Este projeto faz parte do Desafio Prático da **[DIO (Digital Innovation One)](https://dio.me)**, com o objetivo de demonstrar na prática o funcionamento de testes de autenticação automatizados em ambiente seguro e controlado, bem como propor medidas essenciais de mitigação e fortalecimento (*hardening*) dos serviços.

---

## 🎯 Cenário e Topologia

O laboratório foi configurado de forma 100% isolada utilizando o **VirtualBox** em modo de rede exclusiva do hospedeiro (*Host-Only*):

- **Máquina de Auditoria:** Kali Linux (`192.168.56.x`)
- **Máquina Alvo:** Metasploitable 2 (`192.168.56.101`)

---

## 🔍 Cenário 1: Auditoria no Serviço FTP (Porta 21)

### 1. Mapeamento de Serviços (Reconhecimento)
Identificação das portas e serviços ativos na máquina de destino com `nmap`:
```bash
nmap -sV 192.168.56.101
```
*Resultado:* Identificado o serviço `vsftpd 2.3.4` ativo na porta 21/TCP.

### 2. Criação das Wordlists
Elaboração de listas personalizadas de usuários (`usuarios.txt`) e senhas (`senhas.txt`) para validar combinações previsíveis:
```bash
cat << 'EOF' > usuarios.txt
admin
user
teste
msfadmin
root
EOF
```

### 3. Execução do Teste com Medusa
Execução do teste de autenticação automatizado direcionado ao protocolo FTP:
```bash
medusa -h 192.168.56.101 -U usuarios.txt -P senhas.txt -M ftp -f
```
*Parâmetros utilizados:*
- `-h`: Endereço IP do host de destino.
- `-U`: Caminho do arquivo contendo a lista de usuários.
- `-P`: Caminho do arquivo contendo a lista de senhas.
- `-M ftp`: Módulo de protocolo FTP.
- `-f`: Interrupção do processo assim que a primeira credencial válida for encontrada.

**Resultado obtido:**
> `ACCOUNT FOUND: [ftp] Host: 192.168.56.101 User: msfadmin Password: msfadmin [SUCCESS]`

### 4. Validação de Acesso
Conexão real no serviço para atestar a autenticidade da credencial:
```bash
ftp 192.168.56.101
# Usuário: msfadmin | Senha: msfadmin
```
Status retornado: `230 Login successful` com listagem de diretórios efetuada com sucesso.

---

## 🛡️ Medidas de Mitigação e Hardening (Como se Defender)

A facilidade com que o serviço foi acessado evidencia falhas críticas que devem ser corrigidas em ambientes de produção:

1. **Política de Senhas Fortes:**
   - Evitar credenciais padrão (`admin/admin`, `msfadmin/msfadmin`).
   - Implementar comprimento mínimo (12+ caracteres) contendo letras maiúsculas, minúsculas, números e caracteres especiais.

2. **Bloqueio de Tentativas e Rate Limiting:**
   - Utilizar ferramentas de proteção como o **Fail2ban** para monitorar logs de autenticação e bloquear temporariamente endereços IP que apresentem falhas consecutivas de login.

3. **Substituição de Protocolos Inseguros:**
   - O FTP trafega dados e credenciais em texto claro. A recomendação é desativar o FTP convencional e adotar **SFTP** (via SSH) ou **FTPS** (FTP com TLS/SSL).

4. **Autenticação Multifator (MFA):**
   - Exigir uma segunda etapa de validação para logins críticos.

---

## 🚀 Próximas Etapas do Laboratório
- [x] Teste de autenticação em serviço FTP
- [ ] Enumeração de usuários e Password Spraying em SMB
- [ ] Automação de tentativas em formulário Web (DVWA)
