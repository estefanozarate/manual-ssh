# SSH-AGENT(1)

## NOMBRE

**ssh-agent** — agente de autenticación de OpenSSH

## SINOPSIS

```
ssh-agent [-c | -s] [-DdU] [-T | -A directory | -a bind_address] [-E fingerprint_hash] [-O option] [-P allowed_providers] [-t life]

ssh-agent [-U] [-T | -A directory | -a bind_address] [-E fingerprint_hash] [-O option] [-P allowed_providers] [-t life] command [arg ...]

ssh-agent [-c | -s] -k

ssh-agent -u

ssh-agent -V
```

## DESCRIPCIÓN

**ssh-agent** es un programa que guarda las claves privadas usadas en la autenticación por clave pública. Mediante variables de entorno, el agente puede localizarse y usarse automáticamente para autenticarse al iniciar sesión en otras máquinas con `ssh(1)`.

Las opciones son las siguientes:

**`-a bind_address`**
: Asocia el agente al socket de dominio Unix *bind_address*. Por defecto se crea un socket en el directorio `$HOME/.ssh/agent` con una ruta aleatoria que coincide con `s.*`.

**`-A socket_path`**
: Especifica otra ruta de directorio bajo la cual crear el socket. Los sockets pueden crearse en una ubicación compartida o en un directorio específico del usuario. Los directorios específicos del usuario se indican anteponiendo `user:` a la ruta. **ssh-agent** se asegurará de que el directorio exista y creará el socket de escucha directamente en él. Las rutas relativas de directorios específicos del usuario se crearán relativas al `$HOME` del usuario.

  Los directorios compartidos pueden indicarse anteponiendo `shared:` a una ruta absoluta. En ese caso se creará un subdirectorio temporal bajo el directorio especificado y el socket de escucha del agente se creará dentro de él.

  Esta opción acepta los tokens descritos en la sección TOKENS de `sshd_config(5)`.

**`-c`**
: Genera comandos de C-shell por la salida estándar. Es el comportamiento por defecto si `SHELL` parece ser una shell de estilo csh.

**`-D`**
: Modo en primer plano. Con esta opción, **ssh-agent** no hará fork.

**`-d`**
: Modo de depuración. Con esta opción, **ssh-agent** no hará fork y escribirá información de depuración en la salida de error estándar.

**`-E fingerprint_hash`**
: Especifica el algoritmo de hash usado al mostrar las huellas (*fingerprints*) de las claves. Las opciones válidas son “md5” y “sha256”. El valor por defecto es “sha256”.

**`-k`**
: Mata el agente actual (indicado por la variable de entorno `SSH_AGENT_PID`).

**`-O option`**
: Especifica una opción al iniciar **ssh-agent**. Las opciones soportadas son: `allow-remote-pkcs11`, `no-restrict-websafe` y `websafe-allow`.

  La opción `allow-remote-pkcs11` permite que los clientes de un **ssh-agent** reenviado carguen bibliotecas de proveedores PKCS#11 o FIDO. Por defecto solo los clientes locales pueden realizar esta operación. Ten en cuenta que es `ssh(1)` quien señala que un cliente de **ssh-agent** es remoto, y el uso de otras herramientas para reenviar el acceso al socket del agente podría eludir esta restricción.

  La opción `no-restrict-websafe` indica a **ssh-agent** que permita firmas con claves FIDO que podrían ser peticiones de autenticación web. Por defecto, **ssh-agent** rechaza las peticiones de firma con claves FIDO cuya cadena de aplicación no empiece por “ssh:” y cuando los datos a firmar no parezcan una petición de autenticación de usuario de `ssh(1)` o una firma de `ssh-keygen(1)`. El comportamiento por defecto evita que el acceso reenviado a una clave FIDO reenvíe también, de forma implícita, la capacidad de autenticarse en sitios web.

  Como alternativa, la opción `websafe-allow` permite especificar una lista de patrones de cadenas de aplicación de clave que sustituye a la lista de permitidos por defecto, por ejemplo: “websafe-allow=ssh:*,example.org,*.example.com”

  Consulta PATTERNS en `ssh_config(5)` para la sintaxis de las listas de patrones.

**`-P allowed_providers`**
: Especifica una lista de patrones de rutas aceptables para las bibliotecas compartidas de proveedores PKCS#11 y de middleware de autenticadores FIDO que pueden usarse con las opciones `-S` o `-s` de `ssh-add(1)`. Las bibliotecas que no coincidan con la lista de patrones se rechazarán. La lista por defecto es “/usr/lib/\*,/usr/local/lib/\*”.

  Consulta PATTERNS en `ssh_config(5)` para la sintaxis de las listas de patrones.

**`-s`**
: Genera comandos de Bourne shell por la salida estándar. Es el comportamiento por defecto si `SHELL` no parece ser una shell de estilo csh.

**`-T`**
: Asocia el socket del agente en un subdirectorio aleatorio de la forma `$TMPDIR/ssh-XXXXXXXXXX/agent.<ppid>`, en lugar del comportamiento por defecto de usar un nombre aleatorio que coincide con `$HOME/.ssh/agent/s.*`.

**`-t life`**
: Establece un valor por defecto para la vida máxima de las identidades añadidas al agente. La vida puede indicarse en segundos o en un formato de tiempo de los especificados en `sshd_config(5)`. Una vida indicada para una identidad con `ssh-add(1)` sobrescribe este valor. Sin esta opción, la vida máxima por defecto es ilimitada.

**`-U`**
: Indica a **ssh-agent** que no limpie los sockets de agente obsoletos bajo `$HOME/.ssh/agent/`.

**`-u`**
: Indica a **ssh-agent** que solo limpie los sockets de agente obsoletos bajo `$HOME/.ssh/agent/` y termine inmediatamente. Si esta opción se indica dos veces, **ssh-agent** borrará los sockets obsoletos sin importar el nombre de host que los creó.

**`command [arg ...]`**
: Si se indica un comando (con argumentos opcionales), se ejecuta como subproceso del agente. El agente termina automáticamente cuando termina el comando indicado en la línea de comandos.

**`-V`**
: Muestra el número de versión y termina.

Hay dos formas principales de poner en marcha un agente. La primera es al inicio de una sesión X, donde todas las demás ventanas o programas se inician como hijos del programa **ssh-agent**. El agente inicia un comando en el que se exportan sus variables de entorno, por ejemplo `ssh-agent xterm &`. Cuando el comando termina, también lo hace el agente.

El segundo método se usa para una sesión de inicio (*login*). Cuando se inicia **ssh-agent**, imprime los comandos de shell necesarios para establecer sus variables de entorno, que a su vez pueden evaluarse en la shell que lo invocó, por ejemplo ``eval `ssh-agent -s` ``.

En ambos casos, `ssh(1)` consulta estas variables de entorno y las usa para establecer una conexión con el agente.

Inicialmente el agente no tiene ninguna clave privada. Las claves se añaden con `ssh-add(1)` o mediante `ssh(1)` cuando `AddKeysToAgent` está establecido en `ssh_config(5)`. En **ssh-agent** pueden almacenarse varias identidades a la vez y `ssh(1)` las usará automáticamente si están presentes. `ssh-add(1)` también se usa para eliminar claves de **ssh-agent** y para consultar las claves que contiene.

Las conexiones a **ssh-agent** pueden reenviarse desde hosts remotos más lejanos usando la opción `-A` de `ssh(1)` (pero consulta las advertencias documentadas allí), evitando la necesidad de almacenar datos de autenticación en otras máquinas. Las frases de contraseña y las claves privadas nunca viajan por la red: la conexión al agente se reenvía a través de las conexiones remotas SSH y el resultado se devuelve al solicitante, lo que permite al usuario acceder a sus identidades desde cualquier punto de la red de forma segura.

**ssh-agent** borrará todas las claves que tenga cargadas al recibir `SIGUSR1`.

## ENTORNO

**`SSH_AGENT_PID`**
: Cuando **ssh-agent** se inicia, guarda en esta variable el ID de proceso (PID) del agente.

**`SSH_AUTH_SOCK`**
: Cuando **ssh-agent** se inicia, crea un socket de dominio Unix y guarda su ruta en esta variable. Solo es accesible para el usuario actual, pero root u otra instancia del mismo usuario pueden abusar de él fácilmente.

## ARCHIVOS

**`$HOME/.ssh/agent/s.*`**
: Sockets de dominio Unix que contienen la conexión con el agente de autenticación. Estos sockets solo deben ser legibles por su propietario. Deberían eliminarse automáticamente cuando el agente termina.

## VÉASE TAMBIÉN

`ssh(1)`, `ssh-add(1)`, `ssh-keygen(1)`, `ssh_config(5)`, `sshd(8)`

## AUTORES

OpenSSH es un derivado de la versión original y libre ssh 1.2.12 de Tatu Ylonen. Aaron Campbell, Bob Beck, Markus Friedl, Niels Provos, Theo de Raadt y Dug Song eliminaron muchos errores, volvieron a añadir funciones más recientes y crearon OpenSSH. Markus Friedl contribuyó el soporte para las versiones 1.5 y 2.0 del protocolo SSH.
