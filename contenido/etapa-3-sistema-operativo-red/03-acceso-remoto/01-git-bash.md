# Instalar Git Bash: una terminal en tu computadora

> Acceso remoto 1 · ≈20 min · [← Práctica: archivos y permisos](../02-primeros-pasos/06-practica-archivos.md) ·
> Siguiente: [NAT: muchos equipos, una sola dirección pública](02-que-es-nat.md)

## Qué vas a hacer

Hasta ahora has escrito todos los comandos **dentro de la ventana de VirtualBox**, como
si estuvieras sentado frente al servidor. Así no se trabaja con un servidor real: los
servidores suelen estar en otro cuarto, en otro edificio o en otra ciudad, sin pantalla
ni teclado conectados. El administrador los controla **desde su propia computadora**,
por la red, con **SSH**, el servicio que activaste en el
[Paso 11](../01-instalacion-ubuntu-server/11-ssh.md) de la instalación.

Para conectarte por SSH necesitas una **terminal en tu computadora** (no en la VM) que
tenga el programa `ssh`. En esta práctica vas a instalar **Git Bash**, y en la siguiente
la vas a usar para entrar a tu servidor.

```
Tu computadora (Windows)                    Tu servidor (VM)
┌──────────────────────────┐               ┌──────────────────────┐
│ Git Bash                 │   SSH (red)   │ Ubuntu Server        │
│ $ ssh usuario@IP  ───────┼──────────────>│ servidor SSH         │
│                          │               │ usuario@admredes:~$  │
└──────────────────────────┘               └──────────────────────┘
   aquí escribes                              aquí se ejecuta
```

**Si tu computadora es una Mac**, no instales nada: la app **Terminal** (búscala con
Cmd + Espacio) ya trae `ssh`. Salta al
[Paso 4](#paso-4-comprobar-que-tienes-ssh) y haz ahí la comprobación.

## ¿Qué es Git Bash?

**Git for Windows** es un paquete gratuito que instala en Windows el programa **Git**
(para guardar versiones de archivos, lo verás más adelante en tu carrera) y, junto con
él, **Git Bash**: una terminal que funciona **como la de Linux** (Git for Windows,
s.f.-a). Esto tiene dos ventajas:

- **Usa los mismos comandos que tu servidor**: `ls`, `pwd`, `cd`, `cat`... lo que
  aprendiste en los primeros pasos también funciona aquí, en tu computadora.
- **Ya incluye `ssh`**: el instalador trae su propia copia de **OpenSSH**, el mismo
  programa que corre en tu servidor, del lado del cliente (Git for Windows, s.f.-b).

## Paso 1: Descargar el instalador

1. En el navegador de tu computadora, entra a
   **[git-scm.com/install/windows](https://git-scm.com/install/windows)**.
2. Haz clic en **Click here to download**: descarga el instalador para equipos **x64**
   (Intel o AMD), el caso de casi todas las computadoras con Windows. Es un archivo con
   un nombre como `Git-2.56.0-64-bit.exe`; el número de versión puede ser otro
   (Git, s.f.).

Si tu computadora tiene procesador **ARM** (por ejemplo, Snapdragon), baja en esa misma
página hasta **Other Git for Windows downloads** y elige **Git for Windows/ARM64
Setup**. Es la misma revisión que hiciste para elegir el ISO de Ubuntu en
[Antes de empezar](../01-instalacion-ubuntu-server/README.md#antes-de-empezar-qué-iso-descargo).

## Paso 2: Instalar

1. Abre el archivo que descargaste. Si Windows pregunta si quieres permitir que la
   aplicación haga cambios, elige **Sí**.
2. El instalador tiene **muchas pantallas**. En casi todas basta con presionar
   **Next** (Siguiente) sin cambiar nada. Hay **una sola** que conviene cambiar y dos
   que vale la pena revisar:

| Pantalla | Qué hacer |
|---|---|
| *Choosing the default editor used by Git* | Cambia la opción por **Use the Nano editor by default**. Es el editor `nano` que ya usaste en tu servidor |
| *Choosing the SSH executable* | Deja **Use bundled OpenSSH** (viene marcada). Es la opción que instala el `ssh` que vas a usar |
| *Configuring the terminal emulator to use with Git Bash* | Deja **Use MinTTY** (viene marcada). Es la ventana donde vas a escribir |
| Todas las demás | **Next**, sin cambiar nada |

3. En la última pantalla presiona **Install** y espera a que termine. Al final,
   desmarca *View Release Notes* y presiona **Finish**.

El orden y el texto exacto de las pantallas pueden cambiar un poco entre versiones del
instalador. Si ves una que no está en la tabla, deja lo que viene marcado y presiona
**Next**.

**Si la computadora no te deja instalar** (por ejemplo, una de la escuela sin permisos
de administrador), avisa a tu profesor. La misma página de descarga ofrece una versión
**Portable** que no necesita instalarse.

## Paso 3: Abrir Git Bash

1. Abre el menú **Inicio** de Windows y escribe **Git Bash**.
2. Haz clic en **Git Bash**. Se abre una ventana negra con algo parecido a esto:

```
usuario@LAPTOP MINGW64 ~
$
```

Es un **prompt**, igual que el de tu servidor, pero de **tu computadora**:

| Parte | Qué es |
|---|---|
| `usuario` | Tu usuario **de Windows**, no el de tu servidor |
| `LAPTOP` | El nombre de **tu computadora** |
| `MINGW64` | Indica que es la terminal de Git Bash. Puedes ignorarlo |
| `~` | Tu carpeta personal de Windows (`C:\Users\usuario`) |
| `$` | La terminal espera un comando |

**Aprende a distinguir los dos prompts**: cuando veas `MINGW64` estás en tu
computadora; cuando veas `usuario@admredes` estás en tu servidor. En la siguiente
práctica vas a pasar de uno al otro.

## Paso 4: Comprobar que tienes ssh

Escribe:

```bash
ssh -V
```

La opción `-V` (con **V mayúscula**, de *version*) muestra la versión de `ssh` y
termina (OpenBSD, s.f.). Debe aparecer una línea que empieza con **`OpenSSH_`** seguida
de un número, por ejemplo:

```
OpenSSH_10.2p1, OpenSSL 3.5.4 30 Sep 2025
```

El número puede ser otro; lo importante es que diga `OpenSSH`. Si en cambio aparece
`command not found`, revisa que hayas escrito `-V` con mayúscula y que hayas abierto
**Git Bash** y no otra terminal.

Prueba también que los comandos de Linux funcionan aquí:

```bash
pwd
ls
```

`pwd` muestra tu carpeta personal de Windows escrita a la manera de Linux
(`/c/Users/usuario`), y `ls` lista lo que hay en ella: Documentos, Descargas,
Escritorio...

> 📸 **Captura `NombreApellido-GitBash.png`:** la ventana de Git Bash con la salida de
> `ssh -V`. Esta captura es de **tu computadora**, no de la VM: tómala con
> **Windows + Shift + S** (en Mac, **Cmd + Shift + 4**), selecciona solo la ventana de
> Git Bash y guárdala en tu carpeta `capturas-admredes`.

## Comandos que usaste

```
ssh -V     versión de ssh (comprueba que está instalado)
pwd        carpeta actual (también funciona en Git Bash)
ls         lista archivos (también funciona en Git Bash)
```

## Referencias

Git for Windows. (s.f.-a). *Git for Windows*. https://gitforwindows.org/

Git for Windows. (s.f.-b). *Using an external OpenSSH client*.
https://gitforwindows.org/using-an-external-openssh-client

Git. (s.f.). *Install for Windows*. https://git-scm.com/install/windows

OpenBSD. (s.f.). *ssh(1): OpenSSH remote login client* [Manual page].
https://man.openbsd.org/ssh
