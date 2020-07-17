---
title: Payload II - EMOTET
date: 2020-02-20 00:00:00 Z
permalink: "/PayloadII/"
layout: post
---

### Payload II – Special Emotet

Debido a la curiosidad recibida por las campañas Emotet, realizaré una investigación para conocer más sobre este asunto.

El artículo lo voy a enfocar a los ataques por apertura de documento, tales como envíos de correo con documentos de la suite de office.

Esto no es creación de un Emotet, pero si os servirá para entender el funcionamiento y la simplicidad del asunto.

-----------------------

## Emotet 

Cito a nuestra amiga Wikipedia: _Emotet es un malware troyano polimórfico (cambia automáticamente su código cada cierto tiempo o con acciones determinadas del dispositivo), haciendo que sea más difícil para los antivirus detectar su firma._

Características más importantes:

-	Hablamos de un malware modular, adaptación y personalización del entorno del que se encuentre. (Empresas, objetivos individuales..) 

-	Capaz de distribuir troyanos bancarios, ramsonware, robo de información.

-	Cuenta con mecanismos avanzados de persistencia.

-	Capaz de detectar entornos de laboratorios, máquinas virtuales y generar indicadores falsos.

-	Los modos principales de propagación son por Spam y mensajes de correo. 

-	Emotet construye nuevos mensajes de ataque en respuesta a algunos de los mensajes de correo electrónico no leídos de esa víctima, citando los cuerpos de mensajes reales en los hilos.

-	En constante actualización.

### Modo de infección

Una vez recibido un correo de emotet, si no lo habeís visto, revisar vuestra carpeta de spam de vuestro correo.

Va a contener archivos de la suite office, como, por ejemplo .doc, y un mensaje sobre información que podamos considerar importante, como nóminas, mensajes de jefaso o cualquier acontecimiento que resulte importante. 

Estos words contienen código VBA ofuscando dentro del Word, en concreto en un .doc, formato word del año 97 al 2003, el cuál ejecuta un powershell en segundo plano.
Y este descargará malware del wordpress vulnerado.

### Creación del documento Word

Creación de macros .vba através del programa ps1encode:

![ps1.jpg]({{site.baseurl}}/images/Payload2/ps1.jpg)

Nos devolverá una serie de código que deberemos inyectar en el Word.

Abrimos un nuevo documento Word, es necesario añadir el apartado Desarrollador:

![Desarrollador.jpg]({{site.baseurl}}/images/Payload2/Desarrollador.jpg)

Hacemos clic en Visual Basic.

![vba.jpg]({{site.baseurl}}/images/Payload2/vba.jpg)

Observamos varios Project, a nosotros en el que nos interesa en el Documento1.

Doble clic sobre ThisDocument, y modificaremos el apartado General a Document, Declaraciones a open.

Borramos las instrucciones y añadimos el código VBA.

Y quedaría tal que así:

![macros.jpg]({{site.baseurl}}/images/Payload2/macros.jpg)

*Una vez añadas el código, se modificará los campos General – Workbook_Open automáticamente.*

El código para entenderlo sería stringA + stringB ¿A que ahora sí? jeje.

El siguiente paso, es escribir un mensaje que provoque a la víctima habilitar esas macros a través de la ingeniería social.

Conociendo nuestro objetivo podremos usar esa información para enfocar nuestro mensaje.

Como por ejemplo situación actual de la empresa, logotipos...

Mi ejemplo y un ejemplo que he visto en los Emotet es algo tan simple como esto:

![word.jpg]({{site.baseurl}}/images/Payload2/word.jpg)

Debemos guardar este documento con la extensión .doc

Emotet -> [https://any.run/malware-trends/emotet](https://any.run/malware-trends/emotet)

Entendemos que el documento ha sido enviado a la víctima.

El siguiente paso es abrir metasploit.

![metasploit.jpg]({{site.baseurl}}/images/Payload2/metasploit.jpg)

Una vez se habilite las macros nos devolverá un meterpreter:

![sysinfo.jpg]({{site.baseurl}}/images/Payload2/sysinfo.jpg)

Una vez conseguido el acceso, empieza la fase de post-explotación, en donde buscaremos expandirnos por la red, explotando vulnerabilidades, o esperándolas.


### Material Interesante
Análisis:

[https://blog.malwarebytes.com/botnets/2019/09/emotet-is-back-botnet-springs-back-to-life-with-new-spam-campaign/](https://blog.malwarebytes.com/botnets/2019/09/emotet-is-back-botnet-springs-back-to-life-with-new-spam-campaign/)

[https://security-soup.net/analysis-of-a-new-emotet-maldoc-with-vba-downloader/](https://security-soup.net/analysis-of-a-new-emotet-maldoc-with-vba-downloader/)

[https://atr-blog.gigamon.com/2020/01/13/emotet-not-your-run-of-the-mill-malware/](https://atr-blog.gigamon.com/2020/01/13/emotet-not-your-run-of-the-mill-malware/)

Reversing -> [https://www.youtube.com/watch?v=cjlctph9cZE](https://www.youtube.com/watch?v=cjlctph9cZE)
