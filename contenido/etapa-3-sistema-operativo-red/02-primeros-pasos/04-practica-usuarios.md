# Práctica: usuarios y sudo

> Primeros pasos 4 · ≈50 min · [← El administrador, sudo y los paquetes](03-administrador-y-paquetes.md) ·
> Siguiente: [Archivos, carpetas y permisos](05-archivos-y-permisos.md)

## Qué vas a hacer

En esta práctica juntas lo que viste en las dos lecturas anteriores,
[Usuarios](02-usuarios.md) y [El administrador, sudo y los paquetes](03-administrador-y-paquetes.md):

1. Vas a **crear un usuario nuevo** en tu servidor, como si llegara una persona nueva.
2. Vas a comprobar que **cada usuario tiene su propio espacio** y no puede entrar al
   del otro.
3. Vas a **entrar como ese usuario** sin cerrar tu sesión.
4. Vas a comprobar que ese usuario **no puede usar `sudo`**, porque no es
   administrador.

Antes de empezar, enciende tu VM e inicia sesión con tu usuario. Sigue guardando las
capturas en tu carpeta `capturas-admredes`, como se explica en
[Cómo tomar y guardar las capturas](01-primer-inicio.md#cómo-tomar-y-guardar-las-capturas).

## Cómo seguir las instrucciones

- Los comandos aparecen en recuadros grises. **Escríbelos tal cual**, con las mismas
  minúsculas y guiones, y presiona **Enter** después de cada uno.
- **Escríbelos tú**: no los copies y pegues.
- Debajo de cada comando se explica **qué deberías ver**. Compáralo con tu pantalla
  antes de pasar al siguiente.

## Paso 1: Crear un usuario nuevo

Vas a crear una segunda cuenta en tu servidor. Se va a llamar **`user-practica-01`**. El
comando es `adduser` (de *add user*, "agregar usuario"):

```bash
sudo adduser user-practica-01
```

Crear usuarios cambia todo el servidor: es una tarea de administrador, y por eso el
comando empieza con `sudo`. Si no recuerdas qué hace `sudo`, repasa la
[sección 3](03-administrador-y-paquetes.md#3-sudo-pedir-permiso-de-administrador-para-un-solo-comando)
de la lectura anterior.

El comando te va a hacer varias preguntas, una tras otra. Los textos pueden salir en
inglés o en español:

1. **Primero pide TU contraseña**, la de tu propio usuario: es `sudo` confirmando que
   eres tú. Recuerda que **no se ve nada mientras la escribes**. Escríbela y presiona
   Enter. (Si usaste `sudo` hace pocos minutos, puede que no te la pida.)
2. Luego pide la contraseña **del usuario nuevo** (`New password`). Escribe
   **`123123`** y Enter. Tampoco se ve nada al escribirla.
3. Te pide repetirla (`Retype new password`): escribe otra vez **`123123`** y Enter.
   Si aparece un aviso de que la contraseña es débil o muy corta, es normal: en esta
   práctica la usamos así a propósito.
4. Después pregunta datos de la persona: nombre completo (`Full Name`), número de
   oficina, teléfonos y otros. **Todos son opcionales**: presiona **Enter** en cada
   uno para dejarlos vacíos.
5. Al final pregunta si la información es correcta (`Is the information correct?
   [Y/n]`). Presiona **Enter**: la letra en mayúscula es la respuesta por defecto, Sí.

⚠️ **`123123` es una contraseña pésima**, y la usamos solo porque es una cuenta de
práctica en tu VM, que no está conectada a ninguna red compartida. En un servidor real,
**nunca** se crea una cuenta con una contraseña así: sería lo primero que probaría un
atacante.

Comprueba que el usuario existe:

```bash
ls /home
id user-practica-01
```

- `ls /home` ahora muestra **dos carpetas**: la tuya y la de `user-practica-01`. El
  sistema le creó su carpeta personal automáticamente.
- `id user-practica-01` muestra su número de usuario, que es **el siguiente al tuyo**
  (1001), y sus grupos. Compáralo con tu propio `id`, el de tu captura `NombreApellido-MiUsuario.png`:
  **`user-practica-01` no está en el grupo `sudo`**. Es un usuario normal, sin permisos
  de administrador.

> 📸 **Captura `NombreApellido-NuevoUsuario.png`:** la salida de `ls /home` y de
> `id user-practica-01`.

## Paso 2: Cada quien en su carpeta

Intenta ver qué hay en la carpeta personal del usuario nuevo:

```bash
ls /home/user-practica-01
```

Deberías ver un error que dice **Permission denied** ("permiso denegado"). Es la
**privacidad** que viste en la
[sección 5 de Usuarios](02-usuarios.md#5-para-qué-separar-a-los-usuarios): aunque están
en el mismo servidor, no puedes entrar a la carpeta personal de otro usuario. Por qué
pasa exactamente lo vas a entender en la siguiente lectura, la de
[permisos](05-archivos-y-permisos.md).

## Paso 3: Entrar como el otro usuario

El comando `su` (de *substitute user*, "sustituir usuario") te cambia a otra cuenta sin
cerrar tu sesión (Kerrisk, s.f.). El guion `-` hace que entres como si acabaras de
iniciar sesión con esa cuenta, directo en su carpeta personal:

```bash
su - user-practica-01
```

Te pide la contraseña **del usuario al que te cambias**: escribe `123123` y Enter.

Fíjate en el **prompt**: ya no dice tu nombre, ahora dice `user-practica-01@admredes`.
Eres otra persona dentro del mismo servidor. Compruébalo:

```bash
whoami
pwd
```

`whoami` dice `user-practica-01` y `pwd` dice `/home/user-practica-01`: estás en
**su** carpeta. Ahora intenta entrar a la tuya. Cambia `usuario` por **tu** nombre de
usuario:

```bash
ls /home/usuario
```

Otra vez **Permission denied**: la privacidad funciona en los dos sentidos.

> 📸 **Captura `NombreApellido-CambioDeUsuario.png`:** el prompt con
> `user-practica-01`, la salida de `whoami` y `pwd`, y el error al intentar ver tu
> carpeta.

## Paso 4: Un usuario que no es administrador

Sin salir de `user-practica-01`, intenta actualizar la lista de paquetes, igual que
hiciste con tu cuenta en la lectura anterior:

```bash
sudo apt update
```

Si `sudo` te pide contraseña, escribe la de **`user-practica-01`** (`123123`). Aun con
la contraseña correcta, `sudo` **se niega**: aparece un mensaje de que este usuario no
tiene permiso para usar `sudo`, y la lista no se actualiza. Es lo que viste en
[Quién puede usar sudo](03-administrador-y-paquetes.md#quién-puede-usar-sudo):
`user-practica-01` **no está en el grupo `sudo`**.

> 📸 **Captura `NombreApellido-SudoNegado.png`:** el prompt con `user-practica-01` y el
> mensaje de `sudo` negándose.

## Paso 5: Regresar a tu cuenta

Para dejar de ser `user-practica-01` y regresar a tu propia cuenta, escribe:

```bash
exit
whoami
```

El prompt vuelve a mostrar tu nombre, y `whoami` lo confirma. **Siempre regresa con
`exit`** cuando termines de usar otra cuenta.

**No borres `user-practica-01`**: lo vas a usar de nuevo en la lectura de permisos.

## Comandos que usaste

```
sudo adduser <nombre>   crea un usuario nuevo (con su carpeta en /home)
id <usuario>            muestra el UID y los grupos de otro usuario
su - <usuario>          entra como otro usuario (pide SU contraseña)
exit                    regresa a tu propia cuenta
```

## Referencias

Kerrisk, M. (s.f.). *su(1): Linux manual page*. The Linux man-pages project.
https://man7.org/linux/man-pages/man1/su.1.html
