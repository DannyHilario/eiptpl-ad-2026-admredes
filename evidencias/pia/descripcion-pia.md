# PIA: El servidor de una pequeña empresa

En **equipo**, van a montar el servidor de una **empresa ficticia** que ustedes
inventan. Es el cierre del semestre: en una sola práctica usan lo que vieron en las
cuatro etapas.

| Lo que van a hacer | De qué etapa viene |
|---|---|
| Crear el **sitio web** de la empresa con IA generativa | Etapa 2 (WWW) |
| Subirlo al servidor por **SFTP** y publicarlo con nginx | Etapa 2 (FTP y SFTP, WWW) y Etapa 3 (servidor y permisos) |
| Proteger el servidor con un **firewall** | Etapa 1 (administración de **seguridad**) |
| Revisar quién visitó el sitio en el **registro** (*log*) | Etapa 1 (administración de **fallas** y **contabilidad**) |
| Sacar un **respaldo** del sitio | Etapa 1 (administración de **configuración**) |
| Hacer el **plan de direccionamiento** de la empresa | Etapa 4 (direccionamiento IP y subneteo) |

Igual que en la Evidencia 3, es una práctica **paso a paso**: cada paso es **un solo
comando**, con lo que significa y lo que deben ver. No tienen que inventar nada
técnico; tienen que **seguir los pasos con cuidado, entender qué hace cada uno y
explicarlo** en su reporte técnico.

---

## Cómo se organiza el equipo

**Todo el PIA lo hace un solo integrante, en su VM.** No tienen que ponerse de
acuerdo para trabajar a la misma hora ni conectarse a la VM de nadie: el integrante
que tenga su VM funcionando hace las partes 1 a 5 y arma el reporte.

Ese integrante **no** trabaja con su usuario personal, sino con un usuario del equipo
llamado **`equipo-N`**, donde **N** es su **número de equipo** (el que les asignó su
profesor). *Ejemplo:* el equipo 3 usa **`equipo-3`**. Ese usuario aparece en el
prompt de todas las capturas y en el pie de página del sitio: es lo que identifica el
trabajo de su equipo.

**¿Ya habían avanzado?** No repitan nada. Si uno o varios de ustedes ya hicieron
pasos con su usuario personal (por ejemplo `dghm-1410751`), **eso sigue valiendo**:
el integrante que va más adelantado termina en su VM con ese mismo usuario, se salta
la sección 4 de *Antes de empezar* y, donde diga `equipo-3`, escribe su usuario. Solo
se entrega **un** juego de capturas, el suyo. El usuario `equipo-N` es para los equipos
que todavía no empiezan.

⚠️ **En los comandos aparece `equipo-3` y `Tacos El Servidor`. Cada vez que los
vean, escriban el usuario de SU equipo y el nombre de SU empresa.**

---

## Antes de empezar

### 1. Inventen su empresa

Elijan un **nombre** para su empresa ficticia (por ejemplo, *Tacos El Servidor* o
*Refaccionaria Bits*).

### 2. Cómo tomar las capturas

Igual que en la Evidencia 3: **Ver → Tomar captura de pantalla...** en la ventana de
la VM, o **Windows + Shift + S** y seleccionan solo la ventana. Si trabajan por SSH,
capturen la ventana de Git Bash. Antes de cada captura pueden escribir `clear` para
limpiar la pantalla.

### 3. Enciendan su servidor

Enciendan la VM (debe seguir en modo **puente**, como la dejaron en la Etapa 3) e
inicien sesión con su **usuario de siempre**, el que crearon al instalar Ubuntu, que ya
es administrador.

### 4. Creen el usuario del equipo

*(Si ya habían avanzado con un usuario personal, sáltense esta sección.)*

Estos pasos se hacen **una sola vez**, con su usuario de siempre. Son los mismos de la
sección *"Crear tu usuario con tus iniciales"* de la
[Evidencia 4](../etapa-4/descripcion-evidencia-4.md), pero con el nombre `equipo-N`.
En Ubuntu un nombre de usuario debe empezar con una letra y puede llevar guiones y
números, así que `equipo-3` es válido (Debian, s.f.).

#### Crear el usuario

```bash
sudo adduser equipo-3
```

```
sudo   adduser   equipo-3
└┬─┘   └──┬──┘   └──┬───┘
 │        │         el nombre de la cuenta nueva: equipo- y su número de equipo
 │        "add user": agregar un usuario
 como administrador
```

**Qué deben hacer:** el comando hace varias preguntas, unas en inglés y otras en
español:

1. Si `sudo` les pide su contraseña de siempre, escríbanla.
2. **`New password:`** la contraseña del usuario del equipo. **No aparece nada**
   mientras escriben. Elijan una que recuerden: la van a usar para subir el sitio.
   **No la escriban en el reporte.**
3. **`Retype new password:`** escríbanla otra vez.
4. **`Nombre completo []:`** escriban `Equipo 3` (con su número).
5. **`Número de habitación []:`**, **`Teléfono del trabajo []:`**, **`Teléfono de
   casa []:`** y **`Otro []:`**: presionen **Enter** en cada uno.
6. **`Is the information correct? [Y/n]`:** presionen **Enter**.

Si dice que el usuario **ya existe**, es que ya lo habían creado antes: sigan con el
paso siguiente.

#### Convertirlo en administrador

```bash
sudo usermod -aG sudo equipo-3
```

```
sudo   usermod   -aG    sudo   equipo-3
       └──┬──┘   └┬─┘   └┬─┘   └──┬───┘
          │       │      │        a qué usuario
          │       │      al grupo "sudo": los administradores
          │       -a: agregar, -G: a un grupo
          "user modify": modificar un usuario
```

Lo necesita porque en la práctica van a usar `sudo` con este usuario: en Ubuntu, solo
los usuarios del grupo `sudo` pueden usarlo (Canonical Ltd., s.f.-c).

**Qué deben ver:** nada. Si regresa el prompt sin mensajes, salió bien.

#### Comprobar sus grupos

```bash
groups equipo-3
```

**Qué deben ver:** una línea como `equipo-3 : equipo-3 sudo users`. Lo importante es
que en la lista aparezca **`sudo`**.

#### Entrar con el usuario del equipo

```bash
su - equipo-3
```

```
su   -   equipo-3
└┬┘  │   └──┬───┘
 │   │      el usuario del equipo
 │   empieza una sesión nueva, en la carpeta de ese usuario
 "substitute user": cambiar de usuario
```

Escriban la contraseña **del usuario del equipo** (no aparece nada mientras la
escriben).

**Qué deben ver:** el prompt cambia y ahora empieza con `equipo-3@...:~$`. **Todo el
resto de la práctica se hace con este usuario**, y debe verse en el prompt de las
capturas. Si cierran la VM y regresan otro día, vuelvan a entrar con su usuario de
siempre y repitan solo este `su -`.

---

## Parte 1: Preparar el servidor

### Paso 1: Descargar la lista de actualizaciones

```bash
sudo apt update
```

Ya lo conocen de la Evidencia 3: descarga la **lista** de paquetes más reciente, sin
instalar nada todavía.

**Qué deben ver:** `sudo` les pide su contraseña (un asterisco por letra), varias
líneas `Get:` o `Hit:` (o `Obj:` y `Des:` en español) y al final el prompt. **No debe aparecer ninguna línea que
empiece con `Error:`.**

### Paso 2: Instalar los programas de la práctica

```bash
sudo apt install -y nginx ufw ipcalc
```

```
sudo   apt   install   -y   nginx   ufw   ipcalc
                       │    └─┬─┘   └┬┘   └─┬──┘
                       │      │      │      calculadora de redes (Etapa 4)
                       │      │      firewall de Ubuntu
                       │      servidor web
                       responde "sí" automáticamente
```

`apt install` acepta **varios paquetes a la vez**, separados por espacios. Si alguno
ya estaba instalado (nginx, de la Evidencia 3), apt solo avisa que ya está en su
versión más reciente y sigue con los demás.

**Qué deben ver:** líneas de descarga e instalación, y al final el prompt, sin líneas que
empiecen con `Error:`.

---

## Parte 2: El sitio web de la empresa

En la Evidencia 3 su página era una sola línea. Ahora van a montar un **sitio web
completo**, como el de una empresa de verdad, y lo van a hacer como se hace en la
práctica:

1. **Crean** el sitio en su **computadora**, con ayuda de una **IA generativa**.
2. Lo **suben** al servidor por **SFTP**, el servicio que instalaron en la Evidencia 3,
   desde la terminal o con un programa visual como FileZilla.
3. **nginx** lo entrega a quien lo visite.

```
  Su computadora                         Su servidor (VM)
 ┌────────────────────┐    SFTP        ┌──────────────────────────┐
 │  index.html        │ ─────────────> │  /var/www/html/index.html │
 │  (lo hizo la IA)   │   puerto 22    │            │              │
 └────────────────────┘                │          nginx            │
                                       └────────────┼─────────────┘
                                                    │ HTTP, puerto 80
                                                    v
                                         navegador de quien visita
```

El sitio es **estático** y **desconectado**: un solo archivo con todo adentro, que no
descarga nada de internet. Así funciona aunque el servidor no tenga salida a internet,
y ven exactamente qué se está publicando.

En esta parte van a ir y venir entre **dos ventanas**. Al inicio de cada paso dice en
cuál se escribe:

- **[Computadora]**: Git Bash en su computadora (en Mac, Terminal).
- **[Servidor]**: su VM, en la ventana de VirtualBox o por SSH, con el usuario del
  equipo (`equipo-3`).

### Paso 3: Pedirle el sitio a la IA

Este paso **no es un comando**. Abran una IA generativa en el navegador (ChatGPT,
Gemini, Copilot, Claude o la que tengan) y **copien este texto tal cual**, cambiando
solo el **nombre de la empresa** y el **usuario del equipo**:

```
Genera una página web para una empresa ficticia llamada "Tacos El Servidor".
Reglas:
- Todo en un solo archivo index.html, con el CSS dentro de una etiqueta <style>.
- Sin JavaScript.
- Sin imágenes, sin íconos, sin SVG, sin fuentes de Google y sin nada que se
  descargue de internet: la palabra "http" no debe aparecer en el código.
- En español, con tres secciones: Inicio, Nuestros servicios y Contacto.
- Los datos de contacto deben ser inventados y decir que son de ejemplo.
- En el pie de página escribe exactamente: Sitio administrado por equipo-3
Responde solo con el código, sin explicaciones.
```

**Qué deben ver:** la IA responde con un bloque de código que empieza con
`<!DOCTYPE html>` y termina con `</html>`. **Cópienlo completo**, con el botón
**Copiar** que aparece en la esquina del bloque.

**¿Por qué así?** Cada regla tiene su razón:

| Regla | Por qué |
|---|---|
| Un solo archivo | Solo hay que subir **un** archivo al servidor |
| Sin JavaScript | Un sitio **estático** solo muestra contenido; no ejecuta programas |
| Nada de internet | El sitio no depende de servidores ajenos: todo lo que se ve sale de **su** servidor |
| Datos inventados | **Nunca** pongan datos personales reales (teléfono, dirección, fotos) en un sitio de práctica |
| El usuario del equipo en el pie | Es la prueba de que el sitio es **de su equipo** |

⚠️ **La IA puede equivocarse o no hacer caso.** Un administrador **revisa** lo que va a
publicar, no lo sube a ciegas. En el paso 7 lo van a comprobar.

### Paso 4: [Computadora] Crear una carpeta para el sitio

Abran **Git Bash** y escriban:

```bash
mkdir ~/sitio-empresa
```

```
mkdir   ~/sitio-empresa
└─┬─┘   └─────┬──────┘
  │           la carpeta nueva, dentro de su carpeta personal (~)
  "make directory": crear una carpeta
```

**Qué deben ver:** nada. Si regresa el prompt sin mensajes, la carpeta se creó.

### Paso 5: [Computadora] Entrar a la carpeta

```bash
cd ~/sitio-empresa
```

`cd` (*change directory*, "cambiar de carpeta") los mete a la carpeta que acaban de
crear. Todo lo que hagan a partir de aquí en Git Bash pasa **dentro** de ella.

**Qué deben ver:** el prompt ahora termina con `~/sitio-empresa`.

### Paso 6: [Computadora] Guardar el código en un archivo

```bash
notepad index.html
```

Abre el **Bloc de notas** de Windows con un archivo llamado exactamente `index.html`,
**dentro de la carpeta del paso 4**.

**Qué deben hacer:**

1. Si el Bloc de notas pregunta si quieren **crear un archivo nuevo**, respondan
   **Sí**.
2. **Peguen** el código de la IA con **Ctrl + V**.
3. Guarden con **Ctrl + S** y cierren el Bloc de notas.

**¿Por qué desde Git Bash y no desde el menú del Bloc de notas?** Porque así el
archivo queda con el nombre exacto y en la carpeta correcta. Si lo guardan desde el
menú, Windows suele agregarle `.txt` al final sin avisar (`index.html.txt`), y nginx
no lo reconoce como página.

*En Mac:* en lugar de `notepad`, escriban `nano index.html`. Peguen con **Cmd + V**,
guarden con **Ctrl + O** y **Enter**, y salgan con **Ctrl + X**.

### Paso 7: [Computadora] Revisar que el sitio esté desconectado

```bash
grep -c "http" index.html
```

```
grep   -c   "http"   index.html
└─┬┘   └┬┘  └──┬─┘   └───┬────┘
  │     │      │         en qué archivo buscar
  │     │      qué palabra buscar
  │     cuenta cuántas líneas la tienen, en lugar de mostrarlas
  busca texto dentro de un archivo
```

**Qué deben ver:** **`0`**. Ninguna línea del código tiene `http`, así que el sitio no
descarga nada de internet.

Si sale **otro número**, la IA no siguió la regla. Pídanle: *"Quita del código todo lo
que se descargue de internet"*, y repitan los pasos 6 y 7 con el código nuevo.

### Paso 8: [Servidor] Averiguar la dirección del servidor

En su **VM**, escriban:

```bash
hostname -I
```

`hostname -I` (con **I mayúscula**) muestra la dirección IP del servidor, algo como
`192.168.1.50`. **Anótenla**: la van a usar para subir el sitio y para visitarlo.

Es posible que después aparezca otra dirección mucho más larga, con `:` en lugar de
puntos (por ejemplo `2001:db8:...`): es una dirección **IPv6**. Ignórenla y usen la
**primera**, la de 4 números separados por puntos.

### Dos formas de usar SFTP: terminal o programa visual

Para subir el sitio necesitan un **cliente de SFTP**: el programa que se conecta al
servidor y le manda archivos. Pueden usar el que prefieran, porque **el servidor es el
mismo y el protocolo es el mismo**:

| Opción | Cliente | Cómo se ve |
|---|---|---|
| **A** | El comando `sftp` de **Git Bash** (o de Terminal, en Mac) | Se escriben órdenes como `put` |
| **B** | Un **programa visual**: **FileZilla**, **WinSCP** (solo Windows) o **Cyberduck** (Windows y Mac) | Dos paneles, su computadora y el servidor, y se **arrastra** el archivo |

- Si eligen la **opción A**, sigan los pasos 9 a 16 tal como están.
- Si eligen la **opción B**, hagan el paso 8, después sigan la sección
  [Opción B: subir el sitio con un programa visual](#opción-b-subir-el-sitio-con-un-programa-visual)
  (está después de la tabla de errores) y regresen al **paso 17**. Los pasos 12 a 14
  (los permisos) **también** se hacen en la opción B: ahí se indica cuándo.

Con cualquiera de las dos van a ver el mismo error de permisos la primera vez, y lo van
a resolver de la misma forma: el permiso lo da el **servidor**, no el programa.

### Paso 9: [Computadora] Conectarse por SFTP

Regresen a **Git Bash** (deben seguir en la carpeta `~/sitio-empresa`) y escriban, con
el **usuario del equipo** y **la dirección del paso 8**:

```bash
sftp equipo-3@192.168.1.50
```

```
sftp   equipo-3 @ 192.168.1.50
└┬─┘   └──┬───┘   └─────┬────┘
 │        │             la dirección de su servidor
 │        con qué usuario entrar: el del equipo
 el cliente de SFTP: transferir archivos de forma cifrada
```

**Qué deben hacer:**

1. Si pregunta `Are you sure you want to continue connecting (yes/no/[fingerprint])?`,
   escriban **`yes`** y Enter. Es la huella del servidor, igual que en la lectura de
   SSH; solo lo pregunta la primera vez.
2. Escriban la **contraseña del usuario del equipo**. No aparece nada mientras escriben.

**Qué deben ver:** el prompt cambia a **`sftp>`**. Ahora todo lo que escriban son
órdenes para el servidor de archivos, no comandos de Git Bash.

### Paso 10: [Computadora, en `sftp>`] Intentar subir el sitio

Intenten dejar el archivo directo en la carpeta de las páginas web:

```bash
put index.html /var/www/html/index.html
```

```
put   index.html   /var/www/html/index.html
└┬┘   └───┬────┘   └──────────┬───────────┘
 │        │                   a dónde va en el servidor
 │        el archivo de su computadora
 "poner": subir un archivo
```

**Qué deben ver:** **un error** con las palabras **`Permission denied`** ("permiso
denegado"). **Así tiene que salir**: es a propósito, y el siguiente paso explica por
qué.

**¿Por qué falla?** SFTP entra al servidor **con el usuario del equipo**, y la carpeta
`/var/www/html` **le pertenece a `root`**. Ese usuario puede **leerla**, pero no
**escribir** en ella. Con `sudo` esto no pasa, porque `sudo` trabaja como `root`; pero
SFTP no tiene `sudo`. Es el mismo sistema de permisos de la lectura
[Archivos, carpetas y permisos](../../contenido/etapa-3-sistema-operativo-red/02-primeros-pasos/05-archivos-y-permisos.md).

### Paso 11: [Computadora, en `sftp>`] Salir de SFTP

```bash
bye
```

**Qué deben ver:** regresa el prompt normal de Git Bash.

### Paso 12: [Servidor] Ver de quién es la carpeta del sitio

En su **VM**:

```bash
ls -ld /var/www/html
```

```
ls   -ld   /var/www/html
└┤   └┬┘   └─────┬─────┘
 │    │          la carpeta de las páginas web de nginx
 │    -l: con detalles (permisos y dueño), -d: de la carpeta misma, no de lo que tiene
 "list": listar
```

**Qué deben ver:** una línea parecida a esta:

```
drwxr-xr-x 2 root root 4096 nov  6 18:00 /var/www/html
```

| Parte | Qué significa |
|---|---|
| `d` | Es una carpeta (*directory*) |
| `rwx` | El **dueño** puede leer, escribir y entrar |
| `r-x` `r-x` | El **grupo** y **los demás** solo pueden leer y entrar, **no escribir** |
| `root root` | El dueño es **`root`**, y el grupo también |

Ahí está la causa del error: el dueño es `root`, y para todos los demás, incluido su
usuario, la carpeta es de **solo lectura**.

### Paso 13: [Servidor] Hacer al usuario del equipo dueño de la carpeta

```bash
sudo chown -R equipo-3 /var/www/html
```

```
sudo   chown   -R   equipo-3   /var/www/html
       └─┬─┘   └┬┘  └──┬───┘   └─────┬─────┘
         │      │      │             qué carpeta
         │      │      el nuevo dueño: el usuario del equipo
         │      la carpeta y todo lo que tiene adentro (recursivo)
         "change owner": cambiar el dueño
```

Lleva `sudo` porque solo el administrador puede regalar archivos de `root` a otro
usuario (Free Software Foundation, s.f.-a).

**Qué deben ver:** nada. Si regresa el prompt sin mensajes, salió bien.

**¿Y nginx podrá seguir leyendo el sitio?** Sí. nginx trabaja con su propio usuario,
**`www-data`**, que para esta carpeta cuenta como "los demás", y "los demás" siguen
teniendo permiso de **leer** (`r-x`). Ustedes ganan permiso de **escribir**, y nginx no
pierde nada.

*En un servidor real con varios administradores, en lugar de un dueño único se usa un
grupo para el equipo web. Para esta práctica, un solo dueño es suficiente y más claro.*

### Paso 14: [Servidor] Comprobar el cambio

```bash
ls -ld /var/www/html
```

**Qué deben ver:** la misma línea del paso 12, pero ahora con **el usuario del equipo** como
dueño: `drwxr-xr-x 2 equipo-3 root ...`. Los permisos (`rwxr-xr-x`) son los
mismos; lo que cambió es **quién** tiene el `rwx`.

### Paso 15: [Computadora] Conectarse otra vez por SFTP

En **Git Bash**, en la carpeta `~/sitio-empresa`:

```bash
sftp equipo-3@192.168.1.50
```

Igual que en el paso 9: escriban su contraseña y esperen el prompt `sftp>`.

### Paso 16: [Computadora, en `sftp>`] Subir el sitio

```bash
put index.html /var/www/html/index.html
```

El mismo comando que falló en el paso 10.

**Qué deben ver:** ahora **sí** funciona. Aparece una línea que dice que está subiendo
el archivo (`Uploading index.html to /var/www/html/index.html`) y otra con el avance
hasta **`100%`** (OpenBSD, s.f.).

> 📸 **Captura 1:** la ventana de Git Bash con la conexión `sftp` y el `put` llegando
> a `100%` (o, en la opción B, la ventana del programa visual con `index.html` dentro
> de `/var/www/html`).

Después salgan con:

```bash
bye
```

### Paso 17: [Servidor] Ver el archivo en el servidor

En su **VM**:

```bash
ls -l /var/www/html
```

**Qué deben ver:** la lista de archivos de la carpeta. Debe aparecer **`index.html`**,
con el usuario del equipo como dueño y los permisos **`-rw-r--r--`**: ustedes pueden leerlo y
escribirlo, y **los demás (nginx) pueden leerlo**. También aparece
`index.nginx-debian.html`, la página de bienvenida que trae nginx; no estorba, porque
nginx busca primero `index.html` (F5, s.f.).

### Paso 18: [Servidor] Visitar el sitio desde el propio servidor

```bash
curl -s 127.0.0.1 | grep "equipo-3"
```

```
curl -s 127.0.0.1   |   grep "equipo-3"
└───────┬───────┘   │   └──────┬──────┘
        │           │          de toda la página, solo la línea con el usuario
        │           la tubería: pasa la página al siguiente comando
        pide la página a nginx (-s: sin mensajes de avance)
```

`127.0.0.1` es la dirección de **loopback**: el propio servidor hablándose a sí mismo
(la vieron en la Etapa 4). El sitio ahora es largo, así que en lugar de mostrar todo el
código, `grep` deja solo la línea que importa.

**Qué deben ver:** la línea del pie de página, con `Sitio administrado por
equipo-3` (puede venir sola o entre etiquetas de HTML, según cómo la escribió la
IA). Eso confirma que nginx está entregando **el
sitio que subieron**.

> 📸 **Captura 2:** el prompt con el usuario del equipo, la salida de
> `ls -l /var/www/html` y la de `curl` con la línea del pie de página.

### Paso 19: Visitar el sitio desde su computadora

Este paso **no es un comando**: en el **navegador** de su computadora escriban, con la
dirección del paso 8:

```
http://192.168.1.50
```

Escriban `http://` al inicio: sin él, algunos navegadores intentan `https://` y no
abre.

**Qué deben ver:** **el sitio de su empresa**, con colores, secciones y el usuario del equipo en
el pie de página. Si ven la página anterior, recarguen con **Ctrl + F5**.

> 📸 **Captura 3:** el navegador mostrando el sitio, con la dirección visible en la
> barra.

### Si algo sale mal

| Qué pasa | Por qué | Qué hacer |
|---|---|---|
| `grep -c` no da `0` (paso 7) | La IA puso algo que se descarga de internet | Pídanle que lo quite y repitan los pasos 6 y 7 |
| `ls` en Git Bash muestra `index.html.txt` | El Bloc de notas agregó `.txt` | `mv index.html.txt index.html` |
| `sftp` dice `Connection timed out` o `Connection refused` | No llega al servidor | Revisen la dirección del paso 8, que la VM esté en modo **puente** y que el servicio SSH esté activo (Evidencia 3, paso 14) |
| `sftp` pide la contraseña una y otra vez y termina en `Permission denied` | Usuario o contraseña equivocados | Revisen que escribieron el usuario del equipo (`equipo-N`), sin espacios |
| `put` sigue diciendo `Permission denied` en el paso 16 | La carpeta sigue siendo de `root` | Repitan el paso 13 revisando el usuario del equipo, y compruébenlo con el paso 14 |
| El navegador muestra **Welcome to nginx!** | El archivo no llegó a `/var/www/html` | Revisen que en el `put` escribieron el destino completo: `/var/www/html/index.html` |
| El navegador muestra **403 Forbidden** | nginx no tiene permiso de **leer** el archivo | En el servidor: `sudo chmod 644 /var/www/html/index.html` y recarguen |
| El sitio se ve sin colores ni diseño | La IA puso el CSS en otro archivo | Pídanle que ponga el CSS dentro de `<style>`, en el mismo archivo |
| En el paso 24 la página **sigue abriendo** | El navegador reusó la conexión anterior; el firewall solo bloquea las nuevas | Esperen 2 minutos y recarguen, o ábranla en una ventana de incógnito |
| En el paso 26 aparece `::1` en lugar de `127.0.0.1` | En el paso 18 escribieron `localhost`, que Ubuntu resuelve a `::1`, el loopback de IPv6 | Es la misma visita del propio servidor; para ver `127.0.0.1`, repitan el paso 18 tal como está escrito |
| FileZilla dice que no puede conectar o pide "FTP" | Le falta el `sftp://` o el puerto `22` | Escriban `sftp://` antes de la dirección y `22` en el puerto |

### Opción B: subir el sitio con un programa visual

Sigan esta sección **en lugar de los pasos 9 a 16**, después de hacer el paso 8. Los
ejemplos usan **FileZilla**, que funciona en Windows y Mac; en WinSCP y Cyberduck los
campos se llaman casi igual. Los nombres exactos de los botones pueden cambiar un poco
según la versión.

#### B1. Instalar el programa

Descarguen el programa **solo de su página oficial**:

| Programa | Página oficial |
|---|---|
| FileZilla Client | https://filezilla-project.org (FileZilla Project, s.f.) |
| WinSCP | https://winscp.net (Prikryl, s.f.) |
| Cyberduck | https://cyberduck.io (iterate GmbH, s.f.) |

⚠️ **Lean cada pantalla del instalador.** Algunos instaladores, como el de FileZilla,
pueden ofrecer instalar **otros programas** que no tienen nada que ver (un antivirus,
una barra del navegador). **Rechácenlos** y sigan solo con la instalación del cliente.
Descargar de páginas que no son la oficial es la forma más común de instalar un virus.

#### B2. Conectarse al servidor

En FileZilla, en la barra de **Conexión rápida** de arriba, llenen:

| Campo | Qué escribir |
|---|---|
| Servidor | `sftp://192.168.1.50`, con la dirección del **paso 8** |
| Nombre de usuario | El usuario del equipo, por ejemplo `equipo-3` |
| Contraseña | La contraseña del usuario del equipo |
| Puerto | `22` |

El `sftp://` al inicio es importante: le dice a FileZilla que use **SFTP**. Sin él,
intenta FTP normal, que no está instalado en su servidor (y que además no cifra la
contraseña, como vieron en la Etapa 2). Presionen **Conexión rápida**.

La primera vez aparece una ventana de **clave de host desconocida**: es la **huella**
del servidor, la misma que pregunta `ssh` la primera vez. Marquen la casilla para
**confiar siempre en este servidor** y acepten.

**Qué deben ver:** el panel de la **izquierda** es **su computadora** y el de la
**derecha** es **su servidor**. Abajo, en el registro de mensajes, una línea que dice
que la conexión se estableció.

#### B3. Ir a las carpetas correctas

- **Izquierda (su computadora):** abran la carpeta `sitio-empresa` de su carpeta
  personal (la del paso 4) y localicen `index.html`.
- **Derecha (su servidor):** en el cuadro **Sitio remoto**, escriban `/var/www/html` y
  presionen **Enter**.

#### B4. Intentar subir el archivo

**Arrastren** `index.html` del panel izquierdo al panel derecho.

**Qué deben ver:** **un error**. En el registro de mensajes aparece una línea en rojo
con las palabras **permission denied** ("permiso denegado"), y abajo, en la lista de
**transferencias fallidas**, queda `index.html`. Es el mismo error del paso 10, por la
misma razón: el programa entra con **el usuario del equipo**, y la carpeta es de `root`. Lean la
explicación del paso 10.

#### B5. Dar el permiso en el servidor

Vayan a su **VM** y hagan los **pasos 12, 13 y 14**, igual que en la opción A. No
cierren el programa visual.

#### B6. Subir el archivo otra vez

Regresen al programa y **arrastren otra vez** `index.html` al panel derecho. Si les
pregunta si quieren **sobrescribir** un archivo que ya existe, respondan que sí: es la
página de bienvenida de nginx o el intento anterior.

**Qué deben ver:** ahora la transferencia termina bien, sin línea roja, e `index.html`
aparece en el panel derecho, dentro de `/var/www/html`.

> 📸 **Captura 1 (opción B):** la ventana del programa con el usuario del equipo y la dirección
> del servidor visibles arriba, e `index.html` dentro de `/var/www/html` en el panel
> derecho.

Listo: continúen en el **paso 17**.

---

## Parte 3: El firewall

Un **firewall** ("cortafuegos") es un programa que decide **qué conexiones dejar
entrar** al servidor y cuáles bloquear. Es la primera medida de la **administración de
seguridad** de la Etapa 1. Ubuntu trae **ufw** (*Uncomplicated Firewall*, "firewall sin
complicaciones"), que **al instalarse está apagado** y que, al encenderlo, **bloquea
todo lo que entra** salvo lo que ustedes permitan (Canonical Ltd., s.f.-a).

Su servidor recibe dos tipos de conexiones:

| Puerto | Servicio | ¿Para qué lo necesitan? |
|---|---|---|
| **22** | SSH y SFTP | Para entrar al servidor desde Git Bash y para subir el sitio |
| **80** | HTTP (nginx) | Para que se vea la página de la empresa |

⚠️ **El orden de esta parte es muy importante.** Primero se **permiten** los puertos y
**después** se enciende el firewall. Si lo encienden sin permitir el puerto 22 y están
conectados por SSH, **pierden la conexión**. No se rompe nada: entran por la ventana de
VirtualBox y corrigen, pero se ahorran el susto siguiendo el orden.

### Paso 20: Permitir SSH

```bash
sudo ufw allow 22/tcp
```

```
sudo   ufw   allow   22/tcp
└┬─┘   └┬┘   └─┬─┘   └─┬──┘
 │      │      │       el puerto 22 (SSH), con el protocolo TCP
 │      │      "permitir"
 │      el firewall
 como administrador
```

`tcp` es el protocolo de transporte que usan SSH y HTTP (lo vieron en Redes y
Telecomunicaciones).

**Qué deben ver:** un mensaje que dice que las reglas se actualizaron (*Rules
updated*), una vez para IPv4 y otra para IPv6 (`(v6)`). Todavía no bloquea nada: el
firewall sigue apagado, solo anotaron la regla.

### Paso 21: Permitir la página web

```bash
sudo ufw allow 80/tcp
```

Igual que el anterior, pero para el puerto **80**, el de nginx.

**Qué deben ver:** el mismo mensaje de reglas actualizadas.

### Paso 22: Encender el firewall

```bash
sudo ufw enable
```

`enable` significa "activar". A partir de aquí, ufw bloquea todo lo que entra excepto
los puertos 22 y 80.

**Qué deben ver:** si están conectados por SSH, ufw les advierte que encenderlo puede
cortar las conexiones SSH y pregunta si quieren continuar (`y|n`) (Canonical Ltd.,
s.f.-b). Escriban **`y`** y presionen Enter: ya permitieron el 22, así que no se van a
desconectar. Después dice que el firewall está **activo** y que se encenderá cada vez
que arranque el sistema.

### Paso 23: Revisar las reglas

```bash
sudo ufw status verbose
```

`status` muestra el estado del firewall y `verbose` ("detallado") agrega más
información.

**Qué deben ver:**

- `Status: active`: el firewall está encendido.
- Una línea `Default:` que dice `deny (incoming)`: **todo lo que entra se bloquea**
  por defecto, y `allow (outgoing)`: lo que sale se permite.
- Una tabla con las columnas `To` ("hacia"), `Action` ("acción") y `From` ("desde"),
  con sus dos reglas: `22/tcp` y `80/tcp` con `ALLOW IN` y `Anywhere` ("desde
  cualquier lugar"). Cada una aparece dos veces: una normal y otra con `(v6)`.

> 📸 **Captura 4:** el prompt con el usuario del equipo y la salida de `sudo ufw status verbose`.

### Paso 24: Comprobar que el firewall bloquea

Ahora van a ver al firewall trabajando. **Quiten** el permiso del puerto 80:

```bash
sudo ufw delete allow 80/tcp
```

Para borrar una regla se escribe `delete` seguido de la regla tal como la crearon
(Canonical Ltd., s.f.-b).

**Qué deben ver:** un mensaje de que la regla se borró (*Rule deleted*), para IPv4 y
para IPv6.

Ahora regresen al **navegador** de su computadora y **recarguen** la página
(tecla **F5**). **Ya no abre**: el navegador se queda cargando y al final marca error.
El servidor web sigue encendido, pero el firewall ya no deja pasar las conexiones al
puerto 80.

⚠️ **Si todavía abre, no es un error.** El navegador deja abierta por un rato la
conexión de la visita anterior, y el firewall solo bloquea las conexiones **nuevas**.
Esperen **2 minutos** y recarguen otra vez, o abran la dirección en una **ventana de
incógnito** (**Ctrl + Shift + N**), que siempre empieza una conexión nueva.

> 📸 **Captura 5:** el navegador de su computadora sin poder abrir la página.

### Paso 25: Volver a permitir la página

```bash
sudo ufw allow 80/tcp
```

Recarguen el navegador: la página vuelve a abrir. Así funciona la seguridad de un
servidor: **solo queda abierto lo que el administrador decide**.

---

## Parte 4: Registro y respaldo

### Paso 26: Ver quién visitó el sitio

```bash
sudo tail -n 20 /var/log/nginx/access.log
```

```
sudo   tail   -n 20   /var/log/nginx/access.log
       └─┬┘   └─┬─┘   └───────────┬───────────┘
         │      │                 el registro de visitas de nginx
         │      solo las últimas 20 líneas
         muestra el final de un archivo ("cola")
```

nginx anota **cada visita** en el archivo `access.log`: es el registro de accesos que
vieron en la Etapa 3. Lleva `sudo` porque solo el administrador puede leer los
registros del servidor.

**Qué deben ver:** hasta 20 líneas, una por visita. Cada línea empieza con la
**dirección IP de quien visitó**: en la visita del paso 18 es `127.0.0.1` (el propio
servidor, con `curl`), y en las del navegador es la dirección **privada de su
computadora**, algo como `192.168.1.20`. Más adelante en la línea aparece la fecha y la
hora, `GET /` (pidió la página principal), el número `200` (se entregó bien) y el
programa que hizo la visita: `curl/...` o el nombre del navegador.

También pueden aparecer líneas con `GET /favicon.ico` y el número `404` (el navegador
pide solo el icono de la pestaña, y su sitio no tiene) o con `304` (el navegador
recargó y la página no había cambiado). Son normales. La que importa es la de `curl`,
desde `127.0.0.1`.

Esto es la **administración de fallas y de contabilidad** de la Etapa 1: el registro
dice quién usó el servicio, cuándo y si funcionó.

> 📸 **Captura 6:** el prompt con el usuario del equipo y la salida del registro, donde se vea al
> menos una visita desde `127.0.0.1` y una desde su computadora.

### Paso 27: Sacar un respaldo del sitio

```bash
tar -czf ~/respaldo-sitio.tar.gz -C /var/www html
```

Un **respaldo** es una copia guardada aparte, para poder recuperar el sitio si algo se
borra o se descompone. `tar` junta una carpeta completa en **un solo archivo**
comprimido (Free Software Foundation, s.f.-b):

| Parte | Qué hace |
|---|---|
| `tar` | El programa que empaqueta archivos (de *tape archive*, "archivo en cinta") |
| `-c` | **Crear** un paquete nuevo |
| `-z` | **Comprimirlo**, para que ocupe menos (por eso termina en `.gz`) |
| `-f ~/respaldo-sitio.tar.gz` | El **archivo** donde se guarda: en su carpeta personal (`~`) |
| `-C /var/www` | **Entrar** primero a la carpeta `/var/www` |
| `html` | Lo que se respalda: la carpeta `html`, donde está la página |

No lleva `sudo`: cualquier usuario puede **leer** la página (por eso nginx la puede
entregar), y el respaldo se guarda en su propia carpeta.

**Qué deben ver:** nada. Si no aparece ningún mensaje y regresa el prompt, el
respaldo se creó bien. En Linux, "sin mensaje" casi siempre significa "todo salió
bien".

### Paso 28: Comprobar el respaldo

```bash
tar -tzf ~/respaldo-sitio.tar.gz
```

La `-t` (en lugar de `-c`) significa **listar**: muestra lo que hay dentro del
respaldo sin sacarlo.

**Qué deben ver:** la carpeta `html/` y sus archivos, entre ellos `html/index.html`,
la página de la empresa. También puede aparecer la página de bienvenida que trae nginx.

> 📸 **Captura 7:** el prompt con el usuario del equipo y la salida de `tar -tzf`, donde se vea
> `html/index.html`.

**El servidor está listo.** Ahora tiene el sitio de la empresa,
un firewall que solo deja entrar SSH y web, un registro de visitas y un respaldo.

---

## Parte 5: El plan de direccionamiento

La empresa va a crecer y necesita su propia red interna, con **4 departamentos**:
**Ventas**, **Administración**, **Almacén** y **Wi-Fi de clientes**. Su red es:

| Equipo | Red de la empresa |
|---|---|
| 1 | `192.168.101.0/24` |
| 2 | `192.168.102.0/24` |
| 3 | `192.168.103.0/24` |
| 4 | `192.168.104.0/24` |
| 5 | `192.168.105.0/24` |
| 6 | `192.168.106.0/24` |
| 7 | `192.168.107.0/24` |

Busquen su número de equipo en la tabla: esa es su red.

⚠️ **Esa red no se configura en ningún lado.** Es el **plan** de la red que tendría la
empresa, como el plano de un edificio antes de construirlo. No le cambien la dirección
a su VM ni a su computadora: ipcalc es solo una calculadora y no modifica nada. Su VM
sigue con la dirección que le da el router de su casa (la del paso 8).

Igual que en la Evidencia 4, **no hay que calcular nada**: lo hace ipcalc (Jodies,
s.f.). Estos dos pasos se hacen en la misma VM, con el usuario del equipo.

### Paso 29: Dividir la red de la empresa en 4 subredes

Como son **4** departamentos, el prefijo de las subredes es **`/26`** (la misma tabla
de la Evidencia 4: 2 subredes → `/25`, 4 → `/26`, 8 → `/27`). Escriban el comando con
la red de **su** equipo (de la tabla) en lugar de `192.168.103.0`:

```bash
ipcalc -b 192.168.103.0/24 /26
```

```
ipcalc   -b   192.168.103.0/24   /26
         └┬┘  └───────┬───────┘   └┬┘
          │           │            el prefijo de las 4 subredes
          │           la red de su empresa (de la tabla)
          sin binario, para que se lea fácil
```

**Qué deben ver:** la línea `Subnets after transition from /24 to /26`, cuatro bloques
numerados `1.` a `4.` (uno por subred, cada uno con `Network`, `HostMin`, `HostMax` y
`Broadcast`) y al final **`Subnets: 4`**.

> 📸 **Captura 8:** el prompt con el usuario del equipo y la salida completa, hasta
> `Subnets: 4`.

### Paso 30: Llenar la tabla de la empresa

Este paso **no es un comando**: **copien** lo que mostró el paso 29 a esta tabla, en
el reporte. Cada departamento toma un bloque, en orden. Por costumbre, el **gateway**
(la dirección del router de cada departamento) es la primera dirección para equipos,
el `HostMin`:

| Departamento | Subred (`Network`) | Gateway (`HostMin`) | Último equipo (`HostMax`) | Broadcast |
|---|---|---|---|---|
| Ventas (bloque 1) | | | | |
| Administración (bloque 2) | | | | |
| Almacén (bloque 3) | | | | |
| Wi-Fi de clientes (bloque 4) | | | | |

*Ejemplo de la primera fila para el equipo 3:* Ventas · `192.168.103.0/26` ·
`192.168.103.1` · `192.168.103.62` · `192.168.103.63`.

---

## Reporte técnico

Un **reporte técnico** explica qué se hizo, cómo y qué resultó, de forma que otra
persona pueda repetirlo. Armen **un solo documento** con estas secciones, en este
orden:

1. **Portada:** nombre de la empresa, número de equipo, nombre y matrícula de cada
   integrante, materia y fecha.
2. **Objetivo:** en 3 o 4 líneas, qué servidor montaron y para qué sirve a la empresa.
3. **Servicios configurados:** una tabla con cada servicio o herramienta (SFTP,
   permisos con `chown`, nginx, ufw, access.log, tar, ipcalc), para qué sirve y de qué
   etapa del curso viene. Usen la tabla del inicio de este documento como guía.
4. **Procedimiento:** los pasos que siguieron, con los comandos, explicados **con sus
   palabras** (no copien las explicaciones de este documento). Incluyan el texto que le
   pidieron a la IA en el paso 3 y expliquen **por qué falló el primer `put`** y cómo lo
   resolvieron.
5. **Evidencias:** las capturas 1 a 8, cada una con su título (*Captura 4: reglas
   del firewall*).
6. **Plan de direccionamiento:** la tabla del paso 30.
7. **Problemas y soluciones:** qué falló durante la práctica y cómo lo resolvieron. Si
   nada falló, cuenten qué paso les costó más trabajo entender.
8. **Conclusión:** un párrafo sobre su experiencia con la instalación y el manejo del
   servidor.

Guárdenlo como PDF: **Archivo → Guardar como → PDF**.

---

## Entregable

Un solo archivo **PDF** con el reporte técnico completo. **No hay presentación**: el
reporte es lo único que se entrega y se califica.

---

## Forma de entrega

- **Modalidad:** En equipo. Un solo integrante hace la práctica y sube el PDF en
  nombre del equipo.
- **Plataforma:** NEXUS. Carguen su archivo en la evidencia *"Proyecto de Servidor"*, en la sección *Producto Integrador de Aprendizaje*.
- Nombren su archivo de la siguiente forma:

```
PIA_EquipoN.pdf
```

*Ejemplo: `PIA_Equipo3.pdf`*

- **Fecha de entrega:** sábado 14 de noviembre de 2026.
- **Hora límite:** 10:00 p.m.

---

## Rúbrica de evaluación

| CRITERIO / NIVEL DE DOMINIO | Evidencia Completa | Evidencia Suficiente | Evidencia Débil | Sin evidencia |
|---|---|---|---|---|
| **Conocimientos** — Analiza las características del proceso de administración de redes. Conoce el alcance de cada una de las áreas administrativas | **3** — El reporte tiene las 8 secciones. En *Servicios configurados* y *Procedimiento* explica **con sus palabras** para qué sirve cada herramienta (SFTP, permisos con `chown`, nginx, ufw, access.log, tar, ipcalc) y la relaciona con su área administrativa (seguridad, fallas, contabilidad, configuración) y con su etapa. Está la conclusión. | **2** — Explica solo algunas herramientas, o no las relaciona con su área, o falta la conclusión. | **1** — Copia las explicaciones de este documento, o no relaciona ninguna herramienta con las áreas administrativas, o faltan varias secciones. | **0** — Sin evidencia. |
| **Habilidades** — Identifica problemas y sugiere soluciones con software para cada área de estudio. Opera el servidor con rapidez y claridad | **5** — Están las capturas 1 a 7 con el mismo usuario en el prompt (`equipo-N`, o el usuario personal con el que ya habían avanzado): subieron el sitio por SFTP, el sitio generado abre desde el servidor y desde la computadora, el firewall muestra sus reglas y bloquea el puerto 80, el registro muestra visitas y el respaldo contiene `html/index.html`. Está la captura 8 y la tabla del plan de direccionamiento coincide con ella. La sección *Problemas y soluciones* describe qué falló y cómo se resolvió. | **4** — Falta una parte: una captura, una prueba que no funcionó, la tabla no coincide con la captura 8, o no se describen los problemas. | **3** — Faltan dos o más partes, o no se ve que el servidor funcione. | **0** — Sin evidencia. |
| **Actitudes / Valores** — Respeta las decisiones del docente y opiniones de sus compañeros | **2** — Siguieron las indicaciones del docente (la portada tiene a todos los integrantes) y entregaron en tiempo y forma: en la fecha y hora indicadas, en un solo PDF con el nombre `PIA_EquipoN.pdf`. | **1** — Siguieron las indicaciones y entregaron un día posterior a la fecha establecida. | **0** — Entregaron dos días después de la fecha establecida. | **0** — Sin evidencia. |

*Se respetan los criterios, niveles y puntos de la rúbrica de NEXUS; solo se adaptó el
contenido de cada nivel a esta práctica.*

---

## Referencias

Canonical Ltd. (s.f.-a). *Firewalls* [Ubuntu Server documentation].
https://documentation.ubuntu.com/server/how-to/security/firewalls/

Canonical Ltd. (s.f.-b). *ufw(8): program for managing a netfilter firewall* [Ubuntu
manpages, resolute]. https://manpages.ubuntu.com/manpages/resolute/man8/ufw.8.html

Canonical Ltd. (s.f.-c). *User management* [Ubuntu Server documentation].
https://documentation.ubuntu.com/server/how-to/security/user-management/

Debian. (s.f.). *adduser.conf(5): configuration file for adduser*. Debian Manpages.
https://manpages.debian.org/testing/adduser/adduser.conf.5.en.html

F5. (s.f.). *Module ngx_http_index_module* [nginx documentation].
https://nginx.org/en/docs/http/ngx_http_index_module.html

FileZilla Project. (s.f.). *FileZilla: The free FTP solution*.
https://filezilla-project.org

Free Software Foundation. (s.f.-a). *GNU Coreutils: chown invocation*.
https://www.gnu.org/software/coreutils/manual/html_node/chown-invocation.html

Free Software Foundation. (s.f.-b). *GNU tar manual*.
https://www.gnu.org/software/tar/manual/tar.html

iterate GmbH. (s.f.). *Cyberduck*. https://cyberduck.io

Jodies, K. (s.f.). *ipcalc(1): An IPv4 netmask/broadcast/etc calculator*.
https://jodies.de/ipcalc

OpenBSD. (s.f.). *sftp(1): OpenSSH secure file transfer*.
https://man.openbsd.org/sftp

Prikryl, M. (s.f.). *WinSCP*. https://winscp.net
