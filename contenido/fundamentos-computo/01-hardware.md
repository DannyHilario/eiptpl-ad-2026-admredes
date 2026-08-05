# Hardware: las piezas físicas de una computadora

> 50 min · Siguiente: [Software y el Sistema Operativo](02-software-so.md)

## Hardware vs. software

- **Hardware** es todo lo que puedes **tocar**: la pantalla, el teclado, las piezas
  dentro de la computadora.
- **Software** es todo lo que **no puedes tocar**: instrucciones, programas, el sistema
  operativo — código que le dice al hardware qué hacer.

**Analogía que vamos a reutilizar todo este módulo:** tu computadora es un **escritorio de
oficina**. El hardware son los muebles y objetos físicos del escritorio; el software (lo
que ves en la Sesión 2) son las instrucciones de trabajo que alguien sigue usando esos
muebles.

## Las piezas físicas, una por una

```
┌──────────────────────────────────────────────────────────┐
│                      Motherboard                          │
│                                                            │
│   ┌────────┐   ┌────────┐   ┌────────┐   ┌─────────────┐ │
│   │  CPU   │   │  RAM   │   │  GPU   │   │Almacenamiento│ │
│   └────────┘   └────────┘   └────────┘   └─────────────┘ │
│                                                            │
└──────────────────────────────────────────────────────────┘
                          │
                  ┌───────────────┐
                  │ Fuente de poder│
                  └───────────────┘
```

### Motherboard (placa madre) — el escritorio mismo

La **motherboard** es la placa física donde se conectan todas las demás piezas: el CPU va
en un zócalo (socket) específico para su familia de procesador, la RAM va en ranuras
(slots) DIMM, y tiene puertos para almacenamiento (SATA, M.2), tarjetas de expansión
(ranuras PCIe para GPU dedicado), USB, red y audio. Un chip llamado **chipset** en la
motherboard coordina la comunicación entre el CPU y todo lo demás.

**Ejemplos reales:** ASUS Prime B660M-A (para procesadores Intel), MSI B550 Tomahawk
(para procesadores AMD Ryzen). El socket de la motherboard debe coincidir con la familia
del CPU — una motherboard con socket LGA1700 no acepta un CPU con socket AM5.

### CPU (procesador) — quien trabaja en el escritorio

El **CPU** (Central Processing Unit) ejecuta las instrucciones del software. Estos
atributos determinan qué tan capaz es (Patterson & Hennessy, 2020):

| Atributo | Qué significa | Por qué importa |
|---|---|---|
| **Núcleos (cores)** | Unidades de procesamiento independientes dentro del mismo chip | Más núcleos = más tareas distintas en paralelo real, no por turnos |
| **Hilos (threads)** | Algunos núcleos se dividen en 2 hilos lógicos vía **Hyper-Threading** (Intel) o **SMT** (AMD) | Un CPU "6 núcleos / 12 hilos" son 6 trabajadores físicos, cada uno atendiendo 2 tareas entrelazadas |
| **Velocidad de reloj (GHz)** | Ciclos de trabajo por segundo — 3.5 GHz = 3,500 millones de ciclos/segundo | Más GHz = cada núcleo termina cada tarea individual más rápido |
| **Turbo/Boost** | El CPU sube su GHz temporalmente en tareas exigentes, y lo baja en reposo para ahorrar energía | La velocidad "base" y la velocidad "turbo" son dos números distintos en toda hoja de especificaciones |
| **Caché (L1/L2/L3)** | Memoria diminuta y extremadamente rápida integrada en el propio chip | Guarda lo que el CPU usa en este instante — más rápida que la RAM, pero mucho más pequeña (KB-MB, no GB) |
| **TDP (Thermal Design Power)** | Cuánto calor/energía disipa el CPU en uso normal, en Watts | Determina qué tan grande debe ser el disipador/ventilador que necesita |

**Comparativa de CPUs reales, por nivel:**

| Modelo | Núcleos / Hilos | Reloj base / turbo | Caché L3 | TDP |
|---|---|---|---|---|
| Intel Core i3-12100 (entrada) | 4 / 8 | 3.3 / 4.3 GHz | 12 MB | 60 W |
| Intel Core i5-12400 (medio) | 6 / 12 | 2.5 / 4.4 GHz | 18 MB | 65 W |
| AMD Ryzen 5 5600 (medio) | 6 / 12 | 3.5 / 4.4 GHz | 32 MB | 65 W |
| Intel Core i9-13900K (alto) | 24 / 32 | 3.0 / 5.8 GHz | 36 MB | 125 W |
| AMD EPYC 9654 (servidor) | 96 / 192 | 2.4 / 3.7 GHz | 384 MB | 360 W |

Fíjate en el último renglón: un CPU de servidor no busca el GHz más alto — busca
**muchísimos núcleos**, porque un servidor atiende a cientos de usuarios/peticiones al
mismo tiempo, no una sola tarea que necesite terminarse lo más rápido posible. Volvemos a
esto más abajo.

### RAM (memoria) — la superficie del escritorio

La **RAM** (Random Access Memory) guarda **temporalmente** lo que la computadora está
usando en este momento — se borra por completo al apagar. Estos atributos la describen:

| Atributo | Qué significa | Por qué importa |
|---|---|---|
| **Capacidad (GB)** | Cuánto puede guardar a la vez — 8, 16, 32, 64 GB | Más capacidad = más programas/pestañas abiertos sin quedarse sin espacio |
| **Generación (DDR3/DDR4/DDR5)** | Cada generación DDR (Double Data Rate) es más rápida y eficiente que la anterior, definidas por JEDEC (JEDEC Solid State Technology Association, s.f.) | Un módulo DDR4 no encaja físicamente en una ranura DDR5 — son incompatibles entre sí |
| **Velocidad (MHz)** | Qué tan rápido se transfieren datos entre la RAM y el CPU | Un DDR4-3200 transfiere datos más rápido que un DDR4-2400, aunque ambos sean DDR4 |
| **Latencia (CL, CAS Latency)** | Cuántos ciclos de reloj tarda la RAM en responder a una solicitud | Menor CL = respuesta más rápida, aunque la velocidad en MHz sea la misma |
| **Canales (single/dual/quad)** | Cuántos módulos idénticos puede leer la motherboard simultáneamente | Doble canal (2 módulos idénticos) casi duplica el ancho de banda frente a un solo módulo de la misma capacidad total |
| **ECC (Error-Correcting Code)** | Memoria que detecta y corrige errores de un bit automáticamente | Casi nunca en PCs de escritorio; **estándar en servidores**, que corren 24/7 sin poder tolerar errores silenciosos de memoria |

**Comparativa de RAM real, por nivel:**

| Producto | Capacidad | Generación / velocidad | Latencia | Uso típico |
|---|---|---|---|---|
| Kingston ValueRAM 8 GB | 8 GB | DDR4-2666 | CL19 | Oficina básica |
| Kingston Fury Beast 16 GB | 16 GB (2×8 GB) | DDR4-3200 | CL16 | Uso general, doble canal |
| Corsair Vengeance 32 GB | 32 GB (2×16 GB) | DDR5-5600 | CL36 | Gaming/creación de contenido |
| Kingston Server Premier 32 GB ECC | 32 GB | DDR4-3200 ECC | CL22 | Servidor |

Nota que la RAM ECC de servidor no es necesariamente "más rápida" en MHz — su valor está
en la **corrección de errores**, no en la velocidad.

### Almacenamiento (HDD o SSD) — el archivero/cajón

El **almacenamiento** guarda información **de forma permanente**: documentos, fotos,
programas instalados, el propio sistema operativo — sigue ahí aunque apagues la
computadora.

- **HDD** (Hard Disk Drive): un disco físico que gira a una velocidad medida en RPM
  (5,400 o 7,200 RPM son comunes) y una aguja que lee/escribe sobre él. Es más barato por
  GB, pero lento por sus partes mecánicas en movimiento.
- **SSD** (Solid State Drive): sin partes mecánicas, guarda datos en memoria flash. Se
  conecta de dos formas distintas:
  - **SATA:** el mismo tipo de conexión que usaban los HDD tradicionales — un SSD SATA es
    mucho más rápido que un HDD, pero limitado por la velocidad máxima del propio
    conector SATA.
  - **NVMe (M.2):** se conecta directo a la motherboard vía PCIe, sin el límite del
    conector SATA — varias veces más rápido que un SSD SATA (NVM Express, Inc., s.f.).

**Ejemplos reales:** Seagate Barracuda 1 TB (HDD, 7,200 RPM), WD Blue 500 GB (SSD SATA),
Samsung 970 EVO Plus 1 TB (SSD NVMe).

### GPU (tarjeta gráfica) — el especialista en dibujar

El **GPU** (Graphics Processing Unit) genera las imágenes que ves en pantalla. A
diferencia del CPU (pocos núcleos muy versátiles, buenos para lógica compleja), un GPU
tiene **miles de núcleos simples** diseñados para hacer el mismo tipo de cálculo repetido
miles de veces en paralelo — ideal para gráficos, donde hay que calcular el color de
millones de píxeles al mismo tiempo, y también para IA, que hace el mismo tipo de
operación matemática (multiplicaciones de matrices) millones de veces.

| Atributo | Qué significa | Por qué importa |
|---|---|---|
| **Núcleos gráficos** | Miles de unidades simples de cálculo paralelo (CUDA cores en NVIDIA, stream processors en AMD) | Más núcleos = más píxeles/operaciones calculadas al mismo tiempo |
| **VRAM** | Memoria dedicada exclusiva del GPU, tipo GDDR (no comparte la RAM del sistema) | Limita la resolución/complejidad de lo que puede procesar sin quedarse sin espacio |
| **Ancho de banda de memoria** | Qué tan rápido el GPU lee/escribe su propia VRAM | Un GPU con muchos núcleos pero poco ancho de banda se queda "esperando" datos |
| **Integrado vs. dedicado** | Integrado: vive en el mismo chip que el CPU y usa la RAM del sistema. Dedicado: tarjeta aparte, con su propia VRAM | El integrado ahorra costo y espacio; el dedicado da mucho más rendimiento gráfico |

### Gráficos integrados, a fondo

La mayoría de las laptops de oficina y muchos CPUs de escritorio ya traen un GPU
**integrado** dentro del mismo chip del CPU — no es una pieza aparte. Compite por la
**misma RAM** que usa el resto del sistema (se le llama memoria compartida), en vez de
tener su propia VRAM dedicada. Por eso una computadora con GPU integrado y solo 8 GB de
RAM reparte esos 8 GB entre el sistema operativo, tus programas, *y* los gráficos — un
GPU dedicado con 6 GB de VRAM propia no le quita nada a la RAM del sistema.

Es perfectamente suficiente para: navegación web, oficina, reproducir video, la mayoría
del trabajo administrativo — exactamente el tipo de equipo que vas a encontrar en un
laboratorio escolar o una oficina. Se queda corto para: videojuegos exigentes, edición de
video pesada, diseño 3D, entrenar modelos de inteligencia artificial.

**Comparativa de GPU real, por nivel:**

| Modelo | Tipo | VRAM | Uso típico |
|---|---|---|---|
| Intel UHD Graphics 730 | Integrado | Comparte RAM del sistema | Oficina, laboratorio escolar, navegación |
| AMD Radeon Graphics (Ryzen "G") | Integrado | Comparte RAM del sistema | Oficina, algo de videojuegos ligeros |
| NVIDIA GeForce RTX 3050 | Dedicado | 6 GB GDDR6 | Videojuegos, edición de video básica |
| AMD Radeon RX 6600 | Dedicado | 8 GB GDDR6 | Videojuegos en resoluciones altas |
| NVIDIA A100 (servidor/IA) | Dedicado | 40-80 GB HBM2e | Entrenamiento de IA, centros de datos |

Nota la última fila: los GPU de servidor para inteligencia artificial no se especifican
por qué tan bien se ve un videojuego — se especifican por cuánta VRAM y cuántos núcleos
pueden dedicar a procesar datos en paralelo, a una escala mucho mayor que cualquier GPU
de escritorio.

### Fuente de poder (PSU) — la electricidad de todo

La **fuente de poder** convierte la corriente alterna del enchufe en la corriente directa
que usan las piezas internas, a distintos voltajes. Se especifica por:

- **Potencia (Watts):** cuánta energía puede entregar en total — 450 W, 650 W, 850 W. Debe
  ser suficiente para sumar lo que consumen CPU, GPU y el resto de las piezas.
- **Certificación de eficiencia (80+ Bronze/Gold/Platinum):** qué porcentaje de la
  electricidad que consume del enchufe se convierte en energía útil (el resto se pierde
  como calor) — más alto en la escala es más eficiente.

**Ejemplo real:** Corsair CV650, 650 W, certificación 80+ Bronze.

---

## Cómo estos componentes, en conjunto, determinan qué tan potente es un PC

Ninguna pieza por sí sola define "qué tan buena" es una computadora — lo que importa es
el **equilibrio** entre ellas. Un componente débil se convierte en **cuello de botella
(bottleneck)**: limita lo que las demás piezas, aunque sean potentes, pueden lograr en la
práctica (Patterson & Hennessy, 2020).

**Ejemplos de cuello de botella:**

| Situación | Qué pasa | Por qué |
|---|---|---|
| GPU potente + CPU débil | El GPU pasa tiempo "esperando" instrucciones | El CPU no logra alimentarlo con datos lo bastante rápido |
| Mucha RAM + almacenamiento HDD lento | El sistema sigue sintiéndose lento al abrir programas | Cargar un programa del disco al RAM sigue limitado por la velocidad del HDD |
| CPU rápido + poca RAM (4 GB) | La computadora se congela con pocos programas abiertos | El SO se queda sin espacio en RAM y empieza a usar memoria virtual (mucho más lenta) constantemente |

**Qué combinación de specs necesita cada tipo de uso:**

| Uso | CPU | RAM | Almacenamiento | GPU |
|---|---|---|---|---|
| Oficina / laboratorio escolar | 4 núcleos | 8 GB | SSD 256 GB | Integrado |
| Uso general / estudiante | 6 núcleos | 16 GB | SSD 512 GB | Integrado o dedicado de entrada |
| Diseño / edición de video | 8+ núcleos | 32 GB | SSD NVMe 1 TB | Dedicado, 8+ GB VRAM |
| Servidor de red pequeño | 8-16 núcleos | 32-64 GB ECC | SSD NVMe en RAID | Generalmente no necesita GPU dedicado |

## Por qué esto importa para especificar servidores

Todo lo que acabas de ver — núcleos, hilos, GHz, capacidad y velocidad de RAM, tipo de
almacenamiento — es exactamente el vocabulario que vas a usar más adelante en este curso
(Etapa 3 del manual: instalación de un sistema operativo de red) para **elegir qué
hardware necesita un servidor**. La diferencia es que las prioridades cambian por
completo frente a una PC de escritorio:

| Prioridad | PC de escritorio | Servidor |
|---|---|---|
| CPU | Pocos núcleos, GHz alto (una persona, tareas variadas) | Muchos núcleos, GHz moderado (cientos de peticiones simultáneas) |
| RAM | Rápida, sin corrección de errores | Con **ECC** — un error de memoria no corregido en un servidor 24/7 puede corromper datos de todos los que lo usan |
| Almacenamiento | Un solo disco | Varios discos en **RAID** (redundancia — si uno falla, los datos sobreviven) |
| GPU | Importante si hay uso gráfico/gaming | Casi siempre innecesario, salvo IA o virtualización gráfica |
| Fuente de poder | Una sola | A veces **redundante** (dos fuentes, si una falla la otra sigue dando energía) |
| Disponibilidad | Se puede apagar sin problema | Diseñado para correr 24/7, meses sin reiniciar |

Un servidor no es "una PC más potente" — es una PC con **las mismas piezas que ya
conoces**, elegidas y balanceadas para un objetivo distinto: atender a muchos usuarios de
forma confiable y continua, en vez de a una sola persona haciendo una tarea a la vez.

---

## Ejercicio práctico: identifica el hardware de tu propia computadora (Windows 11)

Están en un equipo de laboratorio con Windows 11 — usa exactamente esta computadora para
los siguientes pasos, y anota lo que encuentres en cada uno.

**1. Especificaciones generales:**
`Inicio` → **Configuración** → **Sistema** → **Acerca de**. Ahí ves el procesador, la RAM
instalada, y el tipo de sistema (32 o 64 bits).

**2. Núcleos e hilos del CPU:**
`Ctrl + Shift + Esc` abre el **Administrador de tareas** → pestaña **Rendimiento** → clic
en **CPU**. En la esquina inferior derecha del gráfico aparecen "Núcleos" e "Hilos
lógicos" — compáralos con lo que investigaste sobre tu modelo de CPU.

**3. Velocidad y uso de la RAM:**
En el mismo Administrador de tareas → **Rendimiento** → **Memoria**. Ahí ves la
capacidad total, cuánta está en uso ahora mismo, la velocidad en MHz, y cuántos slots de
la motherboard están ocupados.

**4. GPU:**
Administrador de tareas → **Rendimiento** → **GPU 0** (o **GPU 1** si hay más de una) —
muestra el nombre del modelo y la memoria dedicada. Para más detalle: `Windows + R` →
escribe `dxdiag` → Enter → pestaña **Pantalla** — ahí aparece el nombre exacto del chip
gráfico y su memoria total.

**5. Almacenamiento:**
Abre el **Explorador de archivos** → **Esta PC** → clic derecho sobre el disco (C:) →
**Propiedades** — ahí ves la capacidad total, espacio usado y libre, y el sistema de
archivos.

**6. Todo junto, en un solo lugar:**
`Windows + R` → escribe `msinfo32` → Enter — abre **Información del sistema**, con un
resumen completo: modelo de la motherboard, BIOS, procesador, y memoria física instalada.

**Siguiente paso:** [Software y el Sistema Operativo](02-software-so.md).

---

## Referencias

Patterson, D. A., & Hennessy, J. L. (2020). *Computer organization and design: The
hardware/software interface* (RISC-V ed.). Morgan Kaufmann.

JEDEC Solid State Technology Association. (s.f.). *Estándares de memoria DDR*.
https://www.jedec.org

NVM Express, Inc. (s.f.). *NVMe specification*. https://nvmexpress.org
