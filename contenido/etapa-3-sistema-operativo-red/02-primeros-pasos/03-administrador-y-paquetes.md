# El administrador, sudo y los paquetes

> Primeros pasos 3 · ≈50 min · [← Usuarios](02-usuarios.md) ·
> Siguiente: [Práctica: usuarios y sudo](04-practica-usuarios.md)

## Antes de empezar

Al terminar la instalación, en el
[primer inicio de sesión](01-primer-inicio.md), escribiste este comando
para actualizar el servidor:

```bash
sudo apt update && sudo apt upgrade
```

Y más adelante vas a escribir muchas veces comandos como este:

```bash
sudo apt install tree
```

Esta lectura explica **qué significa cada palabra** de esos comandos y qué pasa dentro
del servidor cuando los ejecutas. Para eso hay que entender cuatro ideas, en este orden:

1. Quién es el **administrador** del servidor (`root`).
2. Qué es **`sudo`** y por qué lo escribes antes de algunos comandos.
3. Qué es un **paquete** y de dónde se descarga.
4. Qué hace **`apt`** y por qué hay que **actualizar** el servidor.

Primero vas a leer: los ejemplos de comandos son para que veas cómo se usan, no hace
falta que los escribas mientras lees. Al final, en [Pruébalo en tu VM](#pruébalo-en-tu-vm),
vas a hacerlo tú en tu servidor.

## 1. Repaso: no todas las cuentas pueden hacer lo mismo

En la lectura de [Usuarios](02-usuarios.md) viste que Linux es **multiusuario**: cada
persona tiene su propia cuenta, con su nombre de usuario, su contraseña y su carpeta
personal, y cada cuenta tiene **límites** sobre lo que puede hacer.

Tu cuenta, la que creaste en el
[Paso 9](../01-instalacion-ubuntu-server/09-perfil.md) de la instalación, es una
**cuenta normal**. Con ella puedes trabajar con tus propios archivos y usar los
programas que ya están instalados, pero **no puedes cambiar nada que afecte a todo el
servidor**: instalar programas, cambiar la configuración del sistema, crear usuarios o
apagar el equipo.

Esto es una protección, no un defecto: un error de dedo con una cuenta normal solo
afecta tus propios archivos, no al servidor completo.

## 2. root: el administrador

En todo Linux existe una cuenta especial llamada **`root`** (en inglés, "raíz"). Es el
**administrador** o **superusuario**: puede leer, modificar y borrar **cualquier**
archivo, instalar y quitar programas, y cambiar cualquier configuración. El sistema no
le pone ningún límite y **nunca le pregunta "¿estás seguro?"**.

Ese poder es justo lo que hace peligroso trabajar con `root` todo el tiempo: un comando
mal escrito puede borrar el sistema completo. Por eso, en Ubuntu, la cuenta `root`
**viene bloqueada**: no tiene contraseña y nadie puede iniciar sesión con ella
directamente (Canonical, s.f.-c). En su lugar se usa `sudo`.

**Cómo saber si estás como root.** Fíjate en el último símbolo del prompt:

```
usuario@admredes:~$     <-- $ : cuenta normal
root@admredes:~#        <-- # : administrador (root)
```

En este curso siempre vas a trabajar con `$`. Si algún día ves `#`, ten cuidado con lo
que escribes.

## 3. sudo: pedir permiso de administrador para un solo comando

**`sudo`** viene del inglés *superuser do*, "que lo haga el superusuario". Cuando
escribes `sudo` **antes** de un comando, ese comando, y solo ese, se ejecuta con los
permisos de `root`. Al terminar vuelves a ser un usuario normal.

```
sudo  apt update
└┬─┘  └───┬────┘
 │        el comando que quieres ejecutar
 "ejecuta lo que sigue como administrador"
```

Una comparación: en una escuela, los alumnos no pueden entrar a la dirección cuando
quieran. Pero si un alumno de confianza necesita un documento de ahí, puede pedir la
llave, sacarlo y devolverla. No se queda con la llave, y en la dirección queda anotado
quién la pidió y para qué. `sudo` funciona igual.

### Qué pasa cuando escribes sudo

1. **Te pide tu contraseña**, la de **tu** usuario, no la de `root` (que no tiene). Así
   el sistema confirma que eres tú quien está frente al teclado y no alguien que
   encontró la sesión abierta. Vas a ver una línea que termina en `Password:`
   (contraseña).

2. **Mientras escribes la contraseña no aparece nada**, ni asteriscos. Es normal: el
   sistema sí está recibiendo cada tecla. Escríbela completa y presiona Enter.
3. Si la contraseña es correcta, el comando se ejecuta como administrador.
4. Durante los siguientes minutos `sudo` **no te vuelve a pedir la contraseña**, para
   que puedas escribir varios comandos seguidos.
5. Cada uso de `sudo` **queda registrado**: el sistema anota quién lo usó, cuándo y con
   qué comando.

### Quién puede usar sudo

No cualquier usuario puede usar `sudo`: solo los que pertenecen al grupo de
administradores, que en Ubuntu se llama **`sudo`**. El usuario que se crea durante la
instalación entra automáticamente a ese grupo (Canonical, s.f.-c); por eso tú sí
puedes usarlo. Si un usuario que no está en ese grupo lo intenta, `sudo` le responde
con un mensaje de que **no tiene permiso** y el comando no se ejecuta. Lo vas a
comprobar en la [práctica de usuarios](04-practica-usuarios.md).

Un dato más: desde la versión 25.10, Ubuntu usa una versión nueva de `sudo` llamada
**sudo-rs**, reescrita para ser más segura. Se usa exactamente igual; solo algunos
mensajes en pantalla pueden verse un poco distintos a los de tutoriales viejos
(Canonical, s.f.-c).

### Qué pasa si se te olvida sudo

Si escribes un comando de administrador **sin** `sudo`, el sistema simplemente no lo
deja pasar. Por ejemplo, `apt update` sin `sudo` termina con un error como este:

```
E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
```

No tienes que entender toda la línea. La parte importante es **Permission denied**,
"permiso denegado": te faltó `sudo`. No se rompió nada; vuelve a escribir el comando
con `sudo` al principio.

### Regla de oro

**Usa `sudo` solo cuando el comando lo necesite, y lee el comando antes de presionar
Enter.** Con `sudo`, el sistema hace exactamente lo que escribiste, sin preguntar. Si
copias un comando de Internet que empieza con `sudo`, primero entiende qué hace.

## 4. ¿Qué es un paquete?

Para instalar un programa en Windows, normalmente buscas la página del programa,
descargas un instalador `.exe` y le das "Siguiente, Siguiente, Finalizar". El problema
es que cada programa viene de un sitio distinto, y no siempre sabes si el archivo que
descargaste es el original o uno modificado con un virus.

En Ubuntu los programas se instalan de otra forma, más parecida a la **tienda de
aplicaciones de tu celular** (Play Store o App Store): hay un solo lugar de confianza
de donde sale todo, y una sola herramienta para instalar, actualizar y quitar.

Un **paquete** es **un programa listo para instalarse**, guardado en un solo archivo
(en Ubuntu, con extensión `.deb`). Ese archivo contiene (Canonical, s.f.-b):

- **Los archivos del programa**: el programa mismo, su configuración y su manual.
- **Su información**: nombre, versión, una descripción y quién lo mantiene.
- **Su lista de dependencias**: otros paquetes que necesita para funcionar.

### Dependencias

Casi ningún programa funciona solo. Muchos usan piezas que ya existen en otros
paquetes, en lugar de volver a programarlas cada uno. A esos paquetes que un programa
necesita se les llama **dependencias**.

```
nginx (servidor web)
├── necesita: libssl  (para las conexiones HTTPS)
├── necesita: zlib    (para comprimir datos)
└── necesita: ...
```

Este ejemplo está simplificado: la lista real cambia según la versión. Lo importante es
que **tú no tienes que buscar las dependencias**: la herramienta que instala los
paquetes las encuentra y las instala sola.

## 5. ¿De dónde salen los paquetes? Los repositorios

Un **repositorio** es un servidor en Internet que guarda miles de paquetes, ya
revisados y listos para instalarse. Los repositorios oficiales de Ubuntu los mantiene
**Canonical**, la empresa detrás de Ubuntu. Son la "tienda de aplicaciones" de la
comparación anterior.

En el [Paso 6](../01-instalacion-ubuntu-server/06-mirror.md) de la instalación elegiste
un **mirror** (réplica): una copia de esos repositorios de la que tu servidor descarga
los paquetes. La dirección de ese mirror quedó guardada en un archivo de configuración
dentro de `/etc/apt/`.

Los paquetes de los repositorios oficiales vienen **firmados digitalmente**: antes de
instalar algo, el sistema comprueba que el paquete realmente viene de Ubuntu y que nadie
lo modificó en el camino. Si la firma no coincide, se niega a instalarlo
(Canonical, s.f.-b). Esa es la gran ventaja frente a descargar un `.exe` de cualquier
página.

## 6. apt: la herramienta que maneja los paquetes

**`apt`** (de *Advanced Package Tool*) es el programa con el que instalas, actualizas y
quitas paquetes en Ubuntu (Canonical, s.f.-b). Siempre se escribe `apt`, un espacio y
**qué quieres hacer**:

| Comando | Qué hace | ¿Necesita sudo? |
|---|---|---|
| `apt update` | Descarga la **lista** más nueva de paquetes disponibles. No instala nada | Sí |
| `apt upgrade` | **Instala las versiones nuevas** de los paquetes que ya tienes | Sí |
| `apt install <paquete>` | Instala un paquete nuevo, con sus dependencias | Sí |
| `apt remove <paquete>` | Quita un paquete | Sí |
| `apt search <palabra>` | Busca paquetes que tengan esa palabra en su descripción | No |
| `apt show <paquete>` | Muestra la información de un paquete: versión, descripción, tamaño | No |
| `apt list --installed` | Lista los paquetes que ya tienes instalados | No |

Fíjate en la última columna: **lo que solo consulta información no necesita `sudo`**;
lo que **cambia** el servidor (instalar, actualizar, quitar) **sí**.

### La diferencia que más confunde: update y upgrade

Las dos palabras significan "actualizar" en inglés, pero hacen cosas distintas:

- **`apt update`** actualiza **la lista**. Tu servidor guarda una lista de todos los
  paquetes que existen en el repositorio y de su versión más reciente. `update`
  descarga esa lista de nuevo para saber qué hay de nuevo, pero **no instala nada**.
- **`apt upgrade`** actualiza **los programas**. Compara lo que tienes instalado con la
  lista y descarga e instala las versiones nuevas.

Con la tienda del celular: `update` es **abrir la tienda para que revise si hay
actualizaciones**; `upgrade` es **presionar "Actualizar todo"**. Por eso siempre se
corren **en ese orden**: primero `update` y después `upgrade`. Si corres `upgrade` sin
`update`, el sistema compara contra una lista vieja y puede no encontrar las
actualizaciones más recientes.

## 7. Leyendo un comando completo, palabra por palabra

Ahora sí puedes leer el comando del inicio:

```
sudo   apt    install   tree
└┬─┘   └┬┘    └──┬──┘   └┬─┘
 │      │        │       el paquete: tree (muestra carpetas en forma de árbol)
 │      │        lo que quieres hacer: instalar
 │      el programa que maneja paquetes
 como administrador, porque instalar cambia el servidor
```

Y el comando del primer inicio de sesión:

```
sudo apt update   &&   sudo apt upgrade
└──────┬──────┘   └┬┘  └───────┬──────┘
       │           │           después, instala las versiones nuevas
       │           "si lo anterior salió bien, sigue con lo siguiente"
       primero, descarga la lista nueva
```

El `&&` une dos comandos: el segundo solo se ejecuta si el primero terminó sin errores
(Free Software Foundation, s.f.). Si `update` falla, por ejemplo porque no hay
Internet, `upgrade` ni siquiera empieza.

### Qué vas a ver en pantalla al instalar

Cuando ejecutas `sudo apt install` con un paquete, pasa esto, en orden:

1. `sudo` te pide tu contraseña (si no la escribiste hace poco).
2. `apt` busca el paquete en su lista y calcula qué dependencias le faltan.
3. Te muestra **qué va a instalar** y **cuánto espacio va a ocupar**, y te pregunta si
   quieres continuar. Como tu servidor está en español, la pregunta se ve así (en
   inglés diría `Do you want to continue? [Y/n]`):

   ```
   ¿Desea continuar? [S/n]
   ```

   La letra en **mayúscula** es la respuesta por defecto: si solo presionas Enter,
   equivale a **Sí**. Para cancelar, escribe `n` y Enter.

4. Descarga los paquetes del mirror, comprueba su firma, los instala y los configura.
5. Regresa al prompt. Si no apareció ningún mensaje que empiece con `E:` (de *error*),
   el programa quedó instalado.

Si un paquete **ya está instalado**, `apt` solo te avisa que ya tienes la versión más
reciente y no hace nada más. No pasa nada malo por intentarlo dos veces.

## 8. Por qué hay que actualizar

Los programas tienen errores. Algunos solo son molestos, pero otros son **fallas de
seguridad**: huecos que un atacante puede aprovechar para entrar al servidor. Cuando se
descubre una, los desarrolladores publican una versión corregida (un **parche**), y la
única forma de que tu servidor la tenga es **actualizarlo**.

Por eso actualizar es una de las tareas básicas de un administrador, y forma parte de
la **seguridad**, la S de FCAPS que viste en la
[Etapa 1](../../etapa-1-funciones-administracion/05-administracion-seguridad.md). Un
servidor sin actualizar es como una casa con una cerradura que todo el mundo sabe
abrir.

Ubuntu Server ayuda con esto: trae instalado un servicio llamado
**unattended-upgrades** que **instala solo las actualizaciones de seguridad**, sin que
tengas que hacerlo a mano (Canonical, s.f.-a). Aun así, el resto de las actualizaciones
te toca a ti con `sudo apt update && sudo apt upgrade`.

Algunas actualizaciones, sobre todo las del **kernel** (el núcleo del sistema), solo
surten efecto después de **reiniciar**. Cuando eso pasa, al iniciar sesión aparece el
mensaje:

```
*** System restart required ***
```

En un servidor real, el reinicio se programa en un horario en que nadie lo esté usando,
porque mientras reinicia los servicios dejan de funcionar.

## 9. ¿Y los snaps?

En el [Paso 12](../01-instalacion-ubuntu-server/12-snaps.md) de la instalación viste
otro tipo de paquete, el **snap**, que se instala con el comando `snap` en lugar de
`apt`. Los dos existen en Ubuntu, pero en este curso **siempre vas a usar `apt`**, que
es la forma tradicional y la que vas a encontrar en casi toda la documentación de
servidores.

## Resumen

```
root          el administrador: puede hacer todo, sin preguntar
$  /  #       prompt de usuario normal / de root
sudo          ejecutar UN comando como administrador (pide TU contraseña)
paquete       un programa listo para instalar (.deb), con sus dependencias
dependencia   otro paquete que un programa necesita para funcionar
repositorio   servidor de confianza de donde se descargan los paquetes
mirror        copia de un repositorio (la elegiste en el Paso 6)
apt           la herramienta que instala, actualiza y quita paquetes
apt update    descarga la LISTA nueva de paquetes (no instala nada)
apt upgrade   instala las versiones nuevas de lo que ya tienes
apt install   instala un paquete nuevo
&&            "si lo anterior salió bien, haz lo siguiente"
```

## Pruébalo en tu VM

Ahora vas a usar `sudo` y `apt` en tu servidor: vas a ver qué pasa si se te olvida
`sudo` y vas a instalar tu primer programa. Sigue guardando las capturas en tu carpeta `capturas-admredes`, como se explica en
[Cómo tomar y guardar las capturas](01-primer-inicio.md#cómo-tomar-y-guardar-las-capturas).

### Paso 1: Actualizar la lista, sin sudo y con sudo

Primero, a propósito, **sin** `sudo`:

```bash
apt update
```

Aparecen unas líneas y al final un error que empieza con `E:` y menciona **Permission
denied**. Es lo que viste en la sección 3: actualizar la lista cambia el servidor, y tu
cuenta sola no tiene permiso.

> 📸 **Captura `NombreApellido-AptSinSudo.png`:** el error de `apt update` sin `sudo`.

Ahora, con `sudo`:

```bash
sudo apt update
```

Te pide tu contraseña (no se ve al escribirla). Después aparecen varias líneas que
empiezan con `Get:` o `Hit:` (o `Obj:` y `Des:` en español): son las listas que se
están descargando del mirror. Al final no debe aparecer ningún `E:`.

### Paso 2: Consultar un paquete

Antes de instalar algo, conviene saber qué es. Consultar no cambia nada, así que **no
necesita `sudo`**:

```bash
apt show tree
```

Busca la línea `Description`: dice para qué sirve el paquete. `tree` muestra las
carpetas y archivos en forma de árbol, como los dibujos de estas lecturas.

### Paso 3: Instalar tree

```bash
sudo apt install tree
```

Fíjate que esta vez **probablemente no te pidió la contraseña**: `sudo` todavía la
recuerda del paso 1. Lee lo que aparece en pantalla: `apt` dice qué paquetes va a
instalar y cuánto espacio va a ocupar. Si pregunta si deseas continuar (`[S/n]` o
`[Y/n]`), presiona **Enter**.

Cuando regrese el prompt, comprueba que el programa quedó instalado y úsalo:

```bash
which tree
tree /etc/apt
```

- `which` (de *which?*, "¿cuál?") dice **dónde quedó guardado** un programa: debe
  responder `/usr/bin/tree`.
- `tree /etc/apt` dibuja el árbol de la carpeta de configuración de `apt`. Ahí está
  guardada, por ejemplo, la dirección del mirror que elegiste al instalar.

> 📸 **Captura `NombreApellido-InstalarTree.png`:** el final de la instalación de
> `tree` y la salida de `which tree` y `tree /etc/apt`.

## Referencias

Canonical Ltd. (s.f.-a). *Automatic updates*. Ubuntu Server documentation.
https://documentation.ubuntu.com/server/how-to/software/automatic-updates/

Canonical Ltd. (s.f.-b). *Package management*. Ubuntu Server documentation.
https://documentation.ubuntu.com/server/how-to/software/package-management/

Canonical Ltd. (s.f.-c). *User management*. Ubuntu Server documentation.
https://documentation.ubuntu.com/server/how-to/security/user-management/

Free Software Foundation. (s.f.). *GNU Bash manual: Lists of commands*.
https://www.gnu.org/software/bash/manual/html_node/Lists.html
