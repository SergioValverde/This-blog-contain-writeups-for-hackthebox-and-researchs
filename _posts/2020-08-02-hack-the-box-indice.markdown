---
title: Hack The BoX - ÍNDICE - LINUX
date: 2020-08-02 20:30:00 Z
---

* [**Lame**](https://sergiovalverde.github.io/2020/07/17/hack-the-box-lame.html) : Nos encontramos con los puertos **21,22,139,445,3632** abiertos. Enumeraremos los servicios samba con el fin de encontrar directorios.
Encontramos la **versión ** de **Samba 3.0.20** la cuál es **vulnerable ** y podremos ejecutar comandos con el **usuario Nohup**.
Por otro enfoque, encontramos que el puerto **3632 distccd v1** es vulnerable, y con nmap haremos esa comprobación.
En la fase de **post-explotación**, haremos uso de **nmap ** y su funcionalidad **--interactive**.
