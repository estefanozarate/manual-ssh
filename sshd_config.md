# SSHD_CONFIG(5)

## NOMBRE

**sshd_config** — archivo de configuración del demonio OpenSSH

## DESCRIPCIÓN

`sshd(8)` lee los datos de configuración de `/etc/ssh/sshd_config` (o del archivo indicado con `-f` en la línea de comandos). El archivo contiene pares palabra clave-argumento, uno por línea. Salvo que se indique lo contrario, para cada palabra clave se usará el primer valor obtenido. Las líneas que empiezan por `#` y las líneas vacías se interpretan como comentarios. Los argumentos pueden ir opcionalmente entre comillas dobles (`"`) para representar argumentos que contienen espacios.

Las palabras clave posibles y su significado son los siguientes (las palabras clave no distinguen mayúsculas de minúsculas, pero los argumentos sí):

### AcceptEnv

Especifica qué variables de entorno enviadas por el cliente se copiarán en el `environ(7)` de la sesión. Consulta `SendEnv` y `SetEnv` en `ssh_config(5)` para ver cómo configurar el cliente. La variable de entorno `TERM` siempre se acepta cuando el cliente solicita una pseudo-terminal, ya que el protocolo la requiere. Las variables se indican por nombre, que puede contener los comodines `*` y `?`. Varias variables pueden separarse con espacios o repartirse en varias directivas `AcceptEnv`. Ten en cuenta que algunas variables de entorno podrían usarse para eludir entornos de usuario restringidos; por ello, hay que tener cuidado al usar esta directiva. Por defecto no se acepta ninguna variable de entorno.

### AddressFamily

Especifica qué familia de direcciones debe usar `sshd(8)`. Los argumentos válidos son `any` (por defecto), `inet` (solo IPv4) o `inet6` (solo IPv6).

### AgentSocketPath

Especifica la ruta del sistema de archivos usada para los sockets de `ssh-agent(1)` reenviados. Los sockets pueden crearse en una ubicación compartida o en un directorio específico del usuario. Los directorios específicos del usuario se indican anteponiendo `user:` a la ruta. `sshd(8)` se asegurará de que el directorio exista y creará el socket de escucha directamente en él. Las rutas relativas de directorios específicos del usuario se crearán relativas al directorio `$HOME` del usuario.

Los directorios compartidos pueden indicarse anteponiendo `shared:` a una ruta absoluta. En ese caso se creará un subdirectorio temporal bajo el directorio especificado y el socket de escucha del agente se creará dentro de él.

`AgentSocketPath` acepta los tokens descritos en la sección TOKENS. La especificación de ruta por defecto es `user:.ssh/agent`.

### AllowAgentForwarding

Especifica si se permite el reenvío de `ssh-agent(1)`. El valor por defecto es `yes`. Desactivar el reenvío del agente no mejora la seguridad salvo que también se niegue a los usuarios el acceso a una shell, ya que siempre pueden instalar sus propios mecanismos de reenvío.

### AllowGroups

Esta palabra clave puede ir seguida de una lista de patrones de nombres de grupo, separados por espacios. Si se especifica, solo se permite iniciar sesión a los usuarios cuyo grupo principal o lista de grupos suplementarios coincida con alguno de los patrones. Solo son válidos los nombres de grupo; no se reconoce un ID de grupo numérico. Por defecto se permite iniciar sesión a todos los grupos. `AllowGroups` no se consulta para los grupos que coinciden con `DenyGroups`.

Consulta PATTERNS en `ssh_config(5)` para más información sobre patrones. Esta palabra clave puede aparecer varias veces en `sshd_config`; cada aparición se añade a la lista.

### AllowStreamLocalForwarding

Especifica si se permite el reenvío StreamLocal (sockets de dominio Unix). Las opciones disponibles son `yes` (por defecto) o `all` para permitirlo, `no` para impedir todo reenvío StreamLocal, `local` para permitir solo el reenvío local (desde la perspectiva de `ssh(1)`) o `remote` para permitir solo el reenvío remoto. Desactivar el reenvío StreamLocal no mejora la seguridad salvo que también se niegue a los usuarios el acceso a una shell, ya que siempre pueden instalar sus propios mecanismos de reenvío.

### AllowTcpForwarding

Especifica si se permite el reenvío TCP. Las opciones disponibles son `yes` (por defecto) o `all` para permitirlo, `no` para impedir todo reenvío TCP, `local` para permitir solo el reenvío local (desde la perspectiva de `ssh(1)`) o `remote` para permitir solo el reenvío remoto. Desactivar el reenvío TCP no mejora la seguridad salvo que también se niegue a los usuarios el acceso a una shell, ya que siempre pueden instalar sus propios mecanismos de reenvío.

### AllowUsers

Esta palabra clave puede ir seguida de una lista de patrones de nombres de usuario, separados por espacios. Si se especifica, solo se permite iniciar sesión a los nombres de usuario que coincidan con alguno de los patrones. Solo son válidos los nombres de usuario; no se reconoce un ID de usuario numérico. Por defecto se permite iniciar sesión a todos los usuarios. Si el patrón tiene la forma `USER@HOST`, `USER` y `HOST` se comprueban por separado, restringiendo los inicios de sesión a usuarios concretos desde hosts concretos. Los criterios `HOST` pueden contener además direcciones a comparar en formato CIDR dirección/longitud de máscara. `AllowUsers` no se consulta para los usuarios que coinciden con `DenyUsers`.

Consulta PATTERNS en `ssh_config(5)` para más información sobre patrones. Esta palabra clave puede aparecer varias veces en `sshd_config`; cada aparición se añade a la lista.

### AuthenticationMethods

Especifica los métodos de autenticación que deben completarse con éxito para conceder acceso a un usuario. Esta opción debe ir seguida de una o más listas de nombres de métodos de autenticación separados por comas, o de la cadena única `any` para indicar el comportamiento por defecto de aceptar cualquier método de autenticación individual. Si se cambia el valor por defecto, una autenticación exitosa requiere completar todos los métodos de al menos una de estas listas.

Por ejemplo, `"publickey,password publickey,keyboard-interactive"` exigiría al usuario completar la autenticación por clave pública, seguida de autenticación por contraseña o *keyboard-interactive*. En cada etapa solo se ofrecen los métodos que son los siguientes en una o más listas, así que en este ejemplo no sería posible intentar la autenticación por contraseña o *keyboard-interactive* antes de la de clave pública.

Para la autenticación *keyboard-interactive* también es posible restringir la autenticación a un dispositivo concreto añadiendo dos puntos seguidos del identificador de dispositivo `bsdauth` o `pam`, según la configuración del servidor. Por ejemplo, `"keyboard-interactive:bsdauth"` restringiría la autenticación *keyboard-interactive* al dispositivo `bsdauth`.

Si el método `publickey` aparece más de una vez, `sshd(8)` verifica que las claves ya usadas con éxito no se reutilicen en las autenticaciones siguientes. Por ejemplo, `"publickey,publickey"` exige autenticarse con éxito usando dos claves públicas distintas.

Cada método de autenticación listado también debe estar activado explícitamente en la configuración.

Los métodos de autenticación disponibles son: `"gssapi-with-mic"`, `"hostbased"`, `"keyboard-interactive"`, `"none"` (usado para acceder a cuentas sin contraseña cuando `PermitEmptyPasswords` está activado), `"password"` y `"publickey"`.

### AuthorizedKeysCommand

Especifica un programa para buscar las claves públicas del usuario. El programa debe pertenecer a root, no ser escribible por el grupo ni por otros, y especificarse con una ruta absoluta. Los argumentos de `AuthorizedKeysCommand` aceptan los tokens descritos en la sección TOKENS. Si no se indican argumentos, se usa el nombre del usuario objetivo.

El programa debe producir por la salida estándar cero o más líneas en formato `authorized_keys` (consulta AUTHORIZED_KEYS en `sshd(8)`). `AuthorizedKeysCommand` se prueba después de los archivos `AuthorizedKeysFile` habituales y no se ejecutará si allí se encuentra una clave coincidente. Por defecto no se ejecuta ningún `AuthorizedKeysCommand`. Este comando solo se ejecuta para usuarios válidos.

### AuthorizedKeysCommandUser

Especifica el usuario bajo cuya cuenta se ejecuta `AuthorizedKeysCommand`. Se recomienda usar un usuario dedicado que no tenga ningún otro papel en el host más que ejecutar comandos de claves autorizadas. Si se especifica `AuthorizedKeysCommand` pero no `AuthorizedKeysCommandUser`, `sshd(8)` se negará a arrancar.

### AuthorizedKeysFile

Especifica el archivo que contiene las claves públicas usadas para la autenticación de usuarios. El formato se describe en la sección AUTHORIZED_KEYS FILE FORMAT de `sshd(8)`. Los argumentos de `AuthorizedKeysFile` pueden incluir comodines y aceptan los tokens descritos en la sección TOKENS. Tras la expansión, `AuthorizedKeysFile` se toma como una ruta absoluta o relativa al directorio home del usuario. Se pueden indicar varios archivos separados por espacios. Como alternativa, esta opción puede establecerse a `none` para no comprobar claves de usuario en archivos. El valor por defecto es `".ssh/authorized_keys .ssh/authorized_keys2"`. Estos archivos solo se comprueban para usuarios válidos.

### AuthorizedPrincipalsCommand

Especifica un programa para generar la lista de principales de certificado permitidos, al modo de `AuthorizedPrincipalsFile`. El programa debe pertenecer a root, no ser escribible por el grupo ni por otros, y especificarse con una ruta absoluta. Los argumentos de `AuthorizedPrincipalsCommand` aceptan los tokens descritos en la sección TOKENS. Si no se indican argumentos, se usa el nombre del usuario objetivo.

El programa debe producir por la salida estándar cero o más líneas en formato `AuthorizedPrincipalsFile`. Si se especifica `AuthorizedPrincipalsCommand` o `AuthorizedPrincipalsFile`, los certificados ofrecidos por el cliente para autenticarse deben contener un principal que esté en la lista. Por defecto no se ejecuta ningún `AuthorizedPrincipalsCommand`. Este comando solo se ejecuta para usuarios válidos.

### AuthorizedPrincipalsCommandUser

Especifica el usuario bajo cuya cuenta se ejecuta `AuthorizedPrincipalsCommand`. Se recomienda usar un usuario dedicado que no tenga ningún otro papel en el host más que ejecutar comandos de principales autorizados. Si se especifica `AuthorizedPrincipalsCommand` pero no `AuthorizedPrincipalsCommandUser`, `sshd(8)` se negará a arrancar.

### AuthorizedPrincipalsFile

Especifica un archivo que lista los nombres de principales aceptados para la autenticación por certificado. Al usar certificados firmados por una clave listada en `TrustedUserCAKeys`, este archivo lista nombres, uno de los cuales debe aparecer en el certificado para que se acepte en la autenticación. Los nombres se listan uno por línea, precedidos de opciones de clave (según se describe en AUTHORIZED_KEYS FILE FORMAT en `sshd(8)`). Se ignoran las líneas vacías y los comentarios que empiezan por `#`.

Los argumentos de `AuthorizedPrincipalsFile` pueden incluir comodines y aceptan los tokens descritos en la sección TOKENS. Tras la expansión, `AuthorizedPrincipalsFile` se toma como una ruta absoluta o relativa al directorio home del usuario. El valor por defecto es `none`, es decir, no usar un archivo de principales; en ese caso, el nombre del usuario debe aparecer en la lista de principales del certificado para que este se acepte. Este archivo solo se comprueba para usuarios válidos.

`AuthorizedPrincipalsFile` solo se usa cuando la autenticación se hace con una CA listada en `TrustedUserCAKeys`, y no se consulta para autoridades de certificación de confianza definidas mediante `~/.ssh/authorized_keys`, aunque la opción de clave `principals=` ofrece una función similar (consulta `sshd(8)` para más detalles).

### Banner

El contenido del archivo indicado se envía al usuario remoto antes de permitir la autenticación. Si el argumento es `none`, no se muestra ningún banner. Por defecto no se muestra ningún banner.

### CASignatureAlgorithms

Especifica qué algoritmos se permiten para la firma de certificados por parte de autoridades de certificación (CA). El valor por defecto es:

```
ssh-ed25519,ecdsa-sha2-nistp256,
ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,
sk-ssh-ed25519@openssh.com,
sk-ecdsa-sha2-nistp256@openssh.com,
rsa-sha2-512,rsa-sha2-256,
ssh-mldsa44-ed25519
```

Si la lista indicada empieza por `+`, los algoritmos indicados se añadirán al conjunto por defecto en lugar de sustituirlo. Si empieza por `-`, los algoritmos indicados (incluidos comodines) se eliminarán del conjunto por defecto en lugar de sustituirlo.

Los certificados firmados con otros algoritmos no se aceptarán para la autenticación por clave pública ni basada en host.

### ChannelTimeout

Especifica si `sshd(8)` debe cerrar los canales inactivos y con qué rapidez. Los tiempos de espera se especifican como uno o más pares “type=interval” separados por espacios, donde “type” debe ser la palabra clave especial “global” o un nombre de tipo de canal de la lista siguiente, que puede contener comodines.

El valor de tiempo “interval” se especifica en segundos o puede usar cualquiera de las unidades documentadas en la sección FORMATOS DE TIEMPO. Por ejemplo, “session=5m” haría que las sesiones interactivas terminasen tras cinco minutos de inactividad. Un valor cero desactiva el tiempo de espera por inactividad.

El tiempo de espera especial “global” se aplica a todos los canales activos en conjunto. El tráfico en cualquier canal activo reinicia el contador, pero cuando expira se cierran todos los canales abiertos. Este tiempo de espera global no coincide con comodines y debe especificarse explícitamente.

Los nombres de tipo de canal disponibles incluyen:

**`agent-connection`**
: Conexiones abiertas a `ssh-agent(1)`.

**`direct-tcpip`, `direct-streamlocal@openssh.com`**
: Conexiones TCP o de socket Unix (respectivamente) abiertas que se han establecido desde un reenvío local de `ssh(1)`, es decir, `LocalForward` o `DynamicForward`.

**`forwarded-tcpip`, `forwarded-streamlocal@openssh.com`**
: Conexiones TCP o de socket Unix (respectivamente) abiertas que se han establecido hacia un `sshd(8)` que escucha en nombre de un reenvío remoto de `ssh(1)`, es decir, `RemoteForward`.

**`session`**
: La sesión principal interactiva, incluidas la sesión de shell, la ejecución de comandos, `scp(1)`, `sftp(1)`, etc.

**`tun-connection`**
: Conexiones `TunnelForward` abiertas.

**`x11-connection`**
: Sesiones de reenvío X11 abiertas.

En todos los casos anteriores, terminar una sesión inactiva no garantiza que se liberen todos los recursos asociados a ella; p. ej. los procesos de shell o los clientes X11 relacionados con la sesión pueden seguir ejecutándose.

Además, terminar un canal o sesión inactivos no cierra necesariamente la conexión SSH, ni impide que un cliente solicite otro canal del mismo tipo. En particular, que expire una sesión de reenvío inactiva no impide que se cree después otro reenvío idéntico.

Por defecto, ningún tipo de canal expira por inactividad.

### ChrootDirectory

Especifica la ruta de un directorio al que hacer `chroot(2)` tras la autenticación. Al iniciar la sesión, `sshd(8)` comprueba que todos los componentes de la ruta sean directorios propiedad de root que no sean escribibles por el grupo ni por otros. Tras el chroot, `sshd(8)` cambia el directorio de trabajo al directorio home del usuario. Los argumentos de `ChrootDirectory` aceptan los tokens descritos en la sección TOKENS.

El `ChrootDirectory` debe contener los archivos y directorios necesarios para dar soporte a la sesión del usuario. Para una sesión interactiva se necesita al menos una shell, normalmente `sh(1)`, y nodos básicos de `/dev` como los dispositivos `null(4)`, `zero(4)`, `stdin(4)`, `stdout(4)`, `stderr(4)` y `tty(4)`. Para sesiones de transferencia de archivos con SFTP no se necesita configuración adicional del entorno si se usa el sftp-server interno del proceso, aunque en algunos sistemas operativos las sesiones que usan registro pueden requerir `/dev/log` dentro del directorio chroot (consulta `sftp-server(8)` para más detalles).

Por seguridad, es muy importante impedir que otros procesos del sistema (sobre todo los que están fuera de la jaula) modifiquen la jerarquía de directorios. Una mala configuración puede dar lugar a entornos inseguros que `sshd(8)` no puede detectar.

El valor por defecto es `none`, que indica no hacer `chroot(2)`.

### Ciphers

Especifica los cifrados permitidos. Varios cifrados deben separarse con comas. Si la lista indicada empieza por `+`, los cifrados indicados se añadirán al conjunto por defecto en lugar de sustituirlo. Si empieza por `-`, los cifrados indicados (incluidos comodines) se eliminarán del conjunto por defecto. Si empieza por `^`, se colocarán al principio del conjunto por defecto.

Los cifrados soportados son:

```
3des-cbc
aes128-cbc
aes192-cbc
aes256-cbc
aes128-ctr
aes192-ctr
aes256-ctr
aes128-gcm@openssh.com
aes256-gcm@openssh.com
chacha20-poly1305@openssh.com
```

El valor por defecto es:

```
chacha20-poly1305@openssh.com,
aes128-gcm@openssh.com,aes256-gcm@openssh.com,
aes128-ctr,aes192-ctr,aes256-ctr
```

La lista de cifrados disponibles también puede obtenerse con `ssh -Q cipher`.

### ClientAliveCountMax

Establece el número de mensajes *client alive* que pueden enviarse sin que `sshd(8)` reciba ningún mensaje de vuelta del cliente. Si se alcanza este umbral mientras se envían mensajes *client alive*, sshd desconectará al cliente, terminando la sesión. Es importante señalar que el uso de mensajes *client alive* es muy distinto de `TCPKeepAlive`. Los mensajes *client alive* se envían por el canal cifrado y, por tanto, no pueden falsificarse. La opción de keepalive TCP que activa `TCPKeepAlive` sí puede falsificarse. El mecanismo *client alive* es valioso cuando el cliente o el servidor necesitan saber cuándo una conexión ha dejado de responder.

El valor por defecto es 3. Si `ClientAliveInterval` se establece a 15 y `ClientAliveCountMax` se deja en su valor por defecto, los clientes SSH que no respondan se desconectarán tras unos 45 segundos. Establecer `ClientAliveCountMax` a cero desactiva la terminación de la conexión.

### ClientAliveInterval

Establece un intervalo, en segundos, tras el cual, si no se han recibido datos del cliente, `sshd(8)` enviará un mensaje por el canal cifrado para solicitar una respuesta del cliente. El valor por defecto es 0, que indica que no se enviarán estos mensajes al cliente.

### Compression

Especifica si se activa la compresión después de que el usuario se haya autenticado correctamente. El argumento debe ser `yes`, `delayed` (sinónimo heredado de `yes`) o `no`. El valor por defecto es `yes`.

La compresión se aplica a todo el tráfico que circula por la conexión SSH. Si por la conexión se permite tráfico no confiable (como un reenvío de puerto abierto) junto con tráfico confiable, la compresión puede filtrar información sobre el contenido de la sesión. Por ello, no se recomienda activar la compresión en conexiones que compartan tráfico confiable y no confiable.

### DenyGroups

Esta palabra clave puede ir seguida de una lista de patrones de nombres de grupo, separados por espacios. Se prohíbe iniciar sesión a los usuarios cuyo grupo principal o lista de grupos suplementarios coincida con alguno de los patrones. Solo son válidos los nombres de grupo; no se reconoce un ID de grupo numérico. Por defecto se permite iniciar sesión a todos los grupos. `AllowGroups` no se consulta para los grupos que coinciden con `DenyGroups`.

Consulta PATTERNS en `ssh_config(5)` para más información sobre patrones. Esta palabra clave puede aparecer varias veces en `sshd_config`; cada aparición se añade a la lista.

### DenyUsers

Esta palabra clave puede ir seguida de una lista de patrones de nombres de usuario, separados por espacios. Se prohíbe iniciar sesión a los nombres de usuario que coincidan con alguno de los patrones. Solo son válidos los nombres de usuario; no se reconoce un ID de usuario numérico. Por defecto se permite iniciar sesión a todos los usuarios. Si el patrón tiene la forma `USER@HOST`, `USER` y `HOST` se comprueban por separado, restringiendo los inicios de sesión a usuarios concretos desde hosts concretos. Los criterios `HOST` pueden contener además direcciones a comparar en formato CIDR dirección/longitud de máscara. `AllowUsers` no se consulta para los usuarios que coinciden con `DenyUsers`.

Consulta PATTERNS en `ssh_config(5)` para más información sobre patrones. Esta palabra clave puede aparecer varias veces en `sshd_config`; cada aparición se añade a la lista.

### DisableForwarding

Desactiva todas las funciones de reenvío, incluidos X11, `ssh-agent(1)`, TCP y StreamLocal. Esta opción anula todas las demás opciones relacionadas con el reenvío y puede simplificar las configuraciones restringidas.

### ExposeAuthInfo

Escribe un archivo temporal que contiene la lista de métodos de autenticación y credenciales públicas (p. ej. claves) usadas para autenticar al usuario. La ubicación del archivo se expone a la sesión del usuario mediante la variable de entorno `SSH_USER_AUTH`. El valor por defecto es `no`.

### FingerprintHash

Especifica el algoritmo de hash usado al registrar las huellas de las claves. Las opciones válidas son `md5` y `sha256`. El valor por defecto es `sha256`.

### ForceCommand

Fuerza la ejecución del comando indicado por `ForceCommand`, ignorando cualquier comando indicado por el cliente y `~/.ssh/rc` si existe. El comando se invoca usando la shell de inicio del usuario con la opción `-c`. Se aplica a la ejecución de shell, comando o subsistema. Es especialmente útil dentro de un bloque `Match`. El comando indicado originalmente por el cliente está disponible en la variable de entorno `SSH_ORIGINAL_COMMAND`. Indicar el comando `internal-sftp` fuerza el uso de un servidor SFTP interno del proceso, que no necesita archivos de soporte cuando se usa con `ChrootDirectory`. El valor por defecto es `none`.

Esta directiva no limita otros tipos de acceso que un cliente pueda solicitar a través de su conexión, como el reenvío TCP, del agente, de sockets o X11. Si no se desean, deben desactivarse explícitamente, ya sea uno a uno mediante sus opciones respectivas o todos juntos con la opción `DisableForwarding`.

### GatewayPorts

Especifica si se permite a hosts remotos conectarse a los puertos reenviados para el cliente. Por defecto, `sshd(8)` asocia los reenvíos de puertos remotos a la dirección de loopback. Esto impide que otros hosts remotos se conecten a los puertos reenviados. `GatewayPorts` puede usarse para indicar que sshd permita asociar los reenvíos de puertos remotos a direcciones que no sean de loopback, permitiendo así que otros hosts se conecten. El argumento puede ser `no` para forzar que los reenvíos remotos solo estén disponibles para el host local, `yes` para forzar que se asocien a la dirección comodín, o `clientspecified` para permitir que el cliente elija la dirección a la que se asocia el reenvío. El valor por defecto es `no`.

### GSSAPIAuthentication

Especifica si se permite la autenticación de usuario basada en GSSAPI. El valor por defecto es `no`.

### GSSAPICleanupCredentials

Especifica si se destruye automáticamente la caché de credenciales del usuario al cerrar la sesión. El valor por defecto es `yes`.

### GSSAPIDelegateCredentials

Acepta credenciales delegadas en el lado del servidor. El valor por defecto es `yes`.

### GSSAPIStrictAcceptorCheck

Determina si se es estricto con la identidad del aceptador GSSAPI frente al que se autentica un cliente. Si se establece a `yes`, el cliente debe autenticarse frente al servicio de host del nombre de host actual. Si se establece a `no`, el cliente puede autenticarse frente a cualquier clave de servicio almacenada en el almacén por defecto de la máquina. Esta función se ofrece para facilitar el funcionamiento en máquinas con varias interfaces de red (*multi-homed*). El valor por defecto es `yes`. Esta opción puede no ser efectiva en entornos Windows Active Directory.

### HostbasedAcceptedAlgorithms

Especifica los algoritmos de firma que se aceptarán para la autenticación basada en host, como lista de patrones separada por comas. Si la lista empieza por `+`, los algoritmos indicados se añadirán al conjunto por defecto en lugar de sustituirlo. Si empieza por `-`, los algoritmos indicados (incluidos comodines) se eliminarán del conjunto por defecto. Si empieza por `^`, se colocarán al principio del conjunto por defecto. El valor por defecto de esta opción es:

```
ssh-ed25519-cert-v01@openssh.com,
ecdsa-sha2-nistp256-cert-v01@openssh.com,
ecdsa-sha2-nistp384-cert-v01@openssh.com,
ecdsa-sha2-nistp521-cert-v01@openssh.com,
sk-ssh-ed25519-cert-v01@openssh.com,
sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,
webauthn-sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,
rsa-sha2-512-cert-v01@openssh.com,
rsa-sha2-256-cert-v01@openssh.com,
ssh-mldsa44-ed25519-cert,
ssh-ed25519,
ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,
sk-ssh-ed25519@openssh.com,
sk-ecdsa-sha2-nistp256@openssh.com,
webauthn-sk-ecdsa-sha2-nistp256@openssh.com,
rsa-sha2-512,rsa-sha2-256,
ssh-mldsa44-ed25519
```

La lista de algoritmos de firma disponibles también puede obtenerse con `ssh -Q HostbasedAcceptedAlgorithms`. Antes esta opción se llamaba `HostbasedAcceptedKeyTypes`.

### HostbasedAuthentication

Especifica si se permite la autenticación mediante rhosts o `/etc/hosts.equiv` junto con una autenticación exitosa del host cliente por clave pública (autenticación basada en host). El valor por defecto es `no`.

### HostbasedUsesNameFromPacketOnly

Especifica si el servidor intentará hacer una resolución inversa de nombre al comparar el nombre en los archivos `~/.shosts`, `~/.rhosts` y `/etc/hosts.equiv` durante `HostbasedAuthentication`. Un valor `yes` significa que `sshd(8)` usa el nombre proporcionado por el cliente en lugar de intentar resolverlo a partir de la propia conexión TCP. El valor por defecto es `no`.

### HostCertificate

Especifica un archivo que contiene un certificado público de host. La clave pública del certificado debe corresponder a una clave privada de host ya indicada con `HostKey`. Por defecto, `sshd(8)` no carga ningún certificado.

### HostKey

Especifica un archivo que contiene una clave privada de host usada por SSH. Por defecto son `/etc/ssh/ssh_host_ecdsa_key`, `/etc/ssh/ssh_host_ed25519_key`, `/etc/ssh/ssh_host_mldsa44_ed25519_key` y `/etc/ssh/ssh_host_rsa_key`.

`sshd(8)` se negará a usar un archivo si es accesible por el grupo o por todos, y la opción `HostKeyAlgorithms` restringe cuáles de las claves usa realmente `sshd(8)`.

Es posible tener varios archivos de clave de host. También es posible indicar en su lugar archivos de clave pública de host; en ese caso, las operaciones con la clave privada se delegarán a un `ssh-agent(1)`.

### HostKeyAgent

Identifica el socket de dominio UNIX usado para comunicarse con un agente que tiene acceso a las claves privadas de host. Si se indica la cadena `"SSH_AUTH_SOCK"`, la ubicación del socket se leerá de la variable de entorno `SSH_AUTH_SOCK`.

### HostKeyAlgorithms

Especifica los algoritmos de firma de clave de host que ofrece el servidor. El valor por defecto de esta opción es:

```
ssh-ed25519-cert-v01@openssh.com,
ecdsa-sha2-nistp256-cert-v01@openssh.com,
ecdsa-sha2-nistp384-cert-v01@openssh.com,
ecdsa-sha2-nistp521-cert-v01@openssh.com,
sk-ssh-ed25519-cert-v01@openssh.com,
sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,
webauthn-sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,
rsa-sha2-512-cert-v01@openssh.com,
rsa-sha2-256-cert-v01@openssh.com,
ssh-mldsa44-ed25519-cert,
ssh-ed25519,
ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,
sk-ssh-ed25519@openssh.com,
sk-ecdsa-sha2-nistp256@openssh.com,
webauthn-sk-ecdsa-sha2-nistp256@openssh.com,
rsa-sha2-512,rsa-sha2-256,
ssh-mldsa44-ed25519
```

La lista de algoritmos de firma disponibles también puede obtenerse con `ssh -Q HostKeyAlgorithms`.

### IgnoreRhosts

Especifica si se ignoran los archivos `.rhosts` y `.shosts` de cada usuario durante `HostbasedAuthentication`. Los archivos globales del sistema `/etc/hosts.equiv` y `/etc/shosts.equiv` se siguen usando independientemente de este valor.

Los valores aceptados son `yes` (por defecto) para ignorar todos los archivos de usuario, `shosts-only` para permitir el uso de `.shosts` pero ignorar `.rhosts`, o `no` para permitir tanto `.shosts` como `rhosts`.

### IgnoreUserKnownHosts

Especifica si `sshd(8)` debe ignorar el `~/.ssh/known_hosts` del usuario durante `HostbasedAuthentication` y usar solo el archivo global de hosts conocidos `/etc/ssh/ssh_known_hosts`. El valor por defecto es “no”.

### Include

Incluye el o los archivos de configuración indicados. Se pueden especificar varias rutas, y cada una puede contener comodines `glob(7)`, que se expandirán y procesarán en orden léxico. Se asume que los archivos sin ruta absoluta están en `/etc/ssh`. Una directiva `Include` puede aparecer dentro de un bloque `Match` para hacer una inclusión condicional.

### IPQoS

Especifica el valor DSCP (*Differentiated Services Field Codepoint*) de la conexión. Los valores aceptados son `af11`, `af12`, `af13`, `af21`, `af22`, `af23`, `af31`, `af32`, `af33`, `af41`, `af42`, `af43`, `cs0`, `cs1`, `cs2`, `cs3`, `cs4`, `cs5`, `cs6`, `cs7`, `ef`, `le`, un valor numérico, o `none` para usar el valor por defecto del sistema operativo. Esta opción admite uno o dos argumentos separados por espacios. Si se indica uno, se usa como clase de paquete de forma incondicional. Si se indican dos, el primero se selecciona automáticamente para las sesiones interactivas y el segundo para las no interactivas. Por defecto es `ef` (*Expedited Forwarding*) para sesiones interactivas y `none` (valor por defecto del sistema operativo) para sesiones no interactivas.

### KbdInteractiveAuthentication

Especifica si se permite la autenticación *keyboard-interactive*. Se admiten todos los estilos de autenticación de `login.conf(5)`. El valor por defecto es `yes`. El argumento debe ser `yes` o `no`. `ChallengeResponseAuthentication` es un alias obsoleto de esta opción.

### KerberosAuthentication

Especifica si la contraseña proporcionada por el usuario para `PasswordAuthentication` se validará a través del KDC de Kerberos. Para usar esta opción, el servidor necesita un *servtab* de Kerberos que permita verificar la identidad del KDC. El valor por defecto es `no`.

### KerberosGetAFSToken

Si AFS está activo y el usuario tiene un TGT de Kerberos 5, intenta obtener un token AFS antes de acceder al directorio home del usuario. El valor por defecto es `no`.

### KerberosOrLocalPasswd

Si falla la autenticación por contraseña mediante Kerberos, la contraseña se validará mediante cualquier mecanismo local adicional, como `/etc/passwd`. El valor por defecto es `yes`.

### KerberosTicketCleanup

Especifica si se destruye automáticamente el archivo de caché de tickets del usuario al cerrar la sesión. El valor por defecto es `yes`.

### KexAlgorithms

Especifica los algoritmos KEX (intercambio de claves) permitidos que el servidor ofrecerá a los clientes. El orden de esta lista no es importante, ya que es el cliente quien indica el orden de preferencia. Varios algoritmos deben separarse con comas.

Si la lista empieza por `+`, los algoritmos indicados se añadirán al conjunto por defecto en lugar de sustituirlo. Si empieza por `-`, los algoritmos indicados (incluidos comodines) se eliminarán del conjunto por defecto. Si empieza por `^`, se colocarán al principio del conjunto por defecto.

Los algoritmos soportados son:

```
curve25519-sha256
curve25519-sha256@libssh.org
diffie-hellman-group1-sha1
diffie-hellman-group14-sha1
diffie-hellman-group14-sha256
diffie-hellman-group16-sha512
diffie-hellman-group18-sha512
diffie-hellman-group-exchange-sha1
diffie-hellman-group-exchange-sha256
ecdh-sha2-nistp256
ecdh-sha2-nistp384
ecdh-sha2-nistp521
mlkem768nistp256-sha256
mlkem768x25519-sha256
sntrup761x25519-sha512
sntrup761x25519-sha512@openssh.com
```

El valor por defecto es:

```
mlkem768x25519-sha256,
sntrup761x25519-sha512,sntrup761x25519-sha512@openssh.com,
curve25519-sha256,curve25519-sha256@libssh.org,
ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521
```

La lista de algoritmos de intercambio de claves soportados también puede obtenerse con `ssh -Q KexAlgorithms`.

### ListenAddress

Especifica las direcciones locales en las que debe escuchar `sshd(8)`. Pueden usarse las siguientes formas:

```
ListenAddress hostname|address [rdomain domain]
ListenAddress hostname:port [rdomain domain]
ListenAddress IPv4_address:port [rdomain domain]
ListenAddress [hostname|address]:port [rdomain domain]
```

El calificador opcional `rdomain` solicita que `sshd(8)` escuche en un dominio de enrutamiento explícito. Si no se indica `port`, sshd escuchará en la dirección y en todas las opciones `Port` especificadas. Por defecto escucha en todas las direcciones locales del dominio de enrutamiento por defecto actual. Se permiten varias opciones `ListenAddress`. Para más información sobre dominios de enrutamiento, consulta `rdomain(4)`.

### LoginGraceTime

El servidor desconecta tras este tiempo si el usuario no ha iniciado sesión correctamente. Si el valor es 0, no hay límite de tiempo. El valor por defecto es 120 segundos.

### LogLevel

Indica el nivel de detalle usado al registrar mensajes de `sshd(8)`. Los valores posibles son: `QUIET`, `FATAL`, `ERROR`, `INFO`, `VERBOSE`, `DEBUG`, `DEBUG1`, `DEBUG2` y `DEBUG3`. El valor por defecto es `INFO`. `DEBUG` y `DEBUG1` son equivalentes. `DEBUG2` y `DEBUG3` especifican cada uno niveles más altos de salida de depuración. Registrar con un nivel `DEBUG` vulnera la privacidad de los usuarios y no se recomienda.

### LogVerbose

Especifica una o más excepciones a `LogLevel`. Una excepción consiste en una o más listas de patrones que coinciden con el archivo fuente, la función y el número de línea para los que forzar un registro detallado. Por ejemplo, un patrón de excepción como:

```
kex.c:*:1000,*:kex_exchange_identification():*,packet.c:*
```

activaría el registro detallado para la línea 1000 de `kex.c`, todo lo de la función `kex_exchange_identification()` y todo el código del archivo `packet.c`. Esta opción está pensada para depuración y por defecto no hay excepciones activadas.

### MACs

Especifica los algoritmos MAC (código de autenticación de mensajes) disponibles. El algoritmo MAC se usa para proteger la integridad de los datos. Varios algoritmos deben separarse con comas. Si la lista empieza por `+`, los algoritmos indicados se añadirán al conjunto por defecto en lugar de sustituirlo. Si empieza por `-`, los algoritmos indicados (incluidos comodines) se eliminarán del conjunto por defecto. Si empieza por `^`, se colocarán al principio del conjunto por defecto.

Los algoritmos que contienen "-etm" calculan el MAC después del cifrado (*encrypt-then-mac*). Se consideran más seguros y se recomienda su uso. Los MAC soportados son:

```
hmac-md5
hmac-md5-96
hmac-sha1
hmac-sha1-96
hmac-sha2-256
hmac-sha2-512
umac-64@openssh.com
umac-128@openssh.com
hmac-md5-etm@openssh.com
hmac-md5-96-etm@openssh.com
hmac-sha1-etm@openssh.com
hmac-sha1-96-etm@openssh.com
hmac-sha2-256-etm@openssh.com
hmac-sha2-512-etm@openssh.com
umac-64-etm@openssh.com
umac-128-etm@openssh.com
```

El valor por defecto es:

```
umac-64-etm@openssh.com,umac-128-etm@openssh.com,
hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,
hmac-sha1-etm@openssh.com,
umac-64@openssh.com,umac-128@openssh.com,
hmac-sha2-256,hmac-sha2-512,hmac-sha1
```

La lista de algoritmos MAC disponibles también puede obtenerse con `ssh -Q mac`.

### Match

Inicia un bloque condicional. Si se cumplen todos los criterios de la línea `Match`, las palabras clave de las líneas siguientes sobrescriben las definidas en la sección global del archivo de configuración, hasta otra línea `Match` o el final del archivo. Si una palabra clave aparece en varios bloques `Match` que se cumplen, solo se aplica la primera aparición.

Los argumentos de `Match` son uno o más pares criterio-patrón, o uno de los criterios de token único: `All`, que coincide con todos los criterios, o `Invalid-User`, que coincide cuando el nombre de usuario solicitado no corresponde a ninguna cuenta conocida. Los criterios disponibles son `User`, `Group`, `Host`, `LocalAddress`, `LocalPort`, `Version`, `RDomain` y `Address` (donde `RDomain` representa el `rdomain(4)` por el que se recibió la conexión).

Los patrones pueden ser entradas únicas o listas separadas por comas, y pueden usar los operadores de comodín y negación descritos en la sección PATTERNS de `ssh_config(5)`.

Los patrones de un criterio `Address` pueden contener además direcciones a comparar en formato CIDR dirección/longitud de máscara, como `192.0.2.0/24` o `2001:db8::/32`. La longitud de máscara indicada debe ser coherente con la dirección: es un error indicar una longitud de máscara demasiado larga para la dirección, o una con bits activados en la parte de host de la dirección. Por ejemplo, `192.0.2.0/33` y `192.0.2.0/8`, respectivamente.

La palabra clave `Version` se compara con la cadena de versión de `sshd(8)`, por ejemplo “OpenSSH_10.0”.

En las líneas que siguen a una palabra clave `Match` solo puede usarse un subconjunto de palabras clave. Las disponibles son `AcceptEnv`, `AllowAgentForwarding`, `AllowGroups`, `AllowStreamLocalForwarding`, `AllowTcpForwarding`, `AllowUsers`, `AuthenticationMethods`, `AuthorizedKeysCommand`, `AuthorizedKeysCommandUser`, `AuthorizedKeysFile`, `AuthorizedPrincipalsCommand`, `AuthorizedPrincipalsCommandUser`, `AuthorizedPrincipalsFile`, `Banner`, `CASignatureAlgorithms`, `ChannelTimeout`, `ChrootDirectory`, `ClientAliveCountMax`, `ClientAliveInterval`, `DenyGroups`, `DenyUsers`, `DisableForwarding`, `ExposeAuthInfo`, `ForceCommand`, `GatewayPorts`, `GSSAPIAuthentication`, `HostbasedAcceptedAlgorithms`, `HostbasedAuthentication`, `HostbasedUsesNameFromPacketOnly`, `IgnoreRhosts`, `Include`, `IPQoS`, `KbdInteractiveAuthentication`, `KerberosAuthentication`, `LogLevel`, `MaxAuthTries`, `MaxSessions`, `PasswordAuthentication`, `PermitEmptyPasswords`, `PermitListen`, `PermitOpen`, `PermitRootLogin`, `PermitTTY`, `PermitTunnel`, `PermitUserRC`, `PubkeyAcceptedAlgorithms`, `PubkeyAuthentication`, `PubkeyAuthOptions`, `RefuseConnection`, `RekeyLimit`, `RevokedKeys`, `RDomain`, `SetEnv`, `StreamLocalBindMask`, `StreamLocalBindUnlink`, `TrustedUserCAKeys`, `UnusedConnectionTimeout`, `X11DisplayOffset`, `X11Forwarding` y `X11UseLocalhost`.

### MaxAuthTries

Especifica el número máximo de intentos de autenticación permitidos por conexión. Cuando el número de fallos alcanza la mitad de este valor, se registran los fallos adicionales. El valor por defecto es 6.

### MaxSessions

Especifica el número máximo de sesiones abiertas de shell, login o subsistema (p. ej. sftp) permitidas por conexión de red. Los clientes que admiten la multiplexación de conexiones pueden establecer varias sesiones. Establecer `MaxSessions` a 1 desactiva en la práctica la multiplexación de sesiones, mientras que establecerlo a 0 impide todas las sesiones de shell, login y subsistema, pero sigue permitiendo el reenvío. El valor por defecto es 10.

### MaxStartups

Especifica el número máximo de conexiones concurrentes no autenticadas al demonio SSH. Las conexiones adicionales se descartarán hasta que la autenticación tenga éxito o expire el `LoginGraceTime` de una conexión.

Como alternativa, se puede activar el descarte temprano aleatorio (*random early drop*) indicando los tres valores separados por dos puntos `start:rate:full` (p. ej. `"10:30:60"`). El valor por defecto es `10:30:100`. `sshd(8)` rechazará los intentos de conexión con una probabilidad de `rate/100` (30%) si actualmente hay `start` (10) conexiones no autenticadas. La probabilidad aumenta linealmente y se rechazan todos los intentos de conexión si el número de conexiones no autenticadas llega a `full` (60).

### ModuliFile

Especifica el archivo `moduli(5)` que contiene los grupos Diffie-Hellman usados por los métodos de intercambio de claves “diffie-hellman-group-exchange-sha1” y “diffie-hellman-group-exchange-sha256”. Por defecto es `/etc/moduli`.

### PasswordAuthentication

Especifica si se permite la autenticación por contraseña. El valor por defecto es `yes`.

### PermitEmptyPasswords

Cuando se permite la autenticación por contraseña, especifica si el servidor permite iniciar sesión en cuentas con contraseña vacía. El valor por defecto es `no`.

### PermitListen

Especifica las direcciones/puertos en los que puede escuchar un reenvío de puertos TCP remoto. La especificación de escucha debe tener una de las siguientes formas:

```
PermitListen port
PermitListen host:port
```

Se pueden especificar varios permisos separándolos con espacios. El argumento `any` elimina todas las restricciones y permite cualquier petición de escucha. El argumento `none` prohíbe todas las peticiones de escucha. El nombre de host puede contener comodines según se describe en la sección PATTERNS de `ssh_config(5)`. El comodín `*` también puede usarse en lugar de un número de puerto para permitir todos los puertos. Por defecto se permiten todas las peticiones de escucha de reenvío de puertos. La opción `GatewayPorts` puede restringir aún más en qué direcciones se puede escuchar. Ten en cuenta también que `ssh(1)` solicitará el host de escucha “localhost” si no se pidió uno explícitamente, y que este nombre se trata de forma distinta a las direcciones localhost explícitas “127.0.0.1” y “::1”.

### PermitOpen

Especifica los destinos a los que se permite el reenvío de puertos TCP. La especificación de reenvío debe tener una de las siguientes formas:

```
PermitOpen host:port
PermitOpen IPv4_addr:port
PermitOpen [IPv6_addr]:port
```

Se pueden especificar varios reenvíos separándolos con espacios. El argumento `any` elimina todas las restricciones y permite cualquier petición de reenvío. El argumento `none` prohíbe todas las peticiones de reenvío. El comodín `*` puede usarse como host o puerto para permitir todos los hosts o puertos, respectivamente. En otro caso, no se hace coincidencia de patrones ni resolución de direcciones sobre los nombres indicados. Por defecto se permiten todas las peticiones de reenvío de puertos.

### PermitRootLogin

Especifica si root puede iniciar sesión con `ssh(1)`. El argumento debe ser `yes`, `prohibit-password`, `forced-commands-only` o `no`. El valor por defecto es `prohibit-password`.

Si se establece a `prohibit-password` (o su alias obsoleto, `without-password`), se desactivan para root la autenticación por contraseña y la *keyboard-interactive*.

Si se establece a `forced-commands-only`, se permitirá a root iniciar sesión con autenticación por clave pública, pero solo si se ha especificado la opción `command` (lo que puede ser útil para hacer copias de seguridad remotas aunque normalmente no se permita iniciar sesión como root). Todos los demás métodos de autenticación se desactivan para root.

Si se establece a `no`, no se permite a root iniciar sesión.

### PermitTTY

Especifica si se permite la asignación de `pty(4)`. El valor por defecto es `yes`.

### PermitTunnel

Especifica si se permite el reenvío de dispositivos `tun(4)`. El argumento debe ser `yes`, `point-to-point` (capa 3), `ethernet` (capa 2) o `no`. Indicar `yes` permite tanto `point-to-point` como `ethernet`. El valor por defecto es `no`.

Independientemente de este valor, los permisos del dispositivo `tun(4)` seleccionado deben permitir el acceso al usuario.

### PermitUserEnvironment

Especifica si `sshd(8)` procesa `~/.ssh/environment` y las opciones `environment=` de `~/.ssh/authorized_keys`. Las opciones válidas son `yes`, `no` o una lista de patrones que indica qué nombres de variables de entorno aceptar (por ejemplo `"LANG,LC_*"`). El valor por defecto es `no`. Activar el procesamiento del entorno puede permitir a los usuarios eludir restricciones de acceso en algunas configuraciones, mediante mecanismos como `LD_PRELOAD`.

### PermitUserRC

Especifica si se ejecuta algún archivo `~/.ssh/rc`. El valor por defecto es `yes`.

### PerSourceMaxStartups

Especifica el número de conexiones no autenticadas permitidas desde una misma dirección de origen, o “none” si no hay límite. Este límite se aplica además de `MaxStartups`, prevaleciendo el menor de los dos. El valor por defecto es `none`.

### PerSourceNetBlockSize

Especifica el número de bits de la dirección de origen que se agrupan a efectos de aplicar los límites de `PerSourceMaxStartups`. Se pueden indicar valores para IPv4 y, opcionalmente, IPv6, separados por dos puntos. El valor por defecto es `32:128`, lo que significa que cada dirección se considera individualmente.

### PerSourcePenalties

Controla las penalizaciones para diversas condiciones que pueden representar ataques contra `sshd(8)`. Si se aplica una penalización a un cliente, se rechazarán durante un periodo las conexiones desde su dirección de origen y cualquier otra de la misma red, según la defina `PerSourceNetBlockSize`.

Una penalización no afecta a las conexiones concurrentes en curso, pero varias penalizaciones de la misma fuente procedentes de conexiones concurrentes se acumularán hasta un máximo. A la inversa, las penalizaciones no se aplican hasta que se haya acumulado un tiempo mínimo umbral.

Las penalizaciones están activadas por defecto con los valores indicados a continuación, pero pueden desactivarse con la palabra clave `no`. Los valores por defecto pueden cambiarse indicando una o más de las palabras clave siguientes, separadas por espacios. Todas las palabras clave aceptan argumentos, p. ej. `"crash:2m"`.

**`crash:duration`**
: Especifica durante cuánto tiempo se rechaza a los clientes que provocan una caída de `sshd(8)` (por defecto: 90s).

**`authfail:duration`**
: Especifica durante cuánto tiempo se rechaza a los clientes que se desconectan tras uno o más intentos de autenticación fallidos (por defecto: 5s).

**`invaliduser:duration`**
: Especifica durante cuánto tiempo se rechaza a los clientes que intentan iniciar sesión con un usuario no válido (por defecto: 5s).

**`refuseconnection:duration`**
: Especifica durante cuánto tiempo se rechaza a los clientes a los que se prohibió administrativamente la conexión mediante la opción `RefuseConnection` (por defecto: 10s).

**`noauth:duration`**
: Especifica durante cuánto tiempo se rechaza a los clientes que se desconectan sin intentar autenticarse (por defecto: 1s). Este tiempo debe usarse con cautela, ya que podría penalizar herramientas de escaneo legítimas como `ssh-keyscan(1)`.

**`grace-exceeded:duration`**
: Especifica durante cuánto tiempo se rechaza a los clientes que no consiguen autenticarse dentro de `LoginGraceTime` (por defecto: 10s).

**`max:duration`**
: Especifica el tiempo máximo durante el que se denegará el acceso a un rango de direcciones de origen concreto (por defecto: 10m). Las penalizaciones repetidas se acumulan hasta este máximo.

**`min:duration`**
: Especifica la penalización mínima que debe acumularse antes de que empiece a aplicarse (por defecto: 15s).

**`max-sources4:number`, `max-sources6:number`**
: Especifica el número máximo de rangos de direcciones IPv4 e IPv6 de clientes a los que se hará seguimiento de penalizaciones (por defecto: 65536 para ambos).

**`overflow:mode`**
: Controla cómo se comporta el servidor cuando se supera `max-sources4` o `max-sources6`. Hay dos modos de funcionamiento: `deny-all`, que deniega todas las conexiones entrantes salvo las exentas mediante `PerSourcePenaltyExemptList` hasta que expire una penalización, y `permissive`, que permite nuevas conexiones eliminando anticipadamente penalizaciones existentes (por defecto: `permissive`). Las penalizaciones de clientes por debajo del umbral `min` cuentan para el total de penalizaciones seguidas. Las direcciones IPv4 e IPv6 se siguen por separado, de modo que un desbordamiento en una no afecta a la otra.

**`overflow6:mode`**
: Permite especificar un modo de desbordamiento distinto para las direcciones IPv6. Por defecto se usa el mismo modo que se indicó para IPv4.

### PerSourcePenaltyExemptList

Especifica una lista, separada por comas, de direcciones exentas de penalizaciones. Esta lista puede contener comodines y rangos CIDR dirección/longitud de máscara. La longitud de máscara indicada debe ser coherente con la dirección: es un error indicar una longitud de máscara demasiado larga para la dirección, o una con bits activados en la parte de host de la dirección. Por ejemplo, `192.0.2.0/33` y `192.0.2.0/8`, respectivamente. Por defecto no se exime ninguna dirección.

### PidFile

Especifica el archivo que contiene el ID de proceso del demonio SSH, o `none` para no escribirlo. Por defecto es `/var/run/sshd.pid`.

### Port

Especifica el número de puerto en el que escucha `sshd(8)`. El valor por defecto es 22. Se permiten varias opciones de este tipo. Consulta también `ListenAddress`.

### PrintLastLog

Especifica si `sshd(8)` debe imprimir la fecha y hora del último inicio de sesión del usuario cuando este inicia sesión de forma interactiva. El valor por defecto es `yes`.

### PrintMotd

Especifica si `sshd(8)` debe imprimir `/etc/motd` cuando un usuario inicia sesión de forma interactiva. (En algunos sistemas también lo imprime la shell, `/etc/profile` o equivalente.) El valor por defecto es `yes`.

### PubkeyAcceptedAlgorithms

Especifica los algoritmos de firma que se aceptarán para la autenticación por clave pública, como lista de patrones separada por comas. Si la lista empieza por `+`, los algoritmos indicados se añadirán al conjunto por defecto en lugar de sustituirlo. Si empieza por `-`, los algoritmos indicados (incluidos comodines) se eliminarán del conjunto por defecto. Si empieza por `^`, se colocarán al principio del conjunto por defecto. El valor por defecto de esta opción es:

```
ssh-ed25519-cert-v01@openssh.com,
ecdsa-sha2-nistp256-cert-v01@openssh.com,
ecdsa-sha2-nistp384-cert-v01@openssh.com,
ecdsa-sha2-nistp521-cert-v01@openssh.com,
sk-ssh-ed25519-cert-v01@openssh.com,
sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,
webauthn-sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,
rsa-sha2-512-cert-v01@openssh.com,
rsa-sha2-256-cert-v01@openssh.com,
ssh-mldsa44-ed25519-cert,
ssh-ed25519,
ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,
sk-ssh-ed25519@openssh.com,
sk-ecdsa-sha2-nistp256@openssh.com,
webauthn-sk-ecdsa-sha2-nistp256@openssh.com,
rsa-sha2-512,rsa-sha2-256,
ssh-mldsa44-ed25519
```

La lista de algoritmos de firma disponibles también puede obtenerse con `ssh -Q PubkeyAcceptedAlgorithms`.

### PubkeyAuthOptions

Establece una o más opciones de autenticación por clave pública. Las palabras clave soportadas son: `none` (por defecto; indica que no se activa ninguna opción adicional), `touch-required`, `verify-required` y `max-pk-ok`.

La opción `touch-required` hace que la autenticación por clave pública con un algoritmo de autenticador FIDO (es decir, `ecdsa-sk` o `ed25519-sk`) exija siempre que la firma atestigüe que un usuario físicamente presente confirmó explícitamente la autenticación (normalmente tocando el autenticador). Por defecto, `sshd(8)` exige la presencia del usuario salvo que se anule con una opción de `authorized_keys`. El flag `touch-required` desactiva esa anulación.

La opción `verify-required` exige que la firma de una clave FIDO atestigüe que se verificó al usuario, p. ej. mediante un PIN.

Ni `touch-required` ni `verify-required` tienen efecto en otros tipos de clave pública que no sean FIDO.

La opción `max-pk-ok` lleva un argumento numérico separado por dos puntos, p. ej. `max-pk-ok:4`, para indicar cuántas consultas de clave pública se permiten antes de que los intentos se consideren autenticaciones fallidas que cuentan para `MaxAuthTries`. Las consultas de clave pública son un mecanismo del protocolo SSH que permite a un cliente comprobar si un servidor acepta una clave pública dada antes de un intento de autenticación real, para evitar tener que introducir PINs/frases de contraseña o tocar autenticadores FIDO innecesariamente. El valor por defecto es 6.

### PubkeyAuthentication

Especifica si se permite la autenticación por clave pública. El valor por defecto es `yes`.

### RefuseConnection

Indica que `sshd(8)` debe terminar la conexión de forma incondicional. Además, si `PerSourcePenalties` está activado, puede registrarse una penalización `refuseconnection` contra el origen de la conexión. En la práctica esta opción solo es útil dentro de un bloque `Match`.

### RekeyLimit

Especifica la cantidad máxima de datos que pueden transmitirse o recibirse antes de renegociar la clave de sesión, seguida opcionalmente del tiempo máximo que puede pasar antes de renegociarla. El primer argumento se indica en bytes y puede llevar el sufijo `K`, `M` o `G` para indicar kilobytes, megabytes o gigabytes, respectivamente. El valor por defecto está entre `1G` y `4G`, según el cifrado. El segundo valor, opcional, se indica en segundos y puede usar cualquiera de las unidades documentadas en la sección FORMATOS DE TIEMPO. El valor por defecto de `RekeyLimit` es `default none`, lo que significa que la renegociación se hace tras enviar o recibir la cantidad de datos por defecto del cifrado, y no se hace renegociación basada en tiempo.

### RequiredRSASize

Especifica el tamaño mínimo de clave RSA (en bits) que `sshd(8)` aceptará. Se rechazarán las claves de autenticación de usuario y basada en host más pequeñas que este límite. El valor por defecto es 1024 bits. Este límite solo puede aumentarse respecto al valor por defecto.

### RevokedKeys

Especifica el archivo de claves públicas revocadas, o `none` para no usar ninguno. Las claves listadas en este archivo se rechazarán para la autenticación por clave pública. Si este archivo no es legible, se rechazará la autenticación por clave pública para todos los usuarios. Las claves pueden indicarse como un archivo de texto con una clave pública por línea, o como una Lista de Revocación de Claves (KRL) de OpenSSH generada por `ssh-keygen(1)`. `sshd(8)` puede consultar este archivo en cada intento de autenticación por clave pública que reciba y su contenido debe ser coherente en todo momento, por lo que solo debe reemplazarse de forma atómica y nunca modificarse en el sitio mientras el servidor está en ejecución. Para más información sobre las KRL, consulta la sección KEY REVOCATION LISTS de `ssh-keygen(1)`.

### RDomain

Especifica un dominio de enrutamiento explícito que se aplica tras completar la autenticación. La sesión del usuario, así como cualquier socket IP reenviado o de escucha, quedará asociada a este `rdomain(4)`. Si el dominio de enrutamiento se establece a `%D`, se aplicará el dominio en el que se recibió la conexión entrante.

### SecurityKeyProvider

Especifica la ruta a una biblioteca que se usará al cargar claves alojadas en autenticadores FIDO, en lugar del soporte USB HID integrado que se usa por defecto.

### SetEnv

Especifica una o más variables de entorno, como “NAME=VALUE”, que se establecerán en las sesiones hijas iniciadas por `sshd(8)`. El valor puede ir entrecomillado (p. ej. si contiene espacios). Las variables definidas con `SetEnv` sobrescriben el entorno por defecto y cualquier variable indicada por el usuario mediante `AcceptEnv` o `PermitUserEnvironment`.

### SshdAuthPath

Sobrescribe la ruta por defecto del binario `sshd-auth`, que se invoca para completar la autenticación del usuario. Por defecto es `/usr/libexec/sshd-auth`. Esta opción está pensada para su uso en pruebas.

### SshdSessionPath

Sobrescribe la ruta por defecto del binario `sshd-session`, que se invoca para gestionar cada conexión. Por defecto es `/usr/libexec/sshd-session`. Esta opción está pensada para su uso en pruebas.

### StreamLocalBindMask

Establece la máscara octal de modo de creación de archivos (umask) usada al crear un archivo de socket de dominio Unix para el reenvío de puertos local o remoto. Solo se usa para el reenvío de puertos hacia un archivo de socket de dominio Unix.

El valor por defecto es 0177, que crea un archivo de socket de dominio Unix legible y escribible solo por el propietario. No todos los sistemas operativos respetan el modo de archivo en los archivos de socket de dominio Unix.

### StreamLocalBindUnlink

Especifica si se elimina un archivo de socket de dominio Unix existente para el reenvío de puertos local o remoto antes de crear uno nuevo. Si el archivo de socket ya existe y `StreamLocalBindUnlink` no está activado, sshd no podrá reenviar el puerto al archivo de socket de dominio Unix. Solo se usa para el reenvío de puertos hacia un archivo de socket de dominio Unix.

El argumento debe ser `yes` o `no`. El valor por defecto es `no`.

### StrictModes

Especifica si `sshd(8)` debe comprobar los modos y el propietario de los archivos y del directorio home del usuario antes de aceptar el inicio de sesión. Normalmente es deseable, porque los novatos a veces dejan por accidente su directorio o sus archivos escribibles por todos. El valor por defecto es `yes`. Esto no se aplica a `ChrootDirectory`, cuyos permisos y propietario se comprueban de forma incondicional.

### Subsystem

Configura un subsistema externo (p. ej. un demonio de transferencia de archivos). Los argumentos deben ser un nombre de subsistema y un comando (con argumentos opcionales) a ejecutar cuando se solicite el subsistema.

El comando `sftp-server` implementa el subsistema de transferencia de archivos SFTP.

Como alternativa, el nombre `internal-sftp` implementa un servidor SFTP interno del proceso. Esto puede simplificar las configuraciones que usan `ChrootDirectory` para forzar una raíz de sistema de archivos distinta a los clientes. Acepta los mismos argumentos de línea de comandos que `sftp-server` y, aunque se ejecuta dentro del proceso, valores como `LogLevel` o `SyslogFacility` no se le aplican y deben establecerse explícitamente mediante argumentos de línea de comandos.

Por defecto no se define ningún subsistema.

### SyslogFacility

Indica el código de *facility* usado al registrar mensajes de `sshd(8)`. Los valores posibles son: `DAEMON`, `USER`, `AUTH`, `LOCAL0`, `LOCAL1`, `LOCAL2`, `LOCAL3`, `LOCAL4`, `LOCAL5`, `LOCAL6`, `LOCAL7`. El valor por defecto es `AUTH`.

### TCPKeepAlive

Especifica si el sistema debe enviar mensajes keepalive en los sockets TCP que ha abierto. Si se envían, se podrá detectar antes una caída de la conexión o el fallo de uno de los extremos. Sin embargo, esto significa que las conexiones pueden terminar si sufren una interrupción transitoria.

El argumento debe ser `transport` para activar los mensajes keepalive TCP en la conexión de transporte SSH con el cliente, `yes` como alias de `transport`, `all` para activarlos en todos los sockets abiertos por `sshd(8)`, incluidos los creados para reenvío X11 o de puertos, o `no` para desactivarlos. El valor por defecto es `transport`.

Consulta también `ClientAliveInterval` para un mecanismo más robusto de detección de fallos de conexión que funciona a nivel del protocolo SSH.

### TrustedUserCAKeys

Especifica un archivo que contiene las claves públicas de las autoridades de certificación de confianza para firmar certificados de usuario para la autenticación, o `none` para no usar ninguno. Las claves se listan una por línea; se permiten líneas vacías y comentarios que empiezan por `#`. Si se presenta un certificado para autenticarse y su clave de CA firmante está en este archivo, puede usarse para autenticar a cualquier usuario incluido en la lista de principales del certificado. Los certificados sin lista de principales no se permitirán para la autenticación mediante `TrustedUserCAKeys`. Para más detalles sobre certificados, consulta la sección CERTIFICATES de `ssh-keygen(1)`.

### UnusedConnectionTimeout

Especifica si `sshd(8)` debe cerrar las conexiones de clientes sin canales abiertos y con qué rapidez. Los canales abiertos incluyen sesiones activas de shell, ejecución de comandos o subsistema, y reenvíos conectados de red, socket, agente o X11. Los puntos de escucha de reenvío, como los del flag `-R` de `ssh(1)`, no se consideran canales abiertos y no impiden el tiempo de espera. El valor se especifica en segundos o puede usar cualquiera de las unidades documentadas en la sección FORMATOS DE TIEMPO.

Este tiempo empieza a contar cuando la conexión del cliente completa la autenticación del usuario, pero antes de que el cliente tenga ocasión de abrir ningún canal. Hay que tener cuidado con valores cortos, ya que pueden no dar tiempo suficiente al cliente para solicitar y abrir sus canales antes de que se termine la conexión.

El valor por defecto, `none`, consiste en no expirar nunca las conexiones por no tener canales abiertos. Esta opción puede ser útil junto con `ChannelTimeout`.

### UseDNS

Especifica si `sshd(8)` debe resolver el nombre del host remoto y comprobar que el nombre resuelto para la dirección IP remota apunta de vuelta a esa misma dirección IP.

Si esta opción se establece a `no` (por defecto), en las directivas `from` de `~/.ssh/authorized_keys` y `Match Host` de `sshd_config` solo pueden usarse direcciones, no nombres de host.

### VersionAddendum

Especifica opcionalmente un texto adicional que se añadirá al banner del protocolo SSH que envía el servidor al conectar. El valor por defecto es `none`.

### WarnWeakCrypto

Controla si se registran advertencias cuando los algoritmos criptográficos negociados para la conexión son débiles o no recomendados. Las advertencias pueden desactivarse apagando una advertencia concreta o desactivándolas todas. Las advertencias sobre conexiones que no usan un intercambio de claves post-cuántico pueden desactivarse con el flag `no-pq-kex`. `no` desactiva todas las advertencias. El valor por defecto, equivalente a `yes`, es activar todas las advertencias.

### X11DisplayOffset

Especifica el primer número de display disponible para el reenvío X11 de `sshd(8)`. Esto evita que sshd interfiera con servidores X11 reales. El valor por defecto es 10.

### X11Forwarding

Especifica si se permite el reenvío X11. El argumento debe ser `yes` o `no`. El valor por defecto es `no`.

Cuando el reenvío X11 está activado, puede haber una exposición adicional del servidor y de los displays de los clientes si el display proxy de `sshd(8)` está configurado para escuchar en la dirección comodín (consulta `X11UseLocalhost`), aunque no es lo que ocurre por defecto. Además, la suplantación de la autenticación y la verificación y sustitución de los datos de autenticación ocurren en el lado del cliente. El riesgo de seguridad del reenvío X11 es que el servidor de display X11 del cliente puede quedar expuesto a ataques cuando el cliente SSH solicita el reenvío (consulta las advertencias de `ForwardX11` en `ssh_config(5)`). Un administrador de sistemas puede querer proteger a los clientes que podrían exponerse a ataques al solicitar sin saberlo el reenvío X11, lo que puede justificar un valor `no`.

Desactivar el reenvío X11 no impide que los usuarios reenvíen tráfico X11, ya que siempre pueden instalar sus propios mecanismos de reenvío.

### X11UseLocalhost

Especifica si `sshd(8)` debe asociar el servidor de reenvío X11 a la dirección de loopback o a la dirección comodín. Por defecto, sshd asocia el servidor de reenvío a la dirección de loopback y establece la parte de nombre de host de la variable de entorno `DISPLAY` a `localhost`. Esto impide que hosts remotos se conecten al display proxy. Sin embargo, algunos clientes X11 antiguos pueden no funcionar con esta configuración. `X11UseLocalhost` puede establecerse a `no` para indicar que el servidor de reenvío se asocie a la dirección comodín. El argumento debe ser `yes` o `no`. El valor por defecto es `yes`.

### XAuthLocation

Especifica la ruta completa del programa `xauth(1)`, o `none` para no usar ninguno. Por defecto es `/usr/X11R6/bin/xauth`.

## FORMATOS DE TIEMPO

Los argumentos de línea de comandos de `sshd(8)` y las opciones del archivo de configuración que especifican tiempo pueden expresarse con una secuencia de la forma `time[qualifier]`, donde `time` es un entero positivo y `qualifier` es uno de los siguientes:

| Calificador | Unidad |
|---|---|
| ⟨ninguno⟩ | segundos |
| `s` \| `S` | segundos |
| `m` \| `M` | minutos |
| `h` \| `H` | horas |
| `d` \| `D` | días |
| `w` \| `W` | semanas |

Cada miembro de la secuencia se suma para calcular el valor total de tiempo.

Ejemplos de formato de tiempo:

| Valor | Significado |
|---|---|
| `600` | 600 segundos (10 minutos) |
| `10m` | 10 minutos |
| `1h30m` | 1 hora 30 minutos (90 minutos) |

## TOKENS

Los argumentos de algunas palabras clave pueden usar tokens, que se expanden en tiempo de ejecución. Los tokens se expanden sin entrecomillar ni escapar los caracteres de la shell. Es responsabilidad del administrador asegurarse de que son seguros en el contexto en que se usan.

Los tokens admitidos en `sshd_config` son:

| Token | Significado |
|---|---|
| `%%` | Un `%` literal. |
| `%C` | Identifica los extremos de la conexión; contiene cuatro valores separados por espacios: dirección del cliente, puerto del cliente, dirección del servidor y puerto del servidor. |
| `%D` | El dominio de enrutamiento en el que se recibió la conexión entrante. |
| `%F` | La huella de la clave de la CA. |
| `%f` | La huella de la clave o del certificado. |
| `%h` | El directorio home del usuario. |
| `%i` | El ID de clave del certificado. |
| `%K` | La clave de la CA codificada en base64. |
| `%k` | La clave o certificado para la autenticación, codificado en base64. |
| `%s` | El número de serie del certificado. |
| `%T` | El tipo de la clave de la CA. |
| `%t` | El tipo de clave o de certificado. |
| `%U` | El ID numérico del usuario objetivo. |
| `%u` | El nombre de usuario. |

`AgentSocketPath` acepta los tokens `%%`, `%h`, `%U` y `%u`.

`AuthorizedKeysCommand` acepta los tokens `%%`, `%C`, `%D`, `%f`, `%h`, `%k`, `%t`, `%U` y `%u`.

`AuthorizedKeysFile` acepta los tokens `%%`, `%h`, `%U` y `%u`.

`AuthorizedPrincipalsCommand` acepta los tokens `%%`, `%C`, `%D`, `%F`, `%f`, `%h`, `%i`, `%K`, `%k`, `%s`, `%T`, `%t`, `%U` y `%u`.

`AuthorizedPrincipalsFile` acepta los tokens `%%`, `%h`, `%U` y `%u`.

`ChrootDirectory` acepta los tokens `%%`, `%h`, `%U` y `%u`.

`RoutingDomain` acepta el token `%D`.

## ARCHIVOS

**`/etc/ssh/sshd_config`**
: Contiene los datos de configuración de `sshd(8)`. Este archivo solo debe poder escribirlo root, pero se recomienda (aunque no es necesario) que sea legible por todos.

## VÉASE TAMBIÉN

`sftp-server(8)`, `sshd(8)`

## AUTORES

OpenSSH es un derivado de la versión original y libre ssh 1.2.12 de Tatu Ylonen. Aaron Campbell, Bob Beck, Markus Friedl, Niels Provos, Theo de Raadt y Dug Song eliminaron muchos errores, volvieron a añadir funciones más recientes y crearon OpenSSH. Markus Friedl contribuyó el soporte para las versiones 1.5 y 2.0 del protocolo SSH. Niels Provos y Markus Friedl contribuyeron el soporte para la separación de privilegios.
