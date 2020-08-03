---
title: Hack The Box - BEEP - Linux
date: 2020-08-03 09:42:00 Z
---

![Beep.jpg]({{site.baseurl}}/images/Beep/Beep.jpg)

El primer paso será comprobar la conectividad entre máquinas.

![PING.jpg]({{site.baseurl}}/images/Beep/PING.jpg)

# RECONOCIMIENTO

Lanzaremos la herramienta **NMAP**

```
nmap -sC -sV 10.10.10.7 -p- 
[sudo] password for kali: 
Sorry, try again.
[sudo] password for kali: 
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-03 05:10 EDT
Nmap scan report for 10.10.10.7
Host is up (0.047s latency).
Not shown: 65519 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: UIDL USER IMPLEMENTATION(Cyrus POP3 server v2) LOGIN-DELAY(0) STLS AUTH-RESP-CODE RESP-CODES EXPIRE(NEVER) TOP PIPELINING APOP
111/tcp   open  rpcbind    2 (RPC #100000)
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: CHILDREN THREAD=ORDEREDSUBJECT Completed OK URLAUTHA0001 ID X-NETSCAPE LIST-SUBSCRIBED CATENATE IMAP4rev1 STARTTLS SORT CONDSTORE LISTEXT LITERAL+ UIDPLUS MAILBOX-REFERRALS ANNOTATEMORE SORT=MODSEQ THREAD=REFERENCES ACL BINARY NO MULTIAPPEND IMAP4 UNSELECT ATOMIC RIGHTS=kxte IDLE RENAME NAMESPACE QUOTA
443/tcp   open  ssl/https?
|_ssl-date: 2020-08-03T09:18:15+00:00; +3m53s from scanner time.
878/tcp   open  status     1 (RPC #100024)
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
4190/tcp  open  sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4 (included w/cyrus imap)
4445/tcp  open  upnotifyp?
4559/tcp  open  hylafax    HylaFAX 4.3.10
5038/tcp  open  asterisk   Asterisk Call Manager 1.1
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com, localhost; OS: Unix
```

Accedemos vía nagevador y nos encontramos con el portal Elastix.

![elastix.jpg]({{site.baseurl}}/images/Beep/elastix.jpg)

Buscamos posibles exploits con **searchsploit**

![searchsploitelastix.jpg]({{site.baseurl}}/images/Beep/searchsploitelastix.jpg)

Desconocemos que versión está instalada, pero probaremos todos por si nos alguno de utilidad.

Antes de tirar por este puerto, estuve buscando y rebuscando información que puediese sacar de los demás puertos.
Lanzando exploits pero necesitaba información que no tenía en un principio.

Otro puerto que me llamo la atención es el puerto **10000** que corre el servicio **Webmin**
Intentado hacer fuerza bruta con hydra pero a los 5 intentos nos metía en una **blacklist**.

**Enumeración**

Encontramos un [Local File Inclusion](https://www.exploit-db.com/exploits/37637)

El código que introduciremos vía web es este:


`/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action`

Nos aparecerá está página, hacemos **control + u** para ver mejor el texto.
![LF1.jpg]({{site.baseurl}}/images/Beep/LF1.jpg)

En este texto, encontramos el usuario **admin** y su **password**, con la podremos entrar en en **panel de administrador de Elaxtix**

![dashboardelastix.jpg]({{site.baseurl}}/images/Beep/dashboardelastix.jpg)

# Explotación

**Solución 1**

Con las claves de usuario podremos acceder vía ssh como root

![sshroot.jpg]({{site.baseurl}}/images/Beep/sshroot.jpg)

¿Fácil no?

**Solución 2**

Una vez entramos por el panel de Elastix, nos dirigimos al apartado **PBX** , en la columna de la izquierda, hay un último apartado.

![freePBX.jpg]({{site.baseurl}}/images/Beep/freePBX.jpg)

Y accederemos al panel de configuración de FreePBX

![panelFreePBX.jpg]({{site.baseurl}}/images/Beep/panelFreePBX.jpg)

Hacemos clic en el apartado **Tools**

![toolsfreePBX.jpg]({{site.baseurl}}/images/Beep/toolsfreePBX.jpg)

Y nos vamos al apartado **Module Admin**

![moduleadmin.jpg]({{site.baseurl}}/images/Beep/moduleadmin.jpg)

En este apartado subiremos un modulo, donde alojaremos una reverse shell para realizar la conexión entre Beep y mi máquina.


Podemos hacer uso de este repositorio de [github](https://github.com/SamSepiolProxy/FreePBX-Reverse-Shell-Module), nos lo descargamos y **modificamos los parámetros de IP y puerto del fichero install.php**, indicando nuestra IP y el puerto que vamos a abrir.

Y por último, para poder subir el paquete correctamente, debemos comprimir el paquete, indicamos el siguiente nombre, **nombre-versión.tar.gz**
Con el comando

`tar -cvzf shell-1.0.tar.gz shell`

Hacemos clic en **Upload Module**, seleccionamos el paquete comprimido que hemos creado, y confirmamos su subida.

Una vez súbido, debemos instalarlo, seleccionamos el paquete, y hacemos clic en **Process**.

![shellinstall.jpg]({{site.baseurl}}/images/Beep/shellinstall.jpg)

En este momento, **levantamos** con la herramienta **netcat**, el puerto que hemos introducido anteriormente.

![netcat.jpg]({{site.baseurl}}/images/Beep/netcat.jpg)




