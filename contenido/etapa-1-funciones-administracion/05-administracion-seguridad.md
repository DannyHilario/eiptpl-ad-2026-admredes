# Administración de seguridad

> 50 min · [← Administración de contabilidad](04-administracion-contabilidad.md)

## La S de FCAPS, y el cierre de la Etapa 1

Esta es la última letra, **S** (Security), del modelo FCAPS que abrió la Sesión 1. Con
esta sesión completas las cinco funciones de administración de redes: configuración (C),
rendimiento (P), fallas (F), contabilidad (A) y ahora seguridad (S).

## Qué es

**Administración de seguridad** es ofrecer servicios de protección a cada elemento de la
red y a la red en su conjunto, mediante estrategias de **prevención**, **detección** y
**respuesta a incidentes**, sostenidas por **políticas de seguridad** formales. Las
primeras tres son actividades operativas; la cuarta es lo que les da consistencia y
respaldo institucional.

```
        ┌───────────────────────────────────────┐
        │         Políticas de seguridad          │
        │   (definen qué se debe proteger y cómo) │
        └───────────────┬─────────────────────────┘
                         │ sostiene a las tres de abajo
     ┌───────────────┬───┴────────────┬───────────────┐
     v                v                v
┌─────────┐     ┌───────────┐   ┌──────────────┐
│Prevención│     │ Detección │   │  Respuesta a  │
│de ataques│     │de intrusos│   │  incidentes   │
└─────────┘     └───────────┘   └──────────────┘
```

## Prevención de ataques

El objetivo es mantener los recursos de red fuera del alcance de usuarios maliciosos —
usualmente mediante control de acceso: firewalls, listas de control de acceso (ACLs) en
routers y switches, segmentación de red. Es importante tener expectativas realistas: estas
medidas **reducen** los ataques, nunca los eliminan por completo — no existe una red
perfectamente inatacable, solo redes con distintos niveles de resistencia.

## Detección de intrusos

El objetivo es detectar el momento en que un ataque se está llevando a cabo, no solo
después de que ya causó daño. Esto se logra con un **sistema de detección de intrusos**
(IDS) que vigila y registra el tráfico de la red, apoyado en notificaciones o alarmas
que señalan actividad anormal. Un ejemplo real y ampliamente documentado de esta
categoría de herramienta es **Snort**, un IDS de código abierto que analiza tráfico en
tiempo real contra un conjunto de reglas conocidas de ataque (Roesch, 1999).

## Respuesta a incidentes

El objetivo es, una vez detectado un compromiso de seguridad, investigar sus causas y
eliminarlas. Esto no debería improvisarse durante la crisis: el estándar de referencia
para estructurar este proceso es la guía de manejo de incidentes del NIST, que define un
ciclo de cuatro fases — **preparación**, **detección y análisis**, **contención,
erradicación y recuperación**, y **actividad posterior al incidente** (NIST, 2012).

```
 Preparación ──> Detección y análisis ──> Contención / erradicación ──> Post-incidente
      ^                                        y recuperación                  │
      └────────────────────────────────────────────────────────────────────────┘
                    (lo aprendido retroalimenta la preparación)
```

## Políticas de seguridad

La meta principal de una política de seguridad es establecer los requerimientos para
proteger adecuadamente la infraestructura de cómputo y la información que contiene, junto
con los mecanismos concretos para cumplirlos. Deben desarrollarse después de un análisis
real de las necesidades de seguridad de la organización, no copiarse genéricamente de
otra empresa.

## Marcos de referencia reales para no improvisar

En vez de inventar políticas desde cero, la práctica profesional se apoya en marcos ya
establecidos y auditables:

| Marco | Qué aporta |
|---|---|
| **ISO/IEC 27001** | Estándar internacional para sistemas de gestión de seguridad de la información (SGSI): cómo estructurar, auditar y certificar la seguridad de una organización (ISO/IEC 27001:2022). |
| **NIST Cybersecurity Framework (CSF)** | Marco de referencia con cinco funciones — Identificar, Proteger, Detectar, Responder, Recuperar — para organizar la postura de seguridad de cualquier organización (NIST, 2024). |
| **IEC 62443** | Enfocado específicamente en seguridad de sistemas de control industrial y tecnología operativa (OT/ICS) — relevante cuando la red conecta no solo oficinas, sino equipo industrial. |

Nota para entornos industriales (OT/ICS): a diferencia de una red de oficina, donde suele
priorizarse la confidencialidad, en redes que controlan procesos industriales la prioridad
habitual es la **disponibilidad** — una planta que se detiene por una medida de
seguridad mal aplicada puede ser más costosa y riesgosa que la amenaza que se intentaba
prevenir. IEC 62443 está diseñado teniendo en cuenta esa prioridad distinta.

## Ejemplo: el firewall que "ya resolvió" la seguridad

Una empresa instala un firewall perimetral robusto y, con eso, considera el tema de
seguridad de red resuelto. Meses después, un empleado abre por error un archivo adjunto
malicioso recibido por correo — el ataque no vino de afuera hacia adentro (que es lo que
el firewall vigila), sino que se activó desde dentro de la propia red, un vector que la
prevención perimetral no está diseñada para frenar.

Como no existe ningún sistema de **detección de intrusos** corriendo internamente, el
malware se propaga varios días sin que nadie lo note, hasta que empieza a generar tráfico
saliente inusual hacia un servidor externo desconocido — un patrón que sí habría disparado
una alarma temprana si hubiera existido monitoreo de tráfico interno.

Cuando finalmente se detecta (por un pico anómalo de tráfico, no por una alerta dedicada),
la organización tampoco tiene un proceso de **respuesta a incidentes** ya definido: nadie
sabe con claridad quién debe aislar los equipos afectados, a quién avisar primero, ni cómo
preservar evidencia antes de simplemente "limpiar" las máquinas — lo que habría permitido
seguir el ciclo de preparación → detección y análisis → contención/erradicación/
recuperación → actividad posterior al incidente. Al no existir ese proceso preparado de
antemano, la respuesta se improvisa: se reinician equipos apresuradamente, se pierde
evidencia sobre cómo entró el ataque, y semanas después ocurre un incidente similar,
porque nunca hubo una fase de "actividad posterior" que retroalimentara la preparación.

Todo esto habría estado cubierto por una **política de seguridad** formal que exigiera,
desde antes del incidente, las cuatro piezas juntas — no solo la prevención en el
perímetro. El mismo razonamiento cambia de énfasis en un entorno industrial (OT/ICS): ahí,
antes de aislar un equipo comprometido de forma automática, hay que considerar que
desconectarlo podría detener un proceso físico en marcha — por eso frameworks como
**IEC 62443** priorizan explícitamente la disponibilidad del proceso sobre la
confidencialidad de la información al diseñar la respuesta a un incidente en ese tipo de
red.

## Cómo se resuelve con herramientas reales

Las cuatro piezas del ejemplo tienen contraparte real en Linux — el mismo sistema de tu VM
de Ubuntu en VirtualBox, donde más adelante en el curso vas a instalar y configurar
algunas de ellas:

- **Prevención**: `ufw` (Uncomplicated Firewall) es la interfaz simplificada de Ubuntu
  sobre el firewall del kernel de Linux (Canonical Ltd., s.f.) — con reglas legibles como
  `ufw allow 443` en vez de sintaxis más compleja, aplica el mismo concepto de control de
  acceso perimetral del ejemplo.
- **Detección**: **Suricata** (o su predecesor histórico, Snort, ya citado arriba) es un
  IDS/IPS de código abierto que analiza tráfico en tiempo real contra reglas de ataques
  conocidos — instalado internamente, habría detectado el tráfico saliente inusual del
  ejemplo mucho antes de que se volviera un pico visible (OISF, s.f.).
- **Respuesta automatizada a un patrón sospechoso**: **fail2ban** vigila registros (los
  mismos logs de `rsyslog` vistos en la Sesión 3) y, al detectar un patrón definido — por
  ejemplo, muchos intentos fallidos de acceso — banea automáticamente la IP de origen,
  sin esperar a que un humano lo note (fail2ban, s.f.).

```
Tráfico interno         Suricata (detección)         fail2ban (respuesta automática)
     │                        │                              │
     │── tráfico anómalo ────>│── genera alerta ────────────>│── banea la IP de origen
                                                               │   (respuesta inmediata,
                                                               │    antes de intervención
                                                               │    humana)
```

Ninguna de estas tres reemplaza a las otras: `ufw` no detecta nada que ya haya entrado,
Suricata no bloquea nada por sí solo si no está integrado con algo como fail2ban, y ambas
dependen de que exista, detrás, una política de seguridad que defina qué reglas aplicar y
cómo debe proceder el equipo cuando salten. Es la misma idea de "las cuatro piezas juntas"
del ejemplo, ahora con nombre de herramienta concreta.

---

## Referencias

International Organization for Standardization. (1989). *Information technology — Open
Systems Interconnection — Basic Reference Model — Part 4: Management framework* (ISO/IEC
7498-4:1989).

International Organization for Standardization. (2022). *Information security,
cybersecurity and privacy protection — Information security management systems —
Requirements* (ISO/IEC 27001:2022).

National Institute of Standards and Technology. (2012). *Computer security incident
handling guide* (NIST Special Publication 800-61, revisión 2). U.S. Department of
Commerce. https://doi.org/10.6028/NIST.SP.800-61r2

National Institute of Standards and Technology. (2024). *The NIST Cybersecurity Framework
(CSF) 2.0*. U.S. Department of Commerce. https://doi.org/10.6028/NIST.CSWP.29

Canonical Ltd. (s.f.). *UFW — Uncomplicated Firewall*. Ubuntu Server documentation.
https://help.ubuntu.com/community/UFW

Open Information Security Foundation (OISF). (s.f.). *Suricata user guide*.
https://docs.suricata.io/

fail2ban. (s.f.). *Fail2ban documentation*. https://github.com/fail2ban/fail2ban/wiki

Roesch, M. (1999). Snort — Lightweight intrusion detection for networks. *Proceedings of
the 13th USENIX Conference on System Administration (LISA '99)*, 229–238.
