---
title: 1.Instalación
taxonomy:
    category: docs
process:
	twig: true
---

# Preparando el Ordenador Personal

### A) Requisitos Hardware Mínimos 
- Procesador: No demasiado antiguo
- RAM: 16 GB
- Disco Duro: 150 GB libres

### B) Requisitos Software 
- **Acceso a Internet**: Indispensable para acondicionar las Máquinas Virtuales.
- [Windows 10/11](https://www.microsoft.com/es-es/software-download/windows11)
- [VirtualBox >7](https://www.virtualbox.org/wiki/Downloads)
- [Vagrant](https://developer.hashicorp.com/vagrant/install?product_intent=vagrant)
- [MobaXterm](https://mobaxterm.mobatek.net/download.html)
- [Paquetes MobaXterm: Git y Ansible](https://mobaxterm.mobatek.net/plugins.html) 


###  C) Repositorio GIT privado de Indra.
- Acceso desde Intranet, o VPN SACTA desde casa.
- Permisos de acceso al Repositorio GIT

# Levantando la Maqueta

### Pasos para el despliegue de Maqueta
! Precaución: no pueden crearse varias maquetas a la vez en una misma máquina, antes hay que destruirlas con vagrant destroy

+ 1. Abrir Termina MobaXterm local 
+ 2. Clonar el repositorio Git
```bash 
git clone git@gitlab.proteo.internal:cdelgadog/rhcsa-lab.git
```
+ 3. Levantar la maqueta
```bash 
cd rhcsa-lab/vagrant-lab 
vagrant up
```
 
 
 