================================================================================
  MÓDULO M2 - EXPLOTACIÓN DE CVEs
  Máquinas DockerLabs basadas en CVEs reales
  BigSchool - Escuela de Ciberseguridad
================================================================================

AUDITORÍA REALIZADA SOBRE:
  https://github.com/albertomarcostic/DockerLabs-WriteUps  (30 máquinas)

RESULTADO:
  9 máquinas usan CVEs reales con identificador asignado.
  El resto usan vulnerabilidades de configuración o vectores genéricos
  (LFI, SQLi, file upload, weak creds, sudo misconfig...).


────────────────────────────────────────────────────────────────────────────────
  ROADMAP RECOMENDADO PARA EL MÓDULO (4h)
────────────────────────────────────────────────────────────────────────────────

  Cada máquina ↔ un CVE con CVSS, advisory, exploit público y writeup.
  Orden de menor a mayor dificultad y por relevancia didáctica.

  V2.1 → Metodología general (cómo investigar un CVE)
  V2.2 → FirstHacking      [Muy fácil] - CVE-2011-2523  (vsftpd backdoor)
  V2.3 → ChocolateLovers   [Fácil]     - CVE-2015-6033  (Nibbleblog upload)
  V2.4 → HiddenCat         [Media]     - CVE-2020-1938  (Tomcat Ghostcat)
  V2.5 → Eclipse           [Media]     - CVE-2019-0193  (Apache Solr RCE)
  V2.6 → Move              [Media]     - CVE-2021-43798 (Grafana path traversal)
  V2.7 → Fooding           [Media]     - CVE-2023-46604 (ActiveMQ RCE)
  V2.8 → Secretjenkins     [Media]     - CVE-2024-23897 (Jenkins LFI)
  V2.9 → SummerVibes       [Media]     - CVE-2024-27622 (CMS Made Simple RCE)
  V2.10 → Asucar           [Media]     - CVE-2018-7475  (WP Site Editor LFI)


================================================================================
  1. FirstHacking
================================================================================
DIFICULTAD     : Muy fácil
CVE            : CVE-2011-2523
PRODUCTO       : vsftpd 2.3.4
TIPO           : Backdoor / RCE
HISTORIA       : El propio mantenedor del proyecto fue víctima de un
                 supply-chain attack. Alguien subió una versión maliciosa
                 con backdoor activado al hacer login con usuario `:)`
                 (smiley). Cualquier user con `:)` abre puerto 6200 con
                 shell de root.

ENLACE MÁQUINA : https://dockerlabs.es/
WRITEUP        : https://github.com/albertomarcostic/DockerLabs-WriteUps/blob/main/M%C3%A1quina%20FirstHacking.md

REFERENCIAS OFICIALES:
  NVD          : https://nvd.nist.gov/vuln/detail/CVE-2011-2523
  CVE.org      : https://www.cve.org/CVERecord?id=CVE-2011-2523
  Exploit-DB   : https://www.exploit-db.com/exploits/49757
  Metasploit   : https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor/

CSVS           : 9.8 Critical (3.x) / 10.0 (2.0)
DIDÁCTICO PARA: Mostrar qué es un supply-chain attack y por qué auditar
                las dependencias importa.


================================================================================
  2. ChocolateLovers
================================================================================
DIFICULTAD     : Fácil
CVE            : CVE-2015-6033
PRODUCTO       : Nibbleblog 4.0.3 - plugin "My Image"
TIPO           : Arbitrary File Upload → RCE (autenticado)
RESUMEN        : Tras login con creds débiles (admin:admin), el plugin
                 "My Image" permite subir archivos sin validación de
                 extensión. Subir .php = RCE inmediato.

ENLACE MÁQUINA : https://dockerlabs.es/
WRITEUP        : https://github.com/albertomarcostic/DockerLabs-WriteUps/blob/main/M%C3%A1quina%20ChocolateLovers.md

REFERENCIAS OFICIALES:
  NVD          : https://nvd.nist.gov/vuln/detail/CVE-2015-6033
  CVE.org      : https://www.cve.org/CVERecord?id=CVE-2015-6033
  Metasploit   : https://www.rapid7.com/db/modules/exploit/multi/http/nibbleblog_file_upload/

CVSS           : 6.5 Medium (Authenticated)
DIDÁCTICO PARA: Mostrar cómo un CVE "post-auth" sigue siendo crítico si
                el panel admin tiene creds por defecto.


================================================================================
  3. HiddenCat
================================================================================
DIFICULTAD     : Media
CVE            : CVE-2020-1938 - "Ghostcat"
PRODUCTO       : Apache Tomcat 9.0.30 (puerto AJP 8009)
TIPO           : Local File Inclusion vía AJP / RCE bajo ciertas condiciones
RESUMEN        : El protocolo AJP permite leer cualquier archivo de la
                 webapp por defecto (web.xml, configs...) y bajo ciertas
                 condiciones llegar a RCE. Afectó a millones de servidores.

ENLACE MÁQUINA : https://dockerlabs.es/
WRITEUP        : https://github.com/albertomarcostic/DockerLabs-WriteUps/blob/main/M%C3%A1quina%20HiddenCat.md

REFERENCIAS OFICIALES:
  NVD          : https://nvd.nist.gov/vuln/detail/CVE-2020-1938
  CVE.org      : https://www.cve.org/CVERecord?id=CVE-2020-1938
  Apache       : https://tomcat.apache.org/security-9.html
  Exploit-DB   : https://www.exploit-db.com/exploits/48143
  Análisis CN  : https://www.chaitin.cn/en/ghostcat

CVSS           : 9.8 Critical
DIDÁCTICO PARA: Enseñar qué es AJP, por qué hay que cerrar puertos
                "internos" expuestos, y cómo un CVE puede vivir 13 años
                sin que nadie lo encuentre (Tomcat existe desde 2007).


================================================================================
  4. Eclipse
================================================================================
DIFICULTAD     : Media
CVE            : CVE-2019-0193
PRODUCTO       : Apache Solr - puerto 8983
TIPO           : Remote Code Execution vía DataImportHandler
RESUMEN        : El módulo DataImportHandler permite ejecutar código
                 arbitrario JS via configuraciones manipuladas. Solo si
                 está activo (no por defecto, pero muy común).

ENLACE MÁQUINA : https://dockerlabs.es/
WRITEUP        : https://github.com/albertomarcostic/DockerLabs-WriteUps/blob/main/M%C3%A1quina%20Eclipse.md

REFERENCIAS OFICIALES:
  NVD          : https://nvd.nist.gov/vuln/detail/CVE-2019-0193
  CVE.org      : https://www.cve.org/CVERecord?id=CVE-2019-0193
  Apache       : https://lists.apache.org/thread/qbwsm38tdvz0lxotnssdy4m2qo8ipf7w
  Exploit-DB   : https://www.exploit-db.com/exploits/47572

CVSS           : 8.8 High
DIDÁCTICO PARA: Mostrar que un servicio "interno" como Solr expuesto a
                internet es un riesgo crítico. Caso real frecuente.


================================================================================
  5. Move
================================================================================
DIFICULTAD     : Media
CVE            : CVE-2021-43798
PRODUCTO       : Grafana 8.0.0-beta1 a 8.3.0
TIPO           : Path Traversal preautenticado (Arbitrary File Read)
RESUMEN        : El endpoint /public/plugins/:pluginId/ no sanea el path.
                 Permite leer cualquier fichero del servidor sin auth.
                 Descubierto por jordyv y publicado como 0-day en HackerOne.

ENLACE MÁQUINA : https://dockerlabs.es/
WRITEUP        : https://github.com/albertomarcostic/DockerLabs-WriteUps/blob/main/M%C3%A1quina%20Move.md

REFERENCIAS OFICIALES:
  NVD          : https://nvd.nist.gov/vuln/detail/CVE-2021-43798
  CVE.org      : https://www.cve.org/CVERecord?id=CVE-2021-43798
  HackerOne    : https://hackerone.com/reports/1427086
  Detectify    : https://labs.detectify.com/security-guidance/how-i-found-the-grafana-zero-day-path-traversal-exploit-that-gave-me-access-to-your-logs/
  Exploit-DB   : https://www.exploit-db.com/exploits/50581

CVSS           : 7.5 High
DIDÁCTICO PARA: ★ EJEMPLO PERFECTO de disclosure responsable + writeup
                investigador (Detectify) + reporte HackerOne público.
                Material ORO PURO para enseñar cómo se reporta un CVE.


================================================================================
  6. Fooding
================================================================================
DIFICULTAD     : Media
CVE            : CVE-2023-46604
PRODUCTO       : Apache ActiveMQ <= 5.15.15 / 5.16-17 / 5.18.2 (puerto 61616)
TIPO           : Remote Code Execution vía OpenWire protocol
RESUMEN        : Deserialización insegura en el protocolo OpenWire permite
                 instanciar clases arbitrarias y ejecutar comandos.
                 Explotada en ataques reales por ransomware HelloKitty.

ENLACE MÁQUINA : https://dockerlabs.es/
WRITEUP        : https://github.com/albertomarcostic/DockerLabs-WriteUps/blob/main/M%C3%A1quina%20Fooding.md

REFERENCIAS OFICIALES:
  NVD          : https://nvd.nist.gov/vuln/detail/CVE-2023-46604
  CVE.org      : https://www.cve.org/CVERecord?id=CVE-2023-46604
  Apache       : https://activemq.apache.org/security-advisories.data/CVE-2023-46604-announcement.txt
  CISA KEV     : https://www.cisa.gov/known-exploited-vulnerabilities-catalog
                 (incluida en KEV - explotada activamente)

CVSS           : 10.0 Critical
DIDÁCTICO PARA: Vuln crítico MODERNO. Ejemplo de cómo un CVE pasa de
                advisory a explotación masiva por grupos de ransomware
                en cuestión de días.


================================================================================
  7. Secretjenkins
================================================================================
DIFICULTAD     : Media
CVE            : CVE-2024-23897
PRODUCTO       : Jenkins (CLI) <= 2.441 / LTS <= 2.426.2
TIPO           : Arbitrary File Read vía args4j (puede llegar a RCE)
RESUMEN        : El parser de argumentos CLI args4j expande el carácter
                 "@" como referencia a archivo. Un atacante puede leer
                 cualquier fichero del servidor (incluyendo SSH keys,
                 secrets, credentials.xml).

ENLACE MÁQUINA : https://dockerlabs.es/
WRITEUP        : https://github.com/albertomarcostic/DockerLabs-WriteUps/blob/main/M%C3%A1quina%20Secretjenkins.md

REFERENCIAS OFICIALES:
  NVD          : https://nvd.nist.gov/vuln/detail/CVE-2024-23897
  CVE.org      : https://www.cve.org/CVERecord?id=CVE-2024-23897
  Jenkins      : https://www.jenkins.io/security/advisory/2024-01-24/
  Sonar        : https://www.sonarsource.com/blog/excessive-expansion-uncovering-critical-security-vulnerabilities-in-jenkins/

CVSS           : 9.8 Critical
DIDÁCTICO PARA: ★ EJEMPLO RECIENTE de disclosure coordinado. Reportada
                por SonarSource. Tiene blog técnico detallado del
                investigador. Perfecta para enseñar el formato del
                writeup post-fix.


================================================================================
  8. SummerVibes
================================================================================
DIFICULTAD     : Media
CVE            : CVE-2024-27622
PRODUCTO       : CMS Made Simple 2.2.19
TIPO           : Authenticated RCE vía User Defined Tags (UDT)
RESUMEN        : La feature "User Defined Tags" del panel admin permite
                 inyectar PHP arbitrario que se ejecuta cuando se llama
                 al tag desde la web pública.

ENLACE MÁQUINA : https://dockerlabs.es/
WRITEUP        : https://github.com/albertomarcostic/DockerLabs-WriteUps/blob/main/M%C3%A1quina%20SummerVibes.md

REFERENCIAS OFICIALES:
  NVD          : https://nvd.nist.gov/vuln/detail/CVE-2024-27622
  CVE.org      : https://www.cve.org/CVERecord?id=CVE-2024-27622
  CMS Made Simple : https://www.cmsmadesimple.org/

CVSS           : 7.2 High (Authenticated)
DIDÁCTICO PARA: CVE muy reciente (2024). Enseñar diferencia entre
                "Authenticated RCE" vs "Unauthenticated RCE" y por qué
                el primero sigue siendo grave.


================================================================================
  9. Asucar
================================================================================
DIFICULTAD     : Media
CVE            : CVE-2018-7475
PRODUCTO       : WordPress plugin "Site Editor" 1.1.1 (y anteriores)
TIPO           : Local File Inclusion (preautenticado)
RESUMEN        : El plugin tiene un endpoint que recibe un parámetro
                 "ajax_path" sin validación, permitiendo leer cualquier
                 fichero del servidor. Plugin abandonado y nunca parcheado.

ENLACE MÁQUINA : https://dockerlabs.es/
WRITEUP        : https://github.com/albertomarcostic/DockerLabs-WriteUps/blob/main/M%C3%A1quina%20Asucar.md

REFERENCIAS OFICIALES:
  NVD          : https://nvd.nist.gov/vuln/detail/CVE-2018-7475
  CVE.org      : https://www.cve.org/CVERecord?id=CVE-2018-7475
  Exploit-DB   : https://www.exploit-db.com/exploits/44340
  WPScan       : https://wpscan.com/vulnerability/9008

CVSS           : 7.5 High
DIDÁCTICO PARA: Mostrar el problema de plugins abandonados. Aún hoy hay
                miles de WordPress con este plugin instalado y vulnerable.
                Enseñar a usar wpscan para detectarlo.


================================================================================
  RESUMEN VISUAL - TABLA COMPLETA
================================================================================

#   MÁQUINA          DIF.   CVE              SERVICIO                    CVSS
─── ──────────────── ────── ──────────────── ─────────────────────────── ────
1   FirstHacking     Muy F. CVE-2011-2523    vsftpd 2.3.4 backdoor       9.8
2   ChocolateLovers  Fácil  CVE-2015-6033    Nibbleblog 4.0.3 upload     6.5
3   Asucar           Media  CVE-2018-7475    WP Site Editor plugin LFI   7.5
4   Eclipse          Media  CVE-2019-0193    Apache Solr RCE             8.8
5   HiddenCat        Media  CVE-2020-1938    Tomcat 9.0.30 Ghostcat      9.8
6   Move             Media  CVE-2021-43798   Grafana 8.x path traversal  7.5
7   Fooding          Media  CVE-2023-46604   ActiveMQ OpenWire RCE      10.0
8   Secretjenkins    Media  CVE-2024-23897   Jenkins CLI file read       9.8
9   SummerVibes      Media  CVE-2024-27622   CMS Made Simple UDT RCE     7.2


────────────────────────────────────────────────────────────────────────────────
  ENLACES GLOBALES DOCKERLABS
────────────────────────────────────────────────────────────────────────────────

  • Plataforma DockerLabs (descarga de máquinas)
    https://dockerlabs.es

  • Repositorio principal de writeups (Alberto Marcos)
    https://github.com/albertomarcostic/DockerLabs-WriteUps

  • Writeups alternativos (BeaFN28 GitBook)
    https://beafn28.gitbook.io/beafn28/writeups/dockerlabs

  • Repositorio Er3b0r (más writeups)
    https://github.com/Er3b0r/Dockerlabs_WriteUPs

  • Pyth0nK1d Medium (writeups en español)
    https://pyth0nk1d.medium.com

  • Alv-fh Blog
    https://alv-fh.github.io/posts/writeups-dockerlabs/

  • CTF-Writeups (Bitxogm)
    https://github.com/Bitxogm/CTF-Writeups


────────────────────────────────────────────────────────────────────────────────
  MÁQUINAS DOCKERLABS QUE NO SON CVE (PARA REFERENCIA)
────────────────────────────────────────────────────────────────────────────────

  Estas no entran en el módulo de CVEs pero sí pueden usarse en otros
  módulos (Hardening, Escalada de privilegios, Web pentesting general):

  • -Pn               → Tomcat default creds + WAR upload
  • Amor              → SSH brute + steg + sudo ruby
  • AnonymousPingu    → Anonymous FTP + sudo abuse
  • BreakMySSH        → SSH brute force (rockyou)
  • CapyPenguin       → MySQL brute + sudo nano
  • Domain            → SMB + PHP shell + sudo nano
  • Fileception       → FTP anon + steganography
  • HackPenguin       → Steg + KeePass + cron writable
  • HackTheHeaven     → LFI + race condition upload
  • Hidden            → File upload bypass + sudo chains
  • Inclusion         → LFI generic + sudo PHP
  • Injection         → SQLi login bypass
  • Library           → Python library hijacking
  • PyRed             → Python interpreter exposed
  • Stranger          → FTP brute + RSA decrypt
  • StrongJenkins     → Jenkins brute + Script Console
  • Trust             → Sudo vim
  • Upload            → Unrestricted file upload
  • Vacaciones        → Source comments + sudo ruby
  • WalkingCMS        → WP brute + theme editor + SUID env
  • WhereIsMyWebShell → Custom RCE via PHP system()


────────────────────────────────────────────────────────────────────────────────
  PROPUESTA DIDÁCTICA - RECORRIDO PEDAGÓGICO 4H
────────────────────────────────────────────────────────────────────────────────

  HORA 1 - TEORÍA + DEMO METODOLOGÍA (60 min)
    • Qué es un CVE (10 min)
    • CVSS - calculadora (10 min)
    • CNAs / vendor / INCIBE / MITRE (10 min)
    • Cómo se reporta (10 min)
    • Demo metodología nmap → searchsploit → nuclei (20 min)

  HORA 2 - CVEs CLÁSICOS (60 min)
    • FirstHacking          (vsftpd backdoor - CVE-2011-2523)   20 min
    • ChocolateLovers       (Nibbleblog - CVE-2015-6033)        20 min
    • Asucar                (WP Site Editor - CVE-2018-7475)    20 min

  HORA 3 - CVEs MODERNOS DE SERVICIOS (60 min)
    • HiddenCat             (Tomcat Ghostcat - CVE-2020-1938)   20 min
    • Eclipse               (Apache Solr - CVE-2019-0193)       20 min
    • Move                  (Grafana - CVE-2021-43798)          20 min

  HORA 4 - CVEs CRÍTICOS RECIENTES + REPORTE (60 min)
    • Fooding               (ActiveMQ - CVE-2023-46604)         15 min
    • Secretjenkins         (Jenkins CLI - CVE-2024-23897)      15 min
    • SummerVibes           (CMS Made Simple - CVE-2024-27622)  10 min
    • Cómo se reporta un CVE oficialmente (vía INCIBE/MITRE)    20 min


================================================================================
 FIN DEL DOCUMENTO
================================================================================
