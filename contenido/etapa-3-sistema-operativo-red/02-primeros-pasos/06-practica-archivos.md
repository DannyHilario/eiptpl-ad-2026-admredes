# Práctica: archivos y permisos

> Primeros pasos 6 · ≈50 min · [← Archivos, carpetas y permisos](05-archivos-y-permisos.md)

## Qué vas a hacer

En esta práctica vas a usar en tu VM lo que leíste en
[Archivos, carpetas y permisos](05-archivos-y-permisos.md): moverte entre carpetas,
leer archivos del sistema, crear tus propios archivos y revisar sus permisos. Es la
última parte de los primeros pasos: al final revisarás que tengas **todas tus
capturas**, las de las lecturas y las de esta práctica.

Antes de empezar:

1. Enciende tu VM en VirtualBox e **inicia sesión** con tu usuario y contraseña, igual
   que en el [primer inicio de sesión](01-primer-inicio.md).
2. En esta práctica **no vas a usar `sudo`** en ningún momento. Si en algún paso la
   pantalla te pide permisos de administrador, es parte del ejercicio: lee la
   explicación de ese paso.

## Cómo seguir las instrucciones

Los comandos aparecen en recuadros grises como este:

```bash
pwd
```

- **Escribe el comando tal cual**, con las mismas mayúsculas, minúsculas y espacios, y
  presiona **Enter** para ejecutarlo.
- **Escríbelo tú**: no lo copies y pegues. Así te acostumbras a los comandos.
- Lo que aparece después de un `#` es un **comentario**: una nota para ti que explica
  el comando. **No lo escribas**.
- Si aparece `command not found` (o "no se encontró la orden"), revisa que lo hayas
  escrito exactamente igual. Casi siempre es una letra de más o de menos.
- Debajo de cada comando se explica **qué deberías ver**. Compáralo con tu pantalla
  antes de pasar al siguiente.

## Paso 1: ¿Dónde estoy?

Al iniciar sesión aparece una línea parecida a esta, que se llama **prompt**:

```
usuario@admredes:~$
```

El prompt aparece cada vez que la terminal está lista para recibir un comando:

| Parte | Qué significa |
|---|---|
| `usuario` | Tu nombre de usuario |
| `admredes` | El nombre del servidor |
| `~` | La carpeta en la que estás: `~` significa tu carpeta personal |
| `$` | Estás como usuario normal, no como administrador |

Ahora pregúntale a Linux en qué carpeta estás. El comando `pwd` viene de *print working
directory*, "muestra la carpeta de trabajo":

```bash
pwd
```

Debe aparecer algo como `/home/usuario`, con tu nombre de usuario. Esa es la **ruta
absoluta** de tu carpeta personal: la misma carpeta que el prompt abrevia como `~`.

Ahora confirma quién eres, con los comandos que viste en la lectura de
[Usuarios](02-usuarios.md#8-comandos-para-saber-quién-eres):

```bash
whoami
id
```

`whoami` muestra tu nombre de usuario. `id` muestra tu número de usuario (`uid=1000`) y
la lista de grupos a los que perteneces: busca `sudo` en esa lista.

## Paso 2: ¿Qué hay aquí?

El comando `ls` (de *list*, "listar") muestra lo que hay dentro de la carpeta en la que
estás:

```bash
ls
```

Lo más probable es que **no aparezca nada**: tu carpeta parece vacía. Ahora agrégale
**opciones**. Las opciones son letras que van después de un guion y cambian lo que hace
el comando. `-l` muestra los detalles de cada archivo y `-a` muestra **todos**,
incluyendo los ocultos:

```bash
ls -la
```

Ahora sí aparecen varias líneas. Los archivos que empiezan con punto, como `.bashrc` y
`.profile`, son archivos ocultos de configuración que se crearon con tu usuario. También
aparecen `.` (esta carpeta) y `..` (la carpeta de arriba).

Fíjate en el **dueño** de esos archivos (la tercera columna): es tu usuario. Todo lo que
hay en tu carpeta personal es tuyo.

## Paso 3: Moverte entre carpetas

El comando `cd` (de *change directory*, "cambiar de carpeta") te lleva a otra carpeta.
Después de `cd` se escribe un **espacio** y la ruta a donde quieres ir:

```bash
cd /etc
```

Fíjate que el prompt cambió: ahora dice `usuario@admredes:/etc$`. Confírmalo y mira qué
hay:

```bash
pwd
ls
```

`pwd` muestra `/etc`, y `ls` muestra muchos nombres: son los archivos de configuración
del servidor. Ahora sube un nivel con `..`, que significa "la carpeta de arriba":

```bash
cd ..
pwd
```

Estás en `/`, la raíz. Regresa a tu carpeta personal con `~`:

```bash
cd ~
pwd
```

De nuevo estás en `/home/usuario`.

**Un atajo que vas a usar todo el tiempo: la tecla Tab.** Escribe `cd /et` **sin
presionar Enter**, y luego presiona **Tab**: la terminal completa el nombre y queda
`cd /etc/`. Si Tab no completa nada, es que no existe ninguna carpeta que empiece así.
Presiona Enter, y luego regresa con `cd ~`.

**Otro atajo: la flecha ↑** del teclado muestra el último comando que escribiste. Si la
presionas varias veces, recorres los anteriores.

## Paso 4: Leer archivos del sistema

El comando `cat` muestra en pantalla todo lo que tiene escrito un archivo de texto. Lee
el archivo que guarda el nombre del servidor:

```bash
cat /etc/hostname
```

Debe aparecer `admredes`, el nombre que le pusiste durante la instalación. Ese nombre,
que ves en el prompt, sale de este archivo.

Ahora lee el archivo que guarda la versión del sistema:

```bash
cat /etc/os-release
```

Aparecen varias líneas. Busca la que empieza con `PRETTY_NAME`: ahí dice qué versión
de Ubuntu tienes instalada (freedesktop.org, s.f.).

## Paso 5: Crear tu carpeta de prácticas

El comando `mkdir` (de *make directory*, "crear carpeta") crea una carpeta nueva.
Primero asegúrate de estar en tu carpeta personal, y luego crea una que se llame
`practica` (sin acento, para evitar problemas con el teclado):

```bash
cd ~
mkdir practica
ls
```

Ahora `ls` sí muestra algo: la carpeta `practica`. Entra en ella:

```bash
cd practica
pwd
```

`pwd` debe mostrar `/home/usuario/practica`.

## Paso 6: Crear un archivo de texto con nano

`nano` es un **editor de texto** que funciona dentro de la terminal, parecido al Bloc de
notas de Windows pero sin mouse. Después de `nano` va el nombre del archivo. Si el
archivo no existe, nano lo crea:

```bash
nano servidor.txt
```

La pantalla cambia: arriba dice el nombre del archivo, en medio está el espacio para
escribir y abajo hay una lista de atajos. En esa lista, el símbolo `^` significa la
tecla **Ctrl**. Por ejemplo, `^X` quiere decir **Ctrl + X**.

1. Escribe estas tres líneas, con tus datos. Para cambiar de línea presiona Enter, y
   para moverte usa las flechas, porque el mouse no funciona:

   ```
   Servidor: admredes
   Sistema: Ubuntu Server
   Alumno: tu nombre
   ```

2. **Guarda** con **Ctrl + O**. Abajo aparece el nombre del archivo y te pide
   confirmarlo: presiona **Enter**.
3. **Sal** de nano con **Ctrl + X**. Regresas al prompt.

Comprueba que el archivo existe y que tiene lo que escribiste:

```bash
ls
cat servidor.txt
```

> 📸 **Captura `NombreApellido-ServidorTxt.png`:** la salida de `cat servidor.txt`, con
> tus datos.

## Paso 7: Copiar, renombrar y borrar

**Copiar.** El comando `cp` (de *copy*) necesita dos cosas: primero el archivo que
quieres copiar y después el nombre de la copia:

```bash
cp servidor.txt copia.txt
ls
```

Ahora hay dos archivos: `servidor.txt` y `copia.txt`, con el mismo contenido.

**Renombrar.** El comando `mv` (de *move*, "mover") sirve para mover un archivo a otra
carpeta, y también para cambiarle el nombre. Primero va el nombre actual y después el
nuevo:

```bash
mv copia.txt respaldo.txt
ls
```

`copia.txt` ya no existe: ahora se llama `respaldo.txt`.

**Copiar un archivo del sistema.** También puedes copiar archivos de otras carpetas.
Copia el archivo del nombre del servidor a tu carpeta de prácticas. El `.` del final
significa "aquí, en la carpeta donde estoy":

```bash
cp /etc/hostname .
ls
```

Tienes una copia de `hostname` en tu carpeta. El original de `/etc` sigue igual: solo
leíste el original, no lo modificaste. Por eso tu usuario puede hacer la copia.

**Borrar.** El comando `rm` (de *remove*, "quitar") borra un archivo:

```bash
rm respaldo.txt
ls
```

⚠️ **En Linux no hay papelera de reciclaje.** Lo que borras con `rm` desaparece para
siempre, sin preguntarte "¿estás seguro?". Antes de borrar algo, revisa con `ls` qué
hay y con `pwd` en qué carpeta estás.

## Paso 8: Revisar los permisos de tus archivos

Ahora vas a leer permisos como en la sección 8 de
[la lectura](05-archivos-y-permisos.md#8-cómo-se-leen-los-permisos-en-la-pantalla):

```bash
ls -l
```

Verás algo parecido a esto (la fecha y el tamaño cambian):

```
-rw-r--r-- 1 usuario usuario  9 oct 13 10:20 hostname
-rw-rw-r-- 1 usuario usuario 57 oct 13 10:15 servidor.txt
```

Fíjate en tres cosas:

1. **El dueño eres tú**, en los dos archivos, aunque `hostname` lo copiaste de `/etc`.
   Cuando copias un archivo, la copia es tuya.
2. **Los permisos pueden ser un poco distintos** a los de este ejemplo según cómo se
   creó cada archivo. Léelos en grupos de tres, como en la lectura: dueño, grupo y
   otros.
3. **Tú tienes `rw`** en los dos: los puedes leer y modificar.

Ahora compáralos con el original:

```bash
ls -l /etc/hostname
```

```
-rw-r--r-- 1 root root 9 oct 13 10:00 /etc/hostname
```

El original es de `root`, y para "los otros", que te incluye a ti, solo tiene `r--`.

## Paso 9: Comprobar un permiso negado

Intenta crear un archivo dentro de `/etc`. El comando `touch` crea un archivo vacío con
el nombre que le indiques:

```bash
touch /etc/prueba.txt
```

Aparece un error parecido a este (puede salir en inglés o en español):

```
touch: cannot touch '/etc/prueba.txt': Permission denied
```

*Permission denied* significa **"permiso denegado"**. No es una falla: es el sistema
funcionando como debe. ¿Por qué pasó? Revisa los permisos de la carpeta `/etc`. La
opción `-d` hace que `ls` muestre los datos de la carpeta misma, no lo que tiene
dentro:

```bash
ls -ld /etc
```

```
drwxr-xr-x ... root root ... /etc
```

Para "los otros" dice `r-x`: puedes entrar y ver la lista de archivos, pero **no tienes
`w`**, que es el permiso para crear archivos dentro de una carpeta. Solo `root` lo
tiene. Esto protege la configuración del servidor de cambios no autorizados.

Ahora haz lo mismo en tu carpeta, donde sí tienes permiso de escritura:

```bash
touch prueba.txt
ls
```

Esta vez funciona sin ningún error.

> 📸 **Captura `NombreApellido-PermisoNegado.png`:** el error de
> `touch /etc/prueba.txt` y, debajo, el `touch prueba.txt` en tu carpeta funcionando.

## Paso 10: Cambiar permisos con chmod

El comando `chmod` cambia los permisos de un archivo que te pertenece. Después de
`chmod` va el permiso en número (lo viste en la sección 9 de la lectura) y luego el
nombre del archivo (Free Software Foundation, s.f.).

Haz que `servidor.txt` sea **privado**, es decir, que solo tú lo puedas leer y
modificar. Eso es `600` (`rw-------`):

```bash
chmod 600 servidor.txt
ls -l servidor.txt
```

Ahora los permisos dicen `-rw-------`: el grupo y los otros ya no tienen ningún permiso.

> 📸 **Captura `NombreApellido-Chmod600.png`:** la salida de `ls -l servidor.txt` con
> los permisos `-rw-------`.

Regrésalo a la forma normal, `644` (`rw-r--r--`): tú lo modificas y todos lo pueden
leer:

```bash
chmod 644 servidor.txt
ls -l servidor.txt
```

Ahora dice `-rw-r--r--`.

## Paso 11: Revisar el resultado

Regresa a tu carpeta personal y revisa todo lo que hiciste:

```bash
cd ~
tree practica
ls -l practica
```

`tree` es el programa que instalaste en la
[lectura de paquetes](03-administrador-y-paquetes.md#paso-3-instalar-tree). Debe
dibujar algo así:

```
practica
├── hostname
├── prueba.txt
└── servidor.txt

1 directory, 3 files
```

(`1 directory, 3 files` significa "1 carpeta, 3 archivos"). En la salida de `ls -l`,
`servidor.txt` debe tener los permisos `-rw-r--r--`.

> 📸 **Captura `NombreApellido-ResultadoFinal.png`:** la salida de `tree practica` y de
> `ls -l practica`.

**No borres la carpeta `practica`**: la vas a seguir usando en las próximas prácticas.

## Comandos que usaste

```
pwd                  muestra en qué carpeta estás
ls                   lista lo que hay en la carpeta
ls -la               lista todo, con detalles y archivos ocultos
ls -l                lista con detalles: permisos, dueño, tamaño, fecha
ls -ld <carpeta>     detalles de la carpeta misma
cd <ruta>            entra a una carpeta
cd ..                sube a la carpeta de arriba
cd ~                 regresa a tu carpeta personal
cat <archivo>        muestra el contenido de un archivo de texto
mkdir <nombre>       crea una carpeta
nano <archivo>       abre o crea un archivo para editarlo
                       Ctrl+O guardar · Ctrl+X salir
cp <origen> <copia>  copia un archivo
mv <actual> <nuevo>  renombra o mueve un archivo
rm <archivo>         borra un archivo (no hay papelera)
touch <archivo>      crea un archivo vacío
chmod 644 <archivo>  cambia los permisos de un archivo tuyo
Tab                  completa nombres de archivos y carpetas
Flecha ↑             recupera los comandos anteriores
```

## Revisa tus capturas

Al terminar los primeros pasos, en tu carpeta `capturas-admredes` debes tener estas
**14 capturas**, cada una con tu nombre en lugar de `NombreApellido`:

| Archivo | Qué debe verse | Dónde se hace |
|---|---|---|
| `NombreApellido-PrimerInicio.png` | `hostnamectl` | [Primer inicio de sesión](01-primer-inicio.md#el-nombre-y-la-versión-del-servidor) |
| `NombreApellido-MiUsuario.png` | `id` con el grupo `sudo` | [Usuarios](02-usuarios.md#paso-1-quién-soy) |
| `NombreApellido-MiCarpeta.png` | `pwd` y `ls /home` | [Usuarios](02-usuarios.md#paso-2-tu-carpeta-personal) |
| `NombreApellido-AptSinSudo.png` | Error de `apt update` sin `sudo` | [Administrador y paquetes](03-administrador-y-paquetes.md#paso-1-actualizar-la-lista-sin-sudo-y-con-sudo) |
| `NombreApellido-InstalarTree.png` | Instalación de `tree`, `which tree` y `tree /etc/apt` | [Administrador y paquetes](03-administrador-y-paquetes.md#paso-3-instalar-tree) |
| `NombreApellido-NuevoUsuario.png` | `ls /home` e `id user-practica-01` | [Práctica de usuarios](04-practica-usuarios.md#paso-1-crear-un-usuario-nuevo) |
| `NombreApellido-CambioDeUsuario.png` | Sesión como `user-practica-01`: `whoami`, `pwd` y el error en tu carpeta | [Práctica de usuarios](04-practica-usuarios.md#paso-3-entrar-como-el-otro-usuario) |
| `NombreApellido-SudoNegado.png` | `sudo` negándose a `user-practica-01` | [Práctica de usuarios](04-practica-usuarios.md#paso-4-un-usuario-que-no-es-administrador) |
| `NombreApellido-LeerPermisos.png` | Permisos de `/etc/hostname`, `/etc` y `tree` | [Archivos y permisos](05-archivos-y-permisos.md#paso-1-los-permisos-de-un-archivo-una-carpeta-y-un-programa) |
| `NombreApellido-PermisosHome.png` | `ls -ld /home/*` | [Archivos y permisos](05-archivos-y-permisos.md#paso-2-por-qué-no-pudiste-entrar-a-la-otra-carpeta) |
| `NombreApellido-ServidorTxt.png` | `cat servidor.txt` | [Paso 6](#paso-6-crear-un-archivo-de-texto-con-nano) |
| `NombreApellido-PermisoNegado.png` | `touch` negado en `/etc` y permitido en tu carpeta | [Paso 9](#paso-9-comprobar-un-permiso-negado) |
| `NombreApellido-Chmod600.png` | `servidor.txt` con `-rw-------` | [Paso 10](#paso-10-cambiar-permisos-con-chmod) |
| `NombreApellido-ResultadoFinal.png` | `tree practica` y `ls -l practica` | [Paso 11](#paso-11-revisar-el-resultado) |

Revisa que **en todas se vea el prompt con tu nombre de usuario** (en
`CambioDeUsuario` y `SudoNegado`, con `user-practica-01`). **Guárdalas**: tu profesor
te indicará cómo entregarlas.

## Referencias

Free Software Foundation. (s.f.). *GNU Coreutils manual*.
https://www.gnu.org/software/coreutils/manual/coreutils.html

freedesktop.org. (s.f.). *os-release: Operating system identification*.
https://www.freedesktop.org/software/systemd/man/latest/os-release.html
