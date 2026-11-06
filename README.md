

#  Laboratório de Auditoria de Autenticação e Hardening de Sistemas

Projeto prático desenvolvido para o desafio da **[DIO (Digital Innovation One)](https://dio.me)**, abordando técnicas de reconhecimento, auditoria de credenciais em múltiplos serviços de rede e implementação de medidas defensivas (*hardening*).

---

##  Topologia e Ambiente de Testes

O laboratório foi implementado em ambiente 100% controlado e isolado utilizando o **VirtualBox** em rede exclusiva do hospedeiro (*Host-Only*):

- **Máquina de Auditoria:** Kali Linux (`192.168.56.x`)
- **Máquina Alvo:** Metasploitable 2 (`192.168.56.101`)

---

##  1. Mapeamento e Reconhecimento de Serviços

Varredura inicial realizada com o `nmap` para identificar portas abertas e versões de serviços ativos no alvo:

```bash
nmap -sV 192.168.56.101
```

**Principais serviços identificados:**
- **Porta 21/TCP:** FTP (`vsftpd 2.3.4`)
- **Portas 139 e 445/TCP:** SMB (`Samba smbd 3.X - 4.X`)
- **Porta 80/TCP:** HTTP (`Apache httpd 2.2.8`)

---

##  2. Cenário 1: Auditoria no Serviço FTP (Porta 21)

### Criação das Listas de Teste
Elaboração de listas personalizadas de usuários e senhas via terminal no Kali:
```bash
cat << 'EOF' > usuarios.txt
admin
user
teste
msfadmin
root
login
EOF

cat << 'EOF' > senhas.txt
123456
password
admin
msfadmin
toor
senha
EOF

Execução do Teste com Medusa:

medusa -h 192.168.56.101 -U usuarios.txt -P senhas.txt -M ftp -f

Resultado: Credencial válida encontrada: msfadmin:msfadmin com status [SUCCESS].
Validação do Acesso

Conexão direta ao serviço de FTP para comprovar a legitimidade do acesso:

bash
ftp 192.168.56.101
Status retornado: 230 Login successful com listagem de diretórios realizada com sucesso.
 3. Cenário 2: Enumeração e Password Spraying em SMB (Portas 139/445)
Enumeração de Usuários

Identificação de usuários registrados no sistema através do protocolo SMB:

bash
enum4linux -U 192.168.56.101
Resultado: Extraída lista de contas locais existentes no alvo (msfadmin, user, postgres, root, etc.).
Simulação de Password Spraying

Ao contrário da força bruta tradicional (que testa muitas senhas em um usuário), o Password Spraying testa uma única senha comum em múltiplos usuários, reduzindo o risco de bloqueio de conta (account lockout):

bash
medusa -h 192.168.56.101 -U usuarios.txt -p msfadmin -M smbnt
Resultado: Acesso confirmado para a conta msfadmin:msfadmin com permissão ao compartilhamento administrativo ADMIN$.
Validação do Acesso

Listagem dos recursos compartilhados no servidor Samba:

bash
smbclient -L //192.168.56.101 -U msfadmin

 4. Cenário 3: Análise de Autenticação Web no DVWA (Porta 80)

Análise prática realizada na aplicação DVWA (Damn Vulnerable Web Application):

Acesso à aplicação via navegador no Kali Linux (http://192.168.56.101/dvwa/).
Configuração do nível de segurança (DVWA Security) para Low.
Teste prático do formulário de autenticação:
Identificação do envio de credenciais em texto claro via método HTTP GET na URL (.../?username=admin&password=...).
Confirmação de ausência de mecanismos anti-automação (como CAPTCHA ou limitação de taxa de requisições).

1. Defesas Globais de Credenciais e Acesso
Eliminação de Credenciais Padrão (Default Credentials):
Nunca manter usuários e senhas de fábrica (como admin/admin ou msfadmin/msfadmin). Deve-se exigir a alteração da senha no primeiro login de qualquer sistema ou serviço instalado.
Política de Senhas Fortes:
Exigir senhas com comprimento mínimo (12 a 16 caracteres), combinando letras maiúsculas, minúsculas, números e caracteres especiais, além de bloquear senhas que constem em dicionários comuns.
Autenticação Multifator (MFA / 2FA):
Exigir uma segunda camada de validação (como código via app autenticador ou token de segurança físico), tornando testes automatizados de senhas ineficazes por si sós.
2. Defesas Específicas para Serviços de Rede (FTP e SMB)
Implementação de Bloqueio Automático (Rate Limiting e Fail2ban):
Configurar ferramentas de proteção como o Fail2ban. Ele analisa os logs de autenticação do sistema e, ao detectar um número pré-definido de tentativas falhas (por exemplo, 5 erros em 2 minutos), bloqueia automaticamente o endereço IP no firewall (iptables) por um período determinado.
Defesa contra Password Spraying:
Além de monitorar falhas por usuário, configurar regras no SIEM/IDS para alertar quando um mesmo endereço IP tentar autenticar em múltiplas contas diferentes dentro de uma janela de tempo curta.
Desativação de Protocolos Legados e em Texto Claro:
O FTP transmite comandos e senhas sem criptografia pela rede. A recomendação é desativar o FTP convencional (porta 21) e utilizar SFTP (criptografado via SSH na porta 22) ou FTPS (com certificados TLS).
Restrição de Acesso Anônimo no SMB:
Configurar o Samba (smb.conf) com a diretiva restrict anonymous = 2 para impedir que usuários anônimos (sessões nulas) consigam listar e enumerar contas de usuários locais.
3. Defesas para Aplicações Web (DVWA / Formulários de Login)
Uso Exclusivo de HTTPS e Método HTTP POST:
Credenciais jamais devem ser enviadas via método GET (que expõe dados sensíveis na URL, nos logs do servidor e no histórico de navegação). Deve-se sempre utilizar requisições seguras via POST sob canal criptografado com HTTPS (TLS).
Mecanismos Anti-Automação (CAPTCHA):
Inserir desafios como CAPTCHA / Cloudflare Turnstile após poucas tentativas consecutivas de login sem sucesso, inviabilizando ferramentas automatizadas de envio em massa de formulários.
WAF (Web Application Firewall):
Implementar um firewall de aplicação web (como ModSecurity) para detectar e frear rajadas de requisições enviadas ao endpoint de login.
