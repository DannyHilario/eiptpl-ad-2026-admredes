# HTTPS

> 50 min · [← WWW y HTTP](04-www.md) · Siguiente: [FTP](06-ftp.md)

## El problema de HTTP: todo viaja en texto legible

En la sesión anterior viste que una petición HTTP/1.1 es texto plano: método, ruta,
cabeceras y contenido viajan tal cual. Eso incluye lo que escribes en un formulario —
usuario, contraseña, datos de una tarjeta. Cualquier equipo por el que pase ese tráfico
(un punto de acceso Wi-Fi público, un router intermedio) puede leerlo e incluso
modificarlo.

Con la analogía del fraccionamiento: enviar algo por HTTP es como mandar un mensaje
escrito en una hoja suelta que pasa de mano en mano por las calles internas, la caseta y
las carreteras de afuera. Cualquiera en el camino puede leerla, cambiarla o hacerse pasar
por el destinatario. HTTPS resuelve justamente eso.

## Qué es HTTPS

**HTTPS** no es un protocolo distinto de HTTP: es **el mismo HTTP, viajando dentro de una
conexión cifrada con TLS** (Transport Layer Security). Los métodos, los códigos de estado
y las cabeceras son idénticos; lo que cambia es que antes de enviar la primera petición,
cliente y servidor establecen un canal seguro. El esquema `https` usa por default el
puerto TCP **443**, en lugar del 80 de `http` (Fielding et al., 2022).

```
        HTTP                               HTTPS
 ┌──────────────────┐             ┌──────────────────┐
 │ HTTP (texto)     │             │ HTTP (igual)     │
 ├──────────────────┤             ├──────────────────┤
 │                  │             │ TLS (cifrado)    │  <-- capa extra
 │ TCP, puerto 80   │             ├──────────────────┤
 │                  │             │ TCP, puerto 443  │
 └──────────────────┘             └──────────────────┘
```

TLS aporta tres garantías distintas, y conviene no confundirlas (Rescorla, 2026):

| Garantía | Qué evita | En el ejemplo de la hoja suelta |
|---|---|---|
| **Confidencialidad** | Que un tercero lea el contenido | Nadie en el camino puede leerla |
| **Integridad** | Que un tercero lo modifique sin que se note | Si alguien la altera, se detecta |
| **Autenticación** | Que alguien se haga pasar por el servidor | Sabes que la respuesta viene de quien dice ser |

## SSL y TLS: aclarando nombres

El manual oficial habla de "certificados SSL TLS" como si fueran lo mismo. La historia
real: **SSL** fue el protocolo original de los años noventa, y **TLS** es su sucesor
estandarizado por el IETF. Hoy todas las versiones de SSL están prohibidas por inseguras
— SSL 2.0 desde 2011 y SSL 3.0 desde 2015 (Turner & Polk, 2011; Barnes et al., 2015) —
y también TLS 1.0 y 1.1 quedaron descontinuados en 2021 (Moriarty & Farrell, 2021). Lo
que se usa hoy es **TLS 1.2** y, sobre todo, **TLS 1.3**. El nombre "certificado SSL"
sobrevive solo por costumbre comercial.

| Versión | Estado actual |
|---|---|
| SSL 2.0 / 3.0 | Prohibidas (inseguras) |
| TLS 1.0 / 1.1 | Descontinuadas desde 2021 |
| TLS 1.2 | Todavía aceptada |
| TLS 1.3 | Versión vigente recomendada |

TLS 1.3 se publicó originalmente como RFC 8446 en 2018 (Rescorla, 2018). En 2026 el IETF
publicó RFC 9846, una actualización menor que reemplaza al RFC 8446 pero conserva el mismo
número de versión, TLS 1.3, y es compatible con él (Rescorla, 2026).

## El saludo TLS (handshake), simplificado

Antes de enviar cualquier petición HTTP, cliente y servidor hacen un **handshake**. En TLS
1.3 el handshake completo toma un solo viaje de ida y vuelta (1-RTT), más rápido que en
versiones anteriores (Rescorla, 2018):

```
 Navegador                                              Servidor (puerto 443)
     │  1. ClientHello: "hablo TLS 1.3, estos cifrados,      │
     │     esta es mi parte de la clave"  ─────────────────> │
     │                                                        │
     │  <───────────── 2. ServerHello + su parte de la clave  │
     │                    + CERTIFICADO + firma               │
     │                                                        │
     │  3. El navegador verifica el certificado               │
     │     (¿es del dominio correcto? ¿quién lo firmó?        │
     │      ¿sigue vigente?)                                  │
     │                                                        │
     │  4. Ya con clave compartida: GET / (cifrado) ────────> │
     │  <────────────────────────── 200 OK (cifrado) ──────── │
```

El paso 3 es el que hace posible la **autenticación**, y depende de los certificados.

## Certificados y autoridades certificadoras

Un **certificado digital** es un documento electrónico, en formato estándar **X.509**,
que asocia un nombre de dominio con una clave pública e incluye, entre otros datos,
quién lo emitió y entre qué fechas es válido (Cooper et al., 2008).

Cualquiera puede generar un certificado que diga "soy `banco.com`". Lo que lo vuelve
confiable es que esté **firmado por una autoridad certificadora (CA)** — una organización
que verificó que quien pide el certificado de verdad controla ese dominio. Tu sistema
operativo y tu navegador traen preinstalada una lista de CA raíz en las que confían; si un
certificado se puede rastrear hasta una de ellas, el candado aparece sin advertencias.

```
 ┌─────────────────────┐
 │  CA raíz            │   <-- preinstalada en el navegador / SO
 └──────────┬──────────┘
            │ firma a
 ┌──────────v──────────┐
 │  CA intermedia      │
 └──────────┬──────────┘
            │ firma a
 ┌──────────v──────────┐
 │  www.ejemplo.com    │   <-- certificado que entrega el servidor
 └─────────────────────┘
        "cadena de confianza"
```

Si la cadena no llega a una CA conocida (por ejemplo, un **certificado autofirmado**), si
el certificado ya **expiró**, o si fue emitido para otro dominio, el navegador muestra una
advertencia de seguridad en lugar de la página.

## Let's Encrypt y ACME: certificados gratuitos y automáticos

Durante años, obtener un certificado costaba dinero y requería trámites manuales, lo que
frenó la adopción de HTTPS. **Let's Encrypt** es una CA sin fines de lucro que emite
certificados gratuitos; sus certificados por default duran **90 días**, y recomienda
renovarlos cada 60 (Let's Encrypt, s.f.-b). Una vida tan corta solo es práctica porque la
renovación es automática, gracias al protocolo **ACME** (Automatic Certificate Management
Environment), estandarizado en RFC 8555 (Barnes et al., 2019).

Con ACME, un programa en el servidor demuestra automáticamente que controla el dominio —
por ejemplo, publicando un archivo temporal que la CA pide por HTTP — y recibe el
certificado sin intervención humana (Barnes et al., 2019). Conecta con la administración
de configuración de la Etapa 1: un certificado es configuración que caduca, y lo que
caduca debe automatizarse.

## Ejemplo: el lunes que la plataforma "dejó de ser segura"

La preparatoria tiene una plataforma de calificaciones en
`https://calificaciones.prepa.mx`. Funcionó sin problemas durante meses. Un lunes, todos
los que intentan entrar ven una pantalla roja del navegador: "La conexión no es privada".
Algunos alumnos le dan clic a "Continuar de todos modos"; otros escriben al grupo que
"hackearon la plataforma".

Nadie cambió nada en el servidor la semana anterior, y el servidor sigue respondiendo en
el puerto 443. Lo que sí pasó: el certificado se había instalado a mano hace tres meses,
y el sábado llegó a su fecha de expiración. El navegador hace exactamente lo que debe —
en el paso 3 del handshake detecta un certificado vencido y ya no puede garantizar la
autenticación.

El problema tiene una segunda capa, más grave que la caída en sí: acostumbrar a los
usuarios a ignorar la advertencia. Si mañana alguien montara un servidor falso con un
certificado inválido, esos mismos alumnos ya aprendieron a darle "Continuar". Un fallo
de configuración terminó debilitando la seguridad.

## Cómo se resuelve con herramientas reales

En tu **VM de Ubuntu Server en VirtualBox**, las dos herramientas que atacan este ejemplo
son:

- **`openssl s_client`**: se conecta a un servidor TLS como lo haría un navegador y
  muestra el certificado que recibe, la cadena de CA y la versión de TLS negociada
  (OpenSSL Project, s.f.). Permite ver la fecha de expiración sin depender de lo que
  diga el navegador.
- **certbot**: el cliente ACME de la EFF, recomendado por Let's Encrypt. Obtiene el
  certificado, puede configurarlo automáticamente en nginx o Apache, e instala una tarea
  programada que lo renueva antes de que expire (Electronic Frontier Foundation, s.f.).

```
$ openssl s_client -connect calificaciones.prepa.mx:443 </dev/null 2>/dev/null \
    | openssl x509 -noout -issuer -enddate
issuer=C=US, O=Let's Encrypt, CN=...
notAfter=Oct  4 23:59:59 2026 GMT       <-- ya pasó: certificado vencido

$ sudo certbot --nginx -d calificaciones.prepa.mx    # obtiene e instala
$ sudo certbot renew --dry-run                       # simula la renovación
```

Una precisión importante para tu VM: Let's Encrypt solo emite certificados para dominios
públicos que pueda validar desde Internet, así que no funciona con un nombre como
`servidor.local` dentro de VirtualBox (Let's Encrypt, s.f.-a). En un laboratorio local se
usa un certificado autofirmado generado con `openssl` — el navegador mostrará la
advertencia de la cadena de confianza, y ahora sabes exactamente por qué.

---

## Referencias

Barnes, R., Hoffman-Andrews, J., McCarney, D., & Kasten, J. (2019). *Automatic
Certificate Management Environment (ACME)* (RFC 8555). IETF.
https://www.rfc-editor.org/rfc/rfc8555

Barnes, R., Thomson, M., Pironti, A., & Langley, A. (2015). *Deprecating Secure Sockets
Layer version 3.0* (RFC 7568). IETF. https://www.rfc-editor.org/rfc/rfc7568

Cooper, D., Santesson, S., Farrell, S., Boeyen, S., Housley, R., & Polk, W. (2008).
*Internet X.509 public key infrastructure certificate and certificate revocation list
(CRL) profile* (RFC 5280). IETF. https://www.rfc-editor.org/rfc/rfc5280

Electronic Frontier Foundation. (s.f.). *Certbot documentation*.
https://eff-certbot.readthedocs.io/

Fielding, R., Nottingham, M., & Reschke, J. (Eds.). (2022). *HTTP semantics* (RFC 9110).
IETF. https://www.rfc-editor.org/rfc/rfc9110

Let's Encrypt. (s.f.-a). *Certificates for localhost*.
https://letsencrypt.org/docs/certificates-for-localhost/

Let's Encrypt. (s.f.-b). *FAQ*. https://letsencrypt.org/docs/faq/

Moriarty, K., & Farrell, S. (2021). *Deprecating TLS 1.0 and TLS 1.1* (RFC 8996). IETF.
https://www.rfc-editor.org/rfc/rfc8996

OpenSSL Project. (s.f.). *openssl-s_client*.
https://docs.openssl.org/master/man1/openssl-s_client/

Rescorla, E. (2018). *The Transport Layer Security (TLS) protocol version 1.3* (RFC
8446). IETF. https://www.rfc-editor.org/rfc/rfc8446

Rescorla, E. (2026). *The Transport Layer Security (TLS) protocol version 1.3* (RFC
9846). IETF. https://www.rfc-editor.org/rfc/rfc9846

Turner, S., & Polk, T. (2011). *Prohibiting Secure Sockets Layer (SSL) version 2.0* (RFC
6176). IETF. https://www.rfc-editor.org/rfc/rfc6176
