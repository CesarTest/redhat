---
title: OpenShift
taxonomy:
    category: docs
---

# Introducción

## Objetivo
Se pretende dar una visión resumida y completa del sistema cluster kubernetes, para luego poder acometer prácticas que permitan aprobar examenes de certificación.

## Bibliografía

+ [RedHat OCP Collection](https://github.com/redhatci/ansible-collection-redhatci-ocp?target=_blank)
+ [DevSecOps Intro](https://www.cloud.mil/devsecops/?target=blank)
+ [DevSecOps Fundamentals](https://dodcio.defense.g+ov/Portals/0/Documents/Library/DoD%20Enterprise%20DevSecOps%20Fundamentals%20v2.5.pdf?target=blank)
+ [DevSecOps Reference Design](https://dodcio.defense.gov/Portals/0/Documents/Library/DoD%20Enterprise%20DevSecOps%20Reference%20Design%20-%20CNCF%20Kubernetes%20w-DD1910_cleared_20211022.pdf?target=blank)
+ [BigBang Software Factory](https://docs-bigbang.dso.mil/latest/docs/#what-are-the-benefits-of-using-big-bang?target=blank)
+ [BigBang Customer Template](https://repo1.dso.mil/big-bang/customers/template?target=blank)
+ [Software Factory](https://www.softwarefactory-project.io/?target=blank)

!!! Nota: *"BigBang Customer Template"* permite definir cómo inicializar cada entorno de la tubería en factorías software.

# Arquitectura

## Definiciones

+ **DEF. - Kubernetes:** un racimo de ordenadores que opera como uno solo para controlar el ciclo de vida de los contenedores.
+ **DEF. - Namespace (project en OpenShift):** es un cluster virtual que encapsula todos los PODs de una aplicación.
+ **DEF. - POD:** la unidad de despliegue fundamental de kubernetes. Consiste en un conjunto de contenedores que comparten recursos de comunicaciones (la red) y de almacenamiento.
+ **DEF. - End Point:** son PODs que forman parte de una aplicación.
+ **DEF. - Controladores:** son PODs cuya lógica sigue el [patrón controlador](https://kubernetes.io/docs/concepts/architecture/controller/?target=blank) para sondear periódicamente el estado de los End Points, actualizando la base de datos recurso-estado etcd. El plano de control actúa sobre cada recurso del clúster según qué diga la base de datos recurso-estado etcd.
+ **DEF. - Operadores:** son PODs cuya lógica sigue el [patrón operador](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/?target=blank) para sondear distintos elementos del clúster. Los hay de dos tipos:
   - *Infraestructura:* dependen del operador [CVO (Cluster Version Operator)](https://docs.okd.io/latest/operators/operator-reference.html?target=blank) y se ocupan de supervisar elementos de infraestructura del clúster, como por ejemplo, el acceso a la infraestructura del datos.
   - *Aplicación:* dependen del operador  [OLM (Operator Lifecycle Manager)](https://olm.operatorframework.io/docs/?target=blank)  y se ocupa de supervisar la actividad de distintas aplicaciones del clúster, como por ejemplo los servicios de cabecera.   
   
## Tipos de Nodos y su Entorno de Ejecución.

En OpenShift  existen dos tipos de nodos:
 + **Nodos tipos Infra:** encargados de desplegar aplicaciones de infraestructura, como serían los nodos máster que conforman el plano de control del clúster desde el que se gestiona el despliegue de todos los PODs.
 + **Nodos tipos Worker:** donde se están ejecutando los "End Points".

El entorno de ejecución de los nodos se compone de:
+ *Sistema Operativo:* [CoreOS](https://fedoraproject.org/es/coreos/?target=blank), distribución de Linux especializada en desplegar contenedores. El entorno de ejecución tiene los mismos elementos de un entorno Linux, pero los subsistemas son específicos.
+ *Subsistema Contenedores:* [Podman](https://fedoraproject.org/es/coreos/?target=blank)
+ *Run-time Contenedores:* [CRI-O](https://cri-o.io/?target=blank)
+ *Servicios Comunes k8s:* [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/?target=blank) 
+ *Servicios Worker k8s:* [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/?target=blank) , [kubectl](https://kubernetes.io/es/docs/reference/kubectl/?target=blank) 
+ *Servicios Master k8s:* [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/?target=blank), [kube-controller](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/?target=blank),[kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/?target=blank) 

![Entorno Ejecución](image://intro/k8s_runtime.jpg) 

## Roles: Planos de Control y Operación

El plano de control está en los nodos máster y se ocupa de desplegar *End Points* en los nodos worker, además de actuar cuando cualquier recurso sale del estado deseado en su base de datos etcd. Estos estados son actualizado por los controladores de cada "End Point" o por los operadores.

Las propiedades de cada "End Point" se van anotando en el servicio "Service", que actúa a modo de páginas amarillas. Por el otro lado, el servicio "Router" (o Ingress) da visibilidad externa a los "End Points". 

![Esquema de Roles](image://intro/k8s_arch.jpg) 

## Despliegue de Aplicaciones

En el despliegue de aplicaciones, dos son los elementos a tener en cuenta:
+ **Comunicaciones:** tres subredes... entre contenedores de un mismo POD, entre PODs de un mismo clúster virtual y entre aplicaciones encapsuladas en distintos clústeres virtuales.
+ **Datos:** los *"End Points"* tienen dos modos de funcionamiento:
   - *Efímero:* cuando usa el disco local del nodo del clúster donde se despliega. Cualquier redespliegue, pierde cualquier dato.
   - *Persistente:* cuando el clúster accede a un dispositivo de almacenamiento externo, que monta en todos y cada uno de los nodos del clúster recibiendo el nombre de *"PV=Persistent Volume"* (por ejemplo, un LUN de una SAN). A nivel de clúster virtual se anclan (bound) ciertos PVs a través de *"PVCs=Persistent Volume Claims"* . Los PODs del cluster virtual ven estos PVCs como volúmenes que pueden montar. Es importante llevar el control de qué PVs van a qué namespaces.  

Comunicaciones                                          | Acceso a Datos
--------------------------------------------------------|-----------------------------------
![Verificar Infraestructura](image://intro/k8s_sdn.jpg) | ![Verificar Despliegue](image://intro/k8s_data.jpg)

# Automatización: Factorías DevSecOps

La automatización consta de tres partes:
+ **Día 0 - Ciclo de Vida de Clústeres:** instalación y actualización de los clústeres.
+ **Día 1 - Ciclo de Vida de Entornos:** instalación y actualización de servicios de cabecera, según rol dentro de la tubería.
+ **Día 2 - Configuración de Entornos:** personalizaciones de los disntintos entornos de ejecución (GitOps) y orquestación de la cadena (CI/CD).

Las operaciones de Día 2 son las únicas específicas de cada fábrica, el resto es compartido por todas las fábricas. La [Telco Cloud](https://cntt.readthedocs.io/en/stable-elbrus/common/chapter00.html?target=blank) ofrece las automatizaciones de Día 0 y Día 1 en ciclos de entrega y pruebas estables gracias a un modelo de gobierno.

## Día 0: Ciclo de Vida de Clústeres

### Instalación: PXE, Registry e Ignition Files

Los nodos de los clústeres OpenShift emplean [CoreOS](https://fedoraproject.org/es/coreos/?target=blank), un sistema operativo optimizado para la ejecución de contenedores. Una vez instalado el sistema operativo a través de un proceso PXE, se instala automáticamente el entorno de ejecución a partir de un repositorio de contenedores según lo que se declara en unos ficheros llamados *[Ignition Files](https://docs.fedoraproject.org/es/fedora-coreos/producing-ign/?target=blank)* (equivalentes a fichero [kickstart](https://docs.fedoraproject.org/es/fedora/f27/install-guide/advanced/Kickstart_Installations/?target=blank) o [cloud-init](https://www.ibm.com/docs/es/powervc/2.1.1?topic=linux-installing-configuring-cloud-init-rhel?target=blank) en otras distribuciones). 

En otras palabras, toda instalación OpenShift dependerá de tres elementos:
 + **Servidor PXE:** para instalación inicial del sistema operartivo.
 + **Repositorio de Contenedores:** con las componentes necesarias en cada nodo del clúster.
 + **Ignition Files de cada tipo de Nodo:** donde se declara qué lleva cada entorno de ejecución.
 
La imagen resume los pasos que sigue el proceso de despliegue de los clústeres:

![Instalación Clúster](image://intro/k8s_install.jpg) 

### Cluster API: Ciclo de Vida del Cluster

El proyecto kubernetes define un API estándar llamada **["Clúster API"](https://cluster-api.sigs.k8s.io/?target=_blank)** a través del cuál instalar y actualizar clusteres kubernetes en cualquiera de sus distribuciones (OpenShift, Rancher, etc.). Las diferencias entre distribuciones tan solo radican en qué configura y cómo se configuran los entornos de ejecución de cada tipo de nodo del clúster; para caso OpenShift a través de Ignition Files.

### Herramientas: OpenSourceMANO

Existen herramientas en el mercado, como [OSM](https://www.etsi.org/technologies/open-source-mano?target=_blank) de ETSI, que emplean el ["Clúster API"](https://cluster-api.sigs.k8s.io/?target=_blank) tanto para desplegar como para actualizar redes de clústeres kubernetes sobre distintos proveedores de nubes (pública: Azure, Amazon, Google; privada: OpenStack, en el futuro sistemas operativos de hiperconvergencia) desde un único repositorio Git. Cualquier actualización en los clústers, se despliega automáticamente a través de metodología GitOps:

<div class="grav-youtube"><iframe src="https://www.youtube.com/embed/lsKjblPxCoQ?si=cYEgIfhxRNj6bc6v&amp;start=782" frameborder="0" allowfullscreen=""></iframe></div> 

##  Día 1: Ciclo de Vida de los Entornos

### Inicialización: Operadores con Helm

Los servicios de cabecera de instalan en el Clúster con Helm Charts, tal como se aprecia en este demostración.
Una posibilidad, sería emplear perfiles KSU de OSM para desplegar BigBang en los distintos clústeres, automatizando todo el ciclo de vida, tanto del clúster como su entorno de ejecución (conjunto de operadores necesarios en cada entorno).
 
<div class="grav-youtube"><iframe src="https://www.youtube.com/embed/rfufvM3ktYE?si=XNID8ZcUjedWsyvU&amp;start=1046" frameborder="0" allowfullscreen=""></iframe></div> 
 
##  Día 2: Configuración de Entornos

### GitOps: Configurando la Cadena de Entornos

Las configuraciones de los servicios de cabecera del Clúster ser almacenan en repositorio GIT, mediante API estándar.
Ansible es la herramienta elegida por RedHat para implementar la lógica que aplica esas configuraciones.

### Tekton CI/CD: Coordinando la Cadena de Entornos

En lugar de Jenkins, RedHat emplea Tekton como herramienta de entrega continua. También se declaran las configuraciones de los pipelines CI/CD de todos los clústeres virtuales en repositorio GIT como parte de las definiciones de su entorno.

# Diagnósticos por Capas

##  L1 - RedHat OpenShift

### Casos de Uso 
Para tener bajo control la depuración, se limita el número de casos de uso de cada capa. Para el caso de las capa L1: 

+ **Despliegue Aplicación:** 
   - *Instanciación de contenedores ya construidos:* los procesos de construcción de contenedor lo realizan la capa L2.
+ **Control del Clúster:**
   - *Usuarios:* simplificar, delegar la gestión de usuarios a la capa L3.

### Diagnóstico: Procedimientos Generales

Verificación Infraestructura            | Verificación Despliegues
----------------------------------------|-----------------------------------
![Verificar Infraestructura](image://intro/k8s_infra.jpg) | ![Verificar Despliegue](image://intro/k8s_endpoint.jpg)

### Diagnóstico: Resumen de Procedimientos

Administración Clúster                  | Despliegue Aplicaciones
----------------------------------------|-----------------------------------
![Clúster](image://intro/k8s_admin.jpg) | ![Aplicaciones](image://intro/k8s_app.jpg)

##  L2 - RedHat Pipelines

En proceso de elaboración.

##  L3 - RedHat Service Mesh

En proceso de elaboración.

##  L4 - RedHat Serverless

En proceso de elaboración.