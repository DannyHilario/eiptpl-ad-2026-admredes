# Paso 14: Primer inicio de sesión

> Paso 14 de 14 · [← Instalación y reinicio](13-instalacion.md)

Después de reiniciar aparecen varias líneas de texto del arranque y al final:

```
admredes login:
```

Escribe tu usuario, Enter, y luego tu contraseña. **Mientras escribes la contraseña no
aparece nada en pantalla**, ni asteriscos: es normal en Linux, el sistema sí está
recibiendo cada tecla.

## Verificar que todo quedó bien

Ya dentro, con el prompt `usuario@admredes:~$`, cuatro comandos confirman lo que se
configuró durante la instalación:

```bash
hostnamectl              # nombre del servidor, versión de Ubuntu y arquitectura
ip a                     # la IP de la tarjeta (10.0.2.15 si sigue en NAT)
ping -c 3 ubuntu.com     # hay Internet y el DNS funciona
systemctl status ssh     # el servidor SSH dice "active (running)"
```

Con eso el servidor está listo. Lo primero que conviene hacer es actualizarlo:

```bash
sudo apt update && sudo apt upgrade
```

## Si no aceptó la contraseña

Antes de suponer que se escribió mal al instalar, revisa el **teclado**: en el campo
`login:` sí se ve lo que escribes, así que puedes teclear ahí los números y símbolos de
la contraseña para comprobar qué caracteres produce cada tecla, y luego borrarlos.

Si de verdad no se recuerda, el administrador puede restablecerla desde el **modo de
recuperación** (*recovery mode*) de Ubuntu (Ubuntu Wiki, s.f.):

1. Reiniciar la VM y presionar **Esc** repetidamente al arrancar, hasta ver el menú
   **GRUB**.
2. **Advanced options for Ubuntu** → la opción que termina en **(recovery mode)**.
3. En el menú de recuperación, elegir **root** (consola de administrador).
4. Volver el disco escribible y cambiar la contraseña:

```bash
mount -o remount,rw /
passwd usuario
reboot
```

## La lección de seguridad

Fíjate en lo que acaba de pasar: **cualquiera con acceso a la consola** pudo cambiar la
contraseña sin conocer la anterior. Por eso los servidores reales viven en salas con
acceso controlado, y por eso existe el cifrado de disco **LUKS** que se dejó desmarcado
en el [Paso 7](07-almacenamiento.md): con el disco cifrado, este procedimiento no
funciona sin la clave de cifrado. Es un ejemplo directo de que la **seguridad física** es
parte de la [administración de seguridad](../../etapa-1-funciones-administracion/05-administracion-seguridad.md).

## Referencias

Ubuntu Wiki. (s.f.). *RecoveryMode*. https://wiki.ubuntu.com/RecoveryMode
