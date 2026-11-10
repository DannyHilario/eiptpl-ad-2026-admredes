# Evidencia 3: Tu servidor con un administrador propio, nginx y SFTP

De manera **individual**, en tu propia VM de Ubuntu Server, vas a hacer lo que hace un
administrador de redes con un servidor nuevo:

1. **Actualizarlo**, para que tenga las correcciones de seguridad más recientes.
2. **Crear tu propio usuario administrador**, con tus iniciales y tu matrícula.
3. **Instalar nginx**, un servidor web, y comprobar que entrega una página.
4. **Instalar SFTP**, el servicio para transferir archivos de forma segura, y
   comprobar que está activo.

Es una práctica **paso a paso**: cada paso es **un solo comando**. En cada uno vas a
encontrar:

- **El comando**, en un recuadro gris. Escríbelo tal cual y presiona **Enter**.
- **Qué significa cada parte**, palabra por palabra.
- **Qué debes ver** en la pantalla. Compáralo antes de pasar al siguiente paso.

No te saltes las explicaciones: el objetivo no es solo que el comando funcione, sino
que **entiendas qué le estás pidiendo al servidor**. Vas a tomar **6 capturas** y a
entregarlas en un PDF.

---

## Antes de empezar

### Tu usuario: iniciales y matrícula

En esta práctica vas a crear un usuario con este formato:

```
iniciales-matricula
```

- **Iniciales:** la primera letra de cada uno de tus nombres y apellidos, **en
  minúsculas**, sin acentos (si tienes una ñ, usa n).
- Un **guion** (`-`).
- Tu **matrícula** completa.

*Ejemplo:* Daniel Gustavo Hilario Medrano, con matrícula 1410751, crea el usuario
**`dghm-1410751`**.

¿Por qué así? Porque en Ubuntu un nombre de usuario debe **empezar con una letra** y
solo puede llevar **letras sin acentos, números, guiones y guiones bajos**: no acepta
espacios, acentos ni ñ (Debian, s.f.). Las minúsculas no son obligatorias, pero son la
costumbre en Linux, y como Linux distingue mayúsculas de minúsculas, escribir todo en
minúsculas evita equivocarse al teclear el usuario.

⚠️ **En todos los comandos de esta práctica aparece `dghm-1410751`. Cada vez que lo
veas, escribe TU usuario en su lugar.** Es lo único que cambia; todo lo demás se
escribe exactamente igual.

### Cómo tomar las capturas

Cuando el texto diga **📸 Captura**, toma una captura de la ventana de tu VM:

- **Opción 1:** en el menú de la ventana de la VM, **Ver → Tomar captura de pantalla...**
  (*View → Take Screenshot...*) y guárdala en tu computadora.
- **Opción 2 (Windows):** **Windows + Shift + S** y selecciona solo la ventana de la VM.

Antes de cada captura puedes escribir `clear` y presionar Enter: borra lo que hay en
la pantalla, para que en la captura solo se vea lo que importa. En cada captura debe
verse **el prompt** (la línea que termina en `$`).

### Enciende tu servidor

Enciende tu VM en VirtualBox e **inicia sesión con tu usuario de siempre**, el que
creaste al instalar Ubuntu. Ese usuario ya es administrador: con él vas a crear el
nuevo.

---

## Parte 1: Actualizar el servidor

Un servidor recién instalado ya tiene programas con versiones viejas: desde que se
publicó el ISO, Ubuntu ha corregido errores y huecos de seguridad. Por eso **lo primero
que hace un administrador con cualquier servidor es actualizarlo**. Se hace en dos
pasos: primero se descarga la **lista** de lo que hay nuevo, y después se **instalan**
las versiones nuevas (Canonical Ltd., s.f.-b). Si quieres repasarlo con calma, está en
[El administrador, sudo y los
paquetes](../../contenido/etapa-3-sistema-operativo-red/02-primeros-pasos/03-administrador-y-paquetes.md).

### Paso 1: Descargar la lista de actualizaciones

```bash
sudo apt update
```

```
sudo   apt   update
└┬─┘   └┬┘   └──┬─┘
 │      │       qué quieres hacer: actualizar la LISTA de paquetes
 │      el programa que instala, actualiza y quita programas en Ubuntu
 ejecuta lo que sigue como administrador (cambia el servidor)
```

**Qué está pasando:** tu servidor se conecta por Internet al **repositorio** de Ubuntu
(el "almacén" oficial de programas) y descarga la lista más reciente de paquetes y sus
versiones. **Todavía no instala nada**: solo se entera de qué hay nuevo.

**Qué debes ver:**

1. `sudo` te pide **tu contraseña** con el mensaje `[sudo: authenticate] Password:`.
   Mientras la escribes aparece un **asterisco** (`*`) por cada letra: así sabes que
   el teclado sí está escribiendo.
2. Varias líneas que empiezan con `Get:` o `Hit:` (o `Obj:` y `Des:` en español):
   cada una es una lista que se está descargando.
3. Al final, regresa el prompt. **No debe aparecer ninguna línea que empiece con
   `Error:`**.

### Paso 2: Instalar las actualizaciones

```bash
sudo apt upgrade -y
```

```
sudo   apt   upgrade   -y
└┬─┘   └┬┘   └──┬──┘   └┬┘
 │      │       │       responde "sí" (yes) automáticamente a "¿Desea continuar?"
 │      │       qué quieres hacer: instalar las versiones nuevas
 │      el programa que maneja los paquetes
 como administrador
```

**Qué está pasando:** `apt` compara los programas que tienes instalados con la lista que
descargaste en el paso 1. Los que tienen versión nueva los **descarga, comprueba que
vengan firmados por Ubuntu** (para asegurarse de que nadie los modificó) **y los
instala**. Por eso el orden importa: sin el paso 1, `upgrade` compararía contra una
lista vieja.

**Qué debes ver:** una lista de paquetes que se van a actualizar, luego muchas líneas
de descarga e instalación. Puede tardar varios minutos. Al final puede aparecer un
texto que habla de reiniciar servicios (`Service restarts being deferred`, *outdated
binaries*): es un aviso normal, no un error. Si en cambio aparece una **ventana de
color** preguntando qué servicios reiniciar, deja las opciones como vienen y presiona
**Enter**. Termina cuando regresa el prompt. Si dice `0 upgraded` (o `0 actualizados`),
tu servidor ya estaba al día; también está bien.

> 📸 **Captura 1:** el final de `sudo apt upgrade -y`, con el prompt de regreso.

---

## Parte 2: Crear tu usuario administrador

Un servidor real lo administran **varias personas**, y cada una debe tener **su propia
cuenta**: así el sistema registra quién hizo qué. Nunca se comparte una cuenta de
administrador. En esta parte vas a crear tu cuenta personal y a darle permisos de
administrador. Repasa [Usuarios](../../contenido/etapa-3-sistema-operativo-red/02-primeros-pasos/02-usuarios.md)
si no recuerdas qué es una cuenta o un grupo.

### Paso 3: Crear el usuario

```bash
sudo adduser dghm-1410751
```

```
sudo   adduser   dghm-1410751
└┬─┘   └──┬──┘   └─────┬────┘
 │        │            el nombre de la cuenta nueva (¡el TUYO!)
 │        "add user": agregar un usuario
 como administrador: crear cuentas cambia todo el servidor
```

**Qué está pasando:** `adduser` hace varias cosas de una vez (Canonical Ltd., s.f.-c):

- Agrega la cuenta a la lista de usuarios del sistema (el archivo `/etc/passwd`) y le
  asigna un **número de usuario** (UID).
- Crea un **grupo** con el mismo nombre que el usuario.
- Crea su **carpeta personal**: `/home/dghm-1410751`.
- Le pone la contraseña que escribas.

**Qué debes hacer:** el comando te hace varias preguntas, una tras otra. Algunas salen
en inglés y otras en español; así es Ubuntu, no es un error:

1. Si `sudo` te pide **tu contraseña** (la de tu usuario de siempre), escríbela.
2. **`New password:`** la contraseña de tu **usuario nuevo**. Aquí **no aparece nada**
   mientras escribes, ni asteriscos: es normal. **Elige una que recuerdes**: la vas a
   usar varias veces en esta práctica.
3. **`Retype new password:`** escríbela otra vez, para confirmar que no te
   equivocaste. Si coinciden, aparece `passwd: contraseña actualizada correctamente`.
4. **`Nombre completo []:`** escribe tu **nombre completo**. Queda guardado como
   descripción de la cuenta.
5. **`Número de habitación []:`**, **`Teléfono del trabajo []:`**, **`Teléfono de
   casa []:`** y **`Otro []:`** son opcionales. Presiona **Enter** en cada uno para
   dejarlos vacíos.
6. **`Is the information correct? [Y/n]`:** presiona **Enter**. La letra en mayúscula
   (`Y`, de *yes*) es la respuesta que se elige si solo presionas Enter.

### Paso 4: Convertirlo en administrador

```bash
sudo usermod -aG sudo dghm-1410751
```

```
sudo   usermod   -aG    sudo   dghm-1410751
└┬─┘   └──┬──┘   └┬─┘   └┬─┘   └─────┬────┘
 │        │       │      │           a qué usuario
 │        │       │      a qué grupo: el grupo "sudo" (los administradores)
 │        │       -a: AGREGAR (append)  -G: a un Grupo
 │        "user modify": modificar un usuario que ya existe
 como administrador
```

**Qué está pasando:** en Ubuntu, los usuarios que pueden usar `sudo` son los que
pertenecen al **grupo `sudo`** (Canonical Ltd., s.f.-c). Tu usuario nuevo todavía no está
en ese grupo, así que todavía no es administrador. Este comando lo **agrega** al grupo.

La `a` es importante: sin ella, `-G` **reemplazaría** todos los grupos del usuario por
solo el que indicas, y lo sacaría de los demás (Kerrisk, s.f.-b). Con `-aG`, el grupo
se **suma** a los que ya tiene.

**Qué debes ver:** nada. Si el comando sale bien, regresa el prompt sin mostrar ningún
mensaje. En Linux, "no decir nada" casi siempre significa "todo salió bien".

### Paso 5: Comprobar sus grupos

```bash
groups dghm-1410751
```

```
groups   dghm-1410751
└──┬─┘   └─────┬────┘
   │           de qué usuario
   "grupos": muestra a qué grupos pertenece un usuario
```

**Qué está pasando:** consulta la lista de grupos del sistema. Solo **lee**
información, no cambia nada; por eso **no necesita `sudo`**.

**Qué debes ver:** una línea como `dghm-1410751 : dghm-1410751 sudo users` (los grupos
exactos pueden variar). Lo importante: **en la lista debe aparecer `sudo`**. El grupo
con tu mismo nombre es el que `adduser` creó en el paso 3.

> 📸 **Captura 2:** el final de `sudo adduser` y la salida de `groups`, donde se vea
> `sudo`.

### Paso 6: Entrar con tu usuario nuevo

```bash
su - dghm-1410751
```

```
su   -   dghm-1410751
└┬┘  │   └─────┬────┘
 │   │         a qué usuario te cambias
 │   el guion: entra como si iniciaras sesión desde cero (en su carpeta personal)
 "substitute user": cambiar de usuario
```

**Qué está pasando:** cambias de cuenta **sin cerrar tu sesión**: desde este momento
eres tu usuario nuevo (Kerrisk, s.f.-a). El guion `-` hace que sea una **sesión
nueva**: entras a la carpeta personal del usuario nuevo y el sistema carga sus datos
desde cero, **incluyendo el grupo `sudo`** que le acabas de agregar. Sin una sesión
nueva, un usuario no "ve" los grupos que le agregaron después de entrar.

**Qué debes hacer:** te pide la contraseña **del usuario nuevo**, la que elegiste en el
paso 3.

**Qué debes ver:** el **prompt cambia**. Ahora empieza con tu usuario nuevo:
`dghm-1410751@...:~$`.

⚠️ **A partir de aquí, todo lo haces con tu usuario nuevo.** No escribas `exit` hasta
que la práctica te lo indique.

### Paso 7: Preguntar quién eres

```bash
whoami
```

**Qué significa:** *who am I?*, "¿quién soy?". Muestra el nombre del usuario con el
que estás trabajando en este momento.

**Qué debes ver:** tu usuario nuevo, `dghm-1410751`. Confirma que el `su` del paso 6
funcionó.

### Paso 8: Comprobar que eres administrador

```bash
sudo whoami
```

```
sudo   whoami
└┬─┘   └──┬─┘
 │        ¿quién soy?
 ejecútalo como administrador
```

**Qué está pasando:** es la misma pregunta del paso 7, pero ahora con `sudo`. Si tu
usuario nuevo de verdad es administrador, `sudo` acepta ejecutar el comando **como
`root`** (la cuenta de administrador del sistema), y entonces la respuesta a "¿quién
soy?" es `root`.

**Qué debes hacer:** `sudo` te pide la contraseña de **tu usuario nuevo** (con
asteriscos, como en el paso 1). Ojo: `sudo` siempre pide la contraseña **de quien lo
está usando**, nunca la de `root`.

**Qué debes ver:** **`root`**. Si en cambio aparece un mensaje de que tu usuario no
tiene permiso para usar `sudo`, revisa el paso 4: probablemente escribiste mal el
nombre del usuario o faltó la `a` de `-aG`.

> 📸 **Captura 3:** el prompt con tu usuario nuevo, la salida de `whoami` y la de
> `sudo whoami` respondiendo `root`.

---

## Parte 3: Instalar nginx (servidor web)

Un **servidor web** es un programa que espera peticiones de los navegadores y les
responde con páginas. Las peticiones llegan por el protocolo **HTTP**, al **puerto 80**.
**nginx** (se pronuncia "en-yin-ex") es uno de los servidores web más usados del mundo
(F5, s.f.). Es el mismo tipo de servicio que viste en la lectura de
[WWW](../../contenido/etapa-2-servicios-red/04-www.md) de la Etapa 2.

### Paso 9: Instalar nginx

```bash
sudo apt install nginx -y
```

```
sudo   apt   install   nginx   -y
└┬─┘   └┬┘   └──┬──┘   └─┬─┘   └┬┘
 │      │       │        │      responde "sí" automáticamente
 │      │       │        qué paquete: nginx
 │      │       qué quieres hacer: instalar
 │      el programa que maneja los paquetes
 como administrador
```

**Qué está pasando:** `apt` busca `nginx` en la lista que descargaste en el paso 1,
calcula qué otros paquetes necesita (sus **dependencias**), los descarga e instala
todo. Además, en Ubuntu, al instalar nginx el servicio **se enciende solo** y queda
configurado para arrancar cada vez que se prenda el servidor (Canonical Ltd., s.f.-a).

**Qué debes ver:** líneas de descarga e instalación, y luego el prompt. Si ya lo habías
instalado en la práctica de
[acceso remoto](../../contenido/etapa-3-sistema-operativo-red/03-acceso-remoto/04-servidor-web.md),
`apt` no instala nada (en el resumen dice `Instalando 0`), porque ya lo tienes. Está
bien: sigue al siguiente paso.

### Paso 10: Comprobar que el servicio está funcionando

```bash
systemctl status nginx --no-pager
```

```
systemctl   status   nginx   --no-pager
└───┬───┘   └──┬─┘   └─┬─┘   └────┬───┘
    │          │       │          muestra todo de una vez y regresa al prompt
    │          │       de qué servicio
    │          qué quieres saber: su estado
    el programa que controla los servicios del servidor
```

**Qué está pasando:** un **servicio** es un programa que corre en segundo plano, sin
ventana, esperando trabajo. `systemctl` es el programa que los enciende, los apaga y
dice cómo están. Consultar el estado solo **lee** información, por eso no necesita
`sudo`. Sin `--no-pager`, la salida se abre en un visor y tendrías que presionar `q`
para salir.

**Qué debes ver:** busca la línea que empieza con `Active`. Debe decir **`active
(running)`**, "activo, funcionando", seguida de desde cuándo está encendido. Si dijera
`inactive` o `failed`, el servicio no está funcionando.

> 📸 **Captura 4:** la salida de `systemctl status nginx --no-pager` con
> `active (running)`.

### Paso 11: Escribir tu propia página

```bash
echo "<h1>Servidor de dghm-1410751</h1>" | sudo tee /var/www/html/index.html
```

Este es el comando más largo de la práctica. Escríbelo con cuidado, **cambiando
`dghm-1410751` por tu usuario**. Tiene dos partes unidas por una barra vertical `|`:

```
echo "<h1>Servidor de dghm-1410751</h1>"   |   sudo tee /var/www/html/index.html
└────────────────┬─────────────────────┘   │   └───────────────┬───────────────┘
                 │                         │   2. "tee" toma ese texto y lo guarda
                 │                         │      en el archivo, como administrador
                 │                         la "tubería": pasa el texto de la
                 │                         izquierda al comando de la derecha
                 1. "echo" produce este texto
```

- **`echo`** ("eco") repite el texto que le das. Aquí, una línea de **HTML**, el
  lenguaje de las páginas web: `<h1>` y `</h1>` indican que es un **título grande**.
- **`|`** es una **tubería** (*pipe*): en lugar de mostrar el texto en pantalla, se lo
  pasa al siguiente comando.
- **`tee`** recibe el texto, lo **guarda en el archivo** que le indicas y además lo
  muestra en pantalla (Free Software Foundation, s.f.). Su nombre viene de la pieza de
  plomería en forma de **T**, que divide el agua en dos caminos.
- **`/var/www/html/index.html`** es el archivo que nginx entrega cuando alguien visita
  tu servidor. `/var/www/html` es la carpeta de las páginas web, e `index.html` es la
  página principal.

⚠️ **¿Por qué `sudo tee` y no algo más sencillo?** La carpeta `/var/www/html` le
pertenece a `root`, y los demás usuarios no pueden escribir en ella. `sudo tee` es
quien escribe el archivo, **como administrador**, así que no hace falta cambiar ningún
permiso. Si escribieras `sudo echo "..." > /var/www/html/index.html`, verías
**Permission denied**: con `>`, quien abre el archivo para escribir es **tu usuario**,
no `sudo`.

**Qué debes ver:** el mismo texto, `<h1>Servidor de dghm-1410751</h1>`, repetido en la
pantalla. Es `tee` mostrando lo que guardó.

### Paso 12: Visitar tu página desde el servidor

```bash
curl localhost
```

```
curl   localhost
└┬─┘   └───┬───┘
 │         a qué servidor: "localhost" = este mismo servidor
 programa que pide una página web y muestra lo que responde
```

**Qué está pasando:** `curl` hace lo mismo que un navegador: manda una **petición
HTTP** y recibe la respuesta. La diferencia es que no dibuja la página: muestra el
texto tal cual llega. **`localhost`** es un nombre especial que siempre significa
"este mismo equipo". Como no indicaste puerto, `curl` usa el **80**, el de HTTP, que
es justo donde nginx está escuchando.

Así que este comando es tu servidor **visitándose a sí mismo**: `curl` le pide la
página a nginx, nginx lee `/var/www/html/index.html` y se la entrega.

**Qué debes ver:** **tu página**: `<h1>Servidor de dghm-1410751</h1>`, con tu usuario.
Eso confirma tres cosas a la vez: nginx está encendido, está escuchando en el puerto
80 y está entregando el archivo que escribiste.

> 📸 **Captura 5:** la salida de `curl localhost` mostrando tu página con tu usuario.

---

## Parte 4: Instalar SFTP

**SFTP** (*SSH File Transfer Protocol*, "protocolo de transferencia de archivos por
SSH") sirve para **subir y bajar archivos** de un servidor. A diferencia de FTP, que
viste en la [Etapa 2](../../contenido/etapa-2-servicios-red/06-ftp.md) y manda todo
(incluso las contraseñas) **sin cifrar**, SFTP viaja **dentro de SSH**, así que todo va
cifrado.

SFTP no es un programa aparte: **viene incluido en el servidor SSH**, cuyo paquete se
llama `openssh-server` (OpenBSD, s.f.-a). Por eso, para tener SFTP se instala ese paquete
y se comprueba que el servicio está activo.

### Paso 13: Instalar el servidor SSH (con SFTP)

```bash
sudo apt install openssh-server -y
```

```
sudo   apt   install   openssh-server   -y
└┬─┘   └┬┘   └──┬──┘   └──────┬─────┘   └┬┘
 │      │       │             │          responde "sí" automáticamente
 │      │       │             el paquete del servidor SSH, que incluye SFTP
 │      │       instalar
 │      el programa que maneja los paquetes
 como administrador
```

**Qué está pasando:** es el mismo tipo de comando que el paso 9, con otro paquete.

**Qué debes ver:** lo más probable es que `apt` **no instale nada**: al final, en el
resumen, dice `Instalando 0`. Es porque `openssh-server` ya estaba instalado: lo
activaste en el
[Paso 11](../../contenido/etapa-3-sistema-operativo-red/01-instalacion-ubuntu-server/11-ssh.md)
de la instalación de Ubuntu. Está bien: significa que SFTP ya estaba disponible.

### Paso 14: Comprobar que el servicio está activo

```bash
systemctl status ssh.socket --no-pager
```

```
systemctl   status   ssh.socket   --no-pager
└───┬───┘   └──┬─┘   └────┬───┘   └────┬───┘
    │          │          │            todo de una vez, sin visor
    │          │          de qué servicio: el "vigilante" del puerto de SSH
    │          su estado
    el programa que controla los servicios
```

**Qué está pasando:** en Ubuntu, el servidor SSH no está encendido todo el tiempo. En
su lugar, un pequeño servicio llamado **`ssh.socket`** se queda **vigilando el puerto
22**, y en cuanto alguien se conecta, despierta al servidor SSH completo (Ubuntu
Discourse, 2022). Así se ahorran recursos cuando nadie está conectado. Por eso aquí se
revisa `ssh.socket` y no `ssh`.

**Qué debes ver:** en la línea `Active` debe decir **`active (listening)`**, "activo,
escuchando": el servidor está esperando conexiones en el puerto 22, tanto de SSH como
de SFTP. Si en ese momento hay alguien conectado por SSH (por ejemplo, tú desde Git
Bash), dice **`active (running)`**; las dos significan que está funcionando. Más abajo
aparece `Listen:` con el puerto `22`. Al final puede salir el aviso `Warning: some
journal files were not opened due to insufficient permissions`: solo significa que,
sin `sudo`, no te deja ver todo el historial del servicio. No es un error.

#### Si dice `failed`

Si en la línea `Active` aparece **`failed`** en rojo (por ejemplo, `failed (Result:
service-start-limit-hit)`), el servidor SSH no pudo arrancar. Para saber por qué,
pídele a SSH que revise su configuración:

```bash
sudo sshd -t
```

Si responde **`sshd: no hostkeys available`**, a tu servidor le faltan sus **llaves de
identidad** (*host keys*): son las que forman la "huella" que el cliente te pide
aceptar la primera vez que te conectas por SSH. Sin ellas, SSH se niega a arrancar.
Créalas con:

```bash
sudo ssh-keygen -A
```

`ssh-keygen` genera llaves, y `-A` (de *all*) crea **todas** las llaves de servidor que
falten (OpenBSD, s.f.-b). Después, pídele a systemd que olvide los intentos fallidos y
vuelva a encender el servicio:

```bash
sudo systemctl reset-failed ssh.socket ssh.service && sudo systemctl restart ssh.socket
```

Repite el paso 14: ahora debe decir `active (listening)`. Si `sshd -t` muestra otro
error, avísale a tu profesor con una captura.

### Paso 15: Encontrar dónde se activa SFTP

```bash
grep -i sftp /etc/ssh/sshd_config
```

```
grep   -i   sftp   /etc/ssh/sshd_config
└┬─┘   └┬┘  └┬─┘   └─────────┬────────┘
 │      │    │               en qué archivo: la configuración del servidor SSH
 │      │    qué palabra buscar
 │      sin importar mayúsculas o minúsculas
 busca texto dentro de un archivo y muestra las líneas que lo contienen
```

**Qué está pasando:** toda la configuración del servidor SSH está en un archivo de
texto, `/etc/ssh/sshd_config` (la `d` final de `sshd` viene de *daemon*, la palabra que
se usa en Linux para los servicios). En lugar de abrir el archivo completo, `grep` te
muestra **solo las líneas** que contienen la palabra `sftp`.

**Qué debes ver:** una línea que empieza con **`Subsystem sftp`**. Esa línea le dice al
servidor SSH: "cuando un cliente pida SFTP, usa este programa". Es la prueba de que
SFTP está activado dentro de tu servidor SSH (OpenBSD, s.f.-a).

> 📸 **Captura 6:** el prompt con tu usuario nuevo, la salida de
> `systemctl status ssh.socket --no-pager` con `active (listening)` (o
> `active (running)`) y la línea
> `Subsystem sftp`.

### Paso 16: Regresar a tu usuario de siempre

```bash
exit
```

**Qué significa:** "salir". Cierra la sesión de tu usuario nuevo que abriste con `su`
en el paso 6, y regresas a tu usuario de siempre. Fíjate cómo el prompt vuelve a
mostrar tu nombre anterior.

**Terminaste la práctica.** Tu servidor ahora tiene:

- Sus programas **actualizados**.
- **Tu propia cuenta de administrador**.
- Un **servidor web** entregando tu página.
- Un servicio de **transferencia de archivos cifrada** listo para recibir conexiones.

---

## Entregable

Un solo archivo **PDF** con las **6 capturas**, en orden:

1. Pega las capturas en un documento de Word, una debajo de otra, cada una con su
   título (*Captura 1: actualización*, *Captura 2: usuario y grupo sudo*, etc.).
2. Guárdalo como PDF: **Archivo → Guardar como → PDF**.

Revisa antes de entregar que **en las capturas 3 a 6 se vea tu usuario nuevo** en el
prompt o en la salida de los comandos.

---

## Forma de entrega

- **Modalidad:** Individual. Cada alumno entrega su propio archivo, de su propia VM.
- **Plataforma:** NEXUS. Carga tu archivo en la evidencia *"3.3 - Evidencia de
  aprendizaje 3: Exposición por equipos del Servidor Virtualizado"*.
- Nombra tu archivo de la siguiente forma:

```
EV3_ApellidoNombre.pdf
```

*Ejemplo: `EV3_PerezJuan.pdf`*

- **Fecha de entrega:** Martes 20 de octubre de 2026.
- **Hora límite:** 7:40 p.m.

---

## Rúbrica de evaluación

| CRITERIO / NIVEL DE DOMINIO | Evidencia Completa | Evidencia Suficiente | Evidencia Débil | Sin evidencia |
|---|---|---|---|---|
| **Conocimientos** — Identifica las necesidades y requerimientos de su contexto para poder aplicar su conocimiento en el ámbito de las TI | **3** — Están las 6 capturas. El usuario sigue el formato `iniciales-matricula`, pertenece al grupo `sudo` y aparece en el prompt de las capturas 3 a 6. | **2** — Faltan 1 o 2 capturas, o el usuario no sigue el formato `iniciales-matricula`. | **1** — Faltan 3 o más capturas, o las capturas no muestran el usuario nuevo. | **0** — No trabajó. |
| **Habilidades** — Experimenta problemáticas reales en un ambiente simulado para detectar las posibles soluciones y fallas que se pueden presentar | **5** — Todo funciona: el servidor se actualizó, `sudo whoami` responde `root`, nginx está `active (running)` y sirve la página con su usuario, y el servicio SSH/SFTP está `active (listening)` (o `active (running)`) con la línea `Subsystem sftp`. | **4** — Una de las partes no funciona o no se comprueba (por ejemplo, nginx no sirve su página, o el servicio SSH/SFTP no aparece activo). | **3** — Dos o más partes no funcionan o no se comprueban. | **0** — No trabajó. |
| **Actitudes / Valores** — Participa y colabora de forma proactiva en la resolución de problemáticas de red computacional | **2** — Se entrega en la fecha y hora indicadas, en un solo PDF con el nombre `EV3_ApellidoNombre.pdf` y las capturas en orden y legibles. | **1** — Se entrega un día después de la fecha y hora indicadas. | **0** — Se entrega dos o tres días después de la fecha y hora indicadas. | **0** — No trabajó. |

*Se respetan los criterios, niveles y puntos de la rúbrica de NEXUS; solo se adaptó el
contenido de cada nivel a esta práctica.*

---

## Referencias

Canonical Ltd. (s.f.-a). *How to install nginx*. Ubuntu Server documentation.
https://documentation.ubuntu.com/server/how-to/web-services/install-nginx/

Canonical Ltd. (s.f.-b). *Package management*. Ubuntu Server documentation.
https://documentation.ubuntu.com/server/how-to/software/package-management/

Canonical Ltd. (s.f.-c). *User management*. Ubuntu Server documentation.
https://documentation.ubuntu.com/server/how-to/security/user-management/

Debian. (s.f.). *adduser.conf(5): configuration file for adduser*. Debian Manpages.
https://manpages.debian.org/testing/adduser/adduser.conf.5.en.html

F5, Inc. (s.f.). *nginx documentation*. https://nginx.org/en/docs/

Free Software Foundation. (s.f.). *GNU Coreutils: tee invocation*.
https://www.gnu.org/software/coreutils/manual/html_node/tee-invocation.html

Kerrisk, M. (s.f.-a). *su(1): Linux manual page*. The Linux man-pages project.
https://man7.org/linux/man-pages/man1/su.1.html

Kerrisk, M. (s.f.-b). *usermod(8): Linux manual page*. The Linux man-pages project.
https://man7.org/linux/man-pages/man8/usermod.8.html

OpenBSD. (s.f.-a). *sshd_config(5): OpenSSH daemon configuration file*.
https://man.openbsd.org/sshd_config

OpenBSD. (s.f.-b). *ssh-keygen(1): OpenSSH authentication key utility*.
https://man.openbsd.org/ssh-keygen

Ubuntu Discourse. (2022). *SSHd now uses socket-based activation (Ubuntu 22.10 and
later)*. https://discourse.ubuntu.com/t/sshd-now-uses-socket-based-activation-ubuntu-22-10-and-later/30189
