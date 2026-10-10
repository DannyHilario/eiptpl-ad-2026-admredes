# DHCP

> 50 min · [← Servidores e infraestructura de TI](01-servidores.md) · Siguiente: [DNS](03-dns.md)

## Repaso: lo que ya sabes

El semestre pasado viste que **DHCP** (Dynamic Host Configuration Protocol) es lo que hace
que tu laptop "agarre IP sola" al conectarse a una red, sin que tengas que escribir nada.
Esta sesión abre esa caja: qué mensajes viajan exactamente, por qué puertos, qué pasa
cuando la IP "caduca" y qué se rompe cuando DHCP falla.

## Qué entrega DHCP (no solo la IP)

Para que un equipo funcione en red necesita al menos cuatro datos. Sin DHCP, habría que
escribirlos a mano en cada PC, celular e impresora — y cualquier error de dedo deja a ese
equipo sin red. DHCP los entrega todos juntos como **opciones** (Alexander & Droms, 1997):

| Dato | Para qué sirve | Opción DHCP |
|---|---|---|
| **Dirección IP** | El "número de casa" del equipo | (campo principal del mensaje) |
| **Máscara de subred** | Qué parte de la IP es la red y qué parte el equipo | 1 |
| **Router (puerta de enlace)** | Por dónde salir hacia otras redes | 3 |
| **Servidor DNS** | A quién preguntarle nombres (siguiente sesión) | 6 |
| **Tiempo de concesión** | Por cuánto tiempo puede usar esa IP | 51 |

En la analogía del fraccionamiento: DHCP es la administración que, cuando llega una
familia nueva, le asigna número de casa, le dice cuál es la caseta de salida (router) y a
quién preguntarle direcciones (DNS).

## DORA: los cuatro mensajes

DHCP es cliente-servidor y usa **UDP**: el servidor escucha en el puerto **67** y el
cliente en el **68** (Droms, 1997). La asignación típica son cuatro mensajes, que se
recuerdan por sus iniciales, **DORA**:

```
   Cliente (aún sin IP)                         Servidor DHCP
          │                                           │
          │ 1. DHCPDISCOVER  (broadcast)              │
          │   "¿Hay algún servidor DHCP aquí?"  ────> │
          │                                           │
          │ 2. DHCPOFFER                              │
          │ <──── "Te ofrezco 192.168.1.50 por 24 h"  │
          │                                           │
          │ 3. DHCPREQUEST   (broadcast)              │
          │   "Acepto la oferta de ese servidor" ───> │
          │                                           │
          │ 4. DHCPACK                                │
          │ <──── "Confirmado, es tuya"               │
          │                                           │
   Ahora sí configura su IP, máscara, router y DNS
```

Detalles que explican el diseño:

- **¿Por qué broadcast?** El cliente todavía no tiene IP ni sabe la IP del servidor, así
  que envía desde `0.0.0.0` hacia `255.255.255.255` (todos en la red local). Es el
  "grito" al fraccionamiento de la Sesión 5 de Fundamentos.
- **¿Por qué el REQUEST también es broadcast?** Puede haber varios servidores DHCP que
  hicieron ofertas; al anunciar a todos cuál aceptó, los demás saben que su oferta no se
  usó (Droms, 1997).
- Si el servidor ya no puede darle esa IP, responde **DHCPNAK** en lugar de DHCPACK y el
  cliente empieza de nuevo. Si el cliente detecta que la IP ya la usa otro equipo, avisa
  con **DHCPDECLINE**.

## La concesión (lease): las IP se prestan, no se regalan

El servidor no regala la IP: la **presta** por un tiempo (la **concesión** o *lease*) y
lleva una tabla de qué IP le prestó a qué dirección MAC y hasta cuándo. Esto permite
reutilizar direcciones de equipos que ya se fueron. Para no quedarse sin IP, el cliente
renueva antes de que venza, con dos temporizadores definidos en el estándar (Droms, 1997):

```
 0%                      50% (T1)                 87.5% (T2)        100%
 ├────────────────────────┼─────────────────────────┼────────────────┤
 IP asignada        Renueva con su           Si no respondió,     Vence: deja
                    mismo servidor           pregunta a cualquier  de usar la IP
                    (DHCPREQUEST)            servidor (broadcast)
```

Cuando un equipo se apaga de forma ordenada puede enviar **DHCPRELEASE** para devolver la
IP antes de tiempo — aunque el estándar aclara que DHCP no depende de que ese mensaje
llegue (Droms, 1997).

## Los tres mecanismos de asignación (corrigiendo al manual)

El manual describe tres formas de asignar, pero las definiciones de "automática" y
"dinámica" no coinciden con el estándar. Según RFC 2131 (Droms, 1997):

| Mecanismo | Qué hace en realidad |
|---|---|
| **Automática** | Asigna una IP **permanente** al cliente (sin vencimiento) |
| **Dinámica** | Asigna una IP **por tiempo limitado** o hasta que el cliente la libere — la más común |
| **Manual** | El administrador decide qué IP le toca a cada equipo; DHCP solo se la entrega |

La asignación manual es lo que en la práctica se llama **reservación**: se amarra una MAC
a una IP fija. Se usa para impresoras y servidores, que conviene que siempre estén en la
misma dirección, sin tener que configurarlos a mano.

## Cuando el servidor está en otra red: el agente relay

El DHCPDISCOVER es broadcast, y en la Sesión 5 viste que **el broadcast no cruza la
caseta** (el router). Entonces, si una empresa tiene un solo servidor DHCP y diez redes,
¿cómo le llega el grito de un cliente en otra red? Con un **agente relay**: una función
(normalmente en el router) que recibe el broadcast y lo reenvía como mensaje dirigido al
servidor DHCP, anotando de qué red vino para que el servidor elija una IP de esa red
(Wimer, 1993).

```
  Red A (clientes)          Router con relay             Red B
  ┌───────────┐  broadcast  ┌────────────┐  unicast  ┌──────────────┐
  │ Cliente   │ ──────────> │ Relay DHCP │ ────────> │ Servidor DHCP│
  └───────────┘             └────────────┘           └──────────────┘
```

## DHCP en tu propia VM

Ya usas DHCP sin saberlo: en modo NAT, tu VM de VirtualBox recibe su IP de un servidor
DHCP **integrado en el propio VirtualBox**, normalmente en la red `10.0.2.0` (Oracle,
s.f.). Por eso tu VM tiene una IP distinta a la de tu laptop.

## Ejemplo: "el router que alguien trajo de su casa"

En el laboratorio de una escuela, las PCs llevan años funcionando sin problema con el
servidor DHCP del área de sistemas, que entrega direcciones `10.20.0.x` con el DNS y la
puerta de enlace de la escuela.

Un lunes, un profesor conecta un router inalámbrico que trajo de su casa para "tener mejor
WiFi en el salón", enchufándolo a la red del laboratorio. Ese router trae su propio
servidor DHCP encendido de fábrica. Desde ese momento, algunas PCs que se reinician
reciben IPs `192.168.0.x` en lugar de `10.20.0.x`: pueden navegar a medias, pero no
llegan al sistema de calificaciones ni a la impresora. Otras PCs, que se reiniciaron más
tarde, funcionan perfectamente.

Lo que lo hace difícil de diagnosticar es que el problema parece aleatorio: depende de
qué servidor contestó primero el DHCPDISCOVER de cada equipo. Ninguna PC "sabe" que hay
dos servidores DHCP — cada una aceptó la primera oferta que le llegó, exactamente como
dice el protocolo. A este segundo servidor no autorizado se le llama **DHCP intruso**
(*rogue DHCP*).

## Cómo se resuelve con herramientas reales

Para diagnosticar y para montar un servidor DHCP propio en tu **VM de Ubuntu Server**:

- **`tcpdump`**: captura los mensajes DHCP que viajan por la red. Filtrando por los
  puertos 67 y 68 (`sudo tcpdump -n -i <interfaz> port 67 or port 68`) se ve quién
  responde a los DHCPDISCOVER: si aparecen **dos** IPs distintas enviando DHCPOFFER, ahí
  está el intruso (The Tcpdump Group, s.f.).
- **ISC Kea**: el servidor DHCP actual del Internet Systems Consortium (ISC), con
  servicios separados para IPv4 (`kea-dhcp4`) e IPv6 (`kea-dhcp6`) y configuración en
  formato JSON (Internet Systems Consortium, s.f.-b). En Ubuntu se instala con
  `sudo apt install kea` y se configura en `/etc/kea/kea-dhcp4.conf` (Canonical, s.f.).
  Su antecesor, **ISC DHCP** (`dhcpd`), llegó a fin de mantenimiento a finales de 2022 y
  el propio ISC recomienda migrar a Kea (Internet Systems Consortium, s.f.-a) — vas a
  encontrar muchos tutoriales viejos en Internet que todavía usan `dhcpd`.

```
Fragmento de /etc/kea/kea-dhcp4.conf (simplificado)
"subnet4": [ {
    "subnet": "10.20.0.0/24",
    "pools": [ { "pool": "10.20.0.100 - 10.20.0.200" } ],
    "option-data": [
        { "name": "routers", "data": "10.20.0.1" },
        { "name": "domain-name-servers", "data": "10.20.0.2" } ]
} ]
```

Ese fragmento es, línea por línea, la tabla de "Qué entrega DHCP": rango de IPs, router y
DNS. En la escuela del ejemplo, además, los switches administrables suelen tener una
función para permitir respuestas DHCP solo desde el puerto donde está el servidor
autorizado — se verá cuando el curso llegue a seguridad de la red local.

---

## Referencias

Alexander, S., & Droms, R. (1997). *DHCP options and BOOTP vendor extensions* (RFC 2132).
IETF. https://www.rfc-editor.org/rfc/rfc2132

Canonical Ltd. (s.f.). *How to install and configure isc-kea*. Ubuntu Server
documentation. https://documentation.ubuntu.com/server/how-to/networking/install-isc-kea/

Droms, R. (1997). *Dynamic host configuration protocol* (RFC 2131). IETF.
https://www.rfc-editor.org/rfc/rfc2131

Internet Systems Consortium. (s.f.-a). *ISC DHCP*. https://www.isc.org/dhcp/

Internet Systems Consortium. (s.f.-b). *Kea administrator reference manual*.
https://kea.readthedocs.io/en/latest/

Oracle. (s.f.). *Virtual networking: Network Address Translation (NAT)*. Oracle
VirtualBox user guide. https://www.virtualbox.org/manual/topics/networkingdetails.html

The Tcpdump Group. (s.f.). *tcpdump(1) — Man page*.
https://www.tcpdump.org/manpages/tcpdump.1.html

Wimer, W. (1993). *Clarifications and extensions for the bootstrap protocol* (RFC 1542).
IETF. https://www.rfc-editor.org/rfc/rfc1542
