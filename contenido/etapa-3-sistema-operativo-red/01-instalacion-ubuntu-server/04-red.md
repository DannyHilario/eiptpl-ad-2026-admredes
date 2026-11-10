# Paso 4: Red

> Paso 4 de 13 · [← Tipo de instalación](03-tipo-de-instalacion.md) · Siguiente: [Proxy](05-proxy.md)

![Pantalla de configuración de red](../../../assets/instalacion-ubuntu-server/04-red.png)

El instalador muestra las tarjetas de red que encontró y la dirección que obtuvo cada
una. **No cambies nada**: baja con Tab hasta **[ Hecho ]**. Pero vale la pena leerla,
porque reúne en una sola pantalla casi todo lo que viste en Redes y Telecomunicaciones.

| Lo que aparece | Qué es |
|---|---|
| `enp0s8` | Nombre de la tarjeta en Linux. `en` = Ethernet, `p0s8` = su ubicación en el bus PCI: bus 0, ranura 8 (systemd, s.f.). En Windows sería "Ethernet" |
| `eth` | Tipo de interfaz: Ethernet (cable) |
| `DHCPv4 10.0.2.15/24` | IPv4 asignada **automáticamente por DHCP**. `/24` es la máscara 255.255.255.0 |
| `DHCPv6 fd17:...` | Dirección IPv6, también automática |
| `08:00:27:...` | La **dirección MAC**. `08:00:27` es el prefijo tradicional que VirtualBox pone a sus tarjetas virtuales; los otros tres pares los genera al azar (Oracle, s.f.) |
| `Intel 82540EM...` | Modelo de tarjeta que VirtualBox simula |
| `[ Create bond ]` | Unir varias tarjetas en una sola (redundancia). No lo necesitamos |

> **Nota:** el nombre de la tarjeta puede variar según tu equipo (`enp0s3`, `enp0s8`...).
> La captura se tomó en una Mac con procesador ARM; en una computadora con Intel o AMD es
> común ver `enp0s3`. Lo importante es que empiece con `en` (Ethernet) y que tenga una
> dirección IPv4 asignada por DHCP.

## Por qué la IP es 10.0.2.15

Esa dirección es la que VirtualBox entrega en modo **NAT**: la VM queda "detrás" de tu
computadora, como una casa dentro del fraccionamiento que sale a Internet por la caseta,
pero a la que nadie de afuera puede llegar directamente.

```
Internet ── Router (casa/lab) ── Tu computadora ── NAT de VirtualBox ── VM
                                  (192.168.x.x)                      (10.0.2.15)
```

Para instalar es justo lo que queremos: la VM tiene Internet para descargar
actualizaciones. Cuando quieras conectarte a este servidor desde tu computadora (por
SSH, en [Acceso remoto](../03-acceso-remoto/03-conectarte-por-ssh.md)), se apaga la VM y
se cambia el adaptador a **puente**, sin reinstalar: con puente, la VM pide su propia
IP al DHCP de la red real.

El servicio que entregó esa IP es exactamente el que estudiaste en
[DHCP](../../etapa-2-servicios-red/02-dhcp.md) — aquí el servidor DHCP es el propio
VirtualBox.

## Referencias

Oracle. (s.f.). *HostImpl.cpp* [Código fuente de VirtualBox, función
`i_generateMACAddress`]. GitHub.
https://github.com/VirtualBox/virtualbox/blob/main/src/VBox/Main/src-server/HostImpl.cpp

systemd. (s.f.). *systemd.net-naming-scheme(7): Network device naming schemes* [Manual
page]. https://man7.org/linux/man-pages/man7/systemd.net-naming-scheme.7.html
