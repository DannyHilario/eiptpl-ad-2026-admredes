# FTP

> 50 min · [← HTTPS](05-https.md) · Siguiente: [Correo electrónico: SMTP, POP3 e IMAP](07-correo.md)

## Transferir archivos, no páginas

Las sesiones de WWW y HTTPS fueron sobre ver páginas web. Esta sesión es sobre otra
necesidad igual de vieja: **mover archivos completos** de una computadora a otra — subir
un sitio web a su servidor, descargar un respaldo, compartir archivos grandes entre áreas.
El protocolo clásico para eso es **FTP** (File Transfer Protocol), definido en el
RFC 959 (Postel & Reynolds, 1985). Es de 1985 y sigue existiendo, pero como vas a ver, hoy
casi nunca se recomienda usarlo tal cual.

## Qué es un servidor FTP

Un **servidor FTP** es un programa que escucha conexiones de clientes y les permite dos
operaciones básicas:

- **Obtener** (descargar, `get`/`RETR`): el archivo viaja del servidor a tu equipo.
- **Poner** (subir, `put`/`STOR`): el archivo viaja de tu equipo al servidor.

Además de eso, el cliente puede listar directorios, cambiar de carpeta, renombrar y borrar
archivos — siempre que el servidor le dé permisos. Es el mismo modelo cliente-servidor que
ya conoces desde Fundamentos de Cómputo: el cliente pide, el servidor atiende.

## Dos conexiones en vez de una

Lo que hace a FTP distinto de casi cualquier otro protocolo del curso es que usa **dos
conexiones TCP separadas** (Postel & Reynolds, 1985):

| Conexión | Para qué sirve | Puerto del servidor |
|---|---|---|
| **Control** | Comandos y respuestas (`USER`, `PASS`, `LIST`, `RETR`...) | **21** |
| **Datos** | El contenido real: el archivo o el listado del directorio | **20** (modo activo) o uno alto negociado (modo pasivo) |

La conexión de control se mantiene abierta toda la sesión; la de datos se abre y se cierra
por cada transferencia. Las respuestas del servidor por el canal de control son códigos de
tres dígitos, parecidos a los de HTTP: `220` (servicio listo), `230` (sesión iniciada),
`530` (no autenticado), `226` (transferencia completa) (Postel & Reynolds, 1985).

```
  Cliente                                      Servidor FTP
 ┌─────────┐   Control (TCP 21): comandos     ┌─────────────┐
 │         │ ───── USER ana / PASS ... ─────> │             │
 │         │ <──── 230 Login successful ───── │             │
 │         │                                  │             │
 │         │   Datos (otra conexión TCP)      │             │
 │         │ <════ contenido del archivo ════ │             │
 └─────────┘                                  └─────────────┘
```

## Modo activo vs. modo pasivo

La pregunta es: **¿quién abre la conexión de datos?** De eso depende el modo.

- **Modo activo** (el original): el cliente le dice al servidor, con el comando `PORT`,
  "conéctate tú a mí en este puerto". El servidor abre la conexión de datos **desde su
  puerto 20 hacia el cliente** (Postel & Reynolds, 1985).
- **Modo pasivo**: el cliente envía `PASV` y el servidor responde "conéctate tú a mí en
  este puerto alto". El **cliente abre ambas conexiones**, la de control y la de datos
  (Bellovin, 1994).

```
MODO ACTIVO                              MODO PASIVO
Cliente            Servidor              Cliente            Servidor
  │ ──── control ──> :21                   │ ──── control ──> :21
  │ PORT (mi puerto 5001)                  │ PASV
  │                                        │ <── "usa mi puerto 40000"
  │ <── datos ──── :20                     │ ──── datos ───> :40000
  (el SERVIDOR inicia los datos)           (el CLIENTE inicia todo)
```

¿Por qué existe el modo pasivo? Por la **caseta** del fraccionamiento. Tu laptop está
dentro de una red privada, detrás del router (la caseta), que deja salir conexiones
iniciadas desde adentro pero no deja entrar visitas que nadie pidió. En modo activo, el
servidor intenta "entrar" al fraccionamiento para entregar los datos, y la caseta lo
rechaza: la sesión se conecta, el login funciona, pero el listado o la descarga se quedan
colgados. En modo pasivo, todas las conexiones salen desde adentro, así que la caseta las
deja pasar sin problema. Por eso hoy los clientes FTP usan modo pasivo por default
(Bellovin, 1994).

El costo se lo lleva el servidor: tiene que abrir en **su** firewall no solo el puerto 21,
sino también el rango de puertos altos que usa para modo pasivo.

## El problema de FTP: todo viaja en texto claro

FTP se diseñó en una época en la que la red era de confianza. Por eso **no cifra nada**:
el usuario, la contraseña y los archivos viajan tal cual por la red. Cualquiera que pueda
capturar el tráfico en el camino (en la misma Wi-Fi, en un switch comprometido) puede
leer las credenciales directamente de los paquetes. El RFC 2577, dedicado a la seguridad
de FTP, ya advertía de esto y de otros problemas del protocolo (Allman & Ostermann, 1999).

```
Lo que ve alguien capturando tráfico en la conexión de control:

USER ana
331 Please specify the password.
PASS MiContraseña2026          <-- la contraseña, legible tal cual
230 Login successful.
```

En términos de la Etapa 1, esto es un problema de **administración de seguridad** (la S de
FCAPS): un servicio que expone credenciales en la red es un riesgo, aunque "funcione".

## Las alternativas seguras: FTPS y SFTP

Hay dos reemplazos que suelen confundirse por el nombre, pero son cosas muy distintas:

| | **FTPS** | **SFTP** |
|---|---|---|
| Qué es | El mismo FTP, envuelto en TLS | Un protocolo distinto, que corre dentro de SSH |
| Estándar | RFC 4217 (Ford-Hutchinson, 2005) | Borrador IETF, nunca publicado como RFC (Ylonen & Lehtinen, 2001) |
| Puertos | 21 + puertos de datos (sigue usando dos conexiones) | **22**, una sola conexión |
| Cifrado | TLS, igual que HTTPS | El de SSH |
| Problema con firewalls | El mismo que FTP (rango pasivo) | Ninguno: una conexión, un puerto |

- **FTPS** agrega cifrado TLS — el mismo que viste en la sesión de HTTPS — a las dos
  conexiones de FTP. El cliente lo pide con el comando `AUTH TLS` (Ford-Hutchinson, 2005).
  Conserva la complejidad de FTP: dos conexiones y rango de puertos pasivos.
- **SFTP** (SSH File Transfer Protocol) **no es FTP**, aunque el nombre lo sugiera. Es un
  **subsistema de SSH**: un servicio que corre dentro de una conexión SSH ya cifrada y
  autenticada (Ylonen & Lonvick, 2006). Si un servidor ya tiene SSH (el caso de casi
  cualquier servidor Linux), ya tiene SFTP disponible sin instalar nada más.

En la práctica actual, **SFTP es la opción por default** para transferir archivos a un
servidor Linux. FTP plano solo tiene sentido en casos muy acotados (por ejemplo, descargas
públicas anónimas donde no hay contraseña que proteger), y FTPS aparece sobre todo cuando
un sistema heredado solo habla FTP.

## Ejemplo: el sitio web "hackeado" del área de comunicación

Una escuela tiene su sitio web en un servidor Linux. Desde hace años, el área de
comunicación lo actualiza subiendo archivos por FTP: un encargado abre su cliente FTP en la
laptop, se conecta al puerto 21 con su usuario y contraseña, y sube las páginas nuevas.
Funciona, y nadie lo ha cuestionado.

Un día el sitio amanece con una página falsa en la portada. Nadie del área la subió. Al
revisar los registros del servidor, aparece un inicio de sesión FTP válido — con el
usuario y la contraseña correctos del encargado — desde una dirección IP desconocida, en la
madrugada. No hubo "adivinanza" de contraseña: entraron a la primera.

Investigando hacia atrás, se descubre que unas semanas antes el encargado actualizó el
sitio desde la Wi-Fi abierta de una cafetería. Como FTP manda `USER` y `PASS` en texto
claro, cualquiera conectado a esa misma red con un capturador de tráfico pudo leer la
contraseña tal cual pasaba. A eso se suma otro detalle: el servidor tiene abiertos en su
firewall el puerto 21 y un rango amplio de puertos altos "porque si no, FTP no funciona",
algo que nadie sabía explicar bien.

El problema no fue una contraseña débil ni un error del encargado: fue usar un protocolo
que, por diseño, no protege las credenciales en el camino.

## Cómo se resuelve con herramientas reales

En tu **VM de Ubuntu Server en VirtualBox** ya tienes instalado **OpenSSH**, y con él ya
tienes la solución principal del ejemplo, sin instalar nada adicional:

- **`sftp`** (cliente de OpenSSH): se conecta al mismo servidor SSH por el puerto 22 y
  permite `get`, `put`, `ls` y `cd` igual que un cliente FTP, pero con todo cifrado
  (OpenBSD, s.f.). Con SFTP, la escuela puede cerrar el puerto 21 y todo el rango pasivo en
  el firewall y dejar solo el 22, que de todas formas ya usaba para administrar el servidor.

```
$ sftp ana@192.168.56.10
Connected to 192.168.56.10.
sftp> put index.html /var/www/html/
Uploading index.html to /var/www/html/index.html
sftp> bye
```

- **vsftpd** ("very secure FTP daemon"): es el servidor FTP que la documentación oficial
  de Ubuntu Server usa como referencia (Canonical, s.f.; Evans, s.f.). Sirve para los casos
  donde de verdad hace falta FTP — por ejemplo, un equipo heredado que solo habla FTP — y
  puede configurarse para exigir TLS (es decir, convertirse en FTPS) y fijar un rango
  pequeño y controlado de puertos pasivos en lugar de abrir medio firewall.

La regla práctica que vas a ver repetida en el curso: si el servidor es Linux y tiene SSH,
la transferencia de archivos se hace por **SFTP**. FTP plano se queda como algo que hay que
saber reconocer — sobre todo para saber por qué ya no se usa.

---

## Referencias

Allman, M., & Ostermann, S. (1999). *FTP security considerations* (RFC 2577). IETF.
https://www.rfc-editor.org/rfc/rfc2577

Bellovin, S. (1994). *Firewall-friendly FTP* (RFC 1579). IETF.
https://www.rfc-editor.org/rfc/rfc1579

Canonical. (s.f.). *Host files with an FTP server*. Ubuntu Server documentation.
https://documentation.ubuntu.com/server/how-to/networking/ftp/

Evans, C. (s.f.). *vsftpd — Secure, fast FTP server for UNIX-like systems*.
https://security.appspot.com/vsftpd.html

Ford-Hutchinson, P. (2005). *Securing FTP with TLS* (RFC 4217). IETF.
https://www.rfc-editor.org/rfc/rfc4217

OpenBSD. (s.f.). *sftp(1) — OpenSSH secure file transfer*. OpenBSD manual pages.
https://man.openbsd.org/sftp

Postel, J., & Reynolds, J. (1985). *File transfer protocol (FTP)* (RFC 959). IETF.
https://www.rfc-editor.org/rfc/rfc959

Ylonen, T., & Lehtinen, S. (2001). *SSH file transfer protocol*
(draft-ietf-secsh-filexfer-02) [Borrador de Internet]. IETF.
https://datatracker.ietf.org/doc/html/draft-ietf-secsh-filexfer-02

Ylonen, T., & Lonvick, C. (Ed.). (2006). *The Secure Shell (SSH) connection protocol*
(RFC 4254). IETF. https://www.rfc-editor.org/rfc/rfc4254
