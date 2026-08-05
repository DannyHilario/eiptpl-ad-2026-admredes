# Interfaces de usuario: gráfica y línea de comandos

> 50 min · [← Software y el Sistema Operativo](02-software-so.md) · Siguiente: [Cliente-Servidor](04-cliente-servidor.md)

## ¿Qué es una interfaz de usuario?

Ya sabes que el sistema operativo administra el hardware (Sesión 2). Pero, ¿cómo le dices
tú, humano, qué quieres que haga? Eso es una **interfaz de usuario**: el medio por el que
das instrucciones al SO y este te muestra resultados. Hay dos grandes paradigmas, y **el
mismo sistema operativo puede ofrecer ambos al mismo tiempo** — Windows, macOS y Linux
todos tienen las dos.

| | Interfaz gráfica (GUI) | Línea de comandos (CLI) |
|---|---|---|
| **Cómo se usa** | Clics, íconos, ventanas, menús | Escribes comandos de texto y presionas Enter |
| **Nombre común** | GUI (Graphical User Interface) | CLI (Command Line Interface), terminal, consola |
| **Curva de aprendizaje** | Baja — intuitiva, se explora con el mouse | Alta al inicio — hay que memorizar/consultar comandos |
| **Velocidad en tareas repetitivas** | Lenta — repetir los mismos clics una y otra vez | Rápida — un comando, o un script, hace lo mismo cientos de veces |
| **Automatización** | Difícil o imposible sin herramientas extra | Natural — los comandos se guardan en archivos (scripts) y se repiten |
| **Uso de recursos (RAM/CPU)** | Mayor — dibujar ventanas, íconos, animaciones cuesta recursos | Menor — solo texto, sin gráficos que procesar |
| **Administración remota** | Pesada — requiere transmitir toda la imagen de la pantalla | Ligera — solo se transmite texto, ideal para conexiones lentas o remotas |

## Ambas interfaces hablan con el mismo sistema operativo

Esto es clave para no confundirlas con "dos sistemas distintos": cuando borras un archivo
haciendo clic derecho → Eliminar en el Explorador de archivos, y cuando escribes `del
archivo.txt` en una terminal, **ambas acciones terminan pidiéndole exactamente lo mismo al
sistema operativo** — solo cambia cómo se lo pediste. La GUI, por dentro, también ejecuta
comandos del sistema — simplemente los oculta detrás de un botón (Silberschatz et al.,
2018).

```
      Tú
       │
   ┌───┴────┐
   │        │
   v        v
  GUI      CLI          (dos formas de pedir lo mismo)
   │        │
   └───┬────┘
       v
Sistema Operativo        (quien realmente ejecuta la acción)
       │
       v
   Hardware
```

## Por qué la línea de comandos importa tanto en este curso

En una PC personal, la GUI suele ser más cómoda para el uso diario. Pero en
**administración de redes y servidores** (el tema de este curso) la CLI es indispensable
por razones muy concretas:

- **Los servidores casi nunca tienen pantalla ni mouse conectados** — se administran a
  distancia, y mandar solo texto por la red es mucho más ligero que mandar imágenes de
  pantalla completas.
- **Automatización:** configurar 50 servidores a mano con clics tomaría días; un script
  con los comandos correctos lo hace en minutos, igual en los 50.
- **Precisión:** un comando hace exactamente lo que dice, sin ambigüedad de "¿le di clic
  al botón correcto?".

Por eso, más adelante en el curso vas a trabajar principalmente desde una terminal Linux
— no porque la GUI esté mal, sino porque es la herramienta correcta para este tipo de
trabajo.

## Comandos equivalentes en Windows, macOS y Linux

Estas son las mismas tareas, en las tres terminales que existen. Nota dos terminales
distintas dentro de Windows: **CMD** (la clásica, más antigua) y **PowerShell** (la
moderna, instalada por defecto desde Windows 7 en adelante).

| Tarea | CMD (Windows) | PowerShell (Windows) | macOS / Linux (Bash/Zsh) |
|---|---|---|---|
| Ver carpeta actual | `cd` (sin argumento) | `pwd` | `pwd` |
| Listar archivos | `dir` | `dir` / `Get-ChildItem` | `ls` |
| Cambiar de carpeta | `cd carpeta` | `cd carpeta` | `cd carpeta` |
| Crear carpeta | `mkdir carpeta` | `mkdir carpeta` | `mkdir carpeta` |
| Crear archivo vacío | `type nul > a.txt` | `New-Item a.txt` | `touch a.txt` |
| Copiar archivo | `copy origen destino` | `Copy-Item origen destino` | `cp origen destino` |
| Mover / renombrar | `move origen destino` | `Move-Item origen destino` | `mv origen destino` |
| Borrar archivo | `del archivo` | `Remove-Item archivo` | `rm archivo` |
| Borrar carpeta completa | `rmdir /s carpeta` | `Remove-Item -Recurse carpeta` | `rm -r carpeta` |
| Ver contenido de un archivo | `type archivo.txt` | `Get-Content archivo.txt` | `cat archivo.txt` |
| Ver procesos corriendo | `tasklist` | `Get-Process` | `ps aux` |
| Ver configuración de red | `ipconfig` | `Get-NetIPAddress` | `ifconfig` / `ip addr` |
| Limpiar la pantalla | `cls` | `cls` | `clear` |
| Ver ayuda de un comando | `comando /?` | `Get-Help comando` | `man comando` |

### Un dato curioso y útil: PowerShell "habla" ambos idiomas

PowerShell define **alias** — nombres alternativos — para muchos de sus comandos
(llamados *cmdlets*), incluyendo los nombres clásicos de CMD y también los de Unix/Linux.
Por eso en PowerShell `ls`, `dir` y `Get-ChildItem` hacen exactamente lo mismo — los tres
apuntan al mismo cmdlet por dentro (Microsoft, s.f.). Esto no es casualidad: facilita que
alguien que ya sabe comandos de Linux se sienta cómodo en Windows, y viceversa.

---

## Ejercicio práctico: la misma acción, con y sin clics (Windows 11)

**Parte A — Crea una carpeta de dos formas:**
1. En el Escritorio, clic derecho → **Nuevo** → **Carpeta** → nómbrala `prueba-gui`.
2. Abre una terminal: busca "cmd" en el menú de inicio y ábrelo. Escribe:
   ```
   cd Desktop
   mkdir prueba-cli
   dir
   ```
3. Confirma en el Explorador de archivos que **ambas** carpetas existen — una la creaste
   con clics, la otra con un comando, pero el resultado en el sistema de archivos es
   idéntico.

**Parte B — Abre PowerShell y compara:**
1. Busca "PowerShell" en el menú de inicio y ábrelo.
2. Ejecuta `ls` (verás que sí funciona, aunque "ls" no es un comando de Windows/CMD).
3. Ejecuta `Get-ChildItem` — compara que la salida es exactamente la misma que con `ls`.

**Parte C — Limpieza:**
Borra ambas carpetas de prueba, una desde el Explorador (clic derecho → Eliminar) y otra
con `Remove-Item prueba-cli` desde PowerShell.

---

## Referencias

Silberschatz, A., Galvin, P. B., & Gagne, G. (2018). *Operating system concepts* (10a
ed.). Wiley.

Microsoft. (s.f.). *about_Aliases*. PowerShell documentation.
https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_aliases
