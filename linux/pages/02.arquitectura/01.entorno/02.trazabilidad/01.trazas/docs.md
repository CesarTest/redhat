---
title: 2.1.Trazas
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

# Arquitectura
El subsistema de trazas de Linux tiene dos componentes:
+ **Registro Eventos:** el diario de sistema (journal) registra cada acción que realiza el demonio sistema, demonio que se ocupa de controlar el ciclo de vida de cada programa del sistema operativo. Además de la actividad del demonio de sistema, cada demonio tiene su propia traza que describe su actividad, en muchos casos el diario también da acceso a estas trazas. Las trazas del diario de sistema son efímeras en su configuración por defecto (desaparecen tras cada reinicio) y se guardan en binario por motivos de seguridad.
+ **Comunicaciones:** el demonio rsyslog recibe las trazas del diario de sistema, le aplica una serie de filtros y el resultado lo guarda en  <code>/var/log</code> de manera persistente. Diariamente se lanza el proceso <code>lorotate</code> que rota las trazas para que limitar el espacio que ocupan las trazas en disco. Es muy sencillo configurar el demonio Syslog para que transmita las trazas a un servidor central de trazas.
![Subsistema Trazas](image://teoria/teoria-entorno-trazas.jpg)

# Demonio JournalD

## ¿Para qué qué JournalD? Trazado Centralizado de Eventos Indexados en Binario.
![Journal](image://teoria/trazas-journal.jpg?classes=float-left,shadow&resize=200,250) 
**DIARIO DEL SISTEMA**
+ Al reemplazar el gestor del entorno de ejecución de Init a SystemD para un control más centralizado, surge la necesidad de un nuevo demonio para un registro centralizado de los eventos del sistema operativo.
+ Se recolecta mensajes de eventos desde muchas fuentes, incluido el kernel, la salida de las primeras etapas del proceso de arranque, la salida estándar y el error estándar de los daemons cuando se inician y se ejecutan, y los eventos de syslog. 
+ Luego, los reestructura en un formato estándar y los escribe en un diario de sistema indexado, estructurado y en binario por motivos de seguridad. 
 
## Anatomía
[div class="table table-striped demonio"]
 ICONO                                                   | RUTA		                                   
---------------------------------------------------------|---------------------------------------------
![EXE](image://teoria/tabla-demonio-exe.png?classes=float-left,demonio&resize=100,100) | <p><b>Ejecutable del Servidor</b></p><p></br><code>/usr/lib/systemd/systemd-journald</code></p>     
![ETC](image://teoria/tabla-demonio-cnf.png?classes=float-left,demonio&resize=100,100) | <p><b>Configuraciones del Servidor</b></p> <p> <code>/etc/systemd/journal.conf</code></p>
![LIB](image://teoria/tabla-demonio-inmutable.svg?classes=float-left,demonio&resize=100,100) | <p><b>Código fuente y documentación del Servidor</b></p><p><code>/lib/systemd</code></p>    
![VAR](image://teoria/tabla-demonio-var.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento persistente del Servidor</b></p><p><code>/usr/lib/systemd</code></p>
![RUN](image://teoria/tabla-demonio-run.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento volátil del Servidor</b> (desaparece tras cada reinicio)</p><p><code>/run/systemd/journal</code></p><p><code>/run/log/journal</code></p>
![PROC](image://teoria/tabla-demonio-ram.png?classes=float-left,demonio&resize=100,100) | <p><b>Disco virtual que el kernel crea en RAM</b></p><p><code>/proc/1</code></p>
![IPC](image://teoria/tabla-demonio-ipc.svg?classes=float-left,demonio&resize=100,100) | <p><b>Sockets InterProcess Communication </b></p><pre class="language-bash"><code>ss -xpln &#124; grep journald/code></pre></p>
![NET](image://teoria/tabla-demonio-tcp.png?classes=float-left,demonio&resize=100,100) | <p><b>Sockets TCP/UDP </b></p><p><pre class="language-bash"><code>ss -tupln &#124; grep journald</code></pre></p>
[/div]
 
## Configuración
<table class="table table-striped">
  <thead>
    <th>FICHERO</th>
	<th>APLICAR CAMBIOS</th>
	<th>DESCRIPCIÓN</th>	
  </thead>
  <tbody>
    <tr> 
	  <td><p><code>/etc/systemd/journald.conf</code></p><p>Ayuda: <code>man journald.conf</code></p></td>
	  <td><pre class="language-bash"><code>reboot</code></pre></td>
	  <td>
        Las opciones más importantes son las siguientes: 
        <ul>
         <li><b>storage:</b> Controla dónde almacenar los datos del diario. Uno de "volatile", "persistent", "auto" and "none". El valor predeterminado es "automático". 
            <ul>
	            <li><i>"volatile"</i>, los datos del registro de diario se almacenarán sólo en la memoria. Por ej..: debajo del /run/log/journal jerarquía (que se crea si es necesario).</li>
	            <li><i>"persistent”</i>, los datos se almacenarán preferiblemente en el disco. Por ej..: debajo de la jerarquía /var/log/journal (que se crea si es necesario), con un respaldo a /run/log/journal (que se crea si es necesario), durante el inicio temprano y si el disco no se puede escribir. </li>
	            <li><i>"auto"</i> es similar a "persistente", pero el directorio /var/log/journal no se crea si es necesario, de modo que su existencia controla dónde van los datos de registro</li>
	       	 <li><i>"none"</i> desactiva todo el almacenamiento y se eliminarán todos los datos de registro recibidos. Sin embargo, el reenvío a otros destinos, como la consola, el búfer de registro del kernel o un socket syslog seguirá funcionand</li>
	          </ul>
	       </li>
          <li><b>SystemMaxUse=, SystemKeepFree=, SystemMaxFileSize=, SystemMaxFiles=, RuntimeMaxUse=, RuntimeKeepFree=, RuntimeMaxFileSize=, RuntimeMaxFiles=</b> Aplica límites de tamaño a los archivos de diario almacenados
         </li>
        </ul>	  
	 </td>
	</tr>
  </tbody>
</table>
 
## Herramientas de Trabajo
[div class="table table-striped"]
COMANDO                   |  OBJETIVO                                     | DESCRIPCIÓN	                                   
--------------------------|-------------------------------------------------------------------------------
<code>journalctl</code>| Lectura trazas | El servicio systemd-journald almacena datos de registro en un archivo binario estructurado e indexado, que se denomina diario (journal). Estos datos incluyen información adicional sobre el evento de registro.
La clave para usar en forma correcta el diario (journal) para la solución de problemas y auditorías es limitar las búsquedas en el diario (journal) para mostrar solo la salida relevante.
[/div]
 
## Filtrando Actividad del Entorno de Ejecución: Índice de Comandos
Todos los filtros, tanto de entrada como salida, puden combinarse.

### Entrada: Origen de la traza

[div class="table table-striped demonio"]
COMANDO                                                              |  OBJETIVO                                           
---------------------------------------------------------------------|---------------------------------------------------
<pre class="language-bash"><code>journalctl -f</code></pre>   |Eventos en tiempo real
<pre class="language-bash"><code>journalctl -p {level}</code></pre> | Eventos de Prioridad:<ul><li>0=emerg</li><li>1=alert</li><li>2=crit</li><li>3=err</li><li>4=warning</li><li>5=notice</li><li>6=info</li><li>7=debug</li></ul>  
<pre class="language-bash"><code>journalctl -b</code></pre>   | Eventos del arranque
<pre class="language-bash"><code>journalctl -u {unit}</code></pre>  | Eventos de una Unit
<pre class="language-bash"><code>journalctl -k</code></pre> |Eventos del kernel
<pre class="language-bash"><code>journalctl --since <date> --until {date}</code></pre> | Eventos desde/hasta:<ul><li>“yesterday”</li><li>“2 days ago”</li><li>“1 hour ago”</li><li>“09:00”</li><li>“2024-05-17 13:15:30”</li></ul>
<pre class="language-bash"><code>journalctl {origen}={valor}</code></pre> | Eventos cuyo origen puede ser:<ul><li>_PID (proceso) </li> <li>_UID (usuario)</li><li>_GID (grupo usuarios)</li><li>_HOSTNAME</li><li>_COMM (comando)</li><li>_SELINUX_CONTEXT</li></ul>
<pre class="language-bash"><code>journalctl --flush</code></pre> | Elimina todos los ficheros de journal
<pre class="language-bash"><code>journalctl --rotate</code></pre> | Fuerza rotado de ficheros journal
<pre class="language-bash"><code>journalctl --vacuum-size={s} --vacuum-time={t} --vacuum-files={f}</code></pre> | Elimina los archivos de diario archivados más antiguos hasta el umbral especificado (tamaño, tiempo o número de ficheros). 
[/div]

### Salida: Formato de Presentación
[div class="table table-striped demonio"]
COMANDO                                                              |  OBJETIVO                                           
---------------------------------------------------------------------|---------------------------------------------------
<pre class="language-bash"><code>journalctl -F</code></pre> |Eventos en tiempo real, deteniendo la salida cuando se detiene la entrada.
<pre class="language-bash"><code>journalctl -o {format}</code></pre> | Formato Salida:<ul><li>short</li><li>short-full</li><li>verbose</li><li>json-pretty</li><li>export</li></ul>  
<pre class="language-bash"><code>journalctl -x</code></pre> | Formato Extendido.
<pre class="language-bash"><code>journalctl -r</code></pre> | Formato Orden Inverso
<pre class="language-bash"><code>journalctl -t</code></pre> |Formato TImeStamp
<pre class="language-bash"><code>journalctl -n {num}</code></pre>| Formato con Número limitado de Eventos
<pre class="language-bash"><code>journalctl -e</code></pre> |Formato con todos los Eventos, incluso los eliminados
[/div]
 
# Demonio Rsyslog
 
## ¿Para qué qué JournalD? Persistencia de Trazas en /var/log
![Rsyslog](image://teoria/trazas-rsyslog.jpg?classes=float-left,shadow&resize=200,250)
El segundo de los demonios que tiene el subsistema de logging es Rsyslog, que se ocupa de la comunicación de eventos a otros elementos.
Tal como se ha mencionado anteriormente, una de las características de journald es que, en su configuración por defecto, las trazas se almacenan de forma volátil. 
+ **Rsyslog está configurado para recibir los eventos de journald y transferirlos a ficheros en /var/log**, logrando así tanto persistencia como compatibilidad hacia atrás. Rsyslog se basa en filtros, así que las trazas resultantes tienes menos nivel de detalles que las de JournalD
+ Además, <u>Rsylog permite enviar los registros a un servidor de monitorización externo</u>, basta configurar el demonio Rsyslog en un extremo como servidor y en el otro como cliente.
 
## Anatomía
 [div class="table table-striped demonio"]
 ICONO                                                   | RUTA		                                   
---------------------------------------------------------|---------------------------------------------
![EXE](image://teoria/tabla-demonio-exe.png?classes=float-left,demonio&resize=100,100) | <p><b>Ejecutable del Servidor</b></p><p></br><code>/sbin/rsyslogd</code></p>     
![ETC](image://teoria/tabla-demonio-cnf.png?classes=float-left,demonio&resize=100,100) | <p><b>Configuraciones del Servidor</b></p> <p> <code>/etc/rsyslog.conf</code></p> <p> <code>/etc/rsyslog.d</code></p>
![LIB](image://teoria/tabla-demonio-inmutable.svg?classes=float-left,demonio&resize=100,100) | <p><b>Código fuente y documentación del Servidor</b></p><p><code>/usr/lib64/rsyslog</code></p>    
![VAR](image://teoria/tabla-demonio-var.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento persistente del Servidor</b></p><p><code>/var/lib/rsyslog</code></p><p><code>/var/log</code></p>
![RUN](image://teoria/tabla-demonio-run.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento volátil del Servidor</b> (desaparece tras cada reinicio)</p><p><code>/run/rsyslog.pid</code></p>
![PROC](image://teoria/tabla-demonio-ram.png?classes=float-left,demonio&resize=100,100) | <p><b>Disco virtual que el kernel crea en RAM</b></p><p><pre class="language-bash"><code>/proc/$(ps -ef &#124; grep rsyslogd &#124; grep -v grep &#124; awk '{print $2}')</code></pre></p>
![IPC](image://teoria/tabla-demonio-ipc.svg?classes=float-left,demonio&resize=100,100) | <p><b>Sockets InterProcess Communication </b></p><pre class="language-bash"><code>ss -xpln &#124; grep rsyslog</code></pre></p>
![NET](image://teoria/tabla-demonio-tcp.png?classes=float-left,demonio&resize=100,100) | <p><b>Sockets TCP/UDP </b></p><p><pre class="language-bash"><code>ss -tupln &#124; grep rsyslog</code></pre></p>
[/div]

## Configuración
[div class="table table-striped demonio"]
FICHERO                   |  APLICAR CAMBIOS                             | PARÁMETROS PRINCIPALES	                                   
--------------------------|-------------------------------------------------------------------------------
<code>/etc/rsyslog.conf</code>| <pre class="language-bash"><code>systemctl restart rsyslog</code></pre> |  <pre class="language-bash"><code>man rsyslog.conf</code></pre>
[/div]
 
 
## Herramientas de Trabajo
[div class="table table-striped demonio"]
COMANDO                           |  OBJETIVO                | DESCRIPCIÓN	                                   
----------------------------------|-------------------------------------------------------------------------------
<code>rsyslogd</code>             | Trazas Persistente       | Demonio que vuelca el journal a trazas en <code>/var/log</code>
<code>rsyslog-recover-qi.pl</code>| Control colas de Rsyslog | <a href="https://www.rsyslog.com/doc/concepts/queues.html" target="_blank">https://www.rsyslog.com/doc/concepts/queues.html</a> 
[/div]
  
## Índice de LOGS más Importantes
[div class="table table-striped demonio"]
RUTA                           |  OBJETIVO                | DESCRIPCIÓN	                                   
----------------------------------|-------------------------------------------------------------------------------
<pre class="language-bash"><code>/var/log/messages</code></pre>| Mensajes Generales del Demonio de Sistema  |  <ul><li>Arranque y apagado del sistema</li><li>Información sobre hardware detectado</li><li>Mensajes del kernel</li></ul>
<pre class="language-bash"><code>/var/log/anaconda</code></pre>| Logs de Instalación  |  <ul><li>Mensajes generales del sistema</li><li>Información sobre la actividad de red</li><li>Mensajes relacionados con el entorno de escritorio</li></ul>
<pre class="language-bash"><code>/var/log/secure</code></pre>| Logs de Seguridad  |  <ul><li>Registro de autenticaciones exitosas y fallidas</li><li>Actividad de sudo</li><li>Mensajes relacionados con SELinux</li></ul>
<pre class="language-bash"><code>/var/log/boot.log</code></pre>| Logs de Arranque  |  <ul><li>Registro de mensajes durante el proceso de arranque</li></ul>
<pre class="language-bash"><code>/var/log/dnf.log</code></pre>| Logs de Instalación de Paquetes  |  <ul><li>Registro de instalación, actualización y eliminación de paquetes con dnf</li></ul>
<pre class="language-bash"><code>/var/log/audit/audit.log</code></pre>| Logs de Telemetría  |  <ul><li>Registros de auditoría que contienen información las llamadas de las aplicaciones al kernel con un contexto muy rico</li></ul>
<pre class="language-bash"><code>/run/log/journal/{id}</code></pre>| Logs de SystemD y JournalD  |  <ul><li>Base de datos donde el Diario de Sistema guarda de manera efímera el registro de todos los eventos</li></ul>
<pre class="language-bash"><code>/var/log/{daemon}</code></pre>| Logs de Aplicación |  Muchas aplicaciones tiene sus logs de actividad, como por ejemplo <ul><li>Apache: <code>/var/log/httpd/</code></li><li>MySQL: <code>/var/log/mysql/</code></li><li>GDM: <code>/var/log/gdm</code></li><li>LibvirtD: <code>/var/log/libvirt</code></li><li>Xorg: <code>/var/log/xorg/</code>, <code>/home/{user}/.local/share/xorg/</code></li></ul>
[/div]
---
 
 # Demonio Logrotate
 
 ## ¿Para qué qué Logrotate? Gestión espacio de disco de trazas persistentes
+ **Se trata de un proceso que lanza el demonio cron diariamente** para rotar, comprimir o enviar por email las trazas del sistema. 
+ **La rotación consiste en eliminar las trazas más viejas** cuando éstas superan un cierto umbral de espacio en el disco,  evitando que se sature el espacio de disco existente. La rotación sigue el proceso:
  - Los ficheros de trazas llevan el sufijo .1, .2… hasta .N, siendo el .N el que tiene las trazas más viejas.
  - Cuando el fichero de trazas más recientes supera el tamaño máximo permitido, el demonio eliminará el fichero de trazas más viejas con .N… y renombrará todos los ficheros sumándole +1 a su sufijo. 
  - Las nuevas trazas irán al fichero recién creado, que está vacío.

![Rsyslog](image://teoria/trazas-logrotate.jpg) 
 
 ## Anatomía
  [div class="table table-striped demonio"]
 ICONO                                                   | RUTA		                                   
---------------------------------------------------------|---------------------------------------------
![EXE](image://teoria/tabla-demonio-exe.png?classes=float-left,demonio&resize=100,100) | <p><b>Ejecutable del Servidor</b></p><p></br><code>/sbin/logrotate</code></p>     
![ETC](image://teoria/tabla-demonio-cnf.png?classes=float-left,demonio&resize=100,100) | <p><b>Configuraciones del Servidor</b></p> <p> <code>/etc/logrotate.conf</code></p> <p> <code>/etc/logrotate.d/</code></p><p> <code>/etc/rwtab.d/logrotate</code></p><p> <code>/etc/cron.daily/logrotate</code></p>
![LIB](image://teoria/tabla-demonio-inmutable.svg?classes=float-left,demonio&resize=100,100) | <p><b>Código fuente y documentación del Servidor</b></p><p><code>/usr/share/systemd</code></p><p><code>/usr/share/doc/logrotate</code></p><p><code>/usr/share/licenses/logrotate</code></p>    
![VAR](image://teoria/tabla-demonio-var.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento persistente del Servidor</b></p><p><code>/var/lib/logrotate</code></p>
![RUN](image://teoria/tabla-demonio-run.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento volátil del Servidor</b> (desaparece tras cada reinicio)</p><p><code>/run/cron.pid</code></p>
![PROC](image://teoria/tabla-demonio-ram.png?classes=float-left,demonio&resize=100,100) | <p><b>Disco virtual que el kernel crea en RAM</b></p><p><pre class="language-bash"><code>/proc/$(ps -ef &#124; grep cron &#124; grep -v grep &#124; awk '{print $2}')</code></pre></p>
![IPC](image://teoria/tabla-demonio-ipc.svg?classes=float-left,demonio&resize=100,100) | <p><b>Sockets InterProcess Communication </b></p><pre class="language-bash"><code>ss -xpln &#124; grep cron</code></pre></p>
![NET](image://teoria/tabla-demonio-tcp.png?classes=float-left,demonio&resize=100,100) | <p><b>Sockets TCP/UDP </b></p><p><pre class="language-bash"><code>ss -tupln &#124; grep cron</code></pre></p>
[/div]
 
 
 ## Configuración
 [div class="table table-striped demonio"]
FICHERO                   |  APLICAR CAMBIOS                             | PARÁMETROS PRINCIPALES	                                   
--------------------------|-------------------------------------------------------------------------------
<p><code>/etc/logrotate.conf</code></p><p><code>/etc/logrotate.d/<log></code></p>| <pre class="language-bash"><code>reboot</code></pre> |  Define qué hacer con los ficheros de configuración estableciendo límites de espacio de disco. Pueden ubicarse tanto en ficheros separados dentro de la carpeta logrotate.d, como en el fichero logrotate.conf. Ejemplo: <pre class="language-bash"><code>cat /etc/logrotate.d/rsyslog</code></pre>
<p><code>/etc/crontab</code></p><p><code>/etc/cron.daily/logrotate<log></code></p>| <pre class="language-bash"><code>systemctl restart crond</code></pre> | Define cuándo debe lanzarse el proceso. Ejemplo: <pre class="language-bash"><code>cat /etc/crontab</code></pre>
[/div]
  
## Herramientas de Trabajo
[div class="table table-striped demonio"]
COMANDO                 |  OBJETIVO                        | DESCRIPCIÓN	                                   
------------------------|-------------------------------------------------------------------------------
<code>logrotate</code>  | Programar reciclado logs en cron | <pre class="language-bash"><code>man logrotate</code></pre>
<code>logrotated</code> | Activar demonio logrotate        |<pre class="language-bash"><code>man logrotate</code></pre>
[/div] 
 