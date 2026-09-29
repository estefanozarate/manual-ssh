# SSH-ADD(1)

## NOMBRE

**ssh-add** — añade identidades de clave privada al agente de autenticación de OpenSSH

## SINOPSIS

```
ssh-add [-CcDdKkLlNPqvXx] [-E fingerprint_hash] [-H hostkey_file] [-h destination_constraint] [-S provider] [-t life] [file ...]

ssh-add -s pkcs11 [-Cv] [certificate ...]

ssh-add -e pkcs11

ssh-add -T pubkey ...

ssh-add -Q
```

## DESCRIPCIÓN

**ssh-add** añade identidades de clave privada al agente de autenticación, `ssh-agent(1)`. Si se ejecuta sin argumentos, añade los archivos `~/.ssh/id_rsa`, `~/.ssh/id_ecdsa`, `~/.ssh/id_ecdsa_sk`, `~/.ssh/id_ed25519`, `~/.ssh/id_ed25519_sk` y `~/.ssh/id_mldsa44_ed25519`. Tras cargar una clave privada, **ssh-add** intentará cargar la información de certificado correspondiente desde el archivo cuyo nombre resulta de añadir `-cert.pub` al nombre del archivo de la clave privada. Se pueden indicar nombres de archivo alternativos en la línea de comandos.

Si algún archivo requiere frase de contraseña (*passphrase*), **ssh-add** se la pide al usuario. La frase se lee desde la tty del usuario. Si se indican varios archivos de identidad, **ssh-add** reintenta con la última frase introducida.

Para que **ssh-add** funcione, el agente de autenticación debe estar en ejecución y la variable de entorno `SSH_AUTH_SOCK` debe contener el nombre de su socket.

Las opciones son las siguientes:

**`-C`**
: Al cargar claves en el agente o borrarlas de él, procesa solo certificados y omite las claves simples.

**`-c`**
: Indica que las identidades añadidas deben requerir confirmación antes de usarse para autenticarse. La confirmación la realiza `ssh-askpass(1)`. Una confirmación exitosa se indica con un código de salida cero de `ssh-askpass(1)`, no con texto introducido en el diálogo.

**`-D`**
: Borra todas las identidades del agente.

**`-d`**
: En lugar de añadir identidades, las elimina del agente. Si **ssh-add** se ha ejecutado sin argumentos, se eliminarán las claves de las identidades por defecto y sus certificados correspondientes. En caso contrario, la lista de argumentos se interpretará como una lista de rutas a archivos de clave pública que indican las claves y certificados a eliminar del agente. Si no se encuentra una clave pública en una ruta dada, **ssh-add** añadirá `.pub` y lo reintentará. Si la lista de argumentos consiste en `-`, **ssh-add** leerá de la entrada estándar las claves públicas a eliminar.

**`-E fingerprint_hash`**
: Especifica el algoritmo de hash usado al mostrar las huellas de las claves. Las opciones válidas son “md5” y “sha256”. El valor por defecto es “sha256”.

**`-e pkcs11`**
: Elimina las claves proporcionadas por la biblioteca compartida PKCS#11 *pkcs11*.

**`-H hostkey_file`**
: Especifica un archivo de hosts conocidos donde buscar claves de host al usar claves con restricción de destino mediante el flag `-h`. Esta opción puede indicarse varias veces para buscar en varios archivos. Si no se indica ninguno, **ssh-add** usará los archivos de hosts conocidos por defecto de `ssh_config(5)`: `~/.ssh/known_hosts`, `~/.ssh/known_hosts2`, `/etc/ssh/ssh_known_hosts` y `/etc/ssh/ssh_known_hosts2`.

**`-h destination_constraint`**
: Al añadir claves, las restringe para que solo puedan usarse a través de hosts concretos o hacia destinos concretos.

  Las restricciones de destino de la forma `[user@]dest-hostname` permiten usar la clave solo desde el host de origen (el que ejecuta `ssh-agent(1)`) hacia el host de destino indicado, con nombre de usuario opcional.

  Las restricciones de la forma `src-hostname>[user@]dst-hostname` permiten que una clave disponible en un `ssh-agent(1)` reenviado se use a través de un host concreto (indicado por `src-hostname`) para autenticarse en otro host más allá, indicado por `dst-hostname`.

  Al cargar claves se pueden añadir varias restricciones de destino. Al intentar autenticarse con una clave que tiene restricciones de destino, se comprueba toda la ruta de conexión —incluido el reenvío de `ssh-agent(1)`— contra esas restricciones, y cada salto debe estar permitido para que el intento tenga éxito. Por ejemplo, si una clave se reenvía a un host remoto, `host-b`, y se intenta autenticar en otro host, `host-c`, la operación solo tendrá éxito si `host-b` estaba permitido desde el host de origen y el salto siguiente `host-b>host-c` también está permitido por las restricciones de destino.

  Los hosts se identifican por sus claves de host, que **ssh-add** busca en los archivos de hosts conocidos. Se pueden usar patrones con comodines para los nombres de host y se admiten claves de host con certificado. Por defecto, las claves añadidas por **ssh-add** no tienen restricción de destino.

  Las restricciones de destino se añadieron en OpenSSH 8.9. Al usar claves con restricción de destino sobre un canal de `ssh-agent(1)` reenviado, se requiere soporte tanto en el cliente SSH remoto como en el servidor.

  También es importante señalar que `ssh-agent(1)` solo puede hacer cumplir las restricciones de destino cuando se usa una clave, o cuando la reenvía un `ssh(1)` cooperante. En concreto, no impide que un atacante con acceso a un `SSH_AUTH_SOCK` remoto lo reenvíe de nuevo y lo use en otro host (aunque solo hacia un destino permitido).

**`-K`**
: Carga claves residentes desde un autenticador FIDO.

**`-k`**
: Al cargar claves en el agente o borrarlas de él, procesa solo claves privadas simples y omite los certificados.

**`-L`**
: Lista los parámetros de clave pública de todas las identidades que el agente representa actualmente.

**`-P`**
: Al cargar claves PKCS#11 o FIDO desde un token, no solicita un PIN. Esto hará que la operación falle si el token requiere autenticación adicional y no tiene otra forma de obtenerla, como un teclado o un lector biométrico.

**`-l`**
: Lista las huellas de todas las identidades que el agente representa actualmente.

**`-N`**
: Al añadir certificados, por defecto **ssh-add** pedirá al agente que borre automáticamente el certificado poco después de su fecha de caducidad. Este flag suprime ese comportamiento y no especifica una vida para los certificados añadidos al agente.

**`-Q`**
: Consulta al agente la lista de extensiones de protocolo que soporta. Nota: no todos los agentes admiten esta consulta.

**`-q`**
: No muestra mensajes tras una operación exitosa.

**`-S provider`**
: Especifica la ruta a una biblioteca que se usará al añadir claves alojadas en autenticadores FIDO, en lugar del soporte USB HID interno que se usa por defecto.

**`-s pkcs11`**
: Añade las claves proporcionadas por la biblioteca compartida PKCS#11 *pkcs11*. Opcionalmente pueden indicarse archivos de certificado como argumentos. Si están presentes, se cargarán en el agente usando las claves privadas correspondientes cargadas desde el token PKCS#11.

**`-T pubkey ...`**
: Comprueba si las claves privadas correspondientes a los archivos *pubkey* indicados son utilizables, realizando operaciones de firma y verificación con cada una.

**`-t life`**
: Establece una vida máxima al añadir identidades a un agente. La vida puede indicarse en segundos o en un formato de tiempo de los especificados en `sshd_config(5)`.

**`-v`**
: Modo detallado. Hace que **ssh-add** imprima mensajes de depuración sobre su progreso. Es útil para depurar problemas. Varias opciones `-v` aumentan el nivel de detalle. El máximo es 3.

**`-X`**
: Desbloquea el agente.

**`-x`**
: Bloquea el agente con una contraseña.

## ENTORNO

**`DISPLAY`, `SSH_ASKPASS` y `SSH_ASKPASS_REQUIRE`**
: Si **ssh-add** necesita una frase de contraseña, la leerá desde la terminal actual si se ejecutó desde una terminal. Si **ssh-add** no tiene una terminal asociada pero `DISPLAY` y `SSH_ASKPASS` están definidas, ejecutará el programa indicado por `SSH_ASKPASS` (por defecto “ssh-askpass”) y abrirá una ventana X11 para leer la frase. Esto es especialmente útil al llamar a **ssh-add** desde un `.xsession` o script similar.

  `SSH_ASKPASS_REQUIRE` permite un control adicional sobre el uso de un programa askpass. Si esta variable vale “never”, **ssh-add** nunca intentará usarlo. Si vale “prefer”, **ssh-add** preferirá usar el programa askpass en lugar de la TTY al pedir contraseñas. Por último, si vale “force”, se usará el programa askpass para toda entrada de frases de contraseña, esté o no definida `DISPLAY`.

**`SSH_AUTH_SOCK`**
: Identifica la ruta de un socket de dominio Unix usado para comunicarse con el agente.

**`SSH_SK_PROVIDER`**
: Especifica la ruta a una biblioteca que se usará al cargar cualquier clave alojada en autenticadores FIDO, en lugar del soporte USB HID integrado que se usa por defecto.

## ARCHIVOS

**`~/.ssh/id_ecdsa`**<br>
**`~/.ssh/id_ecdsa_sk`**<br>
**`~/.ssh/id_ed25519`**<br>
**`~/.ssh/id_ed25519_sk`**<br>
**`~/.ssh/id_mldsa44_ed25519`**<br>
**`~/.ssh/id_rsa`**
: Contienen la identidad de autenticación del usuario: ECDSA, ECDSA alojada en autenticador, Ed25519, Ed25519 alojada en autenticador, MLDSA44-ED25519 o RSA.

Los archivos de identidad no deben ser legibles por nadie más que el usuario. Ten en cuenta que **ssh-add** ignora los archivos de identidad si otros pueden acceder a ellos.

## ESTADO DE SALIDA

El estado de salida es 0 si hay éxito, 1 si el comando indicado falla y 2 si **ssh-add** no puede contactar con el agente de autenticación.

## VÉASE TAMBIÉN

`ssh(1)`, `ssh-agent(1)`, `ssh-askpass(1)`, `ssh-keygen(1)`, `sshd(8)`

## AUTORES

OpenSSH es un derivado de la versión original y libre ssh 1.2.12 de Tatu Ylonen. Aaron Campbell, Bob Beck, Markus Friedl, Niels Provos, Theo de Raadt y Dug Song eliminaron muchos errores, volvieron a añadir funciones más recientes y crearon OpenSSH. Markus Friedl contribuyó el soporte para las versiones 1.5 y 2.0 del protocolo SSH.
