---
title: 1.1.Kernel
taxonomy:
    category: docs
---

# Ejecutando Programas

En la arquitectura del entorno de ejecución, la primera de sus piezas es el kernel, o núcleo de sistema, responsable de asignar recursos hardware a los múltiples programas que se quieren ejecutar en una máquina.

Comprender cómo se ejecutan los programas en un ordenador y cómo el kernel gestiona todo su ciclo de vida resulta clave para saber diagnosticar problemas a nivel de sistema operativo. A continuación, se analizan las dos partes de esta dinámica de ejecución de programas (las abstracciones proceso y fichero) organizando un maletín de herramientas de análisis para cada una de estas partes, a partir del cual poder establecer protocolos de diagnóstico, o sea, procedimientos de análisis del comportamiento del sistema operativo  que faciliten localizar la causa de un determinado problema.

## Kernel: Asignando Recursos a los Programas

**DEF.: Kernel o Núcleo de Sistema**: proceso responsable de gestionar los recursos hardware de una máquina para que puedan ejercutarse simultáneamente muchos programas, en virtud a las abstracciones:
- *Proceso*: carga de cada programa en memoria y tiempo de ejecución de sus instrucciones en CPU.
- *Fichero*: transferencia de datos entre el programa y los dispositivos hardware o la red.

![Máquina Analítica](image://teoria/teoria-babbage.jpg)

## El Ciclo de Vida de los Programas

En la imagen se puede ver lo que suponía gestionar manualemente toda esa maquinaria para poder ejecutar un solo programa en los antiguos ordenadores; hubo que hacer inmensas inversiones hasta localizar cómo automatizar la asignación de recursos de la máquina a los múltiples programas.

El núcleo carga en memoria las instrucciones de cada programa, y le va asignando intervalos de tiempo en CPU para que vaya avanzando en su conjunto de instrucciones.

Cuando el programa necesita acceder a algún dispositivo hace una llamada al sistema (representada por una campanilla en la imagen) sobre un descriptor de fichero (un punto de acceso al recurso hardware que el núcleo pone a disposición de los programas), lo que provoca una interrupción. A partir de este momento, el programa deja la CPU y el kernel comienza a realizar todas las operaciones necesarias hasta dejarle el dato que necesita de ese dispositivo en una sección de memoria específica, donde puede recogerlo como un dato más. El periférico DMA (Direct Memory Access) juega un papel central en esta dinámica.

![Ejecución de Programas](image://teoria/teoria-demonio.jpg)

## Acceso a Recursos Hardware

En el dibujo se resume cómo el kernel abstrae el acceso al hardware (recursos de entrada/salida) mediante una única interfaz de llamadas al sistema:
+	**Todo recurso de entrada/salida es un fichero, con operaciones de lectura/escritura**, son elementos con los que comunicarse para transferencia de datos.
+	**Los descriptores de fichero son sus puntos acceso**: el sistema operativo pone a disposición de todos los programas en ejecución un descriptor de fichero a través del cual leer/escribir cada recurso. Se identifican a través de número (en la imagen, sobre las campanillas de llamada al sistema, se indican esos identificadores: fd=15).
+	**El sistema operativo abstrae las operaciones de lectura/escritura sobre ficheros mediante tres elementos**:
 - 	<u>Módulo del kernel (```lsmod```)</u>: implementa los métodos de la API Syscall (las operaciones lectura/escritura) para cada dispositivo concreto. En definitiva, se trata del driver del dispositivo.
 -	<u>Sysfs (```/sys```)</u>: propiedades de cada dispositivo necesarias para poder interactuar con él.
 -	<u>Fichero de Nodo (```/dev/```)</u>: relaciona las propiedades de Sysfs (número menor) con el módulo del kernel (número mayor).
+	**Cinco tipos de descriptores de fichero**, pero solo dos pilas de búsqueda en Linux:
 -	<u>Pila 1 - Tipo carácter</u>: donde los datos se transmiten en serie (ejm.: ratón, teclado).
 -	<u>Pila 1 - Tipo bloque</u>: donde los datos se transmiten en ráfagas (ejm.: discos duros).
 -	<u>Pila 2 - Tipo red (o Socket)</u>: es un fichero tipo carácter dedicado a las comunicaciones entre programas, ya sea por red (Socket TCP/UDP), ya sea entre procesos de una misma máquina (Socket IPC). A pesar de ser transferencias de datos en serie, tienen un conjunto de llamadas al sistema muy distinto al resto, además de un hardware específicos, por eso se considera un tipo de descriptor de fichero diferente llamado Socket (enchufe en inglés). 


## El demonio UDEV: Plug & Play del Hardware

El demonio UDEV automatiza todas las operaciones necesarias para agregar dispositivos hardware a esta dinámica general de ejecución de programas:
- *Carga del módulo del kernel.*
- *Creación del modelo de datos con las propiedades del dispositivo en ```/sys```.*
- *Creación del fichero de nodo*, que relaciona Módulo del Kernel (métodos Syscall, número mayor) con propiedades del dispositivo en ```/sys``` (número menor).

!!! Obsérverse que se trata de un estructura orientada a objetos, cada clase de dispositivos comparte métodos (módulo del kernel), pero cada objeto, cada dispositivo concreto, tiene unas propiedades específicas. Por ejemplo, la clase de discos SCSI tienen un único driver para implementar la API Syscall, pero cada objeto, cada unidad de disco SCSI y cada partición tiene sus propiedades específicas.


# Anatomía del Sistema Operativo

## Disposición de Elementos

Una vez comprendida cómo se ejecutan los programas en un sistema operativo, se puede ilustrar las componentes de Linux que permiten esa ejecución concurrente de programas, clave en cualquier proceso de diagnóstico ([Brendan Gregg Blog](https://www.brendangregg.com/linuxperf.html?target=_blank)):

![Componentes Sistema Operativo](image://teoria/teoria-anatomia-so.jpg)

## Clasificación de Herramientas de Análisis

### Análisis Dispositivos Hardware (azul y verde)

Se corresponde con la parte inferior de la interfaz de llamadas al sistema. Cuando, bien el mecanismo plug & play de UDEV no funciona correctamente, bien los drivers tienen algún bug o bien los dispositivos son muy complejos de enganchar y hay que hacerlo manualmente, hay que analizar las siguientes cuestiones:

  -	**Análisis de las dos pilas de búsqueda del kernel**, 
 	 - <u>Dispositivos de bloque y carácter</u>, en azul. 
     - <u>Dispositivos de red</u>, en verde. Al no ser todo automático, es más habitual tener que configurar la red manualmente.
  - **Análisis del flujo de datos que entra al kernel**, tanto el bus de red (en verde) como bus de I/O (en azul).

### Análisis Kernel (ocre)

Se corresponde con el uso de recursos de los programas:
  - **Planificación de procesos.**
  - **Memoria virtual.**	 
  -	**Uso de RAM y CPU.**

![Diagnósticos-HW](image://teoria/teoria-diagnostico-hw.jpg) 

### Análisis Aplicaciones (rojo)

Se corresponde con la parte superior de la interfaz de llamadas al sistema. Los análisis de esta parte del sistema consisten en:
  +	**La instalación** de los programas.
  +	**La ejecución**, analizar el comportamiento de las aplicación.
  	- *La carga en memoria*, con sus correspondientes librerías.
  	- *La administración de la memoria.* 
  	- *La introspección de llamadas al sistema* que realiza.
  + **La seguridad:** analizar cómo gestionan los recursos las aplicaciones, donde se distinguen tres partes:
    - <u>Control de Recursos (MAC=Mandatory Access Control):</u> en las dos variantes disponibles:
		- *AppArmor:* mediante perfiles de capacidades Linux (permisos, ACLs=listas de control de accesos, etc.) se controla el accesos a los recursos.	
	    - *SELinux:* a través de un etiquetado de recursos a nivel de kernel, el núcleo controla los accesos a los recursos.
	- <u>Control de Usuarios (RBAC=Role Based Access Control):</u> análisis de como se registran los usuarios, en sus tres variantes:
        - *PAM (Pluggable Authentication Module):* demonio que se ocupa de los login de usuario en el sistema operativo, o sea, a nivel de máquina.
		- *LDAP (Lightweight Directory Access Protocol):* las autenticaciones se delegan a un servidor externo que centraliza las credenciales de todo un centro de datos.
		- *Kerberos (AAA=Autenticación, Autorización, Accounting):* funciona como un parquímetro para uso de servicios del centro de datos. Los usuario se autentican, recibiendo un ticket de usuario (golden ticket) que le permite el uso de servicios por un tiempo; este proceso de autenticación también se basa en LDAP. Para cada servicio que quiera utilizar tiene que ir renovando los ticket de servicio (silver ticket) que le van dando franjas de tiempo de uso de ese servicio. En otras palabras, para cada servicio tiene que tener un ticket de servicio que le abre una ventana de tiempo de uso. 
    - <u>Control de Comunicaciones:</u> análisis de cómo se comunican las aplicaciones, donde hay tres elementos a tener en cuenta:
	    - *Firewall:* filtrado de comunicaciones para evitar amenazas.
	    - *Políticas Criptográficas:* una vez que se establece una política para todo el sistema, las aplicaciones en RHEL la siguen y se niegan a utilizar algoritmos y protocolos que no cumplan con la política, a menos que usted solicite explícitamente a la aplicación que lo haga.
	    - *TCP Security Report:* análisis de puertas traseras de las aplicaciones.
						
![Diagnósticos-SW](image://teoria/teoria-diagnostico-sw.jpg) 