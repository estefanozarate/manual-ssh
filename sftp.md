# SFTP(1)

## NOMBRE

**sftp** — transferencia segura de archivos de OpenSSH

## SINOPSIS

```
sftp [-46AaCfNpqrv] [-B buffer_size] [-b batchfile] [-c cipher] [-D sftp_server_command] [-F ssh_config] [-i identity_file] [-J destination] [-l limit] [-o ssh_option] [-P port] [-R num_requests] [-S program] [-s subsystem | sftp_server] [-X sftp_option] destination
```

## DESCRIPCIÓN

**sftp** es un programa de transferencia de archivos, similar a `ftp(1)`, que realiza todas las operaciones sobre un transporte cifrado `ssh(1)`. También puede usar muchas funciones de ssh, como la autenticación por clave pública y la compresión.

El destino puede especificarse como `[user@]host[:path]` o como una URI de la forma `sftp://[user@]host[:port][/path]`.

Si el destino incluye una ruta y no es un directorio, **sftp** descargará los archivos automáticamente si se usa un método de autenticación no interactivo; en caso contrario, lo hará tras una autenticación interactiva exitosa.

Si no se especifica ruta, o si la ruta es un directorio, **sftp** iniciará sesión en el host indicado y entrará en el modo de comandos interactivo, cambiándose al directorio remoto si se especificó uno. Se puede usar una barra final opcional para forzar que la ruta se interprete como directorio.

Como los formatos de destino usan dos puntos para separar los nombres de host de las rutas o números de puerto, las direcciones IPv6 deben ir entre corchetes para evitar ambigüedades.

Las opciones son las siguientes:

**`-4`**
: Obliga a **sftp** a usar solo direcciones IPv4.

**`-6`**
: Obliga a **sftp** a usar solo direcciones IPv6.

**`-A`**
: Permite el reenvío de `ssh-agent(1)` al sistema remoto. Por defecto no se reenvía el agente de autenticación.

**`-a`**
: Intenta continuar las transferencias interrumpidas en lugar de sobrescribir las copias parciales o completas existentes. Si el contenido parcial difiere del que se está transfiriendo, es probable que el archivo resultante quede corrupto.

**`-B buffer_size`**
: Especifica el tamaño del búfer que **sftp** usa al transferir archivos. Los búferes más grandes requieren menos viajes de ida y vuelta a costa de un mayor consumo de memoria. El valor por defecto es 32768 bytes.

**`-b batchfile`**
: El modo por lotes lee una serie de comandos desde un archivo *batchfile* en lugar de stdin. Como no hay interacción con el usuario, debe usarse junto con una autenticación no interactiva para evitar tener que introducir una contraseña al conectar (consulta `sshd(8)` y `ssh-keygen(1)` para más detalles).

  Se puede usar `-` como *batchfile* para indicar la entrada estándar. **sftp** abortará si falla cualquiera de los siguientes comandos: `get`, `put`, `reget`, `reput`, `rename`, `ln`, `rm`, `mkdir`, `chdir`, `ls`, `lchdir`, `copy`, `cp`, `chmod`, `chown`, `chgrp`, `lpwd`, `df`, `symlink` y `lmkdir`.

  La terminación por error puede suprimirse comando a comando anteponiendo un carácter `-` al comando (por ejemplo, `-rm /tmp/blah*`). El eco del comando puede suprimirse anteponiendo un carácter `@`. Ambos prefijos pueden combinarse en cualquier orden, por ejemplo `-@ls /bsd`.

**`-C`**
: Activa la compresión (mediante el flag `-C` de ssh).

**`-c cipher`**
: Selecciona el cifrado usado para cifrar las transferencias de datos. Esta opción se pasa directamente a `ssh(1)`.

**`-D sftp_server_command`**
: Se conecta directamente a un servidor sftp local (en lugar de a través de `ssh(1)`). Se puede especificar un comando con argumentos, por ejemplo `"/path/sftp-server -el debug3"`. Puede ser útil para depurar el cliente y el servidor.

**`-F ssh_config`**
: Especifica un archivo de configuración de usuario alternativo para `ssh(1)`. Esta opción se pasa directamente a `ssh(1)`.

**`-f`**
: Solicita que los archivos se vuelquen a disco inmediatamente tras la transferencia. Al subir archivos, esta función solo se activa si el servidor implementa la extensión "fsync@openssh.com".

**`-i identity_file`**
: Selecciona el archivo desde el que se lee la identidad (clave privada) para la autenticación por clave pública. Esta opción se pasa directamente a `ssh(1)`.

**`-J destination`**
: Se conecta al host objetivo estableciendo primero una conexión sftp con el host de salto descrito por *destination* y, desde ahí, estableciendo un reenvío TCP hasta el destino final. Se pueden indicar varios saltos separados por comas. Es un atajo para especificar la directiva de configuración `ProxyJump`. Esta opción se pasa directamente a `ssh(1)`.

**`-l limit`**
: Limita el ancho de banda usado, especificado en Kbit/s.

**`-N`**
: Desactiva el modo silencioso, p. ej. para anular el modo silencioso implícito que establece el flag `-b`.

**`-o ssh_option`**
: Puede usarse para pasar opciones a ssh en el formato usado en `ssh_config(5)`. Es útil para especificar opciones que no tienen un flag propio en **sftp**. Por ejemplo, para indicar un puerto alternativo: `sftp -oPort=24`. Para todos los detalles de las opciones listadas a continuación y sus valores posibles, consulta `ssh_config(5)`.

  `AddKeysToAgent`, `AddressFamily`, `BatchMode`, `BindAddress`, `BindInterface`, `CASignatureAlgorithms`, `CanonicalDomains`, `CanonicalizeFallbackLocal`, `CanonicalizeHostname`, `CanonicalizeMaxDots`, `CanonicalizePermittedCNAMEs`, `CertificateFile`, `ChannelTimeout`, `CheckHostIP`, `Ciphers`, `ClearAllForwardings`, `Compression`, `ConnectTimeout`, `ConnectionAttempts`, `ControlMaster`, `ControlPath`, `ControlPersist`, `DynamicForward`, `EnableEscapeCommandline`, `EnableSSHKeysign`, `EscapeChar`, `ExitOnForwardFailure`, `FingerprintHash`, `ForkAfterAuthentication`, `ForwardAgent`, `ForwardX11`, `ForwardX11Timeout`, `ForwardX11Trusted`, `GSSAPIAuthentication`, `GSSAPIDelegateCredentials`, `GatewayPorts`, `GlobalKnownHostsFile`, `HashKnownHosts`, `Host`, `HostKeyAlgorithms`, `HostKeyAlias`, `HostbasedAcceptedAlgorithms`, `HostbasedAuthentication`, `Hostname`, `IPQoS`, `IdentitiesOnly`, `IdentityAgent`, `IdentityFile`, `IgnoreUnknown`, `Include`, `KbdInteractiveAuthentication`, `KbdInteractiveDevices`, `KexAlgorithms`, `KnownHostsCommand`, `LocalCommand`, `LocalForward`, `LogLevel`, `LogVerbose`, `MACs`, `NoHostAuthenticationForLocalhost`, `NumberOfPasswordPrompts`, `ObscureKeystrokeTiming`, `PKCS11Provider`, `PasswordAuthentication`, `PermitLocalCommand`, `PermitRemoteOpen`, `Port`, `PreferredAuthentications`, `ProxyCommand`, `ProxyJump`, `ProxyUseFdpass`, `PubkeyAcceptedAlgorithms`, `PubkeyAuthentication`, `RekeyLimit`, `RemoteCommand`, `RemoteForward`, `RequestTTY`, `RequiredRSASize`, `RevokedHostKeys`, `SecurityKeyProvider`, `SendEnv`, `ServerAliveCountMax`, `ServerAliveInterval`, `SessionType`, `SetEnv`, `StdinNull`, `StreamLocalBindMask`, `StreamLocalBindUnlink`, `StrictHostKeyChecking`, `SyslogFacility`, `TCPKeepAlive`, `Tag`, `Tunnel`, `TunnelDevice`, `UpdateHostKeys`, `User`, `UserKnownHostsFile`, `VerifyHostKeyDNS`, `VisualHostKey`, `XAuthLocation`

**`-P port`**
: Especifica el puerto al que conectarse en el host remoto.

**`-p`**
: Preserva las fechas de modificación, las fechas de acceso y los modos de los archivos originales transferidos.

**`-q`**
: Modo silencioso: desactiva el indicador de progreso, así como los mensajes de advertencia y diagnóstico de `ssh(1)`.

**`-R num_requests`**
: Especifica cuántas peticiones pueden estar pendientes a la vez. Aumentarlo puede mejorar ligeramente la velocidad de transferencia, pero incrementa el uso de memoria. El valor por defecto es 64 peticiones pendientes.

**`-r`**
: Copia directorios completos de forma recursiva al subir y descargar. Ten en cuenta que **sftp** no sigue los enlaces simbólicos que encuentra al recorrer el árbol.

**`-S program`**
: Nombre del programa a usar para la conexión cifrada. El programa debe entender las opciones de `ssh(1)`.

**`-s subsystem | sftp_server`**
: Especifica el subsistema SSH2 o la ruta de un servidor sftp en el host remoto. Indicar una ruta es útil cuando el `sshd(8)` remoto no tiene configurado un subsistema sftp.

**`-v`**
: Eleva el nivel de registro. Esta opción también se pasa a ssh.

**`-X sftp_option`**
: Especifica una opción que controla aspectos del comportamiento del protocolo SFTP. Las opciones válidas son:

  **`nrequests=value`**
  : Controla cuántas peticiones SFTP de lectura o escritura concurrentes puede haber en curso en cualquier momento durante una descarga o subida. Por defecto pueden estar activas 64 peticiones a la vez.

  **`buffer=value`**
  : Controla el tamaño máximo del búfer para una única operación SFTP de lectura/escritura durante una descarga o subida. Por defecto se usa un búfer de 32 KB.

## COMANDOS INTERACTIVOS

Una vez en modo interactivo, **sftp** entiende un conjunto de comandos similar al de `ftp(1)`. Los comandos no distinguen mayúsculas de minúsculas. Las rutas que contienen espacios deben ir entre comillas. Cualquier carácter especial reconocido por `glob(3)` que aparezca en una ruta debe escaparse con barras invertidas (`\`).

**`bye`**
: Sale de **sftp**.

**`cd [path]`**
: Cambia el directorio remoto a *path*. Si no se indica *path*, cambia al directorio en el que empezó la sesión.

**`chgrp [-h] grp path`**
: Cambia el grupo del archivo *path* a *grp*. *path* puede contener caracteres `glob(7)` y coincidir con varios archivos. *grp* debe ser un GID numérico.

  Con el flag `-h`, no se siguen los enlaces simbólicos. Solo lo soportan los servidores que implementan la extensión "lsetstat@openssh.com".

**`chmod [-h] mode path`**
: Cambia los permisos del archivo *path* a *mode*. *path* puede contener caracteres `glob(7)` y coincidir con varios archivos.

  Con el flag `-h`, no se siguen los enlaces simbólicos. Solo lo soportan los servidores que implementan la extensión "lsetstat@openssh.com".

**`chown [-h] own path`**
: Cambia el propietario del archivo *path* a *own*. *path* puede contener caracteres `glob(7)` y coincidir con varios archivos. *own* debe ser un UID numérico.

  Con el flag `-h`, no se siguen los enlaces simbólicos. Solo lo soportan los servidores que implementan la extensión "lsetstat@openssh.com".

**`copy oldpath newpath`**
: Copia el archivo remoto de *oldpath* a *newpath*.

  Solo lo soportan los servidores que implementan la extensión "copy-data".

**`cp oldpath newpath`**
: Alias del comando `copy`.

**`df [-hi] [path]`**
: Muestra la información de uso del sistema de archivos que contiene el directorio actual (o *path*, si se indica). Con el flag `-h`, la capacidad se muestra con sufijos "legibles para humanos". El flag `-i` solicita mostrar información de inodos además de la de capacidad. Este comando solo lo soportan los servidores que implementan la extensión “statvfs@openssh.com”.

**`exit`**
: Sale de **sftp**.

**`get [-afpR] remote-path [local-path]`**
: Descarga *remote-path* y lo guarda en la máquina local. Si no se indica la ruta local, se le da el mismo nombre que tiene en la máquina remota. *remote-path* puede contener caracteres `glob(7)` y coincidir con varios archivos; en ese caso, si se indica *local-path*, debe ser un directorio.

  Con el flag `-a`, intenta reanudar transferencias parciales de archivos existentes. La reanudación asume que cualquier copia parcial del archivo local coincide con la copia remota. Si el contenido del archivo remoto difiere de la copia local parcial, es probable que el archivo resultante quede corrupto.

  Con el flag `-f`, se llamará a `fsync(2)` al terminar la transferencia para volcar el archivo a disco.

  Con el flag `-p`, también se copian los permisos completos y las fechas de acceso del archivo.

  Con el flag `-R`, los directorios se copian de forma recursiva. **sftp** no sigue los enlaces simbólicos al hacer transferencias recursivas.

**`help`**
: Muestra el texto de ayuda.

**`lcd [path]`**
: Cambia el directorio local a *path*. Si no se indica *path*, cambia al directorio home del usuario local.

**`lls [ls-options [path]]`**
: Muestra el listado del directorio local de *path*, o del directorio actual si no se indica. *ls-options* puede contener cualquier flag que admita el comando `ls(1)` del sistema local. *path* puede contener caracteres `glob(7)` y coincidir con varios archivos.

**`lmkdir [-p] path`**
: Crea el directorio local indicado por *path*. Con el flag `-p`, crea los directorios intermedios necesarios y no se considera error que el directorio ya exista.

**`ln [-s] oldpath newpath`**
: Crea un enlace de *oldpath* a *newpath*. Con el flag `-s`, el enlace creado es simbólico; si no, es un enlace duro.

**`lpwd`**
: Imprime el directorio de trabajo local.

**`ls [-1afhlnrSt] [path]`**
: Muestra el listado del directorio remoto de *path*, o del directorio actual si no se indica. *path* puede contener caracteres `glob(7)` y coincidir con varios archivos.

  Se reconocen los siguientes flags, que modifican el comportamiento de `ls`:

  **`-1`**
  : Salida en una sola columna.

  **`-a`**
  : Lista también los archivos que empiezan por punto (`.`).

  **`-f`**
  : No ordena el listado. El orden por defecto es lexicográfico.

  **`-h`**
  : Con una opción de formato largo, usa sufijos de unidad (Byte, Kilobyte, Megabyte, Gigabyte, Terabyte, Petabyte y Exabyte) para reducir el número de dígitos a cuatro o menos, usando potencias de 2 para los tamaños (K=1024, M=1048576, etc.).

  **`-l`**
  : Muestra detalles adicionales, incluidos permisos y propietario.

  **`-n`**
  : Produce un listado largo con la información de usuario y grupo en formato numérico.

  **`-r`**
  : Invierte el orden del listado.

  **`-S`**
  : Ordena el listado por tamaño de archivo.

  **`-t`**
  : Ordena el listado por fecha de última modificación.

**`lumask umask`**
: Establece el umask local a *umask*.

**`mkdir [-p] path`**
: Crea el directorio remoto indicado por *path*. Con el flag `-p`, crea los directorios intermedios necesarios y no se considera error que el directorio ya exista.

**`progress`**
: Activa o desactiva la visualización del indicador de progreso.

**`put [-afpR] local-path [remote-path]`**
: Sube *local-path* y lo guarda en la máquina remota. Si no se indica la ruta remota, se le da el mismo nombre que tiene en la máquina local. *local-path* puede contener caracteres `glob(7)` y coincidir con varios archivos; en ese caso, si se indica *remote-path*, debe ser un directorio.

  Con el flag `-a`, intenta reanudar transferencias parciales de archivos existentes. La reanudación asume que cualquier copia parcial del archivo remoto coincide con la copia local. Si el contenido del archivo local difiere de la copia remota, es probable que el archivo resultante quede corrupto.

  Con el flag `-f`, se enviará al servidor una petición para que llame a `fsync(2)` tras transferir el archivo. Solo lo soportan los servidores que implementan la extensión "fsync@openssh.com".

  Con el flag `-p`, también se copian los permisos completos y las fechas de acceso del archivo.

  Con el flag `-R`, los directorios se copian de forma recursiva. **sftp** no sigue los enlaces simbólicos al hacer transferencias recursivas.

**`pwd`**
: Muestra el directorio de trabajo remoto.

**`quit`**
: Sale de **sftp**.

**`reget [-fpR] remote-path [local-path]`**
: Reanuda la descarga de *remote-path*. Equivale a `get` con el flag `-a`.

**`reput [-fpR] local-path [remote-path]`**
: Reanuda la subida de *local-path*. Equivale a `put` con el flag `-a`.

**`rename oldpath newpath`**
: Renombra el archivo remoto de *oldpath* a *newpath*.

**`rm path`**
: Borra el archivo remoto indicado por *path*.

**`rmdir path`**
: Elimina el directorio remoto indicado por *path*.

**`symlink oldpath newpath`**
: Crea un enlace simbólico de *oldpath* a *newpath*.

**`version`**
: Muestra la versión del protocolo sftp.

**`!command`**
: Ejecuta *command* en la shell local.

**`!`**
: Sale temporalmente a la shell local.

**`?`**
: Sinónimo de `help`.

## VÉASE TAMBIÉN

`ftp(1)`, `ls(1)`, `scp(1)`, `ssh(1)`, `ssh-add(1)`, `ssh-keygen(1)`, `ssh_config(5)`, `glob(7)`, `sftp-server(8)`, `sshd(8)`

T. Ylonen y S. Lehtinen, *SSH File Transfer Protocol*, draft-ietf-secsh-filexfer-00.txt, enero de 2001, material de trabajo en curso.
