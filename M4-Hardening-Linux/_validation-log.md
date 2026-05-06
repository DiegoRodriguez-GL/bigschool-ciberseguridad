# Log de validación del WRITEUP

> Validación ejecutada en Ubuntu 26.04 LTS (Resolute Raccoon) - kernel 7.0.0-15-generic
> SSH desde Windows host (172.20.10.X) - usuario `admin`
> Fecha: mayo 2026

## Estado general

- [x] PRE - Preparación
- [x] M0 - Baseline Lynis (HI=62)
- [x] M1 - Identidad (PAM + sudo + SSH) -> HI=69 (+7)
- [x] M2 - Permisos (SUID + ACLs + capabilities)
- [x] M3 - Kernel (sysctl + módulos + GRUB)
- [x] M4 - Firewall (UFW + Fail2ban)
- [x] M5 - Servicios + fstab + AIDE init en background
- [x] M6 - auditd + rkhunter + journald
- [x] M7 - OpenSCAP eval (CIS L1: 1952 pass / 910 fail / 405 NA)

## Bugs detectados (11 totales)

### BUG #1 - M0.2 - grep sin sudo falla
**Archivo:** `/var/log/lynis.log` es 640 root:root
**Fix:** añadir `sudo` al `grep`

### BUG #2 - M0.3 - lynis show warnings/suggestions no existe
**Versión Lynis:** 3.1.6 no tiene esos subcomandos
**Fix:** `sudo grep '^warning\[\]=' /var/log/lynis-report.dat` y similar para suggestions

### BUG #3 - M1.4 - pam_faillock viene instalado pero no activado
**Detalle:** En Debian/Ubuntu solo escribir en `faillock.conf` no surte efecto
**Fix:** añadir 3 líneas a `/etc/pam.d/common-auth`:
- `auth required pam_faillock.so preauth` (antes de pam_unix)
- `auth [default=die] pam_faillock.so authfail` (después de pam_sss)
- `auth sufficient pam_faillock.so authsucc`

### BUG #4 - M1.5 - sudo-rs vs sudo.ws en Ubuntu 26.04
**Detalle:** Ubuntu 26.04 trae `sudo-rs 0.2.13` (Rust) por defecto. NO soporta `logfile`, `log_file`, `!visiblepw`, etc.
**Fix:** `sudo update-alternatives --set sudo /usr/bin/sudo.ws` para volver al sudo clásico (1.9.17p2)

### BUG #5 - M1.9 - SSH socket activation
**Detalle:** Ubuntu 24.04+ usa `ssh.socket` para activar SSH. Cambiar `Port 2222` en sshd_config NO cambia el puerto real.
**Fix:** override del socket en `/etc/systemd/system/ssh.socket.d/override.conf` con `ListenStream=0.0.0.0:2222`

### BUG #6 - M2.4 - umask en login.defs
**Detalle:** Ubuntu 26.04 `/etc/login.defs` no trae línea `UMASK` por defecto. El `sed -i 's/^UMASK.../...'` no encuentra patrón y no hace nada.
**Fix:** `if grep -q '^UMASK' /etc/login.defs; then sed; else echo 'UMASK 027' >> ...`

### BUG #7 - M5.3 (CRÍTICO) - ProtectHome=true rompe SSH
**Detalle:** Aplicar el bloque genérico de hardening systemd a `ssh.service` con `ProtectHome=true` deja a sshd sin acceso a `~/.ssh/authorized_keys`. Resultado: nadie puede hacer login.
**Fix:** para SSH usar `ProtectHome=read-only` (no `true`)

### BUG #8 - M5.3 (CRÍTICO) - NoNewPrivileges=true rompe sudo
**Detalle:** Las sesiones SSH heredan los flags del cgroup del servicio. `NoNewPrivileges=true` impide que los usuarios SSH usen `sudo`.
**Fix:** para SSH NO incluir `NoNewPrivileges=true`

### BUG #9 - M5.3 (CRÍTICO) - ProtectSystem=strict rompe escrituras
**Detalle:** Aplicar `ProtectSystem=strict` a SSH deja `/etc /usr /boot` en read-only para todos los procesos hijo (incluido el shell del usuario). No puedes editar configs ni instalar nada por SSH.
**Fix:** documentar que el bloque genérico de hardening systemd NO aplica a SSH. Para SSH se usa un subset reducido. El bloque completo se aplica a otros servicios (nginx, postgresql, redis...).

**Recuperación si se aplica mal:** desde consola física:
```bash
sudo rm /etc/systemd/system/ssh.service.d/hardening.conf
sudo systemctl daemon-reload
sudo systemctl restart ssh.service
```

O desde sesión SSH activa, escapar el cgroup con `systemd-run`:
```bash
sudo systemd-run --uid=0 --quiet --wait --pipe bash -c 'rm /etc/systemd/system/ssh.service.d/hardening.conf && systemctl daemon-reload && systemctl restart ssh.service'
```

### BUG #10 - M5.4 - /tmp con noexec rompe Lynis
**Detalle:** Lynis crea scripts temporales en /tmp y los ejecuta. Con `noexec` falla con "Fatal error: Program execution stopped due to security measure".
**Fix:** ejecutar Lynis con `TMPDIR=/var/lib/lynis-tmp lynis audit system`. O ejecutar Lynis ANTES de aplicar M5.4.

### BUG #11 - M7.3 - libopenscap8 no existe en Ubuntu 26.04
**Detalle:** El paquete `libopenscap8` ya no existe. Ahora se llama `openscap-scanner` (provee binario `oscap`) y `openscap-utils`.
**Fix:** `sudo apt install -y openscap-scanner openscap-utils ssg-debderived`

**Nota adicional:** `ssg-debderived` 0.1.79 trae perfiles para Ubuntu 22.04 y 24.04, pero NO 26.04 (todavía). En 26.04 usar el perfil `ssg-ubuntu2404-ds.xml`. La diferencia con 26.04 es mínima en checks de hardening.

## Detalles importantes detectados (no bugs estrictos)

### INFO #1 - Lynis timer activo tras instalación
Tras `apt install lynis`, queda activo `lynis.timer` que ejecuta `lynis audit system` diariamente y regenera `/var/log/lynis-report.dat`. La copia tras audit (`cp lynis-report.dat lynis-baseline.dat`) es la versión correcta a comparar.

### INFO #2 - Avahi/cups socket activation
Servicios como avahi-daemon, cups, bluetooth tienen socket activation. Solo deshabilitar el `.service` no es suficiente, hay que deshabilitar también los `.socket` y `.path`.

### INFO #3 - GRUB 40_custom con shebang
`/etc/grub.d/40_custom` por defecto trae `#!/bin/sh` y `exec tail -n +3 $0`. Usar `tee -a` (append) en lugar de `tee` (que pisa).

### INFO #4 - Fail2ban backend
En Ubuntu 26.04 fail2ban usa `backend = systemd` por defecto. El `logpath = /var/log/auth.log` que pone el writeup probablemente se ignora (lee del journal). Funciona igualmente.

## Métricas finales

| Indicador | Valor |
|---|---|
| Lynis HI baseline | 62 |
| Lynis HI tras M1 | 69 (+7) |
| Lynis HI final | (pendiente, se ejecuta con TMPDIR alternativo) |
| OpenSCAP CIS L1 Server | 1952 pass / 910 fail / 405 NA |
| Compliance % (excluyendo NA) | ~68% |
