---
title: A)Entorno
taxonomy:
    category: docs
---

# Definiciones 

![Sistema Operativo](image://teoria/teoria-os.jpg?classes=float-left,shadow&resize=200,250)

+ **DEF.: Sistema Operativo**: es un programa que ofrece una interfaz estándar de llamadas al sistema a las aplicaciones, interfaz que les permite usar todos los recursos hardware de la máquina sin conocer sus detalles específicos (un ratón, un teclado, una pantalla, una impresora, etc.). Serán los drivers de dispositivo (en Linux, módulos del kernel) los encargados de implementar esa API estándar de llamadas al sistema para cada caso concreto. Cada sistema operativo tiene su propio juego de llamadas al sistema, aunque están tipificadas en la norma POSIX (Portable Operating System Interface).

+ **DEF.: Proceso**: la gestión de tiempos de CPU y espacio en memoria RAM para ejecutar los distintos programas en un ordenador. 

+ **DEF.: Fichero**: la gestión de la transferencia de datos (I/O=Entrada/Salida) entre las aplicaciones y los dispositivos hardware. El núcleo de sistema habilita descriptores de ficheros que pone a disposición de las aplicaciones, cada descriptor es un número que da acceso a un tipo de recurso de entrada/salida. En Linux, existen cinco tipos de descriptores de fichero, siendo el socket para las comunicaciones entre procesos (locales o remotos) los que tienen un conjunto de llamadas al sistema más específico. 

+ **DEF.: Entorno de Ejecución**: ecosistema de programas que crean las condiciones necesarias para que las aplicaciones puedan ser ejecutadas. Tres son las cuestiones a tener en cuenta:
  - *Elementos*: Kernel (núcleo de sistema) y Demonio de Sistema.
  - *Trazabilidad:*  auditoría del kernel y trazas del demonio de sistema.
  - *Seguridad:* control de accesos a recursos, usuarios y comunicaciones.  
 
# Origen de Linux

## Nacimiento de Unix en Bell Labs
En el año 1970, en los laboratorios Bell de AT&T los programadores Ken Thompson y Dennis Ritchie desarrollan el sistema operativo Unix, a partir de las conclusiones que sacaron del proyecto Multics en colaboración con el MIT (Massachusetts Institute of Technology). Al poco tiempo John Lion publica “A Commentary on the UNIX Operating System” convirtiéndose en el libro de texto de las universidades de computación de Estados Unidos, sentando así las bases doctrinales de los sistemas operativos. Para Unix se inventó el lenguaje de programación C, el primer lenguaje de programación de alto nivel que se parece más a un lenguaje humano que al incomprensible lenguaje máquina, lo que permite entender su código fuente y así adoptarlo por muchísimas universidades y fabricantes, convirtiéndose en un estándar de facto.

## Nacimiento de Minix y SUN Microsystems ante el bloqueo de Unix
Tras la publicación del libro de John Lion, Bell Labs se percata del enorme potencial comercial que tiene Unix, y para su versión 7 (queno estaba anclada a un hardware específico como las anteriores) se prohibió su enseñanza en universidades, y solo permitiendo un estudio teórico de la versión 6 de Unix. A raíz de ese bloqueo, el profesor Andrew Tanenbaum desarrolla Minix basándose en el libro de John Lion, para que sus estudiantes pudieran interactuar con un sistema operativo en miniatura libre de las restricciones que estaba imponiendo Bell Labs a Unix. Por otro lado, en la Universidad de Berkeley se decide reescribir el kernel dando origen al BSD (Berkeley Software Distribution) y en la Universidad de Stanford el estudiante Bill Joy reescribirá el núcleo BSD a una versión simplificada y más eficiente, su compañero Andreas von Bechtolsheim ajustará un hardware específico para optimizar la ejecución de ese kernel, fundando la empresa SUN Microsystems que llegó a ser dominante en el mundo del centro de datos.

## Nacimiento del proyecto GNU
En el año 1983, Richard Stallman funda el proyecto GNU con el objetivo de crear un sistema operativo basado en Unix, pero que fuera libre, es decir, que permita al usuario modificar y redistribuir el código fuente. Surge entonces la licencia GPL para el software. En un primer momento, Richard Stallman desarrolla un núcleo de sistema operativo (o kernel) llamado GNU Hurd, pero no atrajo los suficientes esfuerzos de la comunidad.

## Nacimiento del proyecto Linux
En el 1991, Linus Benedict Torvarlds, un estudiante finlandés de ciencias de la computación porta el kernel de Minix a los PCs, ya que en ese segmento había una lucha encarnizada entre sistemas operativos que no eran Unix (Windows, IBM OS/2, Next, etc.). Al ser un proyecto personal, lo publica con licencia GPL de GNU, que permite el uso libre del código fuente. Los desarrolladores de hardware vieron ahí una forma sencilla y barata de probar sus drivers de dispositivos, atrayendo una fuerte atención por parte de la comunidad GNU, fomentando enormemente su desarrollo.

## La importancia del Código Abierto para la industria del Software
La primera empresa comercial de código abierto será RedHat que empieza a distribuir las aplicaciones GNU en paquetes RPM a través de repositorios. La producción de aplicaciones software es una industria más, y opera bajo los mismos principios que todas las demás industrias: existe un conjunto de componentes básicas comunes a todos los fabricantes a partir de los cuales se crean las soluciones finales. Por ejemplo, en la electrónica analógica los amplificadores operacionales, resistencias o condensadores, en la fontanería los tubos y tuercas, etc. No es diferente la producción de aplicaciones, solo que al ser productos intangibles las piezas tienen unas propiedades diferentes. En electrónica analógica hay muchísimos modelos de amplificadores operacionales entre los que elegir para cada diseño, su equivalente software es una sola pieza que debe poder adaptarse a las distintas condiciones de trabajo. Para esta adaptabilidad, resulta vital el código abierto, como el tiempo ha demostrado. Hoy día, el primer principio arquitectónico de la norma CNTT ([Cloud iNfrastructure Telco Taskforce](https://cntt.readthedocs.io/en/stable-kali/common/chapter00.html?target=_blank)) es el uso de componentes de código abierto, así que toda la industria de las telecomunicaciones se basa en un surtido amplísimo de piezas de código abierto que emplean para crear sus redes de comunicaciones, todas ellas bajo los auspicios de la Fundación Linux. Más próximo a nuestra experiencia, el motor de renderizado Chromium, empleado en los navegadores web Edge y Chrome, tanto Microsoft como Google se liberan del alto coste de mantener la rápida evolución de sus navegadores delegando esta responsabilidad a un único motor de renderizado de código abierto.

## Linux como principal distribuidor de Software de Infraestructura
Cada paquete Linux sigue todo un proceso de compilado, validación y distribución… primero en una distro de código abierto donde supera un tiempo prueba (como Fedora) para luego pasar a las distribuciones comerciales (como RedHat), ofreciendo estas piezas que necesitan las fábricas validadas y fácilmente accesibles. Los “ports” de BSD es una forma más barata de distribuir código abierto al ahorrarse la infraestructura de pruebas de la paquetería, pero no aporta la fiabilidad necesaria para la industria.
El kernel (o núcleo de sistema) siempre ha sido la “joya de la corona” de todas las empresas, de él dependen tanto la comunidad de fabricantes software como hardware, el hecho de que Linux no tuviese ningún espíritu comercial en una pieza tan codiciada por las empresas por ser indispensable en combinación con el abrumador éxito de la arquitectura hardware fuertemente estandarizada de los PCs, tal vez hayan sido las claves del éxito de este núcleo como base para la distribución de la inmensa infraestructura de aplicaciones de código abierto que existe hoy y en continua expansión… también llamado, “Software de Infraestructura" ([Cloud Native Landscape](https://landscape.cncf.io?target=_blank)).
