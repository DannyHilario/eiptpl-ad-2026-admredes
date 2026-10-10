# Servidores e infraestructura de TI

> 50 min · [← Administración de seguridad](../etapa-1-funciones-administracion/05-administracion-seguridad.md) · Siguiente: [DHCP](02-dhcp.md)

## De qué trata la Etapa 2

La Etapa 1 fue el "cómo se administra" una red (FCAPS). La Etapa 2 es el "qué se
administra": los **servicios de red** que corren en servidores y que los usuarios usan
todos los días sin saberlo — DHCP, DNS, web, HTTPS, FTP y correo. Antes de verlos uno por
uno, esta sesión pone la base común a todos: qué es un servidor dentro de una
organización y qué infraestructura lo rodea.

## Repaso: un servidor es un rol, no una pieza

En [Cliente-Servidor](../00-fundamentos-computo/04-cliente-servidor.md) quedó establecido
que un **servidor** es una computadora con las mismas piezas de siempre (CPU, RAM,
almacenamiento), pero **dedicada** a responder peticiones de otras computadoras, casi
siempre encendida 24/7. Lo que la convierte en servidor es que corre uno o varios
**servicios** (procesos sin ventana) que se quedan "escuchando" peticiones.

Lo que se agrega ahora: cada servicio escucha en un **puerto** (repaso de la Sesión 6). Un
mismo servidor físico puede ofrecer varios servicios a la vez, cada uno en su puerto:

```
                    Servidor (una sola IP: 192.168.1.5)
               ┌─────────────────────────────────────────┐
  Petición ──> │ puerto 53  ──> servicio DNS              │
  Petición ──> │ puerto 67  ──> servicio DHCP             │
  Petición ──> │ puerto 443 ──> servicio web (HTTPS)      │
               └─────────────────────────────────────────┘
```

En la analogía del fraccionamiento: el servidor es una casa con varias ventanillas, y el
puerto es el número de ventanilla. Quien llega a la casa correcta (IP) todavía tiene que
saber a qué ventanilla (puerto) acercarse.

## Infraestructura de TI: lo que rodea al servidor

Un servidor aislado no sirve de nada. La **infraestructura de TI** es el conjunto de
elementos interdependientes que permiten que los servicios lleguen a los usuarios. El
manual oficial la divide en estos componentes:

| Componente | Qué incluye | Ejemplo concreto |
|---|---|---|
| **Hardware** | Equipos de cómputo y de red | Servidores, PCs, switches, routers |
| **Software** | Sistemas operativos y aplicaciones de servidor | Ubuntu Server, un servidor web, un ERP |
| **Red** | Lo que conecta todo lo anterior | Switches (LAN), routers (entre redes) |
| **Instalaciones** | El espacio físico y el cableado | Cableado del edificio, cuarto de comunicaciones |
| **Servidor** | El equipo principal que ofrece los servicios | El equipo donde corre DHCP o DNS |
| **Sala de servidores / centro de datos** | Donde se concentran los servidores | Un cuarto con racks, aire acondicionado, UPS |

Dos precisiones sobre el texto del manual. Primero, menciona **hubs** junto a switches y
routers: en redes actuales el hub prácticamente ya no se instala, su función la cubre el
switch (Sesión 5). Segundo, el software de la lista (CMS, CRM, ERP) son **aplicaciones**
que corren *encima* de los servicios de red de esta etapa: un ERP no funciona si DNS no
resuelve el nombre de su servidor o si DHCP no le dio IP a la PC del usuario.

## Formas reales de un servidor

| Forma | Cómo se ve | Cuándo se usa |
|---|---|---|
| **Torre** | Como una PC grande | Oficinas pequeñas con 1-2 servidores |
| **Rack** | Delgado y horizontal, se atornilla en un gabinete | Salas de servidores y centros de datos |
| **Virtual (VM)** | No tiene forma física propia: es software | Muchos servidores en un solo equipo físico |
| **En la nube** | Una VM rentada en el centro de datos de un proveedor | Cuando no conviene tener hardware propio |

La forma virtual es la que vas a usar tú: tu VM de Ubuntu en VirtualBox (Sesión 7 de
Fundamentos) es, para todo efecto, un servidor. En el semestre vas a instalarle servicios
reales de esta etapa.

## La sala de servidores y el centro de datos

Concentrar los servidores en un solo lugar permite protegerlos mejor: energía de respaldo
(**UPS** y planta eléctrica), enfriamiento, control de acceso físico y cableado
ordenado. Qué tan resistente es un centro de datos se clasifica comúnmente con los
**Tiers** del Uptime Institute (Uptime Institute, s.f.):

| Tier | Idea central |
|---|---|
| **I** — Capacidad básica | Infraestructura mínima; hay que apagar para dar mantenimiento |
| **II** — Componentes redundantes | Energía y enfriamiento con piezas de repuesto en línea |
| **III** — Mantenimiento concurrente | Se puede reparar cualquier componente sin apagar los servicios |
| **IV** — Tolerancia a fallas | Una falla de un equipo o ruta no afecta la operación |

Conecta con la Etapa 1: la sala de servidores es donde la **F** (fallas) y la **S**
(seguridad física) de FCAPS se vuelven concretas.

## Un servidor, pocos servicios

Una buena práctica de seguridad es que cada servidor corra **solo los servicios que
necesita** y deshabilite el resto: cada servicio que escucha en un puerto es una puerta
más que alguien puede intentar abrir. La guía de seguridad para servidores del NIST lo
plantea como uno de los primeros pasos al configurar un servidor (Scarfone et al., 2008).
Por eso, en organizaciones medianas y grandes es común separar servicios: un servidor
para DNS, otro para correo, otro para web.

## Mapa de la Etapa 2: servicios y puertos

Todo lo que sigue en esta etapa es un servicio distinto, cada uno con su puerto
registrado oficialmente ante la IANA (IANA, s.f.):

| Sesión | Servicio | Puerto(s) | Transporte |
|---|---|---|---|
| 2 | DHCP (servidor / cliente) | 67 / 68 | UDP |
| 3 | DNS | 53 | UDP y TCP |
| 4 | HTTP (web) | 80 | TCP |
| 5 | HTTPS (web cifrada) | 443 | TCP |
| 6 | FTP (control / datos) | 21 / 20 | TCP |
| 7 | SMTP / POP3 / IMAP (correo) | 25 / 110 / 143 | TCP |

Los detalles (variantes cifradas, puertos alternos) se ven en la sesión de cada servicio.

## Ejemplo: "el servidor de la oficina"

Una agencia de 25 empleados tiene un solo equipo al que todos llaman "el servidor": una
torre debajo del escritorio del encargado de sistemas. Nadie recuerda bien qué tiene
instalado. Se sabe que de ahí salen las carpetas compartidas y que "algo de la red" depende
de él, porque cuando se apaga nadie puede entrar al sistema de facturación.

Un viernes se va la luz diez minutos. El equipo no tiene UPS, así que se apaga de golpe.
Al regresar la energía, la mitad de las PCs no tienen Internet, el sistema de facturación
no abre y las carpetas compartidas sí funcionan. El encargado reinicia el servidor dos
veces sin resultado. Además, un proveedor externo que revisó la red semanas antes había
dejado una nota: "el servidor tiene abiertos puertos que no parecen necesarios".

Nadie puede responder dos preguntas básicas: **¿qué servicios corre ese equipo?** y
**¿cuáles de ellos están escuchando en la red ahora mismo?** Sin esa información, no hay
forma de saber cuál servicio no arrancó tras el apagón, ni cuáles sobran.

## Cómo se resuelve con herramientas reales

En un servidor Linux como tu **VM de Ubuntu Server en VirtualBox**, esas dos preguntas
tienen respuesta directa con dos comandos:

- **`systemctl`**: administra los servicios del sistema (systemd). Con
  `systemctl list-units --type=service` se ven todos los servicios y su estado; con
  `systemctl status <servicio>` se ve si uno arrancó o falló y por qué; con
  `systemctl is-enabled <servicio>` se sabe si arranca solo al encender el equipo — lo que
  habría importado tras el apagón (systemd, s.f.).
- **`ss -tulpn`**: lista los puertos abiertos. Las opciones significan `-t` TCP, `-u` UDP,
  `-l` solo los que están escuchando, `-p` qué proceso los abrió y `-n` mostrar números
  en vez de nombres (Prokop, s.f.).

```
$ sudo ss -tulpn          (salida simplificada)
Netid  State   Local Address:Port   Process
udp    UNCONN  0.0.0.0:67           kea-dhcp4      <-- DHCP
udp    UNCONN  192.168.1.5:53       named          <-- DNS
tcp    LISTEN  0.0.0.0:22           sshd           <-- acceso remoto
tcp    LISTEN  0.0.0.0:23           telnetd        <-- ¿quién instaló esto?
```

Con esa lista, "puertos que no parecen necesarios" deja de ser una sospecha: cada puerto
tiene nombre de proceso, y lo que sobra se detiene y deshabilita con `systemctl`. En las
siguientes sesiones vas a ver qué es cada uno de esos servicios.

---

## Referencias

Internet Assigned Numbers Authority. (s.f.). *Service name and transport protocol port
number registry*. https://www.iana.org/assignments/service-names-port-numbers/

Prokop, M. (s.f.). *ss(8) — Linux manual page*. man7.org.
https://man7.org/linux/man-pages/man8/ss.8.html

Scarfone, K., Jansen, W., & Tracy, M. (2008). *Guide to general server security* (NIST
Special Publication 800-123). National Institute of Standards and Technology.
https://doi.org/10.6028/NIST.SP.800-123

systemd. (s.f.). *systemctl — Control the systemd system and service manager*.
https://www.freedesktop.org/software/systemd/man/latest/systemctl.html

Uptime Institute. (s.f.). *Tier classification system*. https://uptimeinstitute.com/tiers
