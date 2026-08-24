# ¿Qué es una red?

> 50 min · [← Cliente-Servidor](04-cliente-servidor.md) · Siguiente: [Tráfico](06-trafico.md)

## Por qué importa esta sesión

Todo el curso (DNS, DHCP, web, FTP, correo) es una variación del mismo escenario:
**alguien pide algo, algo responde**, viajando por switches y routers. Y "administrar una
red" — el nombre de esta materia — no es más que mantener ese viaje **disponible, rápido,
ordenado, medible y seguro**: las cinco funciones que vas a ver a detalle en la Etapa 1 del
manual (configuración, rendimiento, fallas, contabilidad, seguridad). Ninguna de esas cinco
tiene sentido si primero no sabes describir qué es lo que estás administrando — por eso
empezamos aquí.

## Repaso rápido: lo que ya viste en Redes y Telecomunicaciones

En tu curso anterior ya definiste red, switch, router, protocolo, dirección IP y las
clasificaciones LAN/MAN/WAN. Las próximas dos secciones son un repaso deliberadamente
corto — solo lo suficiente para tenerlo fresco. La parte nueva de esta sesión es
**"A fondo: switch vs. router"**, más abajo: tu manual anterior los define, pero no
explica la diferencia técnica real (MAC vs. IP, broadcast, NAT) — eso es lo que vamos a
profundizar hoy.

## Definición

Una **red** es un conjunto de dispositivos conectados entre sí que pueden **intercambiar
información**. Dos computadoras conectadas por un cable ya son una red — el resto
(routers, switches, Internet) son formas más grandes de la misma idea.

## Las piezas de una red

```
        Red local (LAN) — tu oficina o casa
       ┌─────────────────────────────┐
       │  [Host A]        [Host B]   │
       │      \              /       │
       │       \            /        │
       │          [Switch]           │
       └─────────────┬───────────────┘
                      │
                 [Router]
                      │
              ☁ Internet (WAN)
```

| Elemento | Qué es | Ejemplo cotidiano |
|---|---|---|
| **Host** | Cualquier equipo que participa en la red | Computadora, celular, impresora, servidor |
| **Medio** | Por dónde viaja la información | Cable Ethernet, Wi-Fi, fibra óptica |
| **Switch** | Conecta varios hosts **dentro de una misma red local** | El regulador de tránsito de tu oficina |
| **Router** | Conecta **una red con otra red distinta** | La caseta de salida de tu colonia hacia la carretera |

---

## A fondo: switch vs. router

Esta es la confusión más común de toda la introducción, así que le dedicamos el tiempo
que merece. La regla corta es: **el switch conecta cosas que ya son "familia" entre sí; el
router conecta familias distintas**. Ahora vamos a desarmar esa regla.

### La analogía, ampliada: el fraccionamiento y la caseta

Imagina tu **fraccionamiento** (LAN, tu red local): tiene varias casas y calles internas
que las conectan entre sí. Ese sistema de calles internas — el que le permite a la casa 3
mandarle una carta a la casa 7 sin salir nunca del fraccionamiento — **es el switch**.

A la salida del fraccionamiento hay una **caseta** que lo conecta con la carretera (otras
colonias, otras ciudades, Internet). Esa caseta — la única puerta hacia "lo de afuera" —
**es el router**.

Con esto ya tenemos la intuición. Ahora, las tres diferencias reales que hacen que un
switch y un router *no sean el mismo aparato con nombre distinto*:

### Diferencia 1 — Qué tipo de "dirección" usa cada uno

Dentro del fraccionamiento, para que el cartero interno entregue una carta de la casa 3 a
la casa 7, solo necesita saber **el número de casa dentro del fraccionamiento**. No le
importa la ciudad ni el código postal — esa dirección corta *solo tiene sentido adentro*.
Esa es la **dirección MAC**: un identificador físico, único de fábrica, que el switch usa
para saber a qué cable/puerto está conectado cada dispositivo. Nunca sale del
fraccionamiento.

Para que algo salga del fraccionamiento hacia otra colonia, ya no basta el número de casa
— hace falta una dirección completa que tenga sentido **para cualquiera afuera**: calle,
colonia, ciudad, código postal. Esa es la **dirección IP**: un identificador que sí viaja
fuera de la red local, y que el router usa para decidir hacia qué "carretera" mandar cada
paquete.

| | Switch | Router |
|---|---|---|
| Dirección que lee | **MAC** (física, de fábrica, solo válida dentro de la LAN) | **IP** (lógica, válida globalmente) |
| Pregunta que responde | "¿A qué cable/puerto está conectado este dispositivo?" | "¿Hacia qué red debo mandar esto para que llegue a su destino?" |

### Diferencia 2 — Qué tan lejos llega un "grito"

Si dentro del fraccionamiento alguien grita algo para que todas las casas lo escuchen (un
**broadcast**), el switch lo reparte a todas las casas del fraccionamiento — es su trabajo.
Pero ese grito **nunca sale del fraccionamiento**: el router, en la caseta de salida, no
deja pasar gritos internos hacia la carretera. Si dejara pasar todos los gritos de todas
las colonias del mundo hacia todas las demás, Internet colapsaría en segundos.

Esto tiene una consecuencia práctica: entre más dispositivos metas a una misma red local
(un mismo switch, o switches conectados entre sí sin un router en medio), más "gritos" de
fondo hay para todos — por eso las redes grandes se dividen en varias LANs pequeñas
conectadas por routers, en vez de una sola LAN gigante.

### Diferencia 3 — La "traducción" en la caseta (NAT)

Dentro del fraccionamiento, los números de casa (`192.168.1.10`, `192.168.1.11`...) son
**privados**: se repiten en miles de fraccionamientos distintos en todo el mundo sin
ningún problema, porque nadie de afuera necesita usarlos directamente. El router, en la
caseta, mantiene una libreta donde anota "la casa 10 mandó esta carta" y la reetiqueta con
**la dirección pública** del fraccionamiento completo antes de sacarla a la carretera. Esto
se llama **NAT** (Network Address Translation) — lo vas a volver a ver en Etapa 4.

Por eso tu laptop, tu celular y la impresora de tu casa pueden compartir una sola conexión
a Internet (una sola dirección IP pública) aunque cada uno tenga su propia IP privada
adentro: el router hace de "recepción" que traduce entre el mundo privado de tu casa y el
mundo público de Internet.

### Entonces, ¿por qué mi router de casa hace "las dos cosas"?

Porque el aparato que Telmex/Totalplay/Izzi te instala **integra varios dispositivos en
una sola caja de plástico**: switch (los puertos Ethernet + Wi-Fi para tus dispositivos) +
router (la conexión hacia el proveedor de Internet) + módem (para hablar con el cable o la
fibra del proveedor). Por eso se le llama "router" coloquialmente, aunque técnicamente
adentro hay más de un rol funcionando.

```
Tu "router" de casa (una sola caja física)
┌───────────────────────────────────────────────────────┐
│ Rol de switch:                                         │
│   puertos Ethernet + Wi-Fi (direcciones MAC, local)    │
│                        │                                │
│                        v                                │
│ Rol de router:                                          │
│   NAT + salida a Internet (direcciones IP, externo)    │
│                        │                                │
│                        v                                │
│ Rol de módem:                                           │
│   habla el idioma del cable/fibra del proveedor        │
└──────────────────────┬──────────────────────────────────┘
                        │
              ☁ Proveedor de Internet
```

### Resumen de la sección

- **Switch = adentro de la misma red.** Usa MAC. Reparte todo, incluidos los broadcasts.
- **Router = entre redes distintas.** Usa IP. Decide rutas y traduce direcciones (NAT).
- Un router doméstico es, en el fondo, switch + router + módem en una sola caja.

---

## Clasificación de redes por alcance geográfico

Ya viste esta clasificación el semestre pasado; aquí la completa con **PAN**, la que suele
faltar:

| Sigla | Nombre | Alcance | Ejemplo |
|---|---|---|---|
| **PAN** | Personal Area Network | Unos cuantos metros, sin cables | Tu celular con audífonos Bluetooth |
| **LAN** | Local Area Network | Un edificio, una oficina, una casa | La red de tu escuela |
| **MAN** | Metropolitan Area Network | Una ciudad | La red de semáforos inteligentes de un municipio |
| **WAN** | Wide Area Network | Ciudades, países | Internet es la WAN más grande que existe |

La regla para no confundirlas: entre más grande el alcance, más routers hay en medio —
una PAN casi nunca tiene ni switch (solo un enlace directo), mientras que cruzar una WAN
significa atravesar decenas de routers de distintos proveedores.

## El patrón que se repite todo el curso: cliente–servidor

- **Cliente**: el que **pide** algo (tu navegador, tu celular).
- **Servidor**: el que **responde** (una computadora dedicada a atender peticiones, casi
  siempre encendida 24/7).

```
  Cliente                          Servidor
    │  ── "quiero ver la página web.lab.local" ──>  │
    │  <──────── "aquí está el contenido" ────────  │
```

DNS, DHCP, web, FTP, correo — todos son ese mismo patrón con distinto contenido. En la
Sesión 6 vas a ver ese patrón resuelto paso a paso con un ejemplo real.

### Cliente-servidor no es la única forma de conectarse: punto a punto (P2P)

En cliente-servidor un equipo siempre pide y otro siempre responde — roles fijos. En una
red **punto a punto (P2P — peer to peer)** no hay ese rol fijo: cada equipo puede ser
cliente y servidor al mismo tiempo, y no depende de que un servidor central esté
encendido.

| | Cliente-servidor | Punto a punto (P2P) |
|---|---|---|
| ¿Quién tiene la información? | Centralizada en el servidor | Repartida entre todos los equipos |
| Si un equipo se apaga | Solo afecta si es el servidor | Cualquiera puede afectar, pero el resto sigue funcionando |
| Ejemplo | Un sitio web, tu correo | Compartir una carpeta directo entre dos laptops sin servidor de por medio |

Casi todo lo que vamos a construir en el curso (DNS, DHCP, web) es cliente-servidor —
pero vale la pena saber que no es la única arquitectura que existe.

---

## Ejercicio práctico: encuentra tu propio switch, router e IP ahora mismo

No necesitas la VM para esto — usa la terminal/cmd de tu laptop real ahora:

**En Windows (cmd o PowerShell):**
```
ipconfig /all
```
**En Mac (Terminal):**
```
ifconfig
```

Busca en la salida:
1. Tu propia **dirección IP** (algo como `192.168.1.XX`).
2. La **puerta de enlace** (`Default Gateway` / `router`) — esa IP es tu **router**, la
   caseta de salida de tu fraccionamiento.
3. Tu **dirección MAC** (`Physical Address` / `ether`) — el número de casa que tu switch
   usa para identificarte dentro de la LAN.

Ahora corre esto para ver a otros dispositivos de tu misma red local (tu switch los
conoce por MAC):
```
arp -a
```
Vas a ver una lista de IPs con su MAC correspondiente — son los demás dispositivos que tu
switch ha visto en tu red, sin necesidad de salir nunca por el router.

> ### ✅ Autocheck del ejercicio
> **Pregunta:** ¿por qué `arp -a` solo muestra dispositivos de tu propia red local, y
> nunca aparece ahí, por ejemplo, la MAC del servidor de Google?
> **Respuesta:**
>
> Porque la tabla ARP solo relaciona IP con MAC **dentro de la misma LAN** — es
> justamente el trabajo del switch, y las direcciones MAC nunca salen de la red local
> (repaso de la sección "A fondo: switch vs. router"). El servidor de Google está en otra
> red, identificado por IP; tu router jamás necesita conocer su MAC.

## Autocheck — Sesión 5

> **Pregunta 1:** en tu casa tienes tu laptop, tu celular y una impresora conectados al
> mismo router de Telmex/Totalplay. ¿Qué parte del router está actuando como *switch* y
> qué parte como *router*?
> **Respuesta:**
>
> Los puertos Ethernet/Wi-Fi que unen tu laptop, tu celular y tu impresora entre sí
> funcionan como **switch** (red local, direcciones MAC); el módulo que se conecta al
> proveedor de Internet y les da salida a todos hacia afuera funciona como **router**
> (direcciones IP, NAT).

> **Pregunta 2:** cuando mandas a imprimir desde tu laptop a la impresora de tu casa (ambas
> conectadas al mismo router doméstico), ¿ese tráfico pasa por el "rol de router" (con NAT
> y salida a Internet) o se queda en el "rol de switch"?
> **Respuesta:**
>
> Se queda en el rol de switch. Laptop e impresora están en la misma red local (mismo
> rango de IPs privadas), así que el tráfico nunca necesita salir hacia Internet ni pasar
> por NAT — se entrega directamente usando direcciones MAC dentro de la misma LAN.

> **Pregunta 3:** ¿por qué Internet no se cae cuando alguien en una red local manda un
> broadcast (un "grito" a todas las casas de su fraccionamiento)?
> **Respuesta:**
>
> Porque los routers no reenvían broadcasts hacia otras redes — el broadcast se queda
> contenido dentro de la LAN donde se originó. Si los routers los dejaran pasar, cada
> grito local se propagaría a todas las redes conectadas del mundo.

> **Pregunta 4:** tus audífonos Bluetooth conectados al celular, ¿son PAN, LAN, MAN o WAN?
> ¿Y la red Wi-Fi completa de tu escuela?
> **Respuesta:**
>
> Los audífonos Bluetooth son **PAN** (unos cuantos metros, un enlace directo sin
> switch/router de por medio). La red Wi-Fi de tu escuela es **LAN** (cubre un edificio
> completo, con switches y probablemente varios puntos de acceso, todos bajo el mismo
> router de salida a Internet).

> **Pregunta 5:** cuando compartes una carpeta directo entre dos laptops en una red local
> (sin subirla a ningún servidor), ¿es cliente-servidor o punto a punto? ¿Por qué?
> **Respuesta:**
>
> Es **punto a punto (P2P)**. Ninguna de las dos laptops tiene un rol fijo de servidor
> "siempre encendido" — cualquiera de las dos puede pedir o compartir archivos, y si una
> se apaga, la otra no depende de ella para seguir funcionando en la red.

**Siguiente paso:** [Tráfico](06-trafico.md).

---

## Referencias

Tanenbaum, A., & Wetherall, D. J. (2012). *Redes de computadoras* (5a ed.). Pearson
Educación.

Stallings, W. (2004). *Comunicaciones y redes de computadoras*. Pearson.

Plummer, D. C. (1982). *An Ethernet address resolution protocol* (RFC 826). IETF.
https://www.rfc-editor.org/rfc/rfc826

Srisuresh, P., & Egevang, K. (2001). *Traditional IP network address translator
(Traditional NAT)* (RFC 3022). IETF. https://www.rfc-editor.org/rfc/rfc3022
