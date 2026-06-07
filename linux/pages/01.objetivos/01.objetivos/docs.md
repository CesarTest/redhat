---
title: Objetivos
taxonomy:
    category: docs
---

# Estructura de las Certificaciones

Las certificaciones de RedHat podrían definirse como una colección de procedimientos validados por el fabricante que permiten administrar cualquier sistema Linux. Aunque existe una amplia gama de especializades, a nivel de infraestructura (donde el fabricante tiene su mayor trascendencia) tres son las piezas principales:

![Certificaciones](image://intro/certificaciones.jpg)

#### Linux
Sistema operativo para arquitecturas von Neumann que siguen los PCs. 

  - **Arquitectura ([EX200](https://www.redhat.com/es/services/training/ex200-red-hat-certified-system-administrator-rhcsa-exam?target=_blank)):** Fundamentos del Sistema Operativo.
  - **Automatización ([EX294](https://www.redhat.com/es/services/training/ex294-red-hat-certified-engineer-rhce-exam-red-hat-enterprise-linux-9?target=_blank)):** Sistema de APIs que permite controlar cada componente del sistema operativo. La piedra angular de todas las automatizaciones es la filosofía GitOps, es prioritario comprender esta filosofía y diseñar toda la lógica de automatización para adaptarse a ella.
  - **Diagnósticos ([EX342](https://www.redhat.com/es/services/certification/rhcs-red-hat-enterprise-linux-diagnostics-and-troubleshooting?target=_blank)):** maletín de herramientas de análisis que permiten establecer protocolos de diagnóstico. Se propone un resumen gráfico para analizar cada parte de una arquitectura von Neumann, que permita un acceso rápido a cada uno de ellos. 
  

#### OpenShift 
Racimo de ordenadores Linux que funcionan como si fueran uno solo para la gestión del ciclo de vida de los contenedores.

  - **Arquitectura ([EX188](https://www.redhat.com/es/services/training/ex188-red-hat-certified-specialist-containers-exam?target=_blank),[EX280](https://www.redhat.com/es/services/training/red-hat-certified-openshift-administrator-exam?target=_blank)):** Anatomía de un cluster kubernetes.
  - **Automatización ([EX380](https://www.redhat.com/es/services/training/ex380-certified-specialist-openshift-automation-exam?target=_blank), [EX328](https://www.redhat.com/es/services/training/ex328-red-hat-certified-specialist-in-building-resilient-microservices-exam?target=_blank), [DO400](https://www.redhat.com/es/services/training/do400-red-hat-devops-pipelines-and-processes-with-jenkins-git-and-test-driven-development?target=_blank)):** Sistema de APIs que permite controlar cada componente del cluster.
  - **Diagnósticos ([EX280](https://www.redhat.com/es/services/training/red-hat-certified-openshift-administrator-exam?target=_blank), [EX288](https://www.redhat.com/es/services/training/ex288-red-hat-certified-openshift-application-developer-exam?target=_blank)):** Maletín de herramientas de análisis que permiten establecer protocolos de diagnóstico. Se propone un resumen gráfico para analizar cada parte de una arquitectura kubernetes, que permita un acceso rápido a cada uno de ellos. 
  
#### OpenStack 
Racimo de ordenadores Linux que funcionan como si fueran uno solo para la gestión del ciclo de vida de máquinas virtuales. 
!!! RedHat ha retirado certificaciones antiguas, el nuevo planteamiento es desplegar OpenStack sobre OpenShift a través de [OpenStack-Helm](https://wiki.openstack.org/wiki/Openstack-helm?target=_blank) (proyecto de AT&T que ellos despliegan con su herramienta [Airship](https://www.airshipit.org/?target=_blank)).
!!! Esta documentación no contempla OpenStack.

  - **Arquitectura ([CL110](https://www.redhat.com/es/services/training/cl110-red-hat-openstack-administration-i?target=_blank)):** Anatomía de OpenStack. 
  - **Automatización ([CL210](https://www.redhat.com/es/services/training/cl210-red-hat-openstack-administration?target=_blank)):** Sistema de APIs que permite controlar cada componente de OpenStack.
  - **Diagnósticos ([CL170](https://www.redhat.com/es/services/training/cl170-openstack-administration-control-plane-management?target=_blank)):** Maletín de herramientas de análisis que permiten establecer protocolos de diagnóstico.    
  

# Bibliografía

#### Documentación Linux
+ [Entorno Ejecución RedHat](https://docs.redhat.com/es-es/documentation/red_hat_enterprise_linux/8/pdf/configuring_basic_system_settings/Red_Hat_Enterprise_Linux-8-Configuring_basic_system_settings-es-ES.pdf?target=_blank)
+ [Subsistema NetworkManager](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/pdf/configuring_and_managing_networking/red_hat_enterprise_linux-8-configuring_and_managing_networking-en-us.pdf?target=_blank)
+ [Subsistema Virtualización](https://docs.redhat.com/es-es/documentation/red_hat_enterprise_linux/8/pdf/configuring_and_managing_virtualization/Red_Hat_Enterprise_Linux-8-Configuring_and_managing_virtualization-es-ES.pdf?target=_blank)
+ [Seguridad en Linux](https://docs.redhat.com/es-es/documentation/red_hat_enterprise_linux/8/pdf/security_hardening/Red_Hat_Enterprise_Linux-8-Security_hardening-es-ES.pdf?target=_blank)
+ [Rendimiento en Linux](https://www.brendangregg.com/linuxperf.html?target=_blank)
+ [Resumen de Comandos Linux](https://phoenixnap.com/kb/linux-commands-cheat-sheet?target=_blank)

#### DevSecOps at Defense Acquisition University (DAU)
+ [Serie 1](https://media.dau.edu/playlist/details/1_iu6ulm7r?target=_blank) : 
     **¿Qué es DevSecOps y por qué nos importa?** 
	 Obtenga más información sobre contenedores, contenedores sidecar, orquestación de contenedores, infraestructura como código, modelo Zero Trust, ingeniería del caos, telemetría, canalización de CI/CD.
+ [Serie 2](https://media.dau.edu/playlist/details/1_wwqs338p?target=_blank) :
     **Iniciativa DevSecOps del Departamento de Defensa**, tecnología moderna, registro DCAR, refuerzo de contenedores, microservicios, malla de servicios, ATO continua, arquitectura DevSecOps, enfoque en capas c-ATO, monitoreo continuo, capacidad de compra, valor para los programas del Departamento de Defensa
+ [Serie 3](https://media.dau.edu/playlist/details/1_0kko71p1?target=_blank) :
     **¿Arquitectura de microservicios?** ¿Microservicios? ¿Malla de servicio? ¿Istio?  ¿Patrón estrangulador?
  