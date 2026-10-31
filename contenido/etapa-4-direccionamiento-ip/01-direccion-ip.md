# La dirección IP

> 50 min · [← Tu primer servicio: un servidor web con nginx](../etapa-3-sistema-operativo-red/03-acceso-remoto/04-servidor-web.md) ·
> Siguiente: [Tipos de direcciones IP](02-tipos-de-direcciones.md)

## De qué trata esta etapa

En la Etapa 3 instalaste un servidor, le cambiaste la red a modo puente, entraste a él
por SSH y abriste su página web desde tu celular. Para todo eso usaste una dirección
como `192.168.1.50` sin detenerte mucho a pensar qué significa cada número.

Esta etapa responde justo eso: **qué es una dirección IP, cómo está hecha por dentro y
cómo un administrador de redes reparte las direcciones** entre los equipos de una red.
El tema final, el **subneteo**, es una de las habilidades básicas de cualquier persona
que trabaja con redes, y el orden de las sesiones va construyendo lo que necesitas para
llegar a él:

| Sesión | Tema | Para qué sirve después |
|---|---|---|
| 1 | La dirección IP (esta sesión) | Saber qué es lo que vamos a dividir |
| 2 | Tipos de direcciones IP | Reconocer qué dirección es cuál |
| 3 | El sistema binario | Ver la dirección como la ve la computadora |
| 4 | La máscara de red | Separar la parte de red de la parte del equipo |
| 5 | Subredes | Entender por qué se divide una red |
| 6 | Subneteo | Dividir una red tú mismo, paso a paso |

Ya conoces varias de estas ideas de Redes y Telecomunicaciones. En esta etapa las vas a
**repasar, corregir donde haga falta y llevar hasta el cálculo**, y cada sesión termina
con ejercicios resueltos y con pasos para comprobarlos en tu VM.

## 1. Primero, el protocolo: IP

**IP** (*Internet Protocol*, "protocolo de Internet") es el conjunto de reglas que
define **cómo se escribe la dirección de cada paquete y cómo se lleva de una red a otra**
hasta su destino (Postel, 1981).

Recuerda de la Sesión 6 de fundamentos que todo lo que viaja por una red va partido en
**paquetes**. Cada paquete lleva escritas dos direcciones, como un sobre de carta:

```
┌───────────────────────────────────────────────┐
│ Origen:  192.168.1.20   (quién lo manda)      │
│ Destino: 192.168.1.50   (a quién va)          │
├───────────────────────────────────────────────┤
│                                               │
│   Datos (un pedazo de la página web,          │
│   del archivo, del mensaje...)                │
│                                               │
└───────────────────────────────────────────────┘
```

Los **routers** que hay en el camino solo leen la dirección de destino y deciden hacia
dónde mandar el paquete. No abren los datos; les basta con la dirección, igual que el
cartero no abre las cartas. IP solo se encarga de **llevar** el paquete; otros
protocolos encima de él, como **TCP** y **UDP** (que viste en Redes y
Telecomunicaciones), se encargan de cosas como revisar que llegó completo.

## 2. Qué es una dirección IP

Una **dirección IP** es el número que identifica a una **interfaz de red** (la tarjeta
de red, el adaptador Wi-Fi) dentro de una red IP. Con ella, los demás equipos saben a
dónde mandarle paquetes.

Fíjate en el detalle: la dirección es de la **interfaz**, no del equipo completo. Una
laptop con Wi-Fi y cable de red conectados al mismo tiempo tiene **dos** direcciones,
una por interfaz. Un router, que conecta varias redes, tiene una dirección en cada red
a la que está conectado (Postel, 1981). En la práctica, cuando el equipo tiene una sola
interfaz, decimos "la IP de la laptop" y todos nos entendemos.

Hoy conviven dos versiones del protocolo:

| Versión | Tamaño de la dirección | Ejemplo |
|---|---|---|
| **IPv4** | 32 bits | `192.168.1.50` |
| **IPv6** | 128 bits | `2001:db8::50` |

Esta etapa trabaja con **IPv4**, que es la que configuraste en tu VM y la que vas a
subnetear. IPv6 aparece en la siguiente sesión, porque tu VM ya tiene una dirección de
ese tipo aunque no la hayas configurado.

## 3. Cómo está hecha una dirección IPv4

Una dirección IPv4 mide **32 bits**: 32 posiciones, cada una con un `0` o un `1`
(Postel, 1981). Escrita así, la dirección `172.16.254.1` se vería de esta forma:

```
10101100000100001111111000000001
```

Imposible de leer y de dictar. Por eso se parte en **4 grupos de 8 bits**, llamados
**octetos** (8 bits = 1 byte), y cada octeto se escribe en decimal, separado por puntos.
A eso se le llama **notación decimal con puntos**:

```
  10101100 . 00010000 . 11111110 . 00000001     (lo que hay en realidad)
     172   .    16    .   254    .     1        (como lo escribimos)
  ────────   ────────   ────────   ────────
  octeto 1   octeto 2   octeto 3   octeto 4
```

Cómo se convierte cada grupo de ocho unos y ceros en un número decimal es el tema de la
Sesión 3. Por ahora quédate con dos reglas que salen de ahí:

- Cada octeto va de **0 a 255**. Con 8 bits se pueden formar exactamente 256
  combinaciones (2⁸), del `00000000` (0) al `11111111` (255). Una dirección como
  `192.168.1.300` **no existe**: 300 no cabe en 8 bits.
- En total hay 2³² direcciones IPv4 posibles: unas **4,300 millones** (4,294,967,296).
  En la lectura de NAT de la Etapa 3 viste que eso no alcanza para todos los
  dispositivos del mundo.

## 4. Dos partes: la red y el equipo

Una dirección IP no es un número suelto, como el número de serie de un aparato. Tiene
**dos partes**, igual que una dirección postal tiene la calle y el número de casa
(Postel, 1981):

```
         192.168.1   .   50
         ─────────       ──
       parte de RED    parte de EQUIPO (host)
      "en qué calle"   "qué casa de esa calle"
```

- La **parte de red** dice **a qué red pertenece** el equipo. Todos los equipos de la
  misma red tienen esa parte igual.
- La **parte de equipo** (en inglés **host**, "anfitrión") distingue a cada equipo
  dentro de esa red. Tiene que ser distinta para cada uno.

En la analogía del fraccionamiento de la Sesión 5 de fundamentos: la parte de red es la
**calle** y la parte de host es el **número de casa**. Dos casas pueden tener el número
50 si están en calles distintas, pero dentro de la misma calle no puede haber dos
casas 50.

¿Dónde termina una parte y empieza la otra? **La dirección sola no lo dice.** En el
ejemplo de arriba la división cae después del tercer octeto, pero podría caer en otro
lugar. Lo que marca la frontera es un segundo número que siempre acompaña a la IP: la
**máscara de red**, que es el tema de la Sesión 4. Cuando en tu VM veas
`192.168.1.50/24`, ese `/24` es justamente la máscara, escrita en forma corta.

Esta división en dos partes es la idea más importante de toda la etapa. Subnetear,
como verás al final, es **mover esa frontera** para hacer redes más pequeñas.

## 5. Características de una dirección IP

Las características que cualquier dirección IP cumple, y que conviene tener claras
antes de seguir:

| Característica | Qué significa |
|---|---|
| Es **lógica**, no física | No viene grabada en el equipo: se asigna, y puede cambiar si el equipo cambia de red |
| Es **única dentro de su red** | Dos equipos de la misma red con la misma IP provocan un **conflicto de IP** y ninguno funciona bien |
| Tiene **red y host** | La parte de red dice dónde está; la de host, quién es |
| Siempre va con su **máscara** | Sin la máscara no se sabe dónde termina la parte de red |
| Se asigna **a mano o por DHCP** | Estática (la escribe el administrador) o dinámica (la reparte un servidor DHCP) |

La primera característica se entiende mejor comparándola con la **dirección MAC**, que
también viste el semestre pasado:

| | Dirección MAC | Dirección IP |
|---|---|---|
| Ejemplo | `08:00:27:3a:9f:12` | `192.168.1.50` |
| Quién la pone | El fabricante, en la tarjeta de red | El administrador o el servidor DHCP |
| ¿Cambia si te mueves de red? | No | Sí |
| ¿Para qué sirve? | Entregar dentro de la misma red local | Llevar el paquete entre redes |

Si llevas tu laptop de tu casa a la escuela, su MAC es la misma, pero su IP cambia,
porque ahora está en otra red con otra parte de red. Es como mudarte: sigues siendo la
misma persona, pero tu dirección postal cambia.

Sobre la última característica: en la Etapa 2 viste que **DHCP** es el servicio que le
"presta" una dirección a cada equipo que se conecta, por un tiempo limitado (Droms,
1997). Tu VM, en modo puente, recibió su dirección así, del router de tu casa. Los
servidores importantes, en cambio, casi siempre llevan una dirección **estática**, para
que nunca cambie y todos sepan dónde encontrarlos. Lo verás con detalle en la siguiente
sesión.

## Ejemplo resuelto: leer una dirección

Un compañero te dice que su VM tiene `192.168.1.73/24`. ¿Qué puedes decir de ella?

1. **Es IPv4**: son cuatro números separados por puntos.
2. **Es válida**: los cuatro octetos (192, 168, 1 y 73) están entre 0 y 255.
3. **La frontera está después del tercer octeto**: eso dice el `/24`; lo entenderás
   bien en la Sesión 4. La parte de red es `192.168.1` y la parte de host es `73`.
4. **Otro equipo de la misma red** tendría que empezar también con `192.168.1.` y
   terminar con un número distinto de 73.

## Resumen

- **IP** es el protocolo que pone direcciones a los paquetes y los lleva de red en red.
- Una **dirección IP** identifica a una **interfaz** de red; un equipo con dos
  interfaces tiene dos direcciones.
- Una dirección **IPv4** mide **32 bits**, se escribe en **4 octetos** decimales
  separados por puntos, y cada octeto va de **0 a 255**.
- Tiene dos partes: la de **red** (la calle) y la de **host** (el número de casa). La
  **máscara** dice dónde termina una y empieza la otra.
- Es **lógica** (cambia si cambias de red, a diferencia de la MAC), **única dentro de
  su red**, y se asigna **a mano** (estática) o **por DHCP** (dinámica).

## Ejercicios

1. ¿Cuáles de estas direcciones IPv4 son válidas y cuáles no? Explica por qué.
   `10.0.0.1` · `192.168.1.256` · `172.16.0` · `8.8.8.8` · `200.100.50.25.1`
2. ¿Cuántos bits mide una dirección IPv4 y cuántos bits tiene cada octeto?
3. En `10.20.30.40/24`, ¿cuál es la parte de red y cuál la de host?
4. Tu laptop tiene la IP `192.168.1.20` en tu casa. La llevas a la escuela y te
   conectas al Wi-Fi. ¿Su dirección MAC cambia? ¿Y su IP? ¿Por qué?
5. Dos equipos de la misma red tienen configurada a mano la misma dirección
   `192.168.1.30`. ¿Qué pasa?

## Soluciones

1. `10.0.0.1` y `8.8.8.8` son válidas. `192.168.1.256` no: 256 no cabe en un octeto
   (el máximo es 255). `172.16.0` no: le falta un octeto. `200.100.50.25.1` no: tiene
   cinco octetos, y una IPv4 tiene exactamente cuatro.
2. 32 bits en total, 8 bits por octeto (4 × 8 = 32).
3. Con `/24`, la frontera cae después del tercer octeto: la parte de red es `10.20.30`
   y la parte de host es `40`.
4. La MAC **no** cambia: viene de fábrica en la tarjeta de red. La IP **sí** cambia: la
   red de la escuela tiene otra parte de red, y su servidor DHCP le presta a la laptop
   una dirección de esa red.
5. Hay un **conflicto de IP**: los demás equipos no saben a cuál de los dos entregar
   los paquetes y la comunicación falla en ambos. Por eso la dirección debe ser única
   dentro de la red.

## Pruébalo en tu VM

Vas a ver la dirección IP de tu servidor, con sus dos partes y su máscara. Enciende tu
VM (debe seguir en modo **puente**, como la dejaste en la Etapa 3) e inicia sesión,
en la ventana de VirtualBox o por SSH desde Git Bash, como prefieras.

### Paso 1: ver las direcciones

```bash
ip -4 address
```

`ip` es el comando de Linux para ver y configurar la red; ya lo usaste en la Etapa 3
como `ip a`, que es la forma corta de `ip address`. La opción `-4` le pide mostrar solo
las direcciones **IPv4** para que la salida sea más corta (Linux man-pages project,
s.f.-a).

Aparecen dos bloques, cada uno con un número, un nombre y dos puntos al inicio. Uno
se llama `lo` y el otro algo parecido a `enp0s3`: son dos **interfaces** de tu VM.
Busca en el bloque de `enp0s3` la línea que empieza con `inet`. Se verá parecida a
esta, con tus propios números:

```
    inet 192.168.1.50/24 metric 100 brd 192.168.1.255 scope global dynamic enp0s3
```

Lo que importa de esa línea:

| Parte | Qué es |
|---|---|
| `inet` | "Dirección de Internet": aquí empieza una dirección IPv4 |
| `192.168.1.50` | La **dirección IP** de tu VM en esa interfaz |
| `/24` | La **máscara** en forma corta: marca dónde termina la parte de red |
| `dynamic` | La dirección la dio un servidor **DHCP** (el router de tu casa), no se escribió a mano |
| `enp0s3` | El nombre de la interfaz |

Anota tu dirección y tu `/` en tu cuaderno; los vas a usar en las siguientes
sesiones.

### Paso 2: comprueba las reglas de esta sesión

Con tu dirección anotada, revisa:

1. Que tenga **cuatro** octetos separados por puntos.
2. Que cada octeto esté entre **0 y 255**.
3. Si tu máscara es `/24`, cuál es tu parte de red (los tres primeros octetos) y cuál
   tu parte de host (el último).

### Paso 3: la dirección MAC de la misma interfaz

```bash
ip link
```

`ip link` muestra las interfaces sin sus direcciones IP. En el bloque de `enp0s3`
aparece una línea que empieza con `link/ether` seguida de seis pares de caracteres
separados por dos puntos: es la **dirección MAC** de la tarjeta de red virtual de tu
VM (Linux man-pages project, s.f.-b). En VirtualBox empieza casi siempre con
`08:00:27`. Fíjate en que es un tipo de dirección completamente distinto de la IP:
esta no cambiaría aunque cambiaras tu VM de red.

## Referencias

Droms, R. (1997). *Dynamic host configuration protocol* (RFC 2131). IETF.
https://www.rfc-editor.org/rfc/rfc2131

Linux man-pages project. (s.f.-a). *ip-address(8): Protocol address management*.
https://man7.org/linux/man-pages/man8/ip-address.8.html

Linux man-pages project. (s.f.-b). *ip-link(8): Network device configuration*.
https://man7.org/linux/man-pages/man8/ip-link.8.html

Postel, J. (Ed.). (1981). *Internet protocol* (RFC 791). IETF.
https://www.rfc-editor.org/rfc/rfc791
