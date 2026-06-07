---
title: 2.2.Auditoría
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

# Subsistema de Auditoría

## Introducción 

### Objetivos: Trazabilidad del Kernel
La telemetría consiste en añadir trazas a ciertos eventos que se quieren apreciar con más detalle, es decir, sondas de monitorización personalizadas, aunque el formato de la traza pueda seguir un estándar como Syslog. 

En el caso de Linux, las reglas de auditoría permite una introspección de las llamadas al sistema que hacen los programas, el Kernel generará un evento por cada regla de auditoría, que será transmitido a través del socket NetLink para ser recogido por un demonio encargado de volcar ese evento a un fichero de log. 

Para poder auditar la actividad del sistema se torna crucial conocer las principales llamadas al sistema en Linux, donde se aparecen dos grandes familias:
1.	**Llamadas al Sistema para gestionar Procesos:** lo que implica asignación de espacio de memoria donde cargar el programa y tiempos de CPU donde ejecutarlos.
	+ <code>execve</code>: Esta es la llamada al sistema utilizada para iniciar nuevos procesos. Extremadamente ruidoso pero realmente útil para responder a incidentes, entre otras cosas.
    + <code>ptrace/process_vm_readv/process_vm_writev</code>: estas llamadas permiten hacer una introspección del comportamiento de los procesos. Idealmente, estas llamadas no deberían ocurrir en entornos de producción así que es fácil transformarlas en alertas.
2.	**Llamadas al Sistema para gestionar recursos de Entrada/Salida (los ficheros):** los módulos del kernel implementan estas llamadas al sistema para cada dispositivo concreto.
   + <code>listen/bind/connect/sedmsg/rcvmsg</code>: estas llamadas son los ladrillos de los recursos I/O de red, o sea, introspección de los descriptores de fichero de tipo socket. Ya sea para detectar que alguien está sondeando la actividad de puertos y servicios locales con el comando <code>ncat</code> con fines maliciosos, está abriendo una shell a través de conexión inversa o simplemente un comportamiento de red fuera de lo normal, todo comenzará por el análisis de este tipo de llamadas.
   + <code>open/close/read/write</code>: estas llamadas son los ladrillos de los recursos I/O que no sean de red, es decir, ficheros tipo bloque o carácter. Es extremadamente ruidoso, pero podemos filtrarlo según los aperturas o accesos a ficheros. Reglas comunes sería detectar errores en la apertura de archivos debido a permisos (EPERM, EACCES), lo que puede ser una señal de que alguien ya está en el sistema realizando algún reconocimiento. Esto podría ajustarse fácilmente para registrar todas las aperturas de archivos (exitosas y no) o incluso solo archivos particulares como <code>/etc/passwd</code> que son tocados por un proceso que generalmente no necesita tocarlo (por ejemplo, un proceso web).

### Arquitectura 
![Subsistema Audit](image://teoria/teoria-audit-subsistema.jpg)

El sistema de auditoría que viene por defecto con Linus se compone de dos demonios:
+ **Demonio AuditD:** monitoriza los eventos activados a través de reglas de auditoría.
+ **Demonio AudispD:** permite comunicaciones con otras componentes. Especialmente importante cuando se quiere externalizar la telemetría a un servidor recoja las auditoría de muchos ordenadores y las integre en algún servicio de monitorización (por ejemplo: chkmk o Nagios). No se trata en esta documentación.

### Funcionalidad
Tal como muestra la imagen, cuando se registra una regla de auditoría, cada vez que suceda la condición descrita en la regla el kernel generará un evento. El evento se envía a un proceso de espacio de usuario (generalmente el auditd) a través de algo llamado socket "netlink". (El tl;dr en netlink es que le dice al kernel que envíe mensajes a un proceso a través de su PID, y los eventos aparecen en este socket).
 
![Funcionalidad Audit](image://teoria/teoria-audit-funcionalidad.jpg)

## Demonio AuditD 

### ¿Para qué qué AuditD? Imprimir a Fichero Eventos Kernel.
El demonio AuditD es el responsable de recoger los eventos generados por kernel a partir de las reglas de auditoría, y volcarlas en una traza en <code>/var/log/audit/audit.log</code> (aunque esto es configurable). Las propiedades de las trazas que generan estos eventos son:
+ **Multilínea:** una llamada al sistema puede contener una secuencia de parámetros, cada uno se describe en una línea diferente.
+ **Líneas Intercaladas:** según como produzca el kernel los eventos, pueden aparecer líneas de varios eventos mezcladas. Es necesario saber agrupar las líneas de traza que pertenecen a un solo evento.
+ **Contexto del Evento Detallado:** cada evento puede tener una batería amplia de parámetros de contexto, muchas veces difíciles de interpretar sin graficar la situación bajo análisis.

### Anatomía
[div class="table table-striped demonio"]
 ICONO                                                   | RUTA		                                   
---------------------------------------------------------|---------------------------------------------
![EXE](image://teoria/tabla-demonio-exe.png?classes=float-left,demonio&resize=100,100) | <p><b>Ejecutable del Servidor</b></p><p></br><code>/sbin/auditd</code></p>     
![ETC](image://teoria/tabla-demonio-cnf.png?classes=float-left,demonio&resize=100,100) | <p><b>Configuraciones del Servidor</b></p> <p> <code>/etc/audit</code></p>
![LIB](image://teoria/tabla-demonio-inmutable.svg?classes=float-left,demonio&resize=100,100) | <p><b>Código fuente y documentación del Servidor</b></p><p><code>/usr/share/audit</code></p><p><code>/usr/share/doc/audit</code></p> 
![VAR](image://teoria/tabla-demonio-var.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento persistente del Servidor</b></p><p><code>/usr/var/log/audit </code></p>
![RUN](image://teoria/tabla-demonio-run.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento volátil del Servidor</b> (desaparece tras cada reinicio)</p><p><code>/run/audit.pid</code></p>
![PROC](image://teoria/tabla-demonio-ram.png?classes=float-left,demonio&resize=100,100) | <p><b>Disco virtual que el kernel crea en RAM</b></p><p><code>/proc/$(cat /run/audit.pid)</code></p>
![IPC](image://teoria/tabla-demonio-ipc.svg?classes=float-left,demonio&resize=100,100) | <p><b>Sockets InterProcess Communication </b></p><pre class="language-bash"><code>ss -xpln &#124; grep audit</code></pre></p>
![NET](image://teoria/tabla-demonio-tcp.png?classes=float-left,demonio&resize=100,100) | <p><b>Sockets TCP/UDP </b></p><p><pre class="language-bash"><code>ss -tupln &#124; grep audit</code></pre></p>
[/div]

### Configuración
[div class="table table-striped"]
FICHERO           |  APLICAR CAMBIOS    | DESCRIPCIÓN		                                   
------------------|---------------------|----------------------------------------------
<p><code>/etc/audit/auditd.conf</code></p><p>Ayuda: <code>man auditd.conf</code></p>| <pre class="language-bash"><code>systemctl reload auditd</code></pre>        | Algunos parámetros importantes: <ul><li><b>write_logs:</b>*{yes/no}* trazas persistentes o efímeras.</li><li><b>log_file:</b> ruta fichero de log.</li><li><b>flush:</b>*{none,incremental,incremental_async,data,sync}* algoritmo de eliminación logs viejos.</li><li><b>num_logs:</b> número de logs que se almacenan.</li><li><b>max_log_file:</b>(Megabytes) tamaño máximo log.</li>><li><b>max_log_file_action:</b>*{ignore,syslog,suspend,rotate,keep_logs}* qué hacer cuando se llena log.</li></ul>
<p><code>/etc/audit/auditd.rules</code></p><p><code>/etc/audit/rules.d</code></p>       | <pre class="language-bash"><code>systemctl reload auditd</code></pre>| Reglas de auditoría persistentes. Existen reglas de ejemplo en <code>/usr/share/audit/sample-rules</code>.
[/div]

### Herramientas de Trabajo
[div class="table table-striped demonio"]
COMANDO                   |  OBJETIVO     | DESCRIPCIÓN	                                   
--------------------------|-------------------------------------------------------------------------------
<code>auditctl</code>|Reglas de Auditoría Efímeras | Se utiliza para configurar las opciones del kernel relacionadas con la auditoría, para ver el estado de la configuración y para cargar reglas de auditoría discrecionales.
<code>augenrules</code>|Reglas de Auditoría Persistentes| Script que fusiona todos los archivos de reglas de auditoría de componentes, que se encuentran en el directorio de reglas de auditoría, <code>/etc/audit/rules.d</code>, colocando el archivo combinado en <code>/etc/audit/audit.rules</code>. Los archivos de reglas de auditoría de componentes deben terminar en .rules para poder ser procesados. Todos los demás archivos en <code>/etc/audit/rules.d</code> se ignoran.
<code>ausearch</code>|Búsqueda Trazas Auditoría| Puede consultar los registros del demonio de auditoría en función de eventos según diferentes criterios de búsqueda. La utilidad ausearch también puede recibir información de la entrada estándar siempre que la entrada sean datos de registro sin procesar. Cada opción de línea de comando dada forma una declaración "y". Por ejemplo, buscar con -m y -ui significa devolver eventos que tienen el tipo solicitado y coinciden con la identificación de usuario proporcionada. Una excepción son las opciones -m y -n; Se permiten varios tipos de registros y nodos en una búsqueda que devolverá cualquier nodo y registro coincidentes.
<code>aureport</code>|Generación de Informes Auditoría| Produce informes resumidos de los registros del sistema de auditoría. La utilidad aureport también puede recibir información de la entrada estándar siempre que la entrada sean datos de registro sin procesar. Los informes tienen una etiqueta de columna en la parte superior para ayudar con la interpretación de los distintos campos. Excepto el informe resumido principal, todos los informes tienen el número de evento de auditoría. Posteriormente puede buscar el evento completo con ausearch -un número de evento. Es posible que deba especificar las horas de inicio y finalización si obtiene varias visitas. Los informes producidos por aureport se pueden utilizar como componentes básicos para análisis más complicados.
<code>aulast</code>|Actividad de Accesos| Imprime una lista de los últimos usuarios que iniciaron sesión de manera similar al programa last y lastb.
<code>autrace</code>|Reglas de auditoría para rastrear llamadas al sistema de un proceso, similar a strace| Este comando elimina todas las reglas de auditoría antes de ejecutar el programa de destino y después de ejecutarlo. Como medida de seguridad, no se ejecutará a menos que se eliminen todas las reglas con auditctl antes de su uso.
<code>auditconfig </code>|Configuraciones de auditoria| Permite localizar posibles valores de parámetros auditoría.
[/div]

### Diseñando un Sistema de Auditoría: Índice de Comandos
Todos los flags pueden combinarse.

#### Crear Reglas Efimeras
<pre class="language-bash"><code>
auditctl

-p {rule} { r=read, w=write, x=exit, a=attribute} Agrega permisos a la regla.
          Ejm.: auditctl -p wa -w /etc/passwd -k password_change
-k {key} Agrega una clave a una regla efímera.
-a {l|a} Agregar regla efímera al final de la lista con una cierta acción. 
          l={task, exit,user,exclude,filesystem}
          a={never,always}
          Ejm.: auditctl -a never,exit -S all -F dir=/usr/sbin/ausearch -S {syscall} 
		        Agregar regla a una llamada del sistema (nombre o número, All=todas). 
                  ausyscall --dump para ver los posible números.
-w <path> Agrega regla efímera que vigile un fichero
-F        Agregar campo a una regla (dir, key). man auditctl para ver opciones.			 
-b 8192   Máximo tamaño para el buffer del kernel.
-f 1      {0=silent 1=printk 2=panic} 
          Como informar al núcleo cuando un problema con AuditD ha sido detectado. 
-e 2      {0=habilita, 1=deshabilita, 2=bloquea} 
          Controlar el estado AuditD.
-r 0      Límite de logs registrados por segundo 0 establece sin límite.
-s        Estado del demonio.
-l        Lista las reglas actuales en uso.
-D        Borra todas las reglas en uso.
-W        Borrar una regla relativa al sistema de ficheros. Debe especificarse siempre todos los parámetros, 
            ej. "auditctl -W /usr/bin -p x -k sbin_monitor"
-d        Borra las reglas definidas mediante llamadas al sistema: 
            ej. auditctl -d always,exit  -F arch=b64 -S unlink -S unlinkat -S rename-S renameat -F uid=0  -k sbin_monitor
-R /usr/share/doc/audit-version/stig.rules   Permite definir una ruta a un fichero con reglas.
</code></pre>

#### Cargar Fichero Reglas Persistentes
<pre class="language-bash"><code>
augenrules --check # Comprueba si hay reglas cambiadas o nuevas en /etc/audit/rules.d y se necesita generar de nuevo audit.rules.
augenrules --load  # Refresca las reglas del kernel desde el fichero audit.rules.
</code></pre>

#### Consultar Trazas: AUSEARCH
<pre class="language-bash"><code>
ausearch 
 -k <key>	# Busca por llave (nombre de la regla / grupo de reglas).
 -p <pid>	# Buscar por identificador de proceso.
 -c <comando>	# Busca todos los eventos que tengan que ver con el comando.ausearch -c touch
 -ua <uid>	# Buscar por id de usuario, tanto el efectivo como el login user id (auid).
 -gi <gid>	# Buscar por GID.
 -x <path> 	# Buscar por Path del binario ejecutado (exe="/usr/bin/touch").
 -sc <syscall>	# Buscar por llamada del sistema.
 -sv <yes|no>	# Estado de finalización, si fue exitoso "yes", en caso contrario "no".
 -f file	# Buscar por fichero / directorio.
 -ts <date>
 -te <date>	# Buscar en un intervalo de tiempo. El formato de la fecha se saca con date '+%x' 
</code></pre>

#### Elaborar Informer: AUREPORT
<pre class="language-bash"><code> 
aureport 
  --summary 	# Estadísticas generales (eventos, accesos, procesos, etc). Se puede combinar con otras opciones para realizar   sumatorios. Ejm.: aureport -x --summary
  --success	 # Estadísticas de eventos con resultado exitoso.
  --failed	 # Estadísticas de eventos con resultado fallido.
  -ts <date>
  -te <date> # Informe de un intervalo de tiempo. El formato de la fecha se saca con date '+%x' 
              -ts 10/01/24 00:00:00 -te 17/05/24 00:00:00
  -c	# Informe de cambios en la configuración de auditd.
  -l	# Inform#e de logins en el sistema.
  -p	# Informe de procesos: Fecha, tiempo, id,nombre, syscalls, auid y número de evento.
  -f	# Informe de ficheros: Fecha, tiempo, id,nombre, syscalls, auid y número de evento.
  -u	# Reporte de usuarios: Fecha, tiempo, id,nombre, syscalls, auid y número de evento.
  -s	# Informe de syscalls: Fecha, tiempo, número de llamada, nombre del comando que uso la syscall, auid y número de evento.
</code></pre>

#### Estadística Accesos: AULAST
<pre class="language-bash"><code> 
aulast # Informa de accesos al sistema. Opciones: 
	-f file
    --bad
	--debug
	--stdin
	--proof 
    --extract
	--user name 
	--tty tty
</code></pre>

#### Configuraciones: AUDITCONFIG
<pre class="language-bash"><code> 
auditconfig -lsevent	# Listado de eventos de auditoría
</code></pre>
 
# Diseño de un Sistema de Auditoría  

## Habilitar Servicio
<pre class="language-bash"><code> 
sudo systemctl enable -–now auditd
sudo systemctl status auditd
</code></pre>

## Activar Reglas de Auditoría:
### Reglas Efímeras: AUDITCTL
#### Sintaxis 1: Syscall 
!!! <code>sudo auditctl -a <action,filter> -S <system_call> -F <field=value> -k <key_name></code>
!!! 
!!! + *action*      = {always,never}
!!! + *filter*      = {task, exit, user, exclude}
!!! + *system_call* = { <code>ausycall --dump</code>}
!!! + *field*       = { <code>man auditctl</code> }
!!! + *key_name*    = identificador para búsquedas en las trazas audit
 
#### Sintaxis 2: Watches 
!!! <code>sudo auditctl -w </ruta/al/fichero> -p <permisos> -F <field=value> -k <key_name> </code>
!!!
!!! + *permisos* = {r|w|x|a} = read, write, exec, Access
!!! + *key_name* = identificador para búsquedas en las trazas audit
!!! + *field*    = { <code>man auditctl</code> } , normalmente usuarios y grupos de ficheros.

#### Caso 1: Diagnósticos
###### Ejemplo 1: Observar accesos y modificaciones de ficheros
*Sintaxis 1 – Lectura y Escritura de Ficheros:*
<pre class="language-bash"><code> 
sudo auditctl -w /etc/passwd -p wa -k passwd_changes 
sudo auditctl –l
sudo ausearch –k passwd_changes # Buscar trazas auditoría
sudo auditctl –D                # Borrar reglas efimeras 
</code></pre>

*Sintaxis 1 – Lanzamientos de un Ejecutable:*
<pre class="language-bash"><code> 
sudo auditctl -w /ruta/al/ejecutable -p x -k executable_shoots 
sudo auditctl –l
sudo ausearch –k executable_shoots # Buscar trazas auditoría
sudo auditctl –D                   # Borrar reglas efimeras  
</code></pre>

###### Ejemplo 2: Estadísticas de Acceso a las rutas del Sistema
*Sintaxis 1 – Accesos a los directorios*
<pre class="language-bash"><code> 
sudo auditctl -w /ruta/al/directorio -p wa -k directorio_access 
sudo auditctl –l
sudo ausearch –k directorio_access  # Buscar trazas auditoría
sudo auditctl –D                    # Borrar reglas efimeras  
</code></pre>

###### Ejemplo 3: Análisis de Llamadas al Sistemas (Syscall)
![Contexto Audit](image://teoria/teoria-audit-sudo-abuse.jpg)

*Sintaxis 2 – Usuario accede a un fichero de otro usuario empleando el comando sudo*
<pre class="language-bash"><code> 
auditctl - a exit,always -F dir=/home -F euid=0 -C auid!=obj_uid -k sudoAbuse
sudo auditctl –l
sudo ausearch –k sudoAbuse   # Buscar trazas auditoría
sudo auditctl –D             # Borrar reglas efimeras    
</code></pre>

!!! + -F dir/home: acceso a fichero en /home
!!! + -F euid=0: fichero accedido bajo el contexto UID=0 (usuario root)
!!! + -C auid != obj_uid: propietario del fichero  (obj_uid) no es usuario que accede al fichero (auid) 

*Sintaxis 2 – Errores e intentos de apertura de ficheros:*
<pre class="language-bash"><code> 
sudo auditctl -a always,exit -S open -F arch=b64 -F success=0 -k open_errors
sudo auditctl –l
sudo ausearch –k open_errors  # Buscar trazas auditoría
sudo auditctl –D              # Borrar reglas efimeras    
</code></pre>

*Sintaxis 2 – Apertura de Sockets*
<pre class="language-bash"><code> 
sudo auditctl -w /var/log/faillock -p wa -k logins
sudo auditctl –l
sudo ausearch –k logins   # Buscar trazas auditoría
sudo auditclt –D          # Borrar reglas efimeras      
</code></pre>

- En el abanico de módulo que tiene PAM, el módulo de autenticación pam_faillock de registrar los accesos fallidos. 
- La auditoría agrega a esta información detalles de los usuarios que intentaron iniciar sesión. 

#### Caso 2: Auditoría de Seguridad
###### Ejemplo 1: Análisis de la Actividad de Red

*Sintaxis 1 – Actividad del Demonio PAM (Linux Pluggable Authentication Modules):*
<pre class="language-bash"><code> 
auditctl -a always,exit -F arch=b64 -S getsockopt –k getsockopt
sudo auditctl –l
sudo ausearch –k getsockopt  # Buscar trazas auditoría
sudo auditctl –D             # Borrar reglas efimeras      
</code></pre>

+ Puede aplicarse sobre nftable, iptable y ebtables.

### Reglas Persistentes: AUGENRULES

-	Las reglas efímeras se transforman en persistentes guardándolas en el fichero /etc/audit/auditd.rules
-	La sintaxis de este fichero, es la de auditctl, pero omitiendo la llamada al comando:

#### Regla efímera con auditctl
<pre class="language-bash"><code> 
sudo auditctl -w /ruta/al/ejecutable -p x -k executable_shoots 
sudo auditctl –l
sudo ausearch –k executable_shoots # Buscar trazas auditoría
sudo auditctl –D                   # Borrar reglas efimeras       
</code></pre>

#### Regla persistente en <code>/etc/audit/auditd.rules</code>
##### Transformar Regla Efímera en Persistente
<pre class="language-bash"><code> 
$ vi /etc/audit/audit.rules
-w /ruta/al/ejecutable -p x -k executable_shoots

$ sudo ausearch –k executable_shoots # Buscar trazas auditoría     
</code></pre>

##### Cargar Conjunto de Reglas preestablecidas
- **Las reglas persistentes sueles configurarse de manera modular en <code>/etc/audit/rules.d/</code>** 
    - Lo módulos permiten capitalizar y reutilizar conjuntos de reglas bien consolidadas. 
    - Comando augenrules para activarlas, las agregará al fichero <code>/etc/audit/auditd.rules</code> 
- **En RedHat hay conjuntos de reglas bien probadas en la carpeta <code>/usr/share/audit</code>** 
    - Los prefijos indican la categoría de cada conjunto de reglas

[div class="table table-striped"]
PREFIJO   |  FAMILIA DE REGLAS DE AUDITORÍA		                                   
----------|-------------------------------------------------------------------
10	      | Configuración del kernel y auditctl 
20	      | Reglas que podrían coincidir con las reglas generales pero que usted quiere que coincidan de otra manera 
30	      | Reglas principales 
40	      | Reglas opcionales 
50	      | Reglas específicas del servidor  
70	      | Reglas locales del sistema 
90	      | Finalizar (inmutable)
[/div]

###### Ejemplo 1: Agregar reglas STIG
*Configuración de la regla de auditoría que cumple con los requisitos establecidos por las Guías Técnicas de Implementación de Seguridad (STIG).*

+ PASO 1 - Se pueden copiar reglas de ejemplo del directorio <code>/usr/share/audit/sample-rules</code>:
<pre class="language-bash"><code> 
sudo cp /usr/share/audit/sample-rules/30-stig.rules /etc/audit/rules.d/   
</code></pre>

+ PASO 2 - Se agregan al fichero de reglas persistentes se la siguiente manera
<pre class="language-bash"><code> 
$ sudo augenrules --check
/usr/sbin/augenrules: Rules have changed and should be updated

$sudo augenrules –load 
</code></pre>

+ PASO 3 - Las comprobaciones
<pre class="language-bash"><code> 
// 1.- ¿Están activas?
$ sudo auditctl –l

// 2.- ¿Son persistentes?
$  sudo cat /etc/audit/audit.rules
</code></pre>

## Consultando Trazas

### Búsquedas: AUSEARCH
Ausearch agrupa las trazas por evento, separarando eventos por <code>---</code>. Esto facilita interpretar las trazas de auditoría. 

*Filtrar por clave de evento:*
<pre class="language-bash"><code> 
sudo ausearch -k passwd_changes   
</code></pre>

*Filtrar por rango de tiempo:*
<pre class="language-bash"><code> 
// 1.- Localizar Formato de la fecha
[root@UCS01 ~]# date '+%x'
18/05/24 

// 2.- Filtrar por rango de tiempo con este formato de fecha
[root@UCS01 ~]# sudo ausearch -ts 10/01/24 00:00:00 -te 17/05/24 00:00:00
</code></pre>

### Informes: AUREPORT
*Actividad del día anterior:*
<pre class="language-bash"><code>aureport -ts yesterday</code></pre>

*Resumen de toda la actividad:*
<pre class="language-bash"><code>aureport --summary</code></pre>

### Actividad: AULAST
*Ayuda:*
<pre class="language-bash"><code>
$ aulast -h
usage: aulast [--bad] [--debug] [--stdin] [--proof] [--extract] [-f file] [--user name] [--tty tty]
</code></pre>

*Resumen Actividad:*
<pre class="language-bash"><code>
sudo aulast  --extract
</code></pre>

## Interpretación de la Trazas  

<a href='https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/security_hardening/auditing-the-system_security-hardening#linux-audit_auditing-the-system' target=_blank>https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/security_hardening/auditing-the-system_security-hardening#linux-audit_auditing-the-system</a> 

### 	AUDITD: CAPTURA EVENTOS POR SOCKET NETLINK

#### REGISTRO TIPO SYSCALL

+	Observación 1: Lista de campos, cada uno con formato key=value.
+	Observación 2: Este evento ocupa varias líneas de LOG.
+	Observación 3: Los eventos pueden intercalarse y llegar de manera desordenada.
+	Observación 4: Comando auserch -i, ausearch --interpreter para ver campos en formato humano

<pre class="language-bash"><code>
$ cat /var/log/audit/audit.log
type=SYSCALL msg=audit(1364481363.243:24287): arch=c000003e syscall=2 success=no exit=-13 a0=7fffd19c5592 a1=0 a2=7fffd19c4b50 a3=a items=1 ppid=2686 pid=3538 auid=1000 uid=1000 gid=1000 euid=1000 suid=1000 fsuid=1000 egid=1000 sgid=1000 fsgid=1000 tty=pts0 ses=1 comm="cat" exe="/bin/cat" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="sshd_config"

# 1.- INTERPRETACIÓN
# Campo 1: Type=EVENT_TYPE, qué tipo de evento provoca esta entrada
# Lista Tipos: https://access.redhat.com/articles/4409591#audit-record-types-2 
type=SYSCALL 

# Campo 2: msg=audit(TIMESTAMP:ID):, mensaje de auditoría
#    - Timestap en formato Unix (segundos desde el 1 Enero de 1970)   
#    - Cada tipo de mensaje tiene su colección de campos
msg=audit(1364481363.243:24287)

# Mensajes tipo SYSCALL, campos asociados
arch=c000003e # Arquitectura de CPU del sistema en Hexadecimal
syscall=2     # Numero llamada al sistema, el listado ausyscall --dump
success=no    # Operación de la llamada al sistema tuvo éxito
exit=-13      # Código de salida de la llamada al sistema
a0=7fffd19c5592, a1=0, a2=7fffd19c5592, a3=a # Argumentos llamada al sistema
items=1       # Núm. registros auxiliares PATH tras el registro de llamada al sistema
ppid=2686     # Parent Process ID del proceso que emitió el evento (ejm, Bash)
pid=3538      # Process ID del proceso que emitió el evento (ejm, comando ‘cat’)
auid=1000     # Audit User ID 
uid=1000      # User ID del proceso que envía el evento
gid=1000      # Group ID del usuario el inicio el proceso de análisis
euid=1000     # Effective User ID, el usuario que inició el proceso de análisis.
suid=1000     # Set User ID, el usuario que inició el proceso de análisis.
fsuid=1000    # File System User ID, el usuario que inició el proceso de análisis.
egid=1000     # Effective Group ID, el usuario que inició el proceso de análisis.   
sgid=1000     # Set Group ID, el usuario que inició el proceso de análisis.
fsgid=1000    # File System Group ID, el usuario que inició el proceso de análisis.
tty=pts0      # Terminal desde el que fue lanzado el proceso
ses=1         # Session ID del proceso que generó el evento
comm="cat"    # Comando del proceso que generó el evento
exe="/bin/cat" # Ejecutable del Comando del proceso que generó el evento
subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 # Contexto SELinux
key="sshd_config" # Clave Usuario para identificar el Evento
</code></pre>

#### REGISTRO TIPO CWD (Current Working Directory)

+ Observación 1: Descripción de la Ruta de Trabajo del proceso que lanzó el evento Audit

<pre class="language-bash"><code>
$ cat /var/log/audit/audit.log
type=CWD msg=audit(1364481363.243:24287):  cwd="/home/shadowman"

# 1.- INTERPRETACIÓN
# Campo 1: Type=EVENT_TYPE, qué tipo de evento provoca esta entrada
# Lista Tipos: https://access.redhat.com/articles/4409591#audit-record-types-2 
type=CWD 

# Campo 2: msg=audit(TIMESTAMP:ID):, mensaje de auditoría
#    - Timestap en formato Unix (segundos desde el 1 Enero de 1970)   
#    - Cada tipo de mensaje tiene su colección de campos
msg=audit(1364481363.243:24287)

# Mensajes tipo CWD, campos asociados
cwd="/home/user_name" # Directorio desde el que el proceso lanzó evento Audit

</code></pre>

#### REGISTRO TIPO PATH 

+	Observación 1: ítems=1 del mensaje SYSCALL, indica cuantos registros PATH tiene la llamada al sistema que se está auditando.
+	Observación 2: Ruta de los argumentos que tiene la llamada al sistema.

<pre class="language-bash"><code>
$ cat /var/log/audit/audit.log
type=PATH msg=audit(1364481363.243:24287): item=0 name="/etc/ssh/sshd_config" inode=409248 dev=fd:00 mode=0100600 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:etc_t:s0  objtype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0

# 1.- INTERPRETACIÓN
# Campo 1: Type=EVENT_TYPE, qué tipo de evento provoca esta entrada
# Lista Tipos: https://access.redhat.com/articles/4409591#audit-record-types-2 
type=PATH 

# Campo 2: msg=audit(TIMESTAMP:ID):, mensaje de auditoría
#    - Timestap en formato Unix (segundos desde el 1 Enero de 1970)   
#    - Cada tipo de mensaje tiene su colección de campos
msg=audit(1364481363.243:24287)

# Mensajes tipo PATH, parámetros de una llamada al sistema
item=0 # Índice del campo tipo PATH asociada a la llamada al sistema
name="/etc/ssh/sshd_config" # Parámetro de la llamada al sistema
inode=409248 # find / -inum 409248 -print => /etc/ssh/sshd_config
dev=fd:00    # Numero mayor y menos del dispositivo donde está inodo: /dev/fd/0 => /dev/sda
mode=0100600 # Permisos fichero, en este caso -rw-------
ouid=0       # El ID de usuario del propietario del objeto
ogid=0       # El ID de grupo del propietario del objeto
rdev=00:00   # El ID de dispositivo grabado sólo para archivos especiales (no en este caso)
obj=system_u:object_r:etc_t:s0 # Contexto SELinux
objtype=NORMAL # Intención de parámetro de la llamada al sistema
cap_fp=none    # Config. Tamaño permitido según sistema de archivos 
cap_fi=none    # Config. Capacidad heredada según sistema de archivos 
cap_fe=0       # Bit efectivo según sistema de archivos 
cap_fver=0     # Versión del sistema de archivos 
</code></pre>

#### REGISTRO TIPO PROCTITLE

+	Observación 1: Codificación del comando que lanzó el evento en formato Hexadecimal.

<pre class="language-bash"><code>
$ cat /var/log/audit/audit.log
type=PROCTITLE msg=audit(1364481363.243:24287) : proctitle=636174002F6574632F7373682F737368645F636F6E666967

# 1.- INTERPRETACIÓN
# Campo 1: Type=EVENT_TYPE, qué tipo de evento provoca esta entrada
# Lista Tipos: https://access.redhat.com/articles/4409591#audit-record-types-2 
type=PROCTITLE 

# Campo 2: msg=audit(TIMESTAMP:ID):, mensaje de auditoría
#    - Timestap en formato Unix (segundos desde el 1 Enero de 1970)   
#    - Cada tipo de mensaje tiene su colección de campos
msg=audit(1364481363.243:24287)

# Mensajes tipo PROCTITLE, campos asociados
#  - Codificación comando lanzado en hexadecimal, por motivos de seguridad
#  - En este ejemplo, se trata del comando ‘cat /etc/ssh/sshd_config’)
proctitle=636174002F6574632F7373682F737368645F636F6E666967

</code></pre>