# Primer inicio de sesión

> Primeros pasos 1 · ≈50 min · [← Instalación y reinicio](../01-instalacion-ubuntu-server/13-instalacion.md) ·
> Siguiente: [Usuarios](02-usuarios.md)

## Qué vas a hacer

Tu servidor ya está instalado y se reinició. En esta práctica vas a **entrar por
primera vez**, a comprobar que la instalación quedó bien y a **actualizarlo**. También
vas a preparar el documento donde vas a ir guardando tus capturas durante todos los
primeros pasos.

## Paso 1: Iniciar sesión

Después de reiniciar, en la ventana de la VM pasan varias líneas de texto muy rápido:
es el sistema arrancando. No tienes que leerlas. Al final, la pantalla se detiene en
esto:

```
admredes login:
```

`admredes` es el nombre que le pusiste a tu servidor en el
[Paso 9](../01-instalacion-ubuntu-server/09-perfil.md) de la instalación, y `login`
significa "iniciar sesión". El servidor te está preguntando **quién eres**.

1. Escribe tu **nombre de usuario**, el que elegiste en el Paso 9, y presiona
   **Enter**.
2. Aparece `Password:` (contraseña). Escribe tu contraseña y presiona **Enter**.

**Mientras escribes la contraseña no aparece nada en pantalla**, ni puntos ni
asteriscos. No es que el teclado no funcione: es una medida de seguridad de Linux,
para que nadie que vea tu pantalla sepa cuántas letras tiene tu contraseña. El sistema
sí está recibiendo cada tecla. Escríbela completa, con cuidado, y presiona Enter.

**Si aparece `Login incorrect`** (inicio de sesión incorrecto), el usuario o la
contraseña tienen un error. El servidor vuelve a preguntar `login:`: inténtalo de
nuevo, despacio. Revisa que no tengas activadas las **mayúsculas** (Bloq Mayús), porque
en Linux una `A` y una `a` son letras distintas.

Cuando entras, aparecen varias líneas de bienvenida con información del sistema y, al
final, una línea como esta:

```
usuario@admredes:~$
```

En lugar de `usuario` debe decir tu nombre de usuario. A esa línea se le llama
**prompt**: significa que el servidor está **listo para recibir un comando**. Qué
significa cada parte lo vas a ver en la lectura de [Usuarios](02-usuarios.md).

## Paso 2: Preparar tus capturas

Durante todos los primeros pasos vas a tomar **capturas de pantalla** como evidencia de
tu trabajo. Cada vez que el texto muestre un recuadro **📸 Captura**, tomas una captura
y la guardas como un archivo de imagen con el nombre que ahí se indica. Tu profesor te
dirá más adelante cómo entregarlas: por ahora, **solo ve guardándolas**.

### Cómo tomar y guardar las capturas

1. **Crea una carpeta** en tu computadora (no en la VM), por ejemplo en Documentos, que
   se llame `capturas-admredes`. Ahí vas a guardar todas.
2. **Toma la captura con VirtualBox.** En el menú de la ventana de tu VM, elige
   **Ver → Tomar captura de pantalla...** (en inglés, *View → Take Screenshot...*).
   VirtualBox guarda **solo la pantalla de la VM**, sin nada más de tu computadora, y
   te abre una ventana para elegir dónde guardarla y con qué nombre.
3. **Guárdala con el nombre que indica el recuadro**, en tu carpeta
   `capturas-admredes`. Todos los nombres siguen la misma forma:

   ```
   NombreApellido-PrimerInicio.png
   └─────┬──────┘ └─────┬────┘ └┬┘
         │              │       el tipo de archivo: imagen PNG
         │              qué muestra la captura (lo indica cada recuadro)
         tu nombre y tu primer apellido
   ```

   Cambia `NombreApellido` por **tu** nombre y apellido, juntos, **sin espacios y sin
   acentos**, empezando cada uno con mayúscula. Por ejemplo, si te llamas Juan Pérez,
   la primera captura se llama `JuanPerez-PrimerInicio.png`.
4. **En cada captura debe verse el prompt con tu nombre de usuario.** Así se comprueba
   que la hiciste tú, en tu propio servidor.

Si el menú de VirtualBox no te aparece, también puedes usar la herramienta de tu
computadora (en Windows, **Windows + Shift + S**; en Mac, **Cmd + Shift + 4**),
seleccionar solo la ventana de la VM y guardar la imagen con el mismo nombre.

Si el mouse se queda "atrapado" dentro de la ventana de la VM, presiona la tecla
**Ctrl derecha** del teclado para liberarlo.

Antes de cada captura puedes escribir `clear` (limpiar) y presionar Enter: borra todo
lo que hay en la pantalla, para que en la captura solo se vea lo que importa.

## Paso 3: Comprobar que la instalación quedó bien

Vas a escribir cuatro comandos. Cada uno revisa algo que configuraste durante la
instalación. Escribe cada comando **exactamente igual**, con sus espacios y guiones, y
presiona **Enter** después de cada uno.

### El nombre y la versión del servidor

```bash
hostnamectl
```

`hostnamectl` (de *hostname control*, "control del nombre del equipo") muestra los
datos principales del servidor. Busca estas líneas:

| Línea | Qué significa | Qué debe decir |
|---|---|---|
| `Static hostname` | El nombre del servidor | `admredes` |
| `Operating System` | El sistema operativo instalado | `Ubuntu 26.04 LTS` |
| `Architecture` | La arquitectura del procesador | `arm64` o `x86-64`, según el ISO que usaste |

> 📸 **Captura `NombreApellido-PrimerInicio.png`:** la salida de `hostnamectl`.

### La dirección IP

```bash
ip a
```

`ip a` (de *ip address*, "dirección IP") muestra las tarjetas de red del servidor y su
dirección IP. Aparecen dos:

- `lo`: es una tarjeta "falsa" que el servidor usa para hablar consigo mismo. Tiene la
  dirección `127.0.0.1`. Ignórala.
- La otra tiene un nombre como `enp0s...`, y es la tarjeta de red de la VM. Busca la
  línea que empieza con `inet`: si tu VM está en modo **NAT**, como la configuramos,
  la dirección es `10.0.2.15`, que le asignó VirtualBox.

### La conexión a Internet

```bash
ping -c 3 google.com
```

`ping` manda tres mensajes cortos a Google y espera la respuesta (la opción `-c 3`
significa "manda 3", *count*). Deben aparecer tres líneas que dicen `bytes from...` y
al final un resumen con **`0% packet loss`** ("0 % de paquetes perdidos"). Eso
confirma dos cosas: que hay Internet y que el **DNS** funciona, porque el servidor pudo
convertir el nombre `google.com` en una dirección IP.

### El servicio SSH

En el [Paso 11](../01-instalacion-ubuntu-server/11-ssh.md) de la instalación
activaste el servidor SSH, que más adelante te va a permitir conectarte a tu servidor
desde otra computadora. Comprueba que está esperando conexiones:

```bash
systemctl status ssh.socket
```

`systemctl` es el programa que controla los **servicios** del servidor, y `status`
pide su estado. Busca la línea `Active`: debe decir **`active (listening)`**, que
significa "activo, escuchando". En Ubuntu, el servicio SSH completo no arranca hasta
que alguien se conecta: mientras tanto, `ssh.socket` se queda vigilando el puerto 22
para despertarlo cuando haga falta (Ubuntu Discourse, 2022).

Es posible que la pantalla no regrese sola al prompt y abajo diga `(END)` o `lines
1-10`. Es porque `systemctl` muestra la información como un documento que se puede
recorrer con las flechas. **Para salir, presiona la tecla `q`** (de *quit*, "salir").

## Paso 4: Actualizar el servidor

El ISO con el que instalaste se creó hace semanas o meses, y desde entonces Ubuntu ya
publicó correcciones. Lo primero que se hace con un servidor nuevo es **actualizarlo**:

```bash
sudo apt update && sudo apt upgrade
```

Qué significa cada palabra de este comando lo vas a ver con calma en la lectura de
[El administrador, sudo y los paquetes](03-administrador-y-paquetes.md). Por ahora:

1. Te pide **tu contraseña**. Recuerda que no se ve al escribirla. Escríbela y presiona
   Enter.
2. Aparecen muchas líneas: el servidor está descargando la lista de actualizaciones.
3. Si hay actualizaciones, te muestra la lista y pregunta si deseas continuar
   (`[S/n]` o `[Y/n]`). Presiona **Enter** para aceptar.
4. Espera a que regrese el prompt. Puede tardar varios minutos, según tu Internet.

Si al final aparece una ventana de color preguntando qué servicios reiniciar, deja las
opciones como vienen y presiona **Enter**.

## Paso 5: Apagar el servidor

Cuando termines de trabajar, **no cierres la ventana de VirtualBox con la X** sin más:
sería como desconectar una computadora de la corriente. Apágala correctamente:

1. En el menú de la ventana de la VM, elige **Máquina → Apagado ACPI** (en inglés,
   *Machine → ACPI Shutdown*).
2. El servidor cierra sus programas en orden y se apaga solo. La ventana se cierra
   cuando termina.

**ACPI** es el mismo aviso que recibe una computadora cuando presionas su botón de
encendido una vez: le pide que se apague de forma ordenada.

## Comandos que usaste

```
hostnamectl                    nombre, versión y arquitectura del servidor
ip a                           tarjetas de red y dirección IP
ping -c 3 <sitio>              comprobar Internet y DNS
systemctl status <servicio>    estado de un servicio (sal con q)
sudo apt update && sudo apt upgrade   actualizar el servidor
clear                          limpiar la pantalla
```

## Referencias

Ubuntu Discourse. (2022). *SSHd now uses socket-based activation (Ubuntu 22.10 and
later)*. https://discourse.ubuntu.com/t/sshd-now-uses-socket-based-activation-ubuntu-22-10-and-later/30189
