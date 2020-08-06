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

Nos hace referencia a [este](https://github.com/Arrexel/phpbash) repositorio.

```
phpbash is a standalone, **semi-interactive web shell**. 
It's main purpose is to assist in penetration tests where traditional reverse shells are not possible. 
The design is based on the default Kali Linux terminal colors, so pentesters should feel right at home.
```

Recogeremos más información con la herramienta **gobuster**.

![gobuster.jpg]({{site.baseurl}}/images/Bashed/gobuster.jpg)

Podemos observar multiples **directorios** que son **accesibles**, reciben los códigos:
* 200, los códigos que empiezan por la cifra 2, son **respuestas satisfactorias**.
* 301, los códigos que empiezan por la cifra 3, son **reedirecciones **.

Podemos leer y aprender más sobre los códigos webs a través de está [página](https://developer.mozilla.org/es/docs/Web/HTTP/Status).
