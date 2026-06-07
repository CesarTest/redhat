---
title: UDEV: Plug&Play
taxonomy:
    category: docs
---
<style>
   .demonio { 
	   table, tbody, tr, td { padding: 0; height: 0; cellpadding: 0; cellspacing: 0; }
       p, code { padding: 0; height: 0; }
	   img     { padding: 0; margin-bottom: 0; margin-top: 0; }
    }
</style>

#  El Administrador de Dispositivos.

## ¿Para qué? Plug&Play del Hardware

** UDEVD es el administrador de dispositivos de Linux**, a través de un sistema de reglas automatiza la inserción de los dispositivos hardware al núcleo del sistema operativo para que los programas puedan usar ese recurso hardware a través de la interfaz de llamadas al sistema. La automatización plug&play opera de la siguente manera: 
+ Se configuran <u>reglas</u> que van a activarse <u>después</u> de determinados <u>eventos</u> (ejm.: enchufar un USB). 
+ Estas reglas automatizan la creación de todo lo necesario para que el kernel pueda gestionar el dispositivo hardware: 
   - <u>Métodos Syscall</u>: cargar el módulo del kernel.
   - <u>Propiedades Dispositivo</u>: modelo de datos en Sysfs (/sys). 
   - <u>Relación Métodos con propiedades</u>: fichero de nodo (/dev).

En la imagen se aprecia las acciones que pueden programarse en cada regla.

![UDEV](image://teoria/teoria-entorno-udev.jpg)

## Anatomía
[div class="table table-striped demonio"]
 ICONO                                                   | RUTA		                                   
---------------------------------------------------------|---------------------------------------------
![EXE](image://teoria/tabla-demonio-exe.png?classes=float-left,demonio&resize=100,100) | <p><b>Ejecutable del Servidor</b></p><p></br><code>/usr/lib/systemd/systemd-udevd</code></p>     
![ETC](image://teoria/tabla-demonio-cnf.png?classes=float-left,demonio&resize=100,100) | <p><b>Configuraciones del Servidor</b></p> <p> <code>/etc/udev</code></p><p> <code>/etc/udev/rules.d</code></p>
![LIB](image://teoria/tabla-demonio-inmutable.svg?classes=float-left,demonio&resize=100,100) | <p><b>Código fuente y documentación del Servidor</b></p><p><code>/lib/udev</code></p>    
![VAR](image://teoria/tabla-demonio-var.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento persistente del Servidor</b></p><p><code>/usr/lib/udev</code></p><p><code>/sys/class</code></p>
![RUN](image://teoria/tabla-demonio-run.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento volátil del Servidor</b> (desaparece tras cada reinicio)</p><p><code>/run/udev</code></p>
![PROC](image://teoria/tabla-demonio-ram.png?classes=float-left,demonio&resize=100,100) | <p><b>Disco virtual que el kernel crea en RAM</b></p><p><code>/proc/1</code></p>
![IPC](image://teoria/tabla-demonio-ipc.svg?classes=float-left,demonio&resize=100,100) | <p><b>Sockets InterProcess Communication </b></p><pre class="language-bash"><code>ss -xpln &#124; grep udev</code></pre></p>
![NET](image://teoria/tabla-demonio-tcp.png?classes=float-left,demonio&resize=100,100) | <p><b>Sockets TCP/UDP </b></p><p><pre class="language-bash"><code>ss -tupln &#124; grep udev</code></pre></p>
[/div]

## Configuración
[div class="table table-striped demonio"]
FICHERO                                                                        |  APLICAR CAMBIOS                                                             | DESCRIPCIÓN		                                   
-------------------------------------------------------------------------------|-------------------------------------------------------------------------------
<p><code>/etc/udev/udev.conf</code></p><p>Ayuda: <code>man udev.conf</code></p>| <pre class="language-bash"><code>systemctl daemon-reload</code></pre>        | Las opciones más importantes son las siguientes: <ul><li><b>udev_log:</b> nivel de trazado (err, info, debug).</li></ul>
<p><code>/etc/udev/rules.d</code></p><p>Ayuda: <code>man udev</code></p>       | <pre class="language-bash"><code>systemctl restart systemd-udevd</code></pre>| Conjunto de reglas personalizables agregar tratamientos específicos.
<p><code>/usr/lib/udev/rules.d</code></p><p>Ayuda: <code>man udev</code></p>   | <pre class="language-bash"><code>systemctl restart systemd-udevd</code></pre>| Conjunto de reglas que vienen de fábrica.
[/div]

## Herramientas de Trabajo
[div class="table table-striped demonio"]
COMANDO                   |  OBJETIVO                                     | DESCRIPCIÓN	                                   
--------------------------|-------------------------------------------------------------------------------
<code>/sbin/udevadm</code>| Gestión de reglas y monitorización de eventos | La herramienta udevadm permite controlar reglas sobre qué debe hacerse cuando se conectan dispositivos nuevos <ul><li><b>info:</b> Query sysfs or the udev database</li><li><b>trigger:</b> Request events from the kernel</li> <li><b>settle:</b> Wait for pending udev events</li> <li><b>control:</b> Control the udev daemon</li>  <li><b>monitor:</b> Listen to kernel and udev events</li> <li><b>test:</b> Test an event run</li> <li><b>test-builtin:</b> Test a built-in command</li></ul>
[/div]

# Casos de Uso Relevantes

## CASO 1: Asignación Persistente Nombres a Interfaces de Red

### Descripción
Un caso de uso importante es el renombrado de las interfaces de red. Durantes el reconocimiento del hardware que el kernel hace antes de llamar al demonio de sistema asigna nombres <code>ethX</code> a las interfaces de red. La asignación del número X de la interfaz de red es aleatoria, si hay algún cambio en el hardware, pueden cambiarse el número que asigna el kernel a una determinada boca de red.

Para eliminar esta arbitrariedad en el nombrado de las interfaces, se han creado un conjunto de reglas UDEV que asignan un nombre en función del tipo de hardware y su ubicación en el chasis de la máquina, de esta manera los nombres se asignan de manera persistente a las interfaces de red.

### Ficheros de Reglas

Las reglas realizan el renombrado siguen la siguiente secuencia de ficheros:

1. <b>Opcional: <code>/usr/lib/udev/rules.d/60-net.rules</code> define que reglas de compatibilidad</b>, usa la utilidad auxiliar obsoleta /usr/lib/udev/rename_device para buscar el parámetro HWADDR en/etc/sysconfig/network-scripts/ifcfg-* archivos. Si el valor establecido en la variable coincide con la dirección MAC de una interfaz, la utilidad auxiliar cambia el nombre de la interfaz al nombre establecido en el DEVICE (parámetro del archivo ifcfg). Si el sistema utiliza sólo perfiles de conexión de NetworkManager en formato de archivo de claves, udev omite esto paso.
1. <b>Solo en sistemas Dell: <code>/usr/lib/udev/rules.d/71-biosdevname.rules</code>.</b> Este archivo existe sólo si el paquete biosdevname está instalado y el archivo de reglas define que la utilidad biosdevname cambia el nombre de la interfaz de acuerdo con su política de nomenclatura, si no se le cambió el nombre en el paso anterior. 
1. <b><code>/usr/lib/udev/rules.d/75-net-description.rules</code>, Este archivo define cómo udev examina la interfaz de red y establece las propiedades en las variables internas de udev.</b> Estas variables luego son procesadas en el siguiente paso por el /usr/lib/udev/rules.d/80-net-setup-link.rules. Algunas de las propiedades pueden no estar definidas.
1. <b><code>/usr/lib/udev/rules.d/80-net-setup-link.rules</code>, Este archivo llama al sistema integrado net_setup_link del servicio udev y udev cambia el nombre de la interfaz según el orden de las políticas en el parámetro NamePolicy</b> en el <code>/usr/lib/systemd/network/99-default.link/code> archivo. Para obtener más detalles, consulte políticas de nomenclatura  de la Interfaz de red.

### Lógica de Asignación de Nombres

1. **Firmware:** prefijo que identifica el tipo de interfaz de red.
1. **Tipo, Localización o Topología:** a continuación, el puerto PCI de chasis.  
1. **Índice, ID o Puerto:** finalmente, el sufijo que identifica la boca dentro de la interfaz de red.

![UDEV](image://teoria/teoria-entorno-udev-nic.jpg)
