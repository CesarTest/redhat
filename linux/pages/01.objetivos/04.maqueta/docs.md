---
title: Maqueta
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


1.- **Agregar GIT a /etc/host**
<div class="prism-wrapper"><pre class="language-bash"><code>
     echo '
      172.30.115.21 gitlab.proteo.internal
      172.30.237.188 vcloud.econocom.lab
      172.30.237.188 training.econocom.lab' >> /etc/hosts
</code> </pre></div>

2.- **Modificar Saltos SSH al GIT**
<div class="prism-wrapper"><pre class="language-bash"><code>
     echo '
     Host *
       KexAlgorithms +diffie-hellman-group1-sha1,diffie-hellman-group14-sha1
       HostKeyAlgorithms +ssh-rsa,ssh-dss
     Host 172.30.115.21
       ProxyCommand ssh root@172.30.246.230 nc %h %p
       User cdelgadog
     Host gitlab.proteo.internal
       ProxyCommand ssh root@172.30.246.230 nc %h %p
       User cdelgadog' > $HOME/.ssh/config
     chmod 600 $HOME/.ssh/config
</code> </pre></div>

3.- **Agregar Clave Privada de Acceso a GIt**
<div class="prism-wrapper"><pre class="language-bash"><code>
     echo '-----BEGIN OPENSSH PRIVATE KEY-----
     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
     -----END OPENSSH PRIVATE KEY-----' > $HOME/.ssh/id_rsa
     chmod 600 $HOME/.ssh/id_rsa
     sudo -i
</code> </pre></div>

4.- **Agregar Clave Pública de Acceso a GIT**
<div class="prism-wrapper"><pre class="language-bash"><code>
     echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDR7TUX27vpClGSEQDVTMH5o0OdbYnQF84EguiHiP02Dv6TKBQeTVXvF6yNOTzsoKgLRWrort6hS4eOw2m8OBxUemHH21+p/FcVeA/o5YJvgzgZb/HQfaX2ZZM2teQCiO2vIteeuj+VVwytd+NW+DArFQs6DjtuVW37cd4G+6HY6YxI4aYPlP4M658N/0jasx+qHBcWZoU82+pw0sWN1eh3c9iOZweSOHwOFyVU6z3ID2uumethGgj0yHsvXZtqhGCCJXXxZO7gp6a8Vf0OgRpbDbx6w+OATU4I1Y8r34zni3sq8ESY4+r62LQYqxDOikvfJ89UzFJnUGuJx8MkjBKNqngQ6Jy8hDOPtw00kOzCJasT2Qm+oJshyWeb0wgMCg84GqtCNF3ONrQpYskGviLAvl0TFfQsFX1XPhmqyBb8+r720ZR8yqC5IVAUjUVt1qoRTZE3ZHOACXFx2vcNhxJOPd2/MXz9F11taC8JdfplZhOPGunqF5pfAs8EhSZZZvU= CDELGADO@LAESPF3N9K5L' > $HOME/.ssh/id_rsa.pub
     chmod 600 $HOME/.ssh/id_rsa.pub
</code> </pre></div>

# Levantando la Maqueta

### Pasos para el despliegue de Maqueta
! Precaución: no pueden crearse varias maquetas a la vez en una misma máquina, antes hay que destruirlas con vagrant destroy

+ 1. **Abrir Termina MobaXterm local** 

+ 2. **Limpiar Entorno Trabajo Virtual box**, en <code>C:\Users\<Usuario>\VirtualBox VMs</code> borrar subdirectorio <code>RHCSA9</code> si existiese.

+ 3. **Clonar el repositorio Git**, se supone que el PC tiene ya el acceso GIT configurado (en el PASO 2 se describen los pasos).
<div class="prism-wrapper"><pre class="language-bash"><code>
     git clone git@gitlab.proteo.internal:cdelgadog/rhcsa-lab.git

</code> </pre></div>

+ 4. **Levantar la maqueta**
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd rhcsa-lab/vagrant-lab 
     vagrant up
</code> </pre></div>
 
 