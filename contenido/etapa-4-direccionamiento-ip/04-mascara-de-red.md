# La máscara de red

> 50 min · [← El sistema binario](03-sistema-binario.md) · Siguiente: [Subredes](05-subredes.md)

## Antes de empezar

En la Sesión 1 viste que una dirección IP tiene una **parte de red** (la calle) y una
**parte de host** (el número de casa), y que la dirección sola no dice dónde está la
frontera. En la Sesión 2 viste que antes la frontera la fijaba la clase, y que desde
1993 la marca siempre la **máscara**. Y en la Sesión 3 aprendiste a ver los bits.

Ahora junta las tres cosas. Esta sesión responde cuatro preguntas que un administrador
se hace todos los días con solo mirar una IP y su máscara:

1. ¿Cuál es la **dirección de la red**?
2. ¿Cuál es la **dirección de broadcast**?
3. ¿Qué direcciones **puedo darle a los equipos**, y cuántas son?
4. ¿Estos dos equipos están **en la misma red**, o necesitan un router?

## 1. Qué es la máscara

La **máscara de red** (o **máscara de subred**) es un número de **32 bits**, igual que
una dirección IP, que se pone "encima" de la dirección para marcar la frontera
(Mogul & Postel, 1985):

- Donde la máscara tiene **`1`**, ese bit de la dirección es **parte de red**.
- Donde tiene **`0`**, ese bit es **parte de host**.

Los unos van **siempre juntos a la izquierda** y los ceros **siempre juntos a la
derecha**; nunca se mezclan. Así se ve la máscara más común, `255.255.255.0`:

```
 Dirección:  11000000.10101000.00000001.00110010    192.168.1.50
 Máscara:    11111111.11111111.11111111.00000000    255.255.255.0
             ──────────────────────────┬────────
             24 unos: parte de RED     │ 8 ceros: parte de HOST
                                       │
                                  la frontera
```

El nombre "máscara" viene de ahí: es como una plantilla que **tapa** la parte de host
y deja ver solo la parte de red.

## 2. Dos formas de escribirla

La misma máscara se puede escribir de dos maneras:

| Forma | Ejemplo | Qué indica |
|---|---|---|
| Decimal con puntos | `255.255.255.0` | Los 32 bits, en 4 octetos |
| Prefijo (notación CIDR) | `/24` | Solo **cuántos unos** tiene: 24 |

Las dos dicen exactamente lo mismo. El prefijo es más corto y es el que usan hoy
Linux, los routers y casi toda la documentación (Fuller & Li, 2006): por eso tu VM
mostraba `192.168.1.50/24`. Windows, en cambio, suele mostrar la forma con puntos en
`ipconfig` ("Máscara de subred: 255.255.255.0").

Para pasar de una a otra, usa los valores de máscara de la Sesión 3. Cada octeto
completo de unos es un 255 (8 bits), y el octeto donde cae la frontera tiene uno de
estos valores:

| Unos en ese octeto | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|---|
| Valor | 0 | 128 | 192 | 224 | 240 | 248 | 252 | 254 | 255 |

Ejemplos:

| Prefijo | Cómo se reparte | Máscara |
|---|---|---|
| `/8` | 8 + 0 + 0 + 0 | `255.0.0.0` |
| `/16` | 8 + 8 + 0 + 0 | `255.255.0.0` |
| `/20` | 8 + 8 + **4** + 0 | `255.255.240.0` |
| `/24` | 8 + 8 + 8 + 0 | `255.255.255.0` |
| `/26` | 8 + 8 + 8 + **2** | `255.255.255.192` |
| `/27` | 8 + 8 + 8 + **3** | `255.255.255.224` |
| `/30` | 8 + 8 + 8 + **6** | `255.255.255.252` |

Fíjate: `/8`, `/16` y `/24` son las máscaras de las antiguas clases A, B y C. Con CIDR
la frontera puede caer en **cualquier** bit, también a la mitad de un octeto, como en
`/20` o `/26`. Ahí está la puerta al subneteo.

Una máscara como `255.255.255.100` **no existe**: 100 es `01100100`, con unos y ceros
mezclados.

## 3. La dirección de red: la operación AND

Para saber a qué red pertenece una dirección, la computadora hace una operación
llamada **AND** ("Y") entre la dirección y la máscara, **bit por bit** (Mogul & Postel,
1985). La regla es muy simple:

| Bit de la IP | Bit de la máscara | Resultado |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | **1** |

Solo sale `1` cuando **los dos** son `1`. En la práctica significa: **donde la máscara
tiene 1, el bit de la IP se copia tal cual; donde tiene 0, se vuelve 0**. Es decir, se
conserva la parte de red y la parte de host se llena de ceros.

**Ejemplo fácil: `192.168.1.50/24`.** La máscara tiene 255 en los tres primeros
octetos (se copian) y 0 en el último (se vuelve 0):

```
 IP:        192 . 168 .  1  .  50
 Máscara:   255 . 255 . 255 .   0
 ─────────────────────────────────
 Red:       192 . 168 .  1  .   0     →  192.168.1.0/24
```

Atajo: con un octeto de máscara en **255** el octeto de la IP se copia, y con uno en
**0** se vuelve 0. Solo hay que hacer binario en el octeto donde cae la frontera.

**Ejemplo con la frontera a la mitad: `192.168.10.200/26`.** `/26` es
`255.255.255.192`. Los tres primeros octetos se copian; el cuarto hay que hacerlo en
binario:

```
 200 en binario:   1 1 0 0 1 0 0 0
 192 (máscara):    1 1 0 0 0 0 0 0
                   ───────────────
 AND:              1 1 0 0 0 0 0 0   =  192
```

La dirección de red es **`192.168.10.192/26`**. Esa dirección **identifica a la red
completa**, como el nombre de la calle: no se le asigna a ningún equipo.

## 4. La dirección de broadcast

La **dirección de broadcast** de una red es la que tiene **toda la parte de host en
unos**. Un paquete enviado a ella les llega a **todos** los equipos de esa red (Mogul,
1984). Tampoco se le asigna a ningún equipo.

Siguiendo los ejemplos:

```
 192.168.1.0/24       red:        ... 1 . 00000000  →  192.168.1.0
                      broadcast:  ... 1 . 11111111  →  192.168.1.255

 192.168.10.192/26    red:        ... 10 . 11 000000  →  192.168.10.192
                      broadcast:  ... 10 . 11 111111  →  192.168.10.255
                                           ── ──────
                                          red  host
```

## 5. Las direcciones para equipos y cuántas son

Entre la dirección de red y la de broadcast quedan las direcciones que **sí** se le
pueden dar a los equipos, el **rango de hosts**:

- **Primer host** = dirección de red + 1.
- **Último host** = dirección de broadcast − 1.

En binario se ve por qué. Con `192.168.10.192/26`, la parte de red del último octeto
(`11`) no se mueve, y la parte de host recorre todas sus combinaciones:

```
            red | host
             11 | 000000   →  .192   red        (host todo en 0)
             11 | 000001   →  .193   primer host
             11 | 000010   →  .194   segundo host
             11 | 000011   →  .195   tercer host
                   ...
             11 | 111110   →  .254   último host  (todo en 1 menos el último bit)
             11 | 111111   →  .255   broadcast  (host todo en 1)
```

El primer host es "todo en 0 menos el último bit", y el último host es "todo en 1
menos el último bit". Por eso son siempre la red + 1 y el broadcast − 1.

¿Cuántas son? Si la parte de host tiene **h** bits, hay 2ʰ combinaciones, y se restan
las dos reservadas (red y broadcast):

```
                 hosts utilizables  =  2ʰ − 2
```

| Prefijo | Bits de host (h = 32 − prefijo) | Direcciones (2ʰ) | Hosts (2ʰ − 2) |
|---|---|---|---|
| `/30` | 2 | 4 | 2 |
| `/29` | 3 | 8 | 6 |
| `/28` | 4 | 16 | 14 |
| `/27` | 5 | 32 | 30 |
| `/26` | 6 | 64 | 62 |
| `/25` | 7 | 128 | 126 |
| `/24` | 8 | 256 | 254 |
| `/23` | 9 | 512 | 510 |
| `/22` | 10 | 1,024 | 1,022 |
| `/21` | 11 | 2,048 | 2,046 |
| `/20` | 12 | 4,096 | 4,094 |
| `/16` | 16 | 65,536 | 65,534 |

**Ojo con un error muy común:** una red `/24` (`255.255.255.0`) **no** tiene 24
equipos. Tiene 8 bits de host, 256 direcciones y **254 hosts utilizables**. El 24 es el
número de bits de red, no de equipos.

## 6. La puerta de enlace

De esas direcciones de host, una casi siempre se reserva para el **router** de la red,
la **puerta de enlace** (*gateway*): el equipo al que se le entregan los paquetes que
van **fuera** de la red. Por costumbre suele ser **la primera** (`192.168.1.1`) o la
última, pero es una convención, no una regla: puede ser cualquiera del rango.

## 7. ¿Están en la misma red?

Esta es la pregunta que tu propia computadora se hace **cada vez que manda un
paquete** (Braden, 1989):

1. Calcula la dirección de red **propia** (su IP AND su máscara).
2. Calcula la red del **destino**, con **su propia máscara**.
3. Si salen **iguales**, el destino está en la misma red: le entrega el paquete
   **directo**, a través del switch.
4. Si salen **distintas**, el destino está en otra red: le entrega el paquete **al
   gateway**, para que el router lo lleve.

**Ejemplo.** Tu VM es `192.168.1.50/24`. Su red es `192.168.1.0`.

| Destino | Su red con `/24` | ¿Misma red? | Qué hace tu VM |
|---|---|---|---|
| `192.168.1.20` (tu laptop) | `192.168.1.0` | Sí | Se lo entrega directo |
| `192.168.1.254` | `192.168.1.0` | Sí | Directo |
| `192.168.2.20` | `192.168.2.0` | No | Se lo da al gateway |
| `8.8.8.8` | `8.8.8.0` | No | Se lo da al gateway |

Por eso, si un equipo tiene **mal la máscara**, puede creer que alguien está "fuera"
cuando está a su lado, o al revés, y la comunicación falla aunque los cables estén
perfectos.

## Ejemplo resuelto completo: `172.16.45.130/20`

Una sola dirección con la frontera en el tercer octeto, para practicar todo junto.

**1. Máscara.** `/20` = 8 + 8 + 4 + 0 → `255.255.240.0`. La frontera cae en el
**tercer** octeto (4 bits de red y 4 de host ahí). Bits de host: 32 − 20 = 12.

**2. Dirección de red.** Primer y segundo octeto se copian (máscara 255), el cuarto se
vuelve 0 (máscara 0). El tercero, en binario:

```
  45 en binario:    0 0 1 0 | 1 1 0 1
 240 (máscara):     1 1 1 1 | 0 0 0 0
                    ─────────────────
 AND:               0 0 1 0 | 0 0 0 0   =  32
```

Red: **`172.16.32.0/20`**.

**3. Broadcast.** Todos los bits de host en 1: los 4 de la derecha del tercer octeto y
los 8 del cuarto.

```
 tercer octeto:     0 0 1 0 | 1 1 1 1   =  47
 cuarto octeto:     1 1 1 1 1 1 1 1     =  255
```

Broadcast: **`172.16.47.255`**.

**4. Rango de hosts.** De `172.16.32.1` a `172.16.47.254`. Fíjate que cruza varios
valores del tercer octeto: `172.16.40.0`, por ejemplo, es un host normal de esta red.

**5. Cuántos.** 2¹² − 2 = **4,094** hosts.

## Resumen

- La **máscara** tiene 32 bits: **unos** en la parte de red y **ceros** en la de host,
  siempre los unos a la izquierda.
- Se escribe con puntos (`255.255.255.0`) o como **prefijo** (`/24` = cuántos unos).
- **Dirección de red** = IP **AND** máscara (parte de host en ceros). Identifica la red.
- **Broadcast** = parte de host en unos. Llega a todos los equipos de la red.
- **Hosts utilizables** = 2ʰ − 2, desde red + 1 hasta broadcast − 1. Una `/24` tiene
  254, no 24.
- El **gateway** es el router de la red; suele ser el primer host.
- Un equipo compara su red con la del destino: **igual → directo; distinta → al
  gateway**.

## Ejercicios

1. Escribe como prefijo: `255.255.0.0` · `255.255.255.128` · `255.255.255.248` ·
   `255.255.252.0`
2. Escribe con puntos: `/25` · `/28` · `/22` · `/12`
3. Para `192.168.5.77/24`, calcula red, broadcast, primer host, último host y número
   de hosts.
4. Para `192.168.5.77/27`, calcula lo mismo.
5. Un equipo tiene `10.0.5.10/16`. ¿Están en su misma red `10.0.200.1`, `10.1.5.10` y
   `10.0.5.255`?
6. ¿Es válida la máscara `255.255.255.200`? ¿Por qué?

## Soluciones

1. `255.255.0.0` = `/16`. `255.255.255.128` = 24 + 1 = `/25`. `255.255.255.248` = 24 +
   5 = `/29`. `255.255.252.0` = 16 + 6 = `/22`.
2. `/25` = `255.255.255.128`. `/28` = `255.255.255.240`. `/22` = `255.255.252.0`.
   `/12` = 8 + 4 → `255.240.0.0` (es la máscara del rango privado `172.16.0.0/12`).
3. Red `192.168.5.0`, broadcast `192.168.5.255`, hosts de `192.168.5.1` a
   `192.168.5.254`, 254 hosts.
4. `/27` = `255.255.255.224`, 5 bits de host. 77 = `01001101`; AND con `11100000` =
   `01000000` = 64. Red `192.168.5.64/27`. Broadcast: `01011111` = 95 →
   `192.168.5.95`. Hosts de `192.168.5.65` a `192.168.5.94`: 2⁵ − 2 = 30 hosts.
5. Con `/16` solo cuentan los dos primeros octetos; la red del equipo es `10.0.0.0`.
   `10.0.200.1` → red `10.0.0.0`: **sí**. `10.1.5.10` → red `10.1.0.0`: **no**, va al
   gateway. `10.0.5.255` → red `10.0.0.0`: **sí**. Es un host normal: en una `/16` el
   broadcast es `10.0.255.255`, no `10.0.5.255`.
6. No. 200 = `11001000`: tiene un `1` después de un `0`, y en una máscara los unos van
   siempre juntos a la izquierda.

## Pruébalo en tu VM

Vas a instalar **ipcalc**, una calculadora de redes que hace en un instante todo lo de
esta sesión y además muestra los bits. Su propio manual dice que está pensada también
como **herramienta para aprender**, porque presenta los resultados en binario (Jodies,
s.f.).

### Paso 1: instalar ipcalc

```bash
sudo apt install ipcalc
```

Ya conoces este comando de la Etapa 3: `sudo` para tener permisos de administrador,
`apt install` para instalar un paquete. Te pide tu contraseña y después confirmar con
**S** o **Y**. El paquete viene del repositorio **universe** de Ubuntu (Canonical
Ltd., s.f.), que en Ubuntu Server está activado desde la instalación. Si apt dice que
no encuentra el paquete, corre primero `sudo apt update` y vuelve a intentar.

### Paso 2: calcula tu propia red

Usa la dirección y el prefijo que anotaste en la Sesión 1. Por ejemplo:

```bash
ipcalc 192.168.1.50/24
```

La salida tiene dos columnas: a la izquierda el valor en decimal y a la derecha el
mismo valor **en binario**, con un espacio que separa la parte de red de la parte de
host (y en colores, si tu terminal los muestra). Estas son las líneas y lo que
significa cada una:

| Línea | Qué es |
|---|---|
| `Address:` | Tu dirección IP |
| `Netmask:` | La máscara con puntos, seguida de `= 24` (el prefijo) |
| `Wildcard:` | La máscara al revés (ceros por unos); la usan algunos routers, por ahora ignórala |
| `=>` | Separa lo que escribiste de lo que ipcalc calculó |
| `Network:` | La **dirección de red**, con su prefijo |
| `HostMin:` | El **primer host** utilizable |
| `HostMax:` | El **último host** utilizable |
| `Broadcast:` | La **dirección de broadcast** |
| `Hosts/Net:` | Cuántos **hosts** caben (2ʰ − 2) |

Al final de la última línea, ipcalc agrega una descripción como `Class C, Private
Internet`. "Private Internet" quiere decir que es un rango **privado** (RFC 1918, de la
Sesión 2). "Class C" es la clase **antigua**: ipcalc la sigue mostrando, pero recuerda
que hoy la red la define el prefijo, no la clase.

Mira la columna binaria de `Address:` y compárala con lo que convertiste a mano en la
Sesión 3: deben coincidir.

### Paso 3: comprueba los ejemplos y ejercicios

```bash
ipcalc 192.168.10.200/26
```

`Network:` debe decir `192.168.10.192/26` y `Broadcast:` `192.168.10.255`, como en la
sección 3. Fíjate en la columna binaria: el espacio que separa red de host ahora cae
**dentro** del cuarto octeto.

Haz lo mismo con el ejemplo completo y con los ejercicios 3 y 4:

```bash
ipcalc 172.16.45.130/20
ipcalc 192.168.5.77/24
ipcalc 192.168.5.77/27
```

ipcalc también entiende la máscara con puntos. Prueba que da exactamente lo mismo:

```bash
ipcalc 192.168.5.77 255.255.255.224
```

### Paso 4: una máscara que no existe

```bash
ipcalc 192.168.5.77 255.255.255.200
```

ipcalc no hace el cálculo: responde con un error que dice que la máscara es inválida
(`INVALID MASK`). Es el ejercicio 6, comprobado.

## Referencias

Braden, R. (Ed.). (1989). *Requirements for Internet hosts — Communication layers*
(RFC 1122). IETF. https://www.rfc-editor.org/rfc/rfc1122

Canonical Ltd. (s.f.). *Package: ipcalc* [Ubuntu packages, resolute].
https://packages.ubuntu.com/resolute/ipcalc

Fuller, V., & Li, T. (2006). *Classless Inter-domain Routing (CIDR): The Internet
address assignment and aggregation plan* (RFC 4632). IETF.
https://www.rfc-editor.org/rfc/rfc4632

Jodies, K. (s.f.). *ipcalc(1): An IPv4 netmask/broadcast/etc calculator* [Página de
manual del paquete ipcalc 0.51]. https://jodies.de/ipcalc

Mogul, J. (1984). *Broadcasting Internet datagrams* (RFC 919). IETF.
https://www.rfc-editor.org/rfc/rfc919

Mogul, J., & Postel, J. (1985). *Internet standard subnetting procedure* (RFC 950).
IETF. https://www.rfc-editor.org/rfc/rfc950
