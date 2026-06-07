---
title: Introducción
taxonomy:
    category: docs
---

# Objetivos: Estándares de Programación

La imparable evolución impone una creciente automatización de todos los procesos, una de sus elementos más importantes es la infraestructura como código. La naturaleza de este tipo de programación impone *una especialización en componentes muy pequeñas, posibilitando mantener y evoluciona una infraestructura común de código.* Se torna **esencial que los equipos involucrados en la automatización sigan unos estándares comunes...** de tal forma que el resultado final pareciera haber sido producido por una sola persona.

La serie de **ejercicios que aquí se propone condensa en el mínimo espacio de tiempo el mayor números de estándares** que la industria viene elaborando desde hace relativamente pocos años... un modo acelerado de aterrizar en una metodología de trabajo que permite gradualmente ir levantando una infraestructura de automatización para cualquier tipso de infraestuctura (Bare Metal, Máquina Virtual, Clústeres).

# Planteamiento
## GITOPS: Gestión de Configuraciones en Repositorio GIT.
La imagen ilustra la propuesta de RedHat para la gestión de Infraestructura como código. Como se puede apreciar se trata de  filosofía GitOps donde se torna crucial desacoplar completamente la declaración de las configuraciones, de la lógica que aplica esas configuraciones. La industria, a raíz del proyecto kubernetes, viene desarrollando una forma estandarizada para la declaración de esas configuraciones.
![Infraestructura como Código](image://auto/ansible_ocp.jpg)

## Estructura de los Ejercicios
Los ejercicios gradualmente van construyendo un modo de trabajo.

### Ansible
Aprender los estándares de programación Ansible que está promoviendo RedHat en sus manuales a través de una demostración práctica.
- **Estándares de Programación:** se toma como referencia [Linux System Roles](https://linux-system-roles.github.io/?target=_blank), la colección que emplea RedHat para todos sus despliegues. Se localizan sus políticas de buenas prácticas.
- **Estándares de Ejecución:** encapsular lógica y sus dependencias en contenedor (Ansible Execution Environment).
- **Estándares de Desarrollo:** flujos GitOps... tuberías CI/CD en Ansible.

### GitOps_CD: Continuous Delivery
El desacople entre lógica y declarariones que impone la Infraestructura como Código se traduce en que en Ansible existen dos tipos de proyectos: 

+ **CD: API - Proyectos de Infraestructura**, integración de la API que gestiona las configuraciones desde repositorio GIT. Estos proyectos solo emplean un conjunto de librerías sin desarrollar lógica alguna; su foco está en la aplicación de configuraciones. A estos proyectos se le asocial Workflows de despliegue continuo (CD).
+ **CI: Lógica - Proyectos de Librerías**, librería o colecciones de librerías, con sus pruebas unitarias. A estos proyectos se le asocia Workflows de integración continua (CI) con regresivos que garantizan la continuidad de la funcionalidad.

La sección GitOps_CD se basa en el proyeto de infraestructura visto en la sección Ansible y le agrega workflow de entrega continua, para ver como opera esto en la práctica. Se emplea GitHub Actions, aunque existen herramientas específicas para estas tareas.

### GitOps_CI
La programación de librería ansible sigue la filosofgía TDD (Test Driven Development). Esta sección elabora una pequeña librería, centrando la atención en todos los sistemas de pruebas que permiten aplicar filosofía TDD. 

Tal como se aprecia en esta librería que se toma como referencia, la carpeta tests de los roles de Ansible tiene una prueba unitaria para cada configuración que debe poder aplicar el módulo. Esto se torna esencial, tanto como muestra de API en proyectos de infraestructura que tiene que armar todo lote de configuraciones... como para regresivos que verifiquen la funcionalidad. 
+ [Linux System Roles](https://github.com/linux-system-roles/network?target=_blank)
![Pruebas Ansible](image://auto/molecule.jpg)

**MOLECULE**                        | **TOX** 
------------------------------------|---------------------------------------
<div class="grav-youtube"><iframe src="https://www.youtube.com/embed/hglpWHMyFHA?si=xp7mS_ZOYxIyDror" frameborder="0" allowfullscreen=""></iframe></div> | <div class="grav-youtube"><iframe src="https://www.youtube.com/embed/XUMqKoQEls8?si=WeYPN-9cJ9XxcPcS" frameborder="0" allowfullscreen=""></iframe></div> 

### PXE
El proceso PXE es muy importante en la gestión de Infraestructua como Código. Esta sección arma uno desde cero, dando la oportunidad de integrar todo lo visto anteriormente en un sistema de trabajo:
- **Proyectos de librería Ansible:** se emplea aun servidor DHCP como ejemplo que permite ver todos los aspectos del desarrollo de lógica ansible.
- **Proyectos de Infraestructura** el servidor PXE sirve para ver cómo se trabaja con proyectos de integración. 
- **Bootloader iPXE** está ampliamente extendido en toda la industria este bootloadar para instalaciones desatendidas, debido a su alta compatibilidad con el hardware, la posibilidad de comunidaciones encriptadas en los plataformados y consola de recuperación ante errores. Se construye iPXE desde cero, y se integra en este servidor.
- **Instaladores de Sistema Operativo** se revisan las distintas opciones: kickstart, cloud-init, CloneZilla Server, Ignition Files.

![Instalación Desatendida](image://intro/pxe.jpg)  
 
# TDD = Test Driven Development 
Los principios de desarrollo TDD que sigue este ejemplo se ilustran en la imagen, y que se ilustran a través de los siguiente autores:
+ [Linux System Network Roles](https://github.com/linux-system-roles/?target=_blank)
+ [Open Networking Foundation Roles])(https://github.com/OpenNetworkingFoundation?target=_blank)
+ [Robert de Bock Roles])(https://robertdebock.nl/?target=_blank)
![Pruebas Integración](image://auto/tdd.png)