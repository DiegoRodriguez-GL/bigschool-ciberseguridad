================================================================================
  MÓDULO M2 - EXPLOTACIÓN DE CVEs
  Recursos, fuentes oficiales y ejemplos para enseñar en directo
  BigSchool - Escuela de Ciberseguridad
================================================================================


────────────────────────────────────────────────────────────────────────────────
  ÍNDICE
────────────────────────────────────────────────────────────────────────────────

  1. ¿QUÉ ES UN CVE?            - fuentes oficiales y bases de datos
  2. CVSS                       - calculadora oficial y documentación
  3. CNA vs VENDOR              - quién asigna los CVE IDs
  4. CÓMO REPORTAR              - vías oficiales (MITRE, INCIBE, ENISA)
  5. EJEMPLOS DE REPORTES       - INCIBE, MITRE/NVD, HackerOne, Project Zero
  6. CVEs HISTÓRICOS PARA ENSEÑAR
  7. CISA KEV - VULNS ACTIVAS
  8. VENDOR ADVISORIES
  9. ESTÁNDARES Y NORMATIVA
 10. EJERCICIO EN VIVO PROPUESTO



================================================================================
  1. ¿QUÉ ES UN CVE?
================================================================================

DEFINICIÓN
  Common Vulnerabilities and Exposures - identificador único y público de
  una vulnerabilidad concreta.
  Formato: CVE-AAAA-NNNNN  (año + número secuencial)

GESTOR PRINCIPAL
  MITRE Corporation (USA) en colaboración con CISA y red de CNAs.

FUENTES OFICIALES
  • CVE.org (web oficial del programa CVE)
    https://www.cve.org

  • National Vulnerability Database (NIST)
    https://nvd.nist.gov

  • CVE Records - buscador
    https://www.cve.org/CVERecord/SearchResults

  • CVE List Downloads (JSON, CSV)
    https://www.cve.org/Downloads

BASES DE DATOS COMPLEMENTARIAS
  • Vulners                  https://vulners.com
  • OpenCVE                  https://www.opencve.io
  • CVE Details              https://www.cvedetails.com
  • ExploitDB                https://www.exploit-db.com
  • Snyk Vulnerability DB    https://security.snyk.io
  • GitHub Advisory Database https://github.com/advisories
  • Patchstack DB (WordPress) https://patchstack.com/database
  • EU Vulnerability DB (ENISA) https://euvd.enisa.europa.eu



================================================================================
  2. CVSS - Common Vulnerability Scoring System
================================================================================

DEFINICIÓN
  Sistema estándar para puntuar la gravedad de vulnerabilidades.
  Rango: 0.0 - 10.0
  Mantenido por FIRST.org

NIVELES DE SEVERIDAD
  0.0           None
  0.1 - 3.9     Low       (verde)
  4.0 - 6.9     Medium    (amarillo)
  7.0 - 8.9     High      (naranja)
  9.0 - 10.0    Critical  (rojo)

CALCULADORAS OFICIALES
  • Calculadora CVSS v3.1 (NVD)
    https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator

  • Calculadora CVSS v4.0 (FIRST - versión nueva)
    https://www.first.org/cvss/calculator/4.0

  • Especificación oficial CVSS v3.1
    https://www.first.org/cvss/v3.1/specification-document

  • Especificación oficial CVSS v4.0
    https://www.first.org/cvss/v4.0/specification-document

  • Guía de usuario CVSS v3.1
    https://www.first.org/cvss/v3.1/user-guide

GRUPOS DE MÉTRICAS
  - Base          (intrínsecas a la vuln, no cambian)
                  AV, AC, PR, UI, S, C, I, A
  - Temporal      (cambian con el tiempo: madurez exploit, parche, etc.)
  - Environmental (impacto específico en TU entorno)

EJEMPLO DE VECTOR (Log4Shell)
  CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H  →  10.0 Critical

RECURSOS DIDÁCTICOS
  • CVSS Training (FIRST)
    https://www.first.org/cvss/training
  • Documentación NVD sobre CVSS
    https://nvd.nist.gov/vuln-metrics/cvss



================================================================================
  3. CNA vs VENDOR - quién es quién
================================================================================

VENDOR
  El fabricante del producto vulnerable (Microsoft, Apache, Cisco, etc.)
  Puede ser CNA o no serlo.

CNA - CVE Numbering Authority
  Entidad autorizada por MITRE para asignar CVE IDs dentro de su scope.
  ~400 CNAs activas en el mundo.

ROOT CNA
  CNA que coordina otras CNAs bajo su jerarquía.
  Ejemplos: MITRE (top root), ENISA (root EU), INCIBE (root España).

JERARQUÍA ACTUAL
  MITRE (Top Root)
   ├── ENISA (Root EU desde 2024)
   │    └── INCIBE (Root España desde 2021)
   │         ├── Edgewatch
   │         ├── ATISoluciones
   │         └── ...
   ├── CISA (sector público USA)
   └── ~400 CNAs vendor (Microsoft, Cisco, GitHub, Red Hat, ...)

LISTA OFICIAL DE CNAs
  https://www.cve.org/PartnerInformation/ListofPartners

ÁRBOL DE DECISIÓN PARA ASIGNAR CVE
  ¿El vendor es CNA?
   ├── SÍ → vendor te asigna CVE
   └── NO
        ├── ¿Eres español o vendor español? → INCIBE-CERT
        └── Otro caso                      → MITRE (cveform.mitre.org)



================================================================================
  4. CÓMO REPORTAR UNA VULNERABILIDAD - VÍAS OFICIALES
================================================================================

──────────────────────────────────────────────────
 4.1 - MITRE / Programa CVE
──────────────────────────────────────────────────
  • Formulario de solicitud de CVE
    https://cveform.mitre.org

  • Política del programa CVE
    https://www.cve.org/ResourcesSupport/AllResources/CNARules

  • Documentación para investigadores
    https://www.cve.org/ResourcesSupport/Resources

──────────────────────────────────────────────────
 4.2 - INCIBE-CERT (España)
──────────────────────────────────────────────────
  • Página principal de vulnerabilidades INCIBE-CERT
    https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades

  • Política de divulgación de vulnerabilidades INCIBE
    https://www.incibe.es/incibe-cert/sobre-nosotros/politica-divulgacion-vulnerabilidades

  • Asignación y publicación CVE en INCIBE
    https://www.incibe.es/incibe-cert/asignacion-publicacion-cve

  • CNAs participantes bajo INCIBE Root
    https://www.incibe.es/incibe-cert/asignacion-publicacion-cve/cnas-participantes

  • Contacto INCIBE-CERT (incidencias y vulns)
    incidencias@incibe-cert.es
    https://www.incibe.es/incibe-cert/contacto

──────────────────────────────────────────────────
 4.3 - CCN-CERT (sector público España)
──────────────────────────────────────────────────
  • Web oficial
    https://www.ccn-cert.cni.es

  • Avisos de vulnerabilidades
    https://www.ccn-cert.cni.es/seguridad-al-dia/avisos-ccn-cert.html

──────────────────────────────────────────────────
 4.4 - ENISA (Europa)
──────────────────────────────────────────────────
  • EU Vulnerability Database
    https://euvd.enisa.europa.eu

  • Coordinated Vulnerability Disclosure Policies (mapa EU)
    https://www.enisa.europa.eu/topics/national-cyber-security-strategies/ncss-map/national-cyber-security-strategies-interactive-map/national-implementation/cvd-policy

  • ENISA CSIRTs Network
    https://csirtsnetwork.eu

──────────────────────────────────────────────────
 4.5 - CISA (USA - referencia internacional)
──────────────────────────────────────────────────
  • CISA Vulnerability Disclosure Program
    https://www.cisa.gov/coordinated-vulnerability-disclosure-process

  • CISA Coordinated Disclosure Guide
    https://www.cisa.gov/sites/default/files/publications/CISA_CVD_Process.pdf

──────────────────────────────────────────────────
 4.6 - security.txt (RFC 9116)
──────────────────────────────────────────────────
  • Web oficial del estándar
    https://securitytxt.org

  • RFC 9116
    https://www.rfc-editor.org/rfc/rfc9116

  • Generador de security.txt
    https://securitytxt.org/#generator

  • Buscador de security.txt en webs
    https://securitytxt.dev

──────────────────────────────────────────────────
 4.7 - Plataformas de Bug Bounty
──────────────────────────────────────────────────
  • HackerOne          https://hackerone.com
  • Bugcrowd           https://www.bugcrowd.com
  • YesWeHack          https://www.yeswehack.com
  • Intigriti          https://www.intigriti.com
  • Synack             https://www.synack.com
  • Open Bug Bounty    https://www.openbugbounty.org

──────────────────────────────────────────────────
 4.8 - disclose.io (safe harbor framework)
──────────────────────────────────────────────────
  • Web oficial
    https://disclose.io

  • Plantillas de política de disclosure
    https://github.com/disclose/diodb



================================================================================
  5. EJEMPLOS DE REPORTES - PARA ENSEÑAR EN DIRECTO
================================================================================

──────────────────────────────────────────────────
 5.1 - INCIBE-CERT (avisos en español, formato real)
──────────────────────────────────────────────────
  ★ React2Shell - análisis técnico completo
    CVE-2025-55182
    https://www.incibe.es/incibe-cert/publicaciones/bitacora-de-seguridad/explotacion-de-la-vulnerabilidad-react2shell-cve-2025-55182

  • CVE-2025-1902 - SQL Injection en PHPGurukul Student Record System
    https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2025-1902

  • CVE-2025-2025 - Plugin GiveWP de WordPress
    https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2025-2025

  • CVE-2025-21616
    https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2025-21616

  • CVE-2025-20231
    https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2025-20231

  • Boletín Patch Tuesday Microsoft - Julio 2025
    https://www.incibe.es/incibe-cert/alerta-temprana/avisos/actualizaciones-de-seguridad-de-microsoft-de-julio-de-2025

  • Boletín Patch Tuesday Microsoft - Noviembre 2025
    https://www.incibe.es/incibe-cert/alerta-temprana/avisos/actualizaciones-de-seguridad-de-microsoft-de-noviembre-de-2025

──────────────────────────────────────────────────
 5.2 - MITRE / NVD (formato canónico mundial)
──────────────────────────────────────────────────
  Para cada CVE puedes mostrar la doble vista:
  - CVE.org → metadatos oficiales del programa CVE
  - NVD     → análisis NIST + CVSS calculado + referencias

  ★ CVE-2021-44228 - Log4Shell
    https://www.cve.org/CVERecord?id=CVE-2021-44228
    https://nvd.nist.gov/vuln/detail/CVE-2021-44228

  ★ CVE-2014-0160 - Heartbleed
    https://www.cve.org/CVERecord?id=CVE-2014-0160
    https://nvd.nist.gov/vuln/detail/CVE-2014-0160
    Web dedicada: https://heartbleed.com

  ★ CVE-2021-4034 - PwnKit (polkit, 12 años sin detectar)
    https://www.cve.org/CVERecord?id=CVE-2021-4034
    https://nvd.nist.gov/vuln/detail/CVE-2021-4034
    Writeup Qualys: https://blog.qualys.com/vulnerabilities-threat-research/2022/01/25/pwnkit-local-privilege-escalation-vulnerability-discovered-in-polkit-pkexec-cve-2021-4034

  ★ CVE-2022-0847 - Dirty Pipe
    https://www.cve.org/CVERecord?id=CVE-2022-0847
    https://nvd.nist.gov/vuln/detail/CVE-2022-0847
    Writeup original (Max Kellermann): https://dirtypipe.cm4all.com

  ★ CVE-2021-3156 - Sudo Baron Samedit
    https://www.cve.org/CVERecord?id=CVE-2021-3156
    https://nvd.nist.gov/vuln/detail/CVE-2021-3156
    Writeup Qualys: https://blog.qualys.com/vulnerabilities-research/2021/01/26/cve-2021-3156-heap-based-buffer-overflow-in-sudo-baron-samedit

──────────────────────────────────────────────────
 5.3 - HackerOne Hacktivity (reportes reales públicos)
──────────────────────────────────────────────────
  ORO PURO para enseñar redacción de reportes.

  • Hacktivity (feed público)
    https://hackerone.com/hacktivity

  • Top reports archive (GitHub - recopilación histórica)
    https://github.com/reddelexc/hackerone-reports

  • Top all-time
    https://github.com/reddelexc/hackerone-reports/blob/master/tops_by_program/TOPHACKERONE.md

  REPORTES LEGENDARIOS:
  • #340431 - Shopify GraphQL race condition
    https://hackerone.com/reports/340431

  • #2095245 - GitLab
    https://hackerone.com/reports/2095245

  • #156536 - Twitter (IDOR)
    https://hackerone.com/reports/156536

  • #105663 - Stored XSS Slack (clásico)
    https://hackerone.com/reports/105663

  • #225537 - Uber subdomain takeover
    https://hackerone.com/reports/225537

──────────────────────────────────────────────────
 5.4 - Google Project Zero
──────────────────────────────────────────────────
  Disclosure de élite con embargo de 90 días estricto.

  • Issue Tracker (todos los reportes)
    https://project-zero.issues.chromium.org/p/project-zero/issues/list

  • Blog técnico
    https://googleprojectzero.blogspot.com

  • Política de disclosure
    https://googleprojectzero.blogspot.com/p/vulnerability-disclosure-policy.html

  • 0day "In the Wild" tracker
    https://googleprojectzero.github.io/0days-in-the-wild

──────────────────────────────────────────────────
 5.5 - GitHub Security Advisories
──────────────────────────────────────────────────
  • Advisory Database completa (búsqueda por CVE / CWE / paquete)
    https://github.com/advisories

  • Documentación para reportar via GitHub
    https://docs.github.com/en/code-security/security-advisories

  • Programa GHSA (GitHub Security Advisories)
    https://github.com/advisories?type=reviewed



================================================================================
  6. CVEs HISTÓRICOS - CASOS DIDÁCTICOS PARA CLASE
================================================================================

──────────────────────────────────────────────────
 BUENOS EJEMPLOS DE DISCLOSURE RESPONSABLE
──────────────────────────────────────────────────

  ★ CVE-2021-44228 - Log4Shell
    Investigador: Chen Zhaojun (Alibaba Cloud Security Team)
    Vendor:       Apache Software Foundation
    Lección:      Cómo reportar bien algo crítico. Apache movió el parche
                  en tiempo récord. Coordinación masiva.
    CVSS:         10.0 Critical
    Tipo:         RCE en Java logging library

  ★ CVE-2014-0160 - Heartbleed
    Investigador: Neel Mehta (Google) + Codenomicon
    Vendor:       OpenSSL
    Lección:      Coordinated disclosure multi-stakeholder + branding bug
                  con web propia (heartbleed.com) para concienciar.
    Tipo:         Memory disclosure en TLS heartbeat

  ★ CVE-2021-4034 - PwnKit
    Investigador: Qualys Research Team
    Vendor:       polkit project
    Lección:      Vuln de 12 años. Disclosure técnico modélico con writeup
                  detallado tras parche.
    Tipo:         Local privilege escalation

  ★ CVE-2022-0847 - Dirty Pipe
    Investigador: Max Kellermann (CM4all)
    Vendor:       Linux kernel
    Lección:      Writeup público brutal post-parche. Plantilla perfecta
                  de cómo redactar un advisory técnico.
    Tipo:         LPE en kernel Linux

  ★ CVE-2021-3156 - Baron Samedit (sudo)
    Investigador: Qualys Research Team
    Vendor:       sudo project
    Lección:      Heap overflow en software ubicuo. Disclosure cuidadoso.
    Tipo:         Local privilege escalation

──────────────────────────────────────────────────
 MALOS EJEMPLOS - DISCLOSURE QUE SALIÓ MAL
──────────────────────────────────────────────────

  ✗ CVE-2021-34527 - PrintNightmare
    Qué pasó:    Investigadores de Sangfor publicaron PoC pensando que
                 ya estaba parcheada. NO LO ESTABA. Caos. Microsoft tuvo
                 que sacar parche de emergencia.
    Lección:     Verifica DOS VECES que el fix está publicado antes de
                 hacer disclosure técnico.

  ✗ CVE-2022-30190 - Follina
    Qué pasó:    Reportada vía VirusTotal en abril 2022. Microsoft tardó
                 SEMANAS en reaccionar. Investigador acabó publicando.
    Lección:     A veces la coordinación falla. ¿Qué haces si el vendor
                 ignora tu reporte?

  ✗ Caso Marcus Hutchins (MalwareTech)
    Lección:     Las leyes de cibercrimen pueden volverse contra
                 investigadores legítimos. Importancia del safe harbor
                 (disclose.io) y de tener cobertura legal antes de
                 reportar.

──────────────────────────────────────────────────
 EJERCICIO COMPARATIVO
──────────────────────────────────────────────────
  Muestra LADO A LADO en clase:
  - Log4Shell        (BIEN: coordinación perfecta)
  - PrintNightmare   (MAL: PoC publicada antes del fix)
  La diferencia entre disclosure responsable y caos se ve clarísima.



================================================================================
  7. CISA KEV - VULNERABILIDADES EXPLOTADAS ACTIVAMENTE
================================================================================

KEV - Known Exploited Vulnerabilities Catalog
  Lista mantenida por CISA con vulns que SE ESTÁN EXPLOTANDO ahora mismo
  en ataques reales en el mundo. Actualización semanal.

  • Catálogo principal
    https://www.cisa.gov/known-exploited-vulnerabilities-catalog

  • Catálogo en JSON (para parsear con scripts)
    https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json

  • Catálogo en CSV
    https://www.cisa.gov/sites/default/files/csv/known_exploited_vulnerabilities.csv

  • CISA Cybersecurity Advisories
    https://www.cisa.gov/news-events/cybersecurity-advisories

  • CISA Vulnerability Bulletins
    https://www.cisa.gov/news-events/bulletins



================================================================================
  8. VENDOR ADVISORIES - PORTALES OFICIALES DE LOS GRANDES
================================================================================

  • Microsoft Security Response Center (MSRC)
    https://msrc.microsoft.com/update-guide

  • Cisco Security Advisories
    https://sec.cloudapps.cisco.com/security/center/publicationListing.x

  • Apple Security Releases
    https://support.apple.com/en-us/100100

  • Google Security Bulletins (Android, ChromeOS)
    https://source.android.com/docs/security/bulletin

  • Apache Security Reports
    https://httpd.apache.org/security_report.html

  • Red Hat Security Advisories
    https://access.redhat.com/security/security-updates

  • Ubuntu Security Notices
    https://ubuntu.com/security/notices

  • Debian Security Advisories
    https://www.debian.org/security

  • Oracle Critical Patch Updates
    https://www.oracle.com/security-alerts

  • VMware Security Advisories
    https://www.vmware.com/security/advisories.html

  • Fortinet PSIRT Advisories
    https://www.fortiguard.com/psirt

  • Palo Alto Networks Security Advisories
    https://security.paloaltonetworks.com

  • Atlassian Security Advisories
    https://www.atlassian.com/trust/security/advisories



================================================================================
  9. ESTÁNDARES Y NORMATIVA
================================================================================

ESTÁNDARES INTERNACIONALES
  • ISO/IEC 29147 - Vulnerability Disclosure
    https://www.iso.org/standard/72311.html

  • ISO/IEC 30111 - Vulnerability Handling Processes
    https://www.iso.org/standard/69725.html

  • RFC 9116 - A File Format to Aid in Security Vulnerability Disclosure
    https://www.rfc-editor.org/rfc/rfc9116

GUÍAS DE REFERENCIA
  • CERT/CC Guide to Coordinated Vulnerability Disclosure
    https://vuls.cert.org/confluence/display/CVD

  • OWASP Vulnerability Disclosure Cheat Sheet
    https://cheatsheetseries.owasp.org/cheatsheets/Vulnerability_Disclosure_Cheat_Sheet.html

  • FIRST Multi-Party Vulnerability Coordination
    https://www.first.org/global/sigs/vulnerability-coordination

NORMATIVA EU
  • NIS2 Directive
    https://digital-strategy.ec.europa.eu/en/policies/nis2-directive

  • Cyber Resilience Act (CRA)
    https://digital-strategy.ec.europa.eu/en/policies/cyber-resilience-act



================================================================================
 10. EJERCICIO EN VIVO PROPUESTO (15-20 min)
================================================================================

DEMO GUIADA "DEL CVE A LA EXPLOTACIÓN"
  Recorrer en directo el ciclo completo con los alumnos:

  PASO 1 - Coger un CVE reciente del catálogo CISA KEV
           https://www.cisa.gov/known-exploited-vulnerabilities-catalog

  PASO 2 - Buscarlo en NVD para ver el CVSS, vector y referencias
           https://nvd.nist.gov

  PASO 3 - Hacer click en la referencia del vendor para ver el advisory
           original (lo que publicó el fabricante)

  PASO 4 - Buscarlo en HackerOne Hacktivity por si hay reporte público
           https://hackerone.com/hacktivity

  PASO 5 - Buscarlo en ExploitDB por si hay PoC
           https://www.exploit-db.com

  PASO 6 - Buscar al investigador en Twitter/X o Mastodon para ver
           el "human side" del disclosure

  RESULTADO: los alumnos ven en 20 minutos el ciclo entero
             descubrimiento → reporte → CVE → CVSS → publicación →
             explotación pública.


EJERCICIO COMPLEMENTARIO - RELLENAR EL FORMULARIO MITRE
  Abrir https://cveform.mitre.org y rellenar (sin enviar) un caso ficticio.
  Comentar cada campo en directo. Útil para que vean qué información
  exacta hace falta cuando descubran algo de verdad.



================================================================================
 EXTRA - RECURSOS PARA SEGUIR APRENDIENDO
================================================================================

NEWSLETTERS Y FEEDS
  • Risky Business         https://risky.biz
  • Schneier on Security   https://www.schneier.com
  • Krebs on Security      https://krebsonsecurity.com
  • The Hacker News        https://thehackernews.com
  • Bleeping Computer      https://www.bleepingcomputer.com

PODCASTS
  • Darknet Diaries        https://darknetdiaries.com
  • Risky Business Podcast https://risky.biz/podcasts
  • Tierra de Hackers (ES) https://www.tierradehackers.com

CANALES YOUTUBE / TWITCH
  • LiveOverflow
  • IppSec
  • John Hammond
  • S4vitar (ES)
  • HackPlayers (ES)

COMUNIDADES
  • Reddit r/netsec        https://www.reddit.com/r/netsec
  • Reddit r/cybersecurity https://www.reddit.com/r/cybersecurity
  • HackPlayers (ES)       https://www.hackplayers.com



================================================================================
 FIN DEL DOCUMENTO
================================================================================
