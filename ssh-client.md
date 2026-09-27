# SSH(1) — Manual en español

## NOMBRE

**ssh** — cliente de inicio de sesión remoto de OpenSSH

## SINOPSIS

```
ssh [-46AaCfGgKkMNnqsTtVvXxYyZ] [-B bind_interface] [-b bind_address] [-c cipher_spec]
    [-D [bind_address:]port] [-E log_file] [-e escape_char] [-F configfile] [-I pkcs11]
    [-i identity_file] [-J destination] [-L address] [-l login_name] [-m mac_spec]
    [-O ctl_cmd] [-o option] [-P tag] [-p port] [-R address] [-S ctl_path] [-W host:port]
    [-w local_tun[:remote_tun]] destination [command [argument ...]]

ssh [-Q query_option]
```

## DESCRIPCIÓN

**ssh** (cliente SSH) es un programa para iniciar sesión en una máquina remota y ejecutar comandos en ella. Su propósito es proporcionar comunicaciones cifradas y seguras entre dos hosts que no confían entre sí a través de una red insegura. También se pueden reenviar por el canal seguro conexiones X11, puertos TCP arbitrarios y sockets de dominio Unix.

**ssh** se conecta e inicia sesión en el destino indicado, que puede especificarse como `[user@]hostname` o como un URI de la forma `ssh://[user@]hostname[:port]`. El usuario debe demostrar su identidad ante la máquina remota mediante alguno de varios métodos (ver más abajo).

Si se especifica un comando, este se ejecutará en el host remoto en lugar de una shell de inicio de sesión. Se puede indicar una línea de comando completa como *command*, o bien este puede llevar argumentos adicionales. Si se proporcionan, los argumentos se añadirán al comando, separados por espacios, antes de enviarlo al servidor para su ejecución.

Las opciones son las siguientes:

**`-4`**
Obliga a ssh a usar únicamente direcciones IPv4.

**`-6`**
Obliga a ssh a usar únicamente direcciones IPv6.

**`-A`**
Habilita el reenvío de conexiones desde un agente de autenticación como ssh-agent(1). También puede especificarse por host en un archivo de configuración.

El reenvío del agente debe habilitarse con precaución. Los usuarios capaces de saltarse los permisos de archivos en el host remoto (para el socket de dominio Unix del agente) pueden acceder al agente local a través de la conexión reenviada. Un atacante no puede obtener material de claves del agente, pero sí puede realizar operaciones con las claves que le permitan autenticarse usando las identidades cargadas en el agente. Una alternativa más segura puede ser usar un host de salto (ver `-J`).

**`-a`**
Deshabilita el reenvío de la conexión del agente de autenticación.

**`-B bind_interface`**
Se vincula a la dirección de *bind_interface* antes de intentar conectarse al host de destino. Solo es útil en sistemas con más de una dirección.

**`-b bind_address`**
Usa *bind_address* en la máquina local como dirección de origen de la conexión. Solo es útil en sistemas con más de una dirección.

**`-C`**
Solicita la compresión de todos los datos (incluidos stdin, stdout, stderr y los datos de conexiones reenviadas X11, TCP y de dominio Unix). El algoritmo de compresión es el mismo que usa gzip(1). La compresión es conveniente en líneas de módem y otras conexiones lentas, pero solo ralentizará las cosas en redes rápidas. El valor por defecto puede definirse host por host en los archivos de configuración; ver la opción `Compression` en ssh_config(5).

**`-c cipher_spec`**
Selecciona la especificación de cifrado para la sesión. *cipher_spec* es una lista de cifrados separados por comas, en orden de preferencia. Ver la palabra clave `Ciphers` en ssh_config(5) para más información.

**`-D [bind_address:]port`**
Especifica un reenvío de puertos local “dinámico” a nivel de aplicación. Funciona asignando un socket que escucha en *port* del lado local, opcionalmente vinculado a la *bind_address* indicada. Cada vez que se realiza una conexión a este puerto, se reenvía por el canal seguro, y luego se usa el protocolo de aplicación para determinar a dónde conectarse desde la máquina remota. Actualmente se admiten los protocolos SOCKS4 y SOCKS5, y ssh actuará como servidor SOCKS. Solo root puede reenviar puertos privilegiados. Los reenvíos dinámicos también pueden especificarse en el archivo de configuración.

Las direcciones IPv6 se pueden especificar encerrándolas entre corchetes. Solo el superusuario puede reenviar puertos privilegiados. Por defecto, el puerto local se vincula según la configuración de `GatewayPorts`. Sin embargo, se puede usar una *bind_address* explícita para vincular la conexión a una dirección concreta. Una *bind_address* de “localhost” indica que el puerto de escucha se vincula solo para uso local, mientras que una dirección vacía o ‘*’ indica que el puerto debe estar disponible desde todas las interfaces.

**`-E log_file`**
Añade los registros de depuración a *log_file* en lugar de a la salida de error estándar.

**`-e escape_char`**
Define el carácter de escape para sesiones con pty (por defecto: ‘~’). El carácter de escape solo se reconoce al inicio de una línea. El carácter de escape seguido de un punto (‘.’) cierra la conexión; seguido de control-Z suspende la conexión; y seguido de sí mismo envía el carácter de escape una vez. Establecer el carácter en “none” desactiva cualquier escape y hace la sesión totalmente transparente.

**`-F configfile`**
Especifica un archivo de configuración por usuario alternativo. Si se da un archivo de configuración en la línea de comandos, se ignorará el archivo de configuración global del sistema (`/etc/ssh/ssh_config`). El archivo de configuración por usuario por defecto es `~/.ssh/config`. Si se establece en “none”, no se leerá ningún archivo de configuración.

**`-f`**
Solicita que ssh pase a segundo plano justo antes de ejecutar el comando. Es útil si ssh va a pedir contraseñas o frases de paso, pero el usuario lo quiere en segundo plano. Implica `-n`. La forma recomendada de iniciar programas X11 en un sitio remoto es con algo como `ssh -f host xterm`.

Si la opción de configuración `ExitOnForwardFailure` está en “yes”, un cliente iniciado con `-f` esperará a que todos los reenvíos de puertos remotos se hayan establecido correctamente antes de pasar a segundo plano. Consulte la descripción de `ForkAfterAuthentication` en ssh_config(5) para más detalles.

**`-G`**
Hace que ssh imprima su configuración tras evaluar los bloques `Host` y `Match`, y termine.

**`-g`**
Permite que hosts remotos se conecten a puertos locales reenviados. Si se usa en una conexión multiplexada, esta opción debe especificarse en el proceso maestro.

**`-I pkcs11`**
Especifica la biblioteca compartida PKCS#11 que ssh debe usar para comunicarse con un token PKCS#11 que proporciona claves para la autenticación del usuario.

**`-i identity_file`**
Selecciona un archivo desde el cual se lee la identidad (clave privada) para la autenticación por clave pública. También se puede especificar un archivo de clave pública para usar la clave privada correspondiente cargada en ssh-agent(1) cuando el archivo de clave privada no está presente localmente. Los valores por defecto son `~/.ssh/id_rsa`, `~/.ssh/id_ecdsa`, `~/.ssh/id_ecdsa_sk`, `~/.ssh/id_ed25519`, `~/.ssh/id_ed25519_sk` y `~/.ssh/id_mldsa44_ed25519`. Los archivos de identidad también pueden especificarse por host en el archivo de configuración. Es posible tener varias opciones `-i` (y varias identidades especificadas en los archivos de configuración). Si no se han especificado certificados explícitamente mediante la directiva `CertificateFile`, ssh también intentará cargar información de certificado desde el nombre de archivo obtenido al añadir `-cert.pub` a los nombres de los archivos de identidad.

**`-J destination`**
Se conecta al host de destino estableciendo primero una conexión ssh con el host de salto descrito por *destination* y, desde allí, un reenvío TCP hasta el destino final. Se pueden especificar múltiples saltos separados por comas. Las direcciones IPv6 se pueden especificar encerrándolas entre corchetes. Es un atajo para especificar una directiva de configuración `ProxyJump`. Tenga en cuenta que las directivas de configuración pasadas por línea de comandos generalmente se aplican al host de destino y no a los hosts de salto. Use `~/.ssh/config` para especificar la configuración de los hosts de salto.

**`-K`**
Habilita la autenticación basada en GSSAPI y el reenvío (delegación) de credenciales GSSAPI al servidor.

**`-k`**
Deshabilita el reenvío (delegación) de credenciales GSSAPI al servidor.

**`-L [bind_address:]port:host:hostport`**
**`-L [bind_address:]port:remote_socket`**
**`-L local_socket:host:hostport`**
**`-L local_socket:remote_socket`**
Especifica que las conexiones al puerto TCP o socket Unix indicado en el host local (cliente) se reenvíen al host y puerto, o socket Unix, indicados en el lado remoto. Funciona asignando un socket que escucha, ya sea en un puerto TCP del lado local (opcionalmente vinculado a la *bind_address* especificada), o en un socket Unix. Cada vez que se realiza una conexión al puerto o socket local, esta se reenvía por el canal seguro, y se establece una conexión desde la máquina remota hacia *host* en el puerto *hostport*, o hacia el socket Unix *remote_socket*.

Los reenvíos de puertos también pueden especificarse en el archivo de configuración. Solo el superusuario puede reenviar puertos privilegiados. Las direcciones IPv6 se pueden especificar encerrándolas entre corchetes.

Por defecto, el puerto local se vincula según la configuración de `GatewayPorts`. Sin embargo, se puede usar una *bind_address* explícita para vincular la conexión a una dirección concreta. Una *bind_address* de “localhost” indica que el puerto de escucha se vincula solo para uso local, mientras que una dirección vacía o ‘*’ indica que el puerto debe estar disponible desde todas las interfaces.

**`-l login_name`**
Especifica el usuario con el que iniciar sesión en la máquina remota. También puede especificarse por host en el archivo de configuración.

**`-M`**
Pone el cliente ssh en modo “maestro” para compartir conexiones. Varias opciones `-M` ponen a ssh en modo “maestro” pero exigiendo confirmación mediante ssh-askpass(1) antes de cada operación que cambie el estado de multiplexación (p. ej., abrir una nueva sesión). Consulte la descripción de `ControlMaster` en ssh_config(5) para más detalles.

**`-m mac_spec`**
Lista de algoritmos MAC (código de autenticación de mensajes) separados por comas, en orden de preferencia. Ver la palabra clave `MACs` en ssh_config(5) para más información.

**`-N`**
No ejecuta un comando remoto. Es útil cuando solo se quieren reenviar puertos. Consulte la descripción de `SessionType` en ssh_config(5) para más detalles.

**`-n`**
Redirige stdin desde `/dev/null` (en realidad, impide leer de stdin). Debe usarse cuando ssh se ejecuta en segundo plano. Un truco habitual es usarlo para ejecutar programas X11 en una máquina remota. Por ejemplo, `ssh -n shadows.cs.hut.fi emacs &` iniciará un emacs en shadows.cs.hut.fi, y la conexión X11 se reenviará automáticamente por un canal cifrado. El programa ssh se pondrá en segundo plano. (Esto no funciona si ssh necesita pedir una contraseña o frase de paso; ver también la opción `-f`.) Consulte la descripción de `StdinNull` en ssh_config(5) para más detalles.

**`-O ctl_cmd`**
Controla un proceso maestro activo de multiplexación de conexiones. Cuando se especifica la opción `-O`, el argumento *ctl_cmd* se interpreta y se pasa al proceso maestro. Los comandos válidos son:

| Comando | Descripción |
|---|---|
| `check` | Comprueba que el proceso maestro está en ejecución. |
| `conninfo` | Informa sobre la conexión maestra. |
| `channels` | Informa sobre los canales abiertos. |
| `forward` | Solicita reenvíos sin ejecutar comandos. |
| `cancel` | Cancela reenvíos. |
| `proxy` | Se conecta a un maestro de multiplexación en ejecución en modo proxy. |
| `exit` | Solicita al maestro que termine. |
| `stop` | Solicita al maestro que deje de aceptar nuevas solicitudes de multiplexación. |

**`-o option`**
Permite dar opciones en el formato usado en el archivo de configuración. Es útil para especificar opciones que no tienen un flag de línea de comandos propio. Para detalles completos sobre las opciones y sus valores, ver ssh_config(5).

**`-P tag`**
Especifica un nombre de etiqueta que puede usarse para seleccionar configuración en ssh_config(5). Consulte las palabras clave `Tag` y `Match` en ssh_config(5) para más información.

**`-p port`**
Puerto al que conectarse en el host remoto. Puede especificarse por host en el archivo de configuración.

**`-Q query_option`**
Consulta los algoritmos admitidos por una de las siguientes funcionalidades:

| Opción | Descripción |
|---|---|
| `cipher` | cifrados simétricos admitidos |
| `cipher-auth` | cifrados simétricos admitidos que soportan cifrado autenticado |
| `help` | términos de consulta admitidos para usar con el flag `-Q` |
| `mac` | códigos de integridad de mensajes admitidos |
| `kex` | algoritmos de intercambio de claves |
| `key` | tipos de clave |
| `key-ca-sign` | algoritmos de firma de CA válidos para certificados |
| `key-cert` | tipos de clave de certificado |
| `key-plain` | tipos de clave que no son de certificado |
| `key-sig` | todos los tipos de clave y algoritmos de firma |
| `protocol-version` | versiones del protocolo SSH admitidas |
| `sig` | algoritmos de firma admitidos |

Alternativamente, cualquier palabra clave de ssh_config(5) o sshd_config(5) que acepte una lista de algoritmos puede usarse como alias de la *query_option* correspondiente.

**`-q`**
Modo silencioso. Suprime la mayoría de los mensajes de advertencia y diagnóstico.

**`-R [bind_address:]port:host:hostport`**
**`-R [bind_address:]port:local_socket`**
**`-R remote_socket:host:hostport`**
**`-R remote_socket:local_socket`**
**`-R [bind_address:]port`**
Especifica que las conexiones al puerto TCP o socket Unix indicado en el host remoto (servidor) se reenvíen al lado local.

Funciona asignando un socket que escucha en un puerto TCP o en un socket Unix del lado remoto. Cada vez que se realiza una conexión a este puerto o socket Unix, esta se reenvía por el canal seguro, y se establece una conexión desde la máquina local hacia un destino explícito especificado por *host* en el puerto *hostport*, o *local_socket*; o bien, si no se especificó un destino explícito, ssh actuará como proxy SOCKS 4/5 y reenviará las conexiones a los destinos solicitados por el cliente SOCKS remoto.

Los reenvíos de puertos también pueden especificarse en el archivo de configuración. Los puertos privilegiados solo pueden reenviarse cuando se inicia sesión como root en la máquina remota. Las direcciones IPv6 se pueden especificar encerrándolas entre corchetes.

Por defecto, los sockets TCP de escucha en el servidor se vinculan solo a la interfaz de loopback. Esto puede cambiarse especificando una *bind_address*. Una *bind_address* vacía, o la dirección ‘*’, indica que el socket remoto debe escuchar en todas las interfaces. Especificar una *bind_address* remota solo tendrá éxito si la opción `GatewayPorts` del servidor está habilitada (ver sshd_config(5)).

Si el argumento *port* es ‘0’, el puerto de escucha se asignará dinámicamente en el servidor y se informará al cliente en tiempo de ejecución. Si se usa junto con `-O forward`, el puerto asignado se imprimirá en la salida estándar.

**`-S ctl_path`**
Especifica la ubicación de un socket de control para compartir conexiones, o la cadena “none” para desactivar el uso compartido de conexiones. Consulte la descripción de `ControlPath` y `ControlMaster` en ssh_config(5) para más detalles.

**`-s`**
Puede usarse para solicitar la invocación de un subsistema en el sistema remoto. Los subsistemas facilitan el uso de SSH como transporte seguro para otras aplicaciones (p. ej., sftp(1)). El subsistema se especifica como el comando remoto. Consulte la descripción de `SessionType` en ssh_config(5) para más detalles.

**`-T`**
Desactiva la asignación de pseudoterminal.

**`-t`**
Fuerza la asignación de pseudoterminal. Puede usarse para ejecutar programas arbitrarios basados en pantalla en una máquina remota, lo cual puede ser muy útil, p. ej., al implementar servicios de menús. Varias opciones `-t` fuerzan la asignación de tty incluso si ssh no tiene tty local.

**`-V`**
Muestra el número de versión y termina.

**`-v`**
Modo detallado (verbose). Hace que ssh imprima mensajes de depuración sobre su progreso. Es útil para depurar problemas de conexión, autenticación y configuración. Varias opciones `-v` aumentan el nivel de detalle. El máximo es 3.

**`-W host:port`**
Solicita que la entrada y salida estándar del cliente se reenvíen a *host* en *port* a través del canal seguro. Implica `-N`, `-T`, `ExitOnForwardFailure` y `ClearAllForwardings`, aunque estos pueden anularse en el archivo de configuración o con opciones `-o` en la línea de comandos.

**`-w local_tun[:remote_tun]`**
Solicita el reenvío de dispositivos de túnel con los dispositivos tun(4) especificados entre el cliente (*local_tun*) y el servidor (*remote_tun*).

Los dispositivos pueden especificarse por ID numérico o con la palabra clave “any”, que usa el siguiente dispositivo de túnel disponible. Si no se especifica *remote_tun*, por defecto es “any”. Ver también las directivas `Tunnel` y `TunnelDevice` en ssh_config(5).

Si la directiva `Tunnel` no está definida, se establecerá en el modo de túnel por defecto, que es “point-to-point”. Si se desea un modo de reenvío `Tunnel` diferente, debe especificarse antes de `-w`.

**`-X`**
Habilita el reenvío X11. También puede especificarse por host en un archivo de configuración.

El reenvío X11 debe habilitarse con precaución. Los usuarios capaces de saltarse los permisos de archivos en el host remoto (para la base de datos de autorización X del usuario) pueden acceder a la pantalla X11 local a través de la conexión reenviada. Un atacante podría entonces realizar actividades como monitorizar las pulsaciones de teclado.

Por este motivo, el reenvío X11 está sujeto por defecto a las restricciones de la extensión X11 SECURITY. Consulte la opción `ssh -Y` y la directiva `ForwardX11Trusted` en ssh_config(5) para más información.

**`-x`**
Deshabilita el reenvío X11.

**`-Y`**
Habilita el reenvío X11 de confianza. Los reenvíos X11 de confianza no están sujetos a los controles de la extensión X11 SECURITY.

**`-y`**
Envía la información de registro mediante el módulo del sistema syslog(3). Por defecto, esta información se envía a stderr.

**`-Z`**
Lista las claves públicas que se intentarían para autenticarse en el destino especificado, en orden de preferencia, y termina.

ssh puede además obtener datos de configuración de un archivo de configuración por usuario y de un archivo de configuración global del sistema. El formato del archivo y las opciones de configuración se describen en ssh_config(5).

## AUTENTICACIÓN

El cliente SSH de OpenSSH admite el protocolo SSH 2.

Los métodos de autenticación disponibles son: autenticación basada en GSSAPI, autenticación basada en host, autenticación por clave pública, autenticación keyboard-interactive (interactiva por teclado) y autenticación por contraseña. Los métodos se prueban en el orden indicado, aunque `PreferredAuthentications` puede usarse para cambiar el orden por defecto.

**La autenticación basada en host** funciona así: si la máquina desde la que el usuario inicia sesión figura en `/etc/hosts.equiv` o `/etc/shosts.equiv` en la máquina remota, el usuario no es root y los nombres de usuario son iguales en ambos lados; o si existen los archivos `~/.rhosts` o `~/.shosts` en el directorio personal del usuario en la máquina remota y contienen una línea con el nombre de la máquina cliente y el nombre del usuario en esa máquina, el usuario es considerado para el inicio de sesión. Además, el servidor debe poder verificar la clave de host del cliente (ver la descripción de `/etc/ssh/ssh_known_hosts` y `~/.ssh/known_hosts` más abajo) para permitir el inicio de sesión. Este método de autenticación cierra agujeros de seguridad debidos a suplantación de IP, de DNS y de enrutamiento. [Nota para el administrador: `/etc/hosts.equiv`, `~/.rhosts` y el protocolo rlogin/rsh en general son inherentemente inseguros y deberían deshabilitarse si se desea seguridad.]

**La autenticación por clave pública** funciona así: el esquema se basa en criptografía de clave pública, usando criptosistemas donde el cifrado y el descifrado se hacen con claves distintas, y es inviable derivar la clave de descifrado a partir de la de cifrado. La idea es que cada usuario crea un par de claves pública/privada para fines de autenticación. El servidor conoce la clave pública, y solo el usuario conoce la clave privada. ssh implementa el protocolo de autenticación por clave pública automáticamente, usando uno de los algoritmos ECDSA, Ed25519 o RSA.

El archivo `~/.ssh/authorized_keys` lista las claves públicas autorizadas para iniciar sesión. Cuando el usuario inicia sesión, el programa ssh indica al servidor qué par de claves desea usar para autenticarse. El cliente demuestra que tiene acceso a la clave privada y el servidor comprueba que la clave pública correspondiente está autorizada para acceder a la cuenta.

El servidor puede informar al cliente de los errores que impidieron que la autenticación por clave pública tuviera éxito una vez completada la autenticación con otro método. Pueden verse aumentando el `LogLevel` a `DEBUG` o superior (p. ej., usando el flag `-v`).

El usuario crea su par de claves ejecutando ssh-keygen(1). Esto guarda la clave privada en el siguiente archivo del directorio personal del usuario, y la clave pública en el mismo directorio con el sufijo adicional `.pub`:

| Archivo de clave privada | Algoritmo |
|---|---|
| `~/.ssh/id_ecdsa` | ECDSA |
| `~/.ssh/id_ecdsa_sk` | ECDSA alojado en autenticador |
| `~/.ssh/id_ed25519` | Ed25519 |
| `~/.ssh/id_ed25519_sk` | Ed25519 alojado en autenticador |
| `~/.ssh/id_mldsa44_ed25519` | MLDSA44-ED25519 |
| `~/.ssh/id_rsa` | RSA |

Luego el usuario debe copiar la clave pública a `~/.ssh/authorized_keys` en su directorio personal de la máquina remota. El archivo `authorized_keys` equivale al archivo convencional `~/.rhosts` y contiene una clave por línea, aunque las líneas pueden ser muy largas. Después de esto, el usuario puede iniciar sesión sin dar la contraseña.

Existe una variante de la autenticación por clave pública en forma de **autenticación por certificado**: en lugar de un conjunto de claves pública/privada, se usan certificados firmados. Esto tiene la ventaja de que una sola autoridad de certificación de confianza puede usarse en lugar de muchas claves pública/privada. Ver la sección CERTIFICATES de ssh-keygen(1) para más información.

La forma más cómoda de usar la autenticación por clave pública o por certificado puede ser con un agente de autenticación. Ver ssh-agent(1) y (opcionalmente) la directiva `AddKeysToAgent` en ssh_config(5) para más información.

**La autenticación keyboard-interactive** funciona así: el servidor envía un texto de “desafío” arbitrario y solicita una respuesta, posiblemente varias veces. Ejemplos de autenticación keyboard-interactive son BSD Authentication (ver login.conf(5)) y PAM (en algunos sistemas no OpenBSD).

Finalmente, si los otros métodos de autenticación fallan, ssh pide al usuario una **contraseña**. La contraseña se envía al host remoto para su verificación; sin embargo, como todas las comunicaciones están cifradas, alguien que escuche en la red no puede ver la contraseña.

ssh mantiene y comprueba automáticamente una base de datos con la identificación de todos los hosts con los que se ha usado. Las claves de host se almacenan en `~/.ssh/known_hosts` en el directorio personal del usuario. Además, se comprueba automáticamente el archivo `/etc/ssh/ssh_known_hosts` en busca de hosts conocidos. Cualquier host nuevo se añade automáticamente al archivo del usuario. Si la identificación de un host cambia alguna vez, ssh lo advierte y deshabilita la autenticación por contraseña para evitar la suplantación del servidor o ataques man-in-the-middle, que de otro modo podrían usarse para eludir el cifrado. La opción `StrictHostKeyChecking` puede usarse para controlar los inicios de sesión en máquinas cuya clave de host no es conocida o ha cambiado.

Cuando el servidor acepta la identidad del usuario, ejecuta el comando indicado en una sesión no interactiva o, si no se especificó ningún comando, inicia sesión en la máquina y le da al usuario una shell normal como sesión interactiva. Toda la comunicación con el comando o shell remotos se cifrará automáticamente.

Si se solicita una sesión interactiva, ssh por defecto solo pedirá una pseudoterminal (pty) para sesiones interactivas cuando el cliente tenga una. Los flags `-T` y `-t` pueden usarse para anular este comportamiento.

Si se ha asignado una pseudoterminal, el usuario puede usar los caracteres de escape que se indican más abajo.

Si no se ha asignado una pseudoterminal, la sesión es transparente y puede usarse para transferir datos binarios de forma fiable. En la mayoría de los sistemas, establecer el carácter de escape en “none” también hará la sesión transparente aunque se use una tty.

La sesión termina cuando el comando o la shell en la máquina remota finalizan y se han cerrado todas las conexiones X11 y TCP.

## CARACTERES DE ESCAPE

Cuando se ha solicitado una pseudoterminal, ssh admite varias funciones mediante el uso de un carácter de escape.

Se puede enviar una sola tilde como `~~` o haciendo seguir la tilde de un carácter distinto de los descritos a continuación. El carácter de escape siempre debe ir tras un salto de línea para interpretarse como especial. El carácter de escape puede cambiarse en los archivos de configuración con la directiva `EscapeChar` o en la línea de comandos con la opción `-e`.

Los escapes admitidos (suponiendo el ‘~’ por defecto) son:

| Escape | Función |
|---|---|
| `~.` | Desconectar. |
| `~^Z` | Enviar ssh a segundo plano. |
| `~#` | Listar las conexiones reenviadas. |
| `~&` | Enviar ssh a segundo plano al cerrar sesión mientras se espera a que terminen conexiones reenviadas / sesiones X11. |
| `~?` | Mostrar una lista de caracteres de escape. |
| `~B` | Enviar un BREAK al sistema remoto (solo útil si el otro extremo lo admite). |
| `~C` | Abrir línea de comandos (ver abajo). |
| `~I` | Mostrar información sobre la conexión SSH actual. |
| `~R` | Solicitar el cambio de claves (rekeying) de la conexión (solo útil si el otro extremo lo admite). |
| `~V` | Disminuir el nivel de detalle (`LogLevel`) cuando los errores se escriben en stderr. |
| `~v` | Aumentar el nivel de detalle (`LogLevel`) cuando los errores se escriben en stderr. |

**`~C`** — Abre una línea de comandos. Actualmente permite añadir reenvíos de puertos usando las opciones `-L`, `-R` y `-D` (ver arriba). También permite cancelar reenvíos existentes con `-KL[bind_address:]port` para locales, `-KR[bind_address:]port` para remotos y `-KD[bind_address:]port` para reenvíos dinámicos. `!command` permite al usuario ejecutar un comando local si la opción `PermitLocalCommand` está habilitada en ssh_config(5). Hay una ayuda básica disponible con la opción `-h`.

## REENVÍO TCP

El reenvío de conexiones TCP arbitrarias por un canal seguro puede especificarse en la línea de comandos o en un archivo de configuración. Una posible aplicación del reenvío TCP es una conexión segura a un servidor de correo; otra es atravesar cortafuegos.

En el siguiente ejemplo se cifra la comunicación de un cliente IRC, aunque el servidor IRC al que se conecta no admita directamente comunicación cifrada. Funciona así: el usuario se conecta al host remoto con ssh, especificando los puertos que se usarán para reenviar la conexión. Después, es posible iniciar el programa localmente, y ssh cifrará y reenviará la conexión al servidor remoto.

El siguiente ejemplo tuneliza una sesión IRC desde el cliente hasta un servidor IRC en “server.example.com”, uniéndose al canal “#users” con el apodo “pinky”, usando el puerto IRC estándar 6667:

```sh
$ ssh -f -L 6667:localhost:6667 server.example.com sleep 10
$ irc -c '#users' pinky IRC/127.0.0.1
```

La opción `-f` envía ssh a segundo plano, y el comando remoto “sleep 10” se especifica para dar un margen de tiempo (10 segundos, en el ejemplo) para iniciar el programa que va a usar el túnel. Si no se realizan conexiones dentro del tiempo especificado, ssh terminará.

## REENVÍO X11

Si la variable `ForwardX11` está en “yes” (o ver la descripción de las opciones `-X`, `-x` y `-Y` arriba) y el usuario está usando X11 (la variable de entorno `DISPLAY` está definida), la conexión a la pantalla X11 se reenvía automáticamente al lado remoto de modo que cualquier programa X11 iniciado desde la shell (o comando) pasará por el canal cifrado, y la conexión al servidor X real se hará desde la máquina local. El usuario no debe definir `DISPLAY` manualmente. El reenvío de conexiones X11 puede configurarse en la línea de comandos o en archivos de configuración.

El valor de `DISPLAY` definido por ssh apuntará a la máquina servidor, pero con un número de pantalla mayor que cero. Esto es normal y ocurre porque ssh crea un servidor X “proxy” en la máquina servidor para reenviar las conexiones por el canal cifrado.

ssh también configurará automáticamente los datos de Xauthority en la máquina servidor. Para ello, generará una cookie de autorización aleatoria, la almacenará en Xauthority en el servidor y verificará que cualquier conexión reenviada lleve esta cookie, sustituyéndola por la cookie real cuando se abra la conexión. La cookie de autenticación real nunca se envía a la máquina servidor (y ninguna cookie se envía en texto plano).

Si la variable `ForwardAgent` está en “yes” (o ver la descripción de las opciones `-A` y `-a` arriba) y el usuario está usando un agente de autenticación, la conexión con el agente se reenvía automáticamente al lado remoto.

## VERIFICACIÓN DE CLAVES DE HOST

Al conectarse a un servidor por primera vez, se presenta al usuario una huella (fingerprint) de la clave pública del servidor (a menos que se haya deshabilitado la opción `StrictHostKeyChecking`). Las huellas pueden obtenerse con ssh-keygen(1):

```sh
$ ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key
```

Si la huella ya es conocida, se puede comparar y aceptar o rechazar la clave. Si solo están disponibles huellas heredadas (MD5) del servidor, la opción `-E` de ssh-keygen(1) puede usarse para rebajar el algoritmo de huella y que coincida.

Debido a la dificultad de comparar claves de host solo mirando cadenas de huellas, también se admite la comparación visual de claves de host mediante *random art*. Al establecer la opción `VisualHostKey` en “yes”, se muestra un pequeño gráfico ASCII en cada inicio de sesión en un servidor, sea la sesión interactiva o no. Al aprender el patrón que produce un servidor conocido, el usuario puede darse cuenta fácilmente de que la clave de host ha cambiado cuando aparece un patrón completamente distinto. Sin embargo, como estos patrones no son inequívocos, un patrón que se parezca al recordado solo da una buena probabilidad de que la clave de host sea la misma, no una prueba garantizada.

Para obtener un listado de las huellas junto con su random art para todos los hosts conocidos, se puede usar la siguiente línea de comandos:

```sh
$ ssh-keygen -lv -f ~/.ssh/known_hosts
```

Si la huella es desconocida, hay un método alternativo de verificación: huellas SSH verificadas por DNS. Se añade un registro de recurso (RR) adicional, SSHFP, a un archivo de zona, y el cliente que se conecta puede comparar la huella con la de la clave presentada.

En este ejemplo, conectamos un cliente a un servidor, “host.example.com”. Primero deben añadirse los registros SSHFP al archivo de zona de host.example.com:

```sh
$ ssh-keygen -r host.example.com.
```

Las líneas de salida deben añadirse al archivo de zona. Para comprobar que la zona responde a consultas de huellas:

```sh
$ dig -t SSHFP host.example.com
```

Finalmente, el cliente se conecta:

```sh
$ ssh -o "VerifyHostKeyDNS ask" host.example.com
[...]
Matching host key fingerprint found in DNS.
Are you sure you want to continue connecting (yes/no)?
```

Ver la opción `VerifyHostKeyDNS` en ssh_config(5) para más información.

## REDES PRIVADAS VIRTUALES BASADAS EN SSH

ssh incluye soporte para tunelización de redes privadas virtuales (VPN) usando el pseudodispositivo de red tun(4), lo que permite unir dos redes de forma segura. La opción de configuración `PermitTunnel` de sshd_config(5) controla si el servidor lo admite y a qué nivel (tráfico de capa 2 o 3).

El siguiente ejemplo conectaría la red cliente 10.0.50.0/24 con la red remota 10.0.99.0/24 usando una conexión punto a punto de 10.1.1.1 a 10.1.1.2, siempre que el servidor SSH que se ejecuta en la pasarela hacia la red remota, en 192.168.1.15, lo permita.

En el cliente:

```sh
# ssh -f -w 0:1 192.168.1.15 true
# ifconfig tun0 10.1.1.1 10.1.1.2 netmask 255.255.255.252
# route add 10.0.99.0/24 10.1.1.2
```

En el servidor:

```sh
# ifconfig tun1 10.1.1.2 10.1.1.1 netmask 255.255.255.252
# route add 10.0.50.0/24 10.1.1.1
```

El acceso de los clientes puede ajustarse con más precisión mediante el archivo `/root/.ssh/authorized_keys` (ver abajo) y la opción de servidor `PermitRootLogin`. La siguiente entrada permitiría conexiones en el dispositivo tun(4) 1 del usuario “jane” y en el dispositivo tun 2 del usuario “john”, si `PermitRootLogin` está en “forced-commands-only”:

```
tunnel="1",command="sh /etc/netstart tun1" ssh-rsa ... jane
tunnel="2",command="sh /etc/netstart tun2" ssh-rsa ... john
```

Dado que una configuración basada en SSH conlleva una sobrecarga considerable, puede ser más adecuada para configuraciones temporales, como VPN inalámbricas. Las VPN más permanentes se implementan mejor con herramientas como ipsecctl(8) e isakmpd(8).

## ENTORNO

ssh normalmente define las siguientes variables de entorno:

**`DISPLAY`**
La variable `DISPLAY` indica la ubicación del servidor X11. ssh la define automáticamente con un valor de la forma “hostname:n”, donde “hostname” indica el host donde se ejecuta la shell y ‘n’ es un entero ≥ 1. ssh usa este valor especial para reenviar conexiones X11 por el canal seguro. Normalmente el usuario no debe definir `DISPLAY` explícitamente, ya que eso haría insegura la conexión X11 (y obligaría al usuario a copiar manualmente las cookies de autorización necesarias).

**`HOME`**
Se define con la ruta del directorio personal del usuario.

**`LOGNAME`**
Sinónimo de `USER`; se define por compatibilidad con sistemas que usan esta variable.

**`MAIL`**
Se define con la ruta del buzón de correo del usuario.

**`PATH`**
Se define con el `PATH` por defecto, tal como se especificó al compilar ssh.

**`SSH_ASKPASS`**
Si ssh necesita una frase de paso, la leerá desde la terminal actual si se ejecutó desde una terminal. Si ssh no tiene una terminal asociada pero `DISPLAY` y `SSH_ASKPASS` están definidas, ejecutará el programa especificado por `SSH_ASKPASS` y abrirá una ventana X11 para leer la frase de paso. Esto es especialmente útil al llamar a ssh desde un `.xsession` o un script relacionado. (Tenga en cuenta que en algunas máquinas puede ser necesario redirigir la entrada desde `/dev/null` para que funcione.)

**`SSH_ASKPASS_REQUIRE`**
Permite un mayor control sobre el uso de un programa askpass. Si esta variable está en “never”, ssh nunca intentará usar uno. Si está en “prefer”, ssh preferirá usar el programa askpass en lugar de la TTY al solicitar contraseñas. Por último, si está en “force”, se usará el programa askpass para toda entrada de frase de paso, independientemente de si `DISPLAY` está definida.

**`SSH_AUTH_SOCK`**
Identifica la ruta de un socket de dominio Unix usado para comunicarse con el agente.

**`SSH_CONNECTION`**
Identifica los extremos cliente y servidor de la conexión. La variable contiene cuatro valores separados por espacios: dirección IP del cliente, número de puerto del cliente, dirección IP del servidor y número de puerto del servidor.

**`SSH_ORIGINAL_COMMAND`**
Esta variable contiene la línea de comando original si se ejecuta un comando forzado. Puede usarse para extraer los argumentos originales.

**`SSH_TTY`**
Se define con el nombre de la tty (ruta al dispositivo) asociada a la shell o comando actual. Si la sesión actual no tiene tty, esta variable no se define.

**`SSH_TUNNEL`**
La define opcionalmente sshd(8) para contener los nombres de las interfaces asignadas si el cliente solicitó reenvío de túnel.

**`SSH_USER_AUTH`**
La define opcionalmente sshd(8); esta variable puede contener la ruta a un archivo que lista los métodos de autenticación usados con éxito al establecer la sesión, incluidas las claves públicas utilizadas.

**`TZ`**
Esta variable se define para indicar la zona horaria actual si estaba definida cuando se inició el demonio (es decir, el demonio pasa el valor a las nuevas conexiones).

**`USER`**
Se define con el nombre del usuario que inicia sesión.

Además, ssh lee `~/.ssh/environment` y añade al entorno las líneas con el formato “VARNAME=value” si el archivo existe y los usuarios tienen permitido modificar su entorno. Para más información, ver la opción `PermitUserEnvironment` en sshd_config(5).

## ARCHIVOS

**`~/.rhosts`**
Este archivo se usa para la autenticación basada en host (ver arriba). En algunas máquinas puede ser necesario que sea legible por todos si el directorio personal del usuario está en una partición NFS, porque sshd(8) lo lee como root. Además, este archivo debe pertenecer al usuario y no debe tener permisos de escritura para nadie más. El permiso recomendado en la mayoría de las máquinas es lectura/escritura para el usuario y sin acceso para los demás.

**`~/.shosts`**
Este archivo se usa exactamente igual que `.rhosts`, pero permite la autenticación basada en host sin permitir el inicio de sesión con rlogin/rsh.

**`~/.ssh/`**
Este directorio es la ubicación por defecto de toda la información de configuración y autenticación específica del usuario. No hay un requisito general de mantener en secreto todo el contenido de este directorio, pero los permisos recomendados son lectura/escritura/ejecución para el usuario y sin acceso para los demás.

**`~/.ssh/authorized_keys`**
Lista las claves públicas (ECDSA, Ed25519, RSA) que pueden usarse para iniciar sesión como este usuario. El formato de este archivo se describe en la página de manual de sshd(8). Este archivo no es muy sensible, pero los permisos recomendados son lectura/escritura para el usuario y sin acceso para los demás.

**`~/.ssh/config`**
Este es el archivo de configuración por usuario. El formato del archivo y las opciones de configuración se describen en ssh_config(5). Debido al potencial de abuso, este archivo debe tener permisos estrictos: lectura/escritura para el usuario y sin permiso de escritura para los demás.

**`~/.ssh/environment`**
Contiene definiciones adicionales de variables de entorno; ver ENTORNO, arriba.

**`~/.ssh/id_ecdsa`**
**`~/.ssh/id_ecdsa_sk`**
**`~/.ssh/id_ed25519`**
**`~/.ssh/id_ed25519_sk`**
**`~/.ssh/id_mldsa44_ed25519`**
**`~/.ssh/id_rsa`**
Contienen la clave privada para la autenticación. Estos archivos contienen datos sensibles y deben ser legibles por el usuario, pero no accesibles para los demás (lectura/escritura/ejecución). ssh simplemente ignorará un archivo de clave privada si es accesible por otros. Es posible especificar una frase de paso al generar la clave, que se usará para cifrar la parte sensible de este archivo con AES-128.

**`~/.ssh/id_ecdsa.pub`**
**`~/.ssh/id_ecdsa_sk.pub`**
**`~/.ssh/id_ed25519.pub`**
**`~/.ssh/id_ed25519_sk.pub`**
**`~/.ssh/id_mldsa44_ed25519.pub`**
**`~/.ssh/id_rsa.pub`**
Contienen la clave pública para la autenticación. Estos archivos no son sensibles y pueden (aunque no es necesario) ser legibles por cualquiera.

**`~/.ssh/known_hosts`**
Contiene una lista de claves de host de todos los hosts en los que el usuario ha iniciado sesión que no estén ya en la lista global del sistema de claves de host conocidas. Ver sshd(8) para más detalles sobre el formato de este archivo.

**`~/.ssh/rc`**
Los comandos de este archivo son ejecutados por ssh cuando el usuario inicia sesión, justo antes de que se inicie la shell (o comando) del usuario. Ver la página de manual de sshd(8) para más información.

**`/etc/hosts.equiv`**
Este archivo es para la autenticación basada en host (ver arriba). Solo debe poder escribirlo root.

**`/etc/shosts.equiv`**
Este archivo se usa exactamente igual que `hosts.equiv`, pero permite la autenticación basada en host sin permitir el inicio de sesión con rlogin/rsh.

**`/etc/ssh/ssh_config`**
Archivo de configuración global del sistema. El formato del archivo y las opciones de configuración se describen en ssh_config(5).

**`/etc/ssh/ssh_host_ecdsa_key`**
**`/etc/ssh/ssh_host_ed25519_key`**
**`/etc/ssh/ssh_host_mldsa44_ed25519_key`**
**`/etc/ssh/ssh_host_rsa_key`**
Estos archivos contienen las partes privadas de las claves de host y se usan para la autenticación basada en host.

**`/etc/ssh/ssh_known_hosts`**
Lista global del sistema de claves de host conocidas. Este archivo debe prepararlo el administrador del sistema para que contenga las claves públicas de host de todas las máquinas de la organización. Debe ser legible por todos. Ver sshd(8) para más detalles sobre el formato de este archivo.

**`/etc/ssh/sshrc`**
Los comandos de este archivo son ejecutados por ssh cuando el usuario inicia sesión, justo antes de que se inicie la shell (o comando) del usuario. Ver la página de manual de sshd(8) para más información.

## ESTADO DE SALIDA

ssh termina con el estado de salida del comando remoto, o con 255 si se produjo un error.

## VER TAMBIÉN

scp(1), sftp(1), ssh-add(1), ssh-agent(1), ssh-keygen(1), ssh-keyscan(1), tun(4), ssh_config(5), ssh-keysign(8), sshd(8)

## ESTÁNDARES

- S. Lehtinen y C. Lonvick, *The Secure Shell (SSH) Protocol Assigned Numbers*, RFC 4250, enero de 2006.
- T. Ylonen y C. Lonvick, *The Secure Shell (SSH) Protocol Architecture*, RFC 4251, enero de 2006.
- T. Ylonen y C. Lonvick, *The Secure Shell (SSH) Authentication Protocol*, RFC 4252, enero de 2006.
- T. Ylonen y C. Lonvick, *The Secure Shell (SSH) Transport Layer Protocol*, RFC 4253, enero de 2006.
- T. Ylonen y C. Lonvick, *The Secure Shell (SSH) Connection Protocol*, RFC 4254, enero de 2006.
- J. Schlyter y W. Griffin, *Using DNS to Securely Publish Secure Shell (SSH) Key Fingerprints*, RFC 4255, enero de 2006.
- F. Cusack y M. Forssen, *Generic Message Exchange Authentication for the Secure Shell Protocol (SSH)*, RFC 4256, enero de 2006.
- J. Galbraith y P. Remaker, *The Secure Shell (SSH) Session Channel Break Extension*, RFC 4335, enero de 2006.
- M. Bellare, T. Kohno y C. Namprempre, *The Secure Shell (SSH) Transport Layer Encryption Modes*, RFC 4344, enero de 2006.
- B. Harris, *Improved Arcfour Modes for the Secure Shell (SSH) Transport Layer Protocol*, RFC 4345, enero de 2006.
- M. Friedl, N. Provos y W. Simpson, *Diffie-Hellman Group Exchange for the Secure Shell (SSH) Transport Layer Protocol*, RFC 4419, marzo de 2006.
- J. Galbraith y R. Thayer, *The Secure Shell (SSH) Public Key File Format*, RFC 4716, noviembre de 2006.
- D. Stebila y J. Green, *Elliptic Curve Algorithm Integration in the Secure Shell Transport Layer*, RFC 5656, diciembre de 2009.
- A. Perrig y D. Song, *Hash Visualization: a New Technique to improve Real-World Security*, 1999, International Workshop on Cryptographic Techniques and E-Commerce (CrypTEC '99).

## AUTORES

OpenSSH es un derivado de la versión original y libre ssh 1.2.12 de Tatu Ylonen. Aaron Campbell, Bob Beck, Markus Friedl, Niels Provos, Theo de Raadt y Dug Song corrigieron muchos errores, volvieron a añadir funcionalidades más recientes y crearon OpenSSH. Markus Friedl contribuyó con el soporte para las versiones 1.5 y 2.0 del protocolo SSH.
