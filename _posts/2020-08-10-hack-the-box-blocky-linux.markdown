---
title: Hack The Box - Blocky - Linux
date: 2020-08-10 22:15:00 Z
---

![Blocky.jpg]({{site.baseurl}}/images/Blocky/Blocky.jpg)

Una vez comprobada la conectidad entres los dos equipos.

Empezaremos con el reconocimiento.

**Reconocimiento**

Lanzaremos la herramienta **NMAP** que nos devolvera los puertos que se encuentran abiertos en Blocky.

```
nmap -sC -sV 10.10.10.37 -p- -Pn

PORT      STATE  SERVICE   VERSION
21/tcp    open   ftp       ProFTPD 1.3.5a
22/tcp    open   ssh       OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp    open   http      Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.8
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: BlockyCraft &#8211; Under Construction!
8192/tcp  closed sophos
25565/tcp open   minecraft Minecraft 1.11.2 (Protocol: 127, Message: A Minecraft Server, Users: 0/20)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

**Mi** metodología será siempre comprobar todos los servicios antes de ponerme enfrente del servicio web, y dejarlo para el último paso. 

En este caso, no encontramos nada en el servicio FTP.

Así pasamos al servicio web.

El prmer paso sera ver que se esta alojando en la web.

![Wordpress.jpg]({{site.baseurl}}/images/Blocky/Wordpress.jpg)

Como podemos ver, se trata de un wordpress.

Intentaremos recabar más información, wordpress cuenta con un análisis de vulnerabilidades, llamado **wpscan**.

![wpscan.jpg]({{site.baseurl}}/images/Blocky/wpscan.jpg)

Es interesante, ya que **recopila información** de las **versiones** que se están corriendo, y los **themes instalados**, que pueden sernos utiles para ver si tienen posibles vulnerabilidades.

Seguimos recabando información, ahora con **gobuster**.

`gobuster dir -u http://10.10.10.37/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,sh,py,pl
`
Obtenemos los siguientes resultados, deberemos ir comprobando uno a uno para ver si guardan información que puede sernos útil.

![gobuster.jpg]({{site.baseurl}}/images/Blocky/gobuster.jpg)

En el directorio **Plugins**, nos encontramos alojados dos archivos

![plugins.jpg]({{site.baseurl}}/images/Blocky/plugins.jpg)

Nos los descargamos para ver si encontramos algo.

Una vez descargado, debemos descomprimirlo con `jar xf BlockyCore.jar`

Navegamos por la carpeta **com** y encontramos un fichero llamado **BlockyCore.class**

Para leer estos ficheros, usaremos el comandos **strings**.

![blockycore.jpg]({{site.baseurl}}/images/Blocky/blockycore.jpg)


# Análsis

Una vez recopilada la información, voy extraer los datos importantes sacados.

El dato más importante es el extraído del **fichero BlockyCore.class**, en el que podemos leer un usuario, root y una contraseña, 8YsqfCTnvxAUeduzjNSXe22.

Con estos datos, podemos intentar loguearnos a través de ssh, o acceder al panel **phpmyadmin** que nos recolecto **gobuster**.


![phpmyadmin.jpg]({{site.baseurl}}/images/Blocky/phpmyadmin.jpg)

Y accedemos **exitosamente** al panel.

![panelphpmy.jpg]({{site.baseurl}}/images/Blocky/panelphpmy.jpg)

# Explotación

Necesitaremos ver que usuarios hay en la tabla de **wordpress_users**

Y nos encontramos con el usuario **Notch**

![notch.jpg]({{site.baseurl}}/images/Blocky/notch.jpg)

Intentamos probar la password obtenida anteriormente, la cual pudimos acceder al panel de phpmyadmin, por si reutilizasen las mismas contraseñas en ambas cuentas.

Y acertamos.

![ssh.jpg]({{site.baseurl}}/images/Blocky/ssh.jpg)

# PostExplotación

La fase de escalada será bastante complicada.

Escribimos sudo su y la contraseña y ya estaría :)

Esto es debido a que nos encontramos con el comando /bin/su con permisos SUID.


![root.jpg]({{site.baseurl}}/images/Blocky/root.jpg)
