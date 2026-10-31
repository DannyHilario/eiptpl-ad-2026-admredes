# El sistema binario

> 50 min · [← Tipos de direcciones IP](02-tipos-de-direcciones.md) · Siguiente: [La máscara de red](04-mascara-de-red.md)

## Por qué hace falta esto

En la Sesión 1 viste que `172.16.254.1` es, en realidad, una fila de 32 unos y ceros.
Para casi todo, el decimal basta. Pero el subneteo consiste en **mover la frontera
entre la parte de red y la parte de host**, y esa frontera puede caer **a la mitad de un
octeto**. Para ver dónde cae, hay que mirar los bits. Por eso esta sesión es puro
cálculo: convertir de decimal a binario y de binario a decimal, rápido y sin
calculadora.

La buena noticia: para direcciones IP solo necesitas trabajar con **números de 8 bits**
(de 0 a 255), y todo se resuelve con una sola tabla de 8 números.

## 1. Qué es un sistema de numeración

Tú cuentas en **decimal** (base 10): usas diez dígitos, del 0 al 9, y cada posición
vale **diez veces** más que la de su derecha. Cuando escribes `352`, en realidad
quieres decir:

```
   3        5        2
 ×100     ×10      ×1
 ────     ───      ──
 300  +   50   +   2   =  352
```

El **binario** (base 2) funciona exactamente igual, pero con **solo dos dígitos**, el
`0` y el `1`, y cada posición vale **el doble** que la de su derecha. Un dígito binario
se llama **bit** (de *binary digit*, "dígito binario").

Las computadoras usan binario porque sus circuitos trabajan con señales de **dos
estados**, como un interruptor: hay corriente o no hay, encendido o apagado. Es mucho
más fácil construir un circuito que distinga con seguridad entre dos estados que uno
que distinga entre diez.

## 2. La tabla que vas a usar toda la etapa

Un octeto tiene 8 bits. Estos son los valores de cada posición, de izquierda a derecha:

```
 posición:   8     7     6     5     4     3     2     1
 vale:      128    64    32    16    8     4     2     1
             2⁷    2⁶    2⁵    2⁴    2³    2²    2¹    2⁰
```

**Apréndetela de memoria**: empieza en 1 a la derecha y se va duplicando hacia la
izquierda. Si la olvidas en un examen, escribe `1` y ve duplicando: 1, 2, 4, 8, 16, 32,
64, 128.

Dos datos que salen de la tabla:

- Si todos los bits están en `1`, el octeto vale 128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 =
  **255**. Por eso un octeto nunca pasa de 255.
- Si todos están en `0`, vale **0**. Del `00000000` al `11111111` hay 256 valores.

## 3. De binario a decimal: sumar

Para pasar de binario a decimal, escribe el número debajo de la tabla y **suma los
valores de las posiciones donde hay un 1**. Donde hay un 0, esa posición no cuenta.

**Ejemplo: `10101100`**

```
 vale:     128   64   32   16    8    4    2    1
 bits:       1    0    1    0    1    1    0    0
           ───        ──        ──   ──
           128   +    32   +     8 +  4            =  172
```

**Ejemplo: `01100100`**

```
 vale:     128   64   32   16    8    4    2    1
 bits:       0    1    1    0    0    1    0    0
                 ──   ──             ──
                 64 + 32      +       4            =  100
```

## 4. De decimal a binario: restar

Para el camino inverso, recorre la tabla **de izquierda a derecha** y en cada posición
pregúntate: **"¿cabe este valor en lo que me queda?"**

- Si cabe: escribe `1` y **réstalo**.
- Si no cabe: escribe `0` y sigue con lo mismo.

Siempre terminas con exactamente 8 bits y con 0 de sobra.

**Ejemplo: 172**

| Posición | ¿Cabe en lo que queda? | Bit | Queda |
|---|---|---|---|
| 128 | 172 ≥ 128, sí | 1 | 172 − 128 = 44 |
| 64 | 44 ≥ 64, no | 0 | 44 |
| 32 | 44 ≥ 32, sí | 1 | 44 − 32 = 12 |
| 16 | 12 ≥ 16, no | 0 | 12 |
| 8 | 12 ≥ 8, sí | 1 | 12 − 8 = 4 |
| 4 | 4 ≥ 4, sí | 1 | 4 − 4 = 0 |
| 2 | 0 ≥ 2, no | 0 | 0 |
| 1 | 0 ≥ 1, no | 0 | 0 |

Resultado: **172 = `10101100`**. Comprobación: es el mismo número del ejemplo de la
sección 3, al revés.

**Ejemplo: 200**

| Posición | ¿Cabe? | Bit | Queda |
|---|---|---|---|
| 128 | 200 ≥ 128, sí | 1 | 72 |
| 64 | 72 ≥ 64, sí | 1 | 8 |
| 32 | no | 0 | 8 |
| 16 | no | 0 | 8 |
| 8 | 8 ≥ 8, sí | 1 | 0 |
| 4 | no | 0 | 0 |
| 2 | no | 0 | 0 |
| 1 | no | 0 | 0 |

Resultado: **200 = `11001000`**.

Dos errores muy comunes:

- **Escribir menos de 8 bits.** El 5 es `101`, pero en un octeto se escribe
  **`00000101`**. Los ceros a la izquierda no cambian el valor, pero en una dirección IP
  cada octeto lleva siempre sus 8 bits, para que se pueda ver en qué posición está
  cada uno.
- **Empezar por la derecha.** El método de restar funciona de **izquierda a derecha**,
  empezando por el 128.

## 5. Una dirección IP completa

Para convertir una dirección, conviertes **cada octeto por separado** y los unes con
puntos. Este es el ejemplo de la Sesión 1:

```
     172    .    16    .    254    .     1
  10101100  . 00010000 . 11111110  . 00000001
```

- 16 → solo cabe el 16: `00010000`.
- 254 → todos menos el 1: `11111110` (255 − 1).
- 1 → solo el último: `00000001`.

Y la de tu red de casa, `192.168.1.50`:

```
     192    .   168    .     1     .    50
  11000000  . 10101000 . 00000001  . 00110010
```

- 192 = 128 + 64.
- 168 = 128 + 32 + 8.
- 50 = 32 + 16 + 2.

## 6. Los números que más se repiten en redes

Al subnetear vas a ver una y otra vez los mismos números: los que se forman llenando
el octeto con unos **desde la izquierda**. Vale la pena reconocerlos de vista, porque
son justamente los valores que puede tener un octeto de una máscara de red (Sesión 4):

| Bits | Decimal | Cómo sale |
|---|---|---|
| `00000000` | 0 | Ningún bit |
| `10000000` | 128 | 128 |
| `11000000` | 192 | 128 + 64 |
| `11100000` | 224 | 192 + 32 |
| `11110000` | 240 | 224 + 16 |
| `11111000` | 248 | 240 + 8 |
| `11111100` | 252 | 248 + 4 |
| `11111110` | 254 | 252 + 2 |
| `11111111` | 255 | Todos |

Cada fila es la anterior más el siguiente valor de la tabla.

## 7. Las potencias de 2

La segunda cosa que vas a usar todo el tiempo es **cuántas combinaciones** caben en
cierto número de bits. Con **n** bits se pueden formar **2ⁿ** combinaciones distintas:

| Bits (n) | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 2ⁿ | 2 | 4 | 8 | 16 | 32 | 64 | 128 | 256 | 512 | 1024 | 2048 | 4096 |

Por ejemplo, con 2 bits hay 4 combinaciones: `00`, `01`, `10` y `11`. Esta tabla es la
que vas a usar en la Sesión 6 para responder preguntas como "¿cuántos bits necesito
para hacer 16 subredes?" (respuesta: 4, porque 2⁴ = 16).

## 8. Una mención: el hexadecimal

Además del binario vas a encontrarte con el **hexadecimal** (base 16), que usa los
dígitos del `0` al `9` y las letras de la `a` a la `f` (a = 10, ..., f = 15). Cada
dígito hexadecimal equivale exactamente a 4 bits, así que **dos dígitos hexadecimales
representan un octeto completo**: `ff` = `11111111` = 255. Por eso las direcciones MAC
(`08:00:27:3a:9f:12`) y las IPv6 (`2001:db8::1`) se escriben en hexadecimal: es una
forma corta de escribir binario. Para IPv4 y subneteo no lo necesitas.

## Resumen

- El **binario** usa solo `0` y `1`; cada posición vale el **doble** que la de su
  derecha.
- La tabla de un octeto: **128 · 64 · 32 · 16 · 8 · 4 · 2 · 1**. Con todo en 1 suma 255.
- **Binario a decimal:** suma los valores donde hay un 1.
- **Decimal a binario:** de izquierda a derecha, si el valor cabe, pon 1 y réstalo; si
  no, pon 0. Siempre 8 bits.
- Una IP se convierte **octeto por octeto**.
- Los valores de máscara: **0, 128, 192, 224, 240, 248, 252, 254, 255**.
- Con **n bits** hay **2ⁿ** combinaciones.

## Ejercicios

1. Convierte a binario (8 bits): 10 · 75 · 130 · 224 · 255
2. Convierte a decimal: `11110000` · `00100001` · `10111111` · `11111100`
3. Convierte a binario la dirección `10.0.2.15` (la de la red NAT de VirtualBox).
4. ¿Cuántas combinaciones distintas hay con 5 bits? ¿Y con 10?
5. ¿Cuántos bits necesitas, como mínimo, para formar al menos 6 combinaciones
   distintas?

## Soluciones

1. 10 = 8 + 2 = `00001010`. 75 = 64 + 8 + 2 + 1 = `01001011`. 130 = 128 + 2 =
   `10000010`. 224 = 128 + 64 + 32 = `11100000`. 255 = `11111111`.
2. `11110000` = 128 + 64 + 32 + 16 = 240. `00100001` = 32 + 1 = 33. `10111111` = 128 +
   32 + 16 + 8 + 4 + 2 + 1 = 191 (o más rápido: 255 − 64). `11111100` = 252.
3. `00001010.00000000.00000010.00001111` (10 = 8 + 2; 0; 2; 15 = 8 + 4 + 2 + 1).
4. 2⁵ = 32 y 2¹⁰ = 1024.
5. 3 bits: 2² = 4 no alcanza, 2³ = 8 sí.

## Pruébalo en tu VM

Tu servidor puede hacer las conversiones por ti. Úsalo para **comprobar** tus
respuestas, no para sacarlas: en el examen no vas a tener la VM.

### Paso 1: de binario a decimal

```bash
echo $((2#10101100))
```

`echo` muestra un texto en la pantalla. `$(( ... ))` le pide a la terminal (**bash**)
que haga una operación aritmética, y `2#` al inicio del número le dice que ese número
está escrito **en base 2** (Free Software Foundation, s.f.). La terminal responde
`172`.

Cambia los unos y ceros para comprobar los incisos del ejercicio 2. Por ejemplo:

```bash
echo $((2#11110000))
```

Debe responder `240`.

### Paso 2: de decimal a binario

Bash no tiene una forma directa de convertir a binario, pero Ubuntu Server trae
instalado **Python**, un lenguaje de programación que puede hacerlo en una línea:

```bash
python3 -c 'print(format(172, "08b"))'
```

Qué significa cada parte:

| Parte | Qué hace |
|---|---|
| `python3` | Ejecuta Python |
| `-c '...'` | Le da una instrucción directamente, sin necesidad de un archivo |
| `print(...)` | Muestra el resultado en pantalla |
| `format(172, "08b")` | Escribe el 172 en **binario** (`b`), con **8** posiciones rellenando con **0** a la izquierda (Python Software Foundation, s.f.) |

Responde `10101100`. Copia la línea con la flecha hacia arriba (**↑**), cambia el
número y comprueba los incisos del ejercicio 1. Cuida las comillas: las de afuera son
sencillas (`'`) y las de `"08b"` son dobles.

### Paso 3: tu propia dirección

Busca tu dirección con `ip -4 address`, como en la Sesión 1, y conviértela **a mano**
en tu cuaderno, octeto por octeto. Después comprueba cada octeto con el comando del
paso 2. En la siguiente sesión vas a instalar un programa que muestra la dirección
completa en binario de una vez.

## Referencias

Free Software Foundation. (s.f.). *Shell arithmetic* [Bash reference manual].
https://www.gnu.org/software/bash/manual/html_node/Shell-Arithmetic.html

Python Software Foundation. (s.f.). *Format specification mini-language* [The Python
standard library]. https://docs.python.org/3/library/string.html#formatspec
