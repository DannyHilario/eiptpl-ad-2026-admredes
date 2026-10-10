# DNS

> 50 min · [← DHCP](02-dhcp.md) · Siguiente: [WWW y HTTP](04-www.md)

## Repaso: la libreta de direcciones

En la Sesión 6 de Fundamentos viste a **DNS** (Domain Name System) como la libreta de
direcciones de la red: tu computadora solo entiende IPs, y DNS traduce un nombre como
`www.uanl.mx` a la IP del servidor. También viste en la sesión anterior que es DHCP quien
le dice a tu equipo **a qué servidor DNS preguntarle** (opción 6). Esta sesión explica
cómo está organizada esa libreta, que no vive en un solo lugar sino repartida en miles de
servidores por todo el mundo.

## Un árbol de nombres

El manual dice que los nombres se organizan "en formato de árbol, como el Explorador de
Windows" — y es una buena imagen. Los nombres de dominio forman una jerarquía que se lee
**de derecha a izquierda**, empezando por una raíz sin nombre, representada por un punto
(Mockapetris, 1987a):

```
                              . (raíz)
               ┌──────────────┼───────────────┐
              com             mx              org        <-- dominios de nivel
               │               │                             superior (TLD)
            google           uanl                        <-- dominio
               │          ┌────┴─────┐
              www        www       correo                <-- nombre de equipo
```

`www.uanl.mx.` es un **FQDN** (nombre de dominio completamente calificado): el camino
completo desde la hoja hasta la raíz. El punto final casi nunca se escribe, pero ahí está.

La raíz la atienden **13 identidades de servidores raíz**, nombradas de la A a la M y
operadas por 12 organizaciones independientes; entre todas suman más de mil copias
repartidas por el mundo (Root Server Operators, s.f.).

## Las piezas del sistema (robusteciendo al manual)

El manual nombra tres partes: cliente, servidores y zonas. Con los nombres que usan los
estándares y las herramientas reales:

| Pieza | Qué hace | Ejemplo |
|---|---|---|
| **Resolutor stub** | El "cliente DNS" dentro de tu equipo; solo pregunta y espera la respuesta final | El de Windows, o `systemd-resolved` en Ubuntu |
| **Resolutor recursivo** | Hace todo el trabajo de buscar la respuesta y la guarda en caché | El DNS que te dio DHCP (el router, el de tu proveedor) |
| **Servidor autoritativo** | Tiene la respuesta "oficial" de un dominio | El servidor DNS de la UANL para `uanl.mx` |
| **Zona** | La porción del árbol de la que un servidor es autoritativo | `uanl.mx` y lo que cuelga de él |

Un servidor es **autoritativo** para una zona cuando tiene la información completa de
ella, no una copia de paso (Mockapetris, 1987a).

## Resolución recursiva e iterativa

Existen dos modos de contestar una consulta (Mockapetris, 1987a):

- **Recursiva**: el servidor acepta la pregunta y la persigue por su cuenta hasta tener
  la respuesta final. Es lo que tu equipo le pide a su resolutor.
- **Iterativa**: el servidor no busca por ti; te contesta "yo no sé, pero pregúntale a
  este otro". Es como el resolutor recursivo recorre el árbol.

```
  Tu PC (stub)         Resolutor recursivo         Servidores autoritativos
       │                        │
       │ 1. ¿IP de www.uanl.mx? │
       │ ─────────────────────> │ 2. ¿www.uanl.mx? ───────> Raíz (.)
       │   (recursiva)          │ <── "pregunta a los de .mx"
       │                        │ 3. ¿www.uanl.mx? ───────> TLD (.mx)
       │                        │ <── "pregunta a los de uanl.mx"
       │                        │ 4. ¿www.uanl.mx? ───────> uanl.mx
       │                        │ <── "es 192.0.2.10"         (respuesta final)
       │ 5. "es 192.0.2.10"     │
       │ <───────────────────── │      (pasos 2-4: iterativos)
```

## Caché y TTL

Si cada consulta recorriera el árbol completo, la raíz recibiría todas las preguntas del
planeta. Para evitarlo, el resolutor guarda cada respuesta en **caché** durante el tiempo
que indica su **TTL** (*time to live*), un valor en segundos que define el dueño del
registro (Mockapetris, 1987b). Mientras el TTL no venza, el resolutor contesta de memoria,
sin volver a preguntar. Esto hace a DNS rápido, pero tiene una consecuencia: un cambio en
un registro **no se ve en todos lados al instante**.

## Tipos de registro

Cada dato guardado en una zona es un **registro de recurso** (RR). Los más comunes
(Mockapetris, 1987b; Thomson et al., 2003):

| Registro | Qué asocia | Ejemplo de uso |
|---|---|---|
| **A** | Nombre → dirección IPv4 | `www` → `192.0.2.10` |
| **AAAA** | Nombre → dirección IPv6 | `www` → `2001:db8::10` |
| **CNAME** | Nombre → otro nombre (alias) | `web` es alias de `www` |
| **MX** | Dominio → servidor de correo, con prioridad | Sesión 7 de esta etapa |
| **NS** | Zona → sus servidores autoritativos | Quién responde por `uanl.mx` |
| **SOA** | Inicio de la zona y sus datos de mantenimiento | Uno por zona |
| **PTR** | IP → nombre (resolución **inversa**) | `192.0.2.10` → `www` |
| **TXT** | Texto libre | Verificaciones, políticas de correo |

## Puerto 53: UDP y también TCP

DNS usa el puerto **53**, tanto en **UDP** como en **TCP** (Mockapetris, 1987b). Las
consultas normales viajan por UDP porque son pequeñas y rápidas; el estándar original
limitaba esos mensajes a 512 bytes. Cuando una respuesta no cabe, o cuando un servidor
copia una zona completa a otro (**transferencia de zona**), se usa TCP. Hoy el soporte de
TCP ya no es opcional: toda implementación de DNS de propósito general debe soportar
ambos (Dickinson et al., 2016). Un firewall que solo deja pasar UDP/53 rompe DNS de forma
intermitente, justo con las respuestas grandes.

## Ejemplo: "el sitio nuevo que solo algunos ven"

Una preparatoria migra su sitio web a un servidor nuevo. El viernes en la tarde, el
encargado cambia el registro **A** de `www.prepa-ejemplo.mx` para que apunte a la IP del
servidor nuevo, revisa desde su computadora que el sitio nuevo carga y apaga el servidor
viejo.

El lunes llegan los reportes: desde la red de la escuela, algunos alumnos ven el sitio
nuevo; otros, conectados a otros proveedores de Internet, reciben "no se puede acceder al
sitio" por varias horas más. El encargado vuelve a revisar: desde su computadora todo
funciona. El registro A está bien escrito.

Lo que nadie revisó es el **TTL** que tenía el registro antes del cambio. Era de 86400
segundos (un día completo). Cada resolutor recursivo que había consultado el nombre antes
del cambio sigue entregando la IP del servidor viejo — ya apagado — hasta que su copia en
caché venza. La respuesta "correcta" ya existe en el servidor autoritativo, pero no todos
los resolutores han vuelto a preguntar.

## Cómo se resuelve con herramientas reales

En tu **VM de Ubuntu Server** tienes herramientas para ver exactamente lo que ve cada
resolutor, y para montar tu propio servidor DNS:

- **`dig`**: consulta DNS desde la terminal y muestra la respuesta completa, incluido el
  TTL restante. Con `dig @<servidor> www.prepa-ejemplo.mx` se pregunta a un resolutor
  específico (y se compara la respuesta de varios); con `dig +trace` se recorre el árbol
  de forma iterativa desde la raíz, igual que el diagrama de esta sesión. En Ubuntu viene
  en el paquete `dnsutils` (Canonical, s.f.).
- **`resolvectl`**: Ubuntu usa `systemd-resolved` como resolutor local, que escucha en
  `127.0.0.53` (systemd, s.f.-b). `resolvectl status` muestra qué servidores DNS está
  usando tu VM (los que le dio DHCP) y `resolvectl query <nombre>` hace una consulta a
  través de él (systemd, s.f.-a).
- **BIND 9**: el servidor DNS de código abierto del Internet Systems Consortium; puede
  funcionar como resolutor con caché o como servidor autoritativo de una zona propia
  (Internet Systems Consortium, s.f.). En Ubuntu se instala con el paquete `bind9`, y las
  zonas propias se declaran en `/etc/bind/named.conf.local` (Canonical, s.f.).

```
$ dig www.prepa-ejemplo.mx A          (salida simplificada)
;; ANSWER SECTION:
www.prepa-ejemplo.mx.   71245   IN   A   203.0.113.20
                        ^^^^^
            segundos que este resolutor seguirá usando la IP vieja
```

La práctica habitual para una migración como la del ejemplo es **bajar el TTL** del
registro (por ejemplo, a 300 segundos) un día antes del cambio, y no apagar el servidor
viejo hasta que pase el TTL anterior — algo que `dig` permite comprobar antes de actuar.

---

## Referencias

Canonical Ltd. (s.f.). *Domain Name Service (DNS)*. Ubuntu Server
documentation. https://ubuntu.com/server/docs/how-to/networking/install-dns/

Dickinson, J., Dickinson, S., Bellis, R., Mankin, A., & Wessels, D. (2016). *DNS
transport over TCP — Implementation requirements* (RFC 7766). IETF.
https://www.rfc-editor.org/rfc/rfc7766

Internet Systems Consortium. (s.f.). *BIND 9*. https://www.isc.org/bind/

Mockapetris, P. (1987a). *Domain names — Concepts and facilities* (RFC 1034). IETF.
https://www.rfc-editor.org/rfc/rfc1034

Mockapetris, P. (1987b). *Domain names — Implementation and specification* (RFC 1035).
IETF. https://www.rfc-editor.org/rfc/rfc1035

Root Server Operators. (s.f.). *Root server technical operations*.
https://root-servers.org/

systemd. (s.f.-a). *resolvectl — Resolve domain names, IPv4 and IPv6 addresses, DNS
resource records, and services*. https://man7.org/linux/man-pages/man1/resolvectl.1.html

systemd. (s.f.-b). *systemd-resolved.service — Network name resolution manager*.
https://man7.org/linux/man-pages/man8/systemd-resolved.service.8.html

Thomson, S., Huitema, C., Ksinant, V., & Souissi, M. (2003). *DNS extensions to support
IP version 6* (RFC 3596). IETF. https://www.rfc-editor.org/rfc/rfc3596
