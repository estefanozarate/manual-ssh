# SSH_CONFIG(5)

## NOMBRE

**ssh_config** — archivo de configuración del cliente OpenSSH

## DESCRIPCIÓN

`ssh(1)` obtiene los datos de configuración de las siguientes fuentes, en este orden:

1. opciones de línea de comandos
2. archivo de configuración del usuario (`~/.ssh/config`)
3. archivo de configuración global del sistema (`/etc/ssh/ssh_config`)

Salvo que se indique lo contrario, para cada directiva de configuración se usará el primer valor especificado.

Los archivos de configuración pueden contener secciones separadas por directivas condicionales `Host` o `Match`. La configuración de estas secciones solo se aplica si coincide la directiva condicional que inició la sección.

Como se usa el primer valor obtenido para cada directiva, las declaraciones más específicas de cada host deben ir cerca del principio del archivo, y los valores generales por defecto al final.

El archivo contiene pares directiva/valor, uno por línea. Las líneas que empiezan por `#` y las líneas vacías se interpretan como comentarios. Un `#` fuera de una cadena entrecomillada puede usarse para añadir un comentario al final de una línea. Los espacios al principio o al final de las líneas se ignoran. Los valores pueden ir opcionalmente entre comillas dobles (`"`) para representar argumentos que contienen espacios.

Las directivas de configuración se separan de sus valores mediante espacios o exactamente un carácter `=` (que puede estar rodeado de espacios); este último formato es útil para evitar tener que entrecomillar espacios al especificar opciones de configuración con la opción `-o` de ssh, scp y sftp.

Las palabras clave posibles y su significado son los siguientes (las palabras clave no distinguen mayúsculas de minúsculas, pero los argumentos sí):

### Host

Restringe las declaraciones siguientes (hasta la próxima palabra clave `Host` o `Match`) para que solo se apliquen a los hosts que coincidan con alguno de los patrones indicados tras la palabra clave. Si se indica más de un patrón, deben separarse con espacios. Un único `*` como patrón puede usarse para dar valores globales por defecto a todos los hosts. El host suele ser el argumento *hostname* dado en la línea de comandos (consulta la palabra clave `CanonicalizeHostname` para las excepciones).

Una entrada de patrón puede negarse anteponiéndole un signo de exclamación (`!`). Si coincide una entrada negada, la entrada `Host` se ignora, independientemente de si coinciden otros patrones de la línea. Por tanto, las coincidencias negadas son útiles para definir excepciones a las coincidencias con comodines.

Consulta PATRONES para más información sobre patrones.

### Match

Restringe las declaraciones siguientes (hasta la próxima palabra clave `Host` o `Match`) para que solo se usen cuando se cumplan las condiciones que siguen a la palabra clave `Match`. Las condiciones se especifican con uno o más criterios, o con el token único `all`, que siempre coincide. Las palabras clave de criterio disponibles son: `canonical`, `final`, `exec`, `localnetwork`, `host`, `originalhost`, `tagged`, `command`, `user`, `localuser` y `version`. El criterio `all` debe aparecer solo o inmediatamente después de `canonical` o `final`. Los demás criterios pueden combinarse libremente. Todos los criterios salvo `all`, `canonical` y `final` requieren un argumento. Los criterios pueden negarse anteponiendo un signo de exclamación (`!`).

La palabra clave `canonical` solo coincide cuando el archivo de configuración se está volviendo a analizar tras la canonicalización del nombre de host (consulta la opción `CanonicalizeHostname`). Puede ser útil para especificar condiciones que solo funcionen con nombres de host canónicos.

La palabra clave `final` solicita que la configuración se vuelva a analizar (esté o no activado `CanonicalizeHostname`), y solo coincide durante esta pasada final. Si `CanonicalizeHostname` está activado, `canonical` y `final` coinciden durante la misma pasada.

La palabra clave `exec` ejecuta el comando indicado bajo la shell del usuario. Si el comando devuelve un estado de salida cero, la condición se considera verdadera. Los comandos que contienen espacios deben entrecomillarse. Los argumentos de `exec` aceptan los tokens descritos en la sección TOKENS.

La palabra clave `localnetwork` compara las direcciones de las interfaces de red locales activas con la lista de redes en formato CIDR indicada. Puede ser práctica para variar la configuración efectiva en dispositivos que se mueven entre redes. Ten en cuenta que la dirección de red no es un criterio fiable en muchas situaciones (p. ej. cuando la red se configura automáticamente por DHCP), por lo que hay que tener precaución si se usa para controlar configuración sensible para la seguridad.

Los criterios de las demás palabras clave deben ser entradas únicas o listas separadas por comas, y pueden usar los operadores de comodín y negación descritos en la sección PATRONES.

Los criterios de la palabra clave `host` se comparan con el nombre de host de destino, tras cualquier sustitución hecha por las opciones `Hostname` o `CanonicalizeHostname`. La palabra clave `originalhost` se compara con el nombre de host tal como se indicó en la línea de comandos.

La palabra clave `tagged` coincide con un nombre de etiqueta especificado por una directiva `Tag` previa o en la línea de comandos de `ssh(1)` con el flag `-P`. La palabra clave `command` coincide con el comando remoto solicitado, o con el nombre del subsistema que se invoca (p. ej. "sftp" para una sesión SFTP). La cadena vacía coincide con el caso en que no se ha especificado comando o etiqueta, es decir, `Match tag ""`. La palabra clave `version` se compara con la cadena de versión de `ssh(1)`, por ejemplo “OpenSSH_10.0”.

La palabra clave `user` se compara con el nombre de usuario de destino en el host remoto. La palabra clave `localuser` se compara con el nombre del usuario local que ejecuta `ssh(1)` (puede ser útil en archivos `ssh_config` globales del sistema).

Por último, la palabra clave `sessiontype` coincide con el tipo de sesión solicitado, que puede ser `shell` para sesiones interactivas, `exec` para sesiones de ejecución de comandos, `subsystem` para invocaciones de subsistemas como `sftp(1)`, o `none` para sesiones solo de transporte, como cuando `ssh(1)` se inicia con el flag `-N`.

### Include

Incluye el o los archivos de configuración indicados. Se pueden especificar varias rutas, y cada una puede contener comodines `glob(7)`, tokens según la sección TOKENS, variables de entorno según la sección VARIABLES DE ENTORNO y, en configuraciones de usuario, referencias `~` al estilo de la shell a directorios home de usuarios. Los comodines se expandirán y procesarán en orden léxico. Se asume que los archivos sin ruta absoluta están en `~/.ssh` si se incluyen desde un archivo de configuración de usuario, o en `/etc/ssh` si se incluyen desde el archivo de configuración del sistema. La directiva `Include` puede aparecer dentro de un bloque `Match` o `Host` para hacer una inclusión condicional.

### AddKeysToAgent

Especifica si las claves deben añadirse automáticamente a un `ssh-agent(1)` en ejecución. Si se establece a `yes` y se carga una clave desde un archivo, la clave y su frase de contraseña se añaden al agente con la vida por defecto, como si se hiciera con `ssh-add(1)`. Si se establece a `ask`, `ssh(1)` pedirá confirmación mediante el programa `SSH_ASKPASS` antes de añadir una clave (consulta `ssh-add(1)` para más detalles). Si se establece a `confirm`, cada uso de la clave deberá confirmarse, como si se hubiera indicado la opción `-c` a `ssh-add(1)`. Si se establece a `no`, no se añade ninguna clave al agente. Como alternativa, esta opción puede indicarse como un intervalo de tiempo con el formato descrito en la sección FORMATOS DE TIEMPO de `sshd_config(5)`, para especificar la vida de la clave en `ssh-agent(1)`, tras la cual se eliminará automáticamente. El argumento debe ser `no` (por defecto), `yes`, `confirm` (opcionalmente seguido de un intervalo de tiempo), `ask` o un intervalo de tiempo.

### AddressFamily

Especifica qué familia de direcciones usar al conectar. Los argumentos válidos son `any` (por defecto), `inet` (solo IPv4) o `inet6` (solo IPv6).

### BatchMode

Si se establece a `yes`, se desactiva la interacción con el usuario, como las solicitudes de contraseña y de confirmación de la clave de host. Es útil en scripts y otros trabajos por lotes donde no hay un usuario presente para interactuar con `ssh(1)`. El argumento debe ser `yes` o `no` (por defecto).

### BindAddress

Usa la dirección indicada de la máquina local como dirección de origen de la conexión. Solo es útil en sistemas con más de una dirección.

### BindInterface

Usa la dirección de la interfaz indicada de la máquina local como dirección de origen de la conexión.

### CanonicalDomains

Cuando `CanonicalizeHostname` está activado, esta opción especifica la lista de sufijos de dominio en los que buscar el host de destino indicado.

### CanonicalizeFallbackLocal

Especifica si se debe fallar con un error cuando falla la canonicalización del nombre de host. El valor por defecto, `yes`, intentará resolver el nombre de host no cualificado usando las reglas de búsqueda del resolvedor del sistema. Un valor `no` hará que `ssh(1)` falle de inmediato si `CanonicalizeHostname` está activado y el nombre de host de destino no se encuentra en ninguno de los dominios indicados en `CanonicalDomains`.

### CanonicalizeHostname

Controla si se realiza una canonicalización explícita del nombre de host. El valor por defecto, `no`, consiste en no reescribir ningún nombre y dejar que el resolvedor del sistema gestione todas las búsquedas. Si se establece a `yes`, para las conexiones que no usan `ProxyCommand` ni `ProxyJump`, `ssh(1)` intentará canonicalizar el nombre de host indicado en la línea de comandos usando los sufijos de `CanonicalDomains` y las reglas de `CanonicalizePermittedCNAMEs`. Si `CanonicalizeHostname` se establece a `always`, la canonicalización se aplica también a las conexiones a través de proxy.

Si esta opción está activada, los archivos de configuración se procesan de nuevo usando el nuevo nombre de destino para recoger cualquier configuración nueva de las estrofas `Host` y `Match` que coincidan. Un valor `none` desactiva el uso de un host `ProxyJump`.

### CanonicalizeMaxDots

Especifica el número máximo de puntos en un nombre de host antes de que se desactive la canonicalización. El valor por defecto, 1, permite un único punto (es decir, `hostname.subdomain`).

### CanonicalizePermittedCNAMEs

Especifica las reglas para determinar si deben seguirse los CNAME al canonicalizar nombres de host. Las reglas consisten en uno o más argumentos de la forma `source_domain_list:target_domain_list`, donde `source_domain_list` es una lista de patrones de dominios que pueden seguir CNAMEs en la canonicalización, y `target_domain_list` es una lista de patrones de dominios a los que pueden resolverse.

Por ejemplo, `"*.a.example.com:*.b.example.com,*.c.example.com"` permitirá que los nombres de host que coincidan con `"*.a.example.com"` se canonicalicen a nombres de los dominios `"*.b.example.com"` o `"*.c.example.com"`.

Un único argumento `"none"` hace que no se consideren CNAMEs para la canonicalización. Es el comportamiento por defecto.

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

`ssh(1)` no aceptará certificados de host firmados con algoritmos distintos de los especificados.

### CertificateFile

Especifica un archivo desde el que se lee el certificado del usuario. Para usar este certificado debe proporcionarse por separado la clave privada correspondiente, ya sea mediante una directiva `IdentityFile` o el flag `-i` de `ssh(1)`, mediante `ssh-agent(1)`, o mediante `PKCS11Provider` o `SecurityKeyProvider`.

Los argumentos de `CertificateFile` pueden usar la sintaxis de tilde para referirse al directorio home de un usuario, los tokens descritos en la sección TOKENS y las variables de entorno descritas en la sección VARIABLES DE ENTORNO.

Es posible especificar varios archivos de certificado en los archivos de configuración; estos certificados se probarán en secuencia. Varias directivas `CertificateFile` se añadirán a la lista de certificados usados para la autenticación.

### ChannelTimeout

Especifica si `ssh(1)` debe cerrar los canales inactivos y con qué rapidez. Los tiempos de espera se especifican como uno o más pares “type=interval” separados por espacios, donde “type” debe ser la palabra clave especial “global” o un nombre de tipo de canal de la lista siguiente, que puede contener comodines.

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

### CheckHostIP

Si se establece a `yes`, `ssh(1)` comprobará además la dirección IP del host en el archivo `known_hosts`. Esto le permite detectar si una clave de host cambió por suplantación de DNS (*DNS spoofing*) y, de paso, añadirá las direcciones de los hosts de destino a `~/.ssh/known_hosts`, sin importar el valor de `StrictHostKeyChecking`. Si se establece a `no` (por defecto), no se realiza la comprobación.

### Ciphers

Especifica los cifrados permitidos y su orden de preferencia. Varios cifrados deben separarse con comas. Si la lista indicada empieza por `+`, los cifrados indicados se añadirán al conjunto por defecto en lugar de sustituirlo. Si empieza por `-`, los cifrados indicados (incluidos comodines) se eliminarán del conjunto por defecto. Si empieza por `^`, los cifrados indicados se colocarán al principio del conjunto por defecto.

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

### ClearAllForwardings

Especifica que se eliminen todos los reenvíos de puertos locales, remotos y dinámicos indicados en los archivos de configuración o en la línea de comandos. Es útil sobre todo desde la línea de comandos de `ssh(1)` para eliminar reenvíos definidos en los archivos de configuración, y lo establecen automáticamente `scp(1)` y `sftp(1)`. El argumento debe ser `yes` o `no` (por defecto).

### Compression

Especifica si se usa compresión. El argumento debe ser `yes` o `no` (por defecto).

La compresión se aplica a todo el tráfico que circula por la conexión SSH. Si por la conexión se permite tráfico no confiable (como un reenvío de puerto abierto) junto con tráfico confiable, la compresión puede filtrar información sobre el contenido de la sesión. Por ello, no se recomienda activar la compresión en conexiones que compartan tráfico confiable y no confiable.

### ConnectionAttempts

Especifica el número de intentos (uno por segundo) antes de salir. El argumento debe ser un entero. Puede ser útil en scripts si la conexión falla a veces. El valor por defecto es 1.

### ConnectTimeout

Especifica el tiempo de espera (en segundos) al conectar con el servidor SSH, en lugar de usar el tiempo de espera TCP por defecto del sistema. Se aplica tanto al establecimiento de la conexión como al *handshake* inicial del protocolo SSH y al intercambio de claves.

### ControlMaster

Permite compartir varias sesiones sobre una sola conexión de red. Si se establece a `yes`, `ssh(1)` escuchará conexiones en un socket de control indicado con el argumento `ControlPath`. Otras sesiones pueden conectarse a este socket usando el mismo `ControlPath` con `ControlMaster` a `no` (por defecto). Estas sesiones intentarán reutilizar la conexión de red de la instancia maestra en lugar de iniciar otras nuevas, pero volverán a conectarse normalmente si el socket de control no existe o no está escuchando.

Establecerlo a `ask` hará que `ssh(1)` escuche conexiones de control, pero pida confirmación mediante `ssh-askpass(1)`. Si no se puede abrir el `ControlPath`, `ssh(1)` continuará sin conectarse a una instancia maestra.

Se admite el reenvío de X11 y de `ssh-agent(1)` sobre estas conexiones multiplexadas; sin embargo, el display y el agente reenviados serán los de la conexión maestra, es decir, no es posible reenviar varios displays o agentes.

Dos opciones adicionales permiten la multiplexación oportunista: intentar usar una conexión maestra, pero crear una nueva si aún no existe. Estas opciones son `auto` y `autoask`. La segunda requiere confirmación, como la opción `ask`.

### ControlPath

Especifica la ruta del socket de control usado para compartir conexiones, según se describe en la sección `ControlMaster`, o la cadena `none` para desactivar el uso compartido de conexiones. Los argumentos de `ControlPath` pueden usar la sintaxis de tilde para referirse al directorio home de un usuario, los tokens descritos en la sección TOKENS y las variables de entorno descritas en la sección VARIABLES DE ENTORNO. Se recomienda que cualquier `ControlPath` usado para compartir conexiones de forma oportunista incluya al menos `%h`, `%p` y `%r` (o bien `%C`) y esté en un directorio que otros usuarios no puedan escribir. Así se garantiza que las conexiones compartidas se identifiquen de forma única.

### ControlPersist

Usada junto con `ControlMaster`, especifica que la conexión maestra debe seguir abierta en segundo plano (esperando futuras conexiones de clientes) después de que se haya cerrado la conexión inicial del cliente. Si se establece a `no` (por defecto), la conexión maestra no pasará a segundo plano y se cerrará en cuanto se cierre la conexión inicial del cliente. Si se establece a `yes` o `0`, la conexión maestra permanecerá en segundo plano indefinidamente (hasta que se mate o se cierre mediante un mecanismo como `ssh -O exit`). Si se establece a un tiempo en segundos, o en cualquiera de los formatos documentados en `sshd_config(5)`, la conexión maestra en segundo plano terminará automáticamente tras permanecer inactiva (sin conexiones de clientes) durante ese tiempo.

### DynamicForward

Especifica que un puerto TCP de la máquina local se reenvíe por el canal seguro; después se usa el protocolo de aplicación para determinar a dónde conectarse desde la máquina remota.

El argumento debe ser `[bind_address:]port`. Las direcciones IPv6 se indican entre corchetes. Por defecto, el puerto local se asocia según la configuración de `GatewayPorts`. Sin embargo, puede usarse un `bind_address` explícito para asociar la conexión a una dirección concreta. Un `bind_address` de `localhost` indica que el puerto de escucha se asocie solo para uso local, mientras que una dirección vacía o `*` indica que el puerto debe estar disponible desde todas las interfaces.

Actualmente se admiten los protocolos SOCKS4 y SOCKS5, y `ssh(1)` actuará como servidor SOCKS. Se pueden especificar varios reenvíos, y se pueden añadir más en la línea de comandos. Solo el superusuario puede reenviar puertos privilegiados.

### EnableEscapeCommandline

Activa la opción de línea de comandos en el menú de `EscapeChar` para sesiones interactivas (por defecto `~C`). Por defecto, la línea de comandos está desactivada.

### EnableSSHKeysign

Establecer esta opción a `yes` en el archivo de configuración global del cliente `/etc/ssh/ssh_config` activa el uso del programa auxiliar `ssh-keysign(8)` durante `HostbasedAuthentication`. El argumento debe ser `yes` o `no` (por defecto). Esta opción debe colocarse en la sección no específica de host. Consulta `ssh-keysign(8)` para más información.

### EscapeChar

Establece el carácter de escape (por defecto: `~`). El carácter de escape también puede establecerse en la línea de comandos. El argumento debe ser un único carácter, `^` seguido de una letra, o `none` para desactivar por completo el carácter de escape (haciendo la conexión transparente para datos binarios).

### ExitOnForwardFailure

Especifica si `ssh(1)` debe terminar la conexión si no puede establecer todos los reenvíos de puertos dinámicos, de túnel, locales y remotos solicitados (p. ej. si alguno de los extremos no puede asociarse y escuchar en un puerto indicado). `ExitOnForwardFailure` no se aplica a las conexiones hechas a través de los reenvíos de puertos y, por ejemplo, no hará que `ssh(1)` termine si fallan las conexiones TCP al destino final del reenvío. El argumento debe ser `yes` o `no` (por defecto).

### FingerprintHash

Especifica el algoritmo de hash usado al mostrar las huellas de las claves. Las opciones válidas son `md5` y `sha256` (por defecto).

### ForkAfterAuthentication

Solicita a ssh que pase a segundo plano justo antes de ejecutar el comando. Es útil si ssh va a pedir contraseñas o frases de contraseña, pero el usuario lo quiere en segundo plano. Implica que la opción de configuración `StdinNull` se establezca a “yes”. La forma recomendada de iniciar programas X11 en un sitio remoto es con algo como `ssh -f host xterm`, que equivale a `ssh host xterm` si la opción `ForkAfterAuthentication` está establecida a “yes”.

Si la opción `ExitOnForwardFailure` está a “yes”, un cliente iniciado con `ForkAfterAuthentication` a “yes” esperará a que todos los reenvíos de puertos remotos se hayan establecido correctamente antes de pasar a segundo plano. El argumento de esta palabra clave debe ser `yes` (igual que la opción `-f`) o `no` (por defecto).

### ForwardAgent

Especifica si la conexión con el agente de autenticación (si lo hay) se reenviará a la máquina remota. El argumento puede ser `yes`, `no` (por defecto), una ruta explícita a un socket de agente o el nombre de una variable de entorno (que empiece por `$`) donde encontrar la ruta.

El reenvío del agente debe activarse con precaución. Los usuarios capaces de eludir los permisos de archivo en el host remoto (para el socket de dominio Unix del agente) pueden acceder al agente local a través de la conexión reenviada. Un atacante no puede obtener material de clave del agente, pero sí puede realizar operaciones con las claves que le permitan autenticarse usando las identidades cargadas en el agente.

### ForwardX11

Especifica si las conexiones X11 se redirigirán automáticamente por el canal seguro y se establecerá `DISPLAY`. El argumento debe ser `yes` o `no` (por defecto).

El reenvío X11 debe activarse con precaución. Los usuarios capaces de eludir los permisos de archivo en el host remoto (para la base de datos de autorización X11 del usuario) pueden acceder al display X11 local a través de la conexión reenviada. Un atacante podría entonces realizar actividades como registrar las pulsaciones de teclado si la opción `ForwardX11Trusted` también está activada.

### ForwardX11Timeout

Especifica un tiempo de espera para el reenvío X11 no confiable, con el formato descrito en la sección FORMATOS DE TIEMPO de `sshd_config(5)`. Las conexiones X11 que `ssh(1)` reciba después de ese tiempo se rechazarán. Establecer `ForwardX11Timeout` a cero desactiva el tiempo de espera y permite el reenvío X11 durante toda la vida de la conexión. Por defecto, el reenvío X11 no confiable se desactiva tras veinte minutos.

### ForwardX11Trusted

Si esta opción se establece a `yes`, los clientes X11 remotos tendrán acceso completo al display X11 original.

Si se establece a `no` (por defecto), los clientes X11 remotos se considerarán no confiables y se les impedirá robar o manipular datos de los clientes X11 confiables. Además, el token `xauth(1)` usado para la sesión se configurará para expirar a los 20 minutos. A partir de ese momento se denegará el acceso a los clientes remotos.

Consulta la especificación de la extensión X11 SECURITY para ver todos los detalles de las restricciones impuestas a los clientes no confiables.

### GatewayPorts

Especifica si se permite a hosts remotos conectarse a los puertos locales reenviados. Por defecto, `ssh(1)` asocia los reenvíos de puertos locales a la dirección de loopback. Esto impide que otros hosts remotos se conecten a los puertos reenviados. `GatewayPorts` puede usarse para indicar que ssh asocie los reenvíos de puertos locales a la dirección comodín, permitiendo así que hosts remotos se conecten a los puertos reenviados. El argumento debe ser `yes` o `no` (por defecto).

### GlobalKnownHostsFile

Especifica uno o más archivos, separados por espacios, para usar como base de datos global de claves de host. Por defecto son `/etc/ssh/ssh_known_hosts` y `/etc/ssh/ssh_known_hosts2`.

### GSSAPIAuthentication

Especifica si se permite la autenticación de usuario basada en GSSAPI. El valor por defecto es `no`.

### GSSAPIDelegateCredentials

Reenvía (delega) las credenciales al servidor. El valor por defecto es `no`.

### HashKnownHosts

Indica que `ssh(1)` debe aplicar hash a los nombres de host y direcciones cuando se añaden a `~/.ssh/known_hosts`. `ssh(1)` y `sshd(8)` pueden usar con normalidad estos nombres con hash, pero no revelan visualmente información identificativa si el contenido del archivo se divulga. El valor por defecto es `no`. Los nombres y direcciones existentes en los archivos de hosts conocidos no se convertirán automáticamente, pero pueden convertirse manualmente con `ssh-keygen(1)`.

### HostbasedAcceptedAlgorithms

Especifica los algoritmos de firma que se usarán para la autenticación basada en host, como lista de patrones separada por comas. Si la lista empieza por `+`, los algoritmos indicados se añadirán al conjunto por defecto en lugar de sustituirlo. Si empieza por `-`, los algoritmos indicados (incluidos comodines) se eliminarán del conjunto por defecto. Si empieza por `^`, los algoritmos indicados se colocarán al principio del conjunto por defecto. El valor por defecto de esta opción es:

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

La opción `-Q` de `ssh(1)` puede usarse para listar los algoritmos de firma soportados. Antes esta opción se llamaba `HostbasedKeyTypes`.

### HostbasedAuthentication

Especifica si se intenta la autenticación basada en rhosts junto con autenticación por clave pública. El argumento debe ser `yes` o `no` (por defecto).

### HostKeyAlgorithms

Especifica, en orden de preferencia, los algoritmos de firma de clave de host que el cliente quiere usar. Si la lista empieza por `+`, los algoritmos indicados se añadirán al conjunto por defecto en lugar de sustituirlo. Si empieza por `-`, los algoritmos indicados (incluidos comodines) se eliminarán del conjunto por defecto. Si empieza por `^`, se colocarán al principio del conjunto por defecto. El valor por defecto de esta opción es:

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
sk-ecdsa-sha2-nistp256@openssh.com,
webauthn-sk-ecdsa-sha2-nistp256@openssh.com
sk-ssh-ed25519@openssh.com,
rsa-sha2-512,rsa-sha2-256,
ssh-mldsa44-ed25519
```

Si se conocen claves de host para el host de destino, este valor por defecto se modifica para preferir sus algoritmos.

La lista de algoritmos de firma disponibles también puede obtenerse con `ssh -Q HostKeyAlgorithms`.

### HostKeyAlias

Especifica un alias que debe usarse en lugar del nombre de host real al buscar o guardar la clave de host en los archivos de la base de datos de claves de host y al validar certificados de host. Es útil para tunelizar conexiones SSH o cuando hay varios servidores ejecutándose en un mismo host.

### Hostname

Especifica el nombre de host real en el que iniciar sesión. Puede usarse para definir apodos o abreviaturas de hosts. Los argumentos de `Hostname` aceptan los tokens descritos en la sección TOKENS. También se permiten direcciones IP numéricas (tanto en la línea de comandos como en las especificaciones `Hostname`). Por defecto es el nombre indicado en la línea de comandos.

### IdentitiesOnly

Especifica que `ssh(1)` solo debe usar los archivos de identidad y certificado configurados (ya sean los archivos por defecto, los configurados explícitamente en los archivos `ssh_config` o los pasados en la línea de comandos de `ssh(1)`), aunque `ssh-agent(1)`, un `PKCS11Provider` o un `SecurityKeyProvider` ofrezcan más identidades. El argumento debe ser `yes` o `no` (por defecto). Esta opción está pensada para situaciones en las que ssh-agent ofrece muchas identidades distintas.

### IdentityAgent

Especifica el socket de dominio Unix usado para comunicarse con el agente de autenticación.

Esta opción sobrescribe la variable de entorno `SSH_AUTH_SOCK` y puede usarse para seleccionar un agente concreto. Establecer el nombre del socket a `none` desactiva el uso de un agente de autenticación. Si se indica la cadena `"SSH_AUTH_SOCK"`, la ubicación del socket se leerá de la variable de entorno `SSH_AUTH_SOCK`. En otro caso, si el valor indicado empieza por `$`, se tratará como una variable de entorno que contiene la ubicación del socket.

Los argumentos de `IdentityAgent` pueden usar la sintaxis de tilde para referirse al directorio home de un usuario, los tokens descritos en la sección TOKENS y las variables de entorno descritas en la sección VARIABLES DE ENTORNO.

### IdentityFile

Especifica un archivo desde el que se lee la identidad de autenticación del usuario: ECDSA, ECDSA alojada en autenticador, Ed25519, Ed25519 alojada en autenticador o RSA. También se puede indicar un archivo de clave pública para usar la clave privada correspondiente cargada en `ssh-agent(1)` cuando el archivo de clave privada no está presente localmente. Por defecto son `~/.ssh/id_rsa`, `~/.ssh/id_ecdsa`, `~/.ssh/id_ecdsa_sk`, `~/.ssh/id_ed25519`, `~/.ssh/id_ed25519_sk` y `~/.ssh/id_mldsa44_ed25519`. Además, cualquier identidad representada por el agente de autenticación se usará para autenticarse salvo que `IdentitiesOnly` esté establecido. Si no se han indicado certificados explícitamente con `CertificateFile`, `ssh(1)` intentará cargar la información de certificado desde el archivo cuyo nombre resulta de añadir `-cert.pub` a la ruta del `IdentityFile` indicado.

Los argumentos de `IdentityFile` pueden usar la sintaxis de tilde para referirse al directorio home de un usuario o los tokens descritos en la sección TOKENS. Como alternativa, puede usarse el argumento `none` para indicar que no se cargue ningún archivo de identidad.

Es posible especificar varios archivos de identidad en los archivos de configuración; todas estas identidades se probarán en secuencia. Varias directivas `IdentityFile` se añadirán a la lista de identidades probadas (este comportamiento difiere del de otras directivas de configuración).

`IdentityFile` puede usarse junto con `IdentitiesOnly` para seleccionar qué identidades de un agente se ofrecen durante la autenticación. `IdentityFile` también puede usarse junto con `CertificateFile` para proporcionar cualquier certificado que se necesite para autenticarse con esa identidad.

### IgnoreUnknown

Especifica una lista de patrones de opciones desconocidas que se ignorarán si se encuentran al analizar la configuración. Puede usarse para suprimir errores si `ssh_config` contiene opciones que `ssh(1)` no reconoce. Se recomienda poner `IgnoreUnknown` al principio del archivo de configuración, ya que no se aplicará a las opciones desconocidas que aparezcan antes que ella.

### IPQoS

Especifica el valor DSCP (*Differentiated Services Field Codepoint*) de las conexiones. Los valores aceptados son `af11`, `af12`, `af13`, `af21`, `af22`, `af23`, `af31`, `af32`, `af33`, `af41`, `af42`, `af43`, `cs0`, `cs1`, `cs2`, `cs3`, `cs4`, `cs5`, `cs6`, `cs7`, `ef`, `le`, un valor numérico, o `none` para usar el valor por defecto del sistema operativo. Esta opción admite uno o dos argumentos separados por espacios. Si se indica uno, se usa como clase de paquete de forma incondicional. Si se indican dos, el primero se selecciona automáticamente para las sesiones interactivas y el segundo para las no interactivas. Por defecto es `ef` (*Expedited Forwarding*) para sesiones interactivas y `none` (valor por defecto del sistema operativo) para sesiones no interactivas.

### KbdInteractiveAuthentication

Especifica si se usa la autenticación *keyboard-interactive*. El argumento debe ser `yes` (por defecto) o `no`. `ChallengeResponseAuthentication` es un alias obsoleto de esta opción.

### KbdInteractiveDevices

Especifica la lista de métodos a usar en la autenticación *keyboard-interactive*. Varios nombres de método deben separarse con comas. Por defecto se usa la lista indicada por el servidor. Los métodos disponibles dependen de lo que soporte el servidor. Para un servidor OpenSSH, pueden ser cero o más de: `bsdauth` y `pam`.

### KexAlgorithms

Especifica los algoritmos KEX (intercambio de claves) permitidos y su orden de preferencia. El algoritmo seleccionado será el primero de esta lista que también soporte el servidor. Varios algoritmos deben separarse con comas.

Si la lista empieza por `+`, los algoritmos indicados se añadirán al conjunto por defecto en lugar de sustituirlo. Si empieza por `-`, los algoritmos indicados (incluidos comodines) se eliminarán del conjunto por defecto. Si empieza por `^`, se colocarán al principio del conjunto por defecto.

El valor por defecto es:

```
mlkem768x25519-sha256,
sntrup761x25519-sha512,sntrup761x25519-sha512@openssh.com,
curve25519-sha256,curve25519-sha256@libssh.org,
ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,
diffie-hellman-group-exchange-sha256,
diffie-hellman-group16-sha512,
diffie-hellman-group18-sha512,
diffie-hellman-group14-sha256
```

La lista de algoritmos de intercambio de claves soportados también puede obtenerse con `ssh -Q kex`.

### KnownHostsCommand

Especifica un comando para obtener una lista de claves de host, además de las indicadas en `UserKnownHostsFile` y `GlobalKnownHostsFile`. Este comando se ejecuta después de leer esos archivos. Puede escribir líneas de claves de host en la salida estándar con el mismo formato que los archivos habituales (descrito en la sección VERIFYING HOST KEYS de `ssh(1)`). Los argumentos de `KnownHostsCommand` aceptan los tokens descritos en la sección TOKENS. El comando puede invocarse varias veces por conexión: una al preparar la lista de preferencia de algoritmos de clave de host, otra para obtener la clave de host del nombre de host solicitado y, si `CheckHostIP` está activado, una más para obtener la clave de host que corresponde a la dirección del servidor. Si el comando termina de forma anómala o devuelve un estado de salida distinto de cero, se termina la conexión.

### LocalCommand

Especifica un comando a ejecutar en la máquina local tras conectarse correctamente al servidor. La cadena del comando se extiende hasta el final de la línea y se ejecuta con la shell del usuario. Los argumentos de `LocalCommand` aceptan los tokens descritos en la sección TOKENS.

El comando se ejecuta de forma síncrona y no tiene acceso a la sesión del `ssh(1)` que lo lanzó. No debe usarse para comandos interactivos.

Esta directiva se ignora salvo que se haya activado `PermitLocalCommand`.

### LocalForward

Especifica que un puerto TCP o un socket de dominio Unix de la máquina local se reenvíe por el canal seguro hacia el host y puerto (o socket de dominio Unix) indicados desde la máquina remota. Para un puerto TCP, el primer argumento debe ser `[bind_address:]port` o una ruta de socket de dominio Unix. El segundo argumento es el destino y puede ser `host:hostport` o una ruta de socket de dominio Unix si el host remoto lo admite.

Las direcciones IPv6 se indican entre corchetes.

Si alguno de los argumentos contiene un `/`, ese argumento se interpretará como un socket de dominio Unix (en el host correspondiente) en lugar de un puerto TCP.

Se pueden especificar varios reenvíos, y se pueden añadir más en la línea de comandos. Solo el superusuario puede reenviar puertos privilegiados. Por defecto, el puerto local se asocia según la configuración de `GatewayPorts`. Sin embargo, puede usarse un `bind_address` explícito para asociar la conexión a una dirección concreta. Un `bind_address` de `localhost` indica que el puerto de escucha se asocie solo para uso local, mientras que una dirección vacía o `*` indica que el puerto debe estar disponible desde todas las interfaces. Las rutas de socket de dominio Unix pueden usar los tokens descritos en la sección TOKENS y las variables de entorno descritas en la sección VARIABLES DE ENTORNO.

### LogLevel

Indica el nivel de detalle usado al registrar mensajes de `ssh(1)`. Los valores posibles son: `QUIET`, `FATAL`, `ERROR`, `INFO`, `VERBOSE`, `DEBUG`, `DEBUG1`, `DEBUG2` y `DEBUG3`. El valor por defecto es `INFO`. `DEBUG` y `DEBUG1` son equivalentes. `DEBUG2` y `DEBUG3` especifican cada uno niveles más altos de salida detallada.

### LogVerbose

Especifica una o más excepciones a `LogLevel`. Una excepción consiste en una o más listas de patrones que coinciden con el archivo fuente, la función y el número de línea para los que forzar un registro detallado. Por ejemplo, un patrón de excepción como:

```
kex.c:*:1000,*:kex_exchange_identification():*,packet.c:*
```

activaría el registro detallado para la línea 1000 de `kex.c`, todo lo de la función `kex_exchange_identification()` y todo el código del archivo `packet.c`. Esta opción está pensada para depuración y por defecto no hay excepciones activadas.

### MACs

Especifica los algoritmos MAC (código de autenticación de mensajes) en orden de preferencia. El algoritmo MAC se usa para proteger la integridad de los datos. Varios algoritmos deben separarse con comas. Si la lista empieza por `+`, los algoritmos indicados se añadirán al conjunto por defecto en lugar de sustituirlo. Si empieza por `-`, los algoritmos indicados (incluidos comodines) se eliminarán del conjunto por defecto. Si empieza por `^`, se colocarán al principio del conjunto por defecto.

Los algoritmos que contienen "-etm" calculan el MAC después del cifrado (*encrypt-then-mac*). Se consideran más seguros y se recomienda su uso.

El valor por defecto es:

```
umac-64-etm@openssh.com,umac-128-etm@openssh.com,
hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,
hmac-sha1-etm@openssh.com,
umac-64@openssh.com,umac-128@openssh.com,
hmac-sha2-256,hmac-sha2-512,hmac-sha1
```

La lista de algoritmos MAC disponibles también puede obtenerse con `ssh -Q mac`.

### NoHostAuthenticationForLocalhost

Desactiva la autenticación del host para localhost (direcciones de loopback). El argumento debe ser `yes` o `no` (por defecto).

### NumberOfPasswordPrompts

Especifica el número de solicitudes de contraseña antes de rendirse. El argumento debe ser un entero. El valor por defecto es 3.

### ObscureKeystrokeTiming

Especifica si `ssh(1)` debe intentar ocultar los tiempos entre pulsaciones de teclas a observadores pasivos del tráfico de red. Si está activado, en las sesiones interactivas `ssh(1)` enviará las pulsaciones a intervalos fijos de unas decenas de milisegundos y enviará paquetes de pulsaciones falsas durante un tiempo después de que se deje de teclear. El argumento debe ser `yes`, `no` o un especificador de intervalo de la forma `interval:milliseconds` (p. ej. `interval:80` para 80 milisegundos). Por defecto se ocultan las pulsaciones con un intervalo de paquetes de 20 ms. Los intervalos más pequeños producirán tasas más altas de paquetes de pulsaciones falsas.

### PasswordAuthentication

Especifica si se usa la autenticación por contraseña. El argumento debe ser `yes` (por defecto) o `no`.

### PermitLocalCommand

Permite la ejecución de comandos locales mediante la opción `LocalCommand` o con la secuencia de escape `!command` en `ssh(1)`. El argumento debe ser `yes` o `no` (por defecto).

### PermitRemoteOpen

Especifica los destinos a los que se permite el reenvío de puertos TCP remoto cuando `RemoteForward` se usa como proxy SOCKS. La especificación de reenvío debe tener una de las siguientes formas:

```
PermitRemoteOpen host:port
PermitRemoteOpen IPv4_addr:port
PermitRemoteOpen [IPv6_addr]:port
```

Se pueden especificar varios reenvíos separándolos con espacios. El argumento `any` elimina todas las restricciones y permite cualquier petición de reenvío. El argumento `none` prohíbe todas las peticiones de reenvío. El comodín `*` puede usarse como host o puerto para permitir todos los hosts o puertos, respectivamente. En otro caso, no se hace coincidencia de patrones ni resolución de direcciones sobre los nombres indicados.

### PKCS11Provider

Especifica qué proveedor PKCS#11 usar, o `none` para indicar que no se use ninguno (por defecto). El argumento es la ruta a la biblioteca compartida PKCS#11 que `ssh(1)` debe usar para comunicarse con un token PKCS#11 que proporciona claves para la autenticación del usuario.

### Port

Especifica el número de puerto al que conectarse en el host remoto. El valor por defecto es 22.

### PreferredAuthentications

Especifica el orden en que el cliente debe probar los métodos de autenticación. Esto permite al cliente preferir un método (p. ej. `keyboard-interactive`) sobre otro (p. ej. `password`). El valor por defecto es:

```
gssapi-with-mic,hostbased,publickey,
keyboard-interactive,password
```

### ProxyCommand

Especifica el comando a usar para conectarse al servidor. La cadena del comando se extiende hasta el final de la línea y se ejecuta con la directiva `exec` de la shell del usuario para evitar que quede un proceso de shell residual.

Los argumentos de `ProxyCommand` aceptan los tokens descritos en la sección TOKENS. El comando puede ser básicamente cualquier cosa, y debe leer de su entrada estándar y escribir en su salida estándar. En última instancia debe conectarse a un servidor `sshd(8)` que se ejecute en alguna máquina, o ejecutar `sshd -i` en algún sitio. La gestión de claves de host se hará usando el `Hostname` del host al que se conecta (por defecto, el nombre escrito por el usuario). Establecer el comando a `none` desactiva por completo esta opción. `CheckHostIP` no está disponible para conexiones que usan un comando proxy.

Esta directiva es útil junto con `nc(1)` y su soporte de proxy. Por ejemplo, la siguiente directiva se conectaría a través de un proxy HTTP en 192.0.2.0:

```
ProxyCommand /usr/bin/nc -X connect -x 192.0.2.0:8080 %h %p
```

### ProxyJump

Especifica uno o más proxies de salto, como `[user@]host[:port]` o como URI ssh. Varios proxies pueden separarse con comas y se recorrerán en secuencia. Establecer esta opción hará que `ssh(1)` se conecte al host objetivo estableciendo primero una conexión `ssh(1)` con el host `ProxyJump` indicado y, desde ahí, un reenvío TCP hasta el objetivo final. Establecer el host a `none` desactiva por completo esta opción.

Esta opción compite con la opción `ProxyCommand`: la que se especifique primero impedirá que tengan efecto las apariciones posteriores de la otra.

Ten en cuenta también que, en general, la configuración del host de destino (ya se indique por línea de comandos o en el archivo de configuración) no se aplica a los hosts de salto. Debe usarse `~/.ssh/config` si se necesita una configuración específica para los hosts de salto.

### ProxyUseFdpass

Especifica que `ProxyCommand` devolverá a `ssh(1)` un descriptor de archivo ya conectado en lugar de seguir ejecutándose y pasando datos. El valor por defecto es `no`.

### PubkeyAcceptedAlgorithms

Especifica los algoritmos de firma que se usarán para la autenticación por clave pública, como lista de patrones separada por comas. Si la lista empieza por `+`, los algoritmos que siguen se añadirán al valor por defecto en lugar de sustituirlo. Si empieza por `-`, los algoritmos indicados (incluidos comodines) se eliminarán del conjunto por defecto. Si empieza por `^`, se colocarán al principio del conjunto por defecto. El valor por defecto de esta opción es:

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

### PubkeyAuthentication

Especifica si se intenta la autenticación por clave pública. El argumento debe ser `yes` (por defecto), `no`, `unbound` o `host-bound`. Las dos últimas opciones activan la autenticación por clave pública desactivando o activando, respectivamente, la extensión del protocolo OpenSSH de autenticación ligada al host (*host-bound*), necesaria para el reenvío restringido de `ssh-agent(1)`.

### RefuseConnection

Permite que el archivo de configuración rechace una conexión. Si se especifica esta opción, `ssh(1)` terminará inmediatamente antes de intentar conectar con el host remoto, mostrará un mensaje de error que contiene el argumento de esta palabra clave y devolverá un estado de salida distinto de cero. Puede ser útil para mostrar recordatorios o advertencias al usuario mediante `ssh_config`.

### RekeyLimit

Especifica la cantidad máxima de datos que pueden transmitirse o recibirse antes de renegociar la clave de sesión, seguida opcionalmente del tiempo máximo que puede pasar antes de renegociarla. El primer argumento se indica en bytes y puede llevar el sufijo `K`, `M` o `G` para indicar kilobytes, megabytes o gigabytes, respectivamente. El valor por defecto está entre `1G` y `4G`, según el cifrado. El segundo valor, opcional, se indica en segundos y puede usar cualquiera de las unidades documentadas en la sección FORMATOS DE TIEMPO de `sshd_config(5)`. El valor por defecto de `RekeyLimit` es `default none`, lo que significa que la renegociación se hace tras enviar o recibir la cantidad de datos por defecto del cifrado, y no se hace renegociación basada en tiempo.

### RemoteCommand

Especifica un comando a ejecutar en la máquina remota tras conectarse correctamente al servidor. La cadena del comando se extiende hasta el final de la línea y se ejecuta con la shell del usuario. Los argumentos de `RemoteCommand` aceptan los tokens descritos en la sección TOKENS.

### RemoteForward

Especifica que un puerto TCP o un socket de dominio Unix de la máquina remota se reenvíe por el canal seguro. El puerto remoto puede reenviarse a un host y puerto, o a un socket de dominio Unix, indicados desde la máquina local, o puede actuar como proxy SOCKS 4/5 que permite a un cliente remoto conectarse a destinos arbitrarios desde la máquina local. El primer argumento es la especificación de escucha y puede ser `[bind_address:]port` o, si el host remoto lo admite, una ruta de socket de dominio Unix. Si se reenvía a un destino concreto, el segundo argumento debe ser `host:hostport` o una ruta de socket de dominio Unix; si no se indica argumento de destino, el reenvío remoto se establecerá como proxy SOCKS. Al actuar como proxy SOCKS, el destino de la conexión puede restringirse con `PermitRemoteOpen`.

Las direcciones IPv6 se indican entre corchetes.

Si alguno de los argumentos contiene un `/`, ese argumento se interpretará como un socket de dominio Unix (en el host correspondiente) en lugar de un puerto TCP.

Se pueden especificar varios reenvíos, y se pueden añadir más en la línea de comandos. Los puertos privilegiados solo pueden reenviarse si se inicia sesión como root en la máquina remota. Las rutas de socket de dominio Unix pueden usar los tokens descritos en la sección TOKENS y las variables de entorno descritas en la sección VARIABLES DE ENTORNO.

Si el argumento de puerto es 0, el puerto de escucha se asignará dinámicamente en el servidor y se comunicará al cliente en tiempo de ejecución.

Si no se indica `bind_address`, por defecto solo se asocia a las direcciones de loopback. Si `bind_address` es `*` o una cadena vacía, se solicita que el reenvío escuche en todas las interfaces. Indicar un `bind_address` remoto solo funcionará si la opción `GatewayPorts` del servidor está activada (consulta `sshd_config(5)`).

### RequestTTY

Especifica si se solicita una pseudo-tty para la sesión. El argumento puede ser: `no` (nunca solicitar TTY), `yes` (solicitar siempre TTY cuando la entrada estándar es una TTY), `force` (solicitar siempre TTY) o `auto` (solicitar TTY al abrir una sesión de inicio). Esta opción equivale a los flags `-t` y `-T` de `ssh(1)`.

### RequiredRSASize

Especifica el tamaño mínimo de clave RSA (en bits) que `ssh(1)` aceptará. Las claves de autenticación de usuario más pequeñas que este límite se ignorarán. Los servidores que presenten claves de host más pequeñas que este límite harán que se termine la conexión. El valor por defecto es 1024 bits. Este límite solo puede aumentarse respecto al valor por defecto.

### RevokedHostKeys

Especifica las claves públicas de host revocadas. Las claves listadas en este archivo se rechazarán para la autenticación del host. Si este archivo no existe o no es legible, se rechazará la autenticación de todos los hosts. Las claves pueden indicarse como un archivo de texto con una clave pública por línea, o como una Lista de Revocación de Claves (KRL) de OpenSSH generada por `ssh-keygen(1)`. Para más información sobre las KRL, consulta la sección KEY REVOCATION LISTS de `ssh-keygen(1)`. Los argumentos de `RevokedHostKeys` pueden usar la sintaxis de tilde para referirse al directorio home de un usuario, los tokens descritos en la sección TOKENS y las variables de entorno descritas en la sección VARIABLES DE ENTORNO.

### SecurityKeyProvider

Especifica la ruta a una biblioteca que se usará al cargar cualquier clave alojada en autenticadores FIDO, en lugar del soporte USB HID integrado que se usa por defecto.

Si el valor indicado empieza por `$`, se tratará como una variable de entorno que contiene la ruta de la biblioteca.

### SendEnv

Especifica qué variables del `environ(7)` local deben enviarse al servidor. El servidor también debe admitirlo y estar configurado para aceptar esas variables de entorno. La variable de entorno `TERM` siempre se envía cuando se solicita una pseudo-terminal, ya que el protocolo la requiere. Consulta `AcceptEnv` en `sshd_config(5)` para ver cómo configurar el servidor. Las variables se indican por nombre, que puede contener comodines. Varias variables pueden separarse con espacios o repartirse en varias directivas `SendEnv`.

Consulta PATRONES para más información sobre patrones.

Es posible eliminar nombres de variables `SendEnv` definidos previamente anteponiendo `-` a los patrones. Por defecto no se envía ninguna variable de entorno.

### ServerAliveCountMax

Establece el número de mensajes *server alive* (ver abajo) que pueden enviarse sin que `ssh(1)` reciba ningún mensaje de vuelta del servidor. Si se alcanza este umbral mientras se envían mensajes *server alive*, ssh se desconectará del servidor, terminando la sesión. Es importante señalar que el uso de mensajes *server alive* es muy distinto de `TCPKeepAlive` (ver abajo). Los mensajes *server alive* se envían por el canal cifrado y, por tanto, no pueden falsificarse. La opción de keepalive TCP que activa `TCPKeepAlive` sí puede falsificarse. El mecanismo *server alive* es valioso cuando el cliente o el servidor necesitan saber cuándo una conexión ha dejado de responder.

El valor por defecto es 3. Si, por ejemplo, `ServerAliveInterval` (ver abajo) se establece a 15 y `ServerAliveCountMax` se deja en su valor por defecto, cuando el servidor deje de responder ssh se desconectará tras unos 45 segundos.

### ServerAliveInterval

Establece un intervalo, en segundos, tras el cual, si no se han recibido datos del servidor, `ssh(1)` enviará un mensaje por el canal cifrado para solicitar una respuesta del servidor. El valor por defecto es 0, que indica que no se enviarán estos mensajes al servidor.

### SessionType

Puede usarse para solicitar la invocación de un subsistema en el sistema remoto, o para impedir por completo la ejecución de un comando remoto. Esto último es útil para solo reenviar puertos. El argumento debe ser `none` (igual que la opción `-N`), `subsystem` (igual que la opción `-s`) o `default` (ejecución de shell o comando).

### SetEnv

Especifica directamente una o más variables de entorno y su contenido para enviarlas al servidor, en la forma “NAME=VALUE”. Igual que con `SendEnv`, con la excepción de la variable `TERM`, el servidor debe estar preparado para aceptar la variable de entorno.

“VALUE” puede usar los tokens descritos en la sección TOKENS y las variables de entorno descritas en la sección VARIABLES DE ENTORNO.

### StdinNull

Redirige stdin desde `/dev/null` (en realidad, impide leer de stdin). Debe usarse esta opción o la equivalente `-n` cuando ssh se ejecuta en segundo plano. El argumento debe ser `yes` (igual que la opción `-n`) o `no` (por defecto).

### StreamLocalBindMask

Establece la máscara octal de modo de creación de archivos (umask) usada al crear un archivo de socket de dominio Unix para el reenvío de puertos local o remoto. Solo se usa para el reenvío de puertos hacia un archivo de socket de dominio Unix.

El valor por defecto es 0177, que crea un archivo de socket de dominio Unix legible y escribible solo por el propietario. No todos los sistemas operativos respetan el modo de archivo en los archivos de socket de dominio Unix.

### StreamLocalBindUnlink

Especifica si se elimina un archivo de socket de dominio Unix existente para el reenvío de puertos local o remoto antes de crear uno nuevo. Si el archivo de socket ya existe y `StreamLocalBindUnlink` no está activado, ssh no podrá reenviar el puerto al archivo de socket de dominio Unix. Solo se usa para el reenvío de puertos hacia un archivo de socket de dominio Unix.

El argumento debe ser `yes` o `no` (por defecto).

### StrictHostKeyChecking

Si este flag se establece a `yes`, `ssh(1)` nunca añadirá automáticamente claves de host al archivo `~/.ssh/known_hosts` y se negará a conectar con hosts cuya clave de host haya cambiado. Esto ofrece la máxima protección frente a ataques de intermediario (MITM), aunque puede resultar molesto cuando el archivo `/etc/ssh/ssh_known_hosts` está mal mantenido o cuando se conecta a menudo con hosts nuevos. Esta opción obliga al usuario a añadir manualmente todos los hosts nuevos.

Si se establece a `accept-new`, ssh añadirá automáticamente las claves de host nuevas al archivo `known_hosts` del usuario, pero no permitirá conexiones a hosts cuyas claves hayan cambiado. Si se establece a `no` u `off`, ssh añadirá automáticamente las claves de host nuevas a los archivos de hosts conocidos del usuario y permitirá continuar las conexiones a hosts con claves cambiadas, con algunas restricciones. Si se establece a `ask` (por defecto), las claves nuevas solo se añadirán a los archivos de hosts conocidos del usuario después de que este haya confirmado que es lo que realmente quiere, y ssh se negará a conectar con hosts cuya clave de host haya cambiado. En todos los casos, las claves de host de los hosts conocidos se verificarán automáticamente.

### SyslogFacility

Indica el código de *facility* usado al registrar mensajes de `ssh(1)`. Los valores posibles son: `DAEMON`, `USER`, `AUTH`, `LOCAL0`, `LOCAL1`, `LOCAL2`, `LOCAL3`, `LOCAL4`, `LOCAL5`, `LOCAL6`, `LOCAL7`. El valor por defecto es `USER`.

### TCPKeepAlive

Especifica si el sistema debe enviar mensajes keepalive en los sockets TCP que ha abierto. Si se envían, se podrá detectar antes una caída de la conexión o el fallo de uno de los extremos. Sin embargo, esto significa que las conexiones pueden terminar si sufren una interrupción transitoria.

El argumento debe ser `transport` para activar los mensajes keepalive TCP en la conexión de transporte SSH con el servidor, `yes` como alias de `transport`, `all` para activarlos en todos los sockets abiertos por `ssh(1)`, incluidos los creados para reenvío X11 o de puertos, o `no` para desactivarlos. El valor por defecto es `transport`.

Consulta también `ServerAliveInterval` para un mecanismo más robusto de detección de fallos de conexión que funciona a nivel del protocolo SSH.

### Tag

Especifica un nombre de etiqueta de configuración que luego puede usar una directiva `Match` para seleccionar un bloque de configuración.

### Tunnel

Solicita el reenvío de un dispositivo `tun(4)` entre el cliente y el servidor. El argumento debe ser `yes`, `point-to-point` (capa 3), `ethernet` (capa 2) o `no` (por defecto). Indicar `yes` solicita el modo de túnel por defecto, que es `point-to-point`.

### TunnelDevice

Especifica los dispositivos `tun(4)` a abrir en el cliente (`local_tun`) y en el servidor (`remote_tun`).

El argumento debe ser `local_tun[:remote_tun]`. Los dispositivos pueden indicarse por ID numérico o con la palabra clave `any`, que usa el siguiente dispositivo de túnel disponible. Si no se indica `remote_tun`, por defecto es `any`. El valor por defecto es `any:any`.

### UpdateHostKeys

Especifica si `ssh(1)` debe aceptar notificaciones de claves de host adicionales enviadas por el servidor tras completar la autenticación, y añadirlas a `UserKnownHostsFile`. El argumento debe ser `yes`, `no` o `ask`. Esta opción permite conocer claves de host alternativas de un servidor y facilita la rotación ordenada de claves, al permitir que un servidor envíe claves públicas de sustitución antes de eliminar las antiguas.

Las claves de host adicionales solo se aceptan si la clave usada para autenticar al host ya era de confianza o fue aceptada explícitamente por el usuario, si el host se autenticó mediante `UserKnownHostsFile` (es decir, no `GlobalKnownHostsFile`) y si el host se autenticó con una clave simple y no con un certificado.

`UpdateHostKeys` está activado por defecto si el usuario no ha cambiado el valor por defecto de `UserKnownHostsFile` y no ha activado `VerifyHostKeyDNS`; en otro caso, `UpdateHostKeys` se establecerá a `no`.

Si `UpdateHostKeys` se establece a `ask`, se pide al usuario que confirme las modificaciones del archivo `known_hosts`. Actualmente la confirmación es incompatible con `ControlPersist`, y se desactivará si este está activado.

Actualmente solo el `sshd(8)` de OpenSSH 6.8 o superior admite la extensión de protocolo "hostkeys@openssh.com", usada para informar al cliente de todas las claves de host del servidor.

### User

Especifica el usuario con el que iniciar sesión. Puede ser útil cuando se usa un nombre de usuario distinto en distintas máquinas. Evita tener que acordarse de indicar el nombre de usuario en la línea de comandos. Los argumentos de `User` pueden usar los tokens descritos en la sección TOKENS (salvo `%r` y `%C`) y las variables de entorno descritas en la sección VARIABLES DE ENTORNO.

### UserKnownHostsFile

Especifica uno o más archivos, separados por espacios, para usar como base de datos de claves de host del usuario. Cada nombre de archivo puede usar la notación de tilde para referirse al directorio home del usuario, los tokens descritos en la sección TOKENS y las variables de entorno descritas en la sección VARIABLES DE ENTORNO. Un valor `none` hace que `ssh(1)` ignore cualquier archivo de hosts conocidos específico del usuario. Por defecto son `~/.ssh/known_hosts` y `~/.ssh/known_hosts2`.

### VerifyHostKeyDNS

Especifica si se verifica la clave remota usando DNS y registros de recurso SSHFP. Si se establece a `yes`, el cliente confiará implícitamente en las claves que coincidan con una huella segura obtenida de DNS. Las huellas no seguras se tratarán como si la opción estuviera establecida a `ask`. Si se establece a `ask`, se mostrará información sobre la coincidencia de la huella, pero el usuario seguirá teniendo que confirmar las claves de host nuevas según la opción `StrictHostKeyChecking`. El valor por defecto es `no`.

Consulta también VERIFYING HOST KEYS en `ssh(1)`.

### VersionAddendum

Especifica opcionalmente un texto adicional que se añadirá al banner del protocolo SSH que envía el cliente al conectar. El valor por defecto es `none`.

### VisualHostKey

Si este flag se establece a `yes`, al iniciar sesión y para claves de host desconocidas se imprime una representación en arte ASCII de la huella de la clave del host remoto, además de la cadena de la huella. Si se establece a `no` (por defecto), no se imprime ninguna huella al iniciar sesión y solo se imprimirá la cadena de la huella para las claves de host desconocidas.

### WarnWeakCrypto

Controla si se advierte al usuario cuando los algoritmos criptográficos negociados para la conexión son débiles o no recomendados. Las advertencias pueden desactivarse apagando una advertencia concreta o desactivándolas todas. Las advertencias sobre conexiones que no usan un intercambio de claves post-cuántico pueden desactivarse con el flag `no-pq-kex`. `no` desactiva todas las advertencias. El valor por defecto, equivalente a `yes`, es activar todas las advertencias.

### XAuthLocation

Especifica la ruta completa del programa `xauth(1)`. Por defecto es `/usr/X11R6/bin/xauth`.

## PATRONES

Un patrón consiste en cero o más caracteres que no sean espacios, `*` (un comodín que coincide con cero o más caracteres) o `?` (un comodín que coincide con exactamente un carácter). Por ejemplo, para especificar un conjunto de declaraciones para cualquier host del conjunto de dominios ".co.uk", podría usarse el siguiente patrón:

```
Host *.co.uk
```

El siguiente patrón coincidiría con cualquier host del rango de red 192.168.0.[0-9]:

```
Host 192.168.0.?
```

Una lista de patrones (*pattern-list*) es una lista de patrones separada por comas. Los patrones dentro de una lista pueden negarse anteponiéndoles un signo de exclamación (`!`). Por ejemplo, para permitir que una clave se use desde cualquier parte de una organización salvo desde el grupo "dialup", podría usarse la siguiente entrada (en `authorized_keys`):

```
from="!*.dialup.example.com,*.example.com"
```

Una coincidencia negada nunca producirá por sí sola un resultado positivo. Por ejemplo, intentar hacer coincidir "host3" con la siguiente lista de patrones fallará:

```
from="!host1,!host2"
```

La solución es incluir un término que produzca una coincidencia positiva, como un comodín:

```
from="!host1,!host2,*"
```

## TOKENS

Los argumentos de algunas palabras clave pueden usar tokens, que se expanden en tiempo de ejecución. Los tokens se expanden sin entrecomillar ni escapar los caracteres de la shell. Es responsabilidad del usuario asegurarse de que son seguros en el contexto en que se usan.

Los tokens admitidos en `ssh_config` son:

| Token | Significado |
|---|---|
| `%%` | Un `%` literal. |
| `%C` | Hash de `%l%h%p%r%j`. |
| `%d` | Directorio home del usuario local. |
| `%f` | La huella de la clave de host del servidor. |
| `%H` | El nombre de host o dirección de `known_hosts` que se está buscando. |
| `%h` | El nombre del host remoto. |
| `%I` | Una cadena que describe el motivo de una ejecución de `KnownHostsCommand`: `ADDRESS` al buscar un host por dirección (solo si `CheckHostIP` está activado), `HOSTNAME` al buscar por nombre de host, u `ORDER` al preparar la lista de preferencia de algoritmos de clave de host para el host de destino. |
| `%i` | El ID del usuario local. |
| `%j` | El contenido de la opción `ProxyJump`, o la cadena vacía si no está definida. |
| `%K` | La clave de host codificada en base64. |
| `%k` | El alias de la clave de host si se ha especificado; si no, el nombre de host remoto original dado en la línea de comandos. |
| `%L` | El nombre del host local. |
| `%l` | El nombre del host local, incluido el nombre de dominio. |
| `%n` | El nombre de host remoto original, tal como se dio en la línea de comandos. |
| `%p` | El puerto remoto. |
| `%r` | El nombre de usuario remoto. |
| `%T` | La interfaz de red local `tun(4)` o `tap(4)` asignada si se solicitó reenvío de túnel, o "NONE" en otro caso. |
| `%t` | El tipo de la clave de host del servidor, p. ej. `ssh-ed25519`. |
| `%u` | El nombre de usuario local. |

`CertificateFile`, `ControlPath`, `IdentityAgent`, `IdentityFile`, `Include`, `KnownHostsCommand`, `LocalForward`, `Match exec`, `RemoteCommand`, `RemoteForward`, `RevokedHostKeys`, `UserKnownHostsFile` y `VersionAddendum` aceptan los tokens `%%`, `%C`, `%d`, `%h`, `%i`, `%j`, `%k`, `%L`, `%l`, `%n`, `%p`, `%r` y `%u`.

`KnownHostsCommand` acepta además los tokens `%f`, `%H`, `%I`, `%K` y `%t`.

`Hostname` acepta los tokens `%%` y `%h`.

`LocalCommand` acepta todos los tokens.

`ProxyCommand` y `ProxyJump` aceptan los tokens `%%`, `%h`, `%n`, `%p` y `%r`.

Algunas de estas directivas construyen comandos que se ejecutan mediante la shell. Como `ssh(1)` no filtra ni escapa los caracteres con significado especial en los comandos de shell (p. ej. las comillas), es responsabilidad del usuario asegurarse de que los argumentos pasados a `ssh(1)` no contienen tales caracteres y de que los tokens se entrecomillan adecuadamente al usarlos.

## VARIABLES DE ENTORNO

Los argumentos de algunas palabras clave pueden expandirse en tiempo de ejecución a partir de variables de entorno del cliente, encerrándolas en `${}`; por ejemplo, `${HOME}/.ssh` se referiría al directorio `.ssh` del usuario. Si una variable de entorno indicada no existe, se devolverá un error y se ignorará el valor de esa palabra clave.

Las palabras clave `CertificateFile`, `ControlPath`, `IdentityAgent`, `IdentityFile`, `Include`, `KnownHostsCommand` y `UserKnownHostsFile` admiten variables de entorno. Las palabras clave `LocalForward` y `RemoteForward` solo admiten variables de entorno para rutas de socket de dominio Unix.

## ARCHIVOS

**`~/.ssh/config`**
: Archivo de configuración del usuario. Su formato se describe más arriba. Lo usa el cliente SSH. Por el riesgo de abuso, este archivo debe tener permisos estrictos: lectura/escritura para el usuario y sin permiso de escritura para los demás.

**`/etc/ssh/ssh_config`**
: Archivo de configuración global del sistema. Proporciona valores por defecto para los parámetros que no se especifican en el archivo de configuración del usuario, y para los usuarios que no tienen archivo de configuración. Este archivo debe ser legible por todos.

## VÉASE TAMBIÉN

`ssh(1)`

## AUTORES

OpenSSH es un derivado de la versión original y libre ssh 1.2.12 de Tatu Ylonen. Aaron Campbell, Bob Beck, Markus Friedl, Niels Provos, Theo de Raadt y Dug Song eliminaron muchos errores, volvieron a añadir funciones más recientes y crearon OpenSSH. Markus Friedl contribuyó el soporte para las versiones 1.5 y 2.0 del protocolo SSH.
