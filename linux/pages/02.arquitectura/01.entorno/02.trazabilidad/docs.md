---
title: A2.Trazabilidad
taxonomy:
    category: docs
---

# ¿Qué son las trazas? Registros de Actividad
Las <u>trazas o logs</u> son registros de la actividad del entorno de ejecución que permiten establecer un  sistema de <u>trazabilidad</u> a partir del cual <u>administrar</u> sistemas informáticos.
+ **DEF.–Administración:** la capacidad de monitorizar y controlar el entorno de ejecución. 
+ **DEF.–Trazabilidad:** capacidad de seguir el rastro de los procesos del entorno de ejecución, desde su origen hasta su final, registrando cada etapa intermedia de su recorrido.
+ **DEF.–Trazas o Logs:** archivos de texto que registran cronológicamente la totalidad de actividades e incidencias importantes que ocurren en el sistema operativo o la red, pilares esenciales de cualquier sistema de trazabilidad.

# ¿Para qué existen las trazas? Administración de Sistemas

## Normativa
En el campo de la administración de sistemas también es posible encontrar estándares y normas, que en este caso recogen sus áreas funcionales o responsabilidades:
 + <strong><a href="https://1f8a81b9b0707b63-19211.webchannel-proxy.scarabresearch.com/rec/T-REC-X.700-199209-I/es">Recomendaciones ITU-T X.700]</a>:</strong> marco de gestión para la interconexión de sistema abiertos en aplicaciones del CCITT (Comité Consultivo Internacional Telegráfico y Telefónico).
 + <strong><a href='https://www.une.org/encuentra-tu-norma/busca-tu-norma/iso?c=024406' target='_blank'>ISO/IEC 10.040/1998</a>:</strong> Systems Management Overview.
 
## Áreas Funcionales de la Administración de Sistemas
Estas normativas distingues cinco áreas funcionales principales en la administración de sistemas:
+	**Gestión de la Configuración:** velar por el control de versiones del software y la compatibilidad.l
+	**Gestión de Fallos:** mecanismos de detección, aislamiento, diagnóstico y corrección de averías de la red y condiciones de error.
+	**Gestión de Prestaciones:** métricas de rendimiento a fin de mantenerlos a unos niveles aceptables.
+	**Gestión de Seguridad:** control de acceso a los recursos para evitar accesos a información sensible o sabotajes.
+	**Gestión de Contabilidad:** evaluar el uso de recursos y su coste, así como políticas de tarificación.
![Normativa Adminsitración Sistemas](image://teoria/teoria-administracion-sistemas.jpg)


## Trazabilidad: vital en todas las Áreas Funcionales
Todas las áreas funcionales de la administración de sistema dependen del sistema de trazas. Sin poderle seguirle la pista a lo que sucede en un en entorno de ejecución no es posible gestionar fallos, ni prestaciones, ni configuraciones, ni seguridad, ni contabilidad.

# La Trazabilidad en Linux
![Entorno Ejecución](image://teoria/teoria-entorno.jpg)

## Subsistemas de Trazas y Auditoría
Tal como muestra la imagen, la trazabilidad del entorno de ejecución se basa en dos subsistemas:
+ **El subsistema de trazas estándar:** es el registro de actividad del demonio de sistema. Traza los eventos relacionados con la administración de todos los programas del entorno de ejecución, como paradas, arranques, desconexiones de red, etc.
+ **El subsistema de auditoría:** es el registro de actividad del kernel. Mediante reglas personalizadas se puede hacer una introspección de la actividad de cada programa, trazando sus llamadas al sistema. Por cada regla de auditoría, el kernel genera un evento asociado a alguna llamada al sistema; cuando se dispara el evento, el núcleo se lo transmite a un demonio encargado de volcar dicho evento en una traza. 

## RFC 5424 – Protocolo Syslog: El Formato de las Trazas

Red Hat Enterprise Linux incluye un sistema de registro estándar que se basa en el protocolo Syslog (RFC 5424). Muchos programas usan este sistema para registrar eventos y organizarlos en archivos de registro.

![Protocolo Syslog](image://teoria/teoria-entorno-syslog.jpg)


### Campo: Priority = 8xFacility + Severity

[div class="table table-striped demonio"]
| VALOR | FACILITY          | VALOR |  SEVERITY 
|-------|-------------------|-------|-------
|0=0	| kernel messages   |0      | Emergency: system is unusable
|1=8	|user-level messages|1      | Alert: action must be taken immediately
|2=16	|mail system	    |2	    | Critical: critical conditions
|3=24	|system daemons	    |3	    | Error: error conditions
|4=32	|security/authorization messages          | 4 |Warning: warning conditions
|5=40	|messages generated internally by syslogd | 5 |	Notice: normal but significant condition
|6=48	|line printer subsystem	| 6	 | Informational: informational messages
|7=56	|network news subsystem	| 7  |Debug: debug-level messages
|8=64	|UUCP subsystem
|9=72	|clock daemon
|10=80	|security/authorization messages
|11=88	|FTP daemon
|12=96	|NTP subsystem
|13=104	|log Audit
|14=112	|log alert
|15=120	|clock daemon (note 2)
|16=128	|local use 0  (local0)
|17=136	|local use 1  (local1)
|18=144	|local use 2  (local2)
|19=152	|local use 3  (local3)
|20=160	|local use 4  (local4)
|21=168	|local use 5  (local5)
|22=176	|local use 6  (local6)
|23=184	|local use 7  (local7)
[/div]

##	Evolución del Subsistema de Trazas en Linux

[div class="table table-striped demonio"]
| DEMONIO | AÑO  | DESCRIPCIÓN
|---------|------|-----------
| syslog  | 1980 | Fue el primer sistema de log consolidado que nace a partir de la RFC 5424, estableciendo el formato de trazas que aún hoy perdura. Comienza en 1980 y se extiende durante bastantes años. Para la comunicación de eventos, sólo soporta protocolo UDP, y no garantiza el almacenamiento de todos los mensajes.
| syslog-ng | 1998 | Mejora las capacidades de Syslog agregando:<ul><li>Filtros basados en contenido</li><li>Logging directamente a base de datos</li><li>Ofrece protocolo de transporte TCP</li><li>Ofrece encriptación TLS</li></ul>
|rsyslog |2004 | Añade las siguientes funcionalidades básicas: <ul><li>Soporte de protocolo RELP (RFC 3195 - Reliable Event Logging Protocol)</li><li>Soporte de operaciones en Búfer</li></ul>
|journald| 2015 |Con el nuevo demonio de sistema Systemd se implementa el demonio journald como registro central de eventos del entorno de ejecución, con un mayor grado de detalle en las trazas. Algunas características relevantes son: <ul><li>No se almacena la información en texto plano, sino en archivos binarios aumentando la seguridad.</li><li>Mantiene un registro estructurado y ordenado de eventos del sistema.</li><li>Se requiere la herramienta journalctl para interpretar los logs</li></ul>
[/div]