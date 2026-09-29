# SSHD(8)

## NOMBRE

**sshd** — demonio de OpenSSH

## SINOPSIS

```
sshd [-46DdeGiqTtV] [-C connection_spec] [-c host_certificate_file] [-E log_file] [-f config_file] [-g login_grace_time] [-h host_key_file] [-o option] [-p port] [-u len]
```

## DESCRIPCIÓN

**sshd** (OpenSSH Daemon) es el programa demonio de `ssh(1)`. Proporciona comunicaciones cifradas y seguras entre dos hosts no confiables a través de una red insegura.

**sshd** escucha las conexiones de los clientes. Normalmente se inicia en el arranque desde `/etc/rc`. Crea (fork) un nuevo demonio por cada conexión entrante. Los demonios hijos se encargan del intercambio de claves, el cifrado, la autenticación, la ejecución de comandos y el intercambio de datos.

**sshd** puede configurarse con opciones de línea de comandos o con un archivo de configuración (por defecto `sshd_config(5)`); las opciones de línea de comandos sobrescriben los valores del archivo de configuración. **sshd** vuelve a leer su archivo de configuración cuando recibe la señal de *hangup*, `SIGHUP`, ejecutándose a sí mismo con el nombre y las opciones con que fue iniciado, p. ej. `/usr/sbin/sshd`.

Las opciones son las siguientes:

**`-4`**
: Obliga a **sshd** a usar solo direcciones IPv4.

**`-6`**
: Obliga a **sshd** a usar solo direcciones IPv6.

**`-C connection_spec`**
: Especifica los parámetros de conexión a usar en el modo de prueba extendido `-T`. Si se indica, cualquier directiva `Match` del archivo de configuración que aplique se aplicará antes de escribir la configuración en la salida estándar. Los parámetros se indican como pares `keyword=value` en cualquier orden, ya sea con varias opciones `-C` o como lista separada por comas. Las palabras clave son “addr”, “user”, “host”, “laddr”, “lport” y “rdomain”, y corresponden respectivamente a la dirección de origen, el usuario, el nombre de host de origen resuelto, la dirección local, el puerto local y el dominio de enrutamiento. Además, se puede indicar el flag “invalid-user” (que no lleva valor) para simular una conexión de un nombre de usuario no reconocido.

**`-c host_certificate_file`**
: Especifica la ruta a un archivo de certificado para identificar a **sshd** durante el intercambio de claves. El archivo de certificado debe corresponder a un archivo de clave de host indicado con la opción `-h` o con la directiva de configuración `HostKey`.

**`-D`**
: Con esta opción, **sshd** no se desvincula de la terminal ni se convierte en demonio. Esto facilita la monitorización de **sshd**.

**`-d`**
: Modo de depuración. El servidor envía una salida de depuración detallada a la salida de error estándar y no se pone en segundo plano. El servidor tampoco hará `fork(2)` y solo procesará una conexión. Esta opción está pensada únicamente para depurar el servidor. Varias opciones `-d` aumentan el nivel de depuración. El máximo es 3.

**`-E log_file`**
: Añade los registros de depuración a *log_file* en lugar de al registro del sistema.

**`-e`**
: Escribe los registros de depuración en la salida de error estándar en lugar de en el registro del sistema.

**`-f config_file`**
: Especifica el nombre del archivo de configuración. Por defecto es `/etc/ssh/sshd_config`. **sshd** se niega a arrancar si no hay archivo de configuración.

**`-G`**
: Analiza e imprime el archivo de configuración. Comprueba la validez del archivo, muestra la configuración efectiva por stdout y termina. Opcionalmente pueden aplicarse reglas `Match` indicando los parámetros de conexión con una o más opciones `-C`.

**`-g login_grace_time`**
: Establece el tiempo de gracia para que los clientes se autentiquen (120 segundos por defecto). Si el cliente no consigue autenticar al usuario en ese número de segundos, el servidor desconecta y termina. Un valor de cero indica sin límite.

**`-h host_key_file`**
: Especifica un archivo desde el que se lee una clave de host. Esta opción debe indicarse si **sshd** no se ejecuta como root (ya que los archivos de clave de host normales solo suelen ser legibles por root). Por defecto son `/etc/ssh/ssh_host_ecdsa_key`, `/etc/ssh/ssh_host_ed25519_key`, `/etc/ssh/ssh_host_mldsa44_ed25519_key` y `/etc/ssh/ssh_host_rsa_key`. Es posible tener varios archivos de clave de host para los distintos algoritmos.

**`-i`**
: Indica que **sshd** se está ejecutando desde `inetd(8)`.

**`-o option`**
: Puede usarse para indicar opciones en el formato del archivo de configuración. Es útil para opciones que no tienen un flag de línea de comandos propio. Para todos los detalles de las opciones y sus valores, consulta `sshd_config(5)`.

**`-p port`**
: Especifica el puerto en el que el servidor escucha conexiones (22 por defecto). Se permiten varias opciones de puerto. Los puertos indicados en el archivo de configuración con la opción `Port` se ignoran cuando se indica un puerto por línea de comandos. Los puertos indicados con la opción `ListenAddress` sobrescriben a los de la línea de comandos.

**`-q`**
: Modo silencioso. No se envía nada al registro del sistema. Normalmente se registran el inicio, la autenticación y la terminación de cada conexión.

**`-T`**
: Modo de prueba extendido. Comprueba la validez del archivo de configuración, muestra la configuración efectiva por stdout y termina. Opcionalmente pueden aplicarse reglas `Match` indicando los parámetros de conexión con una o más opciones `-C`. Es similar al flag `-G`, pero incluye las pruebas adicionales que realiza el flag `-t`.

**`-t`**
: Modo de prueba. Solo comprueba la validez del archivo de configuración y la coherencia de las claves. Es útil para actualizar **sshd** de forma fiable, ya que las opciones de configuración pueden cambiar.

**`-u len`**
: Especifica el tamaño del campo de la estructura utmp que guarda el nombre del host remoto. Si el nombre de host resuelto es más largo que *len*, se usará en su lugar el valor decimal con puntos. Esto permite identificar de forma única a hosts con nombres muy largos que desbordarían este campo. `-u0` indica que en el archivo utmp solo deben ponerse direcciones decimales con puntos. `-u0` también puede usarse para evitar que **sshd** haga consultas DNS, salvo que el mecanismo de autenticación o la configuración lo requieran. Los mecanismos de autenticación que pueden requerir DNS incluyen `HostbasedAuthentication` y el uso de una opción `from="pattern-list"` en un archivo de claves. Las opciones de configuración que requieren DNS incluyen usar un patrón `USER@HOST` en `AllowUsers` o `DenyUsers`.

**`-V`**
: Muestra el número de versión y termina.

## AUTENTICACIÓN

El demonio SSH de OpenSSH solo admite la versión 2 del protocolo SSH. Cada host tiene una clave propia (clave de host) que se usa para identificarlo. Cada vez que un cliente se conecta, el demonio responde con su clave pública de host. El cliente compara la clave de host con su propia base de datos para verificar que no ha cambiado. El secreto hacia adelante (*forward secrecy*) se consigue mediante un acuerdo de claves Diffie-Hellman, que produce una clave de sesión compartida. El resto de la sesión se cifra con un cifrado simétrico. El cliente elige el algoritmo de cifrado entre los que ofrece el servidor. Además, la integridad de la sesión se garantiza mediante un código de autenticación de mensajes (MAC) criptográfico.

Por último, servidor y cliente entran en un diálogo de autenticación. El cliente intenta autenticarse mediante autenticación basada en host, por clave pública, por desafío-respuesta o por contraseña.

Si el cliente se autentica con éxito, se entra en un diálogo de preparación de la sesión. En este momento el cliente puede solicitar cosas como asignar una pseudo-tty, reenviar conexiones X11, reenviar conexiones TCP o reenviar la conexión del agente de autenticación por el canal seguro.

Después, el cliente solicita una shell interactiva o la ejecución de un comando no interactivo, que **sshd** ejecutará mediante la shell del usuario con su opción `-c`. Ambos lados entran entonces en modo sesión. En este modo, cualquiera de los dos puede enviar datos en cualquier momento, y esos datos se reenvían desde/hacia la shell o el comando en el lado servidor y la terminal del usuario en el lado cliente.

Cuando el programa del usuario termina y se han cerrado todas las conexiones X11 y demás reenvíos, el servidor envía al cliente el estado de salida del comando y ambos lados terminan.

## PROCESO DE INICIO DE SESIÓN

Cuando un usuario inicia sesión correctamente, **sshd** hace lo siguiente:

1. Si el inicio de sesión es en una tty y no se ha indicado ningún comando, imprime la hora del último inicio de sesión y `/etc/motd` (salvo que se impida en el archivo de configuración o mediante `~/.hushlogin`; consulta la sección ARCHIVOS).
2. Si el inicio de sesión es en una tty, registra la hora de inicio.
3. Comprueba `/etc/nologin`; si existe, imprime su contenido y termina (salvo para root).
4. Pasa a ejecutarse con los privilegios normales del usuario.
5. Configura el entorno básico.
6. Lee el archivo `~/.ssh/environment`, si existe y si se permite a los usuarios cambiar su entorno. Consulta la opción `PermitUserEnvironment` en `sshd_config(5)`.
7. Cambia al directorio home del usuario.
8. Si existe `~/.ssh/rc` y la opción `PermitUserRC` de `sshd_config(5)` está activada, lo ejecuta; si no, si existe `/etc/ssh/sshrc`, lo ejecuta; en otro caso, ejecuta `xauth(1)`. A los archivos “rc” se les pasa por la entrada estándar el protocolo de autenticación X11 y la cookie. Consulta SSHRC, más abajo.
9. Ejecuta la shell o el comando del usuario. Todos los comandos se ejecutan bajo la shell de inicio del usuario, según la base de datos de contraseñas del sistema.

## SSHRC

Si existe el archivo `~/.ssh/rc`, `sh(1)` lo ejecuta tras leer los archivos de entorno pero antes de iniciar la shell o el comando del usuario. No debe producir ninguna salida por stdout; debe usarse stderr en su lugar. Si se está usando el reenvío X11, recibirá el par "proto cookie" por su entrada estándar (y `DISPLAY` en su entorno). El script debe llamar a `xauth(1)`, porque **sshd** no ejecutará xauth automáticamente para añadir las cookies X11.

El propósito principal de este archivo es ejecutar las rutinas de inicialización que puedan ser necesarias antes de que el directorio home del usuario sea accesible; AFS es un ejemplo concreto de este tipo de entorno.

Probablemente este archivo contendrá algo de código de inicialización seguido de algo similar a:

```sh
if read proto cookie && [ -n "$DISPLAY" ]; then
	if [ `echo $DISPLAY | cut -c1-10` = 'localhost:' ]; then
		# X11UseLocalhost=yes
		echo add unix:`echo $DISPLAY |
		    cut -c11-` $proto $cookie
	else
		# X11UseLocalhost=no
		echo add $DISPLAY $proto $cookie
	fi | xauth -q -
fi
```

Si este archivo no existe, se ejecuta `/etc/ssh/sshrc`, y si tampoco existe, se usa xauth para añadir la cookie.

## FORMATO DEL ARCHIVO AUTHORIZED_KEYS

`AuthorizedKeysFile` especifica los archivos que contienen las claves públicas para la autenticación por clave pública; si no se especifica esta opción, por defecto son `~/.ssh/authorized_keys` y `~/.ssh/authorized_keys2`. Cada línea del archivo contiene una clave (las líneas vacías y las que empiezan por `#` se ignoran como comentarios). Las claves públicas constan de los siguientes campos separados por espacios: opciones, tipo de clave, clave codificada en base64 y comentario. El campo de opciones es opcional. Los tipos de clave soportados son:

- `sk-ecdsa-sha2-nistp256@openssh.com`
- `ecdsa-sha2-nistp256`
- `ecdsa-sha2-nistp384`
- `ecdsa-sha2-nistp521`
- `sk-ssh-ed25519@openssh.com`
- `ssh-ed25519`
- `ssh-mldsa44-ed25519`
- `ssh-rsa`

El campo de comentario no se usa para nada (pero puede ser útil para que el usuario identifique la clave).

Ten en cuenta que las líneas de este archivo pueden tener varios cientos de bytes (por el tamaño de la codificación de la clave pública), hasta un límite de 8 kilobytes, lo que permite claves RSA de hasta 16 kilobits. No querrás escribirlas a mano; en su lugar, copia el archivo `id_ecdsa.pub`, `id_ecdsa_sk.pub`, `id_ed25519.pub`, `id_ed25519_sk.pub`, `id_mldsa44_ed25519.pub` o `id_rsa.pub` y edítalo.

**sshd** exige un tamaño mínimo de módulo RSA de 1024 bits.

Las opciones (si están presentes) consisten en especificaciones separadas por comas. No se permiten espacios, salvo dentro de comillas dobles. Se admiten las siguientes especificaciones de opción (las palabras clave de las opciones no distinguen mayúsculas de minúsculas):

**`agent-forwarding`**
: Activa el reenvío del agente de autenticación previamente desactivado por la opción `restrict`.

**`cert-authority`**
: Indica que la clave listada es una autoridad de certificación (CA) de confianza para validar certificados firmados para la autenticación de usuarios.

  Los certificados pueden codificar restricciones de acceso similares a estas opciones de clave. Si hay tanto restricciones del certificado como opciones de la clave, se aplica la combinación más restrictiva de ambas.

**`command="command"`**
: Indica que el comando se ejecuta cada vez que se usa esta clave para autenticarse. Se ignora el comando indicado por el usuario (si lo hay). El comando se ejecuta en una pty si el cliente la solicita; si no, se ejecuta sin tty. Si se necesita un canal limpio de 8 bits, no se debe solicitar una pty o se debe especificar `no-pty`. Se puede incluir una comilla en el comando escapándola con una barra invertida.

  Esta opción puede ser útil para restringir ciertas claves públicas a realizar solo una operación concreta. Un ejemplo sería una clave que permite copias de seguridad remotas pero nada más. Ten en cuenta que el cliente puede especificar reenvío TCP y/o X11 salvo que se prohíba explícitamente, p. ej. con la opción de clave `restrict`.

  El comando indicado originalmente por el cliente está disponible en la variable de entorno `SSH_ORIGINAL_COMMAND`. Esta opción se aplica a la ejecución de shell, comando o subsistema. Además, este comando puede quedar sustituido por una directiva `ForceCommand` de `sshd_config(5)`.

  Si se especifica un comando y además hay un comando forzado incrustado en un certificado usado para la autenticación, el certificado solo se aceptará si ambos comandos son idénticos.

**`environment="NAME=value"`**
: Indica que la cadena debe añadirse al entorno al iniciar sesión con esta clave. Las variables de entorno definidas así sobrescriben otros valores de entorno por defecto. Se permiten varias opciones de este tipo. El procesamiento del entorno está desactivado por defecto y se controla con la opción `PermitUserEnvironment`.

**`expiry-time="timespec"`**
: Especifica un momento a partir del cual la clave no se aceptará. Puede indicarse como fecha `YYYYMMDD[Z]` o como hora `YYYYMMDDHHMM[SS][Z]`. Las fechas y horas se interpretan en la zona horaria del sistema, salvo que lleven el sufijo `Z`, en cuyo caso se interpretan en UTC.

**`from="pattern-list"`**
: Indica que, además de la autenticación por clave pública, el nombre canónico del host remoto o su dirección IP debe estar presente en la lista de patrones separada por comas. Consulta PATTERNS en `ssh_config(5)` para más información sobre patrones.

  Además de la coincidencia con comodines aplicable a nombres de host o direcciones, una estrofa `from` puede hacer coincidir direcciones IP con la notación CIDR dirección/longitud de máscara.

  El propósito de esta opción es aumentar opcionalmente la seguridad: la autenticación por clave pública por sí sola no confía en la red, ni en los servidores de nombres, ni en nada (salvo la clave); sin embargo, si alguien roba la clave de algún modo, esta permite a un intruso iniciar sesión desde cualquier parte del mundo. Esta opción adicional dificulta el uso de una clave robada (habría que comprometer también los servidores de nombres y/o los routers, además de la clave).

**`no-agent-forwarding`**
: Prohíbe el reenvío del agente de autenticación cuando se usa esta clave para autenticarse.

**`no-port-forwarding`**
: Prohíbe el reenvío TCP cuando se usa esta clave para autenticarse. Cualquier petición de reenvío de puertos del cliente devolverá un error. Puede usarse, p. ej., junto con la opción `command`.

**`no-pty`**
: Impide la asignación de tty (una petición para asignar una pty fallará).

**`no-user-rc`**
: Desactiva la ejecución de `~/.ssh/rc`.

**`no-X11-forwarding`**
: Prohíbe el reenvío X11 cuando se usa esta clave para autenticarse. Cualquier petición de reenvío X11 del cliente devolverá un error.

**`permitlisten="[host:]port"`**
: Limita el reenvío de puertos remoto con la opción `-R` de `ssh(1)` para que solo pueda escuchar en el host (opcional) y puerto indicados. Las direcciones IPv6 se indican entre corchetes. Se pueden aplicar varias opciones `permitlisten` separadas por comas. Los nombres de host pueden incluir comodines, según se describe en la sección PATTERNS de `ssh_config(5)`. Un puerto `*` coincide con cualquier puerto. La configuración de `GatewayPorts` puede restringir aún más las direcciones de escucha. Ten en cuenta que `ssh(1)` enviará el nombre de host “localhost” si no se indicó un host de escucha al solicitar el reenvío, y que este nombre se trata de forma distinta a las direcciones localhost explícitas “127.0.0.1” y “::1”.

**`permitopen="host:port"`**
: Limita el reenvío de puertos local con la opción `-L` de `ssh(1)` para que solo pueda conectarse al host y puerto indicados. Las direcciones IPv6 se indican entre corchetes. Se pueden aplicar varias opciones `permitopen` separadas por comas. No se hace coincidencia de patrones ni resolución de nombres sobre los nombres de host indicados; deben ser nombres de host y/o direcciones literales. Un puerto `*` coincide con cualquier puerto.

**`port-forwarding`**
: Activa el reenvío de puertos previamente desactivado por la opción `restrict`.

**`principals="principals"`**
: En una línea `cert-authority`, especifica los principales permitidos para la autenticación por certificado, como lista separada por comas. Al menos un nombre de la lista debe aparecer en la lista de principales del certificado para que este se acepte. Esta opción se ignora en las claves que no estén marcadas como firmantes de certificados de confianza con la opción `cert-authority`.

**`pty`**
: Permite la asignación de tty previamente desactivada por la opción `restrict`.

**`no-touch-required`**
: No exige demostrar la presencia del usuario en las firmas hechas con esta clave. Esta opción solo tiene sentido para los algoritmos de autenticador FIDO `ecdsa-sk` y `ed25519-sk`.

**`verify-required`**
: Exige que las firmas hechas con esta clave atestigüen que se verificó al usuario, p. ej. mediante un PIN. Esta opción solo tiene sentido para los algoritmos de autenticador FIDO `ecdsa-sk` y `ed25519-sk`.

**`restrict`**
: Activa todas las restricciones, es decir, desactiva el reenvío de puertos, del agente y X11, así como la asignación de PTY y la ejecución de `~/.ssh/rc`. Si en el futuro se añaden nuevas capacidades de restricción a los archivos authorized_keys, se incluirán en este conjunto.

**`tunnel="n"`**
: Fuerza un dispositivo `tun(4)` concreto en el servidor. Sin esta opción, se usará el siguiente dispositivo disponible si el cliente solicita un túnel.

**`user-rc`**
: Activa la ejecución de `~/.ssh/rc` previamente desactivada por la opción `restrict`.

**`X11-forwarding`**
: Permite el reenvío X11 previamente desactivado por la opción `restrict`.

Un ejemplo de archivo authorized_keys:

```
# Se permiten comentarios al inicio de línea. Se permiten líneas en blanco.
# Clave simple, sin restricciones
ssh-rsa ...
# Comando forzado, desactiva PTY y todo el reenvío
restrict,command="dump /home" ssh-rsa ...
# Restricción de los destinos de reenvío de ssh -L
permitopen="192.0.2.1:80",permitopen="192.0.2.2:25" ssh-rsa ...
# Restricción de los puntos de escucha de reenvío de ssh -R
permitlisten="localhost:8080",permitlisten="[::1]:22000" ssh-rsa ...
# Configuración para reenvío de túnel
tunnel="0",command="sh /etc/netstart tun0" ssh-rsa ...
# Anula la restricción para permitir asignación de PTY
restrict,pty,command="nethack" ssh-rsa ...
# Permite clave FIDO sin requerir toque
no-touch-required sk-ecdsa-sha2-nistp256@openssh.com ...
# Exige verificación del usuario (p. ej. PIN o biometría) para la clave FIDO
verify-required sk-ecdsa-sha2-nistp256@openssh.com ...
# Confía en la clave de la CA, permite FIDO sin toque si el certificado lo solicita
cert-authority,no-touch-required,principals="user_a" ssh-rsa ...
```

## FORMATO DEL ARCHIVO SSH_KNOWN_HOSTS

Los archivos `/etc/ssh/ssh_known_hosts` y `~/.ssh/known_hosts` contienen las claves públicas de todos los hosts conocidos. El archivo global debe prepararlo el administrador (es opcional), y el archivo por usuario se mantiene automáticamente: cada vez que el usuario se conecta a un host desconocido, su clave se añade al archivo del usuario.

Cada línea de estos archivos contiene los siguientes campos: marcador (opcional), nombres de host, tipo de clave, clave codificada en base64 y comentario. Los campos se separan con espacios.

El marcador es opcional, pero si está presente debe ser “@cert-authority”, para indicar que la línea contiene una clave de autoridad de certificación (CA), o “@revoked”, para indicar que la clave de la línea está revocada y no debe aceptarse nunca. En cada línea de clave solo debe usarse un marcador.

Los nombres de host son una lista de patrones separada por comas (`*` y `?` actúan como comodines); cada patrón se compara por turnos con el nombre de host. Cuando **sshd** autentica a un cliente, por ejemplo al usar `HostbasedAuthentication`, este será el nombre de host canónico del cliente. Cuando `ssh(1)` autentica a un servidor, será el nombre de host que dio el usuario, el valor de `HostkeyAlias` de `ssh(1)` si se especificó, o el nombre de host canónico del servidor si se usó la opción `CanonicalizeHostname` de `ssh(1)`.

Un patrón también puede ir precedido de `!` para indicar negación: si el nombre de host coincide con un patrón negado, no se acepta (en esa línea) aunque coincida con otro patrón de la misma línea. Un nombre de host o dirección puede ir opcionalmente entre corchetes `[` y `]`, seguido de `:` y un número de puerto no estándar.

Como alternativa, los nombres de host pueden guardarse en forma de hash, lo que oculta nombres y direcciones si el contenido del archivo llega a divulgarse. Los nombres con hash empiezan por el carácter `|`. En una línea solo puede aparecer un nombre con hash, y no se le puede aplicar ninguno de los operadores de negación o comodín anteriores.

El tipo de clave y la clave en base64 se toman directamente de la clave de host; pueden obtenerse, por ejemplo, de `/etc/ssh/ssh_host_rsa_key.pub`. El campo de comentario opcional se extiende hasta el final de la línea y no se usa.

Las líneas que empiezan por `#` y las vacías se ignoran como comentarios.

Al realizar la autenticación del host, esta se acepta si alguna línea coincidente tiene la clave adecuada: ya sea una que coincide exactamente o, si el servidor presentó un certificado, la clave de la autoridad de certificación que firmó ese certificado. Para que una clave se considere de confianza como autoridad de certificación, debe usar el marcador “@cert-authority” descrito arriba.

El archivo de hosts conocidos también permite marcar claves como revocadas, por ejemplo cuando se sabe que la clave privada asociada ha sido robada. Las claves revocadas se indican incluyendo el marcador “@revoked” al principio de la línea, y nunca se aceptan para autenticación ni como autoridades de certificación; en su lugar, `ssh(1)` emitirá una advertencia al encontrarlas.

Está permitido (pero no se recomienda) tener varias líneas o distintas claves de host para los mismos nombres. Esto ocurrirá inevitablemente cuando se pongan en el archivo formas cortas de nombres de host de distintos dominios. Es posible que los archivos contengan información contradictoria; la autenticación se acepta si se encuentra información válida en cualquiera de los archivos.

Ten en cuenta que las líneas de estos archivos suelen tener cientos de caracteres, y desde luego no querrás escribir las claves de host a mano. Es mejor generarlas con un script, con `ssh-keyscan(1)` o tomando, por ejemplo, `/etc/ssh/ssh_host_rsa_key.pub` y añadiendo los nombres de host al principio. `ssh-keygen(1)` también ofrece algunas funciones básicas de edición automática de `~/.ssh/known_hosts`, como eliminar los hosts que coinciden con un nombre y convertir todos los nombres a su representación con hash.

Un ejemplo de archivo ssh_known_hosts:

```
# Se permiten comentarios al inicio de línea
cvs.example.net,192.0.2.10 ssh-rsa AAAA1234.....=
# Un nombre de host con hash
|1|JfKTdBh7rNbXkVAQCRp4OQoPfmI=|USECr3SWf1JUPsms5AqfD5QfxkM= ssh-rsa
AAAA1234.....=
# Una clave revocada
@revoked * ssh-rsa AAAAB5W...
# Una clave de CA, aceptada para cualquier host en *.mydomain.com o *.mydomain.org
@cert-authority *.mydomain.org,*.mydomain.com ssh-rsa AAAAB5W...
```

## ARCHIVOS

**`~/.hushlogin`**
: Se usa para suprimir la impresión de la hora del último inicio de sesión y de `/etc/motd`, si `PrintLastLog` y `PrintMotd`, respectivamente, están activados. No suprime la impresión del banner indicado por `Banner`.

**`~/.rhosts`**
: Se usa para la autenticación basada en host (consulta `ssh(1)` para más información). En algunas máquinas este archivo puede necesitar ser legible por todos si el directorio home del usuario está en una partición NFS, porque **sshd** lo lee como root. Además, este archivo debe pertenecer al usuario y no debe tener permisos de escritura para nadie más. El permiso recomendado en la mayoría de máquinas es lectura/escritura para el usuario y sin acceso para los demás.

**`~/.shosts`**
: Se usa exactamente igual que `.rhosts`, pero permite la autenticación basada en host sin permitir el inicio de sesión con rlogin/rsh.

**`~/.ssh/`**
: Este directorio es la ubicación por defecto de toda la información de configuración y autenticación específica del usuario. No hay un requisito general de mantener en secreto todo el contenido de este directorio, pero los permisos recomendados son lectura/escritura/ejecución para el usuario y sin acceso para los demás.

**`~/.ssh/authorized_keys`**
: Lista las claves públicas (ECDSA, Ed25519, RSA) que pueden usarse para iniciar sesión como este usuario. El formato de este archivo se describe más arriba. Su contenido no es muy sensible, pero los permisos recomendados son lectura/escritura para el usuario y sin acceso para los demás.

  Si este archivo, el directorio `~/.ssh` o el directorio home del usuario pueden ser escritos por otros usuarios, el archivo podría ser modificado o sustituido por usuarios no autorizados. En ese caso, **sshd** no permitirá usarlo salvo que la opción `StrictModes` esté establecida a “no”.

**`~/.ssh/environment`**
: Este archivo se carga en el entorno al iniciar sesión (si existe). Solo puede contener líneas vacías, líneas de comentario (que empiezan por `#`) y líneas de asignación de la forma `name=value`. El archivo solo debe poder escribirlo el usuario; no es necesario que otros puedan leerlo. El procesamiento del entorno está desactivado por defecto y se controla con la opción `PermitUserEnvironment`.

**`~/.ssh/known_hosts`**
: Contiene una lista de claves de host de todos los hosts en los que el usuario ha iniciado sesión y que no están ya en la lista global de claves de hosts conocidos del sistema. El formato de este archivo se describe más arriba. Solo debe poder escribirlo root/el propietario y puede, aunque no es necesario, ser legible por todos.

**`~/.ssh/rc`**
: Contiene rutinas de inicialización que se ejecutan antes de que el directorio home del usuario sea accesible. Solo debe poder escribirlo el usuario, y no es necesario que otros puedan leerlo.

**`/etc/hosts.equiv`**
: Este archivo es para la autenticación basada en host (consulta `ssh(1)`). Solo debe poder escribirlo root.

**`/etc/moduli`**
: Contiene los grupos Diffie-Hellman usados por el método de intercambio de claves "Diffie-Hellman Group Exchange". El formato del archivo se describe en `moduli(5)`. Si no se encuentran grupos utilizables en este archivo, se usarán grupos internos fijos.

**`/etc/motd`**
: Consulta `motd(5)`.

**`/etc/nologin`**
: Si este archivo existe, **sshd** no deja iniciar sesión a nadie salvo a root. El contenido del archivo se muestra a cualquiera que intente iniciar sesión, y se rechazan las conexiones que no sean de root. El archivo debe ser legible por todos.

**`/etc/shosts.equiv`**
: Se usa exactamente igual que `hosts.equiv`, pero permite la autenticación basada en host sin permitir el inicio de sesión con rlogin/rsh.

**`/etc/ssh/ssh_host_ecdsa_key`**<br>
**`/etc/ssh/ssh_host_ed25519_key`**<br>
**`/etc/ssh/ssh_host_mldsa44_ed25519_key`**<br>
**`/etc/ssh/ssh_host_rsa_key`**
: Estos archivos contienen las partes privadas de las claves de host. Solo deben pertenecer a root, ser legibles solo por root y no ser accesibles por otros. **sshd** no arranca si estos archivos son accesibles por el grupo o por todos.

**`/etc/ssh/ssh_host_ecdsa_key.pub`**<br>
**`/etc/ssh/ssh_host_ed25519_key.pub`**<br>
**`/etc/ssh/ssh_host_mldsa44_ed25519_key.pub`**<br>
**`/etc/ssh/ssh_host_rsa_key.pub`**
: Estos archivos contienen las partes públicas de las claves de host. Deben ser legibles por todos pero solo escribibles por root. Su contenido debe corresponder a las respectivas partes privadas. En realidad no se usan para nada; se proporcionan por comodidad para que el usuario pueda copiar su contenido a los archivos de hosts conocidos. Se crean con `ssh-keygen(1)`.

**`/etc/ssh/ssh_known_hosts`**
: Lista global del sistema de claves de hosts conocidos. El administrador del sistema debe prepararla para que contenga las claves públicas de host de todas las máquinas de la organización. El formato de este archivo se describe más arriba. Solo debe poder escribirlo root/el propietario y debe ser legible por todos.

**`/etc/ssh/sshd_config`**
: Contiene los datos de configuración de **sshd**. El formato del archivo y las opciones de configuración se describen en `sshd_config(5)`.

**`/etc/ssh/sshrc`**
: Similar a `~/.ssh/rc`, puede usarse para especificar de forma global inicializaciones específicas de la máquina en el momento del inicio de sesión. Solo debe poder escribirlo root y debe ser legible por todos.

**`/var/empty`**
: Directorio `chroot(2)` que usa **sshd** durante la separación de privilegios en la fase previa a la autenticación. No debe contener archivos, debe pertenecer a root y no debe ser escribible por el grupo ni por todos.

**`/var/run/sshd.pid`**
: Contiene el ID de proceso del **sshd** que escucha conexiones (si hay varios demonios ejecutándose a la vez en distintos puertos, contiene el ID del último que se inició). Su contenido no es sensible; puede ser legible por todos.

## VÉASE TAMBIÉN

`scp(1)`, `sftp(1)`, `ssh(1)`, `ssh-add(1)`, `ssh-agent(1)`, `ssh-keygen(1)`, `ssh-keyscan(1)`, `chroot(2)`, `login.conf(5)`, `moduli(5)`, `sshd_config(5)`, `inetd(8)`, `sftp-server(8)`

## AUTORES

OpenSSH es un derivado de la versión original y libre ssh 1.2.12 de Tatu Ylonen. Aaron Campbell, Bob Beck, Markus Friedl, Niels Provos, Theo de Raadt y Dug Song eliminaron muchos errores, volvieron a añadir funciones más recientes y crearon OpenSSH. Markus Friedl contribuyó el soporte para las versiones 1.5 y 2.0 del protocolo SSH. Niels Provos y Markus Friedl contribuyeron el soporte para la separación de privilegios.
