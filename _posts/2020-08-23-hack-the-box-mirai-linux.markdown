---
title: Hack The Box - Mirai - Linux
date: 2020-08-23 10:39:00 Z
---

![Mirai.jpg]({{site.baseurl}}/images/Mirai/Mirai.jpg)

# Reconocimiento 

**NMAP**

```
nmap -sC -sV 10.10.10.48 -p- 
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-23 06:20 EDT
Nmap scan report for 10.10.10.48
Host is up (0.047s latency).
Not shown: 65529 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp    open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp    open  http    lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
1112/tcp  open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
32400/tcp open  http    Plex Media Server httpd
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
|_http-cors: HEAD GET POST PUT DELETE OPTIONS
|_http-title: Unauthorized
32469/tcp open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```


Accedemos al servidor web abierto en el puerto 80 para ver que información tiene alojada.

Nos encontramos con una página vacía.

![Server80.jpg]({{site.baseurl}}/images/Mirai/Server80.jpg)



En este punto, **volvemos** a los resultados revueltos por **nmap**, y vemos que tenemos abierto el **puerto** **53 - DNS**.
Conocimiendo el funcionamiento del servidor [**DNS**](https://www.cloudflare.com/learning/dns/what-is-dns/), vamos agregar la **IP-Hostname** a nuestro fichero **/etc/hosts**

De la siguiente manera:


![hosts.jpg]({{site.baseurl}}/images/Mirai/hosts.jpg)

De nuevo accedemos al servidor web, en lugar de usar la IP usaremos el hostname.


![miraiweb.jpg]({{site.baseurl}}/images/Mirai/miraiweb.jpg)

**Análisis**

Hemos obtenido el acceso a la aplicación Pi-Hole con versión 3.1.4

Intentamos buscar vulnerabilidades pero no encontramos ninguna.

Intentamos acceder al panel de administrador que descubrimos con gobuster.

![gobuster.jpg]({{site.baseurl}}/images/Mirai/gobuster.jpg)

Intentamos loguearnos con usuarios por defecto, como admin-admin, admin-user, user-user, etc.


![pihole.jpg]({{site.baseurl}}/images/Mirai/pihole.jpg)

Buscaremos usuarios por defecto para conectarnos.

**Explotación**

Buscando por la red usuarios por defecto para pi-hole, [descubrimos ](https://blog.cryptoaustralia.org.au/instructions-for-setting-up-pi-hole/)un usuario por defecto para acceder vía ssh a la raspberry.

```
username:pi
password:raspberry
```
Probaremos a conectarnos usando estás credenciales por defecto.

Y conseguimos acceder !

![ssh.jpg]({{site.baseurl}}/images/Mirai/ssh.jpg)


# PostExplotación

Una de las pricipales path para la escalada de privilegios es hacer uso de **sudo -l** que nos listará los posibles comandos que podremos recorrer.

![sudo-l.jpg]({{site.baseurl}}/images/Mirai/sudo-l.jpg)

Nos muestra que podemos realizar todo tipo de comandos sin necesidad de password.

Podemos hacer de sudo su y escalar privilegios

![root.jpg]({{site.baseurl}}/images/Mirai/root.jpg)

Se conseguió fácil, pero a la hora de conseguir la flag...

![flagroot.jpg]({{site.baseurl}}/images/Mirai/flagroot.jpg)

Navegamos por el directorio /dev/ el cuál contiene los archivos de los dispositivos hardware.

Podemos hacer uso del comando find para buscar posible información.

![find.jpg]({{site.baseurl}}/images/Mirai/find.jpg)

No vemos nada, seguimos rebuscando por la red y por los directorios, y nos encontramos una pista en el fichero **/dev/sdb**

![pista.jpg]({{site.baseurl}}/images/Mirai/pista.jpg)

Justo, encima del mensaje, tenemos las **flag** de **root**.

Siempre nos empeñamos en lo dificil, en mirar diferentes things, y lo importante lo tenemos delante...

END :)

