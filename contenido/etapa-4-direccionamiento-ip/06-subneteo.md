# Subneteo paso a paso

> 50 min · [← Subredes](05-subredes.md)

## Antes de empezar

Esta es la sesión hacia la que iba toda la etapa. Ya sabes qué es una subred y para qué
sirve; ahora vas a **calcular** tú mismo cómo dividir una red. Todo lo que necesitas ya
lo tienes:

| Herramienta | De qué sesión |
|---|---|
| La tabla 128 · 64 · 32 · 16 · 8 · 4 · 2 · 1 y las potencias de 2 | Sesión 3 |
| Prefijo ↔ máscara, hosts = 2ʰ − 2 | Sesión 4 |
| Pedir prestados bits: n bits → 2ⁿ subredes | Sesión 5 |

Solo falta un atajo más y un método en orden. Vas a resolver tres ejemplos, de menos a
más difícil; el último es el ejercicio del manual del curso.

## 1. El atajo: el tamaño de bloque

Cuando divides una red en subredes del mismo tamaño, las subredes quedan **una detrás
de otra**, separadas siempre por la misma cantidad. A esa cantidad se le llama
**tamaño de bloque** (o **salto**), y se calcula con el octeto de la máscara donde cae
la frontera, el **octeto interesante**:

```
             tamaño de bloque  =  256 − (valor de ese octeto de la máscara)
```

Por ejemplo, con `/26` = `255.255.255.192`, el octeto interesante es el cuarto, y vale
192. El bloque es 256 − 192 = **64**: las subredes empiezan en `.0`, `.64`, `.128` y
`.192`.

¿Por qué funciona? Porque el bloque es el valor del **último bit de red** de ese
octeto. En `11000000` el último 1 está en la posición que vale 64. Así que también
puedes sacarlo mirando la tabla de la Sesión 3.

Es lo mismo que viste en la tabla de 3 bits prestados de la Sesión 5: con `/27`, al
pasar los bits prestados de `000` a `001` la subred sube **32**, que es lo que vale el
bit prestado de más a la derecha. Y 256 − 224 (la máscara de `/27`) da también 32. El
atajo solo te ahorra escribir el binario.

Con el bloque, lo demás sale solo:

- Cada subred **empieza** en un múltiplo del bloque.
- Su **broadcast** es uno antes de donde empieza la siguiente.
- Sus hosts van de **red + 1** a **broadcast − 1**.

## 2. El método, en 6 pasos

Te dan una red y un requisito. Sigue siempre estos pasos:

1. **Bits a pedir prestados.** Según lo que te pidan:
   - **Por número de subredes:** el menor **n** tal que **2ⁿ ≥ subredes necesarias**.
   - **Por número de hosts:** el menor **h** tal que **2ʰ − 2 ≥ hosts necesarios**;
     entonces pides prestados **los bits de host que te sobren** (los que había menos
     h).
2. **Nuevo prefijo** = prefijo original + bits prestados.
3. **Nueva máscara**, con la tabla de valores de máscara.
4. **Hosts por subred** = 2ʰ − 2, con h = 32 − nuevo prefijo.
5. **Tamaño de bloque** = 256 − el octeto interesante de la máscara.
6. **Tabla de subredes**: empieza en la red original y suma el bloque.

Y al final, **comprueba los dos requisitos**: que alcancen las subredes **y** que en
cada una quepan los hosts.

Una nota sobre libros viejos: algunos dicen que la **primera** y la **última** subred
no se usan, y por eso calculan 2ⁿ − 2 subredes. Era una regla de los años ochenta
(Mogul & Postel, 1985) que hoy ya no aplica: el equipo actual puede usar **todas** las
subredes, así que se calculan 2ⁿ (Pummill & Manning, 1995). Lo que **sí** se sigue
restando son las 2 direcciones de cada subred (red y broadcast).

## Ejemplo 1: por número de subredes

> La escuela tiene la red `192.168.10.0/24` y necesita **4 subredes**: laboratorio,
> Dirección, Control escolar y Wi-Fi de invitados.

**Paso 1. Bits a pedir prestados.** 2¹ = 2 no alcanza, 2² = 4 sí. **n = 2**.

**Paso 2. Nuevo prefijo.** 24 + 2 = **/26**.

**Paso 3. Nueva máscara.** 2 unos en el cuarto octeto = 128 + 64 = 192 →
**`255.255.255.192`**.

**Paso 4. Hosts por subred.** h = 32 − 26 = 6 → 2⁶ − 2 = **62 hosts**.

**Paso 5. Bloque.** 256 − 192 = **64**.

**Paso 6. Tabla.** Empieza en `.0` y suma 64:

| # | Área | Bits prestados | Red | Primer host | Último host | Broadcast |
|---|---|---|---|---|---|---|
| 1 | Laboratorio | `00` | `192.168.10.0/26` | `.1` | `.62` | `.63` |
| 2 | Dirección | `01` | `192.168.10.64/26` | `.65` | `.126` | `.127` |
| 3 | Control escolar | `10` | `192.168.10.128/26` | `.129` | `.190` | `.191` |
| 4 | Wi-Fi invitados | `11` | `192.168.10.192/26` | `.193` | `.254` | `.255` |

La columna de bits prestados son las 2² = 4 combinaciones de los 2 bits, igual que en
la tabla de la Sesión 5. `01` vale 64 y `10` vale 128: el bloque y el binario dicen lo
mismo.

**Comprobación.** 4 subredes, 62 hosts cada una. Si el laboratorio tiene 40
computadoras, caben; si tuviera 70, **no** cabrían y habría que replantear.

La última subred siempre termina en el broadcast de la red original (`.255`). Si tu
tabla no termina ahí, hay un error en el bloque.

## Ejemplo 2: por número de hosts

> Te dan `192.168.20.0/24` y necesitas subredes en las que quepan al menos **25
> equipos** en cada una. ¿Cuántas subredes puedes hacer?

**Paso 1. Bits.** Aquí manda el número de **hosts**. Busca el menor h con 2ʰ − 2 ≥ 25:

| h | 2ʰ − 2 | ¿Alcanza para 25? |
|---|---|---|
| 4 | 14 | No |
| 5 | **30** | **Sí** |

Necesitas **h = 5** bits de host. Había 8, así que pides prestados 8 − 5 = **3**.

**Paso 2.** 24 + 3 = **/27**.

**Paso 3.** 3 unos = 128 + 64 + 32 = 224 → **`255.255.255.224`**.

**Paso 4.** 2⁵ − 2 = **30 hosts**.

**Paso 5.** 256 − 224 = **32**.

**Paso 6.** 2³ = **8 subredes**:

| # | Red | Primer host | Último host | Broadcast |
|---|---|---|---|---|
| 1 | `192.168.20.0/27` | `.1` | `.30` | `.31` |
| 2 | `192.168.20.32/27` | `.33` | `.62` | `.63` |
| 3 | `192.168.20.64/27` | `.65` | `.94` | `.95` |
| 4 | `192.168.20.96/27` | `.97` | `.126` | `.127` |
| 5 | `192.168.20.128/27` | `.129` | `.158` | `.159` |
| 6 | `192.168.20.160/27` | `.161` | `.190` | `.191` |
| 7 | `192.168.20.192/27` | `.193` | `.222` | `.223` |
| 8 | `192.168.20.224/27` | `.225` | `.254` | `.255` |

**Respuesta:** 8 subredes de 30 hosts cada una.

## Ejemplo 3: el ejercicio del manual

> Una empresa tiene la red `172.17.0.0/16` (una antigua clase B). El administrador
> necesita **16 subredes**, y calcula que la más grande tendrá **1,800 equipos**.

Primero, una observación: `172.17.0.0` está dentro del rango privado `172.16.0.0/12`
(Sesión 2), así que es una red **interna** de la empresa, no una que se use en
Internet. Para subnetear da igual: se calcula exactamente igual.

Aquí hay **dos** requisitos. Empieza por uno y comprueba el otro.

**Paso 1. Bits, por subredes.** 2³ = 8 no alcanza, 2⁴ = 16 sí. **n = 4**.

**Paso 2.** 16 + 4 = **/20**.

**Paso 3.** La frontera cae en el **tercer** octeto, con 4 unos: 128 + 64 + 32 + 16 =
240 → **`255.255.240.0`**.

**Paso 4.** h = 32 − 20 = 12 → 2¹² − 2 = **4,094 hosts**. ¿Caben los 1,800 de la más
grande? **Sí**, de sobra.

**Paso 5.** El octeto interesante es el **tercero**: 256 − 240 = **16**. Ahora el
bloque se suma **en el tercer octeto**, no en el cuarto.

**Paso 6.** Como la frontera está en el tercer octeto, el **rango de hosts cruza
varios valores del tercero**. Cada subred empieza en `172.17.X.0`, con X múltiplo de
16, y su broadcast es el número anterior al inicio de la siguiente:

| # | Red | Primer host | Último host | Broadcast |
|---|---|---|---|---|
| 1 | `172.17.0.0/20` | `172.17.0.1` | `172.17.15.254` | `172.17.15.255` |
| 2 | `172.17.16.0/20` | `172.17.16.1` | `172.17.31.254` | `172.17.31.255` |
| 3 | `172.17.32.0/20` | `172.17.32.1` | `172.17.47.254` | `172.17.47.255` |
| 4 | `172.17.48.0/20` | `172.17.48.1` | `172.17.63.254` | `172.17.63.255` |
| ... | (sigue sumando 16) | | | |
| 15 | `172.17.224.0/20` | `172.17.224.1` | `172.17.239.254` | `172.17.239.255` |
| 16 | `172.17.240.0/20` | `172.17.240.1` | `172.17.255.254` | `172.17.255.255` |

Fíjate en la primera fila: el broadcast de la subred 1 es `172.17.15.255`, porque la
siguiente empieza en `172.17.16.0`, y uno antes de `16.0` es `15.255`.

### ¿Y si se empieza por los hosts?

Haz el cálculo al revés y aparece algo interesante. Para 1,800 hosts: 2¹⁰ − 2 = 1,022
no alcanza, 2¹¹ − 2 = **2,046** sí, así que h = 11. Se piden prestados 16 − 11 = **5**
bits → **/21** (`255.255.248.0`), con **2⁵ = 32 subredes** de 2,046 hosts.

Las dos respuestas cumplen los dos requisitos:

| Opción | Subredes | Hosts en cada una | Qué deja de sobra |
|---|---|---|---|
| `/20` | 16 | 4,094 | Mucho espacio para que **cada área crezca** |
| `/21` | 32 | 2,046 | 16 subredes libres para **áreas nuevas** |

¿Cuál es la correcta? Depende del plan: si la empresa espera crecer en equipos,
`/20`; si espera abrir más áreas o sucursales, `/21`. **Elegir entre opciones que
funcionan, pensando en el futuro, es justo el trabajo de un administrador**. Por eso en
la Sesión 5 se insistió en planear antes de calcular.

## 3. Ubicar una dirección en su subred

La pregunta inversa también es muy común: "me dan una IP y su máscara, ¿en qué subred
está?". Con el bloque se resuelve sin binario.

> ¿En qué subred está `192.168.10.150/26`?

1. `/26` → máscara `.192` → bloque **64**.
2. Los múltiplos de 64 son 0, 64, **128**, 192. El 150 queda entre 128 y 192.
3. La subred empieza en el múltiplo que queda **justo debajo**: **`192.168.10.128/26`**.
4. Su broadcast es uno antes de la siguiente: `192.168.10.191`. Hosts de `.129` a `.190`.

Es la subred 3 del ejemplo 1: el equipo está en **Control escolar**.

## 4. Un adelanto: máscara variable (VLSM)

En los tres ejemplos todas las subredes son del mismo tamaño. En una red real, un
laboratorio de 60 equipos y una Dirección de 5 no necesitan lo mismo: darles a los dos
una `/26` desperdicia casi 60 direcciones en Dirección.

**VLSM** (*Variable Length Subnet Mask*, "máscara de subred de longitud variable")
resuelve eso: se subnetea **empezando por la subred más grande** y el espacio que
sobra se vuelve a dividir en subredes más pequeñas, cada una con su propia máscara
(Pummill & Manning, 1995). Por ejemplo, de `192.168.10.0/24`: el laboratorio recibe
`192.168.10.0/26` (62 hosts), y Dirección, `192.168.10.64/29` (6 hosts), dejando libre
todo lo demás. Usa exactamente los mismos pasos de esta sesión, repetidos. Es el
siguiente nivel cuando ya domines el subneteo con máscara fija.

## Resumen

- **Bloque** = 256 − el octeto interesante de la máscara. Las subredes empiezan en
  múltiplos del bloque; el broadcast es uno antes de la siguiente.
- **Por subredes:** menor n con 2ⁿ ≥ subredes. **Por hosts:** menor h con 2ʰ − 2 ≥
  hosts, y se prestan los bits que sobran.
- Nuevo prefijo = original + bits prestados. Hosts = 2ʰ − 2.
- Hoy se usan **todas** las subredes (2ⁿ), no 2ⁿ − 2.
- Siempre **comprueba los dos requisitos**: subredes y hosts. A veces hay más de una
  respuesta válida, y se elige pensando en el crecimiento.
- La última subred termina en el broadcast de la red original: si no, hay un error.
- **VLSM** usa máscaras distintas para subredes de tamaños distintos.

## Ejercicios

1. Divide `192.168.50.0/24` en **6 subredes**. Da el nuevo prefijo, la máscara, los
   hosts por subred, el bloque y la tabla de las primeras tres subredes y la última.
   ¿Cuántas subredes te sobran?
2. Te dan `192.168.100.0/24` y necesitas subredes de al menos **50 equipos**. ¿Qué
   prefijo usas y cuántas subredes obtienes? Escribe la tabla completa.
3. Divide `172.16.0.0/16` en **8 subredes**. Da la máscara, los hosts por subred y la
   red, primer host, último host y broadcast de la **segunda** subred.
4. ¿En qué subred está `192.168.20.77/27`? Da su red, broadcast y rango de hosts.
5. Una empresa tiene `10.10.0.0/16`, necesita **10 subredes** y la más grande tendrá
   **3,000 equipos**. ¿Qué prefijo propones? Comprueba los dos requisitos.

## Soluciones

1. 2² = 4 no alcanza, 2³ = 8 sí: n = 3 → **/27**, `255.255.255.224`, 30 hosts, bloque
   32. Primeras tres: `192.168.50.0/27` (hosts `.1`–`.30`, broadcast `.31`),
   `192.168.50.32/27` (`.33`–`.62`, `.63`), `192.168.50.64/27` (`.65`–`.94`, `.95`).
   Última: `192.168.50.224/27` (`.225`–`.254`, `.255`). Salen 8 y se necesitan 6:
   **sobran 2**, que quedan libres para crecer.
2. 2⁵ − 2 = 30 no alcanza, 2⁶ − 2 = 62 sí: h = 6, se prestan 2 bits → **/26**
   (`255.255.255.192`), **4 subredes** de 62 hosts: `192.168.100.0/26` (`.1`–`.62`,
   `.63`), `192.168.100.64/26` (`.65`–`.126`, `.127`), `192.168.100.128/26`
   (`.129`–`.190`, `.191`), `192.168.100.192/26` (`.193`–`.254`, `.255`).
3. n = 3 → **/19**, `255.255.224.0` (3 unos en el tercer octeto: 224), h = 13 →
   2¹³ − 2 = **8,190 hosts**, bloque 256 − 224 = 32 en el tercer octeto. Segunda
   subred: red `172.16.32.0/19`, primer host `172.16.32.1`, último host
   `172.16.63.254`, broadcast `172.16.63.255`.
4. `/27` → bloque 32. Múltiplos: 0, 32, **64**, 96... El 77 está entre 64 y 96: red
   **`192.168.20.64/27`**, broadcast `192.168.20.95`, hosts de `.65` a `.94`.
5. Por subredes: 2⁴ = 16 ≥ 10 → n = 4 → **/20**, con 4,094 hosts ≥ 3,000. Cumple los
   dos. Por hosts: 2¹¹ − 2 = 2,046 no alcanza, 2¹² − 2 = 4,094 sí → h = 12 → también
   /20. Aquí los dos caminos dan **la misma** respuesta: `/20` (`255.255.240.0`), con
   16 subredes, de las que sobran 6. Un `/21` no sirve: solo tiene 2,046 hosts.

## Pruébalo en tu VM

Resuelve primero los ejercicios **a mano** y solo después compruébalos con ipcalc,
que ya instalaste en la Sesión 4. Recuerda la forma de pedirle subredes: la red
original, y después **la máscara de las subredes** (Jodies, s.f.).

### Paso 1: los tres ejemplos

```bash
ipcalc 192.168.10.0/24 /26
ipcalc 192.168.20.0/24 /27
```

Compara cada subred numerada con las tablas de los ejemplos 1 y 2: `Network:`,
`HostMin:`, `HostMax:` y `Broadcast:` deben coincidir.

El ejemplo 3 genera 16 subredes y no cabe en la pantalla. Para verla por partes,
manda la salida a `less`, como aprendiste en la Sesión 8 de fundamentos:

```bash
ipcalc 172.17.0.0/16 /20 | less
```

Avanza con la **barra espaciadora**, regresa con **b** y sal con **q**. Comprueba que
la subred 1 termina en `172.17.15.255` y que la 16 es `172.17.240.0/20`.

Y la otra opción del ejemplo 3, la de 32 subredes:

```bash
ipcalc 172.17.0.0/16 /21 | less
```

Hasta abajo, `Subnets:` debe decir `32`.

### Paso 2: tus ejercicios

Comprueba cada ejercicio con el comando que le corresponde:

| Ejercicio | Comando |
|---|---|
| 1 | `ipcalc 192.168.50.0/24 /27` |
| 2 | `ipcalc 192.168.100.0/24 /26` |
| 3 | `ipcalc 172.16.0.0/16 /19` |
| 4 | `ipcalc 192.168.20.77/27` (sin segunda máscara: solo esa dirección) |
| 5 | `ipcalc 10.10.0.0/16 /20 \| less` |

En el ejercicio 4, ipcalc te da la subred de la dirección con `Network:`, igual que
en la Sesión 4.

### Paso 3: ¿y si me equivoco de prefijo?

Prueba qué pasa con el ejercicio 5 si eliges `/21`:

```bash
ipcalc 10.10.0.0/16 /21 | less
```

ipcalc lo calcula sin quejarse: salen 32 subredes de 2,046 hosts. **La herramienta no
sabe que la subred más grande necesita 3,000.** Ese requisito solo lo conoces tú. Por
eso hay que saber subnetear a mano: la calculadora hace las cuentas, pero la decisión
es del administrador.

## Referencias

Jodies, K. (s.f.). *ipcalc(1): An IPv4 netmask/broadcast/etc calculator* [Página de
manual del paquete ipcalc 0.51]. https://jodies.de/ipcalc

Mogul, J., & Postel, J. (1985). *Internet standard subnetting procedure* (RFC 950).
IETF. https://www.rfc-editor.org/rfc/rfc950

Pummill, T., & Manning, B. (1995). *Variable length subnet table for IPv4* (RFC 1878).
IETF. https://www.rfc-editor.org/rfc/rfc1878
