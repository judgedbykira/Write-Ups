
# AguaDeMayo - Write-Up

## Enumeración

>Para empezar, deberemos realizarle un ping a la máquina para verificar si se encuentra encendida y para posiblemente deducir su SO mediante el TTL recibido, en este caso la máquina víctima es Linux debido a su TTL similar a 64:

```bash
ping -c 3 172.17.0.2
```
| Parámetro:    | Utilidad:                                           |
| ------------ | -------------------------------------------------- |
| -c           | Permite elegir la cantidad de solicitudes ICMP Echo Requests que serán enviadas.  |

![Pasted image 20240807151802](https://github.com/user-attachments/assets/da5393ad-d932-4746-8ccb-784e54860410)

>Una vez hecho esto, es hora de realizar un escaneo de puertos haciendo uso de la utilidad nmap. Para ello, comenzaremos analizando los puertos TCP de la máquina víctima:

```bash
nmap -p- --open -sS -vvv -Pn -n --min-rate 5000 172.17.0.2
```

| Parámetro:    | Utilidad:                                           |
| ------------ | -------------------------------------------------- |
| -p-           | Escanear todos los puertos de la máquina (65535).  |
| --open   | Mostrar en el resultado solo los puertos abiertos.               |
| --min-rate | Solo enviar paquetes que vayan a una velocidad de x paquetes por segundo. | 
|-sS | Realizar un stealth scan que no concluye el three-way handshake del protocolo TCP enviando un RST en lugar de un ACK. (Agiliza el escaneo)
-vvv | Muestra muy detalladamente el resultado del comando.
-Pn | Dejamos de usar el protocolo de resolución de direcciones ARP para el escaneo. (Agiliza el escaneo)
-n | Dejamos de usar el servicio DNS para resolver la dirección. (Agiliza el escaneo)

>Aquí podemos ver dos puertos interesantes que pueden servirnos como vector entrada en la máquina víctima, el 22 correspondiente al servicio SSH y el 80 correspondiente al servicio HTTP.

![Pasted image 20240807151856](https://github.com/user-attachments/assets/5cd135d2-f199-4fba-84d2-0b51c0797e3b)


>Ahora podríamos realizar un escaneo de versión y con scripts de reconocimiento para obtener información interesante acerca de los servicios que corren en ambos puertos abiertos:

```bash
nmap -p22,80 -sCV -Pn -n -vvv 172.17.0.2
```

| Parámetro:    | Utilidad:                                           |
| ------------ | -------------------------------------------------- |
| -px           | Escanea solo los puertos o el rango especificado en lugar de la x.  |
| -sCV   | Realizar un escaneo de versión y usar scripts de reconocimiento propios de la utilidad.               |

![Pasted image 20240807152154](https://github.com/user-attachments/assets/b6fd27b7-47c5-41f4-9645-9cbe9d1085b5)


>Mediante la versión del servicio SSH podríamos obtener información acerca del SO de la máquina víctima. La máquina esta empleando una distribución de Linux llamada Debian.

>Por otra parte, ya que existe un servicio HTTP vamos a tratar de realizar fuzzing para detectar posibles directorios existentes dentro de la máquina víctima realizando fuerza bruta mediante un diccionario con ayuda de la herramienta gobuster:

```bash
gobuster dir -u http://172.17.0.2 -t 40 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
```

| Parámetro:    | Utilidad:                                           |
| ------------ | -------------------------------------------------- |
| dir           | Permite fuzzear directorios.  |
-u | Permite especificar la URL en la que se realizará el fuzzing.
-t | Permite fijar la cantidad de hilos empleados para el proceso de fuzzing.
-w | Permite especificar el diccionario que se empleará para el fuzzing.

![Pasted image 20240807152658](https://github.com/user-attachments/assets/04e07eaa-02c5-4810-b1b8-52a5090182e6)


>Aquí podemos destacar que existe un directorio llamado images pero previamente a ello vamos a acceder a la página principal:

![Pasted image 20240807152751](https://github.com/user-attachments/assets/97496a1b-983b-421a-b1d7-6f0ecc16c6e2)


>Aquí podremos ver que emplea el index.html por defecto de apache2 pero si revisamos su código fuente podremos encontrar un comentario oculto que contiene una cadena codificada en Brainfuck:

![Pasted image 20240807152917](https://github.com/user-attachments/assets/2f13e31b-ffd2-457d-92ab-f07d7dc9e392)


>Si tratamos de decodificarla mediante una herramienta web podremos obtener el siguiente mensaje:

![Pasted image 20240807152957](https://github.com/user-attachments/assets/96fd71c0-a2ef-4b4f-ae74-052a5e1c7f7c)


>Esta cadena podría corresponder a una credencial o usuario del otro servicio corriendo, que recordemos que era el SSH.

>Ahora trataremos de ver qué contenido hay en el directorio images:

![Pasted image 20240807153118](https://github.com/user-attachments/assets/403bc567-e607-47ee-9222-2d2ca4be6a94)


>Aquí podemos ver una imagen llamada **agua_ssh** la cual puede contrastar nuestra suposición acerca del mensaje decodificado anteriormente, por lo que, probaremos a entrar mediante SSH a la máquina empleando como usuario **agua** y contraseña **bebeaguaqueessano**.

![Pasted image 20240807153347](https://github.com/user-attachments/assets/cfd1dc0b-ba07-4fbd-a519-48b33a5a5a8e)


>Y así de sencillo ya nos encontraríamos dentro de la máquina víctima. Ahora habría que realizar una escalada de privilegios para convertirnos en el usuario root del sistema.

## Escalada de privilegios

>Probaremos a ver qué permisos de sudo tenemos como usuario agua:

```bash
sudo -l
```

| Parámetro:    | Utilidad:                                           |
| ------------ | -------------------------------------------------- |
| -l           | Permite ver los permisos de sudo que tiene el usuario que lo lanza.  |

>Como podemos ver podemos emplear el comando bettercap sin proporcionar contraseña:

![Pasted image 20240807153659](https://github.com/user-attachments/assets/b8720bef-c4ed-4693-bd10-c76b5931bd80)


>Mediante el binario bettercap podemos ejecutar comandos al empezarlos con una "!", esto teniendo en cuenta que el binario está siendo ejecutado como root, nos permitirá ejecutar código como si fuéramos root:

```bash
sudo /usr/bin/bettercap
!whoami
```

![Pasted image 20240807154216](https://github.com/user-attachments/assets/0434dcb7-ddbf-4026-9c30-be7e2cb1b435)


>Podemos ver que al emplear el comando whoami el programa nos devuelve root, así que cambiaremos los permisos de la bash para que sean SUID y poder darnos una bash privilegiada:

```bash
!chmod +s /bin/bash
```

| Parámetro:    | Utilidad:                                           |
| ------------ | -------------------------------------------------- |
| +s           | Permite agregar el permiso SUID al binario. (4000)  |

>Una vez hecho esto, bastará con spawnear una bash privilegiada mediante el siguiente comando y ya habremos comprometido por completo la máquina convirtiéndonos en el usuario root gracias al permiso SUID que le hemos agregado a la bash:

```bash
bash -p
```

| Parámetro:    | Utilidad:                                           |
| ------------ | -------------------------------------------------- |
| -p           | Lanza una bash de forma privilegiada.  |

![Pasted image 20240807154525](https://github.com/user-attachments/assets/2c0bed01-ca2e-42c6-9dab-e8534554fd49)


