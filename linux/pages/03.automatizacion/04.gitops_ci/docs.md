---
title: GitOps_CI
taxonomy:
    category: docs
---
# Lógica: Proyectos de Librerías 
## Pruebas Unitarias en Ansible
+ **Cada Rol Ansible se aloja en un repositorio GIT.**
+ **Las Coleccion Ansible es un repositorio GIT**, con procesos de construcción y pruebas de colección a partir de una batería de repositorios Git.

## CI - Continuous Integration
Cada push a GIT del código de un rol, se lanza un workflow de GitHub para cada tipo de pruebas, GitHub indicará los workflows y su cobertura de manera gráfica:
+ **Pruebas Estilo:** Formato y Sintaxis.
+ **Pruebas Unitarias:** asociadas a cada rol Ansible.

!! **Nota:** El renderizado YAML a HTML puede agregar identaciones, que hay que ajustar en *vim*. 

!!! **Gestión identaciones en vim:**
!!! + Pulsar <code>v</code> para entrar en Visual Mode, seleccionar lineas con cursores. Teclas <code><</code>,<code>></code> para identar en una dirección u otra. Tecla <code>.</code> para repetir selcción líneas, tecla <code>u</code> para deshacer.
!!! + Alternativamente pulsar <code>3<<</code> o <code>3>></code> para identar tres espacios.

# CI - Continuous Integration
+ [Workflows CI Ansible](https://ansible.readthedocs.io/projects/molecule/ci/?target=_blank)
![Pruebas Integración](image://auto/github_ci.jpg)

## PASO 1 - Crear Proyecto de Lógica (Role)

### 1.- **Datos de Alumno**
Sustituir *alumno* por nombre deseado. 
<div class="prism-wrapper"><pre class="language-bash"><code>
      export ALUMNO=alumno
</code> </pre></div>

### 2. **Inicializar el Role**
Se crea el role *${ALUMNO}_role* que sobre un servidor LAMP va a desplegar CMS.
<div class="prism-wrapper"><pre class="language-bash"><code>
	  cd $HOME
	  ansible-galaxy collection init indra.xr12
	  cd $HOME/indra/xr12/roles
      ansible-galaxy role init ${ALUMNO}_role
	  cd ${ALUMNO}_role 
</code> </pre></div>

### 3. **Incorporar Playbook Main 'Hello World!'**
<div class="prism-wrapper"><pre class="language-bash"><code>
     echo '
	 ---
     # role tasks/main.yml
     #
     # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
     # SPDX-License-Identifier: Apache-2.0	  	 
	 
     - name: Hello World!
       debug:
         msg: "Hello World!"
	 ' > $HOME/indra/xr12/roles/${ALUMNO}_role/tasks/main.yml
</code> </pre></div>


#### 4. **Dependencias: <code>requirements.yml</code>**
<div class="prism-wrapper"><pre class="language-yaml"><code>
	  echo '
	  ---
      # role requirements.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0		  
      roles:
        - name: geerlingguy.apache
        - name: geerlingguy.php
	  collections:
	    - name: fedora.linux_system_roles
       ' > $HOME/indra/xr12/roles/${ALUMNO}_role/requirements.yml
</code> </pre></div>

#### 5. **Crear Carpeta de Pruebas**
+ 1- **Crear Role de Redirección**, simplemente redirige a las carpetas del Role principal.
<div class="prism-wrapper"><pre class="language-bash"><code>
      mkdir -p  $HOME/indra/xr12/roles/${ALUMNO}_role/tests/roles/indra.xr12.${ALUMNO}_role
	  cd $HOME/indra/xr12/roles/${ALUMNO}_role/tests/roles/indra.xr12.${ALUMNO}_role
	  FOLDERS=( $(ls ../../../) ) 
	  for FOLDER in ${FOLDERS[@]} ; do
	     ln -s ../../../${FOLDER} ${FOLDER} 
	  done
</code> </pre></div>

+ 2- **Crear Playbook de Prueba HelloWorld**.
<div class="prism-wrapper"><pre class="language-bash"><code>
     echo '
     ---
     # role tests/test_HelloWorld.yml
     #
     # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
     # SPDX-License-Identifier: Apache-2.0		 
	 
     - name: Ensure that the role runs with default parameters
       hosts: all
       gather_facts: false
       roles:
         - indra.xr12.${ALUMNO}_role
	 ' > $HOME/indra/xr12/roles/${ALUMNO}_role/tests/test_HelloWorld.yml
</code> </pre></div>

### 6. **Lanzar Playbook de Prueba**
<div class="prism-wrapper"><pre class="language-bash"><code>
     ansible-playbook -i 127.0.0.1, $HOME/indra/xr12/roles/${ALUMNO}_role/tests/test_HelloWorld.yml
</code> </pre></div>
	 
## PASO 2 - Prepara Entorno de Pruebas
+ **Entorno de Pruebas en Docker:** molecule lanza playbooks en contenedores Docker, algo que GitHub puede lanzar en workflows.
+ **Escenarios:** distintas variantes del entorno de pruebas.
+ **Tox en Molecule:** para comprobar distintas versiones de ansible. 

### 1. **Instalar Entorno Ejecución Molecule**
+ [Instalación Molecule](https://ansible.readthedocs.io/projects/molecule/installation/?target=_blank)

Se elimina Podman y se utiliza Docker, por compatibilidad con GitHub actions. 
<div class="prism-wrapper"><pre class="language-bash"><code>
       cd $HOME
	   sudo dnf install -y remove podman
	   sudo dnf -y install dnf-plugins-core
       sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
	   sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
       sudo dnf install -y gcc python3-pip python3-devel openssl-devel python3-libselinux 
	   python -m venv $HOME/ansible-EE
       source $HOME/ansible-EE/bin/activate
	   pip3 install ansible-dev-tools
	   pip3 install molecule
	   pip3 install molecule-docker
	   pip3 install tox-ansible
	   systemctl enable docker --now
	   systemctl status docker
</code> </pre></div>

### 2. **Desplegar Escenario de Pruebas "CentOS"**
#### 2.1.- *Crear el escenario <code>centos</code>:*
<div class="prism-wrapper"><pre class="language-bash"><code>
	   cd $HOME/indra/xr12/roles/${ALUMNO}_role
       molecule init scenario centos	 	   
</code> </pre></div>

#### 2.2.- *Probar Entorno de Pruebas*
<div class="prism-wrapper"><pre class="language-bash"><code>
       molecule destroy -s centos
       rm -Rf $HOME/.cache/molecule
       molecule test -s centos 
	   molecule list
	   molecule destroy -s centos
	   molecule cleanup -s centos
	   rm -Rf $HOME/.cache/molecule
</code> </pre></div>

## PASO 3 - Inicializar Escenario de Pruebas en CentOS

### 1.- Crear Prueba: <code>converge.yml</code>
<div class="prism-wrapper"><pre class="language-yaml"><code>
	  echo '
      ---
      # role molecula/centos/convergence.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0		
	  
      - name: Use Case 1
        import_playbook: ../../tests/test_HelloWorld.yml
		
      - name: Use Case 2
        import_playbook: ../../tests/test_HelloWorld.yml	
        ' > $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos/converge.yml
</code> </pre></div>

### 2.- Lanzar Prueba
<div class="prism-wrapper"><pre class="language-bash"><code>
       molecule destroy -s centos
       rm -Rf $HOME/.cache/molecule
       molecule test -s centos 
	   molecule list
	   molecule destroy -s centos
	   molecule cleanup -s centos
	   rm -Rf $HOME/.cache/molecule
</code> </pre></div>

## PASO 4 - Personalizar Escenario de Pruebas CentOS
Se configura el escenario <code>centos</code>: 
+ a) *Controlador Ansible CentOS*, responsable también de la creación de máquinas donde lanzar playbooks. 
+ b) *Driver Docker*, se crea infraestructura de pruebas mediante contenedores Docker (lo más ampliamente extendido). 
+ [Inicialización Molecule](https://ansible.readthedocs.io/projects/molecule/getting-started/?target=_blank)

Para configurar un escenario se editan los ficheros:
+ **<code>create.yml</code>, playbook de creación del entorno de pruebas:** instancias y sus datos en instance-config.
+ **<code>destroy.yml</code>, playbook de destrucción del entorno de pruebas:**  instnacias y sus datos en instance-config.
+ **<code>molecule.yml</code>, configuracón de herrramientas necesarias del escenario**
+ **<code>converge.yml</code>, pruebas del escenario**, será llamado con ansible-playbook contra las instancias generacdas.
+ **<code>verify.yml</code>, comprobaciones de las pruebas**, verifica el estado resultante de las instancias.
+ **<code>requirements.yml</code>, dependencias**, del rol que se quiere probar.

### 1. Casos de Uso del Role
#### Test:<code>molecule.yml</code>
Los contenedores Docker vienen sin SystemD, para las pruebas es necesario activarlo.
+ [SystemD Container](https://ansible.readthedocs.io/projects/molecule/guides/systemd-container/?target=_blank)

<div class="prism-wrapper"><pre class="language-yaml"><code>
	  echo '
      ---
      # role molecule/centos/molecule.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0
	  
      driver:
        name: ${LSR_MOLECULE_DRIVER:-docker}
        options:
          managed: false
          login_cmd_template: docker exec -ti {instance} bash
          ansible_connection_options:
            ansible_connection: community.docker.docker
		    ansible_become_method: sudo
      dependency:
        name: galaxy
        options:
          role-file: meta/requirements.yml
          requirements-file: molecule/HelloWorld/requirements.yml
      provisioner:
        name: ansible
        log: true
        env:
          ANSIBLE_BECOME: True
          ANSIBLE_BECOME_METHOD: su
          ANSIBLE_BECOME_USER: root
        config_options:
          defaults:
            local_tmp: /tmp/.ansible-$USER/tmp
            remote_tmp: /tmp/.ansible-$USER/tmp
			ansible_become: true
            ansible_become_method: su	
			ansible_become_user: root		
      lint: |
        set -e
        yamllint .
        ansible-lint
      platforms:
      - name: "indra-${image:-centos}-${tag:-stream8}${TOX_ENVNAME}"
        image: "${namespace:-quay.io/centos}/${image:-centos}:${tag:-stream8}"
        command: /sbin/init
        privileged: true
        pre_build_image: true
          #tmpfs:
          #- /run
          #- /tmp
          #volumes:
          #- /sys/fs/cgroup:/sys/fs/cgroup:ro
	  ' > $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos/molecule.yml   
</code> </pre></div>

####  Dependencias: <code>requirements.yml</code>
<div class="prism-wrapper"><pre class="language-yaml"><code>
	  echo '
	  ---
      # role molecule/centos/requirements.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0
	  
      collections:
        - name: community.docker
          version: ">=3.10.4"
	  ' > $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos/requirements.yml   
</code> </pre></div>

#### Comprobaciones
<div class="prism-wrapper"><pre class="language-bash"><code>
       molecule destroy -s centos
       rm -Rf $HOME/.cache/molecule
       molecule test -s centos 
	   molecule list
	   molecule destroy -s centos
	   molecule cleanup -s centos
	   rm -Rf $HOME/.cache/molecule
</code> </pre></div>

### 3. Preparación y Verificación Casos Uso del Role
#### Preparativos: <code>prepare.yml</code>
<div class="prism-wrapper"><pre class="language-yaml"><code>
	  echo '
      ---
      # role molecule/centos/prepare.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0	
	  
      - name: Prepare
        hosts: all
        become: yes
        gather_facts: no
        tasks:
        - name: Test SystemD
          shell: systemctl status
          register: systemd_status
        - name: Mostrar estado de Systemd
          debug:
            msg: "{{ systemd_status.stdout }}"
        - name: List Repos
          ignore_errors: yes
          find:
            paths: /etc/yum.repos.d
            file_type: file
          register: found_repos
        - name: Fix Repos URL
          ignore_errors: yes
          replace:
              path: "{{ repo.path }}"
              regexp: "mirror.centos.org"
              replace: "vault.centos.org"
          loop: "{{ found_repos.files | default([]) }}"
          loop_control:
             loop_var: repo
        - name: Fix Repos baseurl
          ignore_errors: yes
          replace:
              path: "{{ repo.path }}"
              regexp: "#.*baseurl=http"
              replace: "baseurl=http"
          loop: "{{ found_repos.files | default([]) }}"
          loop_control:
             loop_var: repo
        - name: Fix Repos mirrorlist
          ignore_errors: yes
          replace:
              path: "{{ repo.path }}"
              regexp: "mirrorlist=http"
              replace: "##mirrorlist=http"
          loop: "{{ found_repos.files | default([]) }}"
          loop_control:
             loop_var: repo
        - name: Rebuild RPM Database
          ignore_errors: yes
          command: "rpm --rebuilddb"
      
      - name: Prepare
        hosts: all
        become: yes
        gather_facts: yes
        vars:
           #########################################################
           #            APACHE SERVER CONFIGURATIONS
           #########################################################
           apache_user: "apache"
           apache_shell: "/bin/bash"
           apache_group: "{{ apache_user | default(apache)}}"
           apache_options: "All Indexes FollowSymLinks"
           # Los modulos pueden romper la instalacion, si las dependencias estan mal.
           #apache_mods_enabled:
           #    - rewrite
           #    - ssl
           #    - mpm_itk_module
           apache_mods_disabled: []
      
           #########################################################
           #            PHP CONFIGURATIONS FOR GRAV
           #########################################################
           #------------
           # 1.- Main Config
           #------------
           php_user:  "{{apache_user | default(apache) }}"
           php_group: "{{php_user    | default(apache) }}"
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
      
           #------------
           # 2.- PHP ini
           #-------------
           php_memory_limit: "128M"
           php_max_execution_time: "90"
           php_upload_max_filesize: "256M"
      
           #---------
           # 3.- OPT Cache
           #--------
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
        roles:
          - role: geerlingguy.apache
          - role: geerlingguy.php

           ' > $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos/prepare.yml
</code> </pre></div>
         
#### Verificaciones: <code>verify.yml</code>
<div class="prism-wrapper"><pre class="language-yaml"><code>
         	  echo '
               ---
               - name: Verify
                 hosts: all
                 become: no
                 gather_facts: no
               
                 tasks:
                   - name: Check port 80
                     ansible.builtin.wait_for:
                       port: 80
        ' > $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos/verify.yml
</code> </pre></div>

#### Probar el Escenario CentOS
<div class="prism-wrapper"><pre class="language-bash"><code>
       molecule destroy -s centos
       rm -Rf $HOME/.cache/molecule
	   rm -f $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos/create.yml
	   rm -f $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos/destroy.yml  
       molecule test -s centos 
	   molecule list
	   molecule destroy -s centos
	   molecule cleanup -s centos
	   rm -Rf $HOME/.cache/molecule
</code> </pre></div>

#### Depuración Ansible en Contenedor
<div class="prism-wrapper"><pre class="language-bash"><code>
       molecule --debug test -s centos --destroy never
	   docker ps #  docker exec -it <instancia> bash
	   molecule login -s centos
</code> </pre></div>

## PASO 3 - Crear Escenario para GitHub Actions
GitHub Actions emplea Ububtu como controlador Ansible, así que hay que confeccionar otro escenario.

### 1. Clonar escenario CentOS
<div class="prism-wrapper"><pre class="language-bash"><code>
       cp -Rf $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/ubuntu
	   rm -f $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos/create.yml
	   rm -f $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos/destroy.yml	   
       molecule destroy -s ubuntu
       rm -Rf $HOME/.cache/molecule
       molecule test -s ubuntu 
	   molecule list
	   molecule destroy -s ubuntu
	   molecule cleanup -s ubuntu
	   rm -Rf $HOME/.cache/molecule	   
</code> </pre></div>

### 2. Editar Fichero Configuración de Escenario Ubuntu
<div class="prism-wrapper"><pre class="language-yaml"><code>
	  echo '
      ---
      # role molecule/ubuntu/molecule.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0
	  
      driver:
        name: ${LSR_MOLECULE_DRIVER:-docker}
        options:
          managed: false
          login_cmd_template: 'docker exec -ti {instance} bash'
          ansible_connection_options:
            ansible_connection: community.docker.docker
			ansible_become_method: sudo			
      dependency:
        name: galaxy
        options:
          role-file: meta/requirements.yml
          requirements-file: molecule/ubuntu/requirements.yml
      provisioner:
        name: ansible
        log: true
        env:
          ANSIBLE_BECOME: True
          ANSIBLE_BECOME_METHOD: sudo
          ANSIBLE_BECOME_USER: root
          ANSIBLE_LOCAL_TEMP: /tmp/.ansible-$USER/tmp
          ANSIBLE_REMOTE_TEMP: /tmp/.ansible-$USER/tmp
		  MOLECULE_EPHEMERAL_DIRECTORY: /tmp/.molecule
        config_options:
          defaults:
            local_tmp: /tmp/.ansible-$USER/tmp
            remote_tmp: /tmp/.ansible-$USER/tmp
            ansible_local_tmp: /tmp/.ansible-$USER/tmp
            ansible_remote_tmp: /tmp/.ansible-$USER/tmp
            ansible_become_method: sudo
			ansible_become_user: root	
			allow_world_readable_tmpfiles: True
      lint: |
        set -e
        yamllint .
        ansible-lint
      platforms:
      - name: "indra-${image:-centos}-${tag:-stream8}${TOX_ENVNAME}"
        image: "${namespace:-quay.io/centos}/${image:-centos}:${tag:-stream8}"
        command: /sbin/init
        privileged: true
		pre_build_image: true
      scenario:
        name: ubuntu
        test_sequence:
          - syntax
		  - dependency
          - create
		  - prepare
          - converge
          - idempotence
		  - side_effect
          - verify
		  ' > $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/ubuntu/molecule.yml
</code> </pre></div>

### 2. Ajustes del Driver

En algunos Pipelines CI/CD se adapta el driver mediante los siguientes pasos:
+ [Ajustes del Driver para Docker](https://ansible.readthedocs.io/projects/molecule/examples/docker/?target=_blank)


### 3. Probar Ajustes

Si no llegase a la fase de Prepare, se recomienda copiar el contenido del fichero <code>create.yml</code> por el de la página oficial Molecule.
<div class="prism-wrapper"><pre class="language-bash"><code>
       cd $HOME/indra/xr12/roles/${ALUMNO}_role/
       molecule destroy -s ubuntu
       rm -Rf $HOME/.cache/molecule
       molecule test -s ubuntu 
	   molecule list
	   molecule destroy -s ubuntu
	   molecule cleanup -s ubuntu
	   rm -Rf $HOME/.cache/molecule	   
</code> </pre></div>

## PASO 4 - Crear Workflow de Integración Continua
+ [Molecule GitHub Actions Workflow](https://ansible.readthedocs.io/projects/molecule/ci/?target=_blank)
+ [Workflows CI Ansible](https://ansible.readthedocs.io/projects/molecule/ci/?target=_blank)

![Workflow GitHub Actions](image://auto/github-actions-workflow.png)

### 1.- *Creacion Workflow CI*
Se asocia a evento manual, para probarlo manualmente.
<div class="prism-wrapper"><pre class="language-yml"><code>
      mkdir -p $HOME/indra/xr12/roles/${ALUMNO}_role/.github/workflows
	  echo '
        ---
        name: "CI - Continuous Integration"
        on: 
             workflow_dispatch:
                inputs:
                  name:
                     description: "CI Manual Testing"
                     default: "Continous Integration"
                     required: true
                     type: string	  
        jobs:
             test-ansible:
                runs-on: ubuntu-24.04
				ephemeral_directory: /tmp/.molecule
                strategy:
                     fail-fast: false
                     matrix:
                            config:
                              - image: "centos"
                                tag: "stream6"
                              - image: "centos"
                                tag: "stream7"
                              - image: "centos"
                                tag: "stream8"				  
                steps:
                     - name: Checkout repository
                       uses: actions/checkout@main
                       with:
                            path: "${{ github.repository }}"
                     
                     - name: Set up Python ${{ matrix.python-version }}
                       uses: actions/setup-python@v2
        
                     - name: Set up Docker
                       uses: docker/setup-docker-action@v4
                            
                     - name: Install dependencies
                       run: |
                            python3 -m venv $HOME/ansible-EE
                            source $HOME/ansible-EE/bin/activate               
                            python3 -m pip install --upgrade pip
                            python3 -m pip install ansible
                            python3 -m pip install ansible-dev-tools
                            python3 -m pip install molecule
                            python3 -m pip install molecule-docker
                            python3 -m pip install tox-ansible                    
                     - name: Test with molecule
                       run: |
                            source $HOME/ansible-EE/bin/activate
                            cd ${{ github.repository }}
							export MOLECULE_EPHEMERAL_DIRECTORY=/tmp/.molecule
                            image=${{ matrix.config.image }} tag=${{ matrix.config.tag }} molecule test	-s ubuntu
       ' > $HOME/indra/xr12/roles/${ALUMNO}_role/.github/workflows/ci.yml   
</code> </pre></div>

### 2.- **Crear Repositorio GitHub:** 
#### 2.1.- *Inicialización en Local*
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd $HOME/indra/xr12/roles/${ALUMNO}_role/
     rm -rf .git
     git init
     git add .
     git status
     git commit -m "GitOps: Creando Repositorio"
     git config --global user.name "GitOpsTestsLabs"
     git config --global user.email cesar.econocom@gmail.com 
     git branch -M main
</code> </pre></div>

#### 2.2. *Portal Web, crear repositorio: alumno_role*    
[https://github.com/login](https://github.com/login?target=_blank)
  - *Login:* GitOpsTestsLabs
  - *Password:* GitOps2025..

####  2.3. *Crear Llaves SSH*
<div class="prism-wrapper"><pre class="language-bash"><code>
     ssh-keygen -t ed25519 -C "cesar.econocom@gmail.com"
</code> </pre></div> 

####  2.4. *[Cargar llave en GitHub:](https://docs.github.com/es/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?target=_blank)*
<div class="prism-wrapper"><pre class="language-bash"><code>
     cat ~/.ssh/id_ed25519.pub
</code> </pre></div>

### 3.- **Probar Workflow CI en GitHub:** 
#### 3.1. *Subir el Repositorio*
En este caso se creó en la web el repositorio *${ALUMNO}_role*, donde ${ALUMNO} es el nombre del alumno.

+ **1- Agregar Repositorio**
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd $HOME/indra/xr12/roles/${ALUMNO}_role/
     git add .
     git commit -m "Workflow: Test CI Workflow"   
     git remote remove origin
     git remote add origin https://github.com/GitOpsTestsLabs/${ALUMNO}_role.git
</code> </pre></div>	 

+ **2- Push al Repositorio**
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd $HOME/indra/xr12/roles/${ALUMNO}_role/	    
     touch ~/.ssh/config
     echo '
     Host workstation199
        ForwardAgent yes' >>  ~/.ssh/config
     git config --global credential.helper store
     git config --global credential.helper cache
     git push -u origin main
</code> </pre></div>
+ **Username:** GitOpsTestsLabs
+ **Password:** 

#### 3.2.- *Lanzar Workflow CI manualmente:*

![Lanzar Workflow](image://auto/github_ci_run.jpg)

+ **1- Depurar Errores**, metadatos de la Colección.
<div class="prism-wrapper"><pre class="language-yaml"><code>
	  echo '
      ---
      # role meta/main.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0	
	  
      galaxy_info:
        role_name: ${ALUMNO}_role
        namespace: xr12  
        author: ${ALUMNO} <${ALUMNO}@indra.es>
        description: Testing Ansible CI Environment
        company: Indra
        license: BSD-3-Clause 
        min_ansible_version: "2.9"
        platforms:
          - name: Fedora
            versions:
              - all
          - name: EL
            versions:
              - "6"
              - "7"
              - "8"
              - "9"
        galaxy_tags:
          - centos
          - el6
          - el7
          - el8
          - el9
          - el10
          - fedora
          - network
          - networking
          - redhat
          - rhel
          - system
       ' > $HOME/indra/xr12/roles/${ALUMNO}_role/meta/main.yml  
</code> </pre></div>

+ **2- Subir Camnios a GIT y lanzar Workflow**.
<div class="prism-wrapper"><pre class="language-bash"><code>	    
     cd $HOME/indra/xr12/roles/${ALUMNO}_role/
     git add .
     git commit -m "Workflow: Fix Collections Metadata"   	 
     git push -u origin main
</code> </pre></div>

+ **3- Lanzar Workflow CI**.

GitHub Actions requiere de un trabajo adicional de adaptación. 

+ **4- Ajustar evento**, a cada push a la rama principal.
