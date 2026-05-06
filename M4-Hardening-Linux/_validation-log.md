# Log de validación del WRITEUP

> Validación ejecutada en Ubuntu Server 26.04 LTS (Resolute Raccoon) - kernel 7.0.0-15-generic
> SSH desde Windows host (172.20.10.X) - usuario `admin`
> Fecha de validación: mayo 2026

## Estado general

- [x] PRE - Preparación (OK)
- [x] M0 - Baseline Lynis (HI=62, OK con 2 bugs)
- [ ] M1 - Identidad (PAM + sudo + SSH)
- [ ] M2 - Permisos (SUID + ACLs + capabilities)
- [ ] M3 - Kernel (sysctl + módulos + GRUB)
- [ ] M4 - Firewall (UFW + Fail2ban)
- [ ] M5 - Servicios (systemd + fstab + AIDE)
- [ ] M6 - auditd + rkhunter + journald
- [ ] M7 - Auditoría final + Reporte

## Bugs / cambios necesarios

### BUG #1 - M0.2 - grep sin sudo falla
**Original:**
```bash
grep "Hardening index" /var/log/lynis.log | tee ~/HI-baseline.txt
```
**Error:** `grep: /var/log/lynis.log: Permission denied` (el archivo es 640 root:root)
**Fix:**
```bash
sudo grep "Hardening index" /var/log/lynis.log | tee ~/HI-baseline.txt
```

### BUG #2 - M0.3 - lynis show warnings/suggestions no existen
**Original:**
```bash
sudo lynis show warnings
sudo lynis show suggestions | head -50
```
**Error:** `Unknown argument 'warnings' for lynis show` (Lynis 3.1.6 no tiene esos subcomandos)
**Fix:**
```bash
sudo grep '^warning\[\]=' /var/log/lynis-report.dat
sudo grep '^suggestion\[\]=' /var/log/lynis-report.dat | head -20
```

### INFO - Timer systemd de Lynis activo
Tras `apt install lynis`, queda activo `lynis.timer` que ejecuta lynis diariamente y trunca/regenera `/var/log/lynis-report.dat`. Hacer la copia con `cp` inmediatamente tras el audit es correcto. El timer se podría desactivar con `sudo systemctl disable --now lynis.timer` si interfiere.
