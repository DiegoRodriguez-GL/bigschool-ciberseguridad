================================================================================
  MÓDULO M4 — HARDENING LINUX
  Curriculum detallado — 8 vídeos / ~4h
  BigSchool — Escuela de Ciberseguridad
================================================================================

FILOSOFÍA DEL MÓDULO
────────────────────────────────────────────────────────────────────────────────
   1. Medir antes (V4.1 — Lynis baseline)
   2. Endurecer 6 capas (V4.2 a V4.7)
   3. Medir después (V4.8 — Lynis + OpenSCAP)
   4. Documentar el progreso en un informe profesional

   Objetivo cuantificable: subir Lynis Hardening Index de ~55 a ~85+


ESTÁNDARES DE REFERENCIA
────────────────────────────────────────────────────────────────────────────────
  • CIS Benchmarks      → Center for Internet Security (industria)
  • DISA STIG           → US Department of Defense (más estricto)
  • ANSSI               → Gobierno francés (excelente nivel técnico)
  • NIST SP 800-53/171  → Marco normativo USA
  • PCI-DSS             → Para entornos con tarjetas de pago
  • ISO/IEC 27001       → Marco general de SGSI


ESQUEMA DEL ROADMAP
────────────────────────────────────────────────────────────────────────────────

  V4.1 → Metodología + Lynis baseline                        (25 min)
  V4.2 → Capa de identidad: Usuarios, PAM, sudo, SSH         (30 min)
  V4.3 → Capa de FS: Permisos, SUID, ACLs, capabilities      (30 min)
  V4.4 → Capa de kernel: sysctl, módulos, GRUB               (30 min)
  V4.5 → Capa de red: Firewall + Fail2ban                    (30 min)
  V4.6 → Capa de servicios: systemd hardening + AIDE         (30 min)
  V4.7 → Capa de detección: auditd + rootkit hunters         (30 min)
  V4.8 → Auditoría final + OpenSCAP + Reporte                (35 min)
                                                       ─────────────
                                                       TOTAL: 240 min


================================================================================
  V4.1 — METODOLOGÍA HARDENING + CIS BENCHMARKS + LYNIS BASELINE (25 min)
================================================================================

CONCEPTOS CLAVE
  • Defense-in-depth (capas concéntricas de defensa)
  • Hardening != patching (cerrar superficie expuesta vs corregir bugs)
  • CIS Levels: L1 (server estable), L2 (entornos sensibles)
  • Lynis Hardening Index (HI) — métrica numérica reproducible

ESTRUCTURA DEL VÍDEO
  1. Qué es hardening (3')
  2. Capas de hardening (3')
  3. Estándares: CIS / STIG / ANSSI (5')
  4. Modelo medir-endurecer-medir (2')
  5. Demo Lynis primer scan (5')
  6. Interpretar HI y warnings (3')
  7. Generar baseline.dat para comparar al final (2')
  8. Setup del laboratorio del módulo (2')

COMANDOS PRINCIPALES
  sudo apt update && sudo apt install lynis -y
  sudo lynis audit system
  sudo lynis show details
  sudo lynis show warnings
  sudo lynis show suggestions
  cat /var/log/lynis.log | grep "Hardening index"

REFERENCIAS
  • CIS:    https://www.cisecurity.org/benchmark/distribution_independent_linux
  • STIG:   https://public.cyber.mil/stigs/
  • ANSSI:  https://cyber.gouv.fr/publications/recommandations-de-configuration-dun-systeme-gnulinux
  • Lynis:  https://cisofy.com/lynis/
  • Madaidan: https://madaidans-insecurities.github.io/guides/linux-hardening.html


================================================================================
  V4.2 — USUARIOS, PAM, SUDO Y SSH HARDENING (30 min)
================================================================================

OBJETIVOS
  • Eliminar shells de cuentas sistema
  • Forzar contraseñas robustas (pwquality)
  • Lockout tras intentos fallidos (faillock)
  • SSH solo con clave + 2FA opcional

CHECKS CIS QUE CUBRE
  • CIS 5.1 SSH Server Configuration
  • CIS 5.2 Configure PAM
  • CIS 5.3 User Accounts and Environment
  • CIS 6.2 User and Group Settings

CONFIGS CLAVE

  /etc/security/pwquality.conf
  ────────────────────────────────
    minlen = 14
    minclass = 4
    dcredit = -1
    ucredit = -1
    lcredit = -1
    ocredit = -1
    retry = 3
    remember = 5

  /etc/security/faillock.conf
  ────────────────────────────────
    deny = 5
    unlock_time = 900
    fail_interval = 900

  /etc/ssh/sshd_config
  ────────────────────────────────
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
    Banner /etc/issue.net
    X11Forwarding no
    AllowTcpForwarding no
    UsePAM yes

REFERENCIAS
  • Mozilla OpenSSH Guidelines: https://infosec.mozilla.org/guidelines/openssh
  • PAM docs: https://www.linux-pam.org/Linux-PAM-html/
  • Google Authenticator PAM: https://github.com/google/google-authenticator-libpam


================================================================================
  V4.3 — PERMISOS: SUID/SGID, STICKY, UMASK, ACLs, CAPABILITIES (30 min)
================================================================================

OBJETIVOS
  • Auditar todos los SUID/SGID del sistema
  • Reemplazar SUID por capabilities donde sea posible
  • umask 027 por defecto
  • Saber cuándo usar ACLs en lugar de chmod

ATAQUES QUE PREVIENE
  • Privilege escalation vía SUID abusables (ver GTFOBins)
  • Lectura no autorizada por permisos por defecto débiles
  • Race conditions en /tmp sin sticky bit

COMANDOS DE AUDITORÍA

  # SUID/SGID baseline (guardar y revisar):
  sudo find / -perm -4000 -type f 2>/dev/null > /tmp/suid.txt
  sudo find / -perm -2000 -type f 2>/dev/null > /tmp/sgid.txt

  # World-writable files:
  sudo find / -perm -0002 -type f 2>/dev/null

  # Archivos sin owner/group:
  sudo find / -nouser -o -nogroup 2>/dev/null

  # ACLs:
  setfacl -m u:diego:rwx /opt/proyecto
  setfacl -m g:devs:rx /opt/proyecto
  getfacl /opt/proyecto

  # Capabilities (alternativa a SUID):
  sudo setcap 'cap_net_bind_service=+ep' /usr/local/bin/myapp
  getcap /usr/local/bin/myapp

REFERENCIAS
  • GTFOBins: https://gtfobins.github.io/
  • man capabilities(7), man acl(5)


================================================================================
  V4.4 — KERNEL HARDENING: SYSCTL, MÓDULOS, GRUB, ASLR (30 min)
================================================================================

OBJETIVOS
  • Activar ASLR completo
  • Restringir info expuesta del kernel (dmesg, kptr, ptrace)
  • Endurecer red a nivel kernel
  • Bloquear módulos de hardware no usado (USB, FireWire, BT)
  • Proteger arranque con GRUB password

CONFIG PRINCIPAL: /etc/sysctl.d/99-hardening.conf

  # KERNEL
  kernel.randomize_va_space = 2
  kernel.dmesg_restrict = 1
  kernel.kptr_restrict = 2
  kernel.yama.ptrace_scope = 2
  kernel.unprivileged_bpf_disabled = 1
  kernel.kexec_load_disabled = 1

  # FILESYSTEM
  fs.protected_hardlinks = 1
  fs.protected_symlinks = 1
  fs.protected_fifos = 2
  fs.protected_regular = 2
  fs.suid_dumpable = 0

  # NETWORK
  net.ipv4.tcp_syncookies = 1
  net.ipv4.conf.all.rp_filter = 1
  net.ipv4.conf.default.rp_filter = 1
  net.ipv4.conf.all.accept_source_route = 0
  net.ipv4.conf.all.accept_redirects = 0
  net.ipv4.conf.all.send_redirects = 0
  net.ipv4.icmp_echo_ignore_broadcasts = 1
  net.ipv4.icmp_ignore_bogus_error_responses = 1
  net.ipv4.conf.all.log_martians = 1

MÓDULOS A BLOQUEAR (/etc/modprobe.d/)

  install usb-storage /bin/true
  install firewire-core /bin/true
  install dccp /bin/true
  install sctp /bin/true
  install rds /bin/true
  install tipc /bin/true
  install bluetooth /bin/true   # solo si no se usa

GRUB PASSWORD
  sudo grub-mkpasswd-pbkdf2
  # Añadir a /etc/grub.d/40_custom:
  set superusers="admin"
  password_pbkdf2 admin grub.pbkdf2.sha512.10000.HASH...
  sudo update-grub

REFERENCIAS
  • KSPP: https://kernsec.org/wiki/index.php/Kernel_Self_Protection_Project
  • Kernel sysctl: https://www.kernel.org/doc/Documentation/sysctl/


================================================================================
  V4.5 — FIREWALL (nftables/ufw) + FAIL2BAN (30 min)
================================================================================

OBJETIVOS
  • Default DROP en INPUT y FORWARD
  • Solo abrir puertos estrictamente necesarios
  • Logging de paquetes droppeados
  • Fail2ban contra brute-force de SSH y servicios web

REGLAS UFW (rápido)

  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 2222/tcp comment 'SSH custom'
  sudo ufw allow 80/tcp
  sudo ufw allow 443/tcp
  sudo ufw logging on
  sudo ufw enable

EJEMPLO NFTABLES (/etc/nftables.conf)

  table inet filter {
    chain input {
      type filter hook input priority filter; policy drop;
      ct state established,related accept
      iif lo accept
      icmp type echo-request limit rate 5/second accept
      tcp dport 2222 ct state new limit rate 10/minute accept
      tcp dport { 80, 443 } ct state new accept
      log prefix "DROP-IN: " level warn
    }
    chain forward { type filter hook forward priority filter; policy drop; }
    chain output  { type filter hook output  priority filter; policy accept; }
  }

FAIL2BAN /etc/fail2ban/jail.local

  [DEFAULT]
  bantime = 3600
  findtime = 600
  maxretry = 3
  banaction = nftables-multiport

  [sshd]
  enabled = true
  port = 2222
  filter = sshd
  logpath = /var/log/auth.log

REFERENCIAS
  • nftables: https://wiki.nftables.org/
  • Fail2ban: https://www.fail2ban.org/
  • UFW community: https://help.ubuntu.com/community/UFW


================================================================================
  V4.6 — SYSTEMD HARDENING + SERVICIOS + FSTAB + AIDE (30 min)
================================================================================

OBJETIVOS
  • Deshabilitar/enmascarar servicios innecesarios
  • Endurecer units systemd con flags de protección
  • /tmp /var /home con noexec/nosuid/nodev cuando aplique
  • Baseline de integridad con AIDE

SYSTEMD HARDENING FLAGS (cualquier service)

  [Service]
  ProtectSystem=strict
  ProtectHome=true
  PrivateTmp=true
  PrivateDevices=true
  NoNewPrivileges=true
  ProtectKernelTunables=true
  ProtectKernelModules=true
  ProtectKernelLogs=true
  ProtectControlGroups=true
  RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
  RestrictNamespaces=true
  LockPersonality=true
  MemoryDenyWriteExecute=true
  RestrictRealtime=true
  SystemCallFilter=@system-service
  SystemCallArchitectures=native

  # Auditar:
  systemd-analyze security
  systemd-analyze security <unit>

/etc/fstab — opciones recomendadas

  # /tmp en RAM
  tmpfs   /tmp     tmpfs   defaults,nodev,nosuid,noexec,size=2G  0 0
  # /var/tmp link a /tmp o partición separada
  tmpfs   /var/tmp tmpfs   defaults,nodev,nosuid,noexec          0 0
  # /home si es partición aparte:
  /dev/sda3 /home  ext4    defaults,nodev,nosuid                 0 2

AIDE (File Integrity Monitoring)

  sudo apt install aide aide-common -y
  sudo aideinit                                # genera baseline
  sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
  sudo aide --check                            # detecta cambios

  # Cron diario:
  echo '0 5 * * * root /usr/bin/aide --check | mail -s "AIDE report" admin' \
    | sudo tee /etc/cron.d/aide

REFERENCIAS
  • systemd security: https://www.freedesktop.org/software/systemd/man/systemd.exec.html
  • AIDE: https://aide.github.io/


================================================================================
  V4.7 — LOGGING Y AUDITORÍA: journald, auditd, rkhunter (30 min)
================================================================================

OBJETIVOS
  • journald persistente (los logs sobreviven al reboot)
  • auditd con reglas profesionales (PCI-DSS, CIS)
  • Rotación de logs (logrotate)
  • Detección de rootkits (rkhunter, chkrootkit)
  • Reporte diario por mail (logwatch)

JOURNALD PERSISTENTE
  sudo mkdir -p /var/log/journal
  sudo systemd-tmpfiles --create --prefix /var/log/journal
  # /etc/systemd/journald.conf:
  Storage=persistent
  Compress=yes
  ForwardToSyslog=yes

AUDITD — Reglas mínimas (/etc/audit/rules.d/hardening.rules)

  -D
  -b 8192
  -f 1

  # Identidad
  -w /etc/passwd -p wa -k passwd_changes
  -w /etc/shadow -p wa -k shadow_changes
  -w /etc/group  -p wa -k group_changes
  -w /etc/sudoers -p wa -k sudoers_changes
  -w /etc/sudoers.d/ -p wa -k sudoers_changes

  # SSH config
  -w /etc/ssh/sshd_config -p wa -k ssh_config

  # Cron
  -w /etc/crontab -p wa -k cron_changes
  -w /etc/cron.d/ -p wa -k cron_changes
  -w /var/spool/cron/ -p wa -k cron_changes

  # Privilege escalation
  -a always,exit -F arch=b64 -S execve -F euid=0 -k root_exec
  -a always,exit -F arch=b64 -S setuid -S setgid -k privilege_change

  # Borrado de logs
  -w /var/log/audit/ -p wa -k audit_log_changes

  # Hacer reglas inmutables (CIS): -e 2 al final

  # Cargar:
  sudo augenrules --load
  sudo auditctl -l

QUERIES ÚTILES
  sudo ausearch -k passwd_changes --start today
  sudo ausearch -k root_exec --start recent
  sudo aureport --summary
  sudo aureport -au           # autenticaciones

RKHUNTER + CHKROOTKIT
  sudo apt install rkhunter chkrootkit
  sudo rkhunter --propupd     # baseline tras instalación limpia
  sudo rkhunter --check --skip-keypress
  sudo chkrootkit

REFERENCIAS
  • PCI-DSS auditd rules: https://github.com/linux-audit/audit-userspace/tree/master/rules
  • Florian Roth auditd: https://github.com/Neo23x0/auditd
  • Sigma rules: https://github.com/SigmaHQ/sigma
  • rkhunter: https://rkhunter.sourceforge.net/


================================================================================
  V4.8 — AUDITORÍA FINAL: Lynis + OpenSCAP + REPORTE (35 min)
================================================================================

OBJETIVOS
  • Comparar Lynis HI inicial vs final
  • Ejecutar perfil CIS Ubuntu 22.04 vía OpenSCAP
  • Generar reporte HTML profesional
  • Plantilla de informe entregable
  • (Extra) docker-bench-security

OPENSCAP

  sudo apt install libopenscap8 ssg-debderived -y

  # Listar perfiles disponibles:
  oscap info /usr/share/xml/scap/ssg/content/ssg-ubuntu2204-ds.xml

  # Ejecutar perfil CIS Level 1 Server:
  sudo oscap xccdf eval \
    --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
    --results /tmp/oscap-results.xml \
    --report /tmp/oscap-report.html \
    /usr/share/xml/scap/ssg/content/ssg-ubuntu2204-ds.xml

  # Generar script de remediación automática (¡revisarlo antes de aplicar!):
  sudo oscap xccdf generate fix \
    --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
    /usr/share/xml/scap/ssg/content/ssg-ubuntu2204-ds.xml > /tmp/remediation.sh

DOCKER-BENCH-SECURITY (extra)

  git clone https://github.com/docker/docker-bench-security.git
  cd docker-bench-security
  sudo ./docker-bench-security.sh

ESTRUCTURA DEL INFORME PROFESIONAL

  1. Resumen ejecutivo (1 página)
     - Sistema auditado
     - Score Lynis baseline → final
     - Score OpenSCAP / CIS compliance %
     - Hallazgos críticos resueltos
     - Hallazgos pendientes

  2. Metodología
     - Estándares aplicados (CIS L1)
     - Herramientas usadas
     - Periodo y alcance

  3. Hardening aplicado por capa
     - 3.1 Identidad (V4.2)
     - 3.2 Filesystem (V4.3)
     - 3.3 Kernel (V4.4)
     - 3.4 Red (V4.5)
     - 3.5 Servicios (V4.6)
     - 3.6 Detección (V4.7)

  4. Findings residuales (lo que quedó)
     - Por severidad y CIS check
     - Justificación si se aceptan riesgos

  5. Plan de mantenimiento
     - Cron de Lynis semanal
     - AIDE diario
     - Revisión mensual de auditd
     - Patching: unattended-upgrades

  6. Anexos
     - Outputs Lynis baseline / final
     - Reporte HTML OpenSCAP
     - Configs aplicadas (sysctl, sshd, fstab, audit.rules)

REFERENCIAS
  • OpenSCAP: https://www.open-scap.org/
  • SCAP Security Guide: https://github.com/ComplianceAsCode/content
  • CIS-CAT Pro: https://www.cisecurity.org/cybersecurity-tools/cis-cat-pro
  • Plantillas informe: https://github.com/juliocesarfort/public-pentesting-reports


================================================================================
  HERRAMIENTAS DEL MÓDULO (resumen)
================================================================================

AUDITORÍA AUTOMÁTICA
  • Lynis              → HI scoring + sugerencias
  • OpenSCAP           → CIS/STIG compliance
  • CIS-CAT Pro/Lite   → Oficial CIS
  • docker-bench-sec   → Hardening Docker

FILE INTEGRITY
  • AIDE               → Baseline + check diario
  • debsums / rpm -V   → Integridad de paquetes

DETECCIÓN
  • auditd             → Audit framework kernel
  • rkhunter           → Rootkit hunter
  • chkrootkit         → Detección clásica de rootkits
  • logwatch           → Resumen diario por mail

PROTECCIÓN ACTIVA
  • Fail2ban           → Brute-force prevention
  • ufw / nftables     → Firewall
  • PAM (pwquality, faillock, google-auth) → Autenticación
  • OpenSSH (hardening sshd_config)

AUTOMATIZACIÓN
  • Ansible dev-sec    → Hardening como código
  • unattended-upgrades → Patching automático
  • needrestart        → Reinicio servicios tras update


================================================================================
  LABS RECOMENDADOS
================================================================================

PRIMARIO
  • Ubuntu Server 22.04 LTS (VM/Vagrant)
    - Snapshot 'baseline-clean' antes de empezar
    - Snapshot tras cada vídeo
    - Reset rápido si rompes algo

ALTERNATIVAS
  • Debian 12 Bookworm   → muy similar a Ubuntu, más minimalista
  • Rocky/AlmaLinux 9    → para audiencia RHEL (pequeños cambios apt→dnf)

NO USAR PARA HARDENING
  • Distros desktop (GNOME, KDE)  → ruido innecesario
  • Distros bleeding-edge (Arch)  → CIS no soporta oficialmente
  • Containers solos              → ver V4.8 para containers


================================================================================
  CIS CHECKS CUBIERTOS POR CADA VÍDEO (referencia rápida)
================================================================================

V4.1   → Setup / Lynis (no checks específicos)
V4.2   → CIS 5.1, 5.2, 5.3, 6.2  (Auth, PAM, SSH)
V4.3   → CIS 6.1                  (System File Permissions)
V4.4   → CIS 1, 3                 (Initial Setup, Network)
V4.5   → CIS 3.5                  (Firewall)
V4.6   → CIS 1.1, 2               (Filesystem, Services)
V4.7   → CIS 4                    (Logging and Auditing)
V4.8   → Coverage report completo


================================================================================
  RECURSOS GLOBALES
================================================================================

ESTÁNDARES OFICIALES
  CIS:    https://www.cisecurity.org/benchmark/distribution_independent_linux
  STIG:   https://public.cyber.mil/stigs/
  ANSSI:  https://cyber.gouv.fr/publications/recommandations-de-configuration-dun-systeme-gnulinux
  NIST:   https://csrc.nist.gov/

GUÍAS COMUNITARIAS
  Madaidan:  https://madaidans-insecurities.github.io/guides/linux-hardening.html
  Mozilla:   https://infosec.mozilla.org/guidelines/openssh
  KSPP:      https://kernsec.org/wiki/index.php/Kernel_Self_Protection_Project

AUTOMATIZACIÓN
  Ansible DevSec: https://github.com/dev-sec/ansible-collection-hardening
  ComplianceAsCode: https://github.com/ComplianceAsCode/content

REFERENCIA RÁPIDA
  GTFOBins:  https://gtfobins.github.io/
  Linux Audit Project: https://github.com/linux-audit
  Florian Roth auditd: https://github.com/Neo23x0/auditd


================================================================================
  PROPUESTA DE EJERCICIO FINAL
================================================================================

ENUNCIADO PARA EL ALUMNO:
  Partiendo de una Ubuntu Server 22.04 LTS limpia:

  1. Ejecuta Lynis y guarda el HI inicial.
  2. Aplica las configuraciones de los vídeos V4.2 a V4.7.
  3. Ejecuta Lynis y OpenSCAP de nuevo.
  4. Entrega:
     a) Capturas del HI antes/después
     b) Reporte HTML de OpenSCAP
     c) Informe de hardening (plantilla del V4.8)
     d) Diff de los archivos modificados
  5. Justifica cualquier check que dejes en FAIL.

CRITERIO DE APROBACIÓN
  • Lynis HI ≥ 80
  • OpenSCAP CIS L1 compliance ≥ 75%
  • Informe profesional entregado


================================================================================
 FIN DEL CURRICULUM
================================================================================
