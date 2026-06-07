---
title: Ansible
taxonomy:
    category: docs
---

# Introducción

## Bibliografía
+ [Linux System Roles](https://linux-system-roles.github.io/?target=_blank)
+ [Fedora Linux Roles Collection](https://galaxy.ansible.com/ui/repo/published/fedora/linux_system_roles/?target=_blank)
+ [Demos Linux System Roles](https://github.com/linux-system-roles/linux-system-roles.github.io/tree/main/demo/devconf2021-cz-demo?target=_blank)
+ [Marketplace de GitHub](https://github.com/marketplace?target=_blank)
+ [Git Workflows](https://github.com/marketplace/actions/run-ansible-lint?target=_blank)
+ [Buenas Prácticas](https://redhat-cop.github.io/automation-good-practices/?target=_blank)
+ [CI Changes](https://linux-system-roles.github.io/2020/12/ci-changes?target=_blank)
+ [Ansible Execution Environment](https://github.com/linux-system-roles/ee_linux_system_roles?target=_blank)

## Elementos 

<div class="grav-youtube"><iframe src="https://www.youtube.com/embed/z4ExuSLORJY?si=1rw5U9Xg75ORo4Eq" frameborder="0" allowfullscreen=""></iframe></div> 

Tal com explican en el vídeo, la folosfía GitOps impone un modo de desarrollar el código Asible:

+ **Control - API:** mapeo de una API que declara las configuraciones de infraestructura a variables Ansible; ha de hacerse siguiendo una serie de políticas para adaptarse a las siguientes propiedades del lenguaje:
   - <u>Entorno por Host:</u> cada host tiene su propio entorno, pero existen unas "variables mágicas" que permiten acceder al entorno de cualquier host. 
   - <u>Herencia:</u> El entorno de cada host dependerá de la carga de los distintos YAML, que se hace según un orden de precedencias logrando un esquema de herencias: lo más específico (host_vars -> group_vars -> inventory) sobreescribe lo más genérico.
   - <u>Variables Estáticas:</u> todas las variables son estáticas y no pueden ser modificadas en tiempo de ejecución, aunque es posible crear variables temporales asociadas a determinadas tareas para ejecutar cálculos en tiempo de ejecución.
   - <u>Variables Simples</u> es muy complejo, tanto hacer validaciones de datos, como recorrer modelos de datos complejos. Por este motivo, se descompone la API en muchas variables simples.
   - <u>Prefijos identifican Módulo:</u> las variables llevan prefijos que indican qué módulo va a tratar esos datos.
+ **Operación - Lógica:** no ha de tocarse en producción, solo se modifican las declaraciones.
   - <u>Preferencia por Roles y Colecciones certificadas por fabricante:</u> antes de programar nada, buscar colecciones o roles que estén mantenidas por un fabricante en Automation Hub. Debido a las fuertes inversiones en este tecnología, se están ampliando continuamente el repertorio... además de establecer API estándar para gestionar cada componente del sistema operativo.

Datos: API                             | Lógica: Playbooks, Roles, Collections
---------------------------------------|---------------------------------
![API](image://auto/ansible_datos.jpg) | ![Ansible](image://auto/ansible_logica.jpg)

### Control: Mapeo APIs
La API estándar que define las configuraciones de infraestructura, se mapea en Ansible de la siguiente manera:

+ **inventory:** fichero yaml que puede construirse dinámicamente con los siguientes elementos:
  + *Listado de máquinas agrupados por roles:* servidores web, servidores de aplicaciones, etc.
  + *Variables globales del entorno.* 
+ **group_vars:** carpeta con un fichero yaml o una subcarpeta por cada tipo de máquina donde se declara la configuración del sistema operativo.
+ **host_vars:** configuraciones del hardware de cada máquina. 

Dentro de los proyectos Ansible, se emplea una carpeta por entorno de trabajo (development, production, etc.).

### Operación: Lógica Ansible
La lógica ansible se construye en torno a tres elementos:
+ **Playbooks:** los ejecutables. Pueden representar funciones, o ficheros "main" de otros lenguajes de programación.
+ **Roles:** módulos que configuran cada componente del sistema operativo o del hardware. Su lógica ha de diseñarse completamente desacoplada de la API. En su documentación se debe especificar la lista de variables que emplea y qué parte de la API representa.
+ **Collections:** librerías de módulos Ansible que siguen una estrutura estandarizada.

# Estándares de Programación: Linux System Roles 

!! **Nota:** El renderizado YAML a HTML puede agregar identaciones, que hay que ajustar en *vim*. 

!!! **Gestión identaciones en vim:**
!!! + Pulsar <code>v</code> para entrar en Visual Mode, seleccionar lineas con cursores. Teclas <code><</code>,<code>></code> para identar en una dirección u otra. Tecla <code>.</code> para repetir selcción líneas, tecla <code>u</code> para deshacer.
!!! + Alternativamente pulsar <code>3<<</code> o <code>3>></code> para identar tres espacios.

## PASO 1: Despliegue de la Maqueta

1. **Limpiar Entorno Trabajo Virtual box**, en <code>C:\Users\<Usuario>\VirtualBox VMs</code> borrar subdirectorio <code>RHCSA9</code> si existiese.

2. **Clonar el repositorio Git**, se supone que el PC tiene ya el acceso GIT configurado (en el PASO 2 se describen los pasos).
<div class="prism-wrapper"><pre class="language-bash"><code>
     git clone git@gitlab.proteo.internal:cdelgadog/rhcsa-lab.git
     cd rhcsa-lab/vagrant-lab 
     vagrant up
</code> </pre></div>

3.- **Datos de Alumno**
Sustituir *alumno* por nombre deseado. 
<div class="prism-wrapper"><pre class="language-bash"><code>
      export ALUMNO=alumno
</code> </pre></div>

## PASO 2: Plantilla de Proyecto

1.- **Conectar por SSH**, se puede crear una sesión MobaXterm para facilitar el acceso.
<div class="prism-wrapper"><pre class="language-bash"><code>
     ssh vagrant@192.168.56.149
     sudo -i
</code> </pre></div>

2.- **Agregar GIT a /etc/host**
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

4.- **Agregar Clave Pública de Acceso a GIT*
<div class="prism-wrapper"><pre class="language-bash"><code>
     echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDR7TUX27vpClGSEQDVTMH5o0OdbYnQF84EguiHiP02Dv6TKBQeTVXvF6yNOTzsoKgLRWrort6hS4eOw2m8OBxUemHH21+p/FcVeA/o5YJvgzgZb/HQfaX2ZZM2teQCiO2vIteeuj+VVwytd+NW+DArFQs6DjtuVW37cd4G+6HY6YxI4aYPlP4M658N/0jasx+qHBcWZoU82+pw0sWN1eh3c9iOZweSOHwOFyVU6z3ID2uumethGgj0yHsvXZtqhGCCJXXxZO7gp6a8Vf0OgRpbDbx6w+OATU4I1Y8r34zni3sq8ESY4+r62LQYqxDOikvfJ89UzFJnUGuJx8MkjBKNqngQ6Jy8hDOPtw00kOzCJasT2Qm+oJshyWeb0wgMCg84GqtCNF3ONrQpYskGviLAvl0TFfQsFX1XPhmqyBb8+r720ZR8yqC5IVAUjUVt1qoRTZE3ZHOACXFx2vcNhxJOPd2/MXz9F11taC8JdfplZhOPGunqF5pfAs8EhSZZZvU= CDELGADO@LAESPF3N9K5L' > $HOME/.ssh/id_rsa.pub
     chmod 600 $HOME/.ssh/id_rsa.pub
</code> </pre></div>

5.- **Desplegar Plantilla**, nombre 'infrastructure' puede sustituirse por cualquier otro.
<div class="prism-wrapper"><pre class="language-bash"><code>
      cd $HOME
      git clone git@gitlab.proteo.internal:cdelgadog/ansible-project-template.git
      mv ansible-project-template infrastructure
</code> </pre></div>
 
6.- **Familiarizarse con la estructura de carpetas**
+ ```/hosts``` ... descripción de los entornos de trabajo: prod, dev y test
+ ```/playbooks```... ejecutables
+ ```/roles```... módulos personales
+ ```/roles/requirements.yaml```... dependencias de módulos externos
+ ```/collections```... colecciones personales
+ ```/collections/requirements.yaml```... dependencias de colecciones externar
+ ```/facts.d```... caché de parámetros de facts de los nodos, para no tener que lanzar el "gathering facts" en cada ejecución.

<div class="prism-wrapper"><pre class="language-bash"><code>
     cd $HOME/infrastructure 
     ls -la
</code> </pre></div>

7.- **Ver fichero configuración Ansible**, especial atención al entorno de trabajo.
<div class="prism-wrapper"><pre class="language-bash"><code>
     vi ansible.cfg
</code> </pre></div>
 
## PASO 3: Inventario Máquinas
 
1.- **Crear el fichero inventario y analizar su significado**
<div class="prism-wrapper"><pre class="language-yml"><code>
     echo '---
     all:
        hosts:
          workstation199.lab.example.com:
            ansible_host: 192.168.56.149
            ansible_ssh_common_args: -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no
            ansible_user: vagrant
            ansible_become: true
          servera199.lab.example.com:
            ansible_host: 192.168.56.150
            ansible_ssh_common_args: -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no
            ansible_user: vagrant
            ansible_become: true
          serverb199.lab.example.com:
            ansible_host: 192.168.56.151
            ansible_ssh_common_args: -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no
            ansible_user: vagrant
            ansible_become: true 
        vars:
          metrics_server: servera199.lab.example.com
          logging_server: servera199.lab.example.com
          logging_domain: example.com
        children:
          web_servers:
            hosts:
              serverb199.lab.example.com:
          logging_servers:
            hosts:
              servera199.lab.example.com:
          metrics_servers:
            hosts:
              servera199.lab.example.com:
            vars:
              metrics_monitored_hosts:
               - serverb199.lab.example.com
               - workstation199.lab.example.com
          nfs_servers:
            hosts:
              servera199.lab.example.com:
            vars:
              storage_data_volume: /dev/vdb	
     ' > $HOME/infrastructure/hosts/prod/inventory
</code> </pre></div>

2.- **Corregir configuraciones SSH**
+ 2.1.- *Comprobar las Conexion SSH* (relaciones de confianza, usuarios, etc.)
<div class="prism-wrapper"><pre class="language-bash"><code>
    ansible all -m ping
</code> </pre></div>

+ 2.2.- *Lanzar Ansible con usuario root*
<div class="prism-wrapper"><pre class="language-bash"><code>
     sed  's/ansible_user: vagrant/ansible_user: root/g' -i ./hosts/prod/inventory
     cat  ./hosts/prod/inventory
</code> </pre></div>

+ 2.3.- *Relaciones de Confianza* (password = password)
<div class="prism-wrapper"><pre class="language-bash"><code>
     HOSTS=( 'servera' 'serverb' )
     for HOST in ${HOSTS[@]} ; do
       IP=$(cat /etc/hosts | grep $HOST | awk '{print $1}')
       echo ssh-copy-id root@$IP
       ssh-copy-id root@$IP
     done  
</code> </pre></div>

+ 2.4.- *Agregando Usuario Ansible*, en este caso 'vagrant'. Ejecutar 2 veces.
<div class="prism-wrapper"><pre class="language-bash"><code>
     ansible all -m ansible.builtin.group -a "name=vagrant system=true"
     ansible all -m ansible.builtin.user -a "name=vagrant state=absent" 
     ansible all -m ansible.builtin.user -a "name=vagrant group=vagrant password={{ 'vagrant' | password_hash('sha512', 'mysecretsalt') }}"
     groupadd --system vagrant 
     useradd -m -p $(openssl passwd vagrant) vagrant --groups vagrant
</code> </pre></div>
+ 2.5.- *Creando Relaciones de Confianza* (password = vagrant)
<div class="prism-wrapper"><pre class="language-bash"><code>
     HOSTS=( 'servera' 'serverb' )
     for HOST in ${HOSTS[@]} ; do
       IP=$(cat /etc/hosts | grep $HOST | awk '{print $1}')
       echo ssh-copy-id vagrant@$IP
       ssh-copy-id vagrant@$IP
     done  
     ssh-copy-id vagrant@192.168.56.149 
</code> </pre></div>
+ 2.6.- *Lanzar Ansible con Usuario no root*, en este caso vagrant.
<div class="prism-wrapper"><pre class="language-bash"><code>
     sed  's/ansible_user: root/ansible_user: vagrant/g' -i ./hosts/prod/inventory
     cat  ./hosts/prod/inventory
</code> </pre></div>

+ 2.7.- *Comprobaciones*
<div class="prism-wrapper"><pre class="language-bash"><code>
     ansible all -m ping
</code> </pre></div>

## PASO 4: Playbook Principal

La lógica ejecuta siempre lo mismo en cada hosts, pero aplica los valores que tenga cada host en su entorno, que depende de las declaraciones en el plano de control. 
A pesar de ser una única lógica para todos, se pueden distinguir dos clases de tareas, que en este ejemplos se agrupan en playbooks diferentes ya que facilita probar cada parte del código de manera independiente:
+ *Tareas Generales:* configuraciones del hardware (red, tarjeta de vídeo) y del entorno de ejecución (usuarios, SELinux, firewall, políticas criptográfica, etc.).
+ *Tareas Específicas:* configuraciones de subsistemas Linux de cada tipo de máquina definida en el inventario.
 
### Tareas Generales

1. **Playbook que configura los Entornos de Ejecución**
<div class="prism-wrapper"><pre class="language-yml"><code>
      echo '
      ---
      - hosts: all
        tasks:
        - name: Building Repo File
          copy:
            dest: /etc/yum.repos.d/rpms.repo
            content: |
               [baseos]
               name=CentOS Stream $releasever - BaseOS
               metalink=https://mirrors.centos.org/metalink?repo=centos-baseos-$stream&arch=$basearch&protocol=https,http
               gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
               gpgcheck=0
               repo_gpgcheck=0
               metadata_expire=6h
               countme=1
               enabled=1
               [appstream]
               name=CentOS Stream $releasever - AppStream
               metalink=https://mirrors.centos.org/metalink?repo=centos-appstream-$stream&arch=$basearch&protocol=https,http
               gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
               gpgcheck=0
               repo_gpgcheck=0
               metadata_expire=6h
               countme=1
               enabled=1
            force: yes
        - name: Rebuild RPM Database
          command: "sudo rpm --rebuilddb"
		  become: true
      - hosts: all
        roles:
          - fedora.linux_system_roles.kernel_settings
          - fedora.linux_system_roles.crypto_policies
          - fedora.linux_system_roles.network
          - fedora.linux_system_roles.selinux
          - fedora.linux_system_roles.firewall
          - fedora.linux_system_roles.timesync
          - fedora.linux_system_roles.kdump
          - fedora.linux_system_roles.tlog
          - fedora.linux_system_roles.sshd
          - fedora.linux_system_roles.vpn
      ' > ./playbooks/environment.yml
</code> </pre></div>

2. **Playbook Principal**
<div class="prism-wrapper"><pre class="language-yml"><code>
     echo '
     ---
     - import_playbook: environment.yml
     ' > $HOME/infrastructure/playbooks/main.yml
</code> </pre></div>

### Configurar Dependencias

Se prefieren colecciones Ansible vertificadas por fabricantes antes de desarrollar un rol o una colección propia.

1. **Colecciones**
<div class="prism-wrapper"><pre class="language-yml"><code>
     echo '
     ---
     # Requirements for Ansible Collections
     #
     # Install/upgrade collections:
     #   ansible-galaxy collection install --upgrade -r content/collections/requirements.yml
     # List installed collections:
     #   ansible-galaxy collection list
     # More info:
     #   Using Collections - https://docs.ansible.com/ansible/latest/user_guide/collections_using.html
     #   Galaxy User Guide - https://docs.ansible.com/ansible/latest/galaxy/user_guide.html
     #   Use the `scm` key for repos that do not exist in ansible-galaxy.com!
     #   Use the `include` directive to split a large file into multiple smaller files
     #   https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#installing-multiple-roles-from-multiple-files
     collections:
     - fedora.linux_system_roles
     ' > $HOME/infrastructure/collections/requirements.yml 
</code> </pre></div>

2. **Roles**
<div class="prism-wrapper"><pre class="language-yml"><code>
     echo '
     ---
     # Requirements for Ansible Roles
     #
     # Install/upgrade roles:
     #   ansible-galaxy role install --upgrade -r content/roles/requirements.yml
     # List installed roles:
     #   ansible-galaxy role list
     # More info:
     #   Using Roles - https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html
     #   Galaxy User Guide - https://docs.ansible.com/ansible/latest/galaxy/user_guide.html
     #   Use the `scm` key for repos that do not exist in ansible-galaxy.com!
     #   Use the `include` directive to split a large file into multiple smaller files
     #   https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#installing-multiple-roles-from-multiple-files
     roles:
     - geerlingguy.apache 
     - geerlingguy.php 
     ' > $HOME/infrastructure/roles/requirements.yml 
</code> </pre></div>

+ 3.- **Descargar dependencias:** con el comando *ansible-galaxy*.
<div class="prism-wrapper"><pre class="language-bash"><code>
     ansible-galaxy collection install --upgrade -r collections/requirements.yml
     ansible-galaxy role       install           -r roles/requirements.yml
</code> </pre></div>

### Probar el playbook
<div class="prism-wrapper"><pre class="language-bash"><code>
     ansible-playbook playbooks/main.yml
</code> </pre></div>

## PASO 5: hosts_var - Configuraciones Hardware
 
1.- **Configuraciones de Red de Server A**
<div class="prism-wrapper"><pre class="language-yml"><code>
     mkdir -p hosts/prod/host_vars
     echo '
     ---
     network_connections:
       - name: eth2
         type: ethernet
         state: up
         ip:
           address:
             - "192.168.56.186/24"		
       - name: eth3
         type: ethernet
         ip:
           address:
             - "192.168.56.185/24"	
         state: up
      ' > $HOME/infrastructure/hosts/prod/host_vars/serverb199.lab.example.com.yml
</code> </pre></div>
 
2.- **Aplicar Configuraciones**
<div class="prism-wrapper"><pre class="language-bash"><code>
     ansible-playbook playbooks/main.yml
</code> </pre></div>
 
## PASO 6: group_vars- Configuraciones Sistema Operativo

6.1. **Comunes (all.yml):** entorno de ejecución y subsistemas de todas las máquinas. Obsérvese como la API se descompone en varias varibles Ansible y que estas variables tienen un modelo de datos muy simple (listas simples o diccionarios de un solo nivel).

1. **Se crea fichero:**
<div class="prism-wrapper"><pre class="language-yml"><code>
     echo '
     ---
     kernel_settings_sysctl:
       - name: net.ipv4.tcp_rmem
         value: "4096 87380 16777216"
       - name: net.ipv4.tcp_wmem
         value: "4096 16384 16777216"
       - name: net.ipv4.udp_mem
         value: "3145728 4194304 16777216"
       - name: net.ipv4.conf.default.secure_redirects
         value: '0'
       - name: net.ipv6.conf.all.accept_source_route
         value: '0'
     
     kernel_settings_sysfs:
       - name: /sys/kernel/debug/x86/pti_enabled
         value: 0
       - name: /sys/kernel/debug/x86/retp_enabled
         value: 0
       - name: /sys/kernel/debug/x86/ibrs_enabled
         value: 0
     
     crypto_policies_policy: "DEFAULT:NO-SHA1"
     crypto_policies_reload: true
     tlog_scope_sssd: all
     firewall:
       - port: '44321/tcp'
         state: enabled
       - port: '514/tcp'
         state: enabled
       - port: '20514/tcp'
         state: enabled
       - service: nfs
         state: enabled
       - service: ipsec
         state: enabled
     
     timesync_ntp_servers:
       - hostname: clock.corp.redhat.com
         iburst: yes
         maxpoll: 10
     
     nfs_client_packages:
       - nfs-utils
     nfs_client_directory:
       name: /data
       state: directory
       mode: '0777'
       owner: root
       group: root
     nfs_client_mount:
       name: /data
       src: "{{ hostvars[groups['nfs_servers'][0]]['ansible_eth0']['ipv4']['address'] }}:/data"
       fstype: nfs
       opts: defaults
       state: mounted
     
     sshd_gssapi_authentication: 'yes'
     sshd_gssapi_cleanup_credentials: 'no'
     sshd_kerberos_authentication: 'no'
     sshd_password_authentication: 'no'
     sshd_pubkey_authentication: 'yes'
     sshd_syslog_facility: AUTHPRIV
     sshd_use_dns: 'no'
     sshd_use_pam: 'yes'
     sshd_x11_forwarding: 'no'
     sshd_rekey_limit: '512M 1h'
     sshd_max_sessions: 10
     sshd_max_startups: 10:30:100
     sshd_max_auth_tries: 6
     sshd_client_alive_interval: 600
     sshd_client_alive_count_max: 3
     
     logging_inputs:
       - name: system_input
         type: basics
     logging_outputs:
       - name: relp_client
         type: relp
         target: "{{ logging_server }}"
         tls: true
         ca_cert: /etc/pki/ca-trust/source/anchors/demo-ca.crt
         cert: /etc/pki/tls/certs/{{ inventory_hostname }}.crt
         private_key: /etc/pki/tls/private/{{ inventory_hostname }}.key
         permitted_servers:
           - "*.{{ logging_domain }}"
     logging_flows:
       - name: flow
         inputs: [system_input]
         outputs: [relp_client]
     ' > $HOME/infrastructure/hosts/test/group_vars/all.yaml	
</code> </pre></div>
 
2. **Verificación de Sintaxis:** 
<div class="prism-wrapper"><pre class="language-bash"><code>
     ansible-playbook $HOME/infrastructure/playbooks/main.yml
</code> </pre></div>

 
6.2. **Específicas por Tipo Máquina (tipo.yml):** entorno de ejecución y subsistemas de cada tipo de máquina. Observese el efecto herencia, en los servidores web se aplica una configuración diferente al resto de las máquinas.

1. **Webservers Config**
<div class="prism-wrapper"><pre class="language-yml"><code>
     echo '
	 ---
	 firewall:
       - port: '44321/tcp'
         state: enabled
       - port: '514/tcp'
         state: enabled
       - port: '20514/tcp'
         state: enabled
       - service: http
         state: enabled
       - service: https
         state: enabled  	
     
     apache_listen_ip: "*"
     apache_listen_port: 80
     apache_listen_port_ssl: 443
     
     php_user:  "{{apache_user | default('apache') }}"
     php_group: "{{php_user    | default('apache') }}"
     php_install_from_source: false
     php_enable_webserver: true
     php_webserver_daemon: "httpd"
     php_packages_state: "present"
     php_install_recommends: true
     php_executable: "php"
     php_packages:
         - php
         - php-cli
         - php-common
         - php-devel
         - php-gd
         - php-mbstring
         - php-pdo
         - php-pecl-apcu
         - php-xml
         - php-curl 
         - php-ctype
         - php-dom
         - php-gd
         - php-json
         - php-mbstring
         - php-openssl
         - php-session
         - php-simplexml
         - php-xml
         - php-zip
     
     php_memory_limit: "128M"
     php_max_execution_time: "90"
     php_upload_max_filesize: "256M"
     
     php_opcache_zend_extension: "opcache.so"
     php_opcache_enable: "1"
     php_opcache_enable_cli: "0"
     php_opcache_memory_consumption: "96"
     php_opcache_interned_strings_buffer: "16"
     php_opcache_max_accelerated_files: "4096"
     php_opcache_max_wasted_percentage: "5"
     php_opcache_validate_timestamps: "1"
     php_opcache_revalidate_path: "0"
     php_opcache_revalidate_freq: "2"
     php_opcache_max_file_size: "0"
     ' > $HOME/infrastructure/hosts/test/group_vars/web_server.yaml	
</code> </pre></div>

## PASO 7: Playbooks Específicos

1. **Tareas Grupo Webservers**
<div class="prism-wrapper"><pre class="language-yml"><code>
     echo '
	 ---
     - hosts: webservers
       roles:
         - geerlingguy.apache
         - geerlingguy.php
     ' > $HOME/infrastructure/playbooks/web_server.yml
</code> </pre></div>

2. **Agregar al Playbook Principal**
<div class="prism-wrapper"><pre class="language-yml"><code>
     echo '
     ---
     - import_playbook: environment.yml
     - import_playbook: web_server.yml
     ' > $HOME/infrastructure/playbooks/main.yml
</code> </pre></div>

3. **Probar el playbook**
<div class="prism-wrapper"><pre class="language-bash"><code>
    ansible-playbook $HOME/infrastructure/playbooks/main.yml
</code> </pre></div>

# Estándares de Ejecución: Ansible Execution Environment

## PASO 1: Instalar en Entorno de Ejecución

+ 1. **Usuario Root**
<div class="prism-wrapper"><pre class="language-bash"><code>
    sudo -i
</code> </pre></div>

+ 2. **Instalación del Entorno**
<div class="prism-wrapper"><pre class="language-bash"><code>
     dnf install -y podman python3 python3-pip
     python -m venv $HOME/ansible-EE
     source $HOME/ansible-EE/bin/activate
     pip3 install ansible-navigator
     pip3 install ansible-builder
</code> </pre></div> 

+ 3. **Comprobaciones**
<div class="prism-wrapper"><pre class="language-bash"><code>
     ansible-navigator --version
     ansible-builder --version
	 cd $HOME/infrastructure
</code> </pre></div> 

## PASO 2: Configurar el Proceso Compilación

### 1. **Crear Makefile:** Dentro del proyecto Ansible
<div class="prism-wrapper"><pre class="language-yml"><code>
       echo '
       ---
       version: 3
       
       images:
         base_image:
           name: docker.io/redhat/ubi9:latest
       
       dependencies:
         python_interpreter:
           package_system: python3
         ansible_core:
           package_pip: ansible-core
         ansible_runner:
           package_pip: ansible-runner
         system:
         - openssh-clients
         - sshpass
         galaxy:
           roles:
             - name: geerlingguy.apache 
             - name: geerlingguy.php 
           collections:
             - name: fedora.linux_system_roles
       ' > $HOME/infrastructure/execution-environment.yml
</code> </pre></div>

### 2. **Probar Compilado:** 
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd $HOME/infrastructure
     ansible-builder build --tag lab
</code> </pre></div>

### 3. **Comprobar Compilado:** 
<div class="prism-wrapper"><pre class="language-bash"><code>
     podman image list
</code> </pre></div>

## PASO 3: Lanzar la Ejecución

### 1. **Lanzar Playbook:** 
<div class="prism-wrapper"><pre class="language-bash"><code>
     $HOME/infrastructure
     ansible-navigator run $HOME/infrastructure/playbooks/main.yml --execution-environment-image lab --mode stdout --pull-policy missing --container-options='--user=0'
</code> </pre></div>

# Estándares de Desarrollo: Workflows CI/CD
## PASO 1: Crear Repositorio Git
### 1.- **Crear Repositorio GitHub:** 
#### 1.1.- *Inicialización en Local*
<div class="prism-wrapper"><pre class="language-bash"><code>
     rm -rf .git
     git init
     git add .
     git status
     git commit -m "GitOps: Creando Repositorio"
     git config --global user.name "GitOpsTestsLabs"
     git config --global user.email cesar.econocom@gmail.com 
     git branch -M main
</code> </pre></div>

#### 1.2. *Portal Web, crear repositorio:*    
[https://github.com/login](https://github.com/login?target=_blank)
  - *Login:* GitOpsTestsLabs
  - *Password:* GitOps2025..

####  1.3. *Crear Llaves SSH*
<div class="prism-wrapper"><pre class="language-bash"><code>
     ssh-keygen -t ed25519 -C "cesar.econocom@gmail.com"
</code> </pre></div> 

####  1.4. *[Cargar llave en GitHub:](https://docs.github.com/es/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?target=_blank)*
<div class="prism-wrapper"><pre class="language-bash"><code>
     cat ~/.ssh/id_ed25519.pub
</code> </pre></div>

#### 1.5. *Subir el Repositorio*
En este caso se ceó en la web el repositorio *${ALUMNO}_infra*, cada uno debe crear uno propio.

+ **1- Agregar Repositorio**
<div class="prism-wrapper"><pre class="language-bash"><code>
     git remote remove origin
     git remote add origin https://github.com/GitOpsTestsLabs/${ALUMNO}_infra.git
</code> </pre></div>	 

+ **2- Push al Repositorio**
<div class="prism-wrapper"><pre class="language-bash"><code>	    
     touch ~/.ssh/config
     echo '
     Host workstation199
        ForwardAgent yes' >>  ~/.ssh/config
     git config --global credential.helper store
     git config --global credential.helper cache
     git push -u origin main
</code> </pre></div>
+ **Username:** GitOpsTestsLabs

## PASO 2: Preparar Workflow

+ El [Marketplace de GitHub](https://github.com/marketplace?target=_blank) almacena plantillas de acciones que se pueden usar en las tuberías CI/CD.  
+ Se recomienda basar el desarrollo en workflows verificados por fabricante, como [Linux System Roles](https://linux-system-roles.github.io/2020/12/ci-changes?target=_blank) 

<div class="grav-youtube"><iframe src="https://www.youtube.com/embed/93urFkaJQ44?si=g64j7wvNJLH-tK3G&amp;start=964" frameborder="0" allowfullscreen=""></iframe></div> 

Creando un workflow que verifica la sintaxis del proyecto.

<div class="prism-wrapper"><pre class="language-bash"><code>
     mkdir -p .github/workflows
     echo '
     ---
     on: [push, pull_request]
     jobs:
       build:
         name: Ansible Lint
         runs-on: ubuntu-latest
         steps:
           # Leverages action defined here https://github.com/actions/checkout/blob/main/action.yml
           - name: Checkout repository
             uses: actions/checkout@main
           # Leverages action defined here https://github.com/ansible/ansible-lint/blob/main/action.yml
           - name: Run ansible-lint
             uses: ansible/ansible-lint@main
       ' > .github/worflows/ansible-lint.yml  
</code> </pre></div>

## PASO 3: Probar Workflow

### 1. **Lanzar Workflow**
<div class="prism-wrapper"><pre class="language-bash"><code>
     git add .
     git commit -m "Workflow: Test Workflow for Syntax Verification"	 
</code> </pre></div>

### 2. **Ver resultados del Workflow** 
Para repositorio *alumno* en este ejemplo, reemplazarlo por el repositorio que se haya creado cada uno.
+ [https://github.com/GitOpsTestsLabs/cesar_infra/actions/](https://github.com/GitOpsTestsLabs/cesar_infra/actions/?target=_blank)

### 3. **Ajustar Sintaxis del proyecto y repetir** 
Hasta que supere correctamente las pruebas de sintaxis.
