---
title: PXE
taxonomy:
    category: docs
---

# Introducción
## Objetivos: Metodología de Trabajo.

Integrar todo lo visto anteriormente en un modo de trabajo.

- **OBJETIVOS 1: Servidor PXE desde cero**, esencial entender el proceso de instalación desatendida de sistemas operativos para cualquier operación en centro de datos. La base para boostrapping de clusteres OpenShift. 
- **OBJETIVO 2: Programación TDD en Ansible:** esencial para reusabilidad del código. Cada role Ansible establece una API estándar para la gestión de un subsistema Linux bajo los principios:
  - *Método de desarrollo TDD*, se tomarán varias colecciones Ansible como referencia para ver que todas siguen estos estándares que está promoviendo RedHat.
  - *Pruebas Unitarias* de Roles Ansible, la capacidad de probar de manera aislada cada Role Ansible.
  - *Abstracción* una API estándar para controlar un subsistema, integrando distintos fabricantes, distribuciones y versiones de sistema operativo. Por ejemplo, configurar una tarjeta de vídeo integrando roles de Nvidia y otros fabricantes en una API estándar.
  - *Atomicidad* desacoplar subsistemas. Por ejemplo, tarjeta de vídeo y subsistema X, son roles totalmente independientes. La labor de integración de elementos se realiza a nivel de parametrización de valores en las declaraciones de infraestructura.
- **OBJETIVO 3: GitOps:** integración y despliegue continuos en la gestión de configuraciones de Infraestructura; guarda similitudes con DevOps, como se va a experimentar en este ejemplo.

## Servidor PXE
Tal como se aprecia en la imagen, el proceso PXE consta de tres subsustemas linux: DHCP, bootload (por TFTP) e installer (por HTTP).
+ **CD - Pruebas Integración:** crear proyecto servidor PXE.
+ **CI - Activar Entrega Continua:** pasar roles a una colección ansible, integrar roles a como dependencias del servidor PXE.

# CD Pruebas de Integración: Plataformado de Sistemas Operativos

## PASO 1 - Creación Proyecto Integr### 1.- Creación del Proyecto
+ 1.- **Datos de Alumno**
Sustituir *alumno* por nombre deseado. 
<div class="prism-wrapper"><pre class="language-bash"><code>
      export ALUMNO=alumno
</code> </pre></div>

Variables de Entorno
<div class="prism-wrapper"><pre class="language-bash"><code>
     export WORK_PATH=$HOME/${ALUMNO}_pxe
</code> </pre></div>

+ 2.- **Desplegar Plantilla**, nombre '${ALUMNO}_pxe' puede sustituirse por cualquier otro.
<div class="prism-wrapper"><pre class="language-bash"><code>
      cd $HOME
      git clone git@gitlab.proteo.internal:cdelgadog/ansible-project-template.git
      mv ansible-project-template ${ALUMNO}_pxe
	  rm -rf ${ALUMNO}_pxe/.git
</code> </pre></div>


+ 3.- **Creacion del fichero inventario**
<div class="prism-wrapper"><pre class="language-yml"><code>
     cd ${WORK_PATH}
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
		  pxe_server: servera199.lab.example.com
        children:
          pxe_servers:
            hosts:
              servera199.lab.example.com:
     ' > hosts/prod/inventory
</code> </pre></div>

+ 4.- **Comprobar Inventario**, conexiones ssh, relaciones de confianza, etc.
<div class="prism-wrapper"><pre class="language-bash"><code>
    ansible all -m ping
	ansible pxe_servers -m ping
</code> </pre></div>


+ 5.- **Creacion Playbook principal**, conexiones ssh, relaciones de confianza, etc.
<div class="prism-wrapper"><pre class="language-yml"><code>
     cd ${WORK_PATH}
     echo '
	   ---
       # pxe.check
       - name: assert | Test dhcpd_interfaces
	     hosts: all
		 tasks:
		 - ansible.builtin.debug:
              msg:
              - Hello World! 
     ' > playbooks/main.yml
</code> </pre></div>

+ 6.- **Probar Servidor PXE**
<div class="prism-wrapper"><pre class="language-bash"><code>
    ansible-playbook playbooks/main.yml
</code> </pre></div>

## PASO 2 - Proyectos de Lógica, Subsistemas PXE: DHCP, Bootloader e Installer
Esta paso inicializa los roles de los subsistemas PXE, salvo bootloader que se va a copiar de uno de <code>onf.pxeboot</code>.

### 2.1.- Creación del Role
+  1. **Inicializar el Role**
Se crea el role *${ALUMNO}_role* que sobre un servidor LAMP va a desplegar CMS.
<div class="prism-wrapper"><pre class="language-bash"><code>
      cd ${WORK_PATH}
	  SUBSISTEMAS=( 'dhcp' 'installer' )
	  for SUB in ${SUBSISTEMAS[@]} ; do
        ansible-galaxy role init roles/${ALUMNO}.${SUB}
      done		
</code> </pre></div>

+  2. **Incorporar Playbook Main 'Hello World!'**
<div class="prism-wrapper"><pre class="language-yaml"><code>
     cd ${WORK_PATH}
	 SUBSISTEMAS=( 'dhcp' 'installer' )
     for SUB in ${SUBSISTEMAS[@]} ; do
     echo "
	 ---
     - name: Hello World!
       debug:
         msg: "Hello World! - ${ALUMNO}.${SUB}"
	 " > roles/${ALUMNO}.${SUB}/tasks/main.yml
	 done
</code> </pre></div>

+ **3- Incorporar Metadatos**.
<div class="prism-wrapper"><pre class="language-yaml"><code>
      cd ${WORK_PATH}   
	  SUBSISTEMAS=( 'dhcp' 'installer' )
      for SUB in ${SUBSISTEMAS[@]} ; do	  
	  echo "
      ---
      # pxe meta/main.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0	
	  
      galaxy_info:
        role_name: ${SUB}
        namespace: ${ALUMNO}  
        author: ${ALUMNO} <${ALUMNO}@indra.es>
        description: PXE Server
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
          - pxe
          - redhat
          - rhel
          - system
       " > roles/${ALUMNO}.${SUB}/meta/main.yml
      done	   
</code> </pre></div>

### 2.2.- Sistema de Pruebas del Role
+  1. **Crear Carpeta de Pruebas**, role de redirección al rol principal
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${WORK_PATH}
     SUBSISTEMAS=( 'dhcp' 'installer' )
     for SUB in ${SUBSISTEMAS[@]} ; do
      mkdir -p  roles/${ALUMNO}.${SUB}/tests/roles/${ALUMNO}.${SUB}
	  cd roles/${ALUMNO}.${SUB}/tests/roles/${ALUMNO}.${SUB}
	  FOLDERS=( $(ls ../../../) ) 
	  for FOLDER in ${FOLDERS[@]} ; do
	     ln -s ../../../${FOLDER} ${FOLDER} 
	  done
	 done
</code> </pre></div>

+ 2. **Crear Prueba 'Hello World'**.
<div class="prism-wrapper"><pre class="language-yaml"><code>
     cd ${WORK_PATH}
	 SUBSISTEMAS=( 'dhcp' 'installer' )
     for SUB in ${SUBSISTEMAS[@]} ; do	 
     echo "
     ---
     - name: Shoot the Test of Role |${ALUMNO}.${SUB}|
       hosts: all
       roles:
         - ./${ALUMNO}.${SUB}
	 " > roles/${ALUMNO}.${SUB}/tests/test_role.yml
	 done
</code> </pre></div>

+ 3. **Lanzar Prueba 'Hello World'**
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${WORK_PATH}
	 SUBSISTEMAS=( 'dhcp' 'installer' )
     for SUB in ${SUBSISTEMAS[@]} ; do	 
        ansible-playbook -u vagrant -i 127.0.0.1, roles/${ALUMNO}.${SUB}/tests/test_role.yml
	 done		
</code> </pre></div>

+ 4. **Inicializar Suite de Pruebas del Role con la Prueba 'Hello World'**
<div class="prism-wrapper"><pre class="language-yaml"><code>
	 SUBSISTEMAS=( 'dhcp' 'installer' )
     for SUB in ${SUBSISTEMAS[@]} ; do	
     echo "
     --- 
     - name: Test Suite | Use Case 1
       import_playbook: test_role.yml
	 " > roles/${ALUMNO}.${SUB}/tests/test.yml
	 done
</code> </pre></div>

+  5. **Lanzar Playbook de Prueba de Role**
<div class="prism-wrapper"><pre class="language-bash"><code>
	 SUBSISTEMAS=( 'dhcp' 'installer' )
     for SUB in ${SUBSISTEMAS[@]} ; do	
        ansible-playbook -u vagrant -i 127.0.0.1, roles/${ALUMNO}.${SUB}/tests/test.yml
	done
</code> </pre></div>

### 2.3.- Escenario Molecule para Pruebas Integración Continua
+  1. **Levantar entorno de pruebas**
*Inicializar Molecule*... si da errores, revisar configuraciones de Git.
<div class="prism-wrapper"><pre class="language-bash"><code>
       python -m venv $HOME/ansible-EE
       source $HOME/ansible-EE/bin/activate
	   systemctl enable docker --now
	   
	   # Inicializar Molecule
	   SUBSISTEMAS=( 'dhcp' 'installer' )	   
	   for SUB in ${SUBSISTEMAS[@]} ; do
	      cd ${WORK_PATH}/roles/${ALUMNO}.${SUB}/
          molecule init scenario centos
          molecule test -s centos 	  
	   done
</code> </pre></div>

*Copiar Escenario Existente y Probar*
<div class="prism-wrapper"><pre class="language-bash"><code>   
	   # Probar escenario en todos los subsistemas  
	   for SUB in ${SUBSISTEMAS[@]} ; do
          cd ${WORK_PATH}/roles/${ALUMNO}.${SUB}/	   
          rm -Rf $HOME/.cache/molecule
          \cp -f $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos/molecule.yml molecule/centos/.
          \cp -f $HOME/indra/xr12/roles/${ALUMNO}_role/molecule/centos/requirements.yml molecule/centos/.		  
	      rm -f molecule/centos/create.yml
	      rm -f molecule/centos/destroy.yml
          molecule test -s centos 
	      molecule list
	      rm -Rf $HOME/.cache/molecule
       done		  
</code> </pre></div>

+  2. **Pruebas de Escenario: <code>convergence.yml</code>**
<div class="prism-wrapper"><pre class="language-yaml"><code>
      cd ${WORK_PATH}
	  SUBSISTEMAS=( 'dhcp' 'installer' )	   
	  for SUB in ${SUBSISTEMAS[@]} ; do
	  echo '
      ---
      - name: Test Suite for the Role
        import_playbook: ../../tests/test.yml
	 ' > roles/${ALUMNO}.${SUB}/molecule/centos/converge.yml
	 done
</code> </pre></div>

+  3. **Comprobaciones**
<div class="prism-wrapper"><pre class="language-bash"><code>
      cd ${WORK_PATH}
	  SUBSISTEMAS=( 'dhcp' 'installer' )	   
	  for SUB in ${SUBSISTEMAS[@]} ; do
	   cd roles/${ALUMNO}.${SUB}/
       molecule test -s centos 
	   molecule list
	   rm -Rf $HOME/.cache/molecule
      done	   
</code> </pre></div>

## PASO 3 - Planificación de Casos de Prueba

+ **Subsistema DHCP:**
  - *CASO USO 1* -  BIOS/UEFI32/UEFI64, detección automática.
  - *CASO USO 2* -  MAC Fijas/Variables, ambos casos a la vez.
+ **Subsistema Bootloader:** iPXE es el bootloader más versátil, concebido para PXE.
  - *CASO USO 1* -  iPXE sin Cifrado Comunicaciones
  - *CASO USO 2* -  iPXE con Cifrado Comunicaciones 
+ **Subsistema Installer:** 
  - *CASO USO 1* - Ubuntu/Debian Preseed. 
  - *CASO USO 2* - RedHat/CentOS Kickstart.
  - *CASO USO 3* - CoreOS Ignition.
  - *CASO USO 4* - Cloud-Init. 

## PASO 4 -  Subsistema DHCP
Se toma como referencia:
- [ONF DHCP Ansible Role](https://github.com/OpenNetworkingFoundation/ansible-role-dhcpd?target=_blank)

Objetivos:
- **Elementos de un Role** y para qué sirve cada uno.
- **TDD...** elaboración de roles basados en pruebas, al quedar escritas sirven de ejemplo de uso verificado.
- **Gestión de distribuciones** y versiones de sistemas operativos.
- **El problema de las dependencias** y la importancia de Ansible Execution Environment.

### 4.1.- Lógica del Subsistema
+  1.- **Playbook Subsistema**
<div class="prism-wrapper"><pre class="language-yaml"><code>
     cd ${WORK_PATH}
     echo '
       ---
       # pxe tasks/main.yml
       #
       # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
       # SPDX-License-Identifier: Apache-2.0	  
	  
       # dhcp.check
       - name: assert | Test dhcpd_interfaces
         ansible.builtin.assert:
           that:
             - dhcpd_interfaces is defined
             - dhcpd_interfaces is not none
  	         - dhcpd_interfaces is not string
  	         - dhcpd_interfaces is not mapping
             - dhcpd_interfaces is iterable
           quiet: true	 
    
       - name: assert | Test dhcpd_subnets
         ansible.builtin.assert:
           that:
             - dhcpd_subnets is defined
             - dhcpd_subnets is not none
  	         - dhcpd_subnets is not string
  	         - dhcpd_subnets is mapping
             - dhcpd_interfaces is iterable
           quiet: true	 
    
       - name: include OS-specific vars
         include_vars: "{{ item }}"
         with_first_found:
           - "{{ ansible_distribution }}_{{ ansible_distribution_version }}.yml"
           - "{{ ansible_distribution }}.yml"
           - "{{ ansible_os_family }}.yml"

       # dhcp.install       		   
       - name: Install DHCP Package
	     become: true
         ansible.builtin.package:
           name: "{{ dhcp_package_name }}"
           state: latest

       # dhcp.setup        			       
       - name: Create dhcpd.conf from template
	     become: true	   
         template:
           src: "{{ dhcpd_template_name }}"
           dest: "{{ dhcpd_config_dir }}/{{ dhcpd_config_filename }}"
           backup: true
           mode: "0644"
           owner: root
           group: "{{ dhcpd_groupname }}"
           # validate: 'dhcpd -t -cf %s' # Does not work...
         notify:
           - dhcpd-restart
		   
       - name: dhcpd-start
	     become: true
         service:
           name: "{{ dhcpd_service }}"
           state: started			   
 	 ' > roles/${ALUMNO}.dhcp/tasks/main.yml
</code> </pre></div>

+  2.- **Valores por Defecto**
<div class="prism-wrapper"><pre class="language-yaml"><code>
     cd ${WORK_PATH}
     echo '
      ---
      # pxe defaults/main.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0	  

      # interfaces to listen on
      dhcpd_interfaces: []
		  
      # subnets to serve DHCP on
      dhcpd_subnets: {}
      
      # default PXE and EFI TFTP delivered files
      dhcpd_pxe_filename: "undionly.kpxe"
      dhcpd_efi_filename: "snponly.efi"
      dhcpd_template_name: "dhcpd.conf.j2"
	  dhcpd_config_filename: "dhcpd.conf"      
 	 ' > roles/${ALUMNO}.dhcp/defaults/main.yml	  
</code> </pre></div>


+  3.- **Valores Específicos de Distribucion**
<div class="prism-wrapper"><pre class="language-yaml"><code>
     cd ${WORK_PATH}
     echo '
      ---
      # packages       
	  dhcp_package_name: 
	    - dhcp-server
        
	  # daemon properties
      dhcpd_service: "dhcpd"
	  dhcpd_config_dir: "/etc/dhcp"
	  dhcpd_groupname: "root"
 	 ' > roles/${ALUMNO}.dhcp/vars/RedHat.yml	  
</code> </pre></div>

+  4.- **Handler**
<div class="prism-wrapper"><pre class="language-yaml"><code>
     cd ${WORK_PATH}
     echo '
     ---
     # dhcp handlers/main.yml
     #
     # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
     # SPDX-License-Identifier: Apache-2.0	  
    
     - name: dhcpd-restart
	   become: true
       service:
         name: "{{ dhcpd_service }}"
         state: restarted		 
 	 ' > roles/${ALUMNO}.dhcp/handlers/main.yml	  
</code> </pre></div>


+ 5.- **Dependencias**
*Fichero Dependencias*
<div class="prism-wrapper"><pre class="language-yml"><code>
     cd ${WORK_PATH}
     echo '
     ---
     # dhcp requiremets.yml
     #
     # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
     # SPDX-License-Identifier: Apache-2.0	  
	  
	 # ansible.netcommon for compatibility
     collections:
     - name: ansible.netcommon
       version:  "=7.0.0"
     ' > roles/${ALUMNO}.dhcp/requirements.yml 
</code> </pre></div>

*Fichero Librerías Python*
<div class="prism-wrapper"><pre class="language-yml"><code>
     cd ${WORK_PATH}
     echo '
      # dhcp requiremets.txt
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0	 	 
      
	  # Note: this requirements.txt file is used to specify what dependencies are
      # needed to make the package run rather than for deployment of a tested set of
      # packages.  Thus, this should be the loosest set possible (only required
      # packages, not optional ones, and with the widest range of versions that could
      # be suitable)
      jinja2 >= 3.1.0  # Jinja2 native macro support fixed in 3.1.0
      PyYAML >= 5.1  # PyYAML 5.1 is required for Python 3.8+ support
	  netaddr >= 1.0 # ipaddr filter
     ' > roles/${ALUMNO}.dhcp/requirements.txt
</code> </pre></div>

*Descarga Dependencias*
<div class="prism-wrapper"><pre class="language-yml"><code>
      cd ${WORK_PATH}
      ansible-galaxy collection install --upgrade -r roles/${ALUMNO}.dhcp/requirements.yml
	  python3 -m pip install -r roles/${ALUMNO}.dhcp/requirements.txt
</code> </pre></div>

+  6.- **Plantilla Fichero Configuracion**
No olvidar corregir las comillas de la plantilla.
<div class="prism-wrapper"><pre class="language-jinja2"><code>
     cd ${WORK_PATH}
     echo '
       # dhcp templates/dhcpd.conf.j2 - {{ ansible_managed }}
       {#
       SPDX-FileCopyrightText: © 2020 Open Networking Foundation <support@opennetworking.org>
       SPDX-License-Identifier: Apache-2.0
       #}
       
       # global lease options
       default-lease-time {{ subnet.lease_time | default("240") }};
       max-lease-time {{ subnet.max_lease_time | default("480") }};
       
       # ignore client UID data, which can lead to duplicate leases
       ignore-client-uids true;
       
       # option definitions
       {% if ansible_system != "OpenBSD" %}
       option rfc3442-classless-static-routes code 121 = array of integer 8;
       option client-arch code 93 = unsigned integer 16; # RFC4578
       {% endif %}
       
       {% for subnet, subdata in dhcpd_subnets.items() %}
       subnet {{ subnet | ipaddr('network') }} netmask {{ subnet | ipaddr('netmask') }} {
       
         # routing
       {% if subdata.routers is defined %}
         # custom router IP set
         option routers {{ subdata.routers | map(attribute="ip") | join (",") }};
       {% set r3442ns = namespace(r3442list = []) %}
       {% for rtr in subdata.routers %}
       {% if "rfc3442routes" in rtr %}
       {% set r3442ns.r3442list = r3442ns.r3442list + (rtr | rfc3442_words() ) %}
       {% endif %}
       {% endfor %}
       {% if r3442ns.r3442list %}
         option rfc3442-classless-static-routes {{ r3442ns.r3442list | join(", ") }};
       {% endif %}
       {% else %}
         # first IP address in range used as router
         option routers {{ subnet | ipaddr('next_usable') }};
       {% endif %}
       
         # DNS/naming options
         option domain-name-servers {{ subdata.dns_servers | join(", ") }};
         option domain-name "{{ subdata.dns_search [0] }}";
         option domain-search "{{ subdata.dns_search | join('", "') }}";
       
       {% if subdata.tftpd_server is defined and subdata.tftpd_server %}
         # tftpd options
       {% if ansible_system == "OpenBSD" %}
         filename "{{ subdata.pxe_filename | default(dhcpd_pxe_filename) }}";
       {% else %}
         if option client-arch = 00:07 {
            # amd64 EFI boot
            filename "{{ subdata.efi_filename | default(dhcpd_efi_filename) }}";
         } else {
            # BIOS boot
            filename "{{ subdata.pxe_filename | default(dhcpd_pxe_filename) }}";
         }
       {% endif %}
         next-server {{ subdata.tftpd_server }};
       
       {% endif %}
       {% if subdata.ntp_servers is defined and subdata.ntp_servers %}
         # ntp options
         option ntp-servers {{ subdata.ntp_servers | join(""", """) }};
       
       {% endif %}
       {% if subdata.range is defined %}
         # dynamically assignable range
         range {{ subdata.range | ipaddr('next_usable') }} {{ subdata.range | ipaddr('last_usable') }};
       {% endif %}
       }
       
       {% if subdata.hosts is defined %}
       # hosts for subnet: {{ subdata.dns_search [0] }}
       {% for host in subdata.hosts %}
       host {{ host.name }}.{{ subdata.dns_search [0] }} {
         option host-name "{{ host.name }}";
         fixed-address {{ host.ip_addr }};
         hardware ethernet {{ host.mac_addr | hwaddr('linux') }};
       {% if host.pxe_filename is defined %}
         filename "{{ host.pxe_filename }}";
       {% endif %}
       {% if host.default_url is defined %}
         option default-url "{{ host.default_url }}";
       {% endif %}
       }
       
       {% endfor %}
       {% endif %}
       
       {% endfor %}
 	 ' > roles/${ALUMNO}.dhcp/templates/dhcpd.conf.j2 
</code> </pre></div>

+  7.- **Filtros Jinja2 personalizados**
* Crear el Filtro Jinja2
<div class="prism-wrapper"><pre class="language-python"><code>
     cd ${WORK_PATH}
     mkdir  roles/${ALUMNO}.dhcp/filter_plugins
     echo '
      #!/usr/bin/env python3
      
      # SPDX-FileCopyrightText: © 2021 Open Networking Foundation <support@opennetworking.org>
      # SPDX-License-Identifier: Apache-2.0
      
      # rfc3442_words.py
      # Input: a dict containing a router ip and multiple CIDR format routes
      # Output a list of octets to be appended to option 121 RFC3442 Classless routes option
      #
      # References:
      #  https://tools.ietf.org/html/rfc3442
      #  https://netaddr.readthedocs.io/en/latest/index.html
      
      from __future__ import absolute_import
      import netaddr
      
      
      class FilterModule(object):
          def filters(self):
              return {
                  "rfc3442_words": self.rfc3442_words,
              }
      
          def rfc3442_words(self, var):
      
              words = []
      
              router = var["ip"]
              router_words = netaddr.IPNetwork(router).network.words
      
              for r3442r in var["rfc3442routes"]:
      
                  # add prefix length
                  prefixlen = netaddr.IPNetwork(r3442r).prefixlen
                  words.append(prefixlen)
      
                  # add only the relevant portion of the address, depending ow words
                  (o1, o2, o3, o4) = netaddr.IPNetwork(r3442r).network.words
                  if prefixlen >= 25:
                      words += [o1, o2, o3, o4]
                  elif prefixlen >= 17:
                      words += [o1, o2, o3]
                  elif prefixlen >= 9:
                      words += [o1, o2]
                  elif prefixlen >= 1:
                      words += [o1]
                  # no additional words if prefixlen == 0
      
                  # add router address
                  words += list(router_words)
      
              return words
      
      
      # test when running standalone outside of Ansible
      if __name__ == "__main__":
      
          example = {
              "ip": "192.168.1.10",
              "rfc3442routes": [
                  "10.0.0.0/8",
                  "172.16.0.0/16",
                  "172.17.0.0/22",
                  "172.31.10.0/25",
              ],
          }
      
          jfilter = FilterModule()
      
          words = jfilter.rfc3442_words(example)
      
          print(words)
      
          # verify correct functionality
          assert words == [
              8,
              10,
              192,
              168,
              1,
              10,
              16,
              172,
              16,
              192,
              168,
              1,
              10,
              22,
              172,
              17,
              0,
              192,
              168,
              1,
              10,
              25,
              172,
              31,
              10,
              0,
              192,
              168,
              1,
              10,
          ] ' >  roles/${ALUMNO}.dhcp/filter_plugins/rfc3442_words.py
 </code> </pre></div>
  
 * Regenerar Carpetas de Pruebas
 <div class="prism-wrapper"><pre class="language-bash"><code>
      cd ${WORK_PATH}
      rm -rf roles/${ALUMNO}.dhcp/tests/roles/${ALUMNO}.dhcp
      mkdir -p  roles/${ALUMNO}.dhcp/tests/roles/${ALUMNO}.dhcp
	  cd roles/${ALUMNO}.dhcp/tests/roles/${ALUMNO}.dhcp
	  FOLDERS=( $(ls ../../../) ) 
	  for FOLDER in ${FOLDERS[@]} ; do
	     ln -s ../../../${FOLDER} ${FOLDER} 
	  done
</code> </pre></div>

+  8.- **Comprobaciones de Sintaxis**
Solo comprobar que dispara el rol, al margen del resultado.
<div class="prism-wrapper"><pre class="language-bash"><code>
        cd ${WORK_PATH}/roles/${ALUMNO}.dhcp/
        ansible-playbook -u vagrant -i 192.168.56.150, tests/test.yml
</code> </pre></div>
 
### 4.2.- Prueba Subsistema 
+  1.- **Dependencias: <code>test_dependencies.yml</code>**
Se incluyen los cuatro casos de prueba en una sola, durante la fase de verificación se distinguirá cada uno de ellos.
<div class="prism-wrapper"><pre class="language-yaml"><code>
      cd ${WORK_PATH}
	  echo '
      ---
      # dhcpd tests/test_dependencies.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0	  
	  
      - name: "Test Dependencies"
        hosts: localhost
        tasks:
          - name: make sure packages required are installed on this control node
            pip:
              name:
               - netaddr
            check_mode: true
            register: package_state
          - name: "Check Python Modules Installed"
            ignore_errors: yes
            debug:
              msg:
               - "{{ package_state | default('vacio')}}"
          - name: "Test ipaddr Jinja2 filter"
            ignore_errors: yes
            debug:
              msg:
               - "{{ ip_addr }}"
            vars:
              ip_addr: "{{ '10.0.0.10/24' | ipaddr('network') }}"
          - name: "Test custom Jinja2 filters: rfc3442_words"
            ignore_errors: yes
            debug:
              msg:
               - "{{ filter_plugins }}"
            vars:
			  indices:
			    routers:
                  - ip: "192.168.0.1/24"
              filter_plugins: "{{ indice |  rfc3442_words}}"
        ' > roles/${ALUMNO}.dhcp/tests/test_dependencies.yml
</code> </pre></div>

+  2.- **Pruebas de Dependencias**
<div class="prism-wrapper"><pre class="language-bash"><code>
        cd ${WORK_PATH}/roles/${ALUMNO}.dhcp/
        ansible-playbook -u vagrant -i 127.0.0.1, tests/test_dependencies.yml
</code> </pre></div>

+  3.- **Caso Uso CI: <code>test_ci.yml</code>**
Una prueba puede validar varios casos de uso, simplemente agregando diferentes modos de validación en su playbook.
<div class="prism-wrapper"><pre class="language-yaml"><code>
      cd ${WORK_PATH}
	  echo '
      ---
      # dhcpd tests/verify.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0	  
	  
	  - import_playbook: test_role.yml
	    vars:
           dhcpd_interfaces:
             - "{{ ansible_default_ipv4.interface | default('eth0') }}"
           dhcpd_subnets:
             "192.168.56.0/24":
               range: "192.168.56.128/25"
               dns_servers:
                 - "192.168.0.1"
                 - "192.168.0.2"
               dns_search:
                 - "lab.example.com"
               tftpd_server: "192.168.56.149"
               hosts:
                 - name: "serverc"
                   ip_addr: "192.168.56.150"
                   mac_addr: "08:00:27:a3:64:b0"
               routers:
                 - ip: "192.168.56.254"
		# Different Test Cases Verification should be here here
        ' > roles/${ALUMNO}.dhcp/tests/test_ci.yml
</code> </pre></div>

+ 4. **Modificar Suite de Pruebas DHCP**
<div class="prism-wrapper"><pre class="language-yaml"><code>
     cd ${WORK_PATH}
     echo "
     --- 
     - name: DHCP - CI Use Case
       import_playbook: test_ci.yml
	 " > roles/${ALUMNO}.dhcp/tests/test.yml
</code> </pre></div>

+  5.- **Probar DHCP contra Máquina Vagrant**
<div class="prism-wrapper"><pre class="language-bash"><code>
        cd ${WORK_PATH}/roles/${ALUMNO}.dhcp/
        ansible-playbook -u vagrant -i 192.168.56.150, tests/test.yml
</code> </pre></div>

+  6.- **Comprobaciones Manuales**
Antes de automatizar las verificaciones de los 4 casos de prueba, se hacen manualmente.
<div class="prism-wrapper"><pre class="language-bash"><code>
       ansible -u vagrant -i 192.168.56.150, all -m command -a "sudo systemctl status dhcpd" -vvv
	   ansible -u vagrant -i 192.168.56.150, all -m command -a "sudo journalctl -xeu dhcpd.service" -vvv
	   ansible -u vagrant -i 192.168.56.150, all -m command -a "sudo cat /etc/dhcp/dhcpd.conf" -vvv
	   ansible -u vagrant -i 192.168.56.150, all -m command -a "sudo ss -tupln" -vvv
</code> </pre></div>

### 4.3.- Automatización Pruebas para Integración Continua 

+  1.- **Automatizar las Comprobaciones**
*Verificaciones*
<div class="prism-wrapper"><pre class="language-bash"><code>
      cd ${WORK_PATH}
	  echo '
       ---
       # dhcpd tests/verify.yml
       #
       # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
       # SPDX-License-Identifier: Apache-2.0	    
	   
       - name: Verify
         hosts: all      
         tasks:      
         - name: Populate service facts
           service_facts:
         - name: dhcp-server is running
           assert:
             that: ansible_facts.services["dhcpd.service"]["state"] == "running"
        ' > roles/${ALUMNO}.dhcp/tests/verify.yml	 
</code> </pre></div>

*Automatización de Verificaciones de Escenario: <code>verify.yml</code>*
<div class="prism-wrapper"><pre class="language-yaml"><code>
      cd ${WORK_PATH}
	  SUBSISTEMAS=( 'dhcp' )	   
	  for SUB in ${SUBSISTEMAS[@]} ; do
	  touch roles/${ALUMNO}.${SUB}/tests/verify.yml
	  echo '
      ---     	  
      # dhcpd molecule/centos/verify.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0	
      - name: Test Suite for the Role
        import_playbook: ../../tests/verify.yml
	 ' > roles/${ALUMNO}.${SUB}/molecule/centos/verify.yml
	 done
</code> </pre></div>

+  2.- **Preparar el Contenedor para Prueba**
<div class="prism-wrapper"><pre class="language-yaml"><code>
      cd ${WORK_PATH}
	  echo '
        ---
        # dhcpd molecule/centos/prepare.yml
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
        
        - name: Dependencies
          import_playbook: ../../tests/test_dependencies.yml
		  
        - name: Create Interface
          hosts: all
          become: true
		  gather_facts: yes
          tasks:
          - name: Create a bridge to nowhere so dhcpd can start during testing
            when: "'bridge0' not in ansible_interfaces"
            command:
              cmd: "{{ item }}"
            with_items:
              - "ip link add bridge0 type bridge"
              - "ip addr add 192.168.56.150/24 dev bridge0"
              - "ip link set bridge0 up" 		  
        ' > roles/${ALUMNO}.dhcp/molecule/centos/prepare.yml
</code> </pre></div>

+  3.- **Lanza Caso de Prueba**
<div class="prism-wrapper"><pre class="language-bash"><code>
       python -m venv $HOME/ansible-EE
       source $HOME/ansible-EE/bin/activate
	   systemctl enable docker --now	   
	   cd ${WORK_PATH}/roles/${ALUMNO}.dhcp/
       molecule test -s centos 
	   molecule list
	   rm -Rf $HOME/.cache/molecule	   	   
</code> </pre></div>

## PASO 5 - Integrar Subsistema DHCP en Servidor PXE

### 1.- Subsistema DHCP
+  1.- **Configuraciones: group_vars**
<div class="prism-wrapper"><pre class="language-yaml"><code>
      cd ${WORK_PATH}
	  echo '
      ---
	  ### dhcp server
	  dhcpd_pxe_filename: "ipxe/bios/undionly.kpxe"
      dhcpd_efi_filename: "ipxe/efi32/snponly.efi"
	  dhcpd_efi64_filename: "ipxe/efi64/snponly.efi"
      dhcpd_interfaces:
        - "{{ ansible_default_ipv4.interface | default(""eth0"") }}"
	  dhcpd_tftpd_server: "192.168.56.150"
      dhcpd_subnets:
        "192.168.56.0/24":
          range: "192.168.56.128/25"
          dns_servers:
            - "{{ dhcpd_tftpd_server }}"
          dns_search:
            - "indra.lab"
          tftpd_server: "{{ dhcpd_tftpd_server }}"
          hosts:
            - name: "serverc"
              ip_addr: "192.168.56.129"
              mac_addr: "08:00:27:a3:64:b0"
          routers:
            - ip: "192.168.56.254"
        ' > hosts/prod/group_vars/pxe_servers.yml
</code> </pre></div>

+  2.- **Playbook Principal**
<div class="prism-wrapper"><pre class="language-yml"><code>
     cd ${WORK_PATH}
     echo "
	   ---
       # pxe.check
       - name: DHCP Server
	     hosts: pxe_servers
         roles:
            - name: ${ALUMNO}.dhcp
     " > playbooks/main.yml
</code> </pre></div>

+  3.- **Comprobaciones**
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${WORK_PATH}
     rm -Rf facts.d
	 sed -i 's/explicit/smart/g' ansible.cfg
     ansible-playbook playbooks/main.yml
</code> </pre></div>
 
 
### 2.- Dependencias Servidor PXE
+  1.- **Colecciones**
*Colecciones*
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
     - robertdebock.roles
     ' > ${WORK_PATH}/collections/requirements.yml 
</code> </pre></div>

+  3.- **Dependencias para Molecule**
<div class="prism-wrapper"><pre class="language-yml"><code>
     echo '
     ---
     collections:
     - fedora.linux_system_roles	 
     - robertdebock.roles
     ' >  ${WORK_PATH}/requirements.yml 
</code> </pre></div>

+ 3.- **Descargar dependencias:** con el comando *ansible-galaxy*.
<div class="prism-wrapper"><pre class="language-bash"><code>
     ansible-galaxy collection install --upgrade -r collections/requirements.yml
	 for ROLE in $(ls -d roles/*/) ; do
         ansible-galaxy collection install --upgrade -r $ROLE/requirements.yml
	     ansible-galaxy role       install           -r $ROLE/requirements.yml
		 python3 -m pip            install           -r $ROLE/requirements.txt
	 done	 
</code> </pre></div>

### 3.- Lógica Servidor PXE
+ 1. ** Playbook main.yml**, este proyecto se centra en servidores PXE.
<div class="prism-wrapper"><pre class="language-yml"><code>
     cd ${WORK_PATH}
     echo "
     ---
     - name: PXE Server
       hosts: pxe_servers
       roles:
       - role: robertdebock.roles.bootstrap
       - role: robertdebock.roles.epel
       - role: robertdebock.roles.buildtools
       - role: robertdebock.roles.python_pip
         vars:
           python_pip_update: false
       - role: robertdebock.roles.openssl
         vars:
           openssl_items:
           - name: apache-httpd
             common_name: "{{ ansible_fqdn }}"
       - role: robertdebock.roles.core_dependencies
       - role: fedora.linux_system_roles.firewall		 
       - role: robertdebock.roles.dnsmasq # tftp + dns
       - role: ${ALUMNO}.dhcp
       - role: robertdebock.roles.httpd
       #- role: ${ALUMNO}.bootloader	   
     " > playbooks/main.yml
</code> </pre></div>

+  2.- **Comprobaciones**
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${WORK_PATH}
     rm -Rf facts.d
	 sed -i 's/explicit/smart/g' ansible.cfg
     ansible-playbook playbooks/main.yml
</code> </pre></div>

### 4.- Configuraciones Servidor PXE
+  1.- **Configuraciones: group_vars**
<div class="prism-wrapper"><pre class="language-yaml"><code>
      cd ${WORK_PATH}
	  echo '
 	  ### entorno ejecucion
      bootloader_tftpd_server: "192.168.56.150"
	  bootloader_tftpd_server_network: "192.168.56.0/24"
	  bootloader_domain: "indra.lab"
	  bootloader_web_server: "http://{{ bootloader_tftpd_server }}"
      core_dependencies_shared_packages:
        - acl
        - at
        - bzip2
        - bash
        - git
        - gzip
        - tar
        - unzip
        - python3-policycoreutils
		
      firewall:
        - port: '53/udp'
          state: enabled
          permanent: true		  
        - port: '67/udp'
          state: enabled
          permanent: true		  
        - port: '68/udp'
          state: enabled
          permanent: true		  
        - port: '69/udp'
          state: enabled
          permanent: true		  
        - port: '80/tcp'
          state: enabled
          permanent: true		  
        - port: '443/tcp'
          state: enabled		  
          permanent: true
		  
	  ### dhcp server
	  dhcpd_pxe_filename: "ipxe/bios/undionly.kpxe"
      dhcpd_efi_filename: "ipxe/efi32/snponly.efi"
	  dhcpd_efi64_filename: "ipxe/efi64/snponly.efi"
      dhcpd_interfaces:
        - "{{ ansible_default_ipv4.interface | default('eth0') }}"
	  dhcpd_subnets:
        "192.168.56.0/24":
          range: "192.168.56.128/25"
          dns_servers:
            - "{{ bootloader_tftpd_server }}"
          dns_search:
            - "{{ bootloader_domain }}"
          tftpd_server: "{{ bootloader_tftpd_server }}"
          hosts:
            - name: "serverc"
              ip_addr: "192.168.56.100
              mac_addr: "08:00:27:a3:64:b0"
          routers:
            - ip: "192.168.56.254"
	  
 	  ### tftp + dns server
      dnsmasq_enable_tftp: true
      dnsmasq_tftp_root: "/var/ftp"
      dnsmasq_tftp_no_fail: false
      dnsmasq_tftp_secure: false
      dnsmasq_tftp_no_blocksize: false 
      dnsmasq_domains:
        - name: "{{ bootloader_domain }}"
          subnet: "{{ bootloader_tftpd_server_network }}"
          domain: "{{ bootloader_domain }}"
  
	  ### http   
	  ### bootloader
      bootloader_web_root: "/var/www/html"
      #bootloader_ipxe_tarball: "ipxe_x64.tgz"
   
      # dist dir, where downloaded files are stored
      bootloader_dist_dir: "{{ dnsmasq_tftp_root }}"
      bootloader_boot_images:
       - memtest 
       - debian11
       - ubuntu1804
       - ubuntu2004
       - openbsd70	   	  
      bootloader_hosts:
      - {domain: "{{ bootloader_domain }}", hostname: 'cwp60', mac: '08:00:27:a3:64:b0' }
      ' > hosts/prod/group_vars/pxe_servers.yml
</code> </pre></div>


+  2.- **Comprobaciones**
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${WORK_PATH}
     rm -Rf facts.d
	 sed -i 's/explicit/smart/g' ansible.cfg
     ansible-playbook playbooks/main.yml
</code> </pre></div>
     
## PASO 6 - Subsistema Bootloader

### 1.- Configuración Role PXE
+  1.- **Crear Lógica de Role**, basandose en el rol de Open Networking Foundation
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${WORK_PATH}
     ansible-galaxy role  install onf.ansible_role_pxeboot --ignore-errors
	 mv -f roles/onf.ansible_role_pxeboot roles/${ALUMNO}.bootloader
</code> </pre></div>

+  2.- **Adaptar el Role**, en este caso a instación en RedHat.
*Variables Especificas*
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${WORK_PATH}/roles/${ALUMNO}.bootloader
     cp vars/Debian.yml vars/RedHat.yml
	 touch tasks/RedHat.yml
	 find ./ -type f | xargs sed -i 's/pxeboot_/bootloader_/g'
	 sed -i 's/www-data/apache/g' vars/RedHat.yml 
</code> </pre></div>

*Variables por Defecto*
<div class="prism-wrapper"><pre class="language-bash"><code>
      echo '
      ---
      # bootloader defaults/main.yml
      #
      # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
      # SPDX-License-Identifier: Apache-2.0	
      
      # webserver that serves the chainload script
      bootloader_web_server: "http://boot.example.com"
      bootloader_web_root: "/srv/sites/boot.example.com"
      
      # dist dir, where downloaded files are stored
      bootloader_dist_dir: "/opt/dist/pxeboot"
      
      # List of boot images to include
      bootloader_boot_images:
        - memtest
        - debian11
        - ubuntu1804
        - ubuntu2004
        - openbsd77
      
      # whether or not to include debug options
      bootloader_image_debug: true
      
      # memtest image
      #  see: https://ipxe.org/appnote/memtest
      bootloader_memtest_version: "5.01"
      bootloader_memtest_checksum: "sha256:42b8da364fec58de1ea8d8ec8102bc3d2e7b1f10e8e63b55a2d8f7aba3fad715"
      
      # this beta version crashes on some systems...
      #  bootloader_memtest_version: "5.31b"
      #  bootloader_memtest_checksum: >
      #    "sha256:ef0f31be2f7d72ceac3e9382ff8668b9ace4881eed7.7cc733756bfdb1cb427a"
      
      bootloader_syslinux_version: "5.10"
      bootloader_syslinux_checksum: "sha256:d9cd7727bfed2c0ca5bf07bb3d213286e014a78e92a6a89ac32eb906d6b7ab3f"
      
      # Debian 11 image
      bootloader_debian11_base_url: "https://deb.debian.org/debian/dists/bullseye/main/installer-amd64"
      bootloader_debian11_version: "20210731+deb11u12/images/netboot/debian-installer/amd64/"
      
      # checksums from version as of 2022-06-20
      bootloader_debian11_files:
        - name: "linux"
          checksum: "sha256:f5f9438227ff8aad85c622b89f88a278413c19c5f64bae85b365f6f34405f9d4"
        - name: "initrd.gz"
          checksum: "sha256:bcc8e9c35a5e54a2e76ac759011ee1f9ce37825aaba159647e59c1952a530894"
      
      bootloader_debian11_linux_args: ""
      
      bootloader_debian11_nonfree_url: "http://cdimage.debian.org/cdimage/unofficial/non-free/"
      
      bootloader_debian11_nonfree_files:
        - path: "firmware/bullseye/20220709"
          name: "firmware.cpio.gz"
          checksum: "sha256:290989bb6d96420f95bcc0c993c462b9cace2afaaf872feb7f00a2ac299ebfe1"
      
      # Ubuntu 18.04 image
      bootloader_ubuntu1804_base_url: "http://archive.ubuntu.com/ubuntu/dists/bionic-updates/main/installer-amd64"
      bootloader_ubuntu1804_version: "20101020ubuntu543.19/images/hwe-netboot/ubuntu-installer/amd64"
      
      # checksums from version as of 2021-11-27
      bootloader_ubuntu1804_files:
        - name: "linux"
          checksum: "sha256:cf73517963037c823e4fcad52b92c20525071957f85fe9516f0652d38488db3b"
        - name: "initrd.gz"
          checksum: "sha256:d7609e66d70189ccf4369c3e832d4a8e6a459de28a32b9ab72f1f8eaa82869e3"
      
      bootloader_ubuntu1804_linux_args: ""
      
      # Ubuntu 20.04 image
      bootloader_ubuntu2004_base_url: "http://archive.ubuntu.com/ubuntu/dists/focal-updates/main/installer-amd64"
      bootloader_ubuntu2004_version: "current/legacy-images/netboot/ubuntu-installer/amd64"
      
      # checksums from version as of 2021-11-27
      bootloader_ubuntu2004_files:
        - name: "linux"
          checksum: "sha256:cd6e8d6114d11d668aaba86e5e16c24762723d37fa9fd5b309dc1b8a0c2685ef"
        - name: "initrd.gz"
          checksum: "sha256:b564dd19ab43aadefe901e1f971ef34c60371f1abeb316cc623adf55976704b3"
      
      bootloader_ubuntu2004_linux_args: ""
      
      # OpenBSD 7.7 image
      bootloader_openbsd70_base_url: "https://cdn.openbsd.org/pub/OpenBSD"
      bootloader_openbsd70_version: "7.7/amd64"
      
      # checksums from version as of 2021-11-27
      bootloader_openbsd70_files:
        - name: "cd77.iso"
          checksum: "sha256:b512030f1494d0f9509b5a2ed92372975d25f4bc4cf71e43dfadea3a31d308b0"
      
      # preseed config
      # this should be replaced with a modular crypt string, or login will not work.
      preseed_onfadmin_pw_crypt: "!!"
      
      # this needs to be set as pubkey auth is the only remote login method
      preseed_onfadmin_ssh_pubkey: ""
      
      # list of hosts
      bootloader_hosts:
        - serial: "12345678"
          mac: "a1:b2:c3:d4:e5:f6"
          hostname: "host"
          domain: "domain"
	 ' >  defaults/main.yml
</code> </pre></div>

+  3.- **Tareas Específicas de RedHat**.
<div class="prism-wrapper"><pre class="language-yml"><code>
     echo '
     ---
     # bootloader tasks/RedHat.yml
     #
     # SPDX-FileCopyrightText: © 2025 Econocom Systems <support@econocom.com>
     # SPDX-License-Identifier: Apache-2.0		 
     - name: Copy iPXE Tarball
       copy:
         remote_src: false
         src: "{{ bootloader_ipxe_tarball }}"
         dest: "/tmp"
         owner: "{{ bootloader_username }}"
         group: "{{ bootloader_groupname }}"
         mode: 0644
       when: bootloader_ipxe_tarball is defined
	 
     - name: Unarchive iPXE Tarball
       unarchive:
         remote_src: true
         src: "/tmp/{{bootloader_ipxe_tarball }}"
         dest: "{{ bootloader_dist_dir }}"
         owner: "root"
         group: "root"
         mode: "0644"
	   when: bootloader_ipxe_tarball is defined	 

     - name: Bootloader Domain Activation
	   become: yes
       ansible.builtin.lineinfile:
         path: /etc/hosts
         line: {{ bootloader_tftpd_server  | default('localhost') }} {{ bootloader_domain | default('indra.lab') }}
         create: yes
	   when: 
	     - bootloader_tftpd_server is defined	 	   
	     - bootloader_domain is defined	 		 
	 ' >>  tasks/RedHat.yml
</code> </pre></div>
	 
+  4.- **Comprobaciones**
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${WORK_PATH}
     rm -Rf facts.d
	 sed -i 's/explicit/smart/g' ansible.cfg
     sed -i 's/#- role:/- role:/g' playbooks/main.yml
	 ansible-playbook playbooks/main.yml
</code> </pre></div>


### 2.- Bootloader iPXE

+  1.- **Clonar Lógica del Compilador**, basandose en el rol de Open Networking Foundation
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${HOME}
     git clone https://github.com/GitOpsTestsLabs/ipxe-build.git
</code> </pre></div>

+  2.- **Compilar iPXE**, modificando el script de Chain.
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${HOME}/ipxe-build
     sed -i 's/webserver/indra.lab/g' chain.ipxe
	 make COPY_FILES="chain.ipxe" OPTIONS="EMBED=chain.ipxe" image
</code> </pre></div>

+  3.- **Crear Tarball iPXE**.
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${HOME}/ipxe-build/out
	 mv out ipxe
	 chmod 755 ipxe
	 chmod 755 ipxe/*
	 tar cvzf ipxe_x64.tgz ipxe
</code> </pre></div>

+  4.- **Cargar Tarball en Role**.
<div class="prism-wrapper"><pre class="language-bash"><code>
     mkdir ${WORK_PATH}/roles/${ALUMNO}.bootloader/files
     \cp -f ${HOME}/ipxe-build/out/ipxe_x64.tgz ${WORK_PATH}/roles/${ALUMNO}.bootloader/files
</code> </pre></div>

+ 5.- **Comprobaciones**
<div class="prism-wrapper"><pre class="language-bash"><code>
     cd ${WORK_PATH}
     sed -i 's/#bootloader_ipxe_tarball/bootloader_ipxe_tarball/g' hosts/prod/group_vars/pxe_servers.yml
	 rm -Rf facts.d
	 sed -i 's/explicit/smart/g' ansible.cfg
	 ansible-playbook playbooks/main.yml
</code> </pre></div>

### 3.- Prueba del Bootloader
A la hora de depurar el proceso PXE, la parte que involucra más comunicaciones

####  3.1.- **Configurar Máquina Virtual para Arranque por Red**.


####  3.2.- **Análisis del Proceso PXE** 
![Proceso PXE](image://auto/pxe_msg.jpg?lightbox=600,400) 
Verificación de servicios.
<div class="prism-wrapper"><pre class="language-bash"><code>
ansible -u vagrant -i 192.168.56.150, all -m command -a "sudo systemctl enable firewalld --now" -vvv
ansible -u vagrant -i 192.168.56.150, all -m command -a "sudo systemctl restart dnsmasq" -vvv
ansible -u vagrant -i 192.168.56.150, all -m command -a "sudo cat /etc/hosts" -vvv
ansible -u va
</code> </pre></div>

####  3.3.- **Tabla Errores del Bootloader PXE**



# CI Incorporar Nuevos Roles en Colección Ansible
