# Administración de contabilidad

> 50 min · [← Administración de fallas](03-administracion-fallas.md) · Siguiente: [Administración de seguridad](05-administracion-seguridad.md)

## La A de FCAPS

Esta es la **A** (Accounting) del modelo visto en la Sesión 1. Es la función de FCAPS que
más se presta a confusión por el nombre: **no es contabilidad financiera de la empresa**
— es llevar registro de **quién usó qué recursos de la red, cuánto, y durante cuánto
tiempo**. En inglés a veces se le llama también *usage accounting* para evitar esa
confusión.

## Qué controla

- Vigilar el uso indebido de privilegios de acceso a la red.
- Evitar que un usuario sobrecargue la red en perjuicio de los demás.
- Detectar uso ineficiente de los recursos de red.
- Planificar el crecimiento de la red con datos reales de uso, no estimaciones.
- Ajustar o revocar derechos de acceso según el uso real observado.

## Por qué importa incluso sin cobrar por el servicio

El manual original de este curso liga la contabilidad casi exclusivamente a la
**tarificación** (cobrar a clientes por uso), como hacen los proveedores de servicio de
Internet (ISP). Eso es real y es el caso más visible, pero es solo un uso de esta
función — no todas las organizaciones cobran por el ancho de banda que usan sus propios
empleados. Aun sin facturar a nadie, la contabilidad de red sigue siendo indispensable
para tres cosas que sí aplican en cualquier organización:

1. **Resolución de problemas**: si una parte de la red está saturada, los registros de uso
   dicen quién o qué la está saturando.
2. **Planificación de capacidad**: decidir dónde invertir en más ancho de banda requiere
   saber dónde se está usando de verdad, no adivinarlo.
3. **Auditoría y cumplimiento**: poder demostrar quién accedió a qué recurso y cuándo,
   relevante directamente para la Sesión 5 (seguridad).

## El estándar real detrás de esta función: RADIUS Accounting

El manual describe los datos que se recopilan (tiempo de conexión, mensajes
transmitidos, punto de acceso, razón de desconexión) sin nombrar cómo se implementa esto
en la práctica. Existe un protocolo estándar, ampliamente usado en redes reales, dedicado
exactamente a esto: **RADIUS Accounting** (RFC 2866; Rigney, 2000), la extensión de
contabilidad del protocolo RADIUS que ya se usa para autenticar el acceso a redes
corporativas y Wi-Fi empresarial.

RADIUS Accounting registra, entre otros, estos datos por cada sesión de un usuario:

| Dato (atributo RADIUS) | Qué representa |
|---|---|
| `Acct-Session-Time` | Duración total de la conexión |
| `Acct-Input-Octets` / `Acct-Output-Octets` | Bytes recibidos y enviados durante la sesión |
| `NAS-IP-Address` | Punto de acceso a la red por el que entró el usuario |
| `Acct-Terminate-Cause` | Razón por la que terminó la conexión (desconexión normal, error, timeout...) |

```
   Cliente                Punto de acceso a           Servidor RADIUS
   (usuario)               la red (NAS)              (contabilidad)
     │                          │                          │
     │── se conecta ──────────>│                          │
     │                          │── Accounting-Start ────>│  (registra inicio)
     │   ... sesión activa ...  │                          │
     │── se desconecta ────────>│                          │
     │                          │── Accounting-Stop ─────>│  (registra fin,
     │                          │   (duración, bytes,      │   duración, bytes,
     │                          │    causa)                │   causa)
```

Este es el mismo mecanismo, por ejemplo, que permite a un ISP saber cuántos GB consumió
cada cliente en el mes, o a una empresa saber cuánto tiempo estuvo conectado cada
empleado a la red Wi-Fi corporativa — sin necesidad de que exista un cobro de por medio.

## Relación con planificación de capacidad

La estadística de uso que produce esta función alimenta directamente decisiones que
parecen pertenecer a otras áreas de FCAPS: sin datos de contabilidad, la administración de
rendimiento (Sesión 2) no tiene con qué justificar una ampliación de ancho de banda, y la
administración de fallas (Sesión 3) pierde una fuente de evidencia para diagnosticar
saturación causada por uso excesivo de un usuario o aplicación en particular.

## Ejemplo: la Wi-Fi corporativa que "se satura todas las tardes"

Una empresa mediana, sin cobrar nada a sus empleados por usar la red Wi-Fi interna,
empieza a recibir quejas de que la conexión se pone lenta todas las tardes entre 2 y 4
p.m. No hay ningún cobro de por medio, así que en un inicio nadie piensa en "contabilidad"
como parte del problema — se trata como un asunto de rendimiento (Sesión 2).

El equipo de red revisa los registros de **RADIUS Accounting** de las últimas semanas:
cada sesión Wi-Fi generó un registro con `Acct-Session-Time` (duración), `Acct-Input-Octets`
/ `Acct-Output-Octets` (bytes transferidos) y `NAS-IP-Address` (por qué punto de acceso
entró). Al ordenar esos registros por bytes transferidos durante esa ventana de 2 a 4
p.m., aparece un patrón claro: una sola cuenta de un empleado del área de diseño
consume, todas las tardes, varias veces más ancho de banda que el resto de la empresa
junta — coincide con la hora en que ese empleado sube archivos pesados a un servicio de
almacenamiento en la nube.

Sin los registros de contabilidad, el diagnóstico se habría quedado en "la red está
saturada por las tardes" sin poder decir *por quién* ni *por qué* — la misma clase de
callejón sin salida que ilustra el ejemplo de la Sesión 2, pero aquí resuelto no con
NetFlow sino con datos de uso por usuario. Con el dato concreto en mano, la solución deja
de ser "comprar más ancho de banda para todos" (una decisión cara y no dirigida al
problema real) y pasa a ser una conversación puntual sobre cuándo programar esas subidas
pesadas — exactamente el tipo de decisión de planificación de capacidad que la
contabilidad hace posible, sin que exista ningún cobro de por medio.

## Cómo se resuelve con herramientas reales

El servidor RADIUS que realmente implementa esto en la mayoría de redes corporativas y
académicas del mundo es **FreeRADIUS**, la implementación de código abierto más usada del
protocolo (The FreeRADIUS Project, s.f.) — instalable en un servidor Linux como el que
correría dentro de tu VM de Ubuntu en VirtualBox más adelante en el curso. FreeRADIUS
puede configurarse para guardar cada registro de contabilidad (`Acct-Start`,
`Acct-Stop`, con sus atributos de bytes y duración) directamente en una base de datos SQL
en vez de solo en un archivo de texto — típicamente MySQL o PostgreSQL.

```
Punto de acceso Wi-Fi          FreeRADIUS               Base de datos SQL
      │                            │                          │
      │── Accounting-Stop ────────>│── INSERT registro ─────>│  sesión, bytes,
      │   (usuario, bytes, hora)   │                          │  hora, NAS
                                                               │
                                          Consulta SQL:
                                          "¿quién transfirió más
                                           bytes entre 2 y 4 p.m.?"
```

Con los registros en una base de datos, la pregunta del ejemplo ("¿quién está saturando
la red por las tardes?") deja de requerir revisar manualmente archivos de log: se
resuelve con una sola consulta SQL que ordena a los usuarios por bytes transferidos en esa
ventana de tiempo — el mismo tipo de consulta que, en Fundamentos de Cómputo, hiciste de
forma más simple con `grep` sobre un archivo de texto.

---

## Referencias

The FreeRADIUS Project. (s.f.). *FreeRADIUS documentation*. https://freeradius.org/documentation/

Rigney, C. (2000). *RADIUS accounting* (RFC 2866). IETF.
https://www.rfc-editor.org/rfc/rfc2866

International Organization for Standardization. (1989). *Information technology — Open
Systems Interconnection — Basic Reference Model — Part 4: Management framework* (ISO/IEC
7498-4:1989).

Subramanian, M. (2010). *Network management: Principles and practice* (2a ed.). Pearson
Addison-Wesley.
