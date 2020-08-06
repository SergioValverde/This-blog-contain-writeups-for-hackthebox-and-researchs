---
title: Hack The Box - Bashed
date: 2020-08-06 07:49:00 Z
---

Siguiente la antesala de los anteriores posts, hoy seguimos con **BASHED**

![Bashed.jpg]({{site.baseurl}}/images/Bashed/Bashed.jpg)

El primer paso, como siempre, comprobar si tenemos conectividad entre nuestro equipo y "víctima".

![Ping.jpg]({{site.baseurl}}/images/Bashed/Ping.jpg)

Gracias al [TTL](https://es.wikipedia.org/wiki/Tiempo_de_vida_(inform%C3%A1tica)) reconocemos que la máquina que vamos a auditar es una máquina **Linux**.

# Reconocimiento

Como siempre, ejecutamos la herramienta **NMAP**

`-A: Enable OS detection, version detection, script scanning, and traceroute`

`T5: Acelera el proceso, podemos indicar del 1 al 5, siendo 5 el más rapido.`

```
nmap -A -T5 10.10.10.68 -p-                                                                              
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-06 03:51 EDT                                                            
Warning: 10.10.10.68 giving up on port because retransmission cap hit (2).                                                 
Nmap scan report for 10.10.10.68                                                                                           
Host is up (0.035s latency).                                                                                               
Not shown: 57774 closed ports, 7759 filtered ports           
PORT     STATE SERVICE    VERSION                 
80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)                
|_http-title: Arrexel's Development Site                    
4480/tcp open  tcpwrapped                                    
                                                             
```

