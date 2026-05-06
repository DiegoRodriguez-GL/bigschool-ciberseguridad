# Setup del laboratorio - Conexiones SSH

> Cómo conectar al servidor Ubuntu durante el módulo de Hardening Linux cuando el host es Windows.

## TL;DR

> **Ubuntu corriendo todo el rato + cliente SSH desde Windows host. Levantas Kali solo para el módulo de Fail2ban (15 min) y la apagas.**

---

## Recomendación: enfoque híbrido

| Cuándo | Cómo conectar | Por qué |
|---|---|---|
| **La mayoría del módulo** | Cliente SSH desde Windows host (Windows Terminal con OpenSSH built-in) | Tráfico real, no consume otra VM, muestra "punto de vista del cliente externo" |
| **Solo V4.5 (Fail2ban)** | Kali → Ubuntu (2 VMs) | Hydra brute-force real, IP origen distinta visible en logs/banlist |
| **Localhost (Ubuntu → Ubuntu)** | ❌ No recomendado | UFW y Fail2ban suelen whitelistear `127.0.0.1`, no enseña nada realista |

---

## Por qué cliente Windows → Ubuntu funciona muy bien

Windows 10/11 trae OpenSSH cliente built-in:

```powershell
# Desde Windows Terminal:
ssh -p 2222 admin@192.168.56.10
```

### Ventajas para los vídeos

- ✅ Tráfico genuino entre dos máquinas (host ↔ guest)
- ✅ Los logs (`/var/log/auth.log`) muestran IP origen real (la del host Windows en NAT/bridged)
- ✅ Ves el banner `/etc/issue.net` antes del login → queda profesional
- ✅ Verificas `AllowUsers admin` con conexiones desde fuera de la VM
- ✅ No pierdes RAM con una VM extra
- ✅ Setup en 0 minutos

### Configuración de red recomendada

| Hipervisor | Modo de red |
|---|---|
| **VirtualBox** | Host-Only (`vboxnet0`) o Bridged |
| **VMware** | NAT con port forwarding, o Bridged |
| **Hyper-V** | External Switch o Internal Switch con NAT |

---

## Cuándo SÍ levantar Kali

**Solo para V4.5 - demo de Fail2ban con brute-force real.**

```bash
# Desde Kali:
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.10:2222 -t 4
```

### Lo que verán los alumnos

1. Hydra disparando intentos
2. SSH cayendo tras 3 fallos
3. `sudo fail2ban-client status sshd` mostrando la IP de Kali en `Banned IP list`
4. El intento posterior desde Kali timeoutea → **demo killer**

Después apagas Kali. **No la dejes encendida 4h.**

---

## Recursos mínimos del host Windows

| Setup | RAM mínima host |
|---|---|
| Solo Ubuntu (2GB) + cliente Windows | **8 GB** |
| Ubuntu (2GB) + Kali (2GB) simultáneo | **12 GB** |
| Ubuntu (4GB) + Kali (4GB) si vas holgado | **16 GB** |

Si tienes 16 GB, levanta ambas y olvídate. Si tienes 8 GB, usa el método híbrido.

---

## Alternativas al cliente SSH de Windows

Si te peta el OpenSSH built-in o quieres algo más visual:

| Cliente | Por qué |
|---|---|
| **Windows Terminal + OpenSSH** ⭐ | Built-in, terminal moderno, sin instalar nada |
| **MobaXterm** | Tabs, X11 forwarding y SFTP en una sola app |
| **WSL2 (Ubuntu en Windows)** | `ssh`, `hydra`, `nmap` desde Windows como si fuera Linux |
| **PuTTY** | Clásico, pero el output queda peor en grabación |

**Recomendado para grabar:** Windows Terminal con perfil OpenSSH. Queda limpísimo en pantalla.

---

## Por qué NO usar SSH localhost (Ubuntu → 127.0.0.1)

Aunque sea tentador por simplicidad:

- ❌ Fail2ban suele tener `127.0.0.1` en `ignoreip` → bans no se aplican
- ❌ UFW deja pasar localhost por defecto → no validas reglas
- ❌ Los logs muestran origen `127.0.0.1` → no es realista
- ❌ El cliente y el server son la misma máquina → no enseñas el flujo real

Vale para validar que `sshd` arranca tras `sshd -t`, nada más.

---

## Comandos útiles para verificar la conexión

### Desde Ubuntu (servidor)

```bash
# IP de la VM (la que conectas desde Windows)
ip -4 addr show | grep inet

# SSH escuchando
sudo ss -tlnp | grep ssh

# Test de configuración antes de reiniciar (clave para no quedarte fuera)
sudo sshd -t

# Logs en vivo
sudo tail -f /var/log/auth.log
```

### Desde Windows (cliente)

```powershell
# Conexión simple
ssh -p 2222 admin@192.168.56.10

# Con clave específica
ssh -i C:\Users\diego\.ssh\id_ed25519 -p 2222 admin@192.168.56.10

# Verbose para debugging
ssh -vvv -p 2222 admin@192.168.56.10
```

### Desde Kali (atacante, solo para V4.5)

```bash
# Brute-force con hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.10:2222 -t 4

# Comprobar puertos abiertos
nmap -sV -p 2222 192.168.56.10
```

---

## Checklist pre-grabación

- [ ] VM Ubuntu arrancada y con IP fija conocida
- [ ] Snapshot `baseline-clean` tomado
- [ ] OpenSSH cliente disponible en Windows (`ssh -V` en PowerShell)
- [ ] Conexión funciona desde Windows: `ssh -p 22 admin@<ip-ubuntu>`
- [ ] Para V4.5: VM Kali preparada con Hydra (`hydra -h` responde)
- [ ] Red entre VMs configurada (Host-Only o Bridged)
- [ ] Dos sesiones SSH abiertas durante M1 (por si rompes la primera)

---

## Regla de oro

> **Mantén siempre dos sesiones SSH abiertas durante el M1 (Identidad/SSH hardening). Si rompes la config y reinicias `sshd` sin validar primero con `sshd -t`, la primera sesión te salva el módulo.**
