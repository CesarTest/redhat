---
title: GitOps_CD
taxonomy:
    category: docs
---
# Configuraciones: Proyectos de Infraestructura
## Pruebas de Integración en Ansible
+ **Configuraciones de cada infraestructura en un reposotorio GIT,** no hay lógica, solo dependencias (requirements.yaml)... salvo un playbook principal que llama a los roles.

## CD - Continuous Delivery
+ **Pruebas Integración :** antes de pasar a un entorno real, se tiene un entorno de preproducción donde se preparan pruebas de toda la API de declaraciones de infraestructura. 

!! **Nota:** El renderizado YAML a HTML puede agregar identaciones, que hay que ajustar en *vim*. 

!!! **Gestión identaciones en vim:**
!!! + Pulsar <code>v</code> para entrar en Visual Mode, seleccionar lineas con cursores. Teclas <code><</code>,<code>></code> para identar en una dirección u otra. Tecla <code>.</code> para repetir selcción líneas, tecla <code>u</code> para deshacer.
!!! + Alternativamente pulsar <code>3<<</code> o <code>3>></code> para identar tres espacios.

# CD - Continuous Delivery
## PASO 1 - **Clonación del Proyecto GIT**.

### 1. **Usuario Root**
<div class="prism-wrapper"><pre class="language-bash"><code>
    sudo -i
</code> </pre></div>

### 2. **Clonar Repositorio del Ejercicio Anterior**
Si es que no está ya descargado. Hay que sustituir *alumno_infra* por el repositorio de cada uno.  

<div class="prism-wrapper"><pre class="language-bash"><code>
     git clone https://github.com/GitOpsTestsLabs/alumno_infra.git
</code> </pre></div>
+ **Username:** GitOpsTestsLabs
+ **Password:** 

## PASO 2 - Creación Workflow Despliegue Continuo
### 1. **Subir SSH Key a GIT para relaciones confianza con Infraestructura** 

#### 1.1. **Portal Web, crear repositorio:**    
[https://github.com/login](https://github.com/login?target=_blank)
  - *Login:* GitOpsTestsLabs
  - *Password:* GitOps2025..

#### 1.2. **Crear Llaves SSH**
<div class="prism-wrapper"><pre class="language-bash"><code>
     ssh-keygen -t ed25519 -C "cesar.econocom@gmail.com"
</code> </pre></div> 

#### 1.3. **[Cargar llave en GitHub:](https://docs.github.com/es/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?target=_blank)**
<div class="prism-wrapper"><pre class="language-bash"><code>
     cat ~/.ssh/id_ed25519.pub
</code> </pre></div>

### 2. **Preparar Workflow** manual.
**Manualmente se dispara el workflow y se va ajustando.** Un despliegue automatizado simplemente asociaría este workflow a otros eventos.

<div class="prism-wrapper"><pre class="language-bash"><code>
     rm -rf .github/workflows
     mkdir -p .github/workflows
     echo '
     ---
	 name: 'CD - Continuous Delivery'
     on: 
       workflow_dispatch:
         inputs:
            name:
              description: 'CD Manual Testing'
              default: 'Continous Delivery'
              required: true
              type: string
     jobs:
       deploy:
         name: Apply Configurations
         runs-on: ubuntu-latest
         environment: production
         steps:
          - name: Load Source Code to Container
            uses: actions/checkout@main	    
          - name: Set up SSH
            run: |
               echo "${{ secrets.SSH_PRIVATE_KEY }}" > private_key.pem
               chmod 600 private_key.pem
          - name: Install Ansible
            shell: bash
            run: |
                sudo apt update
                sudo apt install -y ansible 
          - name: Install Collections
            shell: bash
            run: ansible-galaxy collection install --upgrade -r collections/requirements.yml 
          - name: Install Roles
            shell: bash
            run: ansible-galaxy role       install           -r roles/requirements.yml			  
          - name: Run Playbook
            shell: bash
            env:
               ANSIBLE_CONFIG: './ansible.cfg'
               ANSIBLE_HOST_KEY_CHECKING: False
            run: |
              cat ansible.cfg
              echo =================================
              ansible-config  init -c ansible.cfg
              ansible-playbook playbooks/main.yml --private-key private_key.pem	 
            ' > .github/worflows/cd.yml 
</code> </pre></div>

### 3. **Subir Workflow a GIT** 
<div class="prism-wrapper"><pre class="language-bash"><code>
     git add .
	 git commit -m "Continous Delivery Workflow Creation"
	 git status
	 git push
</code> </pre></div>

+ **Username:** GitOpsTestsLabs
+ **Password:**

### 4. **Probar Manualmente el Workflow en GIT**
Desde actions, una vez esté bien la sintaxis YAML, se dispara manualmente hasta lograr que lanza el playbook. 
- [https://github.com/GitOpsTestsLabs/alumno_infra/actions/](https://github.com/GitOpsTestsLabs/alumno_infra/actions/?target=_blank)

![API](image://auto/git_action.jpg) 

! Como el repositorio GIT no ve la infraestructura (no tiene acceso a las IPs del inventory)... falla el playbook, debe comprobarse que sí llega a dispararse,aunque falle. 
! En una instalación real debe existir visión directa desde repositorio Git a la maquinaria que se quiere gestionar desde ahí.











