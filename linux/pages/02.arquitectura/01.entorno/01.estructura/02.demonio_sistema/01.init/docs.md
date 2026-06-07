---
title: Init
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

# Estructura: Disparador de Demonios.

**DEF.: Init es un <u>sistema de inicio y administración de servicios</u> para sistemas operativos Unix.** Es el primer proceso en ejecución tras la carga del núcleo y el que a su vez genera todos los demás procesos. Se ejecuta como demonio y por lo general tiene PID 1. 

[prism classes="language-bash"] 
root@localhost > $ pstree
init─┬─NetworkManager─┬─dhclient
     │                └─{NetworkManager}
     ├─VBoxService───7*[{VBoxService}]
     ├─auditd───{auditd}
     ├─console-kit-dae───63*[{console-kit-da}]
     ├─crond
     ├─dbus-daemon───{dbus-daemon}
     ├─haveged
     ├─login───bash───sudo───bash
     ├─master─┬─pickup
     │        └─qmgr
     ├─5*[mingetty]
     ├─modem-manager
     ├─polkitd
     ├─rsyslogd───3*[{rsyslogd}]
     ├─sshd─┬─sshd───sshd───bash───pstree
     │      └─sshd───sshd───sftp-server
     ├─tuned
     ├─udevd───2*[udevd]
     └─wpa_supplicant
[/prism]


**DEF.: Base de Datos de Unidades de Ejecución:** Init mantiene una base de datos de scripts que controlan  cada demonio del entorno de ejecución. 

**DEF.: Unidad de Ejecución:** Cada unidad de ejecución es un script bash que lanza un demonio, se almacena <code>/etc/rc.d/init.d</code>. Para activar una unidad de ejecución durante el arranque del sistema, ha de crearse un enlace simbólico en la ruta del nivel de ejecución en al que quiere agregarse (<code>/etc/rc.d/rcX.d</code>).

![Init](image://teoria/teoria-entorno-init.jpg)

## ¿Para qué? La Inicialización del Entorno de Ejecución.

Tradicionalmente, la inicialización del entorno de ejecución se ha implementado de forma distinta en los dos grandes sistemas operativos: System V y BSD. En el Unix original, el proceso Init arrancaba los servicios mediante un único script denominado <code>/etc/rc</code>, siguiendo las instrucciones de la tabla de inicialización (<code>/etc/inittab</code>). Posteriormente, la versión System V del Unix de AT&T introdujo un nuevo esquema de directorios en <code>/etc/rc.d/</code> que contenía scripts de arranque/parada de servicios. Linux adoptó el esquema System V de Init que se presenta aquí.

### Propiedades
+ **Secuencial:**  nivel de ejecución a nivel de ejecución, demonio a demonio. A la hora de diseñar la secuencia de arranque el programador debe tener en cuenta las dependencias entre demonios.
+ **Interoperable:** para mantener la interoperabilidad de los demonios entre todos los sistemas Unix, el demonio Init no realiza ningún control sobre el entorno de ejecución. Dicho control es reppnsabilidad de supervisores de aplicación que cada desarrollo debe impplementar para sus productos.
 

### La Unidad de Ejecución

Cada Unit es un script bash que contiene información sobre cómo gestionar un demonio. En los comentarios de estos scripts entre <code>### BEGIN INIT INFO</code> y <code>### END INIT INFO</code>  se indica las dependencias de la Unidad de Ejecución que condiciona su orden de lanzamiento.

[prism classes="language-bash"] 
root@localhost > cat /etc/rc.d/init.d/sysstat	 
#!/bin/sh
#
# chkconfig: 12345 01 99
# description: Reset the system activity logs
#
# /etc/rc.d/init.d/sysstat
# (C) 2000-2009 Sebastien Godard (sysstat <at> orange.fr)
#
### BEGIN INIT INFO
# Provides:             sysstat
# Required-Start:
# Required-Stop:
# Default-Start: 1 2 3 4 5
# Default-Stop: 0 6
# Description: Reset the system activity logs
# Short-Description: reset the system activity logs
### END INIT INFO
#@(#) sysstat-9.0.4 startup script:
#@(#)    Insert a dummy record in current daily data file.
#@(#)    This indicates that the counters have restarted from 0.

RETVAL=0
PIDFILE=/var/run/sysstat.pid

# See how we were called.
case "$1" in
  start)
        [ $UID -eq 0 ] || exit 4
        echo $$ > $PIDFILE || exit 1
        echo -n "Calling the system activity data collector (sadc)... "
          /usr/lib64/sa/sa1 --boot
        [ $? -eq 0 ] || RETVAL=1
        rm -f $PIDFILE
        echo
        ;;

  status)
        [ -f $PIDFILE ] || RETVAL=3
        ;;

  stop)
        [ $UID -eq 0 ] || exit 4
        ;;

  restart|reload|force-reload|condrestart|try-restart)
        ;;

  *)
        echo "Usage: sysstat {start|stop|status|restart|reload|force-reload|condrestart|try-restart}"
        RETVAL=2
esac

exit ${RETVAL}
ism]


La base de datos de Unidades de Ejecución en <code>/etc/rc.d/init.d</code> tiene este aspecto:

[prism classes="language-bash"]
root@localhost > tree /etc/rc.d/init.d
/etc/rc.d
├── init.d
│ ├── auditd
│ ├── crond
│ ├── halt
│ ├── ip6tables
│ ├── iptables
│ ├── NetworkManager
│ ├── rsyslog
│ ├── sshd
│ ├── udev-post 
[/prism]


### La Estructura del Arranque

La estructura del arranque en Init se define en <code>/etc/rc.d/rc{0-6}.d</code> donde aparecen enlaces simbólicos a las unidades de ejecución que han de lanzarse en ese nivel de ejecución durante el proceso de arranque del sistema.

Aparecen una serie de prefijos en estos enlaces simbólicos que indican en qué orden han de levantarse los demonios dentro de cada nivel de ejecución:

[div class="table table-striped demonio"]
|  LETRA                                                                 | NUMERO
|------------------------------------------------------------------------|----------
<p><code>S</code> = Start scripts</p><p><code>K</code> = Kill scripts</p>|  Orden de Arranque, según comentarios de la Unidad de Ejecución
[/div]

[prism classes="language-bash"]
> tree /etc/rc.d
├── rc0.d
│   ├── K10saslauthd -> ../init.d/saslauthd
│   ├── K15tuned -> ../init.d/tuned
│   ├── K25haveged -> ../init.d/haveged
│   ├── [...]
│   └── S01halt -> ../init.d/halt
├── rc1.d
│   ├── K10saslauthd -> ../init.d/saslauthd
│   ├── K15tuned -> ../init.d/tuned
│   ├── K25haveged -> ../init.d/haveged
│   ├── [...]
│   └── S99single -> ../init.d/single
├── rc2.d
│   ├── K10saslauthd -> ../init.d/saslauthd
│   ├── K50dnsmasq -> ../init.d/dnsmasq
│   ├── [...]
│   └── S99local -> ../rc.local
├── rc3.d
│   ├── K10saslauthd -> ../init.d/saslauthd
│   ├── K50dnsmasq -> ../init.d/dnsmasq
│   ├── [...]
│   └── S99local -> ../rc.local
├── rc4.d
│   ├── K10saslauthd -> ../init.d/saslauthd
│   ├── K50dnsmasq -> ../init.d/dnsmasq
│   ├── K75netfs -> ../init.d/netfs
│   ├── [...]
│   └── S99local -> ../rc.local
├── rc5.d
│   ├── K10saslauthd -> ../init.d/saslauthd
│   ├── K50dnsmasq -> ../init.d/dnsmasq
│   ├── K75netfs -> ../init.d/netfs
│   ├── [...]
│   └── S99local -> ../rc.local
├── rc6.d
│   ├── K10saslauthd -> ../init.d/saslauthd
│   ├── K15tuned -> ../init.d/tuned
│   ├── K25haveged -> ../init.d/haveged
│   ├── [...]
│   └── S01reboot -> ../init.d/halt
[/prism]


### Nivel de ejecución por Defecto

El fichero <code>/etc/inittab</code> (tabla de inicialización) indica has que nivel de ejecución llega el proceso de arranque del sistema, y tiene este aspecto: 

[prism classes="language-bash"]
root@localhost ~> cat /etc/inittab
# inittab is only used by upstart for the default runlevel.
#
# ADDING OTHER CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
#
# System initialization is started by /etc/init/rcS.conf
#
# Individual runlevels are started by /etc/init/rc.conf
#
# Ctrl-Alt-Delete is handled by /etc/init/control-alt-delete.conf
#
# Terminal gettys are handled by /etc/init/tty.conf and /etc/init/serial.conf,
# with configuration in /etc/sysconfig/init.
#
# For information on how to write upstart event handlers, or how
# upstart works, see init(5), init(8), and initctl(8).
#
# Default runlevel. The runlevels used are:
#   0 - halt (Do NOT set initdefault to this)
#   1 - Single user mode
#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
#   3 - Full multiuser mode
#   4 - unused
#   5 - X11
#   6 - reboot (Do NOT set initdefault to this)
id:3:initdefault:
[/prism]

## Anatomía

[div class="table table-striped demonio"]
 ICONO                                                   | RUTA		                                   
---------------------------------------------------------|---------------------------------------------
![EXE](image://teoria/tabla-demonio-exe.png?classes=float-left,demonio&resize=100,100) | <p><b>Ejecutable del Servidor</b></p><p></br><code>/sbin/init</code></p> 
![ETC](image://teoria/tabla-demonio-cnf.png?classes=float-left,demonio&resize=100,100) | <p><b>Configuraciones del Servidor</b></p> <p> <code>/etc/inittab</code></p> <p> <code>/etc/rc.d</code></p> <p> <code>/etc/sysconfig</code></p><p> <code>/etc/init</code></p>
![LIB](image://teoria/tabla-demonio-inmutable.svg?classes=float-left,demonio&resize=100,100) | <p><b>Código fuente y documentación del Servidor</b></p><p><code>/usr/lib/share/doc/initscripts</code></p>    
![VAR](image://teoria/tabla-demonio-var.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento persistente del Servidor</b></p><p><code>/var/lib/stateless</code></p>
![RUN](image://teoria/tabla-demonio-run.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento volátil del Servidor</b> (desaparece tras cada reinicio)</p><p><code>/proc/1</code></p>
![PROC](image://teoria/tabla-demonio-ram.png?classes=float-left,demonio&resize=100,100) | <p><b>Disco virtual que el kernel crea en RAM</b></p><p><code>/proc/1</code></p>
![IPC](image://teoria/tabla-demonio-ipc.svg?classes=float-left,demonio&resize=100,100) | <p><b>Sockets InterProcess Communication </b></p><pre class="language-bash"><code>ss -xpln &#124; grep init</code></pre></p>
![NET](image://teoria/tabla-demonio-tcp.png?classes=float-left,demonio&resize=100,100) | <p><b>Sockets TCP/UDP </b></p><p><pre class="language-bash"><code>ss -tupln &#124; grep init</code></pre></p>
[/div]

## Configuración

[div class="table table-striped demonio"]
FICHERO           |   DESCRIPCIÓN		                                   
------------------|-------------------------------------------------------------------------------
<p><code>/etc/rc.d/init.d</code></p> | Base de datos de Unidades de Ejecución.
<p><code>/etc/init</code></p>        | Controles adicionales para la secuenciación del arranque.
[/div]

## Herramientas de Trabajo
[div class="table table-striped demonio"]
COMANDO                   |  OBJETIVO                                     | DESCRIPCIÓN	                                   
--------------------------|-------------------------------------------------------------------------------
<code>service</code>|<b>Controlar Demonio</b> | La herramienta service se emplea para iniciar(start), detener(stop), reiniciar(restart), status(estado) de los demonios. Simplemente lanza el script bash de <code>/etc/rc.d/init.d</code> con el parámetro que le indica qué debe hacer con el demonio.
<code>chkconfig</code>|<b>Habilitar Demonios en Arranque</b>| Genera los enlaces simbólicos en las carpeta de cada nivel de ejecución, <code>/etc/rc.d/rcX.d</code>, con los prefijos adecuados. 
[/div]

## Comandos
### Control de Units
[div class="table table-striped demonio"]
COMANDO                                                              |  OBJETIVO                                           
---------------------------------------------------------------------|---------------------------------------------------
<pre class="language-bash"><code>service --status-all</code></pre>   | Estado de todas las Unidades de Ejecución
<pre class="language-bash"><code>service {unit} status </code></pre> | Estado de una Unit
<pre class="language-bash"><code>service {unit} stop </code></pre>   | Parar Unit
<pre class="language-bash"><code>service {unit} start}</code></pre>  | Arrancar Unit
<pre class="language-bash"><code>service {unit} restart</code></pre> | Reiniciar Units
[/div]


### Control del Arranque
[div class="table table-striped demonio"]
COMANDO                                                        |  OBJETIVO                                           
---------------------------------------------------------------|---------------------------------------------------
<pre class="language-bash"><code>chkconfig --list </code></pre>| Estructura del Arranque
<pre class="language-bash"><code>chkconfig --list {unit} </code></pre>| Niveles de ejecución donde se arranca la Unit.
<pre class="language-bash"><code>chkconfig {unit} on --run-levels 235</code></pre> | Activar Demonio en el arranque. En el ejemplo, niveles de ejecución 2, 3 y 5.
<pre class="language-bash"><code>chkconfig {unit} off --run-levels 235</code></pre> |Desactivar Demonio en el arranque. En el ejemplo, niveles de ejecución 2, 3 y 5.
[/div]


# Funcionalidad: Inicialización del Entorno de Ejecución.

## Control del Entorno de Ejecución
[div class="table table-striped demonio"]
COMANDO                                                        |  OBJETIVO                                           
---------------------------------------------------------------|---------------------------------------------------
<pre class="language-bash"><code>vi /etc/inittab</code></pre>   | Nivel de ejecución hasta que el que sube el arranque
[/div]


### Cambio de Nivel de Ejecución
[div class="table table-striped demonio"]
COMANDO                                                        |  OBJETIVO                                           
---------------------------------------------------------------|---------------------------------------------------
<pre class="language-bash"><code>init {runlevel}</code></pre>   | Cambia de nivel de Ejecución
<pre class="language-bash"><code>telinit {runlevel}</code></pre>| Cambia de nivel de Ejecución
<pre class="language-bash"><code>runlevel</code></pre>          | Nivel ejecución actual e Historial de cambios
<pre class="language-bash"><code>who -r</code></pre>            | Nivel ejecución actual
[/div]

### Estado de una Unit
[div class="table table-striped demonio"]
COMANDO                                                        |  OBJETIVO                                           
---------------------------------------------------------------|---------------------------------------------------
<pre class="language-bash"><code>service {unit} status</pre>    | PID y Estado de un servicio, posibles valores: <ul><li>running/stopped</li><li>loaded/not loaded</li></ul>
[/div]