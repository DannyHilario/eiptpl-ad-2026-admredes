# WWW y HTTP

> 50 min · [← DNS](03-dns.md) · Siguiente: [HTTPS](05-https.md)

## Repaso rápido: lo que ya viste en Redes y Telecomunicaciones

En tu curso anterior ya usaste HTTP sin pensarlo mucho: abrías el navegador, escribías una
dirección y aparecía una página. También viste que HTTP usa el puerto 80 y que existe una
versión "segura" llamada HTTPS. Esta sesión no reenseña eso desde cero — lo retoma para
verlo como lo ve **quien administra el servidor**, no quien navega: qué pide exactamente
el navegador, qué contesta el servidor, y qué estándar define cada pieza.

## WWW no es lo mismo que Internet

Es la confusión más común, y conviene dejarla clara desde el principio:

- **Internet** es la red de redes: la infraestructura física y lógica (cables, routers,
  direcciones IP) que conecta millones de redes entre sí. Es el conjunto de calles y
  carreteras.
- **La World Wide Web (WWW)** es *uno* de los servicios que corren sobre Internet: un
  sistema de documentos enlazados entre sí (hipertexto) que se piden y se entregan con el
  protocolo HTTP. El correo electrónico, FTP o los videojuegos en línea también usan
  Internet, pero no son "la web".

La web nació en el CERN (Suiza): Tim Berners-Lee la propuso en 1989 como un sistema para
que científicos de distintas instituciones compartieran información, y el 30 de abril de
1993 el CERN liberó su software al dominio público, lo que permitió que creciera hasta lo
que conoces hoy (CERN, s.f.).

```
┌─────────────────────────── Internet (la red de redes) ───────────────────────────┐
│                                                                                   │
│   WWW (HTTP/HTTPS)     Correo (SMTP/IMAP)     FTP      DNS      Juegos, VoIP...   │
│   └─ un servicio más   └─ otro servicio       ...      ...      ...               │
│                                                                                   │
└───────────────────────────────────────────────────────────────────────────────────┘
```

Con la analogía del fraccionamiento: Internet son todas las calles y carreteras que
conectan tu fraccionamiento con el resto del mundo, saliendo por la caseta (el router). La
web es solo *un tipo de visita* que circula por esas calles — la más común, pero no la
única.

## Las tres piezas de la web: URL, HTTP y HTML

Berners-Lee definió la web con tres piezas que siguen vigentes (CERN, s.f.):

| Pieza | Qué resuelve | Ejemplo |
|---|---|---|
| **URL** | *Dónde* está el recurso | `https://www.ejemplo.com/cursos/redes.html` |
| **HTTP** | *Cómo* se pide y se entrega | `GET /cursos/redes.html` |
| **HTML** | *Qué* forma tiene el documento | `<h1>Redes</h1>` |

Esta sesión se enfoca en las dos primeras — HTML es tema de desarrollo web, no de
administración de redes.

### Anatomía de una URL

Una **URL** (Uniform Resource Locator) es una dirección con partes bien definidas por el
estándar de identificadores de recursos (Berners-Lee et al., 2005):

```
  http://www.ejemplo.com:80/cursos/redes.html?grupo=3
  └┬─┘   └──────┬──────┘└┬┘└────────┬───────┘└──┬───┘
 esquema      host    puerto      ruta       consulta
 (protocolo) (se resuelve (opcional:  (qué recurso  (parámetros
              con DNS)    si no va,   dentro del    extra)
                          se usa 80)  servidor)
```

Fíjate en dos cosas que conectan con sesiones anteriores: el **host** se traduce a una
dirección IP con DNS (Sesión 3 de este módulo), y el **puerto** casi nunca se escribe
porque el esquema `http` ya implica el puerto TCP 80 por default (Fielding et al., 2022a).

## Cómo funciona HTTP: petición y respuesta

HTTP es un protocolo **cliente-servidor** (lo viste en Fundamentos de Cómputo, Sesión 4):
el cliente (navegador) envía una **petición** y el servidor web contesta con una
**respuesta**. Cada petición es independiente de la anterior — HTTP no "recuerda" por sí
mismo quién eres; para eso existen mecanismos adicionales como las cookies.

```
 Navegador (cliente)                                  Servidor web
 ┌──────────────┐  1. DNS: www.ejemplo.com -> IP      ┌──────────────┐
 │              │  2. Conexión TCP al puerto 80 ────> │              │
 │              │  3. GET /cursos/redes.html ───────> │  Busca el    │
 │              │                                     │  archivo     │
 │              │ <──── 4. 200 OK + contenido HTML ── │              │
 └──────────────┘                                     └──────────────┘
```

Una petición y su respuesta, tal como viajan en HTTP/1.1, son texto legible:

```
GET /cursos/redes.html HTTP/1.1        <-- método, ruta, versión
Host: www.ejemplo.com                  <-- cabecera obligatoria en HTTP/1.1

HTTP/1.1 200 OK                        <-- versión, código de estado
Content-Type: text/html                <-- qué tipo de contenido se entrega
Content-Length: 1532

<html>...                              <-- el contenido en sí
```

### Métodos: qué le pide el cliente al servidor

El estándar actual de la semántica de HTTP, RFC 9110, define los métodos principales
(Fielding et al., 2022a):

| Método | Qué hace | Uso típico |
|---|---|---|
| **GET** | Obtiene un recurso | Abrir una página |
| **HEAD** | Igual que GET, pero solo cabeceras, sin contenido | Revisar si un servidor responde |
| **POST** | Envía datos para que el servidor los procese | Enviar un formulario |
| **PUT** | Crea o reemplaza un recurso en la ruta indicada | APIs |
| **DELETE** | Elimina un recurso | APIs |

### Códigos de estado: qué contesta el servidor

Cada respuesta trae un número de tres dígitos. El primer dígito indica la **clase** de
respuesta (Fielding et al., 2022a) — y para quien administra un servidor es la primera
pista de qué está pasando:

| Clase | Significado | Códigos que vas a ver seguido |
|---|---|---|
| **1xx** | Informativo | (poco visibles para el usuario) |
| **2xx** | Éxito | **200** OK |
| **3xx** | Redirección | **301** movido permanentemente, **304** no modificado |
| **4xx** | Error del cliente | **403** prohibido, **404** no encontrado |
| **5xx** | Error del servidor | **500** error interno, **503** servicio no disponible |

La diferencia entre 4xx y 5xx es clave para diagnosticar: un **404** dice "pediste algo
que no existe" (el problema está en la petición), un **503** dice "el servidor no puede
atenderte ahora" (el problema está del lado del servidor) — conecta directo con la
administración de fallas de la Etapa 1.

## Versiones de HTTP: misma semántica, distinto transporte

Algo que el manual oficial no deja claro: desde 2022 el IETF separó HTTP en una
**semántica** común (métodos, códigos, cabeceras — RFC 9110) y varias formas de
transportarla. GET, 200 y 404 significan lo mismo en las tres versiones vigentes; lo que
cambia es *cómo viajan* los bytes:

| Versión | Estándar vigente | Transporte | Idea principal |
|---|---|---|---|
| **HTTP/1.1** | RFC 9112 (1ra versión: RFC 2068, 1997) | TCP | Texto legible; reutiliza conexiones |
| **HTTP/2** | RFC 9113 (2022) | TCP | Binario; muchas peticiones en paralelo en una conexión |
| **HTTP/3** | RFC 9114 (2022) | QUIC sobre UDP | Evita que un paquete perdido frene a todos |

(Fielding et al., 1997; Fielding et al., 2022b; Thomson & Benfield, 2022; Bishop, 2022)

```
 HTTP/1.1          HTTP/2            HTTP/3
 ┌────────┐       ┌────────┐       ┌────────┐
 │  HTTP  │       │  HTTP  │       │  HTTP  │   <-- misma semántica (RFC 9110)
 ├────────┤       ├────────┤       ├────────┤
 │  TCP   │       │  TCP   │       │  QUIC  │
 ├────────┤       ├────────┤       ├────────┤
 │   IP   │       │   IP   │       │  UDP   │
 └────────┘       └────────┘       ├────────┤
                                   │   IP   │
                                   └────────┘
```

Un dato práctico: HTTP/3 usa UDP, no TCP (Bishop, 2022). Si un firewall solo permite TCP
hacia el puerto 443, HTTP/3 no puede establecerse y el navegador cae de vuelta a HTTP/2 —
la página carga igual, solo que sin las ventajas de HTTP/3.

## Ejemplo: la página de inscripciones que "a veces no carga"

La preparatoria publica en un servidor web interno la página de inscripciones al
siguiente semestre. El primer día, varios alumnos reportan en el grupo de WhatsApp que "la
página no sirve". El reporte es tan vago como el "la red está lenta" de la Etapa 1: no
dice qué falló ni para quién.

Al preguntar con más detalle aparecen tres síntomas distintos. Un alumno ve una página
blanca que dice **404 Not Found**: copió el enlace de un mensaje viejo que apuntaba a
`/inscripciones2025.html`, una ruta que ya no existe en el servidor. Otro alumno ve **503
Service Unavailable** a las 8:00 a.m., justo cuando cientos de alumnos entraron al mismo
tiempo y el servidor dejó de aceptar conexiones nuevas. Un tercero dice que en su casa sí
carga, pero desde la red de la escuela no — y ahí ni siquiera hay código de estado,
porque la conexión nunca llega al servidor.

Son tres problemas diferentes que el usuario describe con la misma frase. El primero es
del cliente (una URL vieja), el segundo es de capacidad del servidor y el tercero es de
red (algo entre la escuela y el servidor bloquea el puerto). Sin mirar qué contesta
realmente el servidor — o si contesta — el administrador estaría adivinando cuál de los
tres atender.

## Cómo se resuelve con herramientas reales

En tu **VM de Ubuntu Server en VirtualBox** puedes tener las dos mitades del ejemplo, el
servidor y el cliente de diagnóstico:

- **nginx** o **Apache HTTP Server (httpd)**: los dos servidores web de código abierto más
  usados, ambos disponibles en los repositorios de Ubuntu (`apt install nginx` o
  `apt install apache2`). Guardan un **registro de accesos** (access log) con la ruta
  pedida y el código de estado de cada petición, así que el 404 de la URL vieja y la
  ráfaga de 503 de las 8:00 a.m. quedan registrados con hora exacta (NGINX, s.f.; The
  Apache Software Foundation, s.f.).
- **`curl -I`**: hace una petición **HEAD** y muestra solo la línea de estado y las
  cabeceras de la respuesta — la forma más rápida de ver, sin navegador, qué código
  devuelve el servidor para una URL (curl project, s.f.).

```
$ curl -I http://inscripciones.prepa.local/inscripciones2025.html
HTTP/1.1 404 Not Found              <-- problema del lado del cliente (ruta vieja)
Server: nginx

$ curl -I http://inscripciones.prepa.local/
curl: (7) Failed to connect ... port 80   <-- ni siquiera hubo respuesta HTTP:
                                              el problema está en la red
```

Si `curl` recibe un código, el servidor está vivo y el código dice dónde buscar; si ni
siquiera conecta, el problema está antes de HTTP — DNS, ruta o firewall. Con eso, la frase
"la página no sirve" se convierte en tres diagnósticos concretos.

---

## Referencias

Berners-Lee, T., Fielding, R., & Masinter, L. (2005). *Uniform resource identifier (URI):
Generic syntax* (RFC 3986). IETF. https://www.rfc-editor.org/rfc/rfc3986

Bishop, M. (Ed.). (2022). *HTTP/3* (RFC 9114). IETF.
https://www.rfc-editor.org/rfc/rfc9114

CERN. (s.f.). *The birth of the Web*.
https://home.cern/science/computing/the-birth-of-the-web

curl project. (s.f.). *curl.1 the man page*. https://curl.se/docs/manpage.html

Fielding, R., Gettys, J., Mogul, J., Frystyk, H., & Berners-Lee, T. (1997). *Hypertext
transfer protocol -- HTTP/1.1* (RFC 2068). IETF. https://www.rfc-editor.org/rfc/rfc2068

Fielding, R., Nottingham, M., & Reschke, J. (Eds.). (2022a). *HTTP semantics* (RFC 9110).
IETF. https://www.rfc-editor.org/rfc/rfc9110

Fielding, R., Nottingham, M., & Reschke, J. (Eds.). (2022b). *HTTP/1.1* (RFC 9112). IETF.
https://www.rfc-editor.org/rfc/rfc9112

NGINX. (s.f.). *nginx documentation*. https://nginx.org/en/docs/

The Apache Software Foundation. (s.f.). *Apache HTTP Server version 2.4 documentation*.
https://httpd.apache.org/docs/2.4/

Thomson, M., & Benfield, C. (Eds.). (2022). *HTTP/2* (RFC 9113). IETF.
https://www.rfc-editor.org/rfc/rfc9113
