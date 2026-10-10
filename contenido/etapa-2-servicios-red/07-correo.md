# Correo electrónico: SMTP, POP3 e IMAP

> 50 min · [← FTP](06-ftp.md) · Siguiente: [Instalación de Ubuntu Server](../etapa-3-sistema-operativo-red/01-instalacion-ubuntu-server/README.md)

## Un servicio, tres protocolos

Todos los servicios que llevamos en esta etapa se resolvían con un solo protocolo: DHCP
para direcciones, DNS para nombres, HTTP para páginas, FTP para archivos. El correo
electrónico es distinto: necesita **al menos tres protocolos** trabajando juntos, porque
enviar un mensaje y leerlo son dos problemas diferentes.

- **SMTP** se encarga de **enviar** y de mover el mensaje entre servidores.
- **POP3** o **IMAP** se encargan de que tú **leas** el mensaje que ya llegó a tu buzón.

Esta sesión cierra la Etapa 2 porque reúne casi todo lo anterior: el correo usa DNS para
encontrar al servidor destino, TLS (lo de la sesión de HTTPS) para cifrar, y el mismo
modelo cliente-servidor de siempre.

## Las piezas del sistema de correo: MUA, MTA y MDA

La arquitectura de correo de Internet nombra a cada participante según su función
(Crocker, 2009):

| Pieza | Nombre completo | Qué hace | Ejemplo |
|---|---|---|---|
| **MUA** | Mail User Agent | El programa con el que la persona escribe y lee correo | Thunderbird, Outlook, la app de correo del celular |
| **MSA** | Mail Submission Agent | Recibe el correo que el usuario quiere enviar, verifica que tenga permiso | El servidor de salida de tu cuenta |
| **MTA** | Mail Transfer Agent | Mueve el correo de un servidor a otro hasta el destino | Postfix |
| **MDA** | Mail Delivery Agent | Deja el mensaje en el buzón del destinatario | Dovecot (su componente de entrega) |

```
 Remitente                                                     Destinatario
┌────────┐ SMTP  ┌─────────┐  SMTP   ┌─────────┐       ┌────────┐ IMAP/ ┌────────┐
│  MUA   │──────>│ MSA/MTA │────────>│   MTA   │──MDA─>│ Buzón  │<──────│  MUA   │
│(Thund.)│ :587  │ origen  │  :25    │ destino │       │        │ POP3  │        │
└────────┘       └─────────┘         └─────────┘       └────────┘       └────────┘
            ENVIAR (SMTP)                                         LEER (IMAP/POP3)
```

Nota que **SMTP solo cubre la parte de "llevar"**: del cliente al servidor de origen, y de
servidor a servidor. Leer el buzón es trabajo de otro protocolo.

## SMTP: enviar y transportar

**SMTP** (Simple Mail Transfer Protocol) está definido actualmente en el RFC 5321
(Klensin, 2008). Es un protocolo de texto, con comandos como `HELO`/`EHLO`, `MAIL FROM`,
`RCPT TO` y `DATA`, y respuestas con códigos de tres dígitos — muy parecido en estilo al
canal de control de FTP que viste en la sesión anterior.

SMTP usa distintos puertos según **quién** habla con **quién**:

| Puerto | Uso | Fuente |
|---|---|---|
| **25** | Servidor a servidor (relay entre MTAs) | Klensin, 2008 |
| **587** | Cliente a servidor: **submission**, el usuario envía su correo, con autenticación | Gellens & Klensin, 2011 |
| **465** | Submission con TLS desde el primer byte ("TLS implícito") | Moore & Newman, 2018 |

La separación entre el 25 y el 587 es importante: el puerto 25 es para que los servidores
se pasen correo entre sí; el 587 es la "ventanilla" donde un usuario autenticado deja su
correo para enviarlo (Gellens & Klensin, 2011). Muchos proveedores de Internet bloquean el
puerto 25 de salida en las redes domésticas, justamente para que una computadora infectada
no se ponga a mandar spam directamente a otros servidores.

## Cómo sabe SMTP a dónde enviar: registros MX

Cuando escribes a `ana@ejemplo.edu.mx`, tu servidor necesita saber **qué máquina** recibe
el correo de `ejemplo.edu.mx`. Para eso consulta al **DNS** — lo de la sesión 3 — por un
registro de tipo **MX** (Mail eXchanger) de ese dominio (Klensin, 2008):

```
ejemplo.edu.mx.   MX   10  correo1.ejemplo.edu.mx.
ejemplo.edu.mx.   MX   20  correo2.ejemplo.edu.mx.
```

El número es la **preferencia**: se intenta primero el valor más bajo (10), y si no
responde, el siguiente (20). Así un dominio puede tener un servidor de respaldo para no
perder correo si el principal se cae.

## Leer el correo: POP3 vs. IMAP

Una vez que el mensaje está en el buzón del servidor, el MUA del destinatario lo recupera
con uno de dos protocolos:

- **POP3** (Post Office Protocol versión 3, RFC 1939): el cliente se conecta, **descarga**
  los mensajes y normalmente los **borra del servidor** (Myers & Rose, 1996). El correo
  queda viviendo en un solo dispositivo. Puertos: **110**, y **995** con TLS
  (Moore & Newman, 2018).
- **IMAP** (Internet Message Access Protocol, versión actual IMAP4rev2, RFC 9051): los
  mensajes **se quedan en el servidor**, y el cliente trabaja sobre ellos a distancia —
  carpetas, marcas de leído, búsquedas — de forma que todos tus dispositivos ven lo mismo
  (Melnikov & Leiba, 2021). Puertos: **143**, y **993** con TLS (Moore & Newman, 2018).

| | POP3 | IMAP |
|---|---|---|
| Dónde vive el correo | En el dispositivo que lo descargó | En el servidor |
| Varios dispositivos | Cada uno ve cosas distintas | Todos ven lo mismo, sincronizado |
| Si pierdes la laptop | Pierdes el correo descargado | El correo sigue en el servidor |
| Espacio en el servidor | Poco (se vacía al descargar) | Mucho (todo se queda) |
| Puertos | 110 / 995 (TLS) | 143 / 993 (TLS) |

Hoy, con el correo revisado desde celular, laptop y web al mismo tiempo, **IMAP es la
opción normal**; POP3 sobrevive en casos de un solo equipo o de buzones con poco espacio.

Sobre los puertos con TLS: el RFC 8314 se titula literalmente *"Cleartext considered
obsolete"* (el texto claro se considera obsoleto) y recomienda que los clientes usen
siempre conexiones cifradas — 465, 993 y 995 — para enviar y leer correo
(Moore & Newman, 2018). Es la misma lección de FTP: un protocolo que manda la contraseña
en texto claro ya no es aceptable.

## SPF, DKIM y DMARC: ¿de verdad viene de quien dice?

SMTP original no verifica el remitente: cualquier servidor puede escribir
`MAIL FROM: rector@ejemplo.edu.mx`. Para combatir la suplantación (phishing) existen tres
mecanismos que se publican en el **DNS** del dominio:

- **SPF** (RFC 7208): una lista de qué servidores tienen permiso de enviar correo a nombre
  del dominio (Kitterman, 2014).
- **DKIM** (RFC 6376): el servidor de origen **firma** digitalmente cada mensaje; el
  receptor verifica la firma con una llave pública publicada en DNS
  (Crocker et al., 2011).
- **DMARC** (RFC 7489): le dice al receptor **qué hacer** si SPF o DKIM fallan (aceptar,
  mandar a spam o rechazar) y a dónde enviar reportes (Kucherawy & Zwicky, 2015).

No necesitas configurarlos todavía; basta con saber que existen y que por eso un correo de
un servidor mal configurado termina en spam aunque "sí se haya enviado".

## Ejemplo: "mis correos llegan, pero nadie los encuentra"

Un despacho pequeño monta su propio servidor de correo para el dominio de la empresa. Al
principio todo parece funcionar: los empleados envían y reciben. A las pocas semanas
llegan dos quejas distintas.

La primera viene de la directora: revisa el correo en la laptop de la oficina y, cuando
sale de viaje, abre el correo en su celular y la bandeja está vacía. Los mensajes viejos
solo existen en la laptop. Cuando le configuraron la cuenta, alguien eligió POP3 "porque
era lo que salía por default", y la laptop descarga y borra cada mensaje del servidor.

La segunda queja es de los clientes del despacho: los correos que les mandan llegan
directo a la carpeta de spam, o a veces ni llegan. Al revisar, resulta que el dominio no
tiene registro SPF ni firma DKIM, así que los servidores receptores no tienen forma de
confirmar que el correo realmente salió del despacho. Además, uno de los empleados
configuró su cliente para enviar por el puerto 25 sin cifrado y sin autenticación, y el
servidor lo acepta — lo que significa que, desde la red del despacho, cualquiera podría
mandar correo a nombre de cualquier usuario.

Ninguno de los dos problemas es un "error" que el servidor reporte: el correo técnicamente
se envía y se recibe. Son decisiones de protocolo y de configuración mal tomadas.

## Cómo se resuelve con herramientas reales

En Linux, un servidor de correo completo se arma con dos piezas de código abierto que la
documentación oficial de Ubuntu Server usa como referencia (Canonical, s.f.-a, s.f.-b), y
que puedes instalar en tu **VM de Ubuntu Server en VirtualBox** cuando el curso llegue a
servicios:

- **Postfix** (MTA): habla SMTP. Recibe el correo de otros servidores en el puerto 25 y
  el de los usuarios en el 587 exigiendo autenticación y TLS — eso cierra el hueco del
  empleado que enviaba sin contraseña por el 25 (Venema, s.f.).
- **Dovecot** (servidor IMAP/POP3): guarda los buzones y los sirve por **IMAP en el 993**
  con TLS (Dovecot, s.f.). Ofrecer solo IMAP, y no POP3, resuelve el problema de la
  directora: el correo se queda en el servidor y todos sus dispositivos ven lo mismo.

```
                 ┌──────────────── Ubuntu Server ────────────────┐
Otros servidores │  Postfix (MTA)            Dovecot             │
 ── SMTP :25 ──> │  recibe y envía  ──────>  buzones  ── IMAP ───┼──> Thunderbird,
Usuarios         │                                      :993     │    celular
 ── SMTP :587 ─> │  (auth + TLS)                                 │
                 └───────────────────────────────────────────────┘
```

Para la parte de spam, no se instala nada nuevo en el servidor: se publican registros
**SPF**, **DKIM** y **DMARC** en el DNS del dominio, y se puede comprobar desde tu VM qué
tiene publicado cualquier dominio con `dig`, por ejemplo `dig MX ejemplo.edu.mx` o
`dig TXT ejemplo.edu.mx` — la misma herramienta de la sesión de DNS.

---

## Referencias

Canonical. (s.f.-a). *Install and configure Dovecot*. Ubuntu Server documentation.
https://documentation.ubuntu.com/server/how-to/mail-services/install-dovecot/

Canonical. (s.f.-b). *Install and configure Postfix*. Ubuntu Server documentation.
https://documentation.ubuntu.com/server/how-to/mail-services/install-postfix/

Crocker, D. (2009). *Internet mail architecture* (RFC 5598). IETF.
https://www.rfc-editor.org/rfc/rfc5598

Crocker, D. (Ed.), Hansen, T. (Ed.), & Kucherawy, M. (Ed.). (2011). *DomainKeys
Identified Mail (DKIM) signatures* (RFC 6376). IETF.
https://www.rfc-editor.org/rfc/rfc6376

Dovecot. (s.f.). *Dovecot documentation*. https://doc.dovecot.org/

Gellens, R., & Klensin, J. (2011). *Message submission for mail* (RFC 6409). IETF.
https://www.rfc-editor.org/rfc/rfc6409

Kitterman, S. (2014). *Sender Policy Framework (SPF) for authorizing use of domains in
email, version 1* (RFC 7208). IETF. https://www.rfc-editor.org/rfc/rfc7208

Klensin, J. (2008). *Simple mail transfer protocol* (RFC 5321). IETF.
https://www.rfc-editor.org/rfc/rfc5321

Kucherawy, M. (Ed.), & Zwicky, E. (Ed.). (2015). *Domain-based Message Authentication,
Reporting, and Conformance (DMARC)* (RFC 7489). RFC Editor (Independent Submission).
https://www.rfc-editor.org/rfc/rfc7489

Melnikov, A. (Ed.), & Leiba, B. (Ed.). (2021). *Internet Message Access Protocol (IMAP) —
Version 4rev2* (RFC 9051). IETF. https://www.rfc-editor.org/rfc/rfc9051

Moore, K., & Newman, C. (2018). *Cleartext considered obsolete: Use of Transport Layer
Security (TLS) for email submission and access* (RFC 8314). IETF.
https://www.rfc-editor.org/rfc/rfc8314

Myers, J., & Rose, M. (1996). *Post office protocol — Version 3* (RFC 1939). IETF.
https://www.rfc-editor.org/rfc/rfc1939

Venema, W. (s.f.). *The Postfix home page*. https://www.postfix.org/
