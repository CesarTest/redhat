---
title: Contexto
taxonomy:
    category: docs
---

# ¿Para qué OpenShift?
IBM compra RedHat para poder levantar factorías software en el proceso de modernización que afecta a las administraciones públicas de Estados Unidos. 

**El objetivo de la modernización es desacoplar la evolución del hardware de la evolución del software**, creando un entorno controlado de entrega y ejecución de aplicaciones para aislar el desarrollo software de la evolución tecnológica (tanto tecnologías como fabricantes). El resultado es una evolución estable y consistente: 

  + **Reducción de costes de fabricación:** de un lado se desacopla las aplicaciones del hardware y sus configuraciones eliminando muchas de sus dependencias (versión de sistema operativo, librerías específicas, etc.); del otro la reutilización masiva de piezas de código reduce su coste tanto de producción como de pruebas ["Why Microservices?"](https://media.dau.edu/playlist/details/1_0kko71p1?target=_blank). Un simil, las tuberías de una casa se construyen en base a piezas prefabricadas y bien probadas en multitud de edificaciones, no se hacen piezas específicas para cada nuevo edificio; en el mundo del software se logra algo similar a través de microservicios encapsulados en contenedores y construidos por fábricas especializadas 
  + **Comportamiento más previsible de las aplicaciones:** al reducirse los tiempos de entrega, se van haciendo cambios más pequeños pero mucho más frecuentes, lo que hace más previsible la evolución de la aplicación. 
  + **Uso de nuevas arquitecturas de seguridad:** las nuevas plataformas de ejecución admiten un replanteamiento en las estrategias de seguridad, cuyo aspecto más visible es el llamado ["Software Defined Perimiter"](https://en.wikipedia.org/wiki/Software-defined_perimeter?target=_blank), que oculta toda la infraestructura informática a internet invalidando muchos de los ataques de las arquitecturas convencionales de perímetro de seguridad.

Sin embargo, todas estas ventajas tienen un precio: la refactorización de las aplicaciones para entregarlas en contenedor, así como una compleja plataforma de ejecución cuyos procedimiento se pretenden abordar a través de las certificaciones que ofrece el fabricante RedHat.

# Aplicaciones de Centro de Datos
## Filosofía de Diseño: Orientación a Servicios

Una misma aplicación monolítica, cuando debe gestionar muchos datos, se divide en múltiples servicios. Cada servicio es una aplicación independiente dedicada a gestionar un dominio de datos (por ejemplo, el dominio de usuarios), aportando dos ventajas frente al diseño monolítico:
 + **Especialización en la gestión de datos:** cada servicio gestiona pocos datos, facilitando su control y manipulación.
 + **Flexibilización de la lógica crítica de negocio:** cada servicio expone un contrato de funcionalidades (una API), abstrayendo las funcionalidades de la tecnología de implementación. Así se logran dos objetivos: 
   - *Independencia tecnológica:* poder reescribir cada pieza de código nuevamente en otras tecnologías si fuera necesario, a través de un sistema de pruebas capaz de verificar el contrato de funcionalidades.
   - *Reutilización:* un mismo servicio (como el servicio de login) es empleado por muchas aplicaciones, mejorando la fiabilidad del código, además de reducir el coste de producción.

 Lógica                             | Datos
 -----------------------------------|-----------------------------------
![Lógica](image://intro/logica.jpg) | ![Datos](image://intro/datos.jpg)

## Arquitectura de Aplicación: Multicapa

Una vez elegida una filosofía de diseño, se plantea la arquitectura de la aplicación, es decir, el esquema organizativo de las distintas componentes de la aplicación, clave para su evolución en el tiempo. Estas arquitecturas vienen evolucionando de estructuras de una sola capa que aparecen en los Mainframe a estructuras multi-capa más modernas, donde cada capa encapsula un determinado problema a través de interfaces. La más extendida, es la arquitectura de tres capas:
  
 + **Capa de Presentación:** servicios que implementan tanto la interfaz gráfica como la gestión de sesiones de usuario.
 + **Capa de Lógica de Negocio:** servicios agrupados en subsistemas que implementan la lógica principal.
 + **Capa de Datos:** plano de datos. 
 
 ![Arquitectura Multicapa](image://intro/multicapa.jpg)

## Evolución a Microservicios: La Refactorización.

El último paso en la evolución de la filosofía de diseño orientada a servicios es la transformación a microservicios para aplicaciones que tienen una gran carga de tráfico entrante. Gracias a la automatización y estandarización de la plataforma de ejecución de servicios se logra encapsularlos mejor en contenedores, mejorando así su reusabilidad y simplificando el sistema de dependencias de la aplicación final.

Sin embargo, el transformar una arquitectura multi-capa de servicios a microservicios, no solo se reduce a encapsular los servicios en contenedor con APIs bien definidas, sino que suele implicar una refactorización interna de las capas de la arquitectura por tres motivos:

  + **Lógica - Ajustarse a los principios del ["Diseño Orientado a Dominios"](https://es.wikipedia.org/wiki/Dise%C3%B1o_guiado_por_el_dominio?target=_blank):** El desacoplar las funcionalidades mejor, con un plano de datos propio, y encapsularlas en contenedores para mejorar su reusabilidad, suele significar redistribuir las funcionalidades. Si la estructura de dominios de datos en los subsistemas internos de cada capa no está bien definida de partida, surgirán muchos problemas díficiles de depurar, ya que cada microservicio es un programa totalmente independiente que se comunica con otros a través de API (comunicaciones síncronas) y/o bus de eventos (comunicaciones asíncronas), toda complejo seguiento dentro de la lógica de cada microservicio y sus comunicaciones para detectar estos problemas, que además, son muy frecuentes cuando no está bien arquitecturada la estructura de datos. 
  + **Datos - La consistencia del modelo de datos depende de la lógica de negocio**, en lugar de depender del motor de base de datos. El modelo de datos es el mismo, pero fraccionado para que cada microservicio pueda gestionar una base de datos propia e independiente. Según como sea el diseño original, esta refactorización puede llegar a ser muy compleja. Al existir menos dependencia de las prestaciones del motor de base de datos (normalmente Oracle), en algunos proyectos se aprovecha esta migración a microservicios para sustituirlo por otros gestores de datos de código abierto y/o distribuidos, eliminando el elevado coste de las licencias de los motores de grandes prestaciones. De hecho, cada microservicio puede emplear un motor de datos diferente.
  + **Formato Entrega - Estandarización en los procesos de encapsulado en contenedor** para que no aparezcan inconsistencias al instanciarlos en plataforma.  

Como paso inicial y para evitar la refactorización, simplemente se lanza un servidor de aplicaciones con la aplicación sin modificar en un clúster virtual, o sea, se lanza lo antiguo sin modificar sobre estas nuevas plataformas; que siguen aportando la automatización de despliegues gracias a Helm Chart (lista de recursos del clúster que permiten la instanciación del servicio).

 Lógica                                   | Datos
 -----------------------------------------|-----------------------------------
![Kubernetes](image://intro/refactor.jpg) | ![Teorema de CAP](image://intro/database.jpg)
    
## Kubernetes: Nueva Plataforma de Ejecución de Servicios

![Kubernetes](image://intro/kubernetes.jpg)

En la imagen, la estructura de la plataforma. Cada huevo representa un servicio en un cluster virtual (namespace de kubernetes, project de OpenShift), cuyo despliegue se automatiza a través de Helm Charts. Cabe destacar la estructura de capas necesarias para la automatización de la gestión de servicios, encapsuladas a través de APIs:
 + **L1 - OpenShift Kubernetes:** el racimo de ordenadores que opera como si fuera uno solo para la gestión del ciclo de vida de los contenedores.
 + **L2 - OpenShift Pipelines:** servicio de cabecera que automatiza el aprovisionamiento de servicios, esto es, las distintas tuberías DevSecOps (en fábrica) o SecOps (en operadora) que alimentan a cada clúster virtual.
 + **L3 - OpenShift Service Mesh:** servicio de cabecera que agregar un "sidecar container" a cada "End Point", un contenedor que controla tanto las comunicaciones como los logs del contenedor principal del "End Point", aportando:
    - *Monitorización de servicios:* acceso a todos los logs
	- *Gestión de identidades*, tanto usuarios, como servicios y otros recursos de la plataforma.
	- *Políticas de lista blanca* entre servicios, o sea, la microsegmentación que es crítica para reducir superficie de ataque.
	- *Automatización de despliegues* de las nuevas versiones de cada aplicación pudiendo estrategia de despliegue (A/B Testing, Canary, etc.).
 + **L4 - OpenShift Serverless:** capa opcional que permite simplificar el control del ecosistema de aplicaciones (FaaS=Function as a Service), o sea, construir y desplegar aplicaciones y servicios sin tener que preocuparse por la administración de la infraestructura.
 
La estandarización de esta estructura de capas es crítica tanto para encapsular las funciones, como lograr una buena trazabilidad que permita la depuración de estas complejas plataformas. Se circunscriben los casos de uso de cada capa, con su casuística de errores, permitiendo así tenerlo todo bajo control.

# Fábricas DevSecOps: Automatización de la construcción y pruebas del Código

![Fábrica DevSecOps](image://intro/factory.png)

+ **Automatización:** Las factorías DevSecOps son cadenas de entornos donde se van ejecutando los distintos procesos de construcción y pruebas del código fuente. 
+ **Control GitOps:** Las configuraciones de cada entorno se declaran en un repositorio Git, cualquier cambio de configuración automáticamente se aplica al entorno.

Existe un conjunto de certificaciones que abordan toda esta automatización necesaria en estas fábricas.