# Administración de configuración

> 50 min · [← Fundamentos de terminal](../fundamentos-computo/08-terminal.md) · Siguiente: [Administración de rendimiento](02-administracion-rendimiento.md)

## El mapa de toda la Etapa 1: el modelo FCAPS

Antes de entrar al primer tema, vale la pena ver el mapa completo, porque los cinco temas
de esta etapa no son una lista arbitraria: son un modelo estándar de la industria llamado
**FCAPS** (Fault, Configuration, Accounting, Performance, Security), definido originalmente
por la ISO como parte del marco de gestión OSI (ISO/IEC 7498-4:1989) y retomado después
por la UIT en su modelo de Red de Gestión de Telecomunicaciones, **TMN** (ITU-T, 2000).
Cada sesión de esta etapa es una de esas cinco letras:

```
        F C A P S
        │ │ │ │ │
        │ │ │ │ └─ Security     (Sesión 5 — seguridad)
        │ │ │ └─── Performance  (Sesión 2 — rendimiento)
        │ │ └───── Accounting  (Sesión 4 — contabilidad)
        │ └─────── Configuration (esta sesión)
        └───────── Fault       (Sesión 3 — fallas)
```

El orden del manual del curso no sigue el orden de las siglas (F-C-A-P-S), pero cubre las
cinco exactamente. Tenlo presente: cuando en el trabajo real alguien mencione "FCAPS", ya
sabes que se refiere a estas mismas cinco funciones que vas a ver a detalle, una por
sesión.

## Por qué administrar una red no es solo "que funcione"

Una red pequeña (tu casa, unas cuantas computadoras) casi se administra sola: enciendes el
router y ya. El problema aparece cuando la red crece: cientos de switches, routers y
servidores, en varios edificios o hasta en la nube, cada uno con su propia configuración.
Sin un proceso ordenado, cada cambio manual es una apuesta — nadie recuerda con certeza
qué se cambió, cuándo, ni por qué (Subramanian, 2010; Clemm, 2006).

**Administración de configuración** es la función de FCAPS encargada exactamente de eso:
llevar control de la configuración de cada dispositivo de la red — qué tiene cada uno
configurado, cuándo cambió, y cómo regresar a un estado que sí funcionaba si algo sale
mal.

## El problema que resuelve: configuración sin control

```
Sin administración de configuración          Con administración de configuración
────────────────────────────────    ────────────────────────────────
Router A: cambio manual, sin registro     Router A ─┐
Router B: cambio manual, sin registro     Router B ─┼─> Sistema central que
Switch C: cambio manual, sin registro     Switch C ─┘   registra cada cambio,
                                                          quién lo hizo y cuándo
Nadie sabe qué configuración               Existe una "línea base" (baseline):
"buena" restaurar si algo falla            la última configuración que
                                            sí funcionaba, lista para restaurar
```

A esto se le llama **configuration drift** (deriva de configuración): la diferencia,
acumulada con el tiempo, entre lo que un dispositivo *debería* tener configurado y lo que
*realmente* tiene, por cambios manuales no documentados. Entre más dispositivos y más
tiempo pasa, más se acumula esa diferencia — y es exactamente lo que la administración de
configuración busca prevenir y corregir.

## Qué hace en la práctica

- **Inventario**: saber qué dispositivos existen en la red y qué configuración tiene cada
  uno, en todo momento (no solo cuando alguien pregunta).
- **Línea base (baseline)**: guardar una copia de la última configuración conocida que
  funcionaba bien, para poder regresar a ella si un cambio sale mal.
- **Control de cambios**: cada modificación queda registrada — quién, qué, cuándo y por
  qué — en vez de cambios manuales silenciosos.
- **Actualización remota**: aplicar cambios o actualizaciones de firmware a muchos
  dispositivos a la vez, sin tener que conectarse uno por uno físicamente.
- **Auditoría y cumplimiento**: poder demostrar, con evidencia, que la configuración de la
  red cumple con las políticas de la organización (relevante para la Sesión 5, seguridad).

## Una idea prestada de desarrollo de software: control de versiones

Si ya usas o has oído de **Git** para código, la administración de configuración de red
resuelve el mismo problema pero para configuraciones de red: cada cambio queda como una
versión con su historial, se puede comparar una versión contra otra, y se puede
regresar (`rollback`) a una versión anterior si algo se rompe. La diferencia es la
herramienta, no el concepto — herramientas de **Network Configuration Management (NCM)**
como Ansible, RANCID/Oxidized (open source) o soluciones comerciales como Cisco Prime
Infrastructure aplican esta misma idea a routers y switches en vez de a archivos de
código (Clemm, 2006).

```
Cambio de configuración, con control de versiones
───────────────────────────────────────────────
v1 (baseline) ──> v2 (cambio del lunes) ──> v3 (cambio del martes, con error)
                                                  │
                                                  v
                                     rollback automático a v2
                                     (la última que funcionaba)
```

## El ciclo de administración de configuración

```
 ┌──────────┐   ┌───────────┐   ┌────────────┐   ┌──────────────┐
 │ Descubrir│──>│ Establecer│──>│  Monitorear│──>│ Detectar     │
 │ inventario│   │ línea base│   │ continuo   │   │ desviación   │
 └──────────┘   └───────────┘   └────────────┘   └──────┬───────┘
       ^                                                │
       │                                                v
       └──────────────── Corregir / restaurar <─────────┘
```

## Ejemplo: el cambio de las 2 a.m. sin documentar

Una empresa con varias sucursales tiene un solo administrador de red para todos los
sitios. Una madrugada, un enlace entre dos sucursales empieza a fallar de forma
intermitente. El administrador, medio dormido y bajo presión, entra al switch principal
de una de las sucursales y cambia manualmente un parámetro de la configuración para
"apagar el incendio" — el enlace se estabiliza y todos vuelven a dormir. No documenta el
cambio en ningún lado: ni un ticket, ni un correo, ni una nota.

Tres meses después ese mismo switch falla por una razón distinta (una fuente de poder que
se descompone) y hay que reemplazarlo por uno nuevo. El equipo intenta restaurar "la
configuración de siempre" desde el respaldo automático más reciente que tenían — pero ese
respaldo es de **antes** del cambio de las 2 a.m., porque nadie generó uno nuevo después
del ajuste manual. El switch de reemplazo queda funcionando con una configuración que
técnicamente es correcta según el respaldo, pero que ya no coincide con lo que la red
necesitaba desde aquella madrugada: el enlace entre sucursales vuelve a fallar
intermitentemente, y nadie en el equipo recuerda con exactitud qué fue lo que se cambió
para arreglarlo la primera vez.

Esto es **configuration drift** sin control de cambios, tal cual se describió arriba: la
diferencia entre "lo que el respaldo dice que hay" y "lo que realmente hay" creció sin que
nadie lo registrara. Con un proceso real de administración de configuración —cada cambio
documentado, y una línea base actualizada inmediatamente después de aplicarlo— el
reemplazo del switch habría restaurado exactamente el estado funcional más reciente, no
uno de tres meses atrás.

Nótese además cómo este único incidente de configuración termina generando, en cascada,
una falla (F, Sesión 3) que en realidad tiene su causa raíz en un problema de
configuración no controlada — la razón por la que FCAPS trata estas cinco funciones como
relacionadas entre sí, no como compartimentos aislados.

## Cómo se resuelve con herramientas reales

Este problema tiene nombre y solución estándar en la industria: tratar la configuración de
red igual que se trata el código fuente, con **control de versiones**. En vez de que el
administrador edite el switch a mano a las 2 a.m., la línea base completa de la red vive
guardada como texto en un repositorio **Git**, y los cambios se aplican con una herramienta
de automatización — la más usada hoy en día es **Ansible** (Red Hat, s.f.): se describe en
un archivo (un "playbook") el estado deseado del dispositivo, y Ansible se conecta por SSH
y aplica exactamente ese estado, sin que nadie tenga que recordar de memoria qué se
cambió.

```
git (historial de configuraciones)        Ansible (aplica el estado deseado)
──────────────────────────────            ──────────────────────────────────
v1: config original                       playbook.yml describe "cómo debe
v2: cambio documentado del lunes           quedar" el switch
v3: cambio de la urgencia — SÍ registrado         │
    (commit con fecha, autor y motivo)             v
                                            ansible-playbook aplica v3 y deja
                                            registro exacto de qué cambió
```

Todavía no conoces Ansible ni Git a fondo — los vas a ver más adelante en el curso, ya con
tu propia **VM de Ubuntu en VirtualBox** (instalada en Fundamentos de Cómputo). Por ahora
quédate con la idea, no con la herramienta: **cada cambio de configuración debería quedar
registrado en algún lugar consultable, no solo en la memoria de quien lo hizo**. Con eso
resuelto, restaurar el switch de reemplazo del ejemplo habría sido cuestión de volver a
aplicar la última versión guardada — la de después del cambio de las 2 a.m., no la de tres
meses atrás.

---

## Referencias

International Organization for Standardization. (1989). *Information technology — Open
Systems Interconnection — Basic Reference Model — Part 4: Management framework* (ISO/IEC
7498-4:1989).

International Telecommunication Union. (2000). *TMN management functions* (ITU-T
Recommendation M.3400). ITU-T.

Subramanian, M. (2010). *Network management: Principles and practice* (2a ed.). Pearson
Addison-Wesley.

Clemm, A. (2006). *Network management fundamentals*. Cisco Press.

Red Hat, Inc. (s.f.). *Ansible documentation*. https://docs.ansible.com/
