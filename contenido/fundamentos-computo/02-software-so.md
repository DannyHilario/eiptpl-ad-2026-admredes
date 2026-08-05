# Software y el Sistema Operativo

> 50 min · [← Hardware](01-hardware.md) · Siguiente: [Interfaces de usuario: gráfica y línea de comandos](03-interfaces-usuario.md)

## ¿Qué es software?

**Software** es un conjunto de instrucciones escritas para que el hardware las ejecute.
No es un objeto físico — es información, código, guardado en el almacenamiento hasta que
se necesita usar. Seguimos con la analogía del escritorio de oficina: hoy hablamos de
quién da las instrucciones de trabajo.

Hay varias capas de software, no solo dos:

| Capa | Qué es | Ejemplo |
|---|---|---|
| **Firmware (BIOS/UEFI)** | Software mínimo grabado en un chip de la motherboard; corre antes que cualquier sistema operativo | UEFI de la motherboard |
| **Sistema Operativo (SO)** | Software base que administra todo el hardware y permite que los demás programas funcionen | Windows, macOS, Linux |
| **Drivers** | Traducen entre el SO y una pieza de hardware específica | Driver de la tarjeta gráfica, de la tarjeta de red |
| **Aplicaciones / Programas** | Software que corre *sobre* el sistema operativo, para tareas específicas | Chrome, Word, un videojuego |

### Firmware: lo primero que corre, antes que el sistema operativo

Al prender la computadora, antes de que Windows/macOS/Linux exista siquiera en la RAM,
corre el **firmware** de la motherboard: **BIOS** (Basic Input/Output System) en equipos
antiguos, o su sucesor moderno, **UEFI** (Unified Extensible Firmware Interface). Su
trabajo es revisar que el hardware básico responda (CPU, RAM, teclado) y luego buscar,
en el almacenamiento, un sistema operativo para entregarle el control. Es el paso previo
indispensable — sin firmware funcionando, el sistema operativo nunca llega a cargar.

## El Sistema Operativo: el gerente del escritorio

Si el hardware son los muebles del escritorio y las aplicaciones son instrucciones de
trabajo específicas, el **sistema operativo es el gerente de la oficina**: reparte los
recursos, decide quién usa qué en qué momento, y evita que dos programas choquen tratando
de usar lo mismo a la vez (Silberschatz et al., 2018). Concretamente:

### Gestión de procesos: multitarea real

Cada programa en ejecución es un **proceso**. El sistema operativo tiene un componente
llamado **planificador (scheduler)** que decide, muchas veces por segundo, a qué proceso
le toca usar cada núcleo del CPU en cada instante — por eso puedes tener Chrome, Word y
Spotify abiertos a la vez en un CPU con menos núcleos que programas abiertos: el
planificador les va dando turnos tan rápido que parecen simultáneos. Si tu CPU tiene
varios núcleos (Sesión 1), varios procesos sí pueden correr en paralelo real, uno por
núcleo, además de turnarse dentro de cada núcleo.

**Proceso vs. servicio — no son lo mismo, pero están relacionados.** Todo lo que corre en
la computadora es un proceso, pero hay un tipo particular de proceso llamado **servicio**
(en Windows) o **daemon** (en Linux — por convención terminan en "d": `sshd`, `httpd`)
que se distingue de un proceso normal en varias cosas (Silberschatz et al., 2018):

| | Proceso normal (ej. Chrome) | Servicio / daemon (ej. spooler de impresión) |
|---|---|---|
| ¿Tiene ventana? | Sí, visible en pantalla | No, corre en segundo plano sin interfaz |
| ¿Quién lo inicia? | Un usuario, con doble clic | El sistema, automáticamente al prender |
| ¿Depende de una sesión de usuario? | Sí, se cierra si cierras sesión | No, puede correr desde antes de iniciar sesión |
| ¿Cuándo termina? | Cuando el usuario lo cierra | Sigue esperando indefinidamente, listo para atender algo |
| ¿Quién lo administra? | El usuario abre/cierra la ventana | Un componente especial del SO |

Esta distinción importa mucho para el resto del curso: DNS, DHCP, un servidor web o de
correo son, técnicamente, **servicios** — programas sin ventana que corren en segundo
plano en un servidor, esperando peticiones de la red constantemente. Es exactamente lo
que vas a instalar y administrar en la Etapa 2 del manual oficial.

### Gestión de memoria: RAM + memoria virtual

El SO decide qué parte de la RAM usa cada proceso, y la libera cuando el proceso termina.
Cuando la RAM física no alcanza, el sistema operativo puede usar un espacio del
almacenamiento como "RAM de emergencia", llamado **memoria virtual** o **archivo de
paginación (pagefile)** en Windows — mucho más lento que la RAM real, pero evita que el
sistema se caiga por falta de memoria (Silberschatz et al., 2018).

### Gestión de archivos: sistemas de archivos

El SO organiza el almacenamiento en carpetas y archivos mediante un **sistema de
archivos** — una estructura específica para saber en qué parte física del disco está
cada dato. Windows usa principalmente **NTFS**; macOS usa **APFS**; Linux usa **ext4**,
entre otros. Por eso un disco formateado para Mac no siempre se lee directamente en
Windows sin software adicional: hablan "idiomas" de organización distintos.

### Gestión de dispositivos: drivers

Cada pieza de hardware (GPU, tarjeta de red, impresora) tiene su propio conjunto de
comandos e idiosincrasias. Un **driver** es el programa intermediario que le permite al
SO comunicarse con una pieza específica sin que el SO necesite saber los detalles
internos de cada fabricante. Cuando conectas una impresora nueva y Windows dice
"instalando dispositivo", está instalando su driver.

## Tipos de sistemas operativos

- **De escritorio:** Windows, macOS, Linux (Ubuntu, Debian...) — para computadoras
  personales.
- **Móviles:** Android, iOS — versiones adaptadas a hardware de celular (batería, pantalla
  táctil).
- **De servidor:** Windows Server, distribuciones Linux orientadas a servidor — sin
  interfaz gráfica pesada, optimizados para atender peticiones de red en vez de un
  usuario frente a una pantalla (relevante para el resto de este curso).

## Ejemplo completo: qué pasa cuando abres Chrome

```
Tú haces doble clic en el ícono de Chrome
        │
        v
El Sistema Operativo busca el programa Chrome en el Almacenamiento
        │
        v
El SO reserva un espacio de RAM y copia ahí las partes de Chrome que
necesita ejecutar ahora mismo (crea un nuevo proceso)
        │
        v
El planificador del SO le da turnos de tiempo de CPU a ese proceso
        │
        v
El driver de la GPU dibuja la ventana de Chrome en tu pantalla
        │
        v
Ves la ventana de Chrome abierta y lista para usar
```

Cuando cierras Chrome, el SO termina ese proceso y libera la RAM que había reservado.
Cada pestaña nueva que abres en Chrome, de hecho, suele ser **otro proceso aparte** — por
eso el Administrador de tareas muestra varias líneas de "Google Chrome" cuando tienes
varias pestañas abiertas.

---

## Ejercicio práctico: mira al Sistema Operativo trabajando en tiempo real (Windows 11)

**1. Procesos, y cuánto consume cada uno:**
`Ctrl + Shift + Esc` → pestaña **Procesos**. Ordena la lista haciendo clic en la columna
**Memoria** o **CPU** para ver qué programa consume más en este momento. Fíjate también
en la columna **Disco** y **Red** — el Administrador de tareas de Windows 11 mide las
cuatro cosas por proceso.

**2. Detalle de versión del sistema operativo:**
`Inicio` → **Configuración** → **Sistema** → **Acerca de** → sección "Especificaciones de
Windows": ahí ves la edición, versión y número de compilación exactos de tu SO.

**3. Memoria virtual / archivo de paginación:**
Busca "Ver información avanzada del sistema" en el buscador de Windows → pestaña
**Avanzado** → en "Rendimiento" clic en **Configuración** → pestaña **Avanzado** →
sección "Memoria virtual" → **Cambiar**. Ahí ves cuánto espacio del almacenamiento está
usando Windows como memoria virtual, además de tu RAM física.

**4. Drivers instalados:**
`Windows + X` → **Administrador de dispositivos**. Expande "Adaptadores de pantalla" (tu
GPU) o "Adaptadores de red", clic derecho sobre el dispositivo → **Propiedades** →
pestaña **Controlador** — ahí ves el proveedor y la versión exacta de ese driver.

**5. Sistema de archivos del disco:**
**Explorador de archivos** → **Esta PC** → clic derecho en (C:) → **Propiedades** — el
campo "Sistema de archivos" muestra `NTFS`.

**6. Servicios corriendo en segundo plano:**
`Windows + R` → escribe `services.msc` → Enter. Ahí ves la lista completa de servicios de
Windows, con su estado (En ejecución / Detenido) y tipo de inicio (Automático / Manual).
Busca **Spooler de impresión** o **Windows Update** y fíjate que ninguno tiene ventana —
solo aparecen aquí, nunca en el Escritorio. Compara esta lista con la pestaña
**Servicios** del Administrador de tareas (`Ctrl + Shift + Esc`), que muestra lo mismo de
forma más resumida.

---

Ya tienes hardware y software resueltos. Falta ver **cómo le das instrucciones** al
sistema operativo — con clics o con comandos escritos — antes de pasar a cómo dos
computadoras se comunican entre sí.

**Siguiente paso:** [Interfaces de usuario: gráfica y línea de comandos](03-interfaces-usuario.md).

## Referencias

Silberschatz, A., Galvin, P. B., & Gagne, G. (2018). *Operating system concepts* (10a
ed.). Wiley.
