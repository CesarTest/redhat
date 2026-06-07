---
title: Programa
taxonomy:
    category: docs
---

# Programa = Tipo Proceso o Demonio

El objetivo del sistema operativo es organizar todo un ecosistema de programas, al que se suele llamar entorno de ejecución. El kernel es el responsable de asignar recursos hardware a los distintos programas en ejecución.

Hay dos tipos de programas:
- **Demonios o Servicios, que son programas residentes**: programas que están siempre ejecutándose en segundo plano a la espera de recibir peticiones de cualquier otro programa del entorno de ejecución.
- **Procesos, que son programas efímeros**, suelen ser dependientes de programas residentes para poder ser lanzados.

Los demonios son los que crean las condiciones necesarias para que las aplicaciones puedan ser ejecutadas. Por ejemplo, sin el demonio de máquina virtual de Java, las aplicaciones Java no pueden ser ejecutadas.

Estos demonios tienen una estructura estandarizada en Linux, que se analiza a continuación.

# Demonio: Proceso Servidor y Recursos I/O

**DEF.: Demonio**: es un Proceso Servidor (a la espera de atender peticiones de otros programas) y sus recursos de Entrada/Salida. En la imagen, se aprecia como los recursos de entrada/salida del demonio son ficheros y están en una ruta dentro del sistema de ficheros.

![Anatomía del Demonio](image://teoria/teoria-demonio-estructura.jpg)

La estructura de recursos de un demonio se puede determinar a partir de la estructura de su paquete, que se localiza con los comandos:

[prism classes="language-bash command-line" cl-prompt="{root@localhost} $"]
DAEMON=sshd
RPM=$(yum provides *bin/$DAEMON | grep .el | head -1)
rpm -ql ${RPM%%-*} 
[/prism]

Siempre que se analice un demonio en esta documentación, se presentará su estructura mediante la siguiente tabla, cuyo contenido se extrae con los comandos anteriores:

<style>
   .demonio { 
	   table, tbody, tr, td { padding: 0; height: 0; cellpadding: 0; cellspacing: 0; }
       p, code { padding: 0; height: 0; }
	   img     { padding: 0; margin-bottom: 0; margin-top: 0; }
    }
</style>

[div class="table table-striped demonio"]
 ICONO                                                   | RUTA		                                   
---------------------------------------------------------|---------------------------------------------
![EXE](image://teoria/tabla-demonio-exe.png?classes=float-left,demonio&resize=100,100) | <p><b>Ejecutable del Servidor</b></p><p></br><code>/sbin/service</code></p><p><code>/bin/service</code></p>     
![ETC](image://teoria/tabla-demonio-cnf.png?classes=float-left,demonio&resize=100,100) | <p><b>Configuraciones del Servidor</b></p> <p> <code>/etc/service</code></p>
![LIB](image://teoria/tabla-demonio-inmutable.svg?classes=float-left,demonio&resize=100,100) | <p><b>Código fuente y documentación del Servidor</b></p><p><code>/lib/service</code></p>    
![VAR](image://teoria/tabla-demonio-var.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento persistente del Servidor</b></p><p><code>/var/service</code></p><p><code>/var/log/service</code></p><p><code>/usr/lib/service</code></p>
![RUN](image://teoria/tabla-demonio-run.png?classes=float-left,demonio&resize=100,100) | <p><b>Almacenamiento volátil del Servidor</b> (desaparece tras cada reinicio)</p><p><code>/run/service</code></p><p><code>/run/{file.pid}</code></p>
![PROC](image://teoria/tabla-demonio-ram.png?classes=float-left,demonio&resize=100,100) | <p><b>Disco virtual que el kernel crea en RAM</b></p><p><code>/proc/{pid}</code></p>
![IPC](image://teoria/tabla-demonio-ipc.svg?classes=float-left,demonio&resize=100,100) | <p><b>Sockets InterProcess Communication </b></p><p><code>ss -xpln &#124; grep service</code></p>
![NET](image://teoria/tabla-demonio-tcp.png?classes=float-left,demonio&resize=100,100) | <p><b>Sockets TCP/UDP </b></p><p><code>ss -tupln &#124; grep service</code></p>
[/div]

A continuación se analizan por separado las dos partes del demonio:
+ **El Proceso Servidor**: de la interfaz de llamadas al sistema, hacia arriba.
+ **Recursos de Entrada/Salida**: de la interfaz de  llamadas al sistema, hacia abajo.

# Proceso Servidor: CPU y RAM

Tal como muestra la imagen, esta primera parte revisa de la interfaz de llamada al sistema (API Syscall) hacia arriba, o sea, el uso de CPU y RAM. Se introduce los conceptos de concurrencia y planificación de procesos para entender cómo gestiona el kernel muchos programas alternándose en las CPUs, a partir de ahí hacer una clasificación de herramientas que permiten analizar el comportamiento de los programas (instalación, ejecución y seguridad de las aplicaciones). 

![El Proceso Servidor](image://teoria/teoria-demonio-proceso.jpg)

## Diagrama de Flujo Simplificado  

![Diagrama Flujo Proceso](image://teoria/teoria-demonio-flujo.jpg)

En la imagen el ciclo de vida de un proceso servidor, donde se distinguen las siguientes etapas:
+ **Inicialización**: donde se generan los metadatos del proceso, como sería el fichero de presencia PID
+ **Configuración**: interpretar los ficheros de configuración de /etc que van condicionar el comportamiento del servidor.
+ **Conexiones**: prepara los sockets, tanto comunicación interna como externa.
+ **Escucha**: quedando en bucle a la espera peticiones externas.
+ **Atender Cliente**: atiende cada petición del cliente, guardando los datos de ese cliente en /var.
+ **Cierre Conexión**: cuando cliente está atendido se cierra la conexión, y vuelve la fase de escucha.

## Concurrencia: Fork y Thread

En la práctica, se emplea algún modelo de concurrencia que permita atender varios clientes simultáneamente, como por ejemplo sería el fork que se aprecia en la imagen: una ramificación de nuevos procesos que mueren cuando terminan la petición de cliente, existiendo un proceso padre siempre a la escucha de peticiones, cada uno de elas provoca ramificación de un nuevo proceso.

En el modelo de concurrencia tipo thread, varios procesos comparten tanto memoria de instrucciones como memoria de datos, haciendo uso más eficiente de los recursos de CPU y RAM, además de tener una respuesta más rápida. El precio que se paga es que es más complejo de programar al aparecer datos y zonas de código críticas (o compartidas) y zonas de código específicas de cada thread, dando pie a imprevisibles condiciones de carrera. 

**DEF.: Condición de Carrera**: cuando hay recursos compartidos, depende de quién acceda antes a ese recurso, la ejecución de los programas obtendrá diferentes resultados, pudiendo provocarse inconsistencias y comportamientos impredecibles y no compatibles con un sistema determinista. La naturaleza imprevisible de las condiciones de carrera da lugar, en muchas ocasiones, a la aparición de bugs de manera repentina que normalmente no ocurren durante el testeo de un software.

## Planificación: Estados y Transiciones.

En los sistemas operativos los programas están rotando constantemente sobre la misma CPU. Los mecanismos de reasignación de CPU (schedulling) por proceso hace que exista un diagrama de estados y transiciones que describe su ciclo de vida.  

![Proceso, Estados y Transiciones](image://teoria/teoria-demonio-estados.jpg)

|ESTADO	         | FLAG   |	   SUBESTADO
|----------------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|**En Ejecución**| **R**  | **TASK_RUNNING**: El proceso se está ejecutando en una CPU o está esperando para ejecutarse. El proceso puede estar ejecutando rutinas de usuario o rutinas de kernel (llamadas al sistema), o estar en cola y listo cuando esté en el estado En ejecución (o Ejecutable).                                           |
|**Durmiendo**   | **S**  | **TASK_INTERRUPTIBLE**: El proceso está esperando alguna condición: una solicitud de hardware, acceso a recursos del sistema o señal. Cuando un evento o La señal satisface la condición, el proceso vuelve a Ejecución.                                                                                             |
|.	             | **D**  | **TASK_UNINTERRUPTIBLE**: Este proceso también es Dormir, pero a diferencia del estado S, no responde a las señales. Se utiliza sólo cuando la interrupción del proceso puede causar un estado impredecible del dispositivo                                                                                          |
|.	             | **K**  | **TASK_KILLABLE**: Idéntico al estado D ininterrumpible, pero modificado para permitir que una tarea en espera responda a la señal de que debe ser eliminada (salir completamente). Las utilidades frecuentemente muestran los procesos Killable como estado D.                                                      |
|.	             | **I**  | **TASK_REPORT_IDLE**: Un subconjunto del estado D. El núcleo no cuenta estos procesos al calcular la carga promedio. Se utiliza para subprocesos del núcleo. Se establecen los indicadores TASK_UNINTERRUPTABLE y TASK_NOLOAD. Similar a TASK_KILLABLE, también un subconjunto del estado D. Acepta señales fatales. |  
|**Parado**	     | **T**  | **TASK_STOPPED**: El proceso ha sido Detenido (suspendido), generalmente por siendo señalado por un usuario u otro proceso. El proceso puede continuar (reanudado) por otra señal para volver a correr                                                                                                               |
|.	             | **T**  | **TASK_TRACED**: Un proceso que se está depurando también se cancela temporalmente. Se detuvo y comparte la misma bandera del estado T. |
|**Zombie**      | **Z**  | **EXIT_ZOMBIE**: Un proceso hijo envía una señal a su padre cuando sale. Todos los recursos excepto la identidad del proceso (PID), se liberan. |
|.	             | **X**  | **EXIT_DEAD**: Cuando el padre limpia (reaps) todo su árbol de procesos hijo, el proceso ahora está completamente liberado. Este estado nunca será observado en utilidades de listado de procesos |


# Los Recursos I/O: Los Ficheros.

![Los Recursos I/O](image://teoria/teoria-demonio-recursos.jpg)

Tal como muestra la imagen, esta segunda parte revisa de la API Syscall hacia abajo, es decir, cómo los programas acceden a los recursos entrada/salida. La mayoría de errores tienen que ver con no poder agregar un dispositivo hardware a esta dinámica general de ejecución de programas del kernel. Se revisan los conceptos de módulo del kernel (driver del dispositivo) y descriptor de fichero (punto a acceso a los recursos hardware que habilita el kernel). 

## Módulos del Kernel: API Syscall.

+	**__Módulo__ – Implementación de los __métodos__ Syscall**: El kernel presenta una única API abstracta de llamadas al sistema para cualquier recurso de entrada salida (un ratón, un teclado, una pantalla, un disco duro, etc.). Los módulos implementan esta API para cada dispositivo concreto. También reciben el nombre de driver de dispositivo.
+	**__Descriptor de Fichero__ – Las __propiedades__ de los Recursos**: el modelo de datos que le permite al módulo gestionar cada recurso se componen de dos partes:
  -	<u>Sysfs (```/sys```)</u>… sistema de ficheros con el modelo de datos de gestión de los dispositivos hardware.
  -	<u>Fichero de Nodo (```/dev```)</u>… relaciona módulo con las propiedades de Sysfs de cada dispositivo. Estos ficheros de nodo tienen propiedades especiales:
     -	__Tipo de Fichero__: carácter o bloque (los dispositivos de red no tienen fichero de nodo).
     -	__Número mayor del dispositivo__: identifica el módulo que implementa las llamadas de sistema de ese dispositivo
     -	__Número menor del dispositivo__: identifica las propiedades de gestión de ese dispositivo en ```/sys```.

## Descriptores de Fichero  

### Punto de Acceso a Recursos

Un descriptor de fichero es un punto de acceso a un recurso hardware que el kernel pone a disposición de todos los programas en ejecución y se identifican a través de números enteros.

### Reservados: Stdin, Stdout Y Stderr.

Hay tres descriptores de fichero reservados: 
- 0=Stdin 
- 1=Stdout 
- 2=Stderr 

A continuación una ayuda gráfica de cómo se redireccionan las salidas de estos descriptores de fichero reservados. 

![Redirecciones](image://teoria/teoria-demonio-descriptor.jpg)

