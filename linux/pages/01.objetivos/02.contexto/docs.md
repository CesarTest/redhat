---
title: Contexto
taxonomy:
    category: docs
---

# Gestión de Proyectos Software
La imagen ilustra los elementos en la gestión de proyectos software y el papel de RedHat en todo esto. El suministro de software de código abierto empaquetado, verificado y distribuido a través de un sistema de repositorios se torna vital para todas las fábricas software y RedHat viene desempeñando un rol central.

![Gestión de Proyectos Software](image://intro/software.jpg) 

A continuación la nomenclatura en estas fábricas software:

## ¿Qué hay que hacer? - Diseño de Aplicación
### ¿Qué? - Arquitectura

Las decisiones arquitectónicas son la clave de que un proyecto pueda evolucionar, porque **puede ir incorporando de manera natural las nuevas funcionalidades** que se le van exigiendo.

+ **¿Qué? - Multicapa:** Organización de las distintas componentes. Esto viene evolucionando de una sola capa (un monolito) a múltiples capas que aíslan cada componente a través de una interfaz. La más común es la de tres capas (presentación, lógica de negocio y plano de datos).  
+ **¿Cómo? - Monolítico, SOA, Microservicios:** Filosofía de diseño de cada capa, según volumen de datos y carga de usuarios, puede ser monolítica, orientada a servicios (en un servidor de aplicaciones) o, la última novedad, los microservicios (en clústeres).

En la aeronáutica, todo está orientado a servicios, pero transformar servicios en microservicios no se reduce a meterlos en contenedor y ya está... normalmente hay que refactorizar la estructura interna de las capas de la aplicación. Ahora no hay un único modelo de datos, sino que cada microservicio tiene su propia base datos, la consistencia de los datos recae en la lógica de los microservicios no el motor de base de datos; y si el dominio de datos no está bien diseñado, al fabricar una pieza autónoma y reutilizable es un auténtico problema integrarla con otros microservicios. Además depurar sus errores es muy complejo, al haber más comunicaciones internas de aplicación y logs distribuidos entre varios contenedores. En resumen, hay que descomponer la lógica interna de cada capa de manera progresiva e identificando bien cómo hay que descomponer la pieza en una colección de piezas aisladas (a modo de piezas de un Lego).

### ¿Cómo? - Ciclo de Vida

Hay muchos modelos de ciclo de vida: en cascada, espiral, prototipado evolutivo, y miles más. La ingeniería del software suele emplear alguna de las múltiples variantes evolutivas porque va por entregas, aunque hasta hace bien poco se usaba la cascada ([Waterfall](https://youtu.be/YjZ4AZ7hRM0?si=JC8fnNbWXqaWSm9R&t=205&target=blank)).

+ **¿Qué? - Etapas:** La secuencia de etapas por las que pasa cada iteración del proyecto.
+ **¿Cómo? - Plazos y Riesgos.**
 
## ¿Cómo hay que hacer? - Implementación de la Aplicación
### ¿Qué? - Metodologías Ágiles

 Actualmente los proyectos utilizan metodologías ágiles, hay una gama amplia (kanban, scrum, etc.), según el tipo de proyecto... lo más común, el scrum con sus scrum master, backlogs y todo este historial tan popular. 
 
 Se puede implementar un ciclo de vida en cascada a través de metodologías ágiles aplicadas en cada una de las etapas de la cascada... igual no se entrega hasta tener el producto final: el plazo de entrega lo dicta el ciclo de vida, no como lo ejecutas. 
 
 En definitiva, es totalmente independiente el modelo de ciclo de vida que apliques a la metodología de ejecución, aunque hay ciclos de vida que parece que encajan mejor con ciertas metodologías de ejecución; por ejemplo, los ciclos evolutivos encaja mejor con metodologías Ágiles.
 
+ **¿Qué? - Normativa de Calidad:** Descripción detallada de cada tarea, normalmente acoplándose a alguna normativa de calidad nacional o internacional.
+ **¿Cómo? - Gantt / Pert:** Asignación de personal a las tareas, para ello suelen emplearse diagramas de Gantt o Pert.

### ¿Cómo? - Automatización DevSecOps

Automatización de todos los procesos de construcción del código.

+ **¿Qué? - DevSecOps / SecOps:** En factoría DevSecOps, en operadora de centro de datos SecOps.
+ **¿Cómo? - Tecnologías RedHat:** IBM compra RedHat precisamente para armar estas fábricas de software en el Gobierno USA ([Factoría Software](https://redhatgov.io/?target=blank) ).