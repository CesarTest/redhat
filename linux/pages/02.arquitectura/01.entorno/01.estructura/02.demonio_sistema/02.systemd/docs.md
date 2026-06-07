---
title: SystemD
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

**DEF.: Systemd es un <u>sistema de inicio, administración de servicios y gestión de sistemas</u> para sistemas operativos GNU/Linux.** Es el primer proceso en ejecución tras la carga del núcleo y el que a su vez genera todos los demás procesos. Se ejecuta como demonio y por lo general tiene PID 1. 

Fue desarrollado por Red Hat (importando el diseño de [Launchd](http://0pointer.net/blog/projects/systemd.html?target=_blank) de MacOS) y se ha convertido en el sistema de inicio predeterminado en la mayoría de las distribuciones de Linux modernas, incluidas Archlinux, Debian, Fedora, openSUSE y Ubuntu. Se trata del proceso ID=1 del sistema, la raíz de todos los demás procesos sobre el que recae la responsabilidad de orquestar el ciclo de vida de cada proceso y organizar todo el ecosistema de procesos.

[prism classes="language-bash"]
root@workstation199 ~> pstree
systemd─┬─NetworkManager───2*[{NetworkManager}]
        ├─VBoxDRMClient───3*[{VBoxDRMClient}]
        ├─VBoxService───8*[{VBoxService}]
        ├─agetty
        ├─auditd─┬─sedispatch
        │        └─2*[{auditd}]
        ├─avahi-daemon───avahi-daemon
        ├─crond
        ├─dbus-broker-lau───dbus-broker
        ├─firewalld───{firewalld}
        ├─httpd─┬─httpd
        │       ├─httpd───68*[{httpd}]
        │       └─3*[httpd───52*[{httpd}]]
        ├─php-fpm───11*[php-fpm]
        ├─polkitd───5*[{polkitd}]
        ├─rsyslogd───2*[{rsyslogd}]
        ├─sshd─┬─sshd───sshd───bash
        │      ├─2*[sshd───sshd───sftp-server]
        │      └─sshd───sshd───bash───pstree
        ├─systemd───(sd-pam)
        ├─systemd-journal
        ├─systemd-logind
        └─systemd-udevd
[/prism]

**DEF.: Base de Datos de Unidades de Ejecución:** SystemD mantiene una base de datos de Unidades de Ejecución que describe cómo ha de gestionarse cada programa o recursos del entorno de ejecución. 

**DEF.: Unidad de Ejecución:** Cada unidad de ejecución es un fichero de texto que describe cómo SystemD debe gestionar un demonio o un recurso del Sistema, se almacena en la ruta de datos persistentes del demonio, donde aparecen unidades de ejecución a nivel de sesión de usuario y de sistema. Para activar una unidad de ejecución durante el arranque del sistema, ha de crearse un enlace simbólico en la ruta de configuraciones del demonio.

![SystemD](image://teoria/teoria-entorno-systemd.jpg)

## ¿Para qué? Diseño y Control Centralizado del espacio de Usuario.

Los sistemas Unix vienen empleando SysV Init como gestor de demonios, donde el proceso Init es la raíz de todos los procesos del sistema (PID=1). Sin embargo, ha sido concebido para entornos muy estáticos, donde solo era necesario arrancar procesos secuencialmente por niveles de ejecución. En entornos donde se debe reaccionar ante eventos (como poner y quitar auriculares a un teléfono móvil) se ve la necesidad de reemplazarlo por otro modelo más integral de gestión de demonios. Será Apple quien lanzará LaunchD con el objetivo de paralelismo (velocidad en el arranque), reactividad (gestión centralizada de eventos) y ahorro de batería en los móviles (centralizar todos los demonios de sistema -cron, inetd, etc.- en uno solo que coordine la actividad del resto). Por otro lado, Canonical lanzará Upstart, hoy absorbido por Systemd.

### Propiedades

#### Nuevas Funcionalidades
SystemD tiene varias características principales que lo distinguen de los otros sistemas de inicio de Linux (SysVinit y Upstart):
+ **Paralelismo:** SystemD puede iniciar servicios en paralelo, lo que acelera el proceso de arranque.
+ **Dependencias:** SystemD puede establecer dependencias entre servicios, lo que garantiza que los servicios se inicien en el orden correcto.
+ **Units:** SystemD utiliza un único sistema de unidades de ejecución para describir el comportamiento del sistema y de los servicios.
+ **Journal:** SystemD utiliza un sistema de trazado centralizado para registrar los eventos del sistema.

#### Ventajas frente Init
En definitiva, diseñar y controlar un entorno de ejecución de manera centralizada, describiendo en los ficheros de unidad cada demonio del sistema, los eventos asociados a su ciclo de vida y sus dependencias (en virtud de un demonio anclado a un kernel, lejos de la añorada interoperabilidad de <a href="https://es.wikipedia.org/wiki/Single_Unix_Specification" target="_blank">Single Unix Specifications</a>) aporta las siguientes ventajas frente a Init:
+ **Eficiencia:** más eficiente al iniciar servicios en paralelo gracias a su sistema de dependencias.
+ **Flexibilidad:** ofrece un mayor control sobre el comportamiento del sistema y de los servicios, eliminando la necesidad de tener una batería de demonios auxiliares para el control del entorno.
+ **Seguridad:** incluye funciones de seguridad integradas para proteger el sistema de ataques (<code>systemd-analyze security</code>).

### La Unidad de Ejecución 

Cada Unit es un archivo de texto que contiene información sobre cómo gestionar un demonio o un recurso de entrada/salida del sistema. Su aspecto es el siguiente:

[prism classes="language-bash"]               
root@localhost ~> systemctl cat sshd
# /usr/lib/systemd/system/sshd.service
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.target
Wants=sshd-keygen.target

[Service]
Type=notify
EnvironmentFile=-/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
[/prism]

Tal como muestra la imagen, hay tres tipos de unidades de ejecución:
+ **Service**: control del ciclo de vida de un demonio.
+ **Target**: representa un grupo de unidades de ejecución que se lanzan simultáneamente.
+ **Object**: control del ciclo de vida de un recurso del entorno de ejecución (socket, un disco, un timer, etc.).

![UDEV](image://teoria/teoria-entorno-systemd-units.jpg)

[div class="table table-striped demonio"]
TIPO UNIT    | Control Ciclo de Vida		                                   
-------------|-------------------------------------------------------------------------------
.service     | Demonio 
.target      | Nivel de Ejecución (conjunto de unidades que se lanzan a la vez)
.automount   | Punto de Montaje
.device      | Dispositivo hardware
.mount       | Punto de Montaje
.path        | Ruta del Sistema Ficheros
.scope       | Un proceso creado externamente.
.slice       | Un grupo de unidades organizadas jerárquicamente que gestionan los procesos del sistema.
.socket      | Un socket de comunicación entre procesos.
.swap        | Un dispositivo de intercambio o un archivo de intercambio.
.timer       | Un temporizador systemd.
[/div]


### La Estructura del Arranque

La estructura del arranque en SystemD se define en <code>/etc/systemd</code> donde se va creando un árbol de carpetas <code>{nombre}.target.wants</code> en las que aparecen enlaces a las unidades de ejecución de ese paquete de unidades que han de lanzarse en paralelo:

[prism classes="language-bash"]               
root@localhost ~> tree /etc/systemd/system
  /etc/systemd/system
  ├── ctrl-alt-del.target -> /usr/lib/systemd/system/reboot.target
  ├── dbus-org.freedesktop.Avahi.service -> /usr/lib/systemd/system/avahi-daemon.service
  ├── dbus-org.freedesktop.nm-dispatcher.service -> /usr/lib/systemd/system/NetworkManager-dispatcher.service
  ├── dbus.service -> /usr/lib/systemd/system/dbus-broker.service
  ├── default.target -> /usr/lib/systemd/system/multi-user.target
  ├── getty.target.wants
  │ └── getty@tty1.service -> /usr/lib/systemd/system/getty@.service
  ├── multi-user.target.wants
  │   ├── auditd.service -> /usr/lib/systemd/system/auditd.service
  │   ├── avahi-daemon.service -> /usr/lib/systemd/system/avahi-daemon.service
  │   ├── crond.service -> /usr/lib/systemd/system/crond.service
  │   ├── httpd.service -> /usr/lib/systemd/system/httpd.service
  │   ├── irqbalance.service -> /usr/lib/systemd/system/irqbalance.service
  │   ├── kdump.service -> /usr/lib/systemd/system/kdump.service
  │   ├── NetworkManager.service -> /usr/lib/systemd/system/NetworkManager.service
  │   ├── remote-fs.target -> /usr/lib/systemd/system/remote-fs.target
  │   ├── rsyslog.service -> /usr/lib/systemd/system/rsyslog.service
  │   ├── sshd.service -> /usr/lib/systemd/system/sshd.service
  │   ├── sssd.service -> /usr/lib/systemd/system/sssd.service
  │   ├── vboxadd.service -> /usr/lib/systemd/system/vboxadd.service
  │   └── vboxadd-service.service -> /usr/lib/systemd/system/vboxadd-service.service
  ├  ── etwok-online.target.wants
  │   └── NetworkManager-wait-online.service -> /usr/lib/systemd/system/NetworkManager-wait-online.service
  ├── nginx.service.d
  ├── php-fpm.service.d
  ├── sockets.target.wants
  │   ├── avahi-daemon.socket -> /usr/lib/systemd/system/avahi-daemon.socket
  │   ├── dbus.socket -> /usr/lib/systemd/system/dbus.socket
  │   └── sssd-kcm.socket -> /usr/lib/systemd/system/sssd-kcm.socket
  ├── sysinit.target.wants
  │   ├── nis-domainname.service -> /usr/lib/systemd/system/nis-domainname.service
  │   ├── selinux-autorelabel-mark.service -> /usr/lib/systemd/system/selinux-autorelabel-mark.service
  │   ├── systemd-boot-update.service -> /usr/lib/systemd/system/systemd-boot-update.service
  │   └── systemd-network-generator.service -> /usr/lib/systemd/system/systemd-network-generator.service
  └── timers.target.wants
      ├── dnf-makecache.timer -> /usr/lib/systemd/system/dnf-makecache.timer
      └── logrotate.timer -> /usr/lib/systemd/system/logrotate.timer
[/prism]

### Nivel de ejecución por Defecto

Tal como muestra el comando, el enlace simbólico <code>/etc/systemd/system/default.target</code> apunta al nivel de ejecución por defecto de la máquina. Con los comandos <code>systemctl get-default</code> y <code>systemctl set-default</code> se controla donde apunta ese enlace simbólico.

## Anatomía
[div class="table table-striped demonio"]
 ICONO                                                   | RUTA		                                   
---------------------------------------------------------|---------------------------------------------
![EXE](image://teoria/tabla-demonio-exe.png?classes=float-left,demonio&resize=100,100) | <p><b>Ejecutable del Servidor</b></p><p></br><code>/sbin/init</code></p><p></br><code>/lib/systemd/systemd</code></p>  
![ETC](image://teoria/tabla-demonio-cnf.png?classes=float-left,demonio&resize=100,100) | <p><b>Configuraciones del Servidor</b></p> <p> <code>/etc/systemd</code></p>
![LIB](image://teoria/tabla-demonio-inmutable.svg?classes=float-left,demonio&resize=100,100) | <p><b>Código fuente y documentación del Servidor</b></p><p><code>/lib/systemd</code></p>    
![VAR](image://teoria/tabla-demonio-var.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento persistente del Servidor</b></p><p><code>/usr/lib/systemd</code></p><p><code>/sys/class</code></p>
![RUN](image://teoria/tabla-demonio-run.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento volátil del Servidor</b> (desaparece tras cada reinicio)</p><p><code>/run/udev</code></p>
![PROC](image://teoria/tabla-demonio-ram.png?classes=float-left,demonio&resize=100,100) | <p><b>Disco virtual que el kernel crea en RAM</b></p><p><code>/proc/1</code></p>
![IPC](image://teoria/tabla-demonio-ipc.svg?classes=float-left,demonio&resize=100,100) | <p><b>Sockets InterProcess Communication </b></p><pre class="language-bash"><code>ss -xpln &#124; grep udev</code></pre></p>
![NET](image://teoria/tabla-demonio-tcp.png?classes=float-left,demonio&resize=100,100) | <p><b>Sockets TCP/UDP </b></p><p><pre class="language-bash"><code>ss -tupln &#124; grep udev</code></pre></p>
[/div]

## Configuración

+ **Más Información:** <a href="https://systemd.io(https://systemd.io" target=_blank>systemd.io</a>

[div class="table"]
FICHERO                                                                        |  APLICAR CAMBIOS                                                             | DESCRIPCIÓN		                                   
-------------------------------------------------------------------------------|-------------------------------------------------------------------------------
<p><code>/etc/systemd/system.conf</code></p><p>Ayuda:</p><p><code>man systemd</code></p><p><code>man systemd-system.conf</code></p><p><code>man systemd.unit</code></p><p><code>man systemd.service</code></p><p><code>man systemd.target</code></p><p><code>man systemd.socket</code></p><p><code>man systemd.mount</code></p><p><code>man systemd.{unit}</code></p>| <pre class="language-bash"><code>systemctl daemon-reload</code></pre>        | Las opciones más importantes son las siguientes: <ul><li><b>DefaultTimeoutStartSec:</b> Esta opción especifica el tiempo de espera predeterminado que systemd debe esperar antes de iniciar un servicio. El valor predeterminado es 90 segundos.</li><li><b>DefaultTimeoutStopSec:</b>Esta opción especifica el tiempo de espera predeterminado que systemd debe esperar antes de detener un servicio. El valor predeterminado es 90 segundos.</li><li><b>DefaultRestartSec:</b>Esta opción especifica el tiempo de espera predeterminado que systemd debe esperar antes de reiniciar un servicio. El valor predeterminado es 90 segundos.</li><li><b>DefaultStartLimitIntervalSec:</b>Esta opción especifica el intervalo predeterminado de tiempo que systemd debe esperar antes de intentar reiniciar un servicio que falló. El valor predeterminado es 60 segundos.</li><li><b>DefaultStartLimitBurst:</b>Esta opción especifica el número predeterminado de veces que systemd debe intentar reiniciar un servicio que falló antes de dar por terminado. El valor predeterminado es 3.</li></ul>
<code>/usr/lib/systemd</code>| <pre class="language-bash"><code>systemctl daemon-reload</code></pre>        |Almacenamiento de Ficheros de Unit: <ul><li><code>/system</code>: de sistema.</li><li><code>/user</code>: de sesión de usuario</li></ul> 
<p><code>/etc/systemd</code>| <pre class="language-bash"><code>systemctl enable/disable {unit}</code></pre>        |Almacenamiento de enlaces simbólicos a las Units que deben iniciarse durante el arranque del ordenador.
[/div]


## Herramientas de Trabajo
[div class="table table-striped demonio"]
COMANDO                   |  OBJETIVO                                     | DESCRIPCIÓN	                                   
--------------------------|-------------------------------------------------------------------------------
<code>systemctl</code>|<b>Control del Entorno de Ejecución</b> | La herramienta systemctl es la herramienta principal para administrar servicios con systemd. Puede usar systemctl para iniciar(start), detener(stop), reiniciar(restart), habilitar(enable), deshabilitar(disable), inhibir(mask) y desinhibir(unmask) 
<code>journalctl</code>|<b>Trazabilidad de eventos del Entorno de Ejecución</b>| La herramienta journalctl muestra el registro de systemd. Puede usar journalctl para ver los eventos que ocurrieron en los servicios
<code>systemd-analyze</code>|<b>Análisis de Rendimiento del Entorno de Ejecución</b>| La herramienta systemd-analyze proporciona información sobre la configuración de systemd. Puede usar systemd-analyze para analizar el procesos de arranque, ajustar nivel de trazas o ver niveles de seguridad de los servicios. 
[/div]

## Comandos

### Control del Entorno
[div class="table table-striped demonio"]
COMANDO                                                        |  OBJETIVO                                           
---------------------------------------------------------------|---------------------------------------------------
<pre class="language-bash"><code>systemctl list-dependencies</code></pre>| Árbol de Dependencias
<pre class="language-bash"><code>systemctl list-sockets</code></pre>     | Listado Sockets
<pre class="language-bash"><code>systemctl list-jobs</code></pre>        | Jobs SystemD activos
<pre class="language-bash"><code>systemctl list-unit-files</code></pre>  | Modo arranque Units
<pre class="language-bash"><code>systemctl list-units</code></pre>       | Status Entorno Ejecución
<pre class="language-bash"><code>systemctl get-default</code></pre>      | Default Target
<pre class="language-bash"><code>systemctl set-default {target}</code></pre>| Configurar Default Target
<pre class="language-bash"><code>systemctl daemon-reload</code></pre>    | Refresco Units y Dependencias
<pre class="language-bash"><code>systemctl --failed</code></pre>         | Unit que fallaron
<pre class="language-bash"><code>systemctl reset-failed</code></pre>     | Reseteo de Units que fallaron
<pre class="language-bash"><code>systemctl reboot</code></pre>           | Reiniciar Máquina
<pre class="language-bash"><code>systemctl poweroff</code></pre>         | Apagar Máquina
<pre class="language-bash"><code>systemctl emergency</code></pre>        | Single User Mode
<pre class="language-bash"><code>systemctl default</code></pre>          | Volver al target por defecto.
<pre class="language-bash"><code>systemd-analyze get-log-level</code></pre> | Nivel de trazas en Systemd.
<pre class="language-bash"><code>systemd-analyze set-log-level {1-7}</code></pre> | Configurar Nivel de trazas en Systemd.
<pre class="language-bash"><code>systemd-analyze security</code></pre>   | Estado de la Seguridad de los servicios
<pre class="language-bash"><code>systemd-analyze critical-chain {unit} </code></pre>   | Secuencia de arranque de una Unidad de Ejecución
<pre class="language-bash"><code>systemd-analyze blame </code></pre>   | Análisis de tiempos del proceso de arranque
<pre class="language-bash"><code>systemd-analyze plot > boot.svg </code></pre>   | Análisis gráfico de tiempos del proceso de arranque
[/div]

### Control de Units

[div class="table table-striped demonio"]
COMANDO                                                        |  OBJETIVO                                           
---------------------------------------------------------------|---------------------------------------------------
<pre class="language-bash"><code>systemctl status {unit}</code></pre> | Descripción Unit
<pre class="language-bash"><code>systemctl stop {unit} </code></pre>  | Parar Unit
<pre class="language-bash"><code>systemctl start {unit}</code></pre>  | Arrancar Unit
<pre class="language-bash"><code>systemctl restart {unit}</code></pre>| Reiniciar Units
<pre class="language-bash"><code>systemctl reload {unit}</code></pre> | Recargar Unit
<pre class="language-bash"><code>systemctl enable {unit}</code></pre> | Habilitar Unit (+symlink <code>/etc/systemd</code>)
<pre class="language-bash"><code>systemctl disable {unit}</code></pre>| Deshabilitar Unit (-symlink <code>/etc/systemd</code>)
<pre class="language-bash"><code>systemctl mask {unit}</code></pre>   | Inhibir Unit (+symlink <code>/dev/null</code>)
<pre class="language-bash"><code>systemctl unmask {unit}</code></pre> | Habilitar Unit (-symlink <code>/dev/null</code>)
<pre class="language-bash"><code>systemctl show {unit}</code></pre>   | Propiedades de la Unit
<pre class="language-bash"><code>systemctl cat {unit}</code></pre>    | Ver fichero Unit
<pre class="language-bash"><code>systemctl edit --full {unit}</code></pre>      | Editar fichero de Unit
<pre class="language-bash"><code>systemctl -H <host> status {unit}</code></pre> | Comando SystemD por SSH
[/div]

# Funcionalidad: Diseño y Control del Entorno de Ejecución

![Funcionalidad SystemD](image://teoria/teoria-entorno-systemd-funcionalidad.jpg)

## Control del Entorno de Ejecución

### Cambio Nivel Ejecución
[div class="table table-striped demonio"]
|ACCIÓN                           | COMANDO
|---------------------------------|---------------------
| Niveles de ejecución existentes | <pre class="language-bash"><code>ls -la /usr/lib/systemd/system/run*.target </code></pre>
| Cambio nivel de ejecución       | <pre class="language-bash"><code>systemdctl isolate {target} </code></pre>
[/div]


Alguno ejemplos:
[div class="table table-striped demonio"]
|EJEMPLO                             | COMANDO
|------------------------------------|---------------------
|<b>Cambiar al modo gráfico:</b>     |<pre class="language-bash"><code>systemctl isolate graphical.target</code></pre>
|<b>Cambiar al modo multiusuario:</b>|<pre class="language-bash"><code>systemctl isolate multi-user.target</code></pre></li>
|<b>Reiniciar el sistema:</b>        |<pre class="language-bash"><code>systemctl isolate reboot.target</code></pre> </li>
|<b>Modo de rescate:</b>             |<pre class="language-bash"><code>systemctl rescue</code></pre> </li>
|<b>Modo Emergencia:</b>             |<pre class="language-bash"><code>systemctl emergency</code></pre></li>
[/div]

### Dependencias de Units

![Dependencias](image://teoria/teoria-entorno-systemd-dependencias.jpg)

### Estado de Units

A continuación se detallan los cuatro parámetros que definen el estado de una Unit:

+ *Estado de la ejecución:* 
 - **LOAD:**   carga en memoria del fichero de Unit y cálculo de dependencias por parte de SystemD.
 - **ACTIVE:** estado genérico de las unidades de ejecución.
 - **SUB:**    estado específico de cada tipo de unidad de ejecución.
+ *Motivo de la ejecución:* 
 - **STATE:**  motivo de arranque de la Unit.

#### Status de una Unit: LOAD, ACTIVE (SUB), STATE

En la descripción del status de una única unidad de ejecución aparecen los cuatro parámetros como se aprecia en la imagen, tanto motivo como estado de la ejecución. 
Sin embargo, cuando quiere ver el entorno de ejecución en su conjunto, se ve por separado el estado  y el motivo de la ejecución de cada Unit.

![Status Unit SystemD](image://teoria/teoria-entorno-systemd-status.jpg)

#### Listado de Units: LOAD, ACTIVE (SUB)

Los parámetros LOAD, ACTIVE (SUB) del status de las Units son relativos a si se está ejecutando y cómo, pero no dice nada sobre el motivo del por qué están encendidos o apagados.

Para ver los posibles valores de estos tres campos, se puede emplear el comando:
[prism classes="language-bash"] 
systemctl --state=help
[/prism]

Este sería un ejemplo de la salida del comando:

[prism classes="language-bash"]  
root@localhost> systemctl list-units

UNIT                                      LOAD   ACTIVE     SUB       DESCRIPTION
proc-sys-fs-binfmt_misc.automount         loaded active     waiting   Arbitrary Executable File>
dev-fuse.device                           loaded activating tentative /dev/fuse
s.mount                                   loaded active     mounted   Root Mount
boot.mount                                loaded active     mounted   /boot
dev-hugepages.mount                       loaded active     mounted   Huge Pages gdm.service                                                                                                         
var-lib-nfs-rpc_pipefs.mount              loaded active     mounted   RPC Pipe File System
cups.path                                 loaded active     running   CUPS Scheduler
systemd-ask-password-plymouth.path        loaded active     waiting   Forward Password Requests>
systemd-ask-password-wall.path            loaded active     waiting   Forward Password Requests>
init.scope                                loaded active     running   System and Service Manager
machine-qemu\x2d61\x2dCWP.rhel8.4.scope   loaded active     running   Virtual Machine qemu-61-C>
session-1.scope                           loaded active     running   Session 1 of user pangea
accounts-daemon.service                   loaded active     running   Accounts Service
auditd.service                            loaded active     running   Security Auditing Service
import-state.service                      loaded active     exited    Import network configurat>
iscsi-shutdown.service                    loaded active     exited    Logout off all iSCSI sess>
kmod-static-nodes.service                 loaded active     exited    Create list of required s>
ksm.service                               loaded active     exited    Kernel Samepage Merging
-.slice                                   loaded active     active    Root Slice
machine.slice                             loaded active     active    Virtual Machine and Conta>
avahi-daemon.socket                       loaded active     running   Avahi mDNS/DNS-SD Stack A
[…]
LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

208 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
[/prism]

#### Listado de Unit Files: STATE

El parámetro STATE del status de las Units indica el motivo por el cuál una Unit se enciende. El sistema de dependencias de SystemD introduce nuevas causas por las qué una Unit puede ser encendida, además de la habilitación a través de enlace simbólico como ya se ha visto.

Los posibles valores de este parámetro son los siguientes:

[div class="table table-striped"]
STATE                               |  Descripción                                           
------------------------------------|---------------------------------------------------
<p>enable</p><p>enables-runtime</p> | Habilitado para lanzamiento durante al arranque en .wants/,.requires/. Alias = enlaces simbólicos (permanentemente en <code>/etc/systemd/system/</code>, o transitoriamente en <code>/run/systemd/system/</code>)
<p>linked</p><p>linked-runtime</p>  | Disponible a través de uno o más enlaces simbólicos al archivo de la unidad (permanentemente en <code>/etc/systemd/system/</code> o transitoriamente en <code>/run/systemd/system/</code>), aunque el archivo de nidad pueda residir fuera de la ruta de datos persistentes de Systemd).
alias                               | Enlace simbólico a otra unidad.
<p>masked</p><p>masked-runtime</p>  | Completamente deshabilitado, por lo que cualquier operación de inicio de "tiempo de ejecución enmascarado" falla (permanentemente en <code>/etc/systemd/system/</code> o transitoriamente en <code>/run/systemd/systemd/</code>)
static                              | La unidad no está habilitada durante el arranque ni tiene una indicación expresa en ningún fichero de unidad, se activa durante el arranque solo si lo llama alguna dependencia.
indirect                            | El archivo de unidad en sí no está habilitado, pero tiene una configuración Also= no vacía en la sección de archivo de unidad \[Install\], que enumera otros archivos de unidad que podrían estar habilitados, o tiene un alias con un nombre diferente a través de un enlace simbólico que es no especificado en Also=. Para archivos de unidad de plantilla, se habilita una instancia diferente a la especificada en DefaultInstance=.
disabled                            | El archivo de unidad no está habilidad durante el arranque, pero contiene una sección [Install] con instrucciones de instalación.
generated                           | El archivo de unidad se ha generado dinámicamente via herramienta de generación. Ver systemd.generator. Las unidades generadas pueden no estar activadas durante el arranque, serán lanzadas durante el implícitamente por sus generadores.
transient                           | El archivo de unidad ha sido creado dinámicamente por el API de runtime. Estas unidades transitorias pueden no estar activas durante el arranque.
bad                                 | Fichero de unidad inválido o hay algún otro error. El comando is-enabled no muestra este estado, solo el comando list-unit-files.
not-found                           | El archivo de unidad no existe.
[/div]

Este sería un ejemplo de la salida del comando:

[prism classes="language-bash"] 
root@localhost> systemctl list-unit-files

UNIT FILE                               STATE
proc-sys-fs-binfmt_misc.automount       static
-.mount                                 generated
boot-efi.mount                          generated
boot.mount                              generated
dev-hugepages.mount                     static
proc-sys-fs-binfmt_misc.mount           static
run-vmblock\x2dfuse.mount               disabled
sys-fs-fuse-connections.mount           static
tmp.mount                               disabled
var-lib-libvirt-images.mount            generated
var-lib-machines.mount                  static
var-lib-nfs-rpc_pipefs.mount            static
cups.path                               enabled
insights-client-results.path            disabled
ostree-finalize-staged.path             disabled
machine-qemu\x2d61\x2dCWP.rhel8.4.scope transient
session-1.scope                         transient
accounts-daemon.service                 enabled
alsa-restore.service                    static
alsa-state.service                      static
arp-ethers.service                      disabled
atd.service                             enabled
auditd.service                          enabled
auth-rpcgss-module.service              static
autovt@.service                         enabled
runlevel0.target                        disabled
runlevel1.target                        static
runlevel2.target                        static
runlevel3.target                        static
runlevel4.target                        static
runlevel5.target                        indirect
runlevel6.target                        enabled
systemd-tmpfiles-clean.timer            static
unbound-anchor.timer                    enabled

492 unit files listed.
[/prism]

## Comportamiento del Entorno de Ejecución

El comando <code>systemd-analyze</code> permite analizar el comportamiento del entorno de ejecución:
+ **Ajustar Nivel de Depuración:** nivel de trazas de SystemD,  trazas que se consultan con el comando <code>journalctl</code>.
+ **Análisis del Proceso de Arranque:** que puede servir para localizar errores de configuración y corregirlos, o eliminar demonios superfluos en un sistema.
+ **Análisis de Seguridad:** detección de vulnerabilidades del entorno de ejecución.
 
#### Criticidad de las Trazas

[div class="table table-striped demonio"]
|ACCIÓN                           | COMANDO
|---------------------------------|---------------------
| Nivel Trazas SystemD            | <pre class="language-bash"><code>systemd-analyze get-log-level</code></pre>
| Cambiar Nivel Trazas SystemD    | <pre class="language-bash"><code>systemd-analyze set-log-level {0-7}</code></pre>
[/div]
 
#### Análisis del Arranque

+ **Para ver la secuencia de arranque de una unidad de ejecución:** el comando critical-chain imprime un árbol de la cadena de unidades de tiempo crítico (para cada una de las UNIDADES especificadas o, en caso contrario, para el default target). 
  - "@" = Tiempo después de que la unidad esté activa o iniciada se imprime después del carácter "@". 
  - "+" = El tiempo que tarda la unidad en iniciarse se imprime después del carácter "+". 
[prism classes="language-bash"]systemd-analyze critical-chain default.target[/prism]

!!! Tenga en cuenta que el resultado puede ser engañoso ya que la inicialización de los servicios puede depender de la activación del socket y de la ejecución paralela de las unidades. Además, de manera similar al comando blame, esto solo tiene en cuenta el tiempo que las unidades pasaron en el estado "activando" y, por lo tanto, no cubre las unidades que nunca pasaron por un estado "activando" (como las unidades de dispositivos que pasan directamente de "inactivo" a "activo"). Además, no muestra información sobre trabajos (y en particular, trabajos que expiraron).


+ **Para ordenar las unidades de ejecución según tiempo de inicio durante el proceso de arranque del sistema:** el comando blame imprime una lista de todas las unidades en ejecución, ordenadas por el tiempo que tardaron en inicializarse. Esta información se puede utilizar para optimizar los tiempos de inicio. 
[prism classes="language-bash"]systemd-analyze blame default.target[/prism]

!!! Tenga en cuenta que el resultado puede ser engañoso ya que la inicialización de un servicio puede ser lenta simplemente porque espera a que se complete la inicialización de otro servicio. También tenga en cuenta: systemd-analyze listening no muestra resultados para servicios con Type=simple, porque systemd considera que dichos servicios se inician inmediatamente, por lo que no se pueden medir los retrasos de inicialización. También tenga en cuenta que este comando solo muestra el tiempo que tardaron las unidades en iniciarse, no muestra cuánto tiempo pasaron los trabajos de la unidad en la cola de ejecución. En particular, muestra las unidades de tiempo que pasan en el estado "activando", que no está definido para unidades como unidades de dispositivos que pasan directamente de "inactivo" a "activo". Por lo tanto, este comando da una impresión del rendimiento del código del programa, pero no puede reflejar con precisión la latencia introducida por la espera de hardware y eventos similares

+ **Para obtener una representación gráfica del proceso de arranque** en fichero de gráficos vectoriales:
[prism classes="language-bash"]systemd-analyze plot > boot.svg[/prism]

![Dependencias](image://teoria/teoria-entorno-systemd-plot.jpg)

#### Análisis de Seguridad
[div class="table table-striped demonio"]
| ACCIÓN                                                | COMANDO
|-------------------------------------------------------|---------------------
|**Para análisis de seguridad del entorno:**            |<pre class="language-bash" data-prompt="$" >systemd-analyze security</code></pre>
|**Para análisis de seguridad de una Unit específica:** | <pre class="language-bash" data-prompt="$" >systemd-analyze security {unit}</code></pre>
[/div]

## Diseño del Entorno de Ejecución: Las Units

###	Campos Comunes
[div class="table table-striped demonio"]
| ACCIÓN  | COMANDO
|---------|---------------------
| Manual: | <pre class="language-bash" >man systemd.unit</code></pre>
[/div]
![Unit Común](image://teoria/teoria-entorno-systemd-unit.jpg)

#### Sección Init: Metadatos y Dependencias

[div class="table table-striped"]
| Directiva	      |  Descripción
|-----------------|--------------------
| Description=:	  | Esta directiva se puede utilizar para describir el nombre y la funcionalidad básica de la unidad. Lo devuelven varias herramientas systemd, por lo que es bueno configurarlo como algo breve, específico e informativo.
| Documentation=: |	Esta directiva proporciona una ubicación para una lista de URI para documentación. Pueden ser páginas de manual disponibles internamente o URL accesibles desde la web. El comando systemctl status expondrá esta información, lo que permitirá una fácil detección.
| Require =:	  | Esta directiva enumera las unidades de las que depende esencialmente esta unidad. Si la unidad actual está activada, las unidades enumeradas aquí también deben activarse exitosamente; de lo contrario, esta unidad fallará. Estas unidades se inician en paralelo con la unidad actual de forma predeterminada.
| Wants=:	      | Esta directiva es similar a Requires=, pero menos estricta. Systemd intentará iniciar cualquier unidad enumerada aquí cuando esta unidad esté activada. Si estas unidades no se encuentran o no se inician, la unidad actual seguirá funcionando. Esta es la forma recomendada de configurar la mayoría de las relaciones de dependencia. Nuevamente, esto implica una activación paralela a menos que otras directivas lo modifiquen.
| BindsTo=:	      | Esta directiva es similar a Requires=, pero también hace que la unidad actual se detenga cuando finaliza la unidad asociada.
| Before=:	      | Las unidades enumeradas en esta directiva no se iniciarán hasta que la unidad actual esté marcada como iniciada si se activan al mismo tiempo. Esto no implica una relación de dependencia y debe usarse junto con una de las directivas anteriores si así lo desea.
| After =:	      | Las unidades enumeradas en esta directiva se iniciarán antes de iniciar la unidad actual. Esto no implica una relación de dependencia y deberá establecerse a través de las directivas anteriores si así se requiere.
| Conflicts=:	  | Esto se puede usar para enumerar las unidades que no se pueden ejecutar al mismo tiempo que la unidad actual. Iniciar una unidad con esta relación hará que las otras unidades se detengan.
| Condition=:	  | Hay una serie de directivas que comienzan con Condición que permiten al administrador probar ciertas condiciones antes de iniciar la unidad. Esto se puede utilizar para proporcionar un archivo de unidad genérico que solo se ejecutará en los sistemas apropiados. Si no se cumple la condición, la unidad se omite correctamente.
| Assert=:	      | Similar a las directivas que comienzan con Condition, estas directivas verifican diferentes aspectos del entorno de ejecución para decidir si la unidad debe activarse. Sin embargo, a diferencia de las directivas de Condición, un resultado negativo provoca un fallo con esta directiva.
[/div]

#### Sección Install: Target de la Unit (no presente en tipo Target)

[div class="table table-striped"]
| Directiva	       |  Descripción
|------------------|--------------------
|WantedBy=:	       | La directiva WantedBy= es la forma más común de especificar cómo se debe habilitar una unidad. Esta directiva le permite especificar una relación de dependencia de manera similar a como lo hace la directiva Wants= en la sección \[Unidad\]. La diferencia es que esta directiva se incluye en la unidad auxiliar, lo que permite que la unidad primaria listada permanezca relativamente limpia. Cuando se habilita una unidad con esta directiva, se creará un directorio dentro de <code>/etc/systemd/system</code> con el nombre de la unidad especificada con .wants agregado al final. Dentro de esto, se creará un enlace simbólico a la unidad actual, creando la dependencia. Por ejemplo, si la unidad actual tiene WantedBy=multi-user.target, se creará un directorio llamado multi-user.target.wants dentro de <code>/etc/systemd/system</code> (si aún no está disponible) y un enlace simbólico a la unidad actual. será colocado dentro. Al deshabilitar esta unidad se elimina el vínculo y se elimina la relación de dependencia.
|RequiredBy=:	   | Esta directiva es muy similar a la directiva WantedBy=, pero en su lugar especifica una dependencia requerida que hará que la activación falle si no se cumple. Cuando está habilitada, una unidad con esta directiva creará un directorio que terminará en .requires.
|Alias=:	       | Esta directiva permite habilitar la unidad también con otro nombre. Entre otros usos, esto permite que estén disponibles múltiples proveedores de una función, de modo que las unidades relacionadas puedan buscar cualquier proveedor con el nombre de alias común.
|Also =:	       | Esta directiva permite habilitar o deshabilitar unidades como un conjunto. Las unidades de apoyo que siempre deberían estar disponibles cuando esta unidad está activa echose pueden enumerar aquí. Se gestionarán en grupo para las tareas de instalación.
|DefaultInstance=: |	Para unidades de plantilla (que se tratan más adelante) que pueden producir instancias de unidad con nombres impredecibles, esto se puede utilizar como valor alternativo para el nombre si no se proporciona un nombre apropiado.
[/div]

###	Tipo Service: Ciclo Vida Demonio

[div class="table table-striped demonio"]
| ACCIÓN | COMANDO
|--------|--------
| Manual:|<pre class="language-bash"><code>man systemd.service</code></pre>
[/div]
![Unit Service](image://teoria/teoria-entorno-systemd-service.jpg)

#### Sección Service: Transiciones y Eventos

[div class="table table-striped"]
| Directiva	      |  Descripción
|-----------------|--------------------
| Type=:	      | Tipo de instanciación.<ul><li><b>simple:</b> El proceso principal del servicio se especifica en la línea de inicio. Este es el valor predeterminado si las directivas Type= y Busname= no están configuradas, pero sí ExecStart=. Cualquier comunicación debe manejarse fuera de la unidad a través de una segunda unidad del tipo apropiado (como a través de una unidad .socket si esta unidad debe comunicarse mediante enchufes).</li><li><b>fork:</b> este tipo de servicio se utiliza cuando el servicio bifurca un proceso secundario y sale del proceso principal casi de inmediato. Esto le dice a systemd que el proceso todavía se está ejecutando aunque el padre salió.</li><li><b>oneshot:</b> Este tipo indica que el proceso será de corta duración y que systemd debe esperar a que el proceso finalice antes de continuar con otras unidades. Este es el valor predeterminado Type= y ExecStart= no están configurados. Se utiliza para tareas puntuales.</li><li><b>dbus:</b> Esto indica que la unidad tomará un nombre en el bus D-Bus. Cuando esto suceda, systemd continuará procesando la siguiente unidad.</li><li><b>notify:</b> Esto indica que el servicio emitirá una notificación cuando haya terminado de iniciarse. El proceso systemd esperará a que esto suceda antes de continuar con otras unidades.</li><li><b>idle:</b> esto indica que el servicio no se ejecutará hasta que se envíen todos los trabajos.</li><ul>
| ExecStart=:	  | Especifica la ruta completa y los argumentos del comando a ejecutar para iniciar el proceso. Esto sólo se puede especificar una vez (excepto para los servicios “oneshot”). Si la ruta al comando está precedida por un guión “-”, se aceptarán estados de salida distintos de cero sin marcar la activación de la unidad como fallida.
| ExecPreStart=:  |	Esto se puede utilizar para proporcionar comandos adicionales que deben ejecutarse antes de que se inicie el proceso principal. Esto se puede utilizar varias veces. Nuevamente, los comandos deben especificar una ruta completa y pueden ir precedidos de "-" para indicar que se tolerará el error del comando.
| ExecPostStart=: | Tiene exactamente las mismas cualidades que ExecStartPre= excepto que especifica los comandos que se ejecutarán después de que se inicie el proceso principal.
| ExecReload=:	  | Esta directiva opcional indica el comando necesario para recargar la configuración del servicio si está disponible.
| ExecStop=:	  | Indica el comando necesario para detener el servicio. Si no se proporciona esto, el proceso finalizará inmediatamente cuando se detenga el servicio.
| ExecStopPost=:  |	Esto se puede utilizar para especificar comandos para ejecutar después del comando de parada.
| RestartSec=:	  | Si está habilitado el reinicio automático del servicio, esto especifica la cantidad de tiempo que se debe esperar antes de intentar reiniciar el servicio.
| Restart=:	      | Esto indica las circunstancias bajo las cuales systemd intentará reiniciar automáticamente el servicio. Esto se puede establecer en valores como "siempre", "en caso de éxito", "en caso de error", "en caso de anomalía", "en caso de aborto" o "en vigilancia". Estos activarán un reinicio según la forma en que se detuvo el servicio.'
| TimeoutSec=:	 | Esto configura la cantidad de tiempo que systemd esperará al detener o detener el servicio antes de marcarlo como fallido o cerrarlo por la fuerza. También puede establecer tiempos de espera separados con TimeoutStartSec= y TimeoutStopSec=.
[/div]

#### Ciclo de Vida

[div class="table table-striped demonio"]
| ACCIÓN                       | COMANDO
|------------------------------|---------------------
|<b>Listado Activas:</b>       |<pre class="language-bash"><code>systemctl list-units --type=service --state=active</code></pre></li>
|<b>Listado Inactivas:</b>     |<pre class="language-bash" data-prompt="$" ><code>systemctl list-units --type=service --state=inactive</code></pre></li>
|<b>Arranque:</b>              |<pre class="language-bash" data-prompt="$" ><code>systemctl start sshd.service</code></pre></li>
|<b>Parada:</b>                |<pre class="language-bash" data-prompt="$" ><code>systemctl stop sshd.service</code></pre></li>
|<b>Habilitar:</b>             |<pre class="language-bash" data-prompt="$" ><code>systemctl enable sshd.service</code></pre></li>
|<b>Deshabilitar:</b>          |<pre class="language-bash" data-prompt="$" ><code>systemctl disable sshd.service</code></pre></li>
|<b>Inhibir:</b>               |<pre class="language-bash" data-prompt="$" ><code>systemctl mask sshd.service</code></pre></li>
|<b>Deshinibir:</b>            |<pre class="language-bash" data-prompt="$" ><code>systemctl unmask sshd.service</code></pre></li>
[/div]


###	Tipo Object: Ciclo Vida Recursos I/O

[div class="table table-striped demonio"]
| ACCIÓN                       | COMANDO
|------------------------------|---------------------
|<b>Manual</b>(pulsar autocompletar para que aparezca la lista de Units): | <pre class="language-bash" data-prompt="$" >man systemd. </code></pre>
[/div]

![Unit Object](image://teoria/teoria-entorno-systemd-object.jpg)

Hay un abanico de objetos del entorno de ejecución que SystemD puede monitorizar. El resultado puede ser:
+ **La generación de nuevas Units (transitorias o permanentes) que definen un nuevo servicio.** Por ejemplo, si se enchufa un punto de montaje, se lanza automáticamente la base de datos que hace uso de ese punto de montaje.
+ **Lanzar demonios o procesos definidos en otras Units:**
   - *Caso de procesos:* cada nuevo evento sobre un recurso I/O genera una Unit temporal en <code>/run/systemd/units</code> describiendo el proceso que acaba de  lanzar. La Unit desaparece cuando el proceso ha terminado su ejecución. 
   - *Caso de demonios:* se lanza un demonio que queda a la escucha, además de apagar la escucha sobre el recursos I/O. Este es el caso de servicio cockpit, tiene una Unit tipo socket que levanta el servicio cuando recibe una petición por un puerto, dejando SystemD de monitorizar ese socket. 

###	Tipo Target: Ciclo Vida del Espacio de Usuario

[div class="table table-striped demonio"]
| ACCIÓN                       | COMANDO
|------------------------------|---------------------
|**Manual:**                   | <pre class="language-bash"><code>man systemd.target</code>
[/div]

![Unit Target](image://teoria/teoria-entorno-systemd-target.jpg)

Estas unidades representan colecciones de unidades de ejecución que se lanzan en paralelo. En todos los demás tipos de unidades de ejecución aparece la sección Install, donde se indica a qué target hay que agregar esa Unidad. Esto se traduce en que la habilitación de la unidad agregará un enlace simbólico a carpeta en <code>/etc/systemd/system/{nombre}.target.wants</code>.
 
En estos ficheros de texto, solo se indica de estructuras de dependencias de cada target, el resultado final es la secuencia de niveles de ejecución, o sea, ese laminado secuencial del proceso de arranque que facilita depurar el entorno de ejecución y su arranque.

[div class="table table-striped demonio"]
| ACCIÓN                       | COMANDO
|------------------------------|---------------------
|**Laminado del Entorno:**     | <pre class="language-bash"><code>systemctl list-dependencies --reverse</code></pre>
[/div]