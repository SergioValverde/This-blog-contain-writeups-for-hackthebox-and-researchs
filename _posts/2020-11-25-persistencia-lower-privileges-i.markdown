---
title: Persistencia - Lower Privileges I
date: 2020-11-25 12:48:00 Z
---

# ¿Quien está ahí?

Definimos persistencia como el proceso de configurar puertas traseras, con el objetivo de mantener el acceso en el equipo. ¿Fácil no? 

Este tutorial intentaré cubrir las técnicas más utilizados para máquinas Windows con privilegios de usuario.

Interesante conocer este concepto debido a que es útil para todos, tanto red team como blue team.

# ¿Qué objetivos tenemos?

* Sobrevivir a un reinicio
* Caída de la red
* Ser detectados 
* Eliminación del vector de acceso




# Startup Folder

Una de las formulas más conocidas, cualquier binario, script que este alojado en este directorio será ejecuta cuando inicie sesión un usuario.

The path of the startup folder is: 
**C:\Users\%username%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup**

Es importante conocer, que la ejecución se produce bajo el usuario logeado.

Si tenemos **permisos administrador**, nosotros podemos colocar nuestra binario en la siguiente ruta:

**C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup**

De esta manera el payload cargara cuando, cualquier usuario se logge, no importa quien, por un usuario normal o por un usuario administrador.

Más detalles:
[https://attack.mitre.org/techniques/T1547/001/](https://attack.mitre.org/techniques/T1547/001/)

Vamos allá, creamos un binario con msfvenom:

![image001.png]({{site.baseurl}}/images/PersistenciaLow1/image001.png)

Lo subimos al equipo, tenemos diferentes opciones, si nos encontramos con una sesión meterpreter, podemos utilizar el comando upload:
![image002.png]({{site.baseurl}}/images/PersistenciaLow1/image002.png)