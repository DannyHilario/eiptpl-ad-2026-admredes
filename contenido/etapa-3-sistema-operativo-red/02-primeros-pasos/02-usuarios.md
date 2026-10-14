# Usuarios: una computadora, muchas personas

> Primeros pasos 2 · ≈50 min · [← Primer inicio de sesión](01-primer-inicio.md) ·
> Siguiente: [El administrador, sudo y los paquetes](03-administrador-y-paquetes.md)

## Antes de empezar

Cuando encendiste tu servidor por primera vez, en el
[primer inicio de sesión](01-primer-inicio.md), lo primero que te pidió
fue esto:

```
admredes login:
```

Te pidió un **nombre de usuario** y una **contraseña** antes de dejarte hacer cualquier
cosa. Esta lectura explica por qué: qué es un usuario, por qué un mismo sistema
operativo puede tener muchos, y por qué eso es tan importante en un servidor.

Primero vas a leer: los ejemplos de comandos son para que veas cómo se usan, no hace
falta que los escribas mientras lees. Al final, en [Pruébalo en tu VM](#pruébalo-en-tu-vm),
vas a hacerlo tú en tu servidor.

## 1. Lo que ya conoces: una computadora, una persona

Piensa en tu celular. Lo usas tú y nadie más: tus fotos, tus mensajes, tus
aplicaciones, tu fondo de pantalla. Cuando lo desbloqueas, no te pregunta **quién**
eres, porque da por hecho que eres tú.

Con muchas laptops pasa lo mismo: tienen un dueño, y todo lo que hay en ellas es de
esa persona. Por eso es fácil pensar que **una computadora = una persona**.

Pero eso no es cierto. Un sistema operativo puede ser usado por **muchas personas**,
cada una con sus propios archivos y su propia configuración, **sin que una vea ni
toque lo de las demás**.

## 2. Un ejemplo que ya viste: las computadoras de la escuela

Piensa en las computadoras del laboratorio de tu escuela, o en una computadora de casa
que usa toda la familia. En Windows, al encenderla, a veces aparecen **varias cuentas**
en la esquina inferior izquierda de la pantalla de inicio: "Mamá", "Papá", "Alumno",
"Administrador". Cada persona elige la suya y escribe su contraseña.

Lo que pasa después es lo importante:

- Cada quien ve **su propio escritorio**, con su fondo de pantalla y sus íconos.
- Cada quien tiene **su propia carpeta de Documentos**. Lo que guarda una persona no
  aparece en los Documentos de otra.
- Si alguien borra un archivo de su carpeta, **no afecta** a los demás.
- Algunas cuentas pueden **instalar programas** y otras no.

Eso es justo lo que significa que un sistema operativo sea **multiusuario**: **un solo
sistema, varias personas, y cada una con su propio espacio y sus propios límites**.

## 3. En un servidor, al mismo tiempo

En la computadora de casa, las personas se turnan: primero entra una, cierra su sesión,
y luego entra otra. En un servidor pasa algo más: **varias personas pueden estar
conectadas al mismo tiempo**.

¿Cómo, si el servidor tiene un solo teclado y una sola pantalla? Porque la mayoría no
se sienta frente a él: se conecta **por la red**, desde su propia computadora, con
**SSH**, el servicio que activaste en el
[Paso 11](../01-instalacion-ubuntu-server/11-ssh.md) de la instalación.

```
  Laptop de Ana ──────┐
                      │   SSH (por la red)
  Laptop de Luis ─────┼────────────────────>  Servidor admredes
                      │                       ├── sesión de ana
  Laptop de Sofía ────┘                       ├── sesión de luis
                                              └── sesión de sofia
```

Los tres están dentro del mismo servidor, al mismo tiempo, y cada uno trabaja en su
propia **sesión**, con su propio usuario. Es exactamente lo que vamos a hacer en el
laboratorio: tus compañeros se van a poder conectar al servidor del profesor, cada uno
con su cuenta.

## 4. ¿Qué es una cuenta de usuario?

Una **cuenta de usuario** es el registro que el sistema guarda de cada persona que
puede usarlo. Cada cuenta tiene:

| Parte | Qué es | Ejemplo |
|---|---|---|
| **Nombre de usuario** | El nombre con el que inicias sesión. En Linux, en minúsculas y sin espacios | `ana` |
| **Contraseña** | Lo que demuestra que de verdad eres esa persona | (secreta) |
| **Carpeta personal** | Una carpeta solo para esa persona, donde guarda sus archivos | `/home/ana` |
| **Número de usuario (UID)** | Un número que el sistema usa internamente para identificar la cuenta | `1000` |
| **Grupos** | Los equipos a los que pertenece (lo verás en la sección 7) | `ana`, `sudo` |

El **UID** (de *user identifier*, "identificador de usuario") es cómo el sistema
reconoce realmente a cada usuario: a la computadora le es más fácil trabajar con
números que con nombres. En Ubuntu, las cuentas de personas empiezan en el número
**1000** (Canonical, s.f.). Tu cuenta, la primera que se creó, casi seguramente tiene
el UID 1000.

La cuenta que tú creaste en el
[Paso 9](../01-instalacion-ubuntu-server/09-perfil.md) de la instalación es una de
estas. Ahí elegiste tu nombre de usuario y tu contraseña, y el instalador creó tu
carpeta personal en `/home/`.

## 5. ¿Para qué separar a los usuarios?

Si todos usaran la misma cuenta, sería más sencillo, pero un desastre. Separar a los
usuarios sirve para cuatro cosas:

**1. Privacidad.** Cada quien tiene sus archivos en su carpeta, y los demás no pueden
leerlos si el dueño no lo permite. Lo que tú guardas en `/home/ana` no lo ve Luis.

**2. Protección contra errores.** Si alguien borra algo por accidente, solo afecta sus
propios archivos, no los de los demás ni los del sistema.

**3. Límites.** No todos necesitan poder hacer todo. Un alumno necesita trabajar con sus
archivos; no necesita poder apagar el servidor ni borrar la configuración. Cada cuenta
tiene solo los permisos que le hacen falta.

**4. Saber quién hizo qué.** Como cada persona entra con su propia cuenta, el sistema
puede **registrar** quién inició sesión, a qué hora y qué hizo. Si algo sale mal, se
puede averiguar qué cuenta lo provocó. Es la **contabilidad**, la A de FCAPS que viste
en la [Etapa 1](../../etapa-1-funciones-administracion/04-administracion-contabilidad.md).

Por eso **nunca se comparten cuentas**, ni contraseñas. Si dos personas usan la misma
cuenta, el sistema no tiene forma de saber cuál de las dos hizo algo.

## 6. Usuarios que no son personas

Esto suele sorprender: en tu servidor ya existen **decenas de usuarios**, aunque solo
tú hayas creado uno. La mayoría **no son personas**: son cuentas que usa el propio
sistema, y cada programa importante tiene la suya.

¿Por qué un programa necesitaría una cuenta? Por la misma razón que una persona: para
tener **límites**. Más adelante vas a instalar servicios como un servidor web. Ese
servidor web va a funcionar con **su propia cuenta**, que solo puede tocar los archivos
de la página web. Si un atacante lograra engañarlo, solo tendría el poder de esa
cuenta, no el de todo el servidor.

Todas las cuentas, de personas y de programas, están anotadas en un archivo de texto:
`/etc/passwd` (Canonical, s.f.). Cada cuenta ocupa una línea, y sus datos están
separados por dos puntos (`:`). Así se vería la línea de una cuenta llamada `ana`. No
hace falta entender todos los datos; fíjate solo en estos cuatro (Kerrisk, s.f.):

```
ana : x : 1000 : 1000 : Ana : /home/ana : /bin/bash
└┬┘       └┬─┘                └───┬───┘   └───┬───┘
 │         │                      │           programa que se abre al entrar
 │         │                      carpeta personal
 │         número de usuario (UID)
 nombre de usuario
```

- Las cuentas de **personas** tienen UID de 1000 en adelante y, al final, `/bin/bash`:
  al iniciar sesión se abre una terminal.
- Las cuentas de **programas** tienen UID menor a 1000 y, al final, `nologin`: **nadie
  puede iniciar sesión con ellas**. Existen solo para que los programas trabajen.

**La contraseña no está en este archivo** (por eso aparece solo una `x`). Se guarda
cifrada en otro archivo que solo el administrador puede leer.

## 7. Grupos: usuarios que comparten permisos

Un **grupo** es un conjunto de usuarios con un nombre. Sirve para darle permisos a
varias personas **a la vez**, en lugar de una por una.

Ejemplo: en el servidor de la escuela hay una carpeta con los ejercicios de la clase.
En lugar de dar permiso a 30 alumnos uno por uno, se crea un grupo `alumnos`, se mete
ahí a los 30, y se le da permiso **al grupo**. Si llega un alumno nuevo, basta con
agregarlo al grupo.

```
grupo: alumnos             grupo: profesores
├── ana                    ├── profe
├── luis                   └── director
└── sofia
```

Algunas cosas que conviene saber:

- **Un usuario puede estar en varios grupos** al mismo tiempo.
- Al crear un usuario, Ubuntu también crea **un grupo con su mismo nombre**, del que
  solo él forma parte. Por eso, en el ejemplo de `/etc/passwd`, `ana` tiene UID 1000 y
  también un grupo con el número 1000.
- Hay un grupo especial llamado **`sudo`**: los usuarios que están en él pueden
  hacer tareas de administrador. Tu cuenta está en ese grupo, porque fue la primera
  que se creó (Canonical, s.f.). Eso lo vas a entender en la
  [siguiente lectura](03-administrador-y-paquetes.md).

## 8. Comandos para saber quién eres

En la práctica vas a usar estos comandos. Por ahora, mira qué hace cada uno:

| Comando | Viene de | Qué muestra |
|---|---|---|
| `whoami` | *who am I?*, "¿quién soy?" | Tu nombre de usuario |
| `id` | *identity*, "identidad" | Tu UID y todos los grupos a los que perteneces |
| `who` | *who?*, "¿quién?" | Quién está conectado al servidor en este momento |

Por ejemplo, si `ana` escribe `id`, ve algo parecido a esto (la lista real de grupos
es más larga):

```
uid=1000(ana) gid=1000(ana) groups=1000(ana),27(sudo)
```

Se lee así: su número de usuario es 1000, su grupo principal es `ana`, y además
pertenece al grupo `sudo`.

## Resumen

```
multiusuario     un sistema operativo usado por varias personas, cada una
                 con su espacio y sus límites (incluso al mismo tiempo)
cuenta           nombre de usuario + contraseña + carpeta personal + UID
UID              número con el que el sistema identifica a cada usuario
                 (personas: desde 1000)
/home/nombre     la carpeta personal de cada usuario
cuentas de       usuarios que usan los programas, no las personas;
  sistema        nadie puede iniciar sesión con ellas
/etc/passwd      el archivo donde están anotadas todas las cuentas
grupo            conjunto de usuarios que comparten permisos
grupo sudo       los usuarios que pueden hacer tareas de administrador
whoami / id      quién soy / mi UID y mis grupos
who              quién está conectado ahora
```

## Pruébalo en tu VM

Ahora vas a comprobar en tu propio servidor lo que acabas de leer: vas a ver tu cuenta,
tus grupos y tu carpeta personal.

Guarda las capturas en tu carpeta `capturas-admredes`, como se explica en el
[primer inicio de sesión](01-primer-inicio.md#cómo-tomar-y-guardar-las-capturas).

### Paso 1: ¿Quién soy?

Inicia sesión en tu VM y escribe estos tres comandos, uno por uno, presionando Enter
después de cada uno:

```bash
whoami
id
who
```

- `whoami` muestra tu nombre de usuario.
- `id` muestra tu número de usuario (`uid=1000`) y, después de `groups=`, todos tus
  grupos. Busca `sudo` en esa lista: por eso tu cuenta puede hacer tareas de
  administrador.
- `who` muestra quién está conectado ahora mismo. Por ahora solo apareces tú.

> 📸 **Captura `NombreApellido-MiUsuario.png`:** la salida de `id`, donde se vea el
> grupo `sudo`.

### Paso 2: Tu carpeta personal

```bash
pwd
ls /home
```

- `pwd` muestra en qué carpeta estás: `/home/` seguido de tu nombre de usuario. Es tu
  **carpeta personal**, la que el sistema creó para ti durante la instalación.
- `ls /home` muestra todas las carpetas personales del servidor. Por ahora hay **una
  sola**, la tuya, porque eres el único usuario que es una persona. En la
  [práctica de usuarios](04-practica-usuarios.md) vas a crear otro y vas a ver aparecer
  una segunda carpeta aquí.

> 📸 **Captura `NombreApellido-MiCarpeta.png`:** la salida de `pwd` y de `ls /home`.

## Referencias

Canonical Ltd. (s.f.). *User management*. Ubuntu Server documentation.
https://documentation.ubuntu.com/server/how-to/security/user-management/

Kerrisk, M. (s.f.). *passwd(5): Linux manual page*. The Linux man-pages project.
https://man7.org/linux/man-pages/man5/passwd.5.html
