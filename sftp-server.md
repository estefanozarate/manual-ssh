# SFTP-SERVER(8)

## NOMBRE

**sftp-server** — subsistema servidor SFTP de OpenSSH

## SINOPSIS

```
sftp-server [-ehR] [-d start_directory] [-f log_facility] [-l log_level] [-P denied_requests] [-p allowed_requests] [-u umask]

sftp-server -Q protocol_feature
```

## DESCRIPCIÓN

**sftp-server** es un programa que habla el lado servidor del protocolo SFTP hacia la salida estándar (stdout) y espera las peticiones del cliente por la entrada estándar (stdin). **sftp-server** no está pensado para invocarse directamente, sino desde `sshd(8)` mediante la opción `Subsystem`.

Los flags de línea de comandos para **sftp-server** deben especificarse en la declaración `Subsystem`. Consulta `sshd_config(5)` para más información.

Las opciones válidas son:

**`-d start_directory`**
: Especifica un directorio inicial alternativo para los usuarios. La ruta puede contener los siguientes tokens, que se expanden en tiempo de ejecución: `%%` se reemplaza por un `'%'` literal, `%d` se reemplaza por el directorio home del usuario que se está autenticando y `%u` se reemplaza por el nombre de ese usuario. Por defecto se usa el directorio home del usuario. Esta opción es útil junto con la opción `ChrootDirectory` de `sshd_config(5)`.

**`-e`**
: Hace que **sftp-server** imprima la información de registro (logging) por stderr en lugar de syslog, para depuración.

**`-f log_facility`**
: Especifica el código de *facility* usado al registrar mensajes de **sftp-server**. Los valores posibles son: `DAEMON`, `USER`, `AUTH`, `LOCAL0`, `LOCAL1`, `LOCAL2`, `LOCAL3`, `LOCAL4`, `LOCAL5`, `LOCAL6`, `LOCAL7`. El valor por defecto es `AUTH`.

**`-h`**
: Muestra la información de uso de **sftp-server**.

**`-l log_level`**
: Especifica qué mensajes registrará **sftp-server**. Los valores posibles son: `QUIET`, `FATAL`, `ERROR`, `INFO`, `VERBOSE`, `DEBUG`, `DEBUG1`, `DEBUG2` y `DEBUG3`. `INFO` y `VERBOSE` registran las transacciones que **sftp-server** realiza en nombre del cliente. `DEBUG` y `DEBUG1` son equivalentes. `DEBUG2` y `DEBUG3` especifican cada uno niveles más altos de salida de depuración. El valor por defecto es `ERROR`.

**`-P denied_requests`**
: Especifica una lista, separada por comas, de peticiones del protocolo SFTP que el servidor prohíbe. **sftp-server** responderá con un fallo a cualquier petición denegada. El flag `-Q` puede usarse para conocer los tipos de petición soportados. Si se especifican tanto la lista de denegadas como la de permitidas, la lista de denegadas se aplica antes que la de permitidas. Este flag, junto con `-p`, puede usarse para desactivar operaciones irrelevantes o no deseadas para el servidor. Por ejemplo, un **sftp-server** que acepta conexiones de clientes no confiables podría querer desactivar las operaciones “copy-data” o “users-groups-by-id”.

**`-p allowed_requests`**
: Especifica una lista, separada por comas, de peticiones del protocolo SFTP que el servidor permite. Todos los tipos de petición que no estén en la lista de permitidas se registrarán y se responderán con un mensaje de fallo.

  Hay que tener cuidado al usar esta función para asegurarse de que se permiten las peticiones que los clientes SFTP realizan de forma implícita.

**`-Q protocol_feature`**
: Consulta las características del protocolo soportadas por **sftp-server**. Actualmente la única característica consultable es “requests”, que puede usarse para denegar o permitir peticiones concretas (flags `-P` y `-p` respectivamente).

**`-R`**
: Pone esta instancia de **sftp-server** en modo de solo lectura. Se denegarán los intentos de abrir archivos para escritura, así como otras operaciones que cambien el estado del sistema de archivos.

**`-u umask`**
: Establece un `umask(2)` explícito que se aplicará a los archivos y directorios recién creados, en lugar de la máscara por defecto del usuario.

En algunos sistemas, **sftp-server** necesita acceder a `/dev/log` para que el registro funcione, por lo que usar **sftp-server** en una configuración chroot requiere que `syslogd(8)` establezca un socket de registro dentro del directorio chroot.

## VÉASE TAMBIÉN

`sftp(1)`, `ssh(1)`, `sshd_config(5)`, `sshd(8)`

T. Ylonen y S. Lehtinen, *SSH File Transfer Protocol*, draft-ietf-secsh-filexfer-02.txt, octubre de 2001, material de trabajo en curso.

## HISTORIA

**sftp-server** apareció por primera vez en OpenBSD 2.8.

## AUTORES

Markus Friedl <markus@openbsd.org>
