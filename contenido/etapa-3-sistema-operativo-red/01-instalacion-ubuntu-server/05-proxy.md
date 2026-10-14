# Paso 5: Proxy

> Paso 5 de 13 · [← Red](04-red.md) · Siguiente: [Mirror](06-mirror.md)

El instalador pide la dirección de un **proxy**: un equipo intermediario por el que
algunas empresas o escuelas obligan a pasar el tráfico web hacia Internet, para
filtrarlo o registrarlo.

**Qué hacer:** dejar el campo **vacío** y elegir **[ Done ]**.

Ni una red doméstica ni el router del laboratorio usan proxy. Solo se llena si el
administrador de la red te dio una dirección con formato `http://servidor:puerto`. Si
pusieras una dirección inventada, el instalador no podría descargar nada en el paso
siguiente.

La documentación oficial de Ubuntu Server indica justamente eso: no configurar proxy ni
réplica salvo que la red lo exija (Canonical, s.f.).

## Referencias

Canonical Ltd. (s.f.). *Basic installation* [Ubuntu Server documentation].
https://ubuntu.com/server/docs/tutorial/basic-installation/
