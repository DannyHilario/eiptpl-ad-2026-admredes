# Administración de fallas

> 50 min · [← Administración de rendimiento](02-administracion-rendimiento.md) · Siguiente: [Administración de contabilidad](04-administracion-contabilidad.md)

## La F de FCAPS

Esta es la **F** (Fault) del modelo visto en la Sesión 1 — probablemente la más visible de
las cinco: cuando falla, todos se enteran. Mientras rendimiento (Sesión 2) se pregunta
"¿qué tan bien responde algo que sí funciona?", administración de fallas se pregunta
"¿qué dejó de funcionar, y cómo lo arreglo lo más rápido posible?".

## Qué es

**Administración de fallas** es el conjunto de técnicas, herramientas y procesos para
**identificar, diagnosticar y solucionar** problemas en una red de comunicaciones. Es
crítica para prevenir interrupciones de servicio, reducir tiempos de inactividad, y
mantener la calidad de la transmisión de datos, voz y video (ISO/IEC 7498-4:1989 la
define formalmente como una de las cinco áreas funcionales de gestión OSI).

## Dos métricas que resumen toda esta función: MTBF y MTTR

La industria mide la salud de fallas de una red con dos números:

| Sigla | Significado | Qué responde |
|---|---|---|
| **MTBF** (Mean Time Between Failures) | Tiempo medio entre fallas | ¿Qué tan seguido falla este equipo/servicio? |
| **MTTR** (Mean Time To Repair/Recovery) | Tiempo medio de reparación | Cuando falla, ¿qué tan rápido se resuelve? |

Una red confiable no es necesariamente la que nunca falla (eso es prácticamente
imposible a gran escala) — es la que tiene un **MTBF alto** (falla poco) *y* un
**MTTR bajo** (cuando falla, se repara rápido). La administración de fallas ataca ambos
lados: reduce la frecuencia de fallas con monitoreo preventivo, y reduce el tiempo de
reparación con un proceso claro de diagnóstico.

## El ciclo de vida de una falla

```
 ┌───────────┐   ┌────────────┐   ┌───────────┐   ┌────────────┐   ┌─────────────┐
 │ Monitoreo │──>│ Detección  │──>│Diagnóstico│──>│ Corrección │──>│ Documentación│
 │ constante │   │ (alarma /  │   │ y         │   │ (aplicar   │   │ (post-mortem,│
 │           │   │  trap)     │   │ aislamiento│  │  la solución)│  │  para MTBF) │
 └───────────┘   └────────────┘   └───────────┘   └────────────┘   └─────────────┘
```

- **Monitoreo constante**: detectar el problema antes de que el usuario lo reporte —
  idealmente el equipo de red se entera de la falla antes que el usuario final.
- **Detección**: un sistema de gestión recibe una señal de que algo salió mal. En redes
  reales esto llega casi siempre por dos vías: **syslog** (el dispositivo manda un mensaje
  de texto describiendo el evento, formato estandarizado en RFC 5424; Gerhards, 2009) o una
  **trampa SNMP** (el dispositivo notifica proactivamente, en vez de esperar a que lo
  pregunten — a diferencia del sondeo SNMP normal que viste en la Sesión 2).
- **Diagnóstico y aislamiento**: usar herramientas para encontrar la causa exacta y
  delimitar qué tan extendido está el problema — la misma lógica de `tracert`/`traceroute`
  que ya usaste (Fundamentos de Cómputo): identificar en qué punto exacto de la ruta
  empieza a fallar algo.
- **Corrección**: aplicar la solución — puede ser tan simple como reiniciar un servicio, o
  requerir restaurar una configuración de línea base (Sesión 1) si la falla vino de un
  cambio reciente.
- **Documentación**: registrar la falla y su solución como referencia histórica. Esto
  alimenta directamente el cálculo del MTBF/MTTR reales de la red, y evita reinventar el
  diagnóstico la próxima vez que se repita un problema similar.

## Comunicación durante el incidente

Un aspecto de administración de fallas que suele subestimarse es la comunicación
**hacia el usuario**, no solo hacia el equipo técnico: notificar periódicamente el estatus
de un incidente reportado — aunque no haya solución todavía — mejora la percepción del
servicio independientemente del resultado final. En la operación real, la carga de
trabajo suele rebasar las buenas intenciones de "hacerlo todo ordenado"; por eso conviene
dejar el proceso de notificación planeado *antes* de que ocurra la falla, no
improvisarlo en medio de la urgencia.

## Prevención, no solo reacción

La administración de fallas no es puramente reactiva. Incluye también:

- Monitoreo constante para detectar anomalías antes de que se conviertan en una falla
  total (relacionado con los umbrales de la Sesión 2).
- Pruebas periódicas de configuración y rendimiento de equipos y sistemas.
- Políticas de respaldo (backup) y redundancia, para que una sola falla no implique
  pérdida total del servicio.

## Ejemplo: el switch núcleo que se cae los viernes por la tarde

Una planta tiene un switch núcleo que conecta la línea de producción con el sistema de
control. Un viernes por la tarde, ese switch deja de responder por completo durante unos
minutos, y la línea se detiene. El equipo de red recibe una **trampa SNMP** del switch
(no un sondeo normal — el propio dispositivo notificó la caída antes de que nadie
preguntara) casi al mismo tiempo que llegan los mensajes de **syslog** de los dispositivos
vecinos reportando pérdida de conectividad hacia él.

El diagnóstico empieza aislando el problema: usando la misma lógica de `tracert` vista en
Fundamentos de Cómputo, el equipo confirma que el problema está exactamente en ese switch
y no en el resto de la red — los demás enlaces siguen respondiendo con normalidad. Al
revisar el historial (documentación de fallas anteriores), encuentran que **este mismo
switch ya se había caído dos viernes antes**, siempre a una hora similar. La corrección
inmediata es reiniciarlo para restaurar el servicio — pero la administración de fallas no
se detiene ahí.

Con el patrón repetido documentado, el equipo cruza la hora de las caídas con otro dato:
los viernes por la tarde es cuando el área de sistemas corre un respaldo pesado que
sobrecarga momentáneamente ese switch hasta hacerlo fallar por temperatura. La causa raíz
no era un defecto aleatorio del equipo, sino una condición predecible — y solo se pudo
identificar porque las fallas anteriores habían quedado **documentadas**, no solo
resueltas y olvidadas.

Este caso muestra el ciclo completo: si el switch tiene, digamos, tres fallas en dos
meses (MTBF bajo) pero cada una se resuelve en 5 minutos con un reinicio (MTTR bajo), la
red sigue siendo confiable en la práctica aunque no sea perfecta — y el MTTR bajo solo fue
posible porque el equipo ya sabía, por experiencia documentada, exactamente qué buscar.

## Cómo se resuelve con herramientas reales

En Linux (el sistema de tu VM de Ubuntu en VirtualBox), el equivalente real de "el equipo
recibe una trampa SNMP y mensajes de syslog casi al mismo tiempo" tiene nombre concreto:

- **`rsyslog`**: el demonio de syslog que trae Ubuntu por defecto (Gerhards, s.f.). Puede
  configurarse para no solo guardar los mensajes localmente, sino **reenviarlos** a un
  servidor central — así los mensajes de todos los dispositivos de la red llegan a un solo
  lugar, en vez de quedar dispersos switch por switch.
- **`snmptrapd`**: parte del paquete `net-snmp`, es el programa que "escucha" las trampas
  SNMP que un dispositivo manda proactivamente y las convierte en una alerta o en una
  entrada de log — el receptor del lado del sistema de monitoreo (Net-SNMP Project, s.f.).

```
Switch núcleo                    Servidor central de logs/alertas
┌──────────────┐                 ┌────────────────────────────┐
│ syslog local  │ ── rsyslog ──> │ Mensajes de todos los       │
│ trampa SNMP   │ ── snmptrapd ─>│ dispositivos, en un solo    │
│               │                │ lugar y con marca de tiempo │
└──────────────┘                 └────────────────────────────┘
```

Con ambos centralizados, el "historial de fallas anteriores" del ejemplo deja de depender
de la memoria del equipo: es una búsqueda de texto sobre los logs guardados (por ejemplo,
con `grep`, que ya conoces de Fundamentos de Cómputo) para confirmar que ese mismo switch
ya había fallado los viernes anteriores a la misma hora. Herramientas de monitoreo de
código abierto como **Zabbix** o **Nagios** automatizan un paso más: además de centralizar
estos mensajes, pueden abrir un ticket automáticamente y notificar al equipo sin que nadie
tenga que estar viendo una pantalla en el momento exacto de la falla.

---

## Referencias

International Organization for Standardization. (1989). *Information technology — Open
Systems Interconnection — Basic Reference Model — Part 4: Management framework* (ISO/IEC
7498-4:1989).

Gerhards, R. (2009). *The syslog protocol* (RFC 5424). IETF.
https://www.rfc-editor.org/rfc/rfc5424

Subramanian, M. (2010). *Network management: Principles and practice* (2a ed.). Pearson
Addison-Wesley.

Gerhards, R. (s.f.). *rsyslog documentation*. https://www.rsyslog.com/doc/

Net-SNMP Project. (s.f.). *snmptrapd(8) — man page*.
http://www.net-snmp.org/docs/man/snmptrapd.html
