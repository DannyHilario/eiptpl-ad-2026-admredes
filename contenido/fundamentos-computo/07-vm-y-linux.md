# Máquinas Virtuales y Linux

> 50 min · [← Tráfico](06-trafico.md) · Siguiente: [Terminal](08-terminal.md)

## A diferencia de las sesiones 5 y 6, esto es 100% nuevo

Tu curso anterior (Redes y Telecomunicaciones) fue Windows de principio a fin — nunca
instalaste una VM ni tocaste Linux. No hay repaso posible aquí porque no hay nada previo
que repasar; dale a esta sesión toda tu atención.

## Parte A — Máquinas Virtuales (≈20 min)

### ¿Qué es?

Una **máquina virtual** es una computadora completa **simulada por software**, corriendo
dentro de tu computadora real. Tiene su propio "disco duro" (un archivo), su propia
memoria asignada, y su propio sistema operativo — aparte del de tu computadora real.

```
Tu computadora real
┌─────────────────────────────────────────────┐
│ Hardware físico                              │
│         │                                    │
│         v                                    │
│ Tu sistema operativo (Windows/macOS)         │
│         │                                    │
│         v                                    │
│ Hipervisor (VirtualBox)                      │
│         │                                    │
│         v                                    │
│  ┌─────────────────────────────────────┐    │
│  │ Máquina Virtual                      │    │
│  │  Linux (Ubuntu) — instalado desde cero│    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

**Analogía:** si tu computadora es una casa, una VM es construir otra casa completa
*dentro* de una habitación de la primera. Paredes propias, puerta propia, dirección
propia — pero sigue estando físicamente dentro de la casa original.

### ¿Por qué usamos VMs?

- **Rompes cosas sin miedo**: si algo sale mal dentro de la VM, la borras y creas otra; tu
  computadora real nunca se ve afectada.
- **Practicas instalar un SO de servidor** (Windows Server) sin necesitar hardware extra.
- **Simulas varias "computadoras" a la vez** (varias VMs) aunque solo tengas una laptop —
  incluso puedes simular un fraccionamiento completo (switch + router + varias VMs) de la
  Sesión 5, todo dentro de tu laptop.

El programa que crea y corre VMs se llama **hipervisor**. Usaremos **VirtualBox**
(gratuito, Windows/Mac/Linux). Alternativa equivalente: VMware Workstation Player.

### VM vs Contenedor (adelanto)

| | Máquina Virtual | Contenedor (Docker) |
|---|---|---|
| ¿Qué simula? | Una computadora completa con su propio SO | Solo un programa aislado, comparte el SO anfitrión |
| Peso | Pesada (GB), arranca en minutos | Ligero (MB), arranca en segundos |
| Cuándo se usa en el curso | Instalar un SO real de principio a fin (Etapa 3) | Servicios de red rápido (Etapas 1, 2, 4) |

> ### ✅ Autocheck rápido
> **Pregunta:** borraste tu VM por error (o la rompiste a propósito practicando). ¿Qué le
> pasa a tu computadora real?
> **Respuesta:**
>
> Nada. La VM es un archivo (o un conjunto de archivos) dentro de tu computadora real; al
> borrarla solo desaparece ese archivo. Tu sistema operativo real, tus documentos y tus
> programas quedan intactos — esa es justo la razón de usar VMs para practicar.

---

## Qué vas a necesitar tener listo para esta sesión

Esta sesión es práctica (instalas VirtualBox y Ubuntu de verdad), así que conviene
saberlo con tiempo:

- Tu propia laptop, con permisos de administrador para instalar software (no una
  computadora de la escuela restringida).
- Al menos **10 GB libres de disco** (la VM y el ISO de Ubuntu ocupan espacio real).
- Conexión a Internet razonable para descargar VirtualBox (~150 MB) y Ubuntu Desktop
  (~5 GB) — mejor hacerlo con anticipación, no el mismo día de la sesión.
- La **virtualización debe estar habilitada en el BIOS** de tu laptop (Intel VT-x / AMD-V).
  En la mayoría de las laptops modernas ya viene activada por defecto; si VirtualBox se
  queja de esto al crear la VM, es lo primero que hay que revisar.

## Parte B — Práctica guiada: instalar VirtualBox y tu primera VM (≈20 min)

**Paso 1 — Instalar VirtualBox**

1. [virtualbox.org](https://www.virtualbox.org/) → Downloads → instalador de tu sistema.
2. Ejecuta con las opciones por defecto (Next → Next → Install).
3. Si Windows pide permiso para un "adaptador de red virtual", acepta — la VM lo necesita
   para tener red.

**Paso 2 — Conseguir el sistema operativo (Ubuntu Desktop)**

1. [ubuntu.com/download/desktop](https://ubuntu.com/download/desktop) → descarga el `.iso`
   (la "imagen" del disco de instalación, como un CD en un archivo).

**Paso 3 — Crear la VM (el "hardware" vacío)**

1. VirtualBox → **Nueva**.
2. Nombre: `mi-primera-vm` (VirtualBox detecta Linux/Ubuntu por el nombre).
3. RAM: 2048 MB mínimo. Nunca asignes más de la mitad de tu RAM real.
4. Disco: "Crear un disco duro virtual ahora", tipo VDI, dinámico, 20 GB — un archivo en tu
   disco real que solo ocupa espacio conforme lo uses.
5. **Crear**. Ya existe la VM, vacía — una laptop nueva sin sistema operativo.

**Paso 4 — Conectar el ISO e iniciar**

1. VM → **Configuración → Almacenamiento** → controlador IDE/SATA → disco vacío → elige el
   `.iso` de Ubuntu.
2. **Iniciar** — arranca desde el ISO, igual que una computadora física desde un CD.

**Paso 5 — Instalar el sistema operativo**

1. Idioma → **Instalar Ubuntu** → teclado → Continuar.
2. Tipo de instalación: deja la opción por defecto ("Borrar disco e instalar Ubuntu") — es
   seguro, el disco que se borra es el archivo virtual del Paso 3, **no** tu disco real.
3. Zona horaria → Ciudad de México (o la que corresponda).
4. Crea usuario y contraseña **dentro de la VM** (independiente de tu usuario real).
5. Espera a que termine y reinicia cuando lo pida.

Al terminar tienes una computadora Linux completa corriendo dentro de la tuya. Es lo mismo
que repetirás en Etapa 3 instalando Windows Server, para un propósito distinto (servidor de
red).

---

## A fondo: cómo se conecta tu VM a la red (NAT, Bridged, Host-only)

Tu VM no está flotando sin red — VirtualBox le da una tarjeta de red virtual, y tú decides
**cómo** esa tarjeta se conecta al mundo real. Esto conecta directo con switch/router de
la Sesión 5: cada modo es una forma distinta de resolver "¿en qué red local vive mi VM, y
cómo sale de ahí?"

VM → **Configuración → Red** en VirtualBox, ahí eliges el modo:

| Modo | ¿Qué hace tu VM? | Analogía (Sesión 5) |
|---|---|---|
| **NAT** (por defecto) | Comparte la IP de tu laptop real; el resto de tu red **no puede verla** ni conectarse a ella directamente | La VM vive en un cuarto extra de tu casa, con acceso a la calle pero sin dirección propia — el cartero solo conoce la dirección de tu casa |
| **Bridged (puente)** | La VM obtiene su **propia IP** dentro de tu misma red local, como si fuera otra laptop física conectada al mismo switch | La VM es una casa más en tu mismo fraccionamiento, con su propio número de casa |
| **Host-only** | La VM solo puede hablar con tu laptop real, **sin salida a Internet** | Un intercomunicador que solo conecta dos cuartos de la misma casa, sin puerta a la calle |

Para este curso, el modo por defecto (**NAT**) es suficiente para navegar y descargar
paquetes desde tu VM. Más adelante, cuando simules servidores que otros dispositivos de
tu red deban poder alcanzar, vas a necesitar **Bridged** — ahí es donde tu VM
literalmente se vuelve "otra casa" visible en tu red local, con su propia IP asignada por
tu router o DHCP.

> ### ✅ Autocheck rápido
> **Pregunta:** tienes tu VM en modo **NAT** y corres `ipconfig`/`ifconfig` dentro de
> ella. La IP que ves, ¿es la misma que verías si corrieras el comando en tu laptop real
> directamente?
> **Respuesta:**
>
> No. En modo NAT, VirtualBox le da a la VM una IP privada propia dentro de una red
> **virtual e interna** que solo existe entre tu laptop y la VM (típicamente algo como
> `10.0.2.15`) — distinta de la IP que tu laptop real tiene en tu red Wi-Fi/Ethernet. Por
> eso el resto de tu red no puede ver la VM directamente: para el mundo exterior, todo el
> tráfico de la VM sale disfrazado con la IP de tu laptop real, gracias a un NAT que hace
> el propio VirtualBox — el mismo concepto de NAT que viste en la Sesión 5, aplicado aquí
> por el hipervisor en vez de por tu router doméstico.

### Extra: snapshots — tu "guardado rápido" antes de experimentar

Antes de probar algo que podría romper tu VM (cambiar configuración de red, instalar algo
riesgoso), VirtualBox te deja tomar una **snapshot** (foto del estado exacto de la VM en
ese momento): VM → clic derecho → **Tomar instantánea**. Si algo sale mal, **Restaurar**
regresa la VM exactamente a como estaba, sin tener que reinstalar Ubuntu desde cero.

---

## Parte C — Fundamentos de Linux (≈10 min)

### ¿Qué es Linux?

**Linux** es un sistema operativo, igual que Windows o macOS. La diferencia clave: es
**gratuito y de código abierto** (cualquiera puede ver y modificar cómo funciona por
dentro), y por eso es el sistema operativo más usado del mundo en **servidores**.

### ¿Por qué importa tanto en redes?

- La mayoría de los servidores del mundo (web, correo, bases de datos) corren Linux, no
  Windows ni macOS.
- Las herramientas de red más usadas (`ping`, `traceroute`, `tcpdump`, routers, firewalls)
  nacieron y viven en el mundo Linux.
- Docker corre contenedores basados en Linux por dentro, aunque tu computadora sea Windows
  o Mac.

### ¿Qué es una distribución (distro)?

Linux por sí solo es solo el "motor" (kernel). Una **distribución** es ese motor empacado
con programas, instalador y forma de administrarlo, listo para usar. Ejemplos: **Ubuntu**
(la que acabas de instalar), Debian, CentOS, Fedora — todas comparten los mismos comandos
básicos.

### La terminal: cómo se controla Linux

A diferencia de Windows, en Linux (sobre todo en servidores) casi todo se controla
escribiendo comandos en una **terminal**, no haciendo clic en ventanas. No es lo contrario
de "moderno" — es al revés: es más rápido, se puede automatizar, y es la única forma de
administrar un servidor remoto sin pantalla ni mouse conectados.

> El manejo detallado de la terminal (moverte entre carpetas, crear/editar archivos, ver
> procesos) se practica a fondo antes de Etapa 1, con ejercicios paso a paso.

> ### ✅ Autocheck rápido
> **Pregunta:** ¿por qué no basta con memorizar comandos de Ubuntu si mañana te toca
> administrar un servidor con Debian o CentOS?
> **Respuesta:**
>
> Porque todas las distribuciones comparten el mismo kernel Linux y los mismos comandos
> básicos (`cd`, `ls`, `cat`, `ps`, etc.) — lo que aprendes en Ubuntu se transfiere casi
> intacto a cualquier otra distro. Lo que cambia entre distros es sobre todo cómo se
> instalan programas nuevos, no cómo se navega o se administra el sistema.

---

Ya instalaste tu VM con Ubuntu — falta el último paso antes de Etapa 1: dominar la
terminal que vive dentro de ella.

**Siguiente paso:** [Terminal](08-terminal.md).

---

## Referencias

Oracle Corporation. (s.f.). *Oracle VM VirtualBox user manual*.
https://www.virtualbox.org/manual/

Canonical Ltd. (s.f.). *Ubuntu desktop*. https://ubuntu.com/download/desktop
