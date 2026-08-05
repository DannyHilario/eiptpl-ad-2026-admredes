# Cliente-Servidor y qué es un Servidor

> 50 min · [← Interfaces de usuario](03-interfaces-usuario.md)

## Qué es un servidor

Un **servidor** no es un tipo de hardware distinto y misterioso — es una computadora con
exactamente las mismas piezas que ya conoces (Sesión 1: CPU, RAM, almacenamiento, etc.) y
el mismo tipo de software (Sesión 2: sistema operativo, procesos, servicios), pero
**dedicada** a una tarea concreta: **responder a peticiones de otras computadoras**, casi
siempre encendida las 24 horas del día.

Lo que hace que una computadora "sea" un servidor no es una pieza física especial, sino
su **rol**: corre uno o varios **servicios** (repaso de la Sesión 2 — procesos sin
ventana, siempre esperando) que atienden peticiones constantemente. Un servidor web, un
servidor de correo, un servidor de base de datos son, en el fondo, una computadora con un
servicio corriendo que sabe responder un tipo específico de petición.

**Forma física real:** muchos servidores no se ven como una PC de escritorio — vienen en
formato **rack** (delgados, horizontales, se apilan en un gabinete metálico) para caber
muchos en poco espacio en un centro de datos. Ejemplos reales: Dell PowerEdge R750, HPE
ProLiant DL380. Por dentro, siguen teniendo CPU, RAM, almacenamiento y fuente de poder —
como vimos en la Sesión 1, con las prioridades de servidor (muchos núcleos, RAM ECC,
almacenamiento redundante).

## Arquitectura cliente-servidor

Es el patrón que sostiene casi todo lo que haces en Internet y en una red local:

```
   Cliente                              Servidor
     │                                      │
     │  ── "necesito algo específico" ──>   │   (petición)
     │                                      │
     │  <── "aquí está / aquí está el       │   (respuesta)
     │       error / no encontrado" ──      │
     │                                      │
```

- **Cliente:** el que **inicia** la conversación, pidiendo algo puntual. Tu navegador, tu
  celular, tu aplicación de correo.
- **Servidor:** el que **espera** peticiones y responde. Siempre encendido, siempre
  "escuchando" (mismo concepto de servicio de la Sesión 2).

La relación es asimétrica a propósito: un servidor puede atender a miles de clientes al
mismo tiempo, pero un cliente típico solo habla con el servidor cuando necesita algo — no
está esperando peticiones de nadie.

*(Existe otra arquitectura, punto a punto/P2P, sin roles fijos — la vas a ver a detalle en
la Introducción del curso. Cliente-servidor es, por mucho, la más común.)*

## Cómo funciona una petición, a alto nivel

Sin importar de qué sea la petición (una página web, un correo, un archivo), el patrón
general siempre tiene estos 4 pasos:

1. **El cliente prepara la petición:** decide qué necesita y a qué servidor pedírselo.
2. **El cliente envía la petición.**
3. **El servidor procesa la petición:** puede necesitar buscar información, calcular algo,
   o consultar otra fuente antes de poder responder.
4. **El servidor envía la respuesta**, y el cliente hace algo con ella (mostrarla,
   guardarla, usarla).

Vas a ver este mismo patrón de 4 pasos repetirse, con detalles técnicos distintos, en
cada protocolo que estudies el resto del semestre (DNS, DHCP, correo, FTP) — todos son
variaciones del mismo patrón cliente-servidor.

## El ejemplo más clásico: navegar una página web

Cuando escribes una dirección en tu navegador:

```
Tú (cliente, navegador)                         Servidor web
        │                                             │
        │ ── "mándame el contenido de esta página" ─> │
        │                                             │
        │ <── el contenido de la página ────────────  │
        │
   El navegador dibuja la página en tu pantalla
```

En algún lugar del mundo (o de tu propia red local) existe una computadora encendida
24/7, corriendo un **servicio de servidor web**, con los archivos de esa página guardados
en su almacenamiento. Tu navegador es el cliente: pide esos archivos, el servidor los
encuentra y los manda de regreso, y tu navegador los convierte en lo que ves en pantalla.

Este ejemplo, aquí, es intencionalmente el resumen de alto nivel. Todo lo que falta —
cómo tu computadora encuentra la dirección exacta de ese servidor en la red, cómo viaja
la información realmente, qué protocolo específico se usa — es exactamente el contenido
de la [Introducción](../introduccion/01-redes.md) del curso, empezando por su Sesión 1.

---

## Ejercicio práctico: observa una petición real (Windows 11 + Edge)

1. Abre **Microsoft Edge** (viene instalado por defecto en Windows 11).
2. Visita cualquier página, por ejemplo `www.uanl.mx`.
3. Presiona `F12` para abrir las **Herramientas de desarrollador**.
4. Ve a la pestaña **Red** (Network) — si ya estaba abierta, presiona `Ctrl + R` para
   recargar la página con las herramientas capturando.
5. Verás una lista de peticiones — cada línea es un cliente (tu navegador) pidiéndole algo
   a un servidor. Haz clic en la primera línea (normalmente el documento HTML principal):
   - **Estado (Status):** `200` significa que el servidor respondió exitosamente.
   - **Encabezados (Headers):** pestaña "Response Headers" — el servidor manda
     información extra junto con el contenido (qué tipo de servidor es, cuándo se generó
     la respuesta, etc.).
6. Cuenta cuántas peticiones distintas hizo el navegador para cargar una sola página — casi
   siempre son muchas más de una (imágenes, estilos, scripts), cada una siguiendo el mismo
   patrón cliente-servidor por separado.

---

Con esto completas el módulo de Fundamentos de Cómputo: hardware, software/sistema
operativo, y ahora cómo dos computadoras se comunican bajo el patrón cliente-servidor.

**Siguiente paso:** [Introducción — Sesión 1: Redes](../introduccion/01-redes.md).

## Referencias

Kurose, J. F., & Ross, K. W. (2017). *Computer networking: A top-down approach* (7a ed.).
Pearson.
