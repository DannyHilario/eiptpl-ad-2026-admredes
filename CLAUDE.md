# eiptpl-ad-2026-admredes

Contenido del curso **"Administración de Redes"** — EIPTPL "Pablo Livas" (UANL),
preparatoria, 2026. Repositorio de material didáctico, no de código de producción.

Repo público en GitHub: `DannyHilario/eiptpl-ad-2026-admredes`.

## Estructura

`contenido/` es lo único que vive en este repositorio (ver `.gitignore` para lo que queda
deliberadamente fuera). Tiene dos módulos, en **este orden de estudio**:

1. `contenido/fundamentos-computo/` — prerrequisito de cómputo básico (hardware/software/SO)
2. `contenido/introduccion/` — introducción de redes propiamente dicha

### `contenido/fundamentos-computo/` — prerrequisito de cómputo básico, publicado

Módulo agregado tras detectar que el grupo llega sin bases mínimas de cómputo (confunden
RAM con almacenamiento, etc.). No forma parte del temario del manual oficial — es un
prerrequisito propio, en **4 sesiones de 50 minutos**:

| Archivo | Sesión | Tema |
|---|---|---|
| `01-hardware.md` | 1 | Hardware: motherboard, CPU (núcleos/hilos/GHz/caché), RAM (DDR, MHz, ECC), almacenamiento (HDD/SSD/NVMe), GPU (integrado/dedicado), fuente de poder, cuellos de botella, specs de servidor |
| `02-software-so.md` | 2 | Software por capas (firmware/SO/drivers/apps), gestión de procesos/memoria/archivos/dispositivos, proceso vs. servicio |
| `03-interfaces-usuario.md` | 3 | GUI vs. CLI, comandos equivalentes Windows (CMD/PowerShell) / macOS / Linux |
| `04-cliente-servidor.md` | 4 | Qué es un servidor, arquitectura cliente-servidor, ejemplo clásico (web) — puente hacia `contenido/introduccion/` |

Analogía única del módulo: la computadora como **escritorio de oficina** (RAM = superficie
del escritorio, almacenamiento = archivero, CPU = quien trabaja, SO = el gerente). Sin
autochecks — cada sesión termina con un ejercicio práctico paso a paso en Windows 11 (el
grupo trabaja en equipos de laboratorio con Windows 11) en vez de preguntas de repaso.

**Encabezados sin "Sesión N":** en este módulo el `# H1` es solo el tema (`# Hardware:
las piezas físicas de una computadora`, no `# Sesión 1 (50 min) — Hardware...`). La
duración y la navegación van en la línea de blockquote debajo: `> 50 min · [← Anterior] ·
Siguiente: [Tema]`, también sin decir "Sesión N". `contenido/introduccion/` **todavía usa
el estilo viejo** (`# Sesión N (50 min) — Tema`, con "Sesión N" en la navegación) — no se
ha migrado a este módulo porque esa parte del contenido se dejó fuera de este ciclo de
cambios a propósito. Si se retoma `introduccion/`, aplicar ahí el mismo estilo sin
"Sesión N".

### `contenido/introduccion/` — introducción de redes, publicado

Introducción del curso, dividida en **4 sesiones de 50 minutos**:

| Archivo | Sesión | Tema |
|---|---|---|
| `01-redes.md` | 1 | Redes (repaso + switch vs. router a fondo + PAN/MAN + P2P) |
| `02-trafico.md` | 2 | Tráfico, paquetes, protocolos, puertos (repaso + síntesis) |
| `03-vm-y-linux.md` | 3 | Máquinas virtuales, modos de red VirtualBox (100% nuevo) |
| `04-terminal.md` | 4 | Terminal Linux/WSL2, pipes, sudo/apt (100% nuevo) |

**Nombres de archivo:** `0X-tema.md` en ambos módulos — sin la palabra "sesion" en el
nombre (ej. `01-hardware.md`, no `01-sesion-1-hardware.md`).

No hay archivo de panorama/índice en ningún módulo — se eliminó a propósito porque no
aportaba contenido real; la Sesión 1 de cada módulo es el punto de entrada. Sigue ese
patrón (sin agregar un índice nuevo) si agregas una sesión o un módulo.

**Ningún módulo tiene `glosario.md` ni `fuentes.md` separados** — se probaron y se
descartaron a propósito, no aportaban valor como archivos aparte. Cada sesión es
autocontenida: los términos se explican en negritas donde aparecen, y las citas van
inline en el texto (`(Autor, año)`) con una sección `## Referencias` al final del mismo
archivo, solo con las fuentes citadas en esa sesión — no una lista global del módulo.

## Contexto pedagógico importante

Los alumnos **no llegan sin conocimientos previos**. El semestre anterior cursaron
**Redes y Telecomunicaciones**, donde ya vieron: red, protocolo, IP (clases,
pública/privada), DNS, DHCP, modelo OSI completo, TCP/UDP, HTTP/HTTPS, topologías
(bus/estrella/anillo/híbrida), y hasta crimparon cables RJ45 y configuraron redes en
Windows.

Lo que **nunca vieron** es virtualización, Linux o terminal — su curso anterior fue
100% Windows/GUI. Por eso la Introducción de este curso:
- Sesiones 1 y 2 son **repaso rápido + lo que faltaba** (no reenseñan desde cero).
- Sesiones 3 y 4 son **completamente nuevas** y llevan el peso real del tiempo de clase.

**Actualización tras la primera clase real con el grupo:** el nivel de base es más bajo de
lo que el temario anterior asumía — confunden conceptos elementales como RAM vs.
almacenamiento. De ahí surgió `contenido/fundamentos-computo/`, un prerrequisito aún más
atrás que ni el manual oficial ni "Redes y Telecomunicaciones" cubren. Si en clase
detectas más huecos de este tipo (no solo de redes, sino de cómputo básico), coméntalo —
es probable que necesite su propio módulo corto, igual que este.

El manual oficial de este curso, "Administración de Redes", organiza el resto del
semestre en 4 etapas: Etapa 1 funciones de administración (configuración, rendimiento,
fallas, contabilidad, seguridad), Etapa 2 servicios de red (DHCP, DNS, WWW, FTP, correo),
Etapa 3 instalación de un sistema operativo de red, Etapa 4 direccionamiento IP y
subneteo. Al escribir contenido nuevo, conviene conectarlo explícitamente con la etapa del
manual que prepara.

**El manual oficial define el temario, nunca el contenido técnico.** Su redacción está
desactualizada y en partes mal hecha — úsalo únicamente para saber qué temas tocar en
cada etapa, jamás como fuente de verdad para explicar cómo funciona algo. Todo hecho
técnico (números de puerto, comandos, pasos de instalación, protocolos) debe verificarse
contra fuentes reales y quedar reflejado en la sección `## Referencias` de la sesión
correspondiente.

## Verificabilidad del contenido

Todo el contenido técnico de `contenido/` (cualquier módulo) debe ser verificable:
- Hechos duros (puertos, RFCs, núcleos/hilos, estándares de memoria, comandos, pasos de
  software) → cita inline `(Autor, año)` + entrada en `## Referencias` al final del mismo
  archivo de sesión (APA 7), con referencias reales que puedas confirmar que existen —
  nunca inventes una cita.
- Analogías y framing pedagógico (fraccionamiento/casas, escritorio de oficina) no
  necesitan cita — son recursos didácticos propios, no afirmaciones de hecho.
- Si no puedes verificar un dato con confianza, dilo explícitamente en vez de presentarlo
  como si estuviera confirmado.

## Convenciones de contenido

- **Español**, tono directo, sin asumir jerga no explicada previamente.
- **Sin Mermaid, sin imágenes**: todos los diagramas son **ASCII puro** dentro de bloques
  ``` ``` ```, no ```mermaid```. Se ven igual en GitHub, en cualquier editor y en texto
  plano, sin depender de ninguna extensión ni de que el visor soporte renderizado
  especial. Si agregas un diagrama nuevo, hazlo en ASCII (cajas con `─│┌┐└┘`, flechas con
  `-->`/`v`/`<-`), no en Mermaid.
- **Autochecks**: solo en `contenido/introduccion/`. Markdown puro, sin tags HTML
  (`<details>`/`<summary>` no se usan — se descartaron a propósito). Formato:
  `> **Pregunta:** ...` seguido de `> **Respuesta:** ...` en el mismo blockquote,
  respuesta siempre visible. `contenido/fundamentos-computo/` no usa autochecks a
  propósito — cierra cada sesión con un ejercicio práctico paso a paso en su lugar.
- Una sola analogía por concepto, reutilizada entre sesiones — no metáforas nuevas que
  compitan entre sí para la misma idea.
- Nada de líneas de código de más de ~80-90 caracteres si se puede evitar (aunque no
  afecta el renderizado, ayuda a la legibilidad del fuente).
- **Ejercicios prácticos con comandos reales**, no solo teoría — cuando el concepto lo
  permita, dale al alumno un comando que pueda correr en su propia laptop (`ipconfig`,
  `nslookup`, `tracert`) o en su VM, no solo una explicación abstracta.

## Git

- Commits en español, formato convencional (`feat:`, `fix:`, `docs:`, etc.).
- **Todos los commits deben quedar a nombre de Daniel G. Hilario
  `<danny.hilario.m@gmail.com>`** (autor y committer) — nunca con una identidad de bot,
  de Claude, ni con el correo corporativo `dhilario@deacero.com`. El repo ya tiene
  `user.name`/`user.email` configurados localmente para esto; no los cambies.
- El repo es público — no incluir credenciales, tokens, ni datos sensibles en ningún
  commit.

## Pendiente / siguiente etapa natural

Las 4 etapas del manual oficial todavía no tienen su versión propia en `contenido/` — son
el siguiente contenido a desarrollar después de la Introducción, siguiendo el mismo estilo
(repaso de lo que aplique del curso anterior + diagramas + autochecks).
