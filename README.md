# Administración de Redes

Material de apoyo del curso **Administración de Redes** — Escuela Industrial y
Preparatoria Técnica "Pablo Livas" (EIPTPL), UANL, 2026.

Aquí vas a encontrar las notas, explicaciones y ejercicios prácticos que acompañan las
clases, organizados por módulo y sesión. La idea es que puedas repasar cualquier tema
después de clase, o adelantarte si quieres.

## Cómo está organizado

El contenido vive en la carpeta [`contenido/`](contenido/), dividido en módulos. Cada
módulo tiene sesiones numeradas (`01-`, `02-`, ...) que se estudian en ese orden.

### 📗 [00 — Fundamentos de Cómputo](contenido/00-fundamentos-computo/)

Antes de hablar de redes, necesitamos el mismo vocabulario básico sobre qué es una
computadora, y de ahí pasamos a la introducción real de redes. Este módulo cubre:

1. [Hardware: las piezas físicas de una computadora](contenido/00-fundamentos-computo/01-hardware.md)
2. [Software y el Sistema Operativo](contenido/00-fundamentos-computo/02-software-so.md)
3. [Interfaces de usuario: gráfica y línea de comandos](contenido/00-fundamentos-computo/03-interfaces-usuario.md)
4. [Cliente-Servidor y qué es un Servidor](contenido/00-fundamentos-computo/04-cliente-servidor.md)
5. [Redes: switch vs. router, PAN/MAN, P2P](contenido/00-fundamentos-computo/05-redes.md)
6. [Tráfico, paquetes, protocolos y puertos](contenido/00-fundamentos-computo/06-trafico.md)
7. [Máquinas virtuales y modos de red en VirtualBox](contenido/00-fundamentos-computo/07-vm-y-linux.md)
8. [Terminal Linux/WSL2, pipes y gestión de paquetes](contenido/00-fundamentos-computo/08-terminal.md)

### 📘 [Etapa 1 — Funciones de Administración de Redes (FCAPS)](contenido/etapa-1-funciones-administracion/)

Primer módulo que desarrolla directamente el manual oficial del curso. Cada sesión cubre
una de las cinco funciones del modelo **FCAPS** (Fault, Configuration, Accounting,
Performance, Security):

1. [Administración de configuración](contenido/etapa-1-funciones-administracion/01-administracion-configuracion.md)
2. [Administración de rendimiento](contenido/etapa-1-funciones-administracion/02-administracion-rendimiento.md)
3. [Administración de fallas](contenido/etapa-1-funciones-administracion/03-administracion-fallas.md)
4. [Administración de contabilidad](contenido/etapa-1-funciones-administracion/04-administracion-contabilidad.md)
5. [Administración de seguridad](contenido/etapa-1-funciones-administracion/05-administracion-seguridad.md)

### 📙 [Etapa 2 — Servicios de Red y Servidores](contenido/etapa-2-servicios-red/)

Los servicios que hacen funcionar una red, cada uno con su protocolo y puerto reales:

1. [Servidores e infraestructura de TI](contenido/etapa-2-servicios-red/01-servidores.md)
2. [DHCP](contenido/etapa-2-servicios-red/02-dhcp.md)
3. [DNS](contenido/etapa-2-servicios-red/03-dns.md)
4. [WWW y HTTP](contenido/etapa-2-servicios-red/04-www.md)
5. [HTTPS](contenido/etapa-2-servicios-red/05-https.md)
6. [FTP](contenido/etapa-2-servicios-red/06-ftp.md)
7. [Correo electrónico: SMTP, POP3 e IMAP](contenido/etapa-2-servicios-red/07-correo.md)

### 📕 [Etapa 3 — Instalación del Sistema Operativo de Red](contenido/etapa-3-sistema-operativo-red/)

Guía paso a paso, con capturas de pantalla, para instalar el servidor que usaremos el
resto del semestre:

- [Instalación de Ubuntu Server 26.04 LTS](contenido/etapa-3-sistema-operativo-red/01-instalacion-ubuntu-server/README.md)
  — 13 pasos, desde elegir el ISO correcto hasta el reinicio
- Primeros pasos en el servidor:
  1. [Primer inicio de sesión](contenido/etapa-3-sistema-operativo-red/02-primeros-pasos/01-primer-inicio.md)
  2. [Usuarios: una computadora, muchas personas](contenido/etapa-3-sistema-operativo-red/02-primeros-pasos/02-usuarios.md)
  3. [El administrador, sudo y los paquetes](contenido/etapa-3-sistema-operativo-red/02-primeros-pasos/03-administrador-y-paquetes.md)
  4. [Práctica: usuarios y sudo](contenido/etapa-3-sistema-operativo-red/02-primeros-pasos/04-practica-usuarios.md)
  5. [Archivos, carpetas y permisos](contenido/etapa-3-sistema-operativo-red/02-primeros-pasos/05-archivos-y-permisos.md)
  6. [Práctica: archivos y permisos](contenido/etapa-3-sistema-operativo-red/02-primeros-pasos/06-practica-archivos.md)
- Acceso remoto:
  1. [Instalar Git Bash: una terminal en tu computadora](contenido/etapa-3-sistema-operativo-red/03-acceso-remoto/01-git-bash.md)
  2. [NAT: muchos equipos, una sola dirección pública](contenido/etapa-3-sistema-operativo-red/03-acceso-remoto/02-que-es-nat.md)
  3. [Conectarte a tu servidor por SSH](contenido/etapa-3-sistema-operativo-red/03-acceso-remoto/03-conectarte-por-ssh.md)
  4. [Tu primer servicio: un servidor web con nginx](contenido/etapa-3-sistema-operativo-red/03-acceso-remoto/04-servidor-web.md)

La Etapa 4 del manual oficial (direccionamiento IP y subneteo) se agregará conforme
avance el semestre.

## Cómo usar este material

- Lee cada sesión en orden — cada una da por sabida la anterior.
- Los ejercicios prácticos están pensados para hacerse en el momento, en tu propia
  computadora o en el equipo del laboratorio.
- Si algo no te queda claro, pregúntalo en clase — este material es un apoyo, no
  sustituye la explicación de tu profesor.
