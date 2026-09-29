# SCP(1)

## NOMBRE

**scp** — copia segura de archivos de OpenSSH

## SINOPSIS

```
scp [-346ABCOpqRrTv] [-c cipher] [-D sftp_server_path] [-F ssh_config] [-i identity_file] [-J destination] [-l limit] [-o ssh_option] [-P port] [-S program] [-X sftp_option] source ... target
```

## DESCRIPCIÓN

**scp** copia archivos entre hosts de una red.

**scp** usa el protocolo SFTP sobre una conexión `ssh(1)` para la transferencia de datos, y usa la misma autenticación y ofrece la misma seguridad que una sesión de inicio (*login*).

**scp** pedirá contraseñas o frases de contraseña si son necesarias para la autenticación.

El origen (*source*) y el destino (*target*) pueden especificarse como una ruta local, un host remoto con ruta opcional de la forma `[user@]host:[path]`, o una URI de la forma `scp://[user@]host[:port][/path]`. Los nombres de archivo locales pueden hacerse explícitos usando rutas absolutas o relativas, para evitar que **scp** trate los nombres de archivo que contienen `:` como especificadores de host.

Al copiar entre dos hosts remotos, si se usa el formato URI, no se puede especificar un puerto en el destino si se usa la opción `-R`.

Las opciones son las siguientes:

**`-3`**
: Las copias entre dos hosts remotos se transfieren a través del host local. Este es el modo por defecto; consulta también la opción `-R` para copiar datos directamente entre dos hosts remotos. Ten en cuenta que al usar el protocolo SCP heredado (mediante el flag `-O`), esta opción selecciona el modo por lotes (*batch*) para el segundo host, ya que **scp** no puede pedir contraseñas o frases de contraseña para ambos hosts.

**`-4`**
: Obliga a **scp** a usar solo direcciones IPv4.

**`-6`**
: Obliga a **scp** a usar solo direcciones IPv6.

**`-A`**
: Permite el reenvío de `ssh-agent(1)` al sistema remoto. Por defecto no se reenvía el agente de autenticación.

**`-B`**
: Selecciona el modo por lotes (evita que se pidan contraseñas o frases de contraseña).

**`-C`**
: Activa la compresión. Pasa el flag `-C` a `ssh(1)` para activar la compresión.

**`-c cipher`**
: Selecciona el cifrado usado para cifrar la transferencia de datos. Esta opción se pasa directamente a `ssh(1)`.

**`-D sftp_server_path`**
: Se conecta directamente a un programa servidor SFTP local en lugar de a uno remoto mediante `ssh(1)`. Puede ser útil para depurar el cliente y el servidor.

**`-F ssh_config`**
: Especifica un archivo de configuración de usuario alternativo para ssh. Esta opción se pasa directamente a `ssh(1)`.

**`-i identity_file`**
: Selecciona el archivo desde el que se lee la identidad (clave privada) para la autenticación por clave pública. Esta opción se pasa directamente a `ssh(1)`.

**`-J destination`**
: Se conecta al host objetivo estableciendo primero una conexión scp con el host de salto (*jump host*) descrito por *destination* y, desde ahí, estableciendo un reenvío TCP hasta el destino final. Se pueden indicar varios saltos separados por comas. Es un atajo para especificar la directiva de configuración `ProxyJump`. Esta opción se pasa directamente a `ssh(1)`.

**`-l limit`**
: Limita el ancho de banda usado, especificado en Kbit/s.

**`-O`**
: Usa el protocolo SCP heredado para las transferencias en lugar del protocolo SFTP. Forzar el uso del protocolo SCP puede ser necesario con servidores que no implementan SFTP, por retrocompatibilidad con ciertos patrones de comodines en nombres de archivo, y para expandir rutas con prefijo `~` en servidores SFTP antiguos.

**`-o ssh_option`**
: Puede usarse para pasar opciones a ssh en el formato usado en `ssh_config(5)`. Es útil para especificar opciones que no tienen un flag propio en **scp**. Para todos los detalles de las opciones listadas a continuación y sus valores posibles, consulta `ssh_config(5)`.

  `AddKeysToAgent`, `AddressFamily`, `BatchMode`, `BindAddress`, `BindInterface`, `CASignatureAlgorithms`, `CanonicalDomains`, `CanonicalizeFallbackLocal`, `CanonicalizeHostname`, `CanonicalizeMaxDots`, `CanonicalizePermittedCNAMEs`, `CertificateFile`, `ChannelTimeout`, `CheckHostIP`, `Ciphers`, `ClearAllForwardings`, `Compression`, `ConnectTimeout`, `ConnectionAttempts`, `ControlMaster`, `ControlPath`, `ControlPersist`, `DynamicForward`, `EnableEscapeCommandline`, `EnableSSHKeysign`, `EscapeChar`, `ExitOnForwardFailure`, `FingerprintHash`, `ForkAfterAuthentication`, `ForwardAgent`, `ForwardX11`, `ForwardX11Timeout`, `ForwardX11Trusted`, `GSSAPIAuthentication`, `GSSAPIDelegateCredentials`, `GatewayPorts`, `GlobalKnownHostsFile`, `HashKnownHosts`, `Host`, `HostKeyAlgorithms`, `HostKeyAlias`, `HostbasedAcceptedAlgorithms`, `HostbasedAuthentication`, `Hostname`, `IPQoS`, `IdentitiesOnly`, `IdentityAgent`, `IdentityFile`, `IgnoreUnknown`, `Include`, `KbdInteractiveAuthentication`, `KbdInteractiveDevices`, `KexAlgorithms`, `KnownHostsCommand`, `LocalCommand`, `LocalForward`, `LogLevel`, `LogVerbose`, `MACs`, `NoHostAuthenticationForLocalhost`, `NumberOfPasswordPrompts`, `ObscureKeystrokeTiming`, `PKCS11Provider`, `PasswordAuthentication`, `PermitLocalCommand`, `PermitRemoteOpen`, `Port`, `PreferredAuthentications`, `ProxyCommand`, `ProxyJump`, `ProxyUseFdpass`, `PubkeyAcceptedAlgorithms`, `PubkeyAuthentication`, `RekeyLimit`, `RemoteCommand`, `RemoteForward`, `RequestTTY`, `RequiredRSASize`, `RevokedHostKeys`, `SecurityKeyProvider`, `SendEnv`, `ServerAliveCountMax`, `ServerAliveInterval`, `SessionType`, `SetEnv`, `StdinNull`, `StreamLocalBindMask`, `StreamLocalBindUnlink`, `StrictHostKeyChecking`, `SyslogFacility`, `TCPKeepAlive`, `Tag`, `Tunnel`, `TunnelDevice`, `UpdateHostKeys`, `User`, `UserKnownHostsFile`, `VerifyHostKeyDNS`, `VisualHostKey`, `XAuthLocation`

**`-P port`**
: Especifica el puerto al que conectarse en el host remoto. Esta opción se escribe con `P` mayúscula porque `-p` ya está reservado para preservar las fechas y los bits de modo del archivo.

**`-p`**
: Preserva las fechas de modificación, las fechas de acceso y los bits de modo del archivo de origen.

**`-q`**
: Modo silencioso: desactiva el indicador de progreso, así como los mensajes de advertencia y diagnóstico de `ssh(1)`.

**`-R`**
: Por defecto, las copias entre dos hosts remotos se transfieren a través del host local. Esta opción, en cambio, copia entre los dos hosts remotos conectándose al host de origen y ejecutando **scp** allí. Esto requiere que el **scp** que se ejecuta en el host de origen pueda autenticarse en el host de destino sin pedir contraseña.

**`-r`**
: Copia directorios completos de forma recursiva. Ten en cuenta que **scp** sigue los enlaces simbólicos que encuentra al recorrer el árbol.

**`-S program`**
: Nombre del programa a usar para la conexión cifrada. El programa debe entender las opciones de `ssh(1)`.

**`-T`**
: Desactiva la comprobación estricta de nombres de archivo. Por defecto, al copiar archivos de un host remoto a un directorio local, **scp** comprueba que los nombres recibidos coinciden con los solicitados en la línea de comandos, para evitar que el extremo remoto envíe archivos inesperados o no deseados. Debido a las diferencias en cómo los distintos sistemas operativos y shells interpretan los comodines, estas comprobaciones pueden hacer que se rechacen archivos que sí se querían. Esta opción desactiva esas comprobaciones, a costa de confiar plenamente en que el servidor no enviará nombres de archivo inesperados.

**`-v`**
: Modo detallado. Hace que **scp** y `ssh(1)` impriman mensajes de depuración sobre su progreso. Es útil para depurar problemas de conexión, autenticación y configuración.

**`-X sftp_option`**
: Especifica una opción que controla aspectos del comportamiento del protocolo SFTP. Las opciones válidas son:

  **`nrequests=value`**
  : Controla cuántas peticiones SFTP de lectura o escritura concurrentes puede haber en curso en cualquier momento durante una descarga o subida. Por defecto pueden estar activas 64 peticiones a la vez.

  **`buffer=value`**
  : Controla el tamaño máximo del búfer para una única operación SFTP de lectura/escritura durante una descarga o subida. Por defecto se usa un búfer de 32 KB.

## ESTADO DE SALIDA

La utilidad **scp** termina con 0 si tiene éxito y con >0 si ocurre un error.

## VÉASE TAMBIÉN

`sftp(1)`, `ssh(1)`, `ssh-add(1)`, `ssh-agent(1)`, `ssh-keygen(1)`, `ssh_config(5)`, `sftp-server(8)`, `sshd(8)`

## HISTORIA

**scp** se basa en el programa rcp del código fuente BSD de los Regents de la Universidad de California.

Desde OpenSSH 9.0, **scp** usa por defecto el protocolo SFTP para las transferencias.

## AUTORES

Timo Rinne <tri@iki.fi>
Tatu Ylonen <ylo@cs.hut.fi>

## ADVERTENCIAS

El protocolo SCP heredado (seleccionado con el flag `-O`) requiere ejecutar la shell del usuario remoto para hacer la coincidencia de patrones `glob(3)`. Esto obliga a entrecomillar con cuidado cualquier carácter con significado especial para la shell remota, como las comillas.
