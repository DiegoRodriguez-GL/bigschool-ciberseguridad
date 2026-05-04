# BigSchool — Módulos de Ciberseguridad

Material didáctico para los módulos prácticos de ciberseguridad ofensiva y defensiva impartidos en BigSchool.

> Cada módulo está diseñado como un curso de 3h 30min – 4h con roadmap, comandos exactos, writeup paso a paso y referencias oficiales.

---

## 📚 Módulos disponibles

### 🔓 M2 — Explotación de CVEs

Aprender a investigar, explotar y reportar vulnerabilidades **CVE reales** sobre máquinas vulnerables modernas (DockerLabs).

| Documento | Contenido |
|---|---|
| [DockerLabs — Máquinas CVE](M2-Explotacion-CVEs/DockerLabs-Maquinas-CVE.md) | 9 máquinas con CVEs reales (2011–2024), CVSS, exploits y writeups |
| [Recursos y Referencias](M2-Explotacion-CVEs/Recursos-y-Referencias.md) | Fuentes oficiales: NVD, MITRE, CISA KEV, INCIBE, HackerOne |

**CVEs cubiertos:**
`CVE-2011-2523` (vsftpd) · `CVE-2015-6033` (Nibbleblog) · `CVE-2018-7475` (WP Site Editor) · `CVE-2019-0193` (Apache Solr) · `CVE-2020-1938` (Tomcat Ghostcat) · `CVE-2021-43798` (Grafana) · `CVE-2023-46604` (ActiveMQ) · `CVE-2024-23897` (Jenkins) · `CVE-2024-27622` (CMS Made Simple)

---

### 🛡️ M4 — Hardening Linux

Endurecer un servidor Ubuntu 22.04 LTS desde cero hasta cumplir CIS Benchmark Level 1, con métricas medibles (Lynis Hardening Index).

| Documento | Contenido |
|---|---|
| [Writeup paso a paso](M4-Hardening-Linux/WRITEUP-Paso-a-Paso.md) | **Práctica completa de 3h 55min** con comandos directos y checkpoints |
| [Curriculum detallado](M4-Hardening-Linux/Curriculum-Detallado.md) | Teoría, configs y referencias para 8 vídeos |
| [Setup del laboratorio (SSH)](M4-Hardening-Linux/Setup-Lab-SSH.md) | Cómo conectar al lab cuando el host es Windows |

**Capas cubiertas:**
1. Identidad (PAM + sudo + SSH key-only)
2. Filesystem (SUID + ACLs + capabilities)
3. Kernel (sysctl + módulos + GRUB)
4. Red (UFW / nftables + Fail2ban)
5. Servicios (systemd + fstab + AIDE)
6. Detección (auditd + rkhunter + journald)
7. Auditoría (Lynis + OpenSCAP + Reporte)

**Estándares aplicados:** CIS Benchmarks · DISA STIG · ANSSI · NIST 800-53

---

## 🎯 Filosofía pedagógica

Cada módulo sigue un formato consistente:

| Fase | Cómo se materializa |
|---|---|
| **1. Marco teórico** | Diapositivas con conceptos, estándares y casos reales |
| **2. Demo en vivo** | Comandos exactos sobre máquina real, paso a paso |
| **3. Métricas medibles** | CVSS / Lynis HI / OpenSCAP score — no opiniones |
| **4. Reporte profesional** | Plantilla de informe entregable al cliente |

---

## 📋 Requisitos comunes

- VM con Ubuntu 22.04 LTS (limpia, snapshot inicial)
- Docker + Docker Compose para los módulos de explotación
- Conexión a Internet para `apt`, repos y scanners
- Editor con renderizado Markdown (VS Code, Obsidian, Typora)

---

## 📖 Cómo usar este repo

1. **Para el formador:** abre el `WRITEUP-Paso-a-Paso.md` del módulo, sigue los checkpoints y mide el progreso con la herramienta de auditoría correspondiente.
2. **Para el alumno:** los `Curriculum` y `Recursos` son la teoría de apoyo. El writeup es la práctica.
3. **Para revisión:** todos los CVEs y configs incluyen enlaces a fuentes oficiales (NVD, advisories del vendor, RFCs).

---

## ⚖️ Licencia y uso

Material didáctico interno. Las referencias a CIS Benchmarks, DISA STIG, ANSSI y otros estándares son enlaces a fuentes oficiales. Los writeups de DockerLabs son enlaces al repositorio público de [albertomarcostic/DockerLabs-WriteUps](https://github.com/albertomarcostic/DockerLabs-WriteUps).
