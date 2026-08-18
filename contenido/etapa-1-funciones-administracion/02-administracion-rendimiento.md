# Administración de rendimiento

> 50 min · [← Administración de configuración](01-administracion-configuracion.md) · Siguiente: [Administración de fallas](03-administracion-fallas.md)

## La P de FCAPS

Esta sesión es la **P** (Performance) del modelo FCAPS visto en la Sesión 1. Mientras la
administración de configuración se pregunta "¿qué tiene configurado cada dispositivo?",
la administración de rendimiento se pregunta algo distinto: **dado que todo está bien
configurado, ¿qué tan bien está respondiendo la red en este momento?**

## Qué es "rendimiento" en una red

**Administración de rendimiento** es optimizar el tiempo de respuesta y mantener
consistente la calidad de los servicios de red — individuales y en conjunto. Para la
mayoría de los usuarios, el tiempo de respuesta *es* la percepción de "la red funciona
bien" o "la red está lenta"; es la variable que más moldea el éxito percibido de una red,
tanto para usuarios como para quien la administra (Subramanian, 2010).

## No se puede administrar lo que no se mide: la línea base

El primer paso, antes de poder decir "la red está lenta hoy", es tener una **línea base**
(baseline) de cómo se comporta la red en un día normal. Sin esa referencia, cualquier
observación es una opinión, no un dato — "se siente lento" no es información accionable,
"el tiempo de respuesta subió de 20 ms a 200 ms respecto a la línea base" sí lo es.

```
Rendimiento normal (línea base)         Sin línea base
─────────────────────────────           ─────────────────
Latencia promedio: 20 ms                "Hoy la red se siente lenta"
                                         (¿comparado con qué? ¿cuánto es "lento"?)
Con línea base:
Latencia hoy: 180 ms  ──> +160 ms sobre lo normal ──> alerta objetiva y medible
```

## Indicadores de rendimiento (KPIs de red)

Estos son los indicadores concretos que se miden en la práctica:

| Indicador | Qué mide | Por qué importa |
|---|---|---|
| **Latencia** | Tiempo que tarda un paquete en ir y volver (ida y vuelta) | Videollamadas y juegos en línea la sienten de inmediato |
| **Jitter** | Qué tan variable es la latencia entre un paquete y otro | Alta variabilidad corta voz/video aunque el promedio sea bueno |
| **Pérdida de paquetes (packet loss)** | % de paquetes que nunca llegan | Obliga a reenviar datos, lo que además empeora la latencia percibida |
| **Rendimiento efectivo (throughput)** | Cantidad real de datos transmitidos por segundo | El límite real que percibe el usuario, no la velocidad "contratada" |
| **Disponibilidad (uptime)** | % de tiempo que un servicio estuvo accesible | Base de los acuerdos de nivel de servicio (SLA) con el cliente |

## Factores críticos de éxito

Para que un proceso cuente como "factor crítico de éxito" en rendimiento, debe cumplir dos
condiciones: su presencia mejora la disponibilidad del servicio, y su ausencia la empeora
de forma medible (Subramanian, 2010). En la práctica, esto se traduce en:

- Recopilar una línea base real de la red y de las aplicaciones (no una suposición).
- Documentar objetivos de nivel de servicio medibles — no "que la red sea rápida", sino
  "latencia menor a 50 ms para el 95% de las peticiones".
- Revisar esa línea base y esos objetivos de forma periódica, no una sola vez al
  instalar la red.
- Tener un proceso definido para cuándo y cómo escalar recursos (más ancho de banda, más
  capacidad) cuando se detecta que la demanda se acerca al límite — antes de que el
  usuario lo note.

## Cómo se mide en la práctica: SNMP y NetFlow

Dos mecanismos reales sostienen casi toda la administración de rendimiento moderna:

- **SNMP** (Simple Network Management Protocol): un protocolo que permite a un sistema de
  monitoreo central preguntarle periódicamente a cada dispositivo de red (routers,
  switches) por sus contadores internos — uso de CPU, tráfico por interfaz, errores —
  originalmente definido en RFC 1157 (Case et al., 1990) y evolucionado hasta SNMPv3, con
  autenticación y cifrado (Harrington et al., 2002).
- **NetFlow** (y su equivalente abierto, **sFlow**): en vez de solo contadores agregados,
  registra información de cada flujo de tráfico (quién habló con quién, cuánto y por
  cuánto tiempo) — el nivel de detalle que permite hacer análisis de capacidad real, no
  solo ver que "hay mucho tráfico" (Claise, 2004).

```
     Router / Switch                    Sistema de monitoreo central
     ┌──────────────┐                   ┌──────────────────────┐
     │ Contadores    │ ── SNMP poll ──> │ Grafica tendencias,   │
     │ CPU, tráfico, │ <── respuesta ── │ compara vs. línea     │
     │ errores...    │                   │ base, dispara alertas│
     └──────────────┘                   └──────────────────────┘
```

## Diagnóstico básico al alcance de cualquiera

Antes de llegar a SNMP/NetFlow (herramientas de nivel de infraestructura), el diagnóstico
de rendimiento más elemental se hace con las mismas herramientas que ya usaste en
Fundamentos de Cómputo: `ping` (mide latencia y pérdida de paquetes hacia un destino
puntual) y `tracert`/`traceroute` (muestra en qué salto del camino empieza a degradarse el
tiempo de respuesta). Son la versión manual, de un solo punto en el tiempo, de lo que SNMP
hace de forma continua y automatizada sobre toda la red.

## El ciclo de administración de rendimiento

```
 ┌───────────┐   ┌───────────┐   ┌────────────┐   ┌───────────┐
 │  Línea    │──>│ Umbral de │──>│  Alerta al │──>│ Análisis  │
 │  base     │   │ tolerancia│   │  excederse │   │ de causa  │
 └───────────┘   └───────────┘   └────────────┘   └─────┬─────┘
       ^                                                 │
       │                                                 v
       └────────────── Ajuste / escalamiento <───────────┘
```

## Ejemplo: "la red está lenta" en la oficina de ventas

Un lunes por la mañana, varios empleados del área de ventas reportan que "la red está
lenta" y que sus videollamadas con clientes se cortan seguido. Sin una línea base, el
equipo de red no tendría con qué comparar esa queja — podría ser una percepción
exagerada, o un problema real; no hay forma de saberlo solo con la queja.

El administrador de red sí tiene una línea base recolectada por SNMP de las últimas
semanas: latencia promedio de 20 ms hacia el servidor de videollamadas, con throughput
estable. Al revisar los datos de esa misma mañana, la latencia promedio en realidad sigue
siendo de 20 ms — normal. Esto descartaría un problema de rendimiento… si la única
métrica que existiera fuera el promedio de latencia.

El siguiente paso es revisar los datos de **NetFlow** de esa interfaz, que muestran el
detalle por flujo: hay una variabilidad muy alta en la latencia de paquete a paquete
durante los mismos minutos en que los empleados reportaron cortes — es decir, alto
**jitter**, aunque el promedio agregado siga viéndose bien. Cruzando ese dato con el
registro de tráfico, aparece la causa: esa misma mañana, otra área del edificio empezó una
transferencia masiva de respaldo de archivos que satura momentáneamente el mismo enlace
compartido, generando picos de latencia que el promedio diluye pero que sí cortan una
llamada de voz en tiempo real.

Sin la combinación de línea base + indicadores más allá del promedio (jitter, no solo
latencia) + NetFlow para ver el detalle por flujo, este problema se habría quedado sin
diagnóstico real, con la única respuesta de "revisamos y la red está bien" — técnicamente
cierto en promedio, pero inútil para quien de verdad experimentó el problema.

## Cómo se resuelve con herramientas reales

Más adelante en el curso, con tu propia **VM de Ubuntu en VirtualBox**, vas a tener a la
mano dos herramientas de línea de comandos que hacen exactamente el diagnóstico del
ejemplo, sin necesitar todavía un sistema de monitoreo completo instalado:

- **`mtr`** (My Traceroute): combina `ping` y `tracert`/`traceroute` en una sola
  herramienta que corre de forma continua, mostrando por cada salto del camino no solo la
  latencia, sino el **porcentaje de pérdida de paquetes** y la variación entre mediciones
  — el jitter del ejemplo, visible salto por salto, no solo como promedio global.
- **`iperf3`**: en vez de estimar el throughput a partir del tráfico normal, genera tráfico
  de prueba entre dos puntos (por ejemplo, tu VM y un servidor) para medir directamente
  cuánto ancho de banda real hay disponible en ese momento — la forma más directa de
  confirmar o descartar saturación de un enlace.

```
mtr servidor-videollamadas.local
Salto   Pérdida   Latencia prom.   Jitter
1       0%        1 ms             0.2 ms
2       0%        8 ms             0.5 ms
3       2%        45 ms            35 ms   <-- aquí está el problema
4       0%        20 ms            2 ms
```

Un equipo de operación real no corre `mtr`/`iperf3` manualmente cada vez — los mismos
datos (latencia, jitter, pérdida, throughput por interfaz) se recolectan de forma
continua y automática con un sistema de monitoreo apoyado en SNMP/NetFlow, como
**Zabbix** o **LibreNMS** (ambos de código abierto, instalables en un servidor Linux). Por
ahora, `mtr` e `iperf3` son la versión manual y puntual de ese mismo diagnóstico — la que
sí vas a poder correr tú mismo desde tu VM.

---

## Referencias

Subramanian, M. (2010). *Network management: Principles and practice* (2a ed.). Pearson
Addison-Wesley.

Case, J., Fedor, M., Schoffstall, M., & Davin, J. (1990). *A simple network management
protocol (SNMP)* (RFC 1157). IETF. https://www.rfc-editor.org/rfc/rfc1157

Harrington, D., Presuhn, R., & Wijnen, B. (2002). *An architecture for describing Simple
Network Management Protocol (SNMP) management frameworks* (RFC 3411). IETF.
https://www.rfc-editor.org/rfc/rfc3411

Claise, B. (Ed.). (2004). *Cisco systems NetFlow services export version 9* (RFC 3954).
IETF. https://www.rfc-editor.org/rfc/rfc3954

ESnet. (s.f.). *iperf3: A TCP, UDP, and SCTP network bandwidth measurement tool*.
https://iperf.fr/

The mtr Project. (s.f.). *mtr(8) — Linux manual page*.
https://man7.org/linux/man-pages/man8/mtr.8.html
