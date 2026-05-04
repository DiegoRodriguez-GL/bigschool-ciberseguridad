# M4 — HARDENING LINUX — WRITEUP PRÁCTICO

**Sistema objetivo:** Ubuntu Server 22.04 LTS limpia (VM con snapshot)
**Duración total práctica:** ~3h 45min
**Modelo:** Medir → Endurecer (7 módulos) → Medir → Reportar
**Métrica:** Lynis Hardening Index (HI) inicial vs final

---

## ÍNDICE

| Bloque | Tiempo | Tema |
|---|---|---|
| **PRE** | 15 min | Preparación VM + snapshot |
| **M0** | 15 min | Baseline Lynis |
| **M1** | 30 min | Identidad: PAM + sudo + SSH |
| **M2** | 25 min | Permisos: SUID + ACLs + caps |
| **M3** | 30 min | Kernel: sysctl + módulos + GRUB |
| **M4** | 30 min | Firewall + Fail2ban |
| **M5** | 30 min | systemd + fstab + AIDE |
| **M6** | 30 min | auditd + rkhunter + journald |
| **M7** | 30 min | Auditoría final + Reporte |
| **TOTAL** | **3h 45min** | |

> **Regla del módulo:** entre cada bloque se ejecuta `sudo lynis audit system --quick` para ver subir el HI.

---

# PRE — PREPARACIÓN (15 min)

### P.1 — Snapshot inicial
```bash
# En VirtualBox/Vagrant: tomar snapshot "baseline-clean"
```
**Por qué:** vamos a romper cosas. El snapshot es la red de seguridad.

### P.2 — Actualizar sistema
```bash
sudo apt update && sudo apt upgrade -y
```
**Por qué:** partimos de un sistema con parches al día (separamos hardening de patching).

### P.3 — Crear usuario admin con sudo
```bash
sudo adduser admin
sudo usermod -aG sudo admin
```
**Por qué:** todo el módulo se hace con `admin`, nunca con root directo.

### P.4 — Instalar herramientas base
```bash
sudo apt install -y lynis git curl wget vim acl
```
**Por qué:** las necesitaremos en varios módulos.

**✓ CHECK:** `lynis --version` debe responder.

---

# M0 — BASELINE LYNIS (15 min)

### 0.1 — Primera auditoría
```bash
sudo lynis audit system
```
**Por qué:** medir el sistema "tal cual" antes de tocar nada.

### 0.2 — Guardar el HI baseline
```bash
grep "Hardening index" /var/log/lynis.log | tee ~/HI-baseline.txt
sudo cp /var/log/lynis-report.dat ~/lynis-baseline.dat
```
**Por qué:** lo compararemos al final del módulo.

### 0.3 — Ver warnings y suggestions
```bash
sudo lynis show warnings
sudo lynis show suggestions | head -50
```
**Por qué:** lista de tareas que vamos a ir resolviendo módulo a módulo.

**✓ CHECKPOINT M0**
- HI esperado: **55-65**
- Anota tu HI: `___`

---

# M1 — IDENTIDAD: PAM + SUDO + SSH (30 min)

### 1.1 — Auditar cuentas sistema con shell válido
```bash
awk -F: '($3<1000)&&($7!~/nologin|false/){print $1,$7}' /etc/passwd
```
**Por qué:** cuentas de servicio no deben tener shell. Si tienen `/bin/bash` es vector de ataque.

### 1.2 — Bloquear shells de cuentas sistema (si las hay)
```bash
sudo usermod -s /usr/sbin/nologin <user>
```
**Por qué:** elimina el vector de login en cuentas que no son humanas.

### 1.3 — Política de contraseñas (pwquality)
```bash
sudo apt install -y libpam-pwquality
sudo tee -a /etc/security/pwquality.conf <<'EOF'
minlen = 14
minclass = 4
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
retry = 3
remember = 5
EOF
```
**Por qué:** fuerza contraseñas largas y complejas. Bloquea la reutilización de las últimas 5.

### 1.4 — Lockout tras intentos fallidos (faillock)
```bash
sudo tee -a /etc/security/faillock.conf <<'EOF'
deny = 5
unlock_time = 900
fail_interval = 900
EOF
```
**Por qué:** 5 fallos → cuenta bloqueada 15 min. Mata brute-force local.

### 1.5 — sudoers seguro
```bash
sudo visudo
```
Añadir/verificar:
```
Defaults env_reset
Defaults timestamp_timeout=5
Defaults logfile="/var/log/sudo.log"
Defaults !visiblepw
```
**Por qué:** `env_reset` evita inyección por variables. Logfile audita cada sudo.

### 1.6 — Generar par de claves SSH (en tu máquina, no la VM)
```bash
ssh-keygen -t ed25519 -C "admin@hardening-lab"
ssh-copy-id -p 22 admin@<vm-ip>
```
**Por qué:** SSH key-only requiere clave previa. Si la perdemos, nos quedamos fuera.

### 1.7 — Hardening sshd_config
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo tee /etc/ssh/sshd_config.d/99-hardening.conf <<'EOF'
Port 2222
Protocol 2
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 0
LoginGraceTime 30
AllowUsers admin
X11Forwarding no
AllowTcpForwarding no
Banner /etc/issue.net
UsePAM yes
EOF
```
**Por qué:** puerto custom reduce ruido, sin password mata 99% de bots, MaxAuthTries 3 corta brute-force, AllowUsers limita a un usuario.

### 1.8 — Banner de login
```bash
sudo tee /etc/issue.net <<'EOF'
**********************************************************
* Sistema autorizado solo para usuarios autenticados.    *
* Toda actividad es monitorizada y registrada.           *
**********************************************************
EOF
```
**Por qué:** requisito legal en muchas auditorías (ANSSI, STIG). Disuasorio + cobertura legal.

### 1.9 — Aplicar y test SSH
```bash
sudo sshd -t                          # validar config sin reiniciar
sudo systemctl restart ssh
# DESDE OTRA TERMINAL — NO CIERRES LA SESIÓN ACTUAL:
ssh -p 2222 admin@<vm-ip>
```
**Por qué:** si rompes SSH y cierras la única sesión, te quedas fuera. Mantén una abierta hasta confirmar.

**✓ CHECKPOINT M1**
```bash
sudo lynis audit system --quick --tests-from-group authentication
grep "Hardening index" /var/log/lynis.log | tail -1
```
- HI esperado: **+5 a +10 sobre baseline**

---

# M2 — PERMISOS: SUID + ACLs + CAPABILITIES (25 min)

### 2.1 — Inventario de SUID/SGID
```bash
sudo find / -perm -4000 -type f 2>/dev/null | tee ~/suid-baseline.txt
sudo find / -perm -2000 -type f 2>/dev/null | tee ~/sgid-baseline.txt
wc -l ~/suid-baseline.txt
```
**Por qué:** baseline para detectar SUIDs nuevos sospechosos en el futuro.

### 2.2 — Cruzar contra GTFOBins (revisión manual)
```bash
# Abrir https://gtfobins.github.io/ y buscar cada SUID inusual
# Quitar SUID de los que NO sean estándar Linux:
sudo chmod u-s /usr/bin/<binario-no-necesario>
```
**Por qué:** cada SUID en GTFOBins es una ruta directa a root. Solo mantén los necesarios (passwd, sudo, su, mount, ping...).

### 2.3 — Sticky bit en /tmp y /var/tmp
```bash
ls -ld /tmp /var/tmp                  # debe tener "t" al final (drwxrwxrwt)
sudo chmod +t /tmp /var/tmp           # solo si falta
```
**Por qué:** sin sticky, cualquier user borra archivos de otros en /tmp.

### 2.4 — umask seguro
```bash
sudo sed -i 's/^UMASK.*/UMASK 027/' /etc/login.defs
echo 'umask 027' | sudo tee -a /etc/profile
echo 'umask 027' | sudo tee -a /etc/bash.bashrc
```
**Por qué:** archivos nuevos no son legibles por "others". `027` = `rwxr-x---`.

### 2.5 — ACL: caso práctico
```bash
sudo mkdir -p /opt/proyecto
sudo chown root:root /opt/proyecto
sudo chmod 770 /opt/proyecto
sudo setfacl -m u:admin:rwx /opt/proyecto
sudo setfacl -m g:adm:rx /opt/proyecto
getfacl /opt/proyecto
```
**Por qué:** ACL permite permisos finos sin abrir a "others". Evita el `chmod 777` chapuza.

### 2.6 — Capability en lugar de SUID (demo)
```bash
# Imagina app que necesita bind a puerto <1024 sin ser root:
sudo cp /usr/bin/python3 /usr/local/bin/myapp
sudo setcap 'cap_net_bind_service=+ep' /usr/local/bin/myapp
getcap /usr/local/bin/myapp
sudo rm /usr/local/bin/myapp           # demo, limpiar
```
**Por qué:** capability concede UN privilegio puntual. SUID concede TODO root. Mucho más seguro.

### 2.7 — Detectar archivos huérfanos y world-writable
```bash
sudo find / -nouser -o -nogroup 2>/dev/null
sudo find / -perm -0002 -type f -not -path "/proc/*" 2>/dev/null
```
**Por qué:** huérfanos = restos peligrosos. World-writable = cualquiera modifica.

**✓ CHECKPOINT M2**
```bash
sudo lynis audit system --quick --tests-from-group file_permissions
```
- HI esperado: **+3 a +6 sobre M1**

---

# M3 — KERNEL: SYSCTL + MÓDULOS + GRUB (30 min)

### 3.1 — Crear fichero único de hardening sysctl
```bash
sudo tee /etc/sysctl.d/99-hardening.conf <<'EOF'
# === KERNEL ===
kernel.randomize_va_space = 2
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 2
kernel.yama.ptrace_scope = 2
kernel.unprivileged_bpf_disabled = 1
kernel.kexec_load_disabled = 1

# === FILESYSTEM ===
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
fs.protected_fifos = 2
fs.protected_regular = 2
fs.suid_dumpable = 0

# === NETWORK ===
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.conf.all.log_martians = 1
EOF

sudo sysctl -p /etc/sysctl.d/99-hardening.conf
```
**Por qué (cada bloque):**
- **KERNEL:** ASLR completo + ocultar info kernel a no-root + ptrace solo a procesos hijos.
- **FILESYSTEM:** evita race conditions con symlinks/hardlinks. Sin core dumps SUID.
- **NETWORK:** SYN cookies contra DDoS, rp_filter contra IP spoofing, no aceptar ICMP redirects.

### 3.2 — Bloquear módulos de hardware no usado
```bash
sudo tee /etc/modprobe.d/blacklist-hardening.conf <<'EOF'
install usb-storage /bin/true
install firewire-core /bin/true
install bluetooth /bin/true
install dccp /bin/true
install sctp /bin/true
install rds /bin/true
install tipc /bin/true
EOF
```
**Por qué:** USB-storage = robo por pendrive. Firewire/BT = no se usan en server. dccp/sctp/rds/tipc = protocolos exóticos con CVEs históricos.

### 3.3 — GRUB password
```bash
sudo grub-mkpasswd-pbkdf2
# Copia el hash que sale (grub.pbkdf2.sha512.10000.XXXX...)
sudo tee /etc/grub.d/40_custom <<'EOF'
set superusers="grubadmin"
password_pbkdf2 grubadmin grub.pbkdf2.sha512.10000.PEGAR_HASH_AQUI
EOF
sudo update-grub
```
**Por qué:** sin password GRUB, atacante con acceso físico → init=/bin/bash → root sin clave.

### 3.4 — Verificar
```bash
sysctl kernel.randomize_va_space      # debe = 2
sysctl net.ipv4.tcp_syncookies        # debe = 1
sudo lsmod | grep -Ei 'usb|firewire|bluetooth'   # no deben cargar tras reboot
```

**✓ CHECKPOINT M3**
```bash
sudo lynis audit system --quick --tests-from-group kernel
```
- HI esperado: **+5 a +8 sobre M2**

---

# M4 — FIREWALL + FAIL2BAN (30 min)

### 4.1 — UFW: política base
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
**Por qué:** denegar todo entrante por defecto. Solo abrir lo estrictamente necesario.

### 4.2 — Permitir solo SSH custom + HTTP/HTTPS
```bash
sudo ufw allow 2222/tcp comment 'SSH custom'
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw logging on
sudo ufw enable
sudo ufw status verbose
```
**Por qué:** mínima superficie expuesta. Logging para detectar intentos.

### 4.3 — Fail2ban
```bash
sudo apt install -y fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo tee /etc/fail2ban/jail.d/sshd.local <<'EOF'
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
EOF

sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```
**Por qué:** bloquea IPs que fallan 3 intentos en 10 min, ban 1h. Reduce ruido y mata bots.

### 4.4 — Test ban manual
```bash
sudo fail2ban-client set sshd banip 192.0.2.99
sudo fail2ban-client status sshd          # ver IP en banned
sudo fail2ban-client set sshd unbanip 192.0.2.99
```
**Por qué:** valida que el jail funciona antes de exponer el server.

### 4.5 — Test brute-force real (desde Kali, opcional)
```bash
# DESDE OTRA VM (Kali):
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://<vm-ip>:2222 -t 4
# DEBE FALLAR tras 3 intentos. Verificar en server:
sudo fail2ban-client status sshd
```
**Por qué:** confirmar en condiciones reales que la defensa funciona.

**✓ CHECKPOINT M4**
```bash
sudo lynis audit system --quick --tests-from-group firewalls
```
- HI esperado: **+5 a +8 sobre M3**

---

# M5 — SERVICIOS + FSTAB + AIDE (30 min)

### 5.1 — Inventario de servicios habilitados
```bash
systemctl list-unit-files --state=enabled | grep -v static
```
**Por qué:** cada servicio = superficie de ataque. Hay que justificar cada uno.

### 5.2 — Deshabilitar lo innecesario
```bash
sudo systemctl disable --now avahi-daemon cups bluetooth ModemManager 2>/dev/null
sudo systemctl mask avahi-daemon cups
```
**Por qué:** avahi/cups/bluetooth no aplican a server. Mask impide reactivación accidental.

### 5.3 — Hardening de un servicio (ej: SSH)
```bash
sudo mkdir -p /etc/systemd/system/ssh.service.d
sudo tee /etc/systemd/system/ssh.service.d/hardening.conf <<'EOF'
[Service]
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
NoNewPrivileges=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectKernelLogs=true
ProtectControlGroups=true
RestrictNamespaces=true
LockPersonality=true
RestrictRealtime=true
SystemCallArchitectures=native
EOF

sudo systemctl daemon-reload
sudo systemctl restart ssh
systemd-analyze security ssh
```
**Por qué:** sandboxing del servicio. Si SSH es comprometido, no puede tocar el resto del sistema.

### 5.4 — fstab: noexec/nosuid/nodev en /tmp
```bash
sudo cp /etc/fstab /etc/fstab.bak
sudo tee -a /etc/fstab <<'EOF'
tmpfs /tmp tmpfs defaults,nodev,nosuid,noexec,size=2G 0 0
EOF
sudo mount -a
mount | grep /tmp
```
**Por qué:** payloads de exploits suelen caer en /tmp y ejecutarse. `noexec` bloquea ejecutar. `nosuid` ignora SUID. `nodev` no permite dispositivos.

### 5.5 — Deshabilitar core dumps
```bash
echo '* hard core 0' | sudo tee -a /etc/security/limits.conf
echo 'fs.suid_dumpable = 0' | sudo tee -a /etc/sysctl.d/99-hardening.conf
sudo sysctl -p /etc/sysctl.d/99-hardening.conf
```
**Por qué:** core dumps pueden contener secretos en memoria (passwords, claves).

### 5.6 — AIDE: baseline de integridad
```bash
sudo apt install -y aide aide-common
sudo aideinit                           # tarda 5-10 min
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
sudo aide --check                       # debe decir "AIDE found NO differences"
```
**Por qué:** baseline criptográfico de cada archivo del sistema. Detecta cualquier modificación posterior.

### 5.7 — Cron diario AIDE
```bash
echo '0 5 * * * root /usr/bin/aide --check | mail -s "AIDE Report $(hostname)" admin@local' | sudo tee /etc/cron.d/aide
```
**Por qué:** chequeo diario automático. Te enteras al día siguiente si algo cambió.

**✓ CHECKPOINT M5**
```bash
sudo lynis audit system --quick
```
- HI esperado: **+5 a +10 sobre M4**

---

# M6 — AUDITD + RKHUNTER + JOURNALD (30 min)

### 6.1 — journald persistente
```bash
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
sudo sed -i 's/#Storage=auto/Storage=persistent/' /etc/systemd/journald.conf
sudo systemctl restart systemd-journald
journalctl --disk-usage
```
**Por qué:** por defecto journald borra logs al reboot. Persistent = los conserva.

### 6.2 — Instalar auditd
```bash
sudo apt install -y auditd audispd-plugins
sudo systemctl enable --now auditd
```
**Por qué:** auditd registra a nivel kernel quién hizo qué (acceso a archivos, ejecuciones).

### 6.3 — Reglas auditd profesionales (PCI-DSS / CIS)
```bash
sudo tee /etc/audit/rules.d/hardening.rules <<'EOF'
## Borrar reglas previas
-D
-b 8192
-f 1

## Identidad
-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k shadow_changes
-w /etc/group  -p wa -k group_changes
-w /etc/sudoers -p wa -k sudoers_changes
-w /etc/sudoers.d/ -p wa -k sudoers_changes

## SSH y configs críticos
-w /etc/ssh/sshd_config -p wa -k ssh_config
-w /etc/pam.d/ -p wa -k pam_changes

## Cron
-w /etc/crontab -p wa -k cron_changes
-w /etc/cron.d/ -p wa -k cron_changes

## Privilege escalation
-a always,exit -F arch=b64 -S execve -F euid=0 -k root_exec
-a always,exit -F arch=b64 -S setuid -S setgid -F auid>=1000 -k privilege_change

## Cambios al propio audit log
-w /var/log/audit/ -p wa -k audit_log_tampering

## Hacer reglas inmutables (hasta reboot)
-e 2
EOF

sudo augenrules --load
sudo auditctl -l
```
**Por qué:** trazabilidad forense. Si te atacan, sabes qué tocaron. `-e 2` = nadie puede borrar reglas en runtime.

### 6.4 — Test queries auditd
```bash
sudo touch /etc/passwd                  # generar evento
sudo ausearch -k passwd_changes --start recent
sudo aureport --summary
sudo aureport -au                       # autenticaciones fallidas
```
**Por qué:** confirma que auditd está capturando + ver formato de queries.

### 6.5 — rkhunter: rootkit detection
```bash
sudo apt install -y rkhunter
sudo rkhunter --propupd                 # baseline tras instalación limpia
sudo rkhunter --check --skip-keypress
```
**Por qué:** detecta backdoors, archivos modificados, puertos sospechosos. Baseline antes para no falsos positivos.

### 6.6 — chkrootkit (segunda opinión)
```bash
sudo apt install -y chkrootkit
sudo chkrootkit | grep -v "not infected" | grep -v "not found"
```
**Por qué:** complementa rkhunter con técnicas distintas.

### 6.7 — Logwatch para resumen diario
```bash
sudo apt install -y logwatch
sudo logwatch --detail high --range today --output stdout | head -50
```
**Por qué:** resumen automático de logs por mail. Detecta anomalías sin tener que revisar manualmente.

**✓ CHECKPOINT M6**
```bash
sudo lynis audit system --quick --tests-from-group logging
```
- HI esperado: **+3 a +6 sobre M5**

---

# M7 — AUDITORÍA FINAL + REPORTE (30 min)

### 7.1 — Lynis audit completo final
```bash
sudo lynis audit system --report-file ~/lynis-final.dat
grep "Hardening index" /var/log/lynis.log | tail -1 | tee ~/HI-final.txt
```
**Por qué:** medición final del proceso completo.

### 7.2 — Comparar baseline vs final
```bash
echo "=== BASELINE ===" && cat ~/HI-baseline.txt
echo "=== FINAL ===" && cat ~/HI-final.txt
diff ~/lynis-baseline.dat ~/lynis-final.dat | head -50
```
**Por qué:** demostrar el progreso con números, no opiniones.

### 7.3 — OpenSCAP: perfil CIS
```bash
sudo apt install -y libopenscap8 ssg-debderived
ls /usr/share/xml/scap/ssg/content/

# Si no hay perfil ubuntu2204, usa el genérico:
PROFILE_FILE=$(ls /usr/share/xml/scap/ssg/content/ssg-ubuntu*-ds.xml 2>/dev/null | head -1)
oscap info $PROFILE_FILE | grep -A1 "Profile"
```
**Por qué:** OpenSCAP es el estándar oficial CIS/STIG. Genera reporte profesional.

### 7.4 — Ejecutar perfil CIS L1 Server
```bash
sudo oscap xccdf eval \
  --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
  --results /tmp/oscap-results.xml \
  --report /tmp/oscap-report.html \
  $PROFILE_FILE

# Ver el resultado:
firefox /tmp/oscap-report.html &
```
**Por qué:** evaluación contra CIS Benchmark formal. El HTML es entregable.

### 7.5 — Generar script de remediación (NO aplicar sin revisar)
```bash
sudo oscap xccdf generate fix \
  --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
  $PROFILE_FILE > /tmp/remediation.sh
less /tmp/remediation.sh
```
**Por qué:** script automático de los checks que aún fallan. Útil pero PELIGROSO si se ejecuta a ciegas.

### 7.6 — Plantilla de informe
```bash
mkdir -p ~/informe-hardening
cd ~/informe-hardening

tee informe.md <<'EOF'
# Informe de Hardening — <hostname>

## 1. Resumen ejecutivo
- Sistema: Ubuntu Server 22.04 LTS
- Periodo: <fecha-inicio> → <fecha-fin>
- Lynis HI baseline: <X>
- Lynis HI final: <Y>
- OpenSCAP CIS L1 compliance: <Z>%

## 2. Capas endurecidas
- M1 Identidad (PAM, sudo, SSH key-only + 2FA)
- M2 Filesystem (SUID, ACLs, capabilities)
- M3 Kernel (sysctl, módulos, GRUB)
- M4 Red (UFW + Fail2ban)
- M5 Servicios (systemd, fstab, AIDE)
- M6 Detección (auditd + rkhunter + journald)

## 3. Findings residuales
[Pegar aquí los warnings que quedan en Lynis/OpenSCAP con justificación]

## 4. Plan de mantenimiento
- AIDE: cron diario 5 AM
- Lynis: cron semanal
- OpenSCAP: revisión mensual
- Patches: unattended-upgrades

## 5. Anexos
- HI-baseline.txt / HI-final.txt
- /tmp/oscap-report.html
- Configs aplicadas (sysctl, sshd, fstab, audit.rules)
EOF

# Copiar evidencias:
cp ~/HI-baseline.txt ~/HI-final.txt .
cp /tmp/oscap-report.html .
cp ~/lynis-baseline.dat ~/lynis-final.dat .
sudo cp /etc/sysctl.d/99-hardening.conf .
sudo cp /etc/ssh/sshd_config.d/99-hardening.conf .
sudo cp /etc/audit/rules.d/hardening.rules .

ls -la ~/informe-hardening/
```
**Por qué:** entregable profesional. Lo que distingue a un junior de un consultor senior.

### 7.7 — Configurar mantenimiento automático
```bash
# unattended-upgrades:
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Lynis semanal:
echo '0 3 * * 0 root /usr/bin/lynis audit system --quick --cronjob > /var/log/lynis-cron.log' | sudo tee /etc/cron.d/lynis-weekly
```
**Por qué:** cierra el ciclo. Hardening es continuo, no puntual.

**✓ CHECKPOINT FINAL**
```bash
echo "=========================================="
echo "  HARDENING INDEX BASELINE → FINAL"
echo "=========================================="
cat ~/HI-baseline.txt
cat ~/HI-final.txt
echo "=========================================="
```
- HI final esperado: **≥ 80**
- OpenSCAP CIS L1 esperado: **≥ 75% pass**

---

# RESUMEN — TIMELINE COMPLETO

| Bloque | Min | Acumulado | Acción clave | Verificación |
|---|---|---|---|---|
| PRE | 15 | 0:15 | VM + snapshot + admin | `lynis --version` |
| M0 | 15 | 0:30 | Lynis baseline | HI ≈ 55-65 |
| M1 | 30 | 1:00 | PAM + SSH key-only | Lynis auth +5/+10 |
| M2 | 25 | 1:25 | SUID + ACLs + caps | Lynis perms +3/+6 |
| M3 | 30 | 1:55 | sysctl + módulos + GRUB | Lynis kernel +5/+8 |
| M4 | 30 | 2:25 | UFW + Fail2ban | Lynis firewall +5/+8 |
| M5 | 30 | 2:55 | systemd + fstab + AIDE | Lynis +5/+10 |
| M6 | 30 | 3:25 | auditd + rkhunter | Lynis logging +3/+6 |
| M7 | 30 | 3:55 | OpenSCAP + Reporte | HI final ≥ 80 |

**TOTAL: 3h 55min** (con margen de 25 min sobre el target de 3h 30min) ✅

---

# REGLAS DE ORO

1. **Snapshot ANTES de cada módulo.** Si rompes algo, volver atrás cuesta 30 segundos.
2. **Mantén DOS sesiones SSH abiertas en M1.** Si la config rompe, la otra sesión te salva.
3. **Lee TODO comando antes de pegarlo.** Especialmente los `tee` y `sed -i`.
4. **`sshd -t` ANTES de reiniciar SSH.** Valida la config sin tirarte fuera.
5. **No apliques `oscap remediation.sh` a ciegas.** Léelo primero, puede romper servicios.
6. **HI no es perfecto.** Un HI 100 es imposible — siempre hay trade-offs entre seguridad y funcionalidad.

---

# RECURSOS RÁPIDOS

- **Lynis:** https://cisofy.com/lynis/
- **CIS Benchmarks:** https://www.cisecurity.org/benchmark/distribution_independent_linux
- **GTFOBins:** https://gtfobins.github.io/
- **Mozilla SSH:** https://infosec.mozilla.org/guidelines/openssh
- **Madaidan Hardening:** https://madaidans-insecurities.github.io/guides/linux-hardening.html
- **OpenSCAP:** https://www.open-scap.org/
- **Florian Roth auditd:** https://github.com/Neo23x0/auditd

---

**FIN DEL WRITEUP**
