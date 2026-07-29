# Etapa 0 — Fundamentos: Redes, Tráfico, Virtualización y Linux

> Teoría a exponer antes de comenzar la práctica. No se asume ningún conocimiento previo.

## Objetivo de la etapa

Que el alumno entienda, en términos simples, cuatro ideas que van a usar durante todo el
curso:

1. Qué es una red y de qué está hecha.
2. Qué es el "tráfico" que viaja por esa red.
3. Qué es una máquina virtual y cómo se crea una desde cero.
4. Qué es Linux y por qué lo vamos a usar constantemente.

---

## Bloque 1 — ¿Qué es una red?

### Definición simple

Una **red** es un conjunto de dispositivos conectados entre sí que pueden **intercambiar
información**. Eso es todo. Dos computadoras conectadas por un cable ya son una red.

### ¿De qué está hecha una red?

| Elemento | Qué es | Ejemplo cotidiano |
|---|---|---|
| **Dispositivo / host** | Cualquier equipo que participa en la red | Computadora, celular, impresora, servidor |
| **Medio** | Por dónde viaja la información | Cable de red (Ethernet), Wi-Fi, fibra óptica |
| **Switch** | Conecta varios dispositivos dentro de **una misma red local** | El "regulador de tránsito" de tu oficina |
| **Router** | Conecta **una red con otra red distinta** (por ejemplo, tu casa con Internet) | La puerta de salida de tu colonia hacia la carretera |

### Analogía: la red como un sistema de calles

- Los **dispositivos** son las casas.
- El **switch** es como las calles dentro de un mismo fraccionamiento.
- El **router** es la caseta de salida que conecta ese fraccionamiento con la carretera
  (otra red, o Internet).
- La **dirección IP** es la dirección postal de cada casa: sin ella, nadie sabe a dónde
  entregar algo.

### Tipos de red por tamaño

| Sigla | Nombre | Alcance | Ejemplo |
|---|---|---|---|
| **LAN** | Local Area Network | Un edificio, una oficina, una casa | La red de tu escuela |
| **WAN** | Wide Area Network | Ciudades, países | Internet es la WAN más grande que existe |

### El modelo cliente-servidor

Casi todo lo que vamos a hacer en el curso sigue este patrón:

- **Cliente**: el que **pide** algo (tu navegador, tu celular).
- **Servidor**: el que **responde** a esa petición (una computadora dedicada a atender
  esas peticiones, casi siempre prendida 24/7).

```
Cliente  ---- "quiero ver la página web.lab.local" ---->  Servidor
Cliente  <----------- "aquí está el contenido" ----------  Servidor
```

Todo el curso (DNS, DHCP, web, FTP, correo) son distintos "sabores" de este mismo patrón:
un cliente que pide, un servidor que responde.

---

## Bloque 2 — ¿Qué es el tráfico de red?

### Definición simple

**Tráfico** es toda la información que efectivamente viaja por la red en un momento dado.
No es un concepto abstracto: son datos reales moviéndose de un dispositivo a otro.

### La información no viaja completa: viaja en "paquetes"

Cuando mandas algo por la red (un mensaje, una foto, una página web), esa información
**se corta en pedacitos** llamados **paquetes**, se manda cada uno por separado, y se
vuelve a armar al llegar.

**Analogía:** es como mandar un libro por correo, pero en vez de mandar el libro completo
en una sola caja, lo mandas hoja por hoja, cada una en su propio sobre, numerada. El
cartero puede entregar los sobres en cualquier orden — el que recibe los vuelve a ordenar
usando el número de cada hoja.

### ¿Qué lleva un paquete?

| Parte | Qué contiene | Analogía |
|---|---|---|
| **Origen (IP origen)** | Quién lo mandó | El remitente en el sobre |
| **Destino (IP destino)** | A dónde va | La dirección en el sobre |
| **Datos** | El contenido real | La hoja del libro |
| **Protocolo** | Las "reglas del juego" para leer ese paquete | El idioma en que está escrita la carta |

### ¿Qué es un protocolo?

Un **protocolo** es un acuerdo de cómo deben "hablar" dos dispositivos para entenderse.
Ejemplos que vamos a ver en el curso: **HTTP/HTTPS** (páginas web), **FTP** (archivos),
**SMTP/IMAP/POP3** (correo), **DNS** (nombres), **DHCP** (asignación de direcciones).

Cada protocolo es como un idioma distinto — dos dispositivos que hablan HTTP se entienden
perfecto entre sí, aunque ninguno de los dos "sepa hablar" FTP.

### Ejemplo completo: qué pasa cuando entras a una página web

1. Escribes `www.ejemplo.com` en el navegador (eres el **cliente**).
2. Tu computadora no entiende nombres, solo direcciones IP — así que primero le pregunta
   a un servidor **DNS**: "¿cuál es la IP de www.ejemplo.com?" (esto se ve a fondo en
   Etapa 2).
3. El DNS responde con una IP, por ejemplo `93.184.216.34`.
4. Tu navegador manda una petición **HTTP/HTTPS** a esa IP pidiendo la página.
5. El **servidor web** en esa IP responde mandando el contenido de la página, cortado en
   **paquetes**.
6. Tu computadora recibe todos los paquetes, los reordena, y el navegador te muestra la
   página completa.

Todo esto pasa en milisegundos, y es exactamente lo que vamos a poder **ver en cámara
lenta** más adelante con herramientas como `tcpdump` (capturar tráfico) — por eso es
importante entender el concepto ahora, antes de ver la herramienta.

---

## Bloque 3 — Máquinas Virtuales (VM)

### ¿Qué es una máquina virtual?

Una **máquina virtual** es una computadora completa **simulada por software**, que corre
dentro de tu computadora real. Tiene su propio "disco duro" (en realidad es un archivo),
su propia memoria asignada, y puede tener instalado su propio sistema operativo —
totalmente aparte del sistema operativo de tu computadora real.

**Analogía:** si tu computadora es una casa, una VM es como construir otra casa completa
*dentro* de una habitación de la primera casa. Tiene sus propias paredes, su propia
puerta, su propia dirección — pero físicamente sigue estando dentro de la casa original.

### ¿Por qué usamos VMs en este curso?

- Podemos **romper cosas sin miedo**: si algo sale mal dentro de la VM, borras la VM y
  creas otra, tu computadora real nunca se ve afectada.
- Podemos **practicar instalar un sistema operativo de servidor** (Windows Server) sin
  necesitar una computadora física extra.
- Podemos tener **varias "computadoras" a la vez** (varias VMs) para simular una red
  completa, aunque solo tengas una laptop.

### El programa que usa las VMs: el hipervisor

Para crear y correr VMs necesitas un programa llamado **hipervisor**. En este curso
usaremos **VirtualBox** (gratuito, funciona en Windows/Mac/Linux). Alternativa equivalente:
VMware Workstation Player.

### Práctica guiada: instalar VirtualBox y crear tu primera VM

#### Paso 1 — Descargar e instalar VirtualBox

1. Ve a [virtualbox.org](https://www.virtualbox.org/) → Downloads.
2. Descarga el instalador para tu sistema operativo (Windows host / macOS host).
3. Ejecuta el instalador y deja todas las opciones por defecto → Next → Next → Install.
4. Si Windows te pide permiso para instalar un "adaptador de red virtual", acepta — es
   necesario para que la VM tenga red.
5. Al terminar, abre VirtualBox. Deberías ver una ventana vacía, sin ninguna VM todavía.

#### Paso 2 — Conseguir un sistema operativo para instalar

Vamos a usar **Ubuntu Desktop** (gratuito, ligero, ideal para practicar) como primer
sistema operativo dentro de la VM.

1. Ve a [ubuntu.com/download/desktop](https://ubuntu.com/download/desktop).
2. Descarga el archivo `.iso` (es la "imagen" del disco de instalación — como un CD, pero
   en un archivo).

#### Paso 3 — Crear la máquina virtual (el "hardware" vacío)

En VirtualBox:

1. Clic en **Nueva** (Machine → New).
2. **Nombre**: `mi-primera-vm`. VirtualBox detecta solo que es Linux/Ubuntu por el nombre.
3. **Memoria (RAM)**: asigna 2048 MB (2 GB) como mínimo. Nunca asignes más de la mitad de
   la RAM real de tu computadora.
4. **Disco duro**: elige "Crear un disco duro virtual ahora", tipo VDI, tamaño dinámico,
   20 GB. Esto crea un archivo en tu computadora real que actuará como el "disco" de la
   VM — pero solo ocupa espacio real conforme lo vayas usando.
5. Clic en **Crear**. Ya existe la VM, pero está vacía — es una computadora sin sistema
   operativo instalado, como una laptop nueva recién comprada.

#### Paso 4 — Conectar el ISO e iniciar la instalación

1. Selecciona tu VM en la lista → **Configuración → Almacenamiento**.
2. En el controlador IDE/SATA, clic en el ícono de disco vacío → elige el archivo `.iso`
   de Ubuntu que descargaste. Esto es el equivalente a "meter el CD de instalación".
3. Clic en **Iniciar** (el botón verde). La VM arranca desde el ISO, igual que una
   computadora física arrancaría desde un CD de instalación.

#### Paso 5 — Instalar el sistema operativo dentro de la VM

1. Elige idioma → **Instalar Ubuntu**.
2. Distribución de teclado → Continuar.
3. Tipo de instalación: deja la opción por defecto ("Borrar disco e instalar Ubuntu") —
   esto es seguro, porque el "disco" que se va a borrar es el archivo virtual que creamos
   en el Paso 3, **no** el disco real de tu computadora.
4. Zona horaria → Ciudad de México (o la que corresponda).
5. Crea tu usuario y contraseña dentro de la VM (independiente del usuario de tu
   computadora real).
6. Espera a que termine la instalación (varios minutos) y reinicia cuando lo pida.

**Al terminar, tienes una computadora Linux completa y funcional, corriendo dentro de tu
propia computadora.** Esto es exactamente lo que vas a repetir en Etapa 3, pero instalando
Windows Server en vez de Ubuntu, para un propósito distinto (servidor de red).

### VM vs Contenedor (adelanto)

Más adelante en el curso también usaremos **contenedores** (Docker) para practicar
servicios de red sin tener que instalar un sistema operativo completo cada vez. La
diferencia clave:

| | Máquina Virtual | Contenedor |
|---|---|---|
| ¿Qué simula? | Una computadora completa, con su propio sistema operativo | Solo un programa aislado, comparte el sistema operativo con tu computadora real |
| Peso | Pesada (GB), tarda minutos en arrancar | Ligera (MB), arranca en segundos |
| Cuándo lo usamos | Para instalar un SO real de principio a fin (Etapa 3) | Para practicar servicios de red rápido (Etapas 1, 2, 4) |

---

## Bloque 4 — Fundamentos de Linux

### ¿Qué es Linux?

**Linux** es un sistema operativo, igual que Windows o macOS — permite que tu computadora
funcione y te deja instalar y correr programas. La diferencia principal: Linux es
**gratuito y de código abierto** (cualquiera puede ver y modificar cómo funciona por
dentro), y por eso es el sistema operativo más usado del mundo en **servidores**.

### ¿Por qué Linux es tan importante en redes?

- La gran mayoría de los servidores del mundo (los que hospedan páginas web, correos,
  bases de datos) corren Linux, no Windows ni macOS.
- Las herramientas de redes más usadas (routers, firewalls, herramientas de diagnóstico
  como `ping`, `traceroute`, `tcpdump`) nacieron y viven en el mundo Linux.
- Docker, que usaremos en varias etapas, corre contenedores basados en Linux por dentro
  aunque tu computadora sea Windows o Mac.

### ¿Qué es una "distribución" (distro)?

Linux por sí solo es solo el "motor" (kernel). Una **distribución** es ese motor empacado
junto con programas, un instalador y una forma de administrarlo, listo para usar.
Ejemplos: **Ubuntu** (la que instalamos arriba, la más amigable para empezar), Debian,
CentOS, Fedora. Todas comparten los mismos comandos básicos.

### La terminal: cómo se controla Linux

A diferencia de Windows, en Linux (sobre todo en servidores) casi todo se controla
escribiendo comandos en una **terminal**, en lugar de hacer clic en ventanas. Esto no es
al revés de lo moderno — es al contrario: es más rápido, se puede automatizar, y es la
única forma de administrar un servidor remoto que no tiene ni pantalla ni mouse
conectados.

> El manejo detallado de la terminal (moverte entre carpetas, crear/editar archivos, ver
> procesos) se practica a fondo antes de Etapa 1, con ejercicios paso a paso.

### Idea clave para quedarnos con esto

No necesitas memorizar comandos todavía. Lo importante de este bloque es entender **por
qué** vamos a vivir dentro de una terminal Linux durante el resto del curso: porque así
es como se administra una red y sus servidores en el mundo real.

---

## Cierre de la etapa

Con estos cuatro bloques, el alumno ya tiene el vocabulario mínimo para todo lo que
sigue:

- Sabe qué es una red y el patrón cliente-servidor.
- Sabe que la información viaja en paquetes, siguiendo protocolos.
- Ya instaló y usó una máquina virtual de principio a fin.
- Entiende por qué el curso va a vivir dentro de una terminal Linux.

**Siguiente paso:** práctica guiada de terminal (comandos básicos) antes de entrar a
Etapa 1.
