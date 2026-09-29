# SSH-KEYSCAN(1)

## NOMBRE

**ssh-keyscan** — obtiene las claves públicas SSH de servidores

## SINOPSIS

```
ssh-keyscan [-46cDHqv] [-f file] [-O option] [-p port] [-T timeout] [-t type] [host | addrlist namelist]
```

## DESCRIPCIÓN

**ssh-keyscan** es una utilidad para recopilar las claves públicas de host SSH de varios hosts. Se diseñó para ayudar a construir y verificar archivos `ssh_known_hosts`, cuyo formato está documentado en `sshd(8)`. **ssh-keyscan** ofrece una interfaz mínima adecuada para usarse desde scripts de shell y perl.

**ssh-keyscan** usa E/S de sockets no bloqueante para contactar con tantos hosts como sea posible en paralelo, por lo que es muy eficiente. Las claves de un dominio de 1.000 hosts pueden recopilarse en decenas de segundos, incluso si algunos de esos hosts están caídos o no ejecutan `sshd(8)`. Para escanear no se necesita acceso de inicio de sesión a las máquinas escaneadas, y el proceso de escaneo tampoco implica cifrado alguno.

Los hosts a escanear pueden especificarse por nombre de host, dirección o rango de red CIDR (p. ej. `192.168.16/28`). Si se especifica un rango de red, se escanearán todas las direcciones de ese rango.

**ssh-keyscan** no puede verificar la autenticidad de las claves de host que obtiene, y un atacante en la red capaz de interceptar el tráfico podría sustituirlas por sus propias claves para suplantar servidores. Por ello, la salida de **ssh-keyscan** debe verificarse por un canal alternativo (*out of band*), o usarse directamente para la autenticación de hosts solo si la red es de confianza.

Las opciones son las siguientes:

**`-4`**
: Obliga a **ssh-keyscan** a usar solo direcciones IPv4.

**`-6`**
: Obliga a **ssh-keyscan** a usar solo direcciones IPv6.

**`-c`**
: Solicita certificados a los hosts objetivo en lugar de claves simples.

**`-D`**
: Imprime las claves encontradas como registros DNS SSHFP. Por defecto se imprimen en un formato utilizable como archivo `known_hosts` de `ssh(1)`.

**`-f file`**
: Lee hosts o pares “addrlist namelist” desde *file*, uno por línea. Si se indica `-` en lugar de un nombre de archivo, **ssh-keyscan** leerá de la entrada estándar. Los nombres leídos de un archivo deben empezar por una dirección, nombre de host o rango de red CIDR a escanear. Las direcciones y nombres de host pueden ir seguidos opcionalmente de alias (nombres o direcciones) separados por comas, que se copiarán a la salida. Por ejemplo:

  ```
  192.168.11.0/24
  10.20.1.1
  happy.example.org
  10.0.0.1,sad.example.org
  ```

**`-H`**
: Aplica hash a todos los nombres de host y direcciones de la salida. `ssh(1)` y `sshd(8)` pueden usar los nombres con hash con normalidad, pero no revelan información identificativa si el contenido del archivo llega a divulgarse.

**`-O option`**
: Especifica una opción clave/valor. Actualmente solo se admite una opción:

  **`hashalg=algorithm`**
  : Selecciona el algoritmo de hash usado al imprimir registros SSHFP con el flag `-D`. Los algoritmos válidos son “sha1” y “sha256”. Por defecto se imprimen ambos.

**`-p port`**
: Se conecta a *port* en el host remoto.

**`-q`**
: Modo silencioso: no imprime el nombre del host servidor ni los banners en comentarios.

**`-T timeout`**
: Establece el tiempo de espera para los intentos de conexión. Si han pasado *timeout* segundos desde que se inició la conexión a un host o desde la última vez que se leyó algo de ese host, la conexión se cierra y el host se considera no disponible. El valor por defecto es 5 segundos.

**`-t type`**
: Especifica el tipo de clave a obtener de los hosts escaneados. Los valores posibles son “ecdsa”, “ed25519”, “ecdsa-sk”, “ed25519-sk” o “rsa”. Se pueden indicar varios valores separándolos con comas. Por defecto se obtienen todos los tipos anteriores.

**`-v`**
: Modo detallado (*verbose*): imprime mensajes de depuración sobre el progreso.

Si se construye un archivo `ssh_known_hosts` con **ssh-keyscan** sin verificar las claves, los usuarios quedarán expuestos a ataques de intermediario (*man in the middle*). Por otro lado, si el modelo de seguridad admite ese riesgo, **ssh-keyscan** puede ayudar a detectar archivos de claves manipulados o ataques de intermediario que hayan empezado después de crear el archivo `ssh_known_hosts`.

## ARCHIVOS

`/etc/ssh/ssh_known_hosts`

## EJEMPLOS

Imprimir la clave de host RSA de la máquina *hostname*:

```
$ ssh-keyscan -t rsa hostname
```

Escanear un rango de red, imprimiendo todos los tipos de clave soportados:

```
$ ssh-keyscan 192.168.0.64/25
```

Encontrar todos los hosts del archivo `ssh_hosts` que tengan claves nuevas o distintas de las del archivo ordenado `ssh_known_hosts`:

```
$ ssh-keyscan -t rsa,ecdsa,ed25519 -f ssh_hosts | \
	sort -u - ssh_known_hosts | diff ssh_known_hosts -
```

## VÉASE TAMBIÉN

`ssh(1)`, `sshd(8)`

*Using DNS to Securely Publish Secure Shell (SSH) Key Fingerprints*, RFC 4255, 2006.

## AUTORES

David Mazieres <dm@lcs.mit.edu> escribió la versión inicial, y Wayne Davison <wayned@users.sourceforge.net> añadió soporte para la versión 2 del protocolo.
