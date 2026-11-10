# Tipos de direcciones IP

> 50 min · [← La dirección IP](01-direccion-ip.md) · Siguiente: [El sistema binario](03-sistema-binario.md)

## Antes de empezar

No todas las direcciones IPv4 sirven para lo mismo. Algunas solo valen dentro de tu
casa, otras son únicas en todo Internet, otras ni siquiera salen del propio equipo.
Un administrador de redes tiene que reconocerlas a simple vista, porque muchas fallas
se diagnostican así: "esta laptop tiene una `169.254`, entonces no le llegó el DHCP".

Hay varias formas de clasificar una dirección, y cada una responde una pregunta
distinta:

| Clasificación | Pregunta que responde |
|---|---|
| Pública o privada | ¿Sirve en Internet o solo dentro de una red? |
| Estática o dinámica | ¿Quién la asignó y si puede cambiar? |
| Unicast, broadcast o multicast | ¿A cuántos equipos va dirigido el paquete? |
| Por clases (A, B, C, D, E) | La forma antigua de dividir direcciones, que todavía vas a ver mencionada |
| Direcciones especiales | Rangos reservados para un uso muy específico |

Una misma dirección cae en varias a la vez: `192.168.1.50` es privada, dinámica y
unicast.

## 1. Públicas y privadas

Esto ya lo viste a fondo en la lectura de [NAT](../etapa-3-sistema-operativo-red/03-acceso-remoto/02-que-es-nat.md)
de la Etapa 3, así que aquí va el resumen y cómo se comparte una dirección pública.

Se reservaron tres rangos como **privados**: cualquier red puede usarlos por dentro,
sin pedirle permiso a nadie, pero nunca se usan en Internet (Rekhter et al., 1996).

| Rango privado | En notación corta | Dónde lo has visto |
|---|---|---|
| `10.0.0.0` a `10.255.255.255` | `10.0.0.0/8` | La red NAT de VirtualBox (`10.0.2.15`) |
| `172.16.0.0` a `172.31.255.255` | `172.16.0.0/12` | Redes de empresas y escuelas |
| `192.168.0.0` a `192.168.255.255` | `192.168.0.0/16` | Casi todas las redes de casa |

Todas las demás direcciones "normales" son **públicas**: únicas en todo Internet y
asignadas por organismos que las administran. El router de tu casa tiene una, y la
comparte con todos tus equipos (enseguida ves cómo).

Un detalle con trampa: el rango privado de en medio llega hasta `172.31`, **no** hasta
`172.255`. Una dirección como `172.40.0.1` es pública.

### Cómo se comparte una dirección pública: NAT

Si las direcciones privadas no sirven en Internet, ¿cómo navega tu celular con
`192.168.1.30`? Gracias a **NAT** (*Network Address Translation*, "traducción de
direcciones de red"): el router **cambia la dirección privada de cada paquete por su
dirección pública** antes de mandarlo a Internet, y hace el cambio al revés cuando
llega la respuesta (Srisuresh & Egevang, 2001).

Así, una casa con laptop, celular y televisión necesita **una sola** dirección pública
para todos. El problema es que las tres respuestas van a llegar a la misma dirección.
Para no confundirlas, el router le da a cada conexión un **puerto** propio del lado de
afuera y lo anota en su **tabla NAT**:

```
   Red de tu casa (privada)                        Internet

  Laptop   192.168.1.20 ──┐
                          │   ┌───────────────┐
  Celular  192.168.1.30 ──┼──>│    Router     │──>  Los tres salen
                          │   │  203.0.113.7  │     como 203.0.113.7
  TV       192.168.1.40 ──┘   └───────────────┘

  Tabla NAT del router:
  ┌──────────────────────────┬──────────────────────────┐
  │ Adentro (privada:puerto) │ Afuera (pública:puerto)  │
  ├──────────────────────────┼──────────────────────────┤
  │ 192.168.1.20 : 51000     │ 203.0.113.7 : 40001      │
  │ 192.168.1.30 : 51000     │ 203.0.113.7 : 40002      │
  │ 192.168.1.40 : 49500     │ 203.0.113.7 : 40003      │
  └──────────────────────────┴──────────────────────────┘
```

Cuando llega una respuesta a `203.0.113.7`, puerto `40002`, el router busca ese puerto
en la tabla, ve que es del celular y le entrega el paquete a `192.168.1.30`. Fíjate en
que la laptop y el celular usaron **el mismo puerto por dentro** (`51000`): por eso el
router no puede reutilizar el de ellos y les da uno distinto a cada uno hacia afuera. A
esta forma de NAT, que usa puertos, se le llama **NAPT**, y es la que usan los routers
de casa (Srisuresh & Egevang, 2001). La dirección `203.0.113.7` es solo un ejemplo: es
de un rango reservado para documentación (Arkko et al., 2010).

Esto es lo que permite que las privadas **se repitan**: tu casa y la de tu vecino
pueden usar `192.168.1.20` al mismo tiempo sin chocar, porque hacia Internet cada una
sale con su propia dirección pública. Lo que pasa con las conexiones que vienen **de
afuera hacia adentro**, y cómo VirtualBox hace lo mismo con tu VM, está en la lectura
de NAT de la Etapa 3.

### Por qué esto importa tanto: ya no quedan direcciones

Las direcciones públicas IPv4 las reparten cinco **registros regionales** (IANA,
s.f.); el que
atiende a México, Latinoamérica y el Caribe es **LACNIC**. El 19 de agosto de 2020 se
terminó el último bloque de direcciones IPv4 que LACNIC tenía disponible: desde
entonces, quien pide direcciones nuevas entra a una **lista de espera** y solo recibe
direcciones que otra organización haya devuelto (LACNIC, s.f.). En otras palabras,
**IPv4 ya se agotó en nuestra región**. Por eso se usan tanto las privadas con NAT, y
por eso existe IPv6 (más abajo).

## 2. Estáticas y dinámicas

| | Estática | Dinámica |
|---|---|---|
| Quién la asigna | El administrador, a mano | Un servidor **DHCP**, automáticamente |
| ¿Cambia? | No, hasta que alguien la cambie | Puede cambiar: se "presta" por un tiempo |
| Se usa en | Servidores, routers, impresoras de red | Laptops, celulares, casi todo lo demás |

En la Etapa 2 viste que el servidor DHCP le **presta** a cada equipo una dirección por
un tiempo, el **arrendamiento** o *lease*, y que el equipo la renueva antes de que
venza (Droms, 1997). Si el equipo se desconecta mucho tiempo, la dirección puede
quedar libre y dársele a otro.

¿Por qué los servidores usan dirección estática? Porque los demás tienen que saber
**dónde encontrarlos**. Si la dirección de tu servidor web cambiara cada semana, cada
semana tendrías que avisarle a todos la nueva. Tu VM, en cambio, tiene una dirección
dinámica (por eso `ip a` decía `dynamic`): para practicar está bien, pero significa
que un día podría cambiar y tendrías que volver a revisarla antes de entrar por SSH.

## 3. Unicast, broadcast y multicast

Esta clasificación no es de la dirección del equipo, sino del **destino** del paquete:
¿a cuántos equipos va?

```
  UNICAST                  BROADCAST                 MULTICAST
  a uno solo               a todos los de la red     a un grupo inscrito

  A ──────> B              A ──┬──> B                A ──┬──> B (inscrito)
                               ├──> C                    │    C (no inscrito)
                               └──> D                    └──> D (inscrito)
```

- **Unicast**: el paquete va a **un solo** equipo. Es casi todo el tráfico: abrir una
  página, una conexión SSH.
- **Broadcast** (difusión): el paquete va a **todos** los equipos de la red local. La
  dirección `255.255.255.255` significa "todos los de esta red" (Mogul, 1984). Lo
  usa, por ejemplo, un equipo que acaba de conectarse y busca un servidor DHCP: todavía
  no sabe a quién preguntarle, así que le grita a todos. Los routers **no dejan pasar**
  un broadcast a otra red; esto será muy importante en la Sesión 5.
- **Multicast**: el paquete va a un **grupo** de equipos que se inscribieron para
  recibirlo, como una transmisión de video a varios salones a la vez. Usa el rango
  `224.0.0.0` a `239.255.255.255` (Deering, 1989).

## 4. Las clases A, B, C, D y E

En Redes y Telecomunicaciones viste las **clases** de direcciones. Las vas a seguir
encontrando en libros, en exámenes de certificación y hasta en la salida de algunos
programas, así que hay que conocerlas. Pero también hay que saber una cosa muy
importante: **hoy ya no se usan para repartir direcciones**.

### Cómo eran

Al principio, la frontera entre parte de red y parte de host estaba fija, y se sabía
mirando **el primer octeto** de la dirección (Postel, 1981):

| Clase | Primer octeto | Parte de red / host | Máscara que implicaba | Para qué |
|---|---|---|---|---|
| **A** | 0 a 127 | 1 octeto / 3 octetos | `255.0.0.0` (`/8`) | Redes enormes |
| **B** | 128 a 191 | 2 octetos / 2 octetos | `255.255.0.0` (`/16`) | Redes medianas |
| **C** | 192 a 223 | 3 octetos / 1 octeto | `255.255.255.0` (`/24`) | Redes pequeñas |
| **D** | 224 a 239 | No aplica | **Ninguna** | Multicast |
| **E** | 240 a 255 | No aplica | **Ninguna** | Reservada |

Vas a encontrar textos que dicen que las clases D y E tienen la máscara
`255.255.255.255`. **No es así**: esas direcciones no se dividen en red y host, porque
no se asignan a equipos (Deering, 1989). La clase D es para multicast y la E está
reservada y no se usa (Cotton et al., 2013).

Dentro de la clase A hay dos excepciones: el `0` y el `127` no se usan como redes
normales; los dos aparecen en las direcciones especiales, más abajo.

### Por qué se abandonaron

Con clases solo había tres tamaños de red: una clase C con 254 equipos, una clase B
con 65,534 o una clase A con más de 16 millones. Una empresa que necesitaba 2,000
direcciones no cabía en una C, así que recibía una B completa y desperdiciaba más de
63,000. Así se estaban acabando las direcciones a toda velocidad.

En 1993 se reemplazó ese sistema por **CIDR** (*Classless Inter-Domain Routing*,
"enrutamiento entre dominios **sin clases**"): la frontera entre red y host ya no
depende del primer octeto, sino que **se indica siempre con la máscara**, y puede caer
en cualquier bit (Fuller et al., 1993; Fuller & Li, 2006). Esa es la notación `/24` que ya viste en tu VM, y
es la base de todo el subneteo.

Así que, cuando hoy alguien dice "una red clase C", en realidad casi siempre quiere
decir "una red `/24`". La idea de clase sirve para platicar, no para calcular.

## 5. Direcciones especiales

Además de las privadas, hay rangos reservados para usos muy concretos. El registro
oficial de todos ellos lo lleva la IANA (Cotton et al., 2013). Estos son los que te
vas a encontrar en la práctica:

| Rango | Nombre | Para qué sirve |
|---|---|---|
| `127.0.0.0/8` | **Loopback** | Hablar con el propio equipo; el paquete nunca sale a la red. La más usada es `127.0.0.1` (Braden, 1989) |
| `169.254.0.0/16` | **Enlace local** | Dirección que un equipo se pone **solo** cuando pidió DHCP y nadie le respondió (Cheshire et al., 2005) |
| `0.0.0.0` | "Este equipo" | Se usa como origen cuando el equipo todavía no tiene dirección, por ejemplo al pedir DHCP |
| `255.255.255.255` | Broadcast limitado | "Todos los equipos de esta red" (Mogul, 1984) |
| `100.64.0.0/10` | Espacio compartido | Lo usan algunos proveedores de Internet entre su red y tu router (Weil et al., 2012) |
| `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` | Documentación | Solo para ejemplos en libros y manuales; no pertenecen a nadie (Arkko et al., 2010) |
| `224.0.0.0/4` | Multicast | La antigua clase D (Deering, 1989) |
| `240.0.0.0/4` | Reservada | La antigua clase E; no se usa (Cotton et al., 2013) |

Dos de estas son pistas de diagnóstico muy útiles:

- **`169.254.x.x` significa "no me llegó el DHCP".** Si una laptop con Windows tiene
  una dirección así, no es un problema de Internet: es que el equipo no pudo hablar con
  el servidor DHCP (cable desconectado, Wi-Fi mal conectado, servidor apagado). Windows
  la llama **APIPA**.
- **`127.0.0.1` siempre responde**, aunque el equipo no tenga red. Si un `ping` a
  `127.0.0.1` falla, el problema está en el propio sistema operativo, no en la red.

## 6. Un vistazo a IPv6

La solución de fondo al agotamiento de IPv4 es **IPv6**, con direcciones de **128
bits** en lugar de 32 (Deering & Hinden, 2017). Eso da 2¹²⁸ direcciones: un número de
39 cifras, suficiente para no volver a tener el problema.

Una dirección IPv6 se escribe en **hexadecimal** (dígitos del `0` al `9` y letras de la
`a` a la `f`), en 8 grupos separados por dos puntos. Los ceros a la izquierda se pueden
omitir, y una racha de grupos en cero se puede abreviar **una sola vez** con `::`
(Hinden & Deering, 2006):

```
2001:0db8:0000:0000:0000:0000:0000:0050      (completa)
2001:db8::50                                  (abreviada)
```

`2001:db8::/32` es el rango de IPv6 reservado para documentación (Huston et al.,
2004), el equivalente a `203.0.113.0/24`.

Tu VM ya tiene una dirección IPv6, aunque nunca la configuraste: una que empieza con
**`fe80:`**. Es una dirección de **enlace local**, que todo equipo con IPv6 se pone
solo y que solo sirve dentro de la red local (Hinden & Deering, 2006). En el resto de
la etapa seguimos con IPv4; IPv6 queda como el siguiente paso cuando quieras
profundizar.

## Ejemplo resuelto: clasificar direcciones

| Dirección | Pública o privada | Clase (antigua) | Observación |
|---|---|---|---|
| `192.168.1.50` | Privada | C | Una red de casa |
| `10.0.2.15` | Privada | A | La red NAT de VirtualBox |
| `172.20.5.9` | Privada | B | Está entre `172.16` y `172.31` |
| `172.40.5.9` | **Pública** | B | Se pasa de `172.31`: ya no es del rango privado |
| `8.8.8.8` | Pública | A | Un servidor DNS público |
| `127.0.0.1` | Especial | (A) | Loopback: el propio equipo |
| `169.254.10.4` | Especial | (B) | Enlace local: no llegó el DHCP |
| `230.1.1.1` | Especial | D | Multicast: no se le asigna a un equipo |

## Resumen

- **Privadas** (`10/8`, `172.16/12`, `192.168/16`): solo dentro de una red, se repiten
  en millones de casas. **Públicas**: únicas en Internet. En nuestra región, LACNIC
  agotó sus direcciones IPv4 en 2020.
- **NAT**: el router cambia la dirección privada de cada paquete por su dirección
  pública. Con **puertos** distintos en su tabla NAT (**NAPT**), todos los equipos de
  una casa comparten una sola dirección pública.
- **Estáticas**: las pone el administrador y no cambian (servidores). **Dinámicas**: las
  presta un servidor DHCP y pueden cambiar (casi todo lo demás).
- **Unicast** va a uno, **broadcast** a todos los de la red, **multicast** a un grupo.
  Los routers no dejan pasar el broadcast.
- Las **clases** A, B y C fijaban la frontera según el primer octeto. Desde 1993 se usa
  **CIDR**: la frontera la marca siempre la máscara. D (multicast) y E (reservada) no
  tienen máscara.
- `127.0.0.1` es el propio equipo; `169.254.x.x` significa que no llegó el DHCP.
- **IPv6** usa 128 bits en hexadecimal; tu VM ya tiene una dirección `fe80:`.

## Ejercicios

1. Clasifica como pública, privada o especial: `192.168.100.1` · `172.15.0.1` ·
   `172.31.255.254` · `11.0.0.1` · `10.255.0.1` · `127.0.0.53` · `224.0.0.1`
2. ¿A qué clase antigua pertenecen `130.10.0.1`, `223.1.1.1`, `126.0.0.1` y
   `245.0.0.1`?
3. Un alumno dice que su laptop "no tiene Internet" y en su configuración ve
   `169.254.33.7`. ¿Qué le dirías que revise?
4. ¿Por qué un servidor web debe tener una dirección estática?
5. Abrevia la dirección IPv6 `2001:0db8:0000:0000:0000:0000:0000:0001`.

## Soluciones

1. `192.168.100.1` privada. `172.15.0.1` **pública** (el rango privado empieza en
   `172.16`). `172.31.255.254` privada (es casi la última del rango). `11.0.0.1`
   pública. `10.255.0.1` privada. `127.0.0.53` especial: loopback, todo `127.x.x.x` lo
   es. `224.0.0.1` especial: multicast.
2. `130.10.0.1`: B (128 a 191). `223.1.1.1`: C (192 a 223). `126.0.0.1`: A (0 a 127).
   `245.0.0.1`: E (240 a 255), reservada.
3. Que no le llegó la respuesta del servidor DHCP: que revise que el cable esté bien
   conectado o que el Wi-Fi esté realmente conectado a la red, y que el router esté
   encendido. No es un problema "de Internet", sino de la red local.
4. Porque los clientes necesitan saber siempre dónde encontrarlo. Si su dirección
   cambiara, dejarían de llegar a él hasta que se enteraran de la nueva.
5. `2001:db8::1`. Se quitan los ceros a la izquierda de `0db8` y los seis grupos en
   cero se reemplazan por `::`.

## Pruébalo en tu VM

### Paso 1: encuentra la dirección loopback

```bash
ip -4 address
```

En la sesión anterior te fijaste en el bloque de `enp0s3`. Ahora mira el bloque de
**`lo`** (de *loopback*): su línea `inet` dice `127.0.0.1/8`. Esa interfaz no es una
tarjeta de red: es una interfaz virtual para que el sistema hable consigo mismo.

### Paso 2: haz ping al propio equipo

```bash
ping -c 3 127.0.0.1
```

`ping` manda pequeños paquetes de prueba y espera la respuesta; la opción `-c 3`
(*count*) le dice que mande solo 3 y se detenga (Linux man-pages project, s.f.-b). Sin
`-c`, en Linux `ping` sigue para siempre hasta que lo detienes con **Ctrl + C**. Deben
salir tres líneas de respuesta y al final un resumen con `0% packet loss` ("0 % de
paquetes perdidos"). Estos paquetes nunca salieron de tu VM.

### Paso 3: clasifica tus propias direcciones

Con la salida del paso 1, llena esta tabla en tu cuaderno:

| Interfaz | Dirección | ¿Pública, privada o especial? | ¿Estática o dinámica? |
|---|---|---|---|
| `lo` | `127.0.0.1` | | |
| `enp0s3` | (la tuya) | | |

Pista: la de `enp0s3` decía `dynamic` en la sesión anterior.

### Paso 4: tu dirección IPv6

```bash
ip -6 address
```

Igual que `-4`, pero solo muestra direcciones **IPv6** (Linux man-pages project,
s.f.-a). En el bloque de `enp0s3` busca la línea `inet6` que empieza con `fe80::`:
es la dirección de enlace local que tu VM se puso sola. En el bloque de `lo` aparece
`::1/128`, que es el loopback de IPv6, el equivalente a `127.0.0.1`. Puede que también
veas una dirección IPv6 que **no** empieza con `fe80`; si aparece, es que el router de
tu casa sí reparte IPv6, y no hay nada que hacer con ella por ahora.

## Referencias

Arkko, J., Cotton, M., & Vegoda, L. (2010). *IPv4 address blocks reserved for
documentation* (RFC 5737). IETF. https://www.rfc-editor.org/rfc/rfc5737

Braden, R. (Ed.). (1989). *Requirements for Internet hosts — Communication layers*
(RFC 1122). IETF. https://www.rfc-editor.org/rfc/rfc1122

Cheshire, S., Aboba, B., & Guttman, E. (2005). *Dynamic configuration of IPv4
link-local addresses* (RFC 3927). IETF. https://www.rfc-editor.org/rfc/rfc3927

Cotton, M., Vegoda, L., Bonica, R., & Haberman, B. (2013). *Special-purpose IP address
registries* (RFC 6890). IETF. https://www.rfc-editor.org/rfc/rfc6890

Deering, S. (1989). *Host extensions for IP multicasting* (RFC 1112). IETF.
https://www.rfc-editor.org/rfc/rfc1112

Deering, S., & Hinden, R. (2017). *Internet Protocol, version 6 (IPv6) specification*
(RFC 8200). IETF. https://www.rfc-editor.org/rfc/rfc8200

Droms, R. (1997). *Dynamic host configuration protocol* (RFC 2131). IETF.
https://www.rfc-editor.org/rfc/rfc2131

Fuller, V., Li, T., Yu, J., & Varadhan, K. (1993). *Classless Inter-Domain Routing
(CIDR): An address assignment and aggregation strategy* (RFC 1519). IETF.
https://www.rfc-editor.org/rfc/rfc1519

Fuller, V., & Li, T. (2006). *Classless Inter-domain Routing (CIDR): The Internet
address assignment and aggregation plan* (RFC 4632). IETF.
https://www.rfc-editor.org/rfc/rfc4632

Hinden, R., & Deering, S. (2006). *IP version 6 addressing architecture* (RFC 4291).
IETF. https://www.rfc-editor.org/rfc/rfc4291

Huston, G., Lord, A., & Smith, P. (2004). *IPv6 address prefix reserved for
documentation* (RFC 3849). IETF. https://www.rfc-editor.org/rfc/rfc3849

IANA. (s.f.). *Number resources*. https://www.iana.org/numbers

LACNIC. (s.f.). *Phases of IPv4 exhaustion*.
https://www.lacnic.net/en/web/lacnic/agotamiento-ipv4

Linux man-pages project. (s.f.-a). *ip-address(8): Protocol address management*.
https://man7.org/linux/man-pages/man8/ip-address.8.html

Linux man-pages project. (s.f.-b). *ping(8): Send ICMP ECHO_REQUEST to network hosts*.
https://man7.org/linux/man-pages/man8/ping.8.html

Mogul, J. (1984). *Broadcasting Internet datagrams* (RFC 919). IETF.
https://www.rfc-editor.org/rfc/rfc919

Postel, J. (Ed.). (1981). *Internet protocol* (RFC 791). IETF.
https://www.rfc-editor.org/rfc/rfc791

Rekhter, Y., Moskowitz, B., Karrenberg, D., de Groot, G. J., & Lear, E. (1996).
*Address allocation for private internets* (RFC 1918). IETF.
https://www.rfc-editor.org/rfc/rfc1918

Srisuresh, P., & Egevang, K. (2001). *Traditional IP network address translator
(Traditional NAT)* (RFC 3022). IETF. https://www.rfc-editor.org/rfc/rfc3022

Weil, J., Kuarsingh, V., Donley, C., Liljenstolpe, C., & Azinger, M. (2012).
*IANA-reserved IPv4 prefix for shared address space* (RFC 6598). IETF.
https://www.rfc-editor.org/rfc/rfc6598
