---
title: A1.Estructura
taxonomy:
    category: docs
---

# Definición: Entorno de Ejecución.

**DEF.: Entorno de Ejecución**: Un entorno de ejecución es un ecosistema de demonios que crean las condiciones necesarias para que las aplicaciones puedan ser ejecutadas. El demonio de sistema organiza este ecosistema a través de subsistemas y niveles de ejecución.

**DEF.: Demonio**: programa residente (en segundo plano, a la espera de atender peticiones de otros programas) y sus recursos de entrada/salida: El sistema operativo crea un entorno controlado donde los programas solo ven la memoria donde ellos trabajan y actúan sobre los recursos de entrada salida a través de llamadas al sistema (Syscall), el kernel se ocupa de sacar los datos del dispositivo hardware y ponérselos en una sección de memoria donde el programa pueda leerlo.

# Estructura: Kernel y Demonio de Sistema.

La imagen ilustra los dos elementos que vertebran el entorno de ejecución:

+ **Kernel o Núcleo de Sistema**: ofrece una interfaz de llamadas al sistema que abstrae la gestión de recursos hardware a los programas. La trazabilidad de las interacciones entre los programas y el kernel se logra a través del subsistema Audit, que aporta una información muy detallada del contexto de cada llamada al sistema (parámetros, usuarios, etc.). Cada regla de auditoría le indica al kernel que registre un evento, que enviará por un socket netlink al demonio encargado de producir las trazas (AuditD).

+ **Init/SystemD o Demonio de Sistema**: se trata de un programa que levanta todo el ecosistema de programas que compone el entorno de ejecución a partir de una base de datos donde en cada elemento se describe cómo debe arrancar y gestionar cada programa del entorno de ejecución. Existen distintas implementaciones de este demonio de sistema, según el grado de control que se quiere que tenga sobre el entorno de ejecución. Cada demonio de sistema genera sus propios logs donde se van registrando los eventos principales, permitiendo la trazabilidad de los programas del entorno de ejecución.

![Entorno Ejecución](image://teoria/teoria-entorno.jpg)

Respecto a las llamadas al sistema, tal como muestra la imagen, se pueden agrupar en dos familias y a continuación las más comunes dentro de la norma POSIX (Portable Operating System Interface):

+ **Gestión de Procesos**: 
   - *Generación*:  fork/exit/kill
   - *Ciclo de Vida*: exec
+ **Gestión de Recursos Entrada/Salida**:
   - *Ficheros (acceso al hardware)*: open/close/wait , read/write 
   - *Comunicaciones (Sockets)*: TX:{listen/bind/accept}, RX:{connect/shutdown}, sendmsg/recievemsg 

# Funcionalidad: Un Ecosistema Organizado de Demonios

El entorno de ejecución de Linux consiste en un ecosistema de demonios organizados en niveles de ejecución y subsistemas. Cuando el ordenador arranca, en pantalla se puede ver como el Demonio de Sistema  (originariamente el demonio Init, actualmente el demonio SystemD) va lanzando todos esos demonios hasta completar la estructura del entorno de ejecución.

![arranque Entorno Ejecución](image://teoria/teoria-pantalla-inicio.jpg)

## Subsistemas Linux: Una colección de Demonios

El entorno de ejecución se organiza en subsistemas Linux, que son colecciones de demonios que ofrecen una funcionalidad final al ecosistema (en la imagen se indica en color celeste cuando empieza a levantarse cada subsistema). Este documento trata los subsistemas más importantes de un entorno de ejecución Linux, en cada uno se comienza presentando sus demonios, para luego describir sus casos de uso de más importantes. 

**Toda labor de detección de errores en un entorno de ejecución pasa por analizar el comportamiento de estos subsistemas Linux y sus demonios, se torna imprescindible conocer la estructura genérica de un demonio**, su esqueleto, que es siempre el mismo y varía poco entre distintos sistemas Unix.

## Niveles de ejecución: El laminado del Entorno de Ejecución

Los subsistemas se agrupan en niveles de ejecución, que son colecciones de subsistemas que representan un estado del entorno de ejecución. Sobre el nivel 1, se agrega el nivel 2, sobre el nivel 2 el 3 y así sucesivamente, logrando laminar el entorno de ejecución para facilitar su diseño y depuración.

## El Demonio de Sistema: el Administrador del Entorno de Ejecución

![Espacio Usuario](image://teoria/teoria-espacio-usuario.jpg)

El demonio de sistema es un demonio especial, ya que se ocupa tanto de levantar como de controlar todo el entorno de ejecución, como es el padre de todos, tiene PID=1. Mantiene una base de datos de unidades de ejecución donde se describe cómo gestionar cada programa del entorno de ejecución. 
A través de una estructura de carpetas y enlaces simbólicos a estas unidades de ejecución se especifica en qué orden se van lanzando los programas durtante el arranque de la máquina hasta tener levantado todo el entorno de ejecución.

## El Espacio de Kernel y El Espacio de Usuario

En el entorno de ejecución aparecen dos espacios diferentes: el de usuario donde se ejecutan los programas, y el de kernel donde están todos los procesos de apoyo necesarios para abstraer al programa la gestión de los distintos tipos de memoria y los accesos a los distintos dispositivos hardware. 

En la imagen el espacio de kernel son los procesos hijos del proceso PID=2 (demonio de paginación) y el espacio de usuario son los procesos hijos de proceso PID=1. El demonio de sistema administra el espacio de usuario… y sobre el espacio del kernel de procesos auxiliares, no tiene ningún control. 

# El Proceso de Arranque de un Ordenador, y el Rol del Demonio de Sistema

La imagen ilustra las cinco etapas del proceso de arranque de un ordenador:

1.	**BIOS/UEFI**: tras pulsar la tecla de encendido, el programa BIOS (en ordenadores antiguos) o UEFI (en ordenadores más modernos), que reside en una memoria no volátil (ROM), comienza a hacer un reconocimiento de todos los dispositivos hardware de la máquina, proceso que recibe el nombre de POST (Power On Self Test). En el caso del XR12, se trata del iDRAC. 
2.	**Gestor de Arranque (GRUB)**: una vez POST reconoce todo el hardware, procederá a recorrer secuencialmente la lista de dispositivos de arranque, busca un MBR (Master Boot Record) que le referencia a algún sistema operativo o gestor de arranque. En Linux, el MBR apunta al gestor de arranque GRUB, que carga un “sistema operativo reducido”, un kernel y un sistema de discos en memoria (initramfs) que le permite otro reconocimiento de dispositivos hardware antes de presentarnos su menú con las distintas opciones de arranque. Cada opción de arranque del menú referencia a un kernel y a un initramfs de un sistema operativo completo. 
3.	**Kernel**: una vez se selecciona una opción del menú, se carga el núcleo del sistema operativo.
4.	**Initramfs**: una vez el kernel está en memoria, carga el sistema de ficheros en memoria (initramfs) que contiene los drivers que le permiten hacer otro reconocimiento de dispositivos.
5.	**Gestor de Demonios (Init)**: una vez el kernel ha reconocido todos los dispositivos hardware entra el gestor de demonios a levantar el entorno de ejecución. Cada sistema operativo tiene uno diferente, Windows tiene el demonio SVC, MacOS LaunchD, Unix emplea SysInit, Linux emplea SystemD ubicado en /sbin/init. Cuando un sistema Linux arranca, aparece una pantalla de inicio muy característica, donde los demonios se van listando en la medida que el gestor los va levantando, entre corchetes se aprecia el estado, que pasará a color verde cuando ya esté levantado. 

![Arranque Sistema](image://teoria/teoria-arranque.jpg)


