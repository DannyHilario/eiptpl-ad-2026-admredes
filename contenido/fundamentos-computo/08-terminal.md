# Fundamentos de terminal

> 50 min · [← Máquinas Virtuales y Linux](07-vm-y-linux.md) · Siguiente: [Administración de configuración](../etapa-1-funciones-administracion/01-administracion-configuracion.md)

## Por qué existe esta sesión

En la Sesión 7 instalaste Ubuntu dentro de tu VM. Todas las prácticas del resto del curso
(Docker, redes, servidores) se hacen desde una **terminal** de esa VM — línea de comandos,
no clics en ventanas. Esta sesión es el prerrequisito real antes de Etapa 1: moverte entre
carpetas, ver y editar archivos, y entender qué es un proceso.

**Meta de esta sesión:** que sin importar si tu laptop real es Windows o Mac, termines
usando **la misma terminal Linux dentro de tu VM** que el resto de tus compañeros — así
todos los comandos del curso funcionan igual para todos.

---

## Parte 1 — Preparar tu equipo (≈10 min)

### Si tienes Windows

Windows no trae una terminal Linux por defecto, pero Microsoft incluye una manera oficial
de tener una real y completa dentro de Windows: **WSL2** (Windows Subsystem for Linux,
version 2). Es el mismo WSL2 que necesitarás más adelante para que Docker Desktop
funcione, así que este paso no es un rodeo — es un prerequisito real del curso.

**Paso 1. Verifica tu versión de Windows**

WSL2 requiere Windows 10 versión 2004 (build 19041) o superior, o Windows 11 (cualquier
build ya lo soporta). Para revisar tu versión:

1. Presiona `Windows + R`, escribe `winver` y presiona Enter.
2. Se abrirá una ventana mostrando la versión. Si dice Windows 11, estás listo. Si dice
   Windows 10, confirma que el número de build sea 19041 o mayor.

**Paso 2. Instala WSL2**

1. Abre el **Símbolo del sistema (CMD)** o **PowerShell** como Administrador:
   - Presiona `Windows`, escribe `PowerShell`, clic derecho → "Ejecutar como administrador".
2. Ejecuta:
   ```powershell
   wsl --install
   ```
3. Este comando hace TODO automáticamente: activa las características de Windows
   necesarias, instala WSL2 como versión por defecto, y descarga **Ubuntu** (la
   distribución de Linux que usaremos, es la más común y la que tiene más ayuda
   disponible en internet).
4. **Reinicia tu computadora** cuando el instalador te lo pida.

**Paso 3. Primer arranque de Ubuntu**

1. Después de reiniciar, Ubuntu debería abrirse solo y terminar su instalación (tarda 1-2
   minutos la primera vez). Si no se abre solo, búscalo en el menú de inicio como
   "Ubuntu".
2. Te pedirá crear un **usuario y contraseña** — esto es independiente de tu cuenta de
   Windows, es tu usuario dentro de Linux. Anótalo, lo vas a usar seguido.
   ```
   Enter new UNIX username: alumno
   New password: (escribe una contraseña, no se ve mientras escribes, es normal)
   Retype new password:
   ```
3. Cuando termine, verás un prompt así, lo que significa que ya estás dentro de Linux:
   ```
   alumno@DESKTOP-XXXX:~$
   ```

**Paso 4. Verifica que todo quedó bien**

Desde PowerShell (no desde Ubuntu):
```powershell
wsl --list --verbose
```
Deberías ver algo como:
```
  NAME      STATE           VERSION
* Ubuntu    Running         2
```
Lo importante es que diga **VERSION 2** (no 1) — si dice 1, avísale a tu docente para
migrarlo con `wsl --set-version Ubuntu 2`.

**Paso 5. Cómo abrir tu terminal de aquí en adelante**

- Busca "Ubuntu" en el menú de inicio de Windows y ábrelo, o
- Instala la app **Windows Terminal** (gratis, en Microsoft Store) que es más cómoda y
  te deja elegir "Ubuntu" en una pestaña.

A partir de aquí, **toda práctica del curso la haces dentro de esta ventana de Ubuntu**,
no en CMD ni en PowerShell.

> Nota: si en la Sesión 7 ya instalaste Ubuntu dentro de una VM con VirtualBox, WSL2 es
> una alternativa distinta para llegar al mismo lugar (una terminal Linux) sin usar
> VirtualBox. Tu docente te indicará cuál de las dos usar como entorno principal del curso.

---

### Si tienes Mac

Buenas noticias: macOS ya trae una terminal Unix real integrada, no hay que instalar
nada.

**Paso 1. Abre Terminal**

1. Presiona `Cmd + Espacio` (Spotlight) y escribe `Terminal`, presiona Enter.
2. También la encuentras en Finder → Aplicaciones → Utilidades → Terminal.

**Paso 2. Conócela**

Verás un prompt parecido a este:
```
tu-nombre@MacBook-de-Tu-Nombre ~ %
```

El símbolo al final (`%` o `$` dependiendo de tu versión de macOS) indica que está lista
para recibir comandos.

**Nota importante:** macOS usa la terminal de shell **zsh** por defecto (desde macOS
Catalina, 2019), y Ubuntu/WSL usa **bash**. Son muy similares — el 95% de los comandos
que veremos en el curso funcionan igual en ambas. Cuando haya una diferencia, se indicará
explícitamente.

---

## Parte 2 — Comandos básicos, iguales en Windows/WSL, Mac y tu VM de Ubuntu (≈30 min)

A partir de aquí, todo lo que seguimos es **idéntico** sin importar tu sistema operativo,
porque todos están corriendo un shell tipo Unix.

### 2.1 ¿Dónde estoy? — navegación

| Comando | Qué hace | Ejemplo |
|---|---|---|
| `pwd` | Muestra la carpeta (directorio) en la que estás | `pwd` → `/home/alumno` |
| `ls` | Lista el contenido de la carpeta actual | `ls` |
| `ls -la` | Lista todo, incluyendo archivos ocultos, con detalle | `ls -la` |
| `cd <carpeta>` | Entra a una carpeta | `cd Documentos` |
| `cd ..` | Sube un nivel (a la carpeta padre) | `cd ..` |
| `cd ~` | Va directo a tu carpeta de usuario (home) | `cd ~` |
| `cd /` | Va a la raíz del sistema de archivos | `cd /` |

**Ejercicio 1:** Abre tu terminal y ejecuta en orden `pwd`, `ls`, `cd /`, `ls`, `cd ~`,
`pwd`. Anota qué carpeta apareció en cada `pwd`.

### 2.2 Crear, mover, copiar y borrar

| Comando | Qué hace | Ejemplo |
|---|---|---|
| `mkdir <nombre>` | Crea una carpeta nueva | `mkdir practica1` |
| `touch <archivo>` | Crea un archivo vacío | `touch notas.txt` |
| `cp <origen> <destino>` | Copia un archivo | `cp notas.txt notas_backup.txt` |
| `mv <origen> <destino>` | Mueve o renombra un archivo/carpeta | `mv notas.txt practica1/` |
| `rm <archivo>` | Borra un archivo (¡no hay papelera de reciclaje!) | `rm notas_backup.txt` |
| `rm -r <carpeta>` | Borra una carpeta y todo su contenido | `rm -r practica1` |

⚠️ **Advertencia importante:** `rm` borra de forma **permanente**, no hay "deshacer" ni
papelera. Antes de correr `rm -r` en algo, verifica con `ls` que estás en la carpeta
correcta y que es lo que de verdad quieres borrar.

**Ejercicio 2:** Crea una carpeta `lab-redes`, entra a ella, crea un archivo `apuntes.txt`
dentro, y confírmalo con `ls`.

### 2.3 Ver y editar el contenido de un archivo

| Comando | Qué hace | Ejemplo |
|---|---|---|
| `cat <archivo>` | Muestra todo el contenido del archivo de un jalón | `cat apuntes.txt` |
| `less <archivo>` | Muestra el contenido "página por página" (sales con `q`) | `less apuntes.txt` |
| `nano <archivo>` | Abre un editor de texto simple dentro de la terminal | `nano apuntes.txt` |

**Usando `nano` (el editor que usaremos en el curso):**

1. `nano apuntes.txt` abre el editor.
2. Escribe lo que quieras directamente.
3. Para guardar: `Ctrl + O` (Write Out), presiona Enter para confirmar el nombre.
4. Para salir: `Ctrl + X`.

Los atajos siempre aparecen al pie de la pantalla de nano (el símbolo `^` significa
`Ctrl`).

**Ejercicio 3:** Abre `apuntes.txt` con `nano`, escribe tu nombre y el grupo, guarda y
sal. Luego confirma el contenido con `cat apuntes.txt`.

### 2.4 Permisos de archivos (lo mínimo que necesitas saber)

Cuando corres `ls -la` vas a ver algo así:

```
-rw-r--r--  1 alumno alumno   45 Jul 26 10:00 apuntes.txt
drwxr-xr-x  2 alumno alumno 4096 Jul 26 10:00 lab-redes
```

- La primera letra indica el tipo: `-` = archivo normal, `d` = carpeta (directorio).
- Las siguientes 9 letras son permisos en grupos de 3: **dueño / grupo / todos los demás**,
  cada grupo con `r` (leer), `w` (escribir), `x` (ejecutar).
- `chmod +x <archivo>` le da permiso de **ejecución** a un archivo — lo vas a necesitar
  para correr scripts `.sh` como los del laboratorio de Docker:
  ```bash
  chmod +x generate-cert.sh
  ./generate-cert.sh
  ```

### 2.5 Procesos (¿qué está corriendo?)

| Comando | Qué hace |
|---|---|
| `ps aux` | Lista todos los procesos corriendo en el sistema |
| `top` (o `htop` si está instalado) | Muestra procesos en tiempo real, uso de CPU/memoria |
| `Ctrl + C` | Detiene el proceso que está corriendo en tu terminal en este momento |

**Por qué importa para el curso:** cuando levantes el laboratorio con
`docker compose up`, cada contenedor es en el fondo un **proceso aislado** corriendo en tu
máquina. Entender qué es un proceso te ayuda a entender qué es un contenedor — el mismo
contraste que ya viste en la tabla VM vs. Contenedor de la Sesión 7.

### 2.6 Buscar y combinar comandos (lo que hace la terminal realmente poderosa)

| Comando | Qué hace | Ejemplo |
|---|---|---|
| `grep <texto> <archivo>` | Busca líneas que contienen ese texto dentro de un archivo | `grep error log.txt` |
| `find <carpeta> -name <patrón>` | Busca archivos por nombre dentro de una carpeta | `find . -name "*.txt"` |
| `history` | Muestra los comandos que ya ejecutaste antes | `history` |
| `man <comando>` | Abre el manual de ese comando (sales con `q`) | `man ls` |

La terminal no solo ejecuta comandos uno por uno — los puedes **combinar**:

- `>` guarda la salida de un comando en un archivo (lo sobrescribe):
  ```bash
  ls -la > lista.txt
  ```
- `>>` agrega la salida al final de un archivo, sin borrar lo que ya tenía:
  ```bash
  echo "nueva línea" >> apuntes.txt
  ```
- `|` (pipe, "tubería") conecta la salida de un comando con la entrada del siguiente:
  ```bash
  ls -la | grep ".txt"
  ```
  Esto lista todo y de esa lista se queda solo con las líneas que contienen `.txt` — dos
  comandos simples encadenados para lograr algo que ninguno hace solo.

**Ejercicio 4:** dentro de `lab-redes`, corre `ls -la | grep apuntes` y confirma que solo
aparece la línea de `apuntes.txt`. Luego corre `history | grep cd` para ver todas las
veces que usaste `cd` en esta sesión.

### 2.7 Permisos de administrador: `sudo`

Algunos comandos necesitan permisos de administrador del sistema (instalar programas,
cambiar configuración del sistema operativo). Para eso se antepone `sudo` (super user do):

```bash
sudo apt update
sudo apt install curl
```

`apt` es el **gestor de paquetes** de Ubuntu/Debian — instala programas descargándolos
desde repositorios oficiales, en vez de buscar un instalador manualmente como en Windows.
Vas a usar exactamente este patrón (`sudo apt install ...`) para instalar Docker más
adelante en el curso.

⚠️ `sudo` te da permiso de hacer **cualquier cosa** en el sistema, incluyendo romperlo —
úsalo solo cuando el comando lo pida explícitamente, no "por si acaso".

---

## Parte 3 — Chuleta rápida, imprime o guarda esto (≈5 min)

```
pwd                 dónde estoy
ls / ls -la         qué hay aquí
cd <carpeta>        entrar
cd ..               subir un nivel
cd ~                ir a mi carpeta de usuario
mkdir <nombre>      crear carpeta
touch <archivo>     crear archivo vacío
cp origen destino   copiar
mv origen destino   mover / renombrar
rm <archivo>        borrar archivo (¡sin papelera!)
rm -r <carpeta>     borrar carpeta completa
cat <archivo>       ver contenido completo
nano <archivo>      editar (Ctrl+O guardar, Ctrl+X salir)
chmod +x <archivo>  dar permiso de ejecución
ps aux              ver procesos
Ctrl + C            detener lo que está corriendo
grep texto archivo  buscar texto dentro de un archivo
find . -name "*.x"  buscar archivos por nombre
comando > archivo   guardar la salida en un archivo (sobrescribe)
comando >> archivo  agregar la salida al final de un archivo
comando1 | comando2 conectar la salida de uno con la entrada del otro
history             ver comandos anteriores
man comando         abrir el manual de ese comando
sudo comando        ejecutar con permisos de administrador
```

---

## Verificación final antes de pasar a Etapa 1 (≈5 min)

Antes de continuar con el curso, confirma que puedes hacer esto sin ayuda:

1. Abrir tu terminal (Ubuntu en WSL2, o Terminal en Mac, o dentro de tu VM).
2. Crear una carpeta llamada `verificacion-etapa0`.
3. Entrar a ella y crear un archivo `listo.txt` con tu nombre completo adentro (usando
   `nano`).
4. Mostrar el contenido con `cat`.
5. Salir de la carpeta (`cd ..`) y borrarla completa con `rm -r`.

Si completaste los 5 pasos sin trabarte, estás listo para Etapa 1.

---

## Cierre: recorriendo el hilo conductor completo

Ya tienes las cuatro piezas de las cuatro sesiones. Todo el curso (DNS, DHCP, web, FTP,
correo) es una variación del mismo escenario: **alguien pide algo, algo responde**. Este
es el recorrido técnico completo, de tu laptop real hasta la página en pantalla:

```
Tu laptop real (Windows/Mac)
        │
        v
Máquina Virtual (Sesión 7)
        │
        v
Linux + terminal, dentro de la VM (Sesiones 3 y 4)
        │
        v
Red: switches y routers que conectan tu VM con un servidor (Sesión 5)
        │
        v
Tráfico: paquetes siguiendo un protocolo, HTTP/DNS (Sesión 6)
        │
        v
La página web aparece en tu navegador
```

1. Tienes tu **laptop real** (Windows o Mac).
2. Adentro corre la **VM** con Ubuntu que instalaste en la Sesión 7.
3. Dentro de esa VM controlas todo desde una **terminal Linux** — la que acabas de
   practicar en esta sesión.
4. Esa VM está conectada a una **red** — a un servidor DNS y a un servidor web, con
   switches para lo local y routers para salir a Internet (Sesión 5).
5. Cuando pides una página, la petición y la respuesta viajan como **tráfico**: paquetes
   siguiendo el protocolo HTTP, después de que DNS resolvió el nombre (Sesión 6).

Ese recorrido completo — de tu laptop real hasta el contenido de una página, pasando por
VM, Linux, red y tráfico — es exactamente lo que vas a construir con tus propias manos
durante el resto del curso.

---

## Referencias

Microsoft. (s.f.). *Install WSL*. Microsoft Learn.
https://learn.microsoft.com/windows/wsl/install
