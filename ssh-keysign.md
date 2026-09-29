# SSH-KEYSIGN(8)

## NOMBRE

**ssh-keysign** — asistente de OpenSSH para la autenticación basada en host

## SINOPSIS

```
ssh-keysign
```

## DESCRIPCIÓN

`ssh(1)` usa **ssh-keysign** para acceder a las claves de host locales y generar la firma digital requerida durante la autenticación basada en host (*host-based authentication*).

**ssh-keysign** está desactivado por defecto y solo puede activarse en el archivo de configuración global del cliente `/etc/ssh/ssh_config`, estableciendo `EnableSSHKeysign` a “yes”.

**ssh-keysign** no está pensado para que lo invoque el usuario, sino `ssh(1)`. Consulta `ssh(1)` y `sshd(8)` para más información sobre la autenticación basada en host.

## ARCHIVOS

**`/etc/ssh/ssh_config`**
: Controla si **ssh-keysign** está activado.

**`/etc/ssh/ssh_host_ecdsa_key`**<br>
**`/etc/ssh/ssh_host_ed25519_key`**<br>
**`/etc/ssh/ssh_host_mldsa44_ed25519_key`**<br>
**`/etc/ssh/ssh_host_rsa_key`**
: Estos archivos contienen las partes privadas de las claves de host usadas para generar la firma digital. Deben pertenecer a root, ser legibles solo por root y no ser accesibles por otros. Como solo root puede leerlos, **ssh-keysign** debe ser set-uid root si se usa la autenticación basada en host.

**`/etc/ssh/ssh_host_ecdsa_key-cert.pub`**<br>
**`/etc/ssh/ssh_host_ed25519_key-cert.pub`**<br>
**`/etc/ssh/ssh_host_mldsa44_ed25519_key-cert.pub`**<br>
**`/etc/ssh/ssh_host_rsa_key-cert.pub`**
: Si estos archivos existen, se asume que contienen la información de certificado pública correspondiente a las claves privadas anteriores.

## VÉASE TAMBIÉN

`ssh(1)`, `ssh-keygen(1)`, `ssh_config(5)`, `sshd(8)`

## HISTORIA

**ssh-keysign** apareció por primera vez en OpenBSD 3.2.

## AUTORES

Markus Friedl <markus@openbsd.org>
