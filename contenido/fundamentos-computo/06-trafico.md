# ¿Qué es el tráfico de red?

> 50 min · [← Redes](05-redes.md) · Siguiente: [VM y Linux](07-vm-y-linux.md)

## Repaso rápido: lo que ya viste en Redes y Telecomunicaciones

DNS, DHCP, HTTP/HTTPS, TCP y UDP ya los definiste uno por uno el semestre pasado. Lo que
tu curso anterior no hizo es **juntarlos en un solo caso real**: cada protocolo se quedó
como una definición aislada. Esta sesión repasa las piezas rapidísimo y dedica el tiempo
real a la sección **"El ejemplo que vamos a resolver completo"**, donde ves a DNS y HTTP
trabajando en secuencia, uno después del otro, para una sola acción concreta.

## Definición

**Tráfico** es la información que efectivamente viaja por la red en un momento dado. No es
abstracto: son datos reales moviéndose de un dispositivo a otro, ahora mismo, mientras lees
esto tu propia conexión tiene tráfico.

## La información viaja en pedazos: paquetes

Cuando mandas algo por la red (un mensaje, una foto, una página web), esa información se
**corta en pedacitos** llamados **paquetes**, cada uno viaja por separado, y se vuelve a
armar al llegar.

**Analogía (mismo mundo de la Sesión 5):** en vez de mandar un libro completo en una caja,
lo mandas hoja por hoja, cada una en su propio sobre y numerada. El cartero (la red) puede
entregar los sobres en cualquier orden, por cualquier calle del fraccionamiento o cualquier
carretera disponible — quien recibe los reordena usando el número de cada hoja.

## Qué lleva un paquete

| Parte | Qué contiene | Analogía |
|---|---|---|
| **IP origen** | Quién lo mandó | El remitente en el sobre |
| **IP destino** | A dónde va | La dirección en el sobre |
| **Datos** | El contenido real | La hoja del libro |
| **Protocolo** | Las reglas para leer ese paquete | El idioma en que está escrita la carta |

Nota que el paquete usa **IP**, no MAC, para decir origen y destino — es la dirección que
sí tiene sentido fuera de tu red local, la que un router en el camino puede leer para
decidir la ruta (repaso de la Sesión 5).

## ¿Qué es un protocolo?

Un **protocolo** es un acuerdo de cómo deben "hablar" dos dispositivos para entenderse.
Ejemplos que vamos a ver en el curso: **HTTP/HTTPS** (páginas web), **FTP** (archivos),
**SMTP/IMAP/POP3** (correo), **DNS** (nombres), **DHCP** (direcciones). Dos dispositivos que
hablan HTTP se entienden perfecto entre sí, aunque ninguno sepa hablar FTP — son idiomas
distintos, no niveles distintos de un mismo idioma.

## Puertos: cómo un servidor atiende a muchos clientes a la vez

Ya sabes (por TCP/IP del semestre pasado) que un socket es IP + puerto. Falta la pregunta
práctica: si un servidor web solo tiene **una** dirección IP, ¿cómo distingue tu petición
de la de otros miles de personas viendo la misma página al mismo tiempo?

La respuesta son los **puertos**: un número que identifica *qué servicio* dentro de esa IP
debe atender el paquete, y *qué conversación específica* le corresponde a cada cliente.

| Puerto | Servicio |
|---|---|
| 80 | HTTP (web sin cifrar) |
| 443 | HTTPS (web cifrada) |
| 21 | FTP |
| 25 | SMTP (correo saliente) |
| 53 | DNS |

*(Puertos asignados oficialmente por IANA, s.f. — ver Referencias al final.)*

El servidor siempre escucha en un puerto fijo y conocido (443 para HTTPS). Tu navegador,
en cambio, abre un puerto **aleatorio y temporal** de tu lado (por ejemplo, 51342) solo
para esa conversación — así el servidor sabe exactamente a quién responderle, aunque haya
miles de clientes usando el mismo puerto 443 al mismo tiempo.

**Analogía:** la IP es la dirección del edificio; el puerto es el número de
departamento/oficina dentro de ese edificio. El código postal (IP) te lleva al edificio
correcto, pero necesitas el número de puerta (puerto) para llegar al servicio correcto.

## El ejemplo que vamos a resolver completo: entrar a una página web

```
  Tú (navegador)        Servidor DNS           Servidor Web
        │                     │                      │
        │ ¿IP de www.ejemplo.com? ──>                │
        │                     │                      │
        │ <── 93.184.216.34 ──                       │
        │                     │                      │
        │ ── petición HTTP/HTTPS a 93.184.216.34 ──────>
        │                     │                      │
        │ <── contenido de la página, en paquetes ─────
        │
        (el navegador reordena los paquetes y pinta la página)
```

1. Escribes `www.ejemplo.com` — eres el **cliente**.
2. Tu computadora no entiende nombres, solo IPs, así que pregunta a un servidor **DNS**
   (a fondo en Etapa 2).
3. El DNS responde con una IP.
4. Tu navegador manda una petición **HTTP/HTTPS** a esa IP.
5. El **servidor web** responde con el contenido, cortado en **paquetes**.
6. Tu computadora reordena los paquetes y el navegador muestra la página.

Todo esto pasa en milisegundos, y es exactamente lo que vamos a ver **en cámara lenta**
más adelante con `tcpdump` — por eso importa entender el concepto antes que la herramienta.

## Conectando con la Sesión 5: ¿dónde entran el switch y el router aquí?

- Del paso 1 al 3 (pedirle al DNS), el paquete probablemente **nunca sale de tu red
  local** si tu router tiene su propio DNS, o cruza tu router una sola vez si el DNS está
  en Internet.
- Del paso 4 al 6 (hablar con el servidor web), el paquete casi siempre **sí sale** de tu
  red local: pasa por tu router (con NAT, como vimos en la Sesión 5), luego por varios
  routers más de Internet, hasta llegar a la red local donde vive el servidor.
- Dentro de cada red local que el paquete atraviesa, quien lo mueve de un cable a otro es
  un **switch**. Quien decide "por aquí sigue" de una red a la siguiente es un **router**.

---

## Ejercicio práctico: mira el viaje real de un paquete

Usa la terminal/cmd de tu laptop real (no necesitas la VM todavía):

**Paso 1 — Resuelve un nombre a IP tú mismo, como hace el paso 2-3 del ejemplo:**
```
nslookup www.uanl.mx
```
Vas a ver la IP que el DNS regresó — exactamente el paso "DNS responde con una IP" del
diagrama de arriba, pero viéndolo tú en lugar del navegador.

**Paso 2 — Ve cada router que cruza tu paquete en el camino:**

En Windows:
```
tracert www.uanl.mx
```
En Mac:
```
traceroute www.uanl.mx
```
Cada línea de la salida es **un router distinto** que tu paquete atravesó — la cantidad de
"saltos" (hops) te dice cuántas redes distintas cruzó antes de llegar al servidor.

> ### ✅ Autocheck del ejercicio
> **Pregunta:** si tu `tracert` muestra 8 saltos antes de llegar al servidor, y el primer
> salto es tu propio router de casa, ¿ese primer salto usó una IP privada o pública?
> **Respuesta:**
>
> Privada (algo como `192.168.1.1`) — es la puerta de enlace de tu propia red local, la
> misma que viste con `ipconfig`/`ifconfig` en la Sesión 5. A partir del segundo salto ya
> empiezas a ver IPs públicas de la red de tu proveedor de Internet.

---

## Autocheck — Sesión 6

> **Pregunta 1:** si el DNS de la analogía es "la libreta de direcciones", ¿qué pasaría si
> el cartero (la red) entregara las hojas de un libro en desorden y el destinatario no
> tuviera el número de cada hoja?
> **Respuesta:**
>
> No podría reconstruir el libro en el orden correcto — recibiría el contenido completo,
> pero desordenado e inútil. Por eso cada paquete lleva un número de secuencia además del
> contenido: la red no garantiza el orden de entrega, así que el orden se reconstruye del
> lado de quien recibe.

> **Pregunta 2:** un paquete que viaja de tu laptop al servidor web de `www.ejemplo.com`
> normalmente pasa por varios routers en el camino. ¿Ese paquete cambia de dirección MAC,
> de dirección IP, o de ambas, en cada salto?
> **Respuesta:**
>
> La dirección **MAC cambia en cada salto** (cada router la reescribe para el siguiente
> tramo local), pero la dirección **IP de origen y destino se mantiene** de principio a
> fin — es la que le da continuidad al viaje completo, sin importar cuántas redes locales
> distintas cruce.

> **Pregunta 3:** tu laptop abre dos pestañas al mismo tiempo hacia el mismo servidor
> (`www.ejemplo.com:443`). El servidor recibe ambas conexiones en el mismo puerto 443.
> ¿Cómo sabe distinguir cuál paquete pertenece a cuál pestaña?
> **Respuesta:**
>
> Por el **puerto de origen** de tu laptop, que es distinto y aleatorio para cada pestaña
> (por ejemplo, 51342 y 51390). El servidor identifica cada conversación por la
> combinación completa IP+puerto de origen, no solo por el puerto de destino (443), que sí
> es el mismo para ambas.

**Siguiente paso:** [VM y Linux](07-vm-y-linux.md).

---

## Referencias

Kurose, J. F., & Ross, K. W. (2017). *Computer networking: A top-down approach* (7a ed.).
Pearson.

Internet Assigned Numbers Authority. (s.f.). *Service name and transport protocol port
number registry*. IANA.
https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml
