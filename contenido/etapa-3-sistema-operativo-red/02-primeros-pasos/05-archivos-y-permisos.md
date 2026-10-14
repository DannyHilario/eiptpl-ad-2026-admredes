# Archivos, carpetas y permisos en Linux

> Primeros pasos 5 · ≈50 min · [← Práctica: usuarios y sudo](04-practica-usuarios.md) ·
> Siguiente: [Práctica: archivos y permisos](06-practica-archivos.md)

## Antes de empezar

Tu servidor Ubuntu ya está instalado, pero **no tiene escritorio, ni ventanas, ni
explorador de archivos** como Windows. Solo hay una pantalla negra con texto. Para
usarlo necesitas entender tres ideas, que son las que explica esta lectura:

1. Qué es un **archivo** y qué es una **carpeta**.
2. Cómo están **acomodados** los archivos en Linux y cómo se escribe la "dirección" de
   cada uno.
3. Quién puede **leer, modificar o ejecutar** cada archivo: los **permisos**.

Primero vas a leer y entender. Al final, en [Pruébalo en tu VM](#pruébalo-en-tu-vm),
vas a revisar permisos reales en tu servidor, y en la
[práctica siguiente](06-practica-archivos.md) usarás todo esto con tus propios archivos.

## 1. ¿Qué es un archivo?

Un **archivo** es un conjunto de información guardado en el almacenamiento (el disco)
con un **nombre**. Una foto es un archivo, un documento de Word es un archivo, una
canción es un archivo, y un programa como el navegador también es un archivo.

En la [Sesión 1](../../00-fundamentos-computo/01-hardware.md) comparamos la computadora
con un escritorio de oficina, y el almacenamiento con el **archivero**. Siguiendo esa
idea, un archivo es **una hoja guardada dentro del archivero**, con una etiqueta que
dice su nombre.

### Archivos de texto y archivos binarios

Hay dos tipos de archivo que te conviene distinguir:

| Tipo | Qué contiene | Ejemplo | ¿Se puede leer en la terminal? |
|---|---|---|---|
| **De texto** | Solo letras, números y símbolos, como un bloc de notas | Una lista de nombres, un archivo de configuración | **Sí**: se ve igual que cuando lo escribieron |
| **Binario** | Información que solo entiende un programa | Un programa, una foto, un video | **No**: si lo abres como texto, aparecen símbolos sin sentido |

Esto es importante porque **en un servidor Linux, casi toda la configuración está en
archivos de texto**. Cuando más adelante configures un servidor web o un DNS, lo que
vas a hacer es abrir un archivo de texto, cambiar unas líneas y guardarlo.

## 2. ¿Qué es una carpeta?

Una **carpeta** es un contenedor donde se guardan archivos y también otras carpetas.
En el archivero, una carpeta es **un cajón**, y dentro de un cajón puede haber hojas
(archivos) o más separadores (otras carpetas).

En Linux, a las carpetas se les llama **directorios**. Las dos palabras significan lo
mismo, y en este curso vas a ver las dos.

## 3. Reglas para los nombres en Linux

Linux trata los nombres de archivo de forma distinta a Windows. Hay cuatro reglas que
vas a encontrar desde el primer día:

**Las mayúsculas y las minúsculas cuentan.** `Notas.txt`, `notas.txt` y `NOTAS.txt` son
**tres archivos distintos**. En Windows serían el mismo. Si escribes un nombre con una
mayúscula de más, Linux te va a decir que el archivo no existe.

**La extensión es opcional.** La **extensión** es la parte final del nombre después del
punto: `.txt`, `.docx`, `.jpg`. Windows la usa para saber con qué programa abrir el
archivo. A Linux no le hace falta: muchos archivos del sistema no tienen extensión, por
ejemplo uno llamado simplemente `hostname`. Aun así, para tus propios archivos de texto
conviene ponerles `.txt`, para que tú sepas qué son.

**Mejor sin espacios.** En la terminal, el espacio separa una palabra de otra. Un
archivo llamado `mis notas.txt` se confunde con dos cosas distintas: `mis` y
`notas.txt`. Por eso en Linux se usan guiones: `mis-notas.txt`.

**Un punto al inicio oculta el archivo.** Si el nombre empieza con punto, como
`.bashrc`, el archivo está **oculto**: no aparece en la lista normal. Casi siempre son
archivos de configuración que no conviene mover por accidente.

## 4. Cómo están acomodados los archivos: el árbol

En Windows, cada disco tiene una letra: `C:\`, `D:\`. En Linux no hay letras. Todo
empieza en **una sola carpeta principal llamada raíz**, que se escribe con una diagonal:
`/`. Dentro de la raíz hay carpetas, dentro de esas carpetas hay más carpetas, y así
sucesivamente. Por eso se dice que los archivos forman un **árbol**: la raíz arriba y
las ramas hacia abajo.

Las carpetas que hay dentro de la raíz son las mismas en todos los Linux, porque
siguen un estándar llamado FHS (Linux Foundation, 2015). Estas son las que vas a usar
en el curso:

```
/                      la raíz: aquí empieza todo
├── etc/               configuración del sistema y de cada servicio
├── home/              carpetas personales de los usuarios
│   └── usuario/       tu carpeta personal (con tu nombre de usuario)
├── root/              carpeta personal del administrador
├── var/
│   └── log/           registros: lo que hacen los servicios y sus errores
├── tmp/               archivos temporales, se borran al reiniciar
├── usr/
│   └── bin/           los programas (los comandos son archivos de aquí)
└── boot/              lo necesario para encender el sistema
```

| Carpeta | Qué guarda | ¿Puedes modificarla con tu usuario? |
|---|---|---|
| `/home/usuario` | Tus archivos | **Sí**, es tuya |
| `/tmp` | Archivos temporales de cualquier usuario | Sí, pero se borran al reiniciar |
| `/etc` | La configuración de todo el servidor | **No**: solo el administrador |
| `/var/log` | Los registros del sistema | No: puedes leer algunos, no modificarlos |
| `/usr/bin` | Los programas | No: puedes usarlos, no cambiarlos |
| `/root`, `/boot` | Archivos del administrador y del arranque | No |

Fíjate en el patrón: **lo único que es completamente tuyo es tu carpeta personal**.
Todo lo demás pertenece al sistema. Eso lo controlan los permisos, que verás en la
sección 7.

## 5. Rutas: la dirección de un archivo

Una **ruta** es la dirección completa de un archivo dentro del árbol: el camino de
carpetas que hay que recorrer para llegar a él. Las carpetas se separan con `/`. En
Windows se separan con `\`, y es fácil confundirlas al principio.

Por ejemplo, la ruta `/etc/hostname` se lee así:

```
/          empieza en la raíz
etc/       entra a la carpeta etc
hostname   ahí está el archivo llamado hostname
```

### Ruta absoluta y ruta relativa

Hay dos maneras de escribir una ruta. Piensa en cómo darías indicaciones para llegar a
un salón de tu escuela:

- **Absoluta:** "Desde la entrada principal, segundo piso, salón 204". Funciona sin
  importar dónde esté la persona. En Linux, una ruta absoluta **empieza con `/`**
  (desde la raíz): `/home/usuario/notas.txt`.
- **Relativa:** "El salón de al lado". Solo funciona si la persona ya está en el salón
  correcto. En Linux, una ruta relativa **no empieza con `/`** y parte de la carpeta en
  la que estás. Si ya estás dentro de `/home/usuario`, basta con escribir `notas.txt`.

### Tres abreviaturas

| Símbolo | Significa | Ejemplo |
|---|---|---|
| `~` | Tu carpeta personal | `~` es lo mismo que `/home/usuario` |
| `.` | La carpeta en la que estás ahora | `./notas.txt` es "notas.txt, aquí mismo" |
| `..` | La carpeta de arriba (la que contiene a la actual) | Si estás en `/home/usuario`, `..` es `/home` |

## 6. Usuarios: tú y el administrador

Como viste en [Usuarios](02-usuarios.md) y en
[El administrador, sudo y los paquetes](03-administrador-y-paquetes.md), cada persona
tiene su propia cuenta de usuario, los usuarios se juntan en **grupos**, y existe un
administrador, **`root`**, que puede hacer todo. Tú usas una cuenta normal y escribes `sudo` cuando necesitas
hacer algo como administrador. Los permisos de esta lectura son justo lo que define
**qué puede hacer cada cuenta con cada archivo**.

## 7. Permisos: quién puede hacer qué

Cada archivo y cada carpeta tiene un **dueño** (un usuario), pertenece a un **grupo**, y
guarda una lista de **permisos** que responde a dos preguntas: **¿quién?** y **¿qué
puede hacer?**

### ¿Quién? Tres tipos de persona

| Letra | Nombre | Quién es |
|---|---|---|
| `u` | Dueño (*user*) | El usuario al que pertenece el archivo |
| `g` | Grupo (*group*) | Los usuarios que forman parte del grupo del archivo |
| `o` | Otros (*others*) | Todos los demás usuarios del servidor |

### ¿Qué puede hacer? Tres permisos

| Letra | Permiso | En un archivo significa... | En una carpeta significa... |
|---|---|---|---|
| `r` | Lectura (*read*) | Ver lo que tiene escrito | Ver la lista de lo que hay dentro |
| `w` | Escritura (*write*) | Modificarlo | Crear, borrar o renombrar archivos dentro |
| `x` | Ejecución (*execute*) | Usarlo como programa | Poder entrar a ella |

Cada uno de los tres tipos de persona tiene su propio juego de `r`, `w` y `x`. Por
ejemplo, el dueño puede tener los tres permisos mientras los demás solo pueden leer
(Free Software Foundation, s.f.-b).

Con el archivero: el permiso `r` es poder **leer** la hoja, `w` es poder **escribir o
tachar** en ella, y `x` en una carpeta es tener la **llave** del cajón.

## 8. Cómo se leen los permisos en la pantalla

El comando `ls -l` (lo vas a usar en la práctica) muestra cada archivo con todos sus
datos en una línea. Así se ve el archivo `/etc/hostname`, que guarda el nombre del
servidor:

```
-rw-r--r-- 1 root root 9 oct 13 10:00 hostname
```

Cada parte de la línea significa algo (Free Software Foundation, s.f.-a):

```
-rw-r--r--   1   root    root    9      oct 13 10:00   hostname
│└───┬───┘       └─┬─┘   └─┬─┘   │      └─────┬────┘   └──┬───┘
│    │             │       │     │            │           nombre
│    │             │       │     │            fecha de la última modificación
│    │             │       │     tamaño, en bytes (letras)
│    │             │       grupo del archivo
│    │             dueño del archivo
│    los 9 caracteres de permisos
tipo: - es archivo, d es carpeta (directorio)
```

El número `1` que aparece después de los permisos casi nunca lo vas a necesitar, así
que puedes ignorarlo.

### Los 9 caracteres de permisos

Los 9 caracteres se leen **en tres grupos de tres**: primero el dueño, luego el grupo y
al final los otros. Cada grupo siempre tiene el mismo orden, `r` `w` `x`, y un guion
`-` significa **"este permiso no lo tiene"**.

```
 rw-       r--       r--
 └┬┘       └┬┘       └┬┘
dueño     grupo     otros
(root)    (root)    (todos los demás)
```

Así que `/etc/hostname` dice:

- **El dueño (`root`)**: `rw-` → puede leerlo y modificarlo.
- **El grupo (`root`)**: `r--` → solo puede leerlo.
- **Los otros (tú, por ejemplo)**: `r--` → solo puedes leerlo.

En resumen: **tú puedes ver el nombre del servidor, pero no cambiarlo**. Para cambiarlo
necesitarías `sudo`.

### Dos ejemplos más

El programa `ls` es un archivo dentro de `/usr/bin`:

```
-rwxr-xr-x 1 root root ... ls
```

`rwx` para el dueño y `r-x` para el grupo y los otros: todos lo pueden **ejecutar**
(usar como programa), pero solo `root` lo puede modificar.

La carpeta `/etc`:

```
drwxr-xr-x ... root root ... etc
```

La `d` del principio indica que es una **carpeta**. Los otros tienen `r-x`: puedes
**entrar** (`x`) y **ver la lista** de lo que hay (`r`), pero como no tienes `w`, **no
puedes crear ni borrar archivos ahí**. Lo vas a comprobar en la práctica.

## 9. Los permisos en números

En tutoriales y documentación vas a ver los permisos escritos con números, como `644` o
`755`. Es otra forma de escribir lo mismo: cada permiso vale un número y se suman
(Free Software Foundation, s.f.-b).

| Permiso | Vale |
|---|---|
| `r` | 4 |
| `w` | 2 |
| `x` | 1 |
| `-` | 0 |

Cada grupo de tres letras se convierte en un solo número. Por ejemplo, `rw-` es
4 + 2 + 0 = **6**, y `r--` es 4 + 0 + 0 = **4**. Por eso `rw-r--r--` se escribe **644**:

```
 rw-     r--     r--
4+2+0   4+0+0   4+0+0
  6       4       4      -->  644
```

Los más comunes:

| Número | Letras | Para qué se usa |
|---|---|---|
| `644` | `rw-r--r--` | Archivo normal: el dueño lo modifica, todos lo leen |
| `600` | `rw-------` | Archivo privado: solo el dueño lo lee y lo modifica |
| `755` | `rwxr-xr-x` | Programa o carpeta: todos pueden usarlo o entrar, solo el dueño lo cambia |
| `700` | `rwx------` | Carpeta privada: solo el dueño puede entrar |

Los permisos se cambian con el comando `chmod` (de *change mode*, "cambiar modo"), y el
dueño de un archivo se cambia con `chown` (*change owner*). En la práctica vas a usar
`chmod` con tus propios archivos.

## 10. Por qué importa para administrar redes

Los permisos son la primera línea de **seguridad** de un servidor, la S de FCAPS que
viste en la [Etapa 1](../../etapa-1-funciones-administracion/05-administracion-seguridad.md):

- Si cualquier usuario pudiera modificar `/etc`, cualquiera podría cambiar la
  configuración del servidor.
- Cada servicio que instales más adelante (un servidor web, un DNS) corre con **su
  propio usuario**, y solo puede tocar los archivos que le corresponden. Si alguien
  ataca ese servicio, los permisos limitan el daño.
- Cuando un servicio no arranca, una de las primeras cosas que se revisan es si tiene
  permiso de leer sus archivos de configuración.

## Resumen

```
archivo     información guardada con un nombre (una hoja del archivero)
carpeta     contenedor de archivos y carpetas (un cajón), o "directorio"
/           la raíz, la carpeta donde empieza todo
ruta        la dirección de un archivo: /etc/hostname
~           tu carpeta personal: /home/usuario
..          la carpeta de arriba
root        el administrador; sudo = hacer algo como administrador
u g o       dueño, grupo, otros
r w x       leer, escribir, ejecutar (o entrar, si es carpeta)
644         rw-r--r--  (r=4, w=2, x=1)
```

## Pruébalo en tu VM

Ahora vas a leer permisos reales en tu servidor y a **explicarlos con tus palabras**.
Antes de cada captura, responde las preguntas que se indican: tu profesor puede
pedirte que las expliques.

### Paso 1: Los permisos de un archivo, una carpeta y un programa

```bash
ls -l /etc/hostname
ls -ld /etc
ls -l /usr/bin/tree
```

- El primero es el archivo con el nombre del servidor.
- El segundo es la carpeta `/etc`. La opción `-d` hace que `ls` muestre los datos de la
  carpeta misma y no lo que tiene dentro.
- El tercero es el programa `tree` que instalaste en la
  [lectura de paquetes](03-administrador-y-paquetes.md#paso-3-instalar-tree).

> 📸 **Captura `NombreApellido-LeerPermisos.png`:** la salida de los tres comandos.
> Antes de guardarla, responde para **cada uno** de los tres:
> - Si es archivo o carpeta, y cómo lo supiste.
> - Quién es el dueño.
> - Qué puede hacer el dueño, qué puede hacer el grupo y qué pueden hacer los otros.

### Paso 2: Por qué no pudiste entrar a la otra carpeta

En la [práctica de usuarios](04-practica-usuarios.md#paso-2-cada-quien-en-su-carpeta) viste un
**Permission denied** al intentar entrar a la carpeta de `user-practica-01`. Ahora
puedes ver por qué:

```bash
ls -ld /home/*
```

El asterisco `*` significa **"todo lo que haya ahí"**: `/home/*` son todas las
carpetas que hay dentro de `/home`. Aparecen dos líneas, una por cada carpeta
personal, cada una con su dueño.

Lee los últimos tres caracteres de permisos (los de **otros**) de cada carpeta.

> 📸 **Captura `NombreApellido-PermisosHome.png`:** la salida de `ls -ld /home/*`.
> Antes de guardarla, explica con tus palabras por qué `user-practica-01` no pudo
> entrar a tu carpeta ni tú a la suya. Pista: tú eres "otros" para la carpeta de
> `user-practica-01`, y él es "otros" para la tuya.

## Referencias

Free Software Foundation. (s.f.-a). *GNU Coreutils: ls invocation*.
https://www.gnu.org/software/coreutils/manual/html_node/ls-invocation.html

Free Software Foundation. (s.f.-b). *GNU Coreutils: File permissions*.
https://www.gnu.org/software/coreutils/manual/html_node/File-permissions.html

Linux Foundation. (2015). *Filesystem Hierarchy Standard* (versión 3.0).
https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html
