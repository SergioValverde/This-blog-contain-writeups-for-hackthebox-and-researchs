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

O hacer uso del binario certutil.exe, cuya utilidad conocida es la gestión de certificados, podemos hacer usos “no oficial” o inseguro de él, que es descargar ficheros. 

Tienes otros usos, como encodear fichero o decodearlos, puede sernos útil para la evasión de AV.


Os dejo el link para obtener más información:

[https://lolbas-project.github.io/lolbas/Binaries/Certutil/](https://lolbas-project.github.io/lolbas/Binaries/Certutil/)

Movemos el binario a la carpeta indicada anteriormente:

![image003.png]({{site.baseurl}}/images/PersistenciaLow1/image003.png)

Podemos revisarlo y ver como tenemos el fichero colocado en la ruta escrita anteriormente.

![image004.png]({{site.baseurl}}/images/PersistenciaLow1/image004.png)

El siguiente paso será configurar nuestro metasploit.

Añadimos un multi/handler y configuramos las opciones correspondientes

Pasos seguidos desde una sesión meterpreter:


![image005.png]({{site.baseurl}}/images/PersistenciaLow1/image005.png)

![image006.png]({{site.baseurl}}/images/PersistenciaLow1/image006.png)

![image007.png]({{site.baseurl}}/images/PersistenciaLow1/image007.png)

