---
title: 1.2.Demonio Sistema
taxonomy:
    category: docs
highlight:
    enabled: true
    lines: true
---

<style>
   .demonio { 
	   table, tbody, tr, td { padding: 0; height: 0; cellpadding: 0; cellspacing: 0; }
       p, code { padding: 0; height: 0; }
	   pre {padding: 4;}
    }
</style>


# Organizando el Ecosistema de Programas.

En la arquitectura del entorno de ejecución, la segunda de sus piezas es el demonio de sistema, el proceso con PID=1 raíz del árbol de procesos del sistema operativo. Responsable de organizar el ecosistema de programas que compone el entorno de ejecución en subsistemas y niveles de ejecución.

Los niveles de ejecución son colecciones de demonios que representan un estado del entorno de ejecución. Una vez levantados los demonios del nivel de ejecución 1, se pasa a lanzar los demonios del nivel de ejecución 2... así sucesivamente, desde el nivel 0 al nivel 5. El nivel 6 representa el apagado ordenado de demonios.  

Existen dos demonios de sistema principales en Linux:
+ **Init**: demonio de sistema más antiguo, presente en todos los Unix.
+ **SystemD**: demonio de sistema de las últimas entregas de Linux. Nace para una vigilancia centralizada del ciclo de vida de todos los demonios del entorno ejecución, eliminando la necesidad de tener múltiples demonios auxiliares para estas tareas de monitorización y control que Init no hace. Se trata de un clon de LaunchD de MacOS que se concibió para reducir el consumo de batería en los móviles de Apple.

Tres son los parámetros a configurar en los demonios de sistema:
1. **Las unidades de ejecución**: dónde se guardan y con qué sintaxis se expresa cómo controlar el ciclo de vida de un programa.
2. **La estructura del arranque**: cómo se agregan o se quitan unidades de ejecución a los distintos niveles de ejecución.
3. **Nivel de Ejecución por Defecto**: hasta qué nivel de ejecución llega el arranque de este sistema.

![Parámetros](image://teoria/teoria-demonio-sistema-parametros.jpg)

## La Unidad de Ejecución

El demonio de sistema mantiene una base de datos de unidades de ejecución, archivos donde se especifica cómo controlar cada demonio del entorno: cómo arrancarlo, pararlo, reiniciarlo, etc.

El resultado final es que esta base de datos de unidades de ejecución modela el comportamiento de todos los programas del entorno.

+ **Init**: aquí las unidades de ejecución son script bash en <code>/etc/rd.d/init.d</code>.
+ **SystemD**: aquí las unidades de ejecución son ficheros de texto plano en <code>/etc/systemd/{system/user}</code>. Aparecen unidades de ejecución a nivel de sesión de usuario y a nivel de sistema. Adicionalmente, en SystemD además de controlar demonios, también se puede controlar cualquier recurso del sistema (dispositivos, sockets, carpetas, etc.) a través de una tipología de unidades de ejecución. 

El control de los demonios se realiza con los siguientes comandos:

<table class="table table-striped demonio">
  <thead>
    <th>INIT Service </th>
    <th>SYSTEMD Systemctl</th>
    <th>Descripción</th>	
  </thead>
  <tbody>
    <tr>
	  <td><pre class="language-bash"><code>service {demonio} start</code></pre></td>
	  <td><pre class="language-bash"><code>systemctl start {demonio}</code></pre></td>
	  <td>Enciende Demonio</td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>service {demonio} stop</code></pre></td>
	  <td><pre class="language-bash"><code>systemctl stop {demonio}</code></pre></td>
	  <td>Apaga Demonio</td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>service {demonio} restart</code></pre></td>
	  <td><pre class="language-bash"><code>systemctl restart {demonio}</code></pre></td>
	  <td>Reinicia Demonio/td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>service {demonio} condrestart</code></pre></td>
	  <td><pre class="language-bash"><code>systemctl try-restart {demonio}</code></pre></td>
	  <td>Reinicia Demonio solo si se está ejecutando</td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>service {demonio} reload</code></pre> </td>
	  <td><pre class="language-bash"><code>systemctl reload {demonio}</code></pre></td>
	  <td>Recarga Configuración Demonio</td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>service {demonio} status</code></pre> </td>
	  <td><pre class="language-bash"><code>systemctl status {demonio}</code></pre> </td>
	  <td>Estado del Demonio</td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>service {demonio} --status-all</code></pre></td>
	  <td><pre class="language-bash"><code>systemctl --list-units --type service</code></pre></td>
	  <td>Listado de Demonios del Entorno</td>	  
	</tr>	
  </tbody>
</table>

## La Estructura del Arranque

Durante el arranque del sistema, en qué orden se van levantado los demonios hasta completar el entorno de ejecución se indica a través de enlaces simbólicos (accesos directos) a las unidades de ejecución agrupados en carpetas que representan niveles de ejecución.

El resultado es una estructura de carpetas y enlaces simbólicos que define la estructura del arranque del sistema.

+ **Init**: los niveles de ejecución son carpetas ubicadas en <code>/etc/rc.d/rcX.d</code>, donde X es el nivel de ejecución de 0 a 6. En cada carpeta aparecen enlaces simbólicos a las unidades de ejecución que controlan los demonios de ese nivel de ejecución. A través de prefijos en esos enlaces simbólicos se indica la secuenciación del disparo de demonios dentro de cada nivel.
+ **SystemD**: los niveles de ejecución son carpetas en <code>/etc/systemd/{system,user}/{nombre_nivel}.target.wants</code>. En cada carpeta también aparecen enlaces simbólicos a las unidades de ejecución de ese nivel. Los demonios de cada carpeta se lanzan en paralelo, y existe un sistema de dependencias que indica el orden entre unidades de ejecución: si una unidad de ejecución necesita de otra para operar, SystemD levanta antes su dependencia.

La estructura de niveles de ejecución que tiene el arranque es la siguiente:

<table class="table table-striped demonio">
  <thead>
    <th>INIT Runlevel</th>
    <th>SYSTEMD Targets</th>
    <th>Descripción</th>	
  </thead>
  <tbody>
    <tr>
	  <td><code>0</code> </td>
	  <td><code>runlevel0.target,poweroff.target</code></td>
	  <td>Apagado y desconexión ordenada del sistema.</td>	  
	</tr>
    <tr>
	  <td><code>1</code> </td>
	  <td><code>runlevel1.target,rescue.target</code></td>
	  <td>Prepara una consola de rescate.</td>	  
	</tr>
    <tr>
	  <td><code>2</code> </td>
	  <td><code>runlevel2.target,multi-user.target</code></td>
	  <td>Configura un sistema multiusuario no gráfico.</td>	  
	</tr>
	<tr>
	  <td><code>3</code> </td>
	  <td><code>runlevel3.target,multi-user.target</code></td>
	  <td>Configura un sistema multiusuario no gráfico.</td>	  
	</tr>
    <tr>
	  <td><code>4</code> </td>
	  <td><code>runlevel4.target,multi-user.target</code></td>
	  <td>Configura un sistema multiusuario no gráfico.</td>	  
	</tr>
    <tr>
	  <td><code>5</code> </td>
	  <td><code>runlevel5.target,graphical.target</code></td>
	  <td>Configura un sistema gráfico multiusuario.</td>
	</tr>
    <tr>
	  <td><code>6</code> </td>
	  <td><code>runlevel6.target,reboot.target</code></td>
	  <td>Apagado y reinicio ordenado del sistema.</td>	  
	</tr>
  </tbody>
</table>
 
Los comandos para controlar la estructura del arranque son los siguientes:

<table class="table table-striped demonio">
  <thead>
    <th>INIT  Chkconfig</th>
    <th>SYSTEMD Systemctl</th>
    <th>Descripción</th>	
  </thead>
  <tbody>
    <tr>
	  <td><pre class="language-bash"><code>chkconfig {demonio} on --level runlevels</code></pre> </td>
	  <td><pre class="language-bash"><code>systemctl enable {demonio}</code></pre>     </td>
	  <td>Activa demonio en arranque, creando enlaces simbólicos en carpetas de unidad de ejecución</td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>chkconfig {demonio} off --level runlevels</code></pre>   </td>
	  <td><pre class="language-bash"><code>systemctl disable {demonio}</code></pre>   </td>
	  <td>Desactiva demonio en arranque, eliminando enlaces simbólicos en carpetas de unidad de ejecución </td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>chkconfig --list {demonio}</code></pre></td>
	  <td><pre class="language-bash"><code>systemctl is-enabled {demonio}</code></pre></td>
	  <td>Comprueba si un demonio está activado durante arranque</td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>chkconfig --list</code></pre></td>
	  <td><pre class="language-bash"><code>systemctl list-unit-files --type service</code></pre></td>
	  <td>Lista todos los servicios y si están activados en el arranque</td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>chkconfig --list</code></pre>  </td>
	  <td><pre class="language-bash"><code>systemctl list-dependecies --after {demonio}</code></pre> </td>
	  <td>Demonios que se levantan después de esta unidad de ejecución</td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>chkconfig --list</code></pre>  </td>
	  <td><pre class="language-bash"><code>systemctl list-dependecies --before {demonio}</code></pre></td>
	  <td>Demonios que se levantan antes de esta unidad de ejecución</td>	  
	</tr>	
  </tbody>
</table>

## Nivel de Ejecución 

### Cambio de Nivel de Ejecución

+ **Init**: el comando <code>init {numero nivel}</code> cambia de un nivel de ejecucióna otro.
+ **SystemD**: el comando <code>systemctl isolate {nombre target}</code> cambia de un nivel de ejecucióna otro.

<table class="table table-striped demonio">
  <thead>
    <th>INIT </th>
    <th>SYSTEMD </th>
    <th>Descripción</th>	
  </thead>
  <tbody>
    <tr>
	  <td><pre class="language-bash"><code>runlevel</code></pre> </td>
	  <td><pre class="language-bash"><code>systemctl list-units --type target</code></pre>  </td>
	  <td>Nivel de ejecución actual</td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>telinit runlevel</code></pre>   </td>
	  <td><pre class="language-bash"><code>systemctl isolate {nombre target}</code></pre>   </td>
	  <td>Cambio de nivel de ejecucón </td>	  
	</tr>
  </tbody>
</table>


### Nivel de Ejecución por Defecto
+ **Init**: en el archivo <code>inittab</code> se indica hasta qué nivel de ejecución sube el arranque del sistema.
+ **SystemD**: el enlace simbólico <code>default.target</code> apunta a la carpeta del nivel de ejecución al que sube el  arranque del sistema.

<table class="table table-striped demonio">
  <thead>
    <th>INIT <code>inittab</code> </th>
    <th>SYSTEMD <code>default.target</code></th>
    <th>Descripción</th>	
  </thead>
  <tbody>
    <tr>
	  <td><pre class="language-bash"><code>cat /etc/inittab</code></pre> </td>
	  <td><pre class="language-bash"><code>systemctl get-default</code></pre>  </td>
	  <td>Localiza el nivel de arranque por defecto</td>	  
	</tr>
    <tr>
	  <td><pre class="language-bash"><code>vi /etc/inittab</code></pre>   </td>
	  <td><pre class="language-bash"><code>systemctl set-default {nombre target}</code></pre>   </td>
	  <td>Cambia el nivel de arranque por defecto </td>	  
	</tr>
  </tbody>
</table>

