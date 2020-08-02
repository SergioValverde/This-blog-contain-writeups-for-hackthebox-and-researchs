---
title: Hack The Box - ÍNDICE - WINDOWS
date: 2020-08-02 20:46:00 Z
---

* [Legacy ](https://sergiovalverde.github.io/2020/07/28/hack-the-box-slash-legacy.html): Nos encontramos con los puertos **139,445,3389 abiertos** Realizamos una **análisis** con **NMAP**, con los **scripts ** **enum** and **vuln**
Encontramos dos vulnerabilidades en Samba, **MS08-067** y **MS17-010**.
Explotando MS08-067, conseguimos una shell, con una shellcode escrita en python.
Explotando MS17-010, encontramos en github un repositorio alojado en github, en el cuál usamos el script send-and-execute, crearemos una shellcode con msfvenom y la enviaremos.
No es necesario realizar una escalada de privilegios.