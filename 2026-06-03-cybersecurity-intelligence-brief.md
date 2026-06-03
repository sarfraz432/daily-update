# Daily Cybersecurity Intelligence Brief - 2026-06-03

Reporting window: last 24 hours ending 2026-06-03 11:05 IST / 2026-06-03 05:35 UTC. Sources reviewed include The Hacker News Threat Intelligence section, Sekoia.io, Android Security Bulletin, NVD, CISA/KEV-indexed reporting, SecurityWeek, BleepingComputer, GoDaddy Security, and accessible indexed X.com security results. Direct X.com browser review was blocked by browser security policy; indexed X.com results did not expose unique, independently verifiable IOCs today.

## 1. 🧠 Executive Summary

- Gamaredon remains today's highest-confidence malware/campaign item: THN Threat Intelligence amplified Sekoia's GammaPhish/GammaWorm/GammaLoad/GammaSteel ecosystem analysis, including WinRAR path traversal exploitation, NTFS ADS persistence, scheduled-task execution, Telegram/Cloudflare dead-drop resolution, and active Ukrainian targeting.
- Google patched Android CVE-2025-48595, a high-severity Framework integer-overflow elevation-of-privilege issue under limited, targeted exploitation across Android 14, 15, 16, and 16 QPR2.
- Oracle WebLogic CVE-2024-21182 was added to active-exploitation coverage, with CISA-directed remediation by 2026-06-04 and public PoC history increasing risk to internet-exposed WebLogic estates.
- WordPress malware abusing Steam Community profile comments for C2 was re-reported in the last 24 hours; GoDaddy's technical writeup documents ~1,980 infected sites, invisible-Unicode steganography, AES-256-CTR/PBKDF2/HMAC-protected payloads, JavaScript injection, and a cookie-authenticated PHP backdoor.
- No major new ransomware intrusion with fresh public IOCs was observed today; the dominant pattern is trusted-platform abuse for C2 plus rapid exploitation of already-patched enterprise/mobile flaws.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2025-48595 - Android Framework Integer Overflow

- Severity: High; CVSS 8.4.
- Affected systems: Android 14, Android 15, Android 16, and Android 16 QPR2 devices prior to the 2026-06 security patch levels.
- Exploitation status: In-the-wild; Google states there are indications of limited, targeted exploitation.
- Technical root cause: Integer overflow in Android Framework code paths that can lead to code execution and local elevation of privilege without user interaction.
- Operational risk: Likely high-value-device targeting; prioritize executive, government, journalist, legal, telecom, and mobile-admin fleets.

### CVE-2024-21182 - Oracle WebLogic Server Unauthorized Data Access

- Severity: High; CVSS 7.5.
- Affected systems: Oracle WebLogic Server 12.2.1.4.0 and 14.1.1.0.0, fixed in Oracle's July 2024 Critical Patch Update.
- Exploitation status: In-the-wild; CISA added the flaw to KEV on 2026-06-01 and ordered federal remediation by 2026-06-04.
- Technical root cause: Easily exploitable unauthenticated network vulnerability reachable via T3/IIOP that can compromise Oracle WebLogic Server confidentiality.
- Operational risk: Unauthorized access to critical data on WebLogic servers; public PoCs have existed since disclosure, making exposed legacy deployments attractive targets.

### CVE-2025-8088 - WinRAR Path Traversal Used by Gamaredon

- Severity: Critical in campaign context.
- Affected systems: WinRAR versions before 7.13 on Windows endpoints handling attacker-controlled RAR archives.
- Exploitation status: In-the-wild; Sekoia and THN report Gamaredon weaponization against Ukrainian government, military, and critical infrastructure targets.
- Technical root cause: Archive path traversal allowing malicious content to be written into persistence locations such as the user Startup directory.
- Operational risk: User-assisted archive extraction can plant HTA/VBScript stages that execute at next logon and bootstrap Gamma malware.

## 3. 🦠 Malware & Campaign Analysis

### Gamaredon Gamma Ecosystem

- Malware name / family: GammaPhish, GammaLoad, GammaWorm, GammaSteel, and possible GammaWipe.
- Threat actor: Gamaredon / UAC-0010 / ACTINUM / Armageddon / BlueAlpha; publicly linked by Ukrainian authorities to Russia's FSB.
- Initial access vector: Spear-phishing or archive-delivered xHTML lure that smuggles a weaponized RAR exploiting CVE-2025-8088.
- Execution chain:
  1. Victim opens `1_13_5_1691_09.12.2025.xhtml`; lure shows benign text and beacons via 1x1 tracking pixel.
  2. JavaScript checks Windows user-agent/platform identifiers and HTML-smuggles `2_14_6_1033_09.12.2025.rar`.
  3. User extracts the RAR; path traversal writes an HTA into `%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup\`.
  4. On next logon, the HTA launches `mshta.exe` against a remote Supabase-hosted payload URL using a fake `www.bbc.com@...` authority string.
  5. GammaLoad VBScript stages fingerprint the host, update registry-stored C2 configuration, and fetch arbitrary VBScript payloads.
  6. GammaWorm stores modules in NTFS Alternate Data Streams, propagates through USB/network drives, hides real directories, and replaces them with malicious LNKs.
  7. GammaSteel, acquired by Sekoia via replayed GammaLoad requests, is a PowerShell stealer that stores 71 DPAPI-encrypted registry modules and exfiltrates target files to S3-compatible storage or fallback C2.
- Persistence technique: `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce\ExplorerGuard`; scheduled tasks `\DiskDiagnostic\Microsoft\Windows\DiskDiagnosticDataCollector`, `\CertificateServicesClient\SystemTask\SilentCleanup`, and `\InstallService\ScanForUpdatesServer\SmartRetry`; NTFS ADS streams `%USERPROFILE%:GTR`, `%USERPROFILE%:save`, `%USERPROFILE%:URL`, `%USERPROFILE%:LNK`, `%USERPROFILE%:SERVER`.
- C2 communication method: HTTP(S) to dead-drop resolvers on `graph.org`, `workers.dev`, `teletype.in`, `telegra.ph`, Telegram, Cloudflare tunnels, and operator-controlled domains/IPs; GammaWorm sends host fingerprints in randomized HTTP headers and treats HTTP 404 bodies as configuration updates.

### WordPress Steam-Comment C2 Malware

- Malware name / family: Unnamed WordPress malware/backdoor using Steam Community profiles for C2.
- Threat actor: Unknown.
- Initial access vector: Not confirmed; GoDaddy assesses stolen WordPress/FTP/SFTP credentials, vulnerable themes/plugins, or supply-chain compromise as plausible entry points.
- Execution chain:
  1. Attacker plants obfuscated PHP in WordPress plugin/theme paths.
  2. The first-stage code fetches Steam Community profile comments during page loads.
  3. Decoder strips visible text and maps invisible Unicode characters (`U+200C`, `U+200D`, `U+2061` through `U+2064`) into binary payload data.
  4. Optional crypto layer uses AES-256-CTR with PBKDF2 key derivation and HMAC authentication.
  5. Decoded instructions build a remote JavaScript URL, commonly masquerading as a library such as `lodash.core.min.js`.
  6. WordPress pages inject the malicious script for visitors.
  7. A cookie-authenticated server-side backdoor accepts POSTed base64 PHP to modify plugin/theme files.
- Persistence technique: Server-side PHP backdoor across WordPress plugin/theme files; suspicious script handle `asahi-jquery-min-bundle` observed in public cleanup guidance.
- C2 communication method: Steam Community profile comments as dead-drop C2; external JavaScript retrieval from attacker-controlled infrastructure such as `hello-mywordl[.]info`.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "104.194.140[.]6"
  ],
  "domains": [
    "iiwdsxwamylbwwsoyrmj.supabase[.]co",
    "graph[.]org",
    "bold.zsjtn41091.workers[.]dev",
    "teletype[.]in",
    "quitethepastry[.]ru",
    "telegra[.]ph",
    "t[.]me",
    "www.telegram[.]me",
    "efficiency-planes-emotions-fascinating.trycloudflare[.]com",
    "moment-cat-qld-place.trycloudflare[.]com",
    "steamcommunity[.]com",
    "hello-mywordl[.]info"
  ],
  "hashes": [
    "1794369214b7f62e70a0485e61335c61",
    "8e1624d110c090ff57d4b493a9107c66"
  ],
  "urls": [
    "hxxp://iiwdsxwamylbwwsoyrmj.supabase[.]co/functions/v1/clever-responder/KokonGoogle_09.12.2025",
    "hxxps://www.bbc.com@iiwdsxwamylbwwsoyrmj.supabase.co/functions/v1/clever-responder/GurGoogle_09.12.2025/audience/capture.pdf",
    "hxxps://graph[.]org/kyjfkyr-12-06",
    "hxxps://bold.zsjtn41091.workers[.]dev",
    "hxxps://teletype[.]in/@myrain/Xh1Lta2Ccro",
    "hxxps://quitethepastry[.]ru",
    "hxxps://telegra[.]ph/f8bfl6sp-01-02",
    "hxxps://www.telegram[.]me/s/oberfarir",
    "hxxps://efficiency-planes-emotions-fascinating.trycloudflare[.]com/@myrain/Xh1Lta2Ccro?84wtj9ob-01-31"
  ],
  "mutex": [
    "%USERPROFILE%:save"
  ]
}
```

Registry/scheduled-task observables: `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce\ExplorerGuard`; `HKCU\Console\WindowsUpdates`; `HKCU\Console\WindowsResponby`; `HKCU\Console\WindowsDetect`; `HKCU\Console\URLTeletype`; `HKCU\Console\WindowsTelegra`; `HKCU\Console\URLTelegra`; `HKCU\Console\IpURL`; `\DiskDiagnostic\Microsoft\Windows\DiskDiagnosticDataCollector`; `\CertificateServicesClient\SystemTask\SilentCleanup`; `\InstallService\ScanForUpdatesServer\SmartRetry`.

## 5. 🔗 Technical Resources & Blogs

- The Hacker News Threat Intelligence section: https://thehackernews.com/search/label/Threat%20Intelligence
- THN: Gamaredon Exploits WinRAR to Deliver GammaWorm and GammaSteel Against Ukraine: https://thehackernews.com/2026/06/gamaredon-exploits-winrar-to-deliver.html
- Sekoia.io: FSB's matryoshka #1/3 - GammaPhish and GammaWorm: https://blog.sekoia.io/fsbs-matryoshka-1-3-gamaredons-gifts-that-keeps-unpacking-gammaphish-and-gammaworm/
- Android Security Bulletin - June 2026: https://source.android.com/docs/security/bulletin/2026/2026-06-01
- NVD: CVE-2025-48595: https://nvd.nist.gov/vuln/detail/CVE-2025-48595
- SecurityWeek: Oracle WebLogic Vulnerability Exploited in the Wild: https://www.securityweek.com/oracle-weblogic-vulnerability-exploited-in-the-wild/
- BleepingComputer: CISA flags two-year-old Oracle flaw as actively exploited: https://www.bleepingcomputer.com/news/security/cisa-orders-feds-to-patch-actively-exploited-oracle-weblogic-flaw/
- GoDaddy Security: Malware Targeting WordPress Abuses Steam Community Profiles: https://www.godaddy.com/resources/news/malware-targeting-wordpress-abuses-steam-community-profiles
- BleepingComputer: WordPress malware campaign hides payloads in Steam profiles: https://www.bleepingcomputer.com/news/security/wordpress-malware-campaign-hides-payloads-in-steam-profiles/

## 6. 🧪 Sandbox / Sample Links

- VirusTotal search - GammaPhish MD5: https://www.virustotal.com/gui/search/1794369214b7f62e70a0485e61335c61
- VirusTotal search - GammaWorm MD5: https://www.virustotal.com/gui/search/8e1624d110c090ff57d4b493a9107c66
- Any.Run links: Not observed today.
- Hybrid Analysis links: Not observed today.
- MalwareBazaar sample links: Not observed today.
- GitHub PoC repositories: Public PoCs are reported for CVE-2024-21182, but no specific high-confidence repo was validated today.

## 7. 🛡️ Detection & Mitigation

### Gamaredon Gamma

- Patch or remove WinRAR versions older than 7.13; block extraction of archive entries that resolve into Startup, PowerShell profile, DLL search-order, or user-writable persistence paths.
- Hunt for `mshta.exe` command lines containing remote URLs, especially `www.bbc.com@` authority abuse, Supabase function paths, or execution from Startup HTA files.
- Monitor ADS creation under `%USERPROFILE%` with stream names `GTR`, `save`, `URL`, `LNK`, and `SERVER`; run `dir /R` or EDR ADS telemetry on suspected hosts.
- Alert on `wscript.exe` executing paths with colon-delimited ADS syntax where the colon is not a drive separator, for example `wscript.exe "C:\Users\<user>:GTR"`.
- Detect scheduled-task creation under `\DiskDiagnostic\Microsoft\Windows\`, `\CertificateServicesClient\SystemTask\`, and `\InstallService\ScanForUpdatesServer\` by non-administrative user contexts.
- Network hunt: recurring requests from `wscript.exe`, `mshta.exe`, or hidden PowerShell to `graph.org`, `teletype.in`, `telegra.ph`, `t.me`, `workers.dev`, `trycloudflare.com`, and Supabase function URLs.
- Remediation: Sekoia recommends full wipe/rebuild for infected hosts because DDR-driven payload refresh and fallback mechanisms make partial cleaning unreliable.

### Android CVE-2025-48595

- Deploy Android 2026-06-01 or 2026-06-05 patch levels; prioritize managed high-risk devices and Android 14+ fleets.
- Hunt mobile telemetry for unexpected privilege-boundary transitions, crashes in Framework components, sudden permission changes, and suspicious app install chains preceding privilege escalation.
- Treat suspected exploitation as targeted compromise; collect device forensic images before wipe where legal/operationally feasible.

### Oracle WebLogic CVE-2024-21182

- Apply Oracle's July 2024 CPU or later to WebLogic Server 12.2.1.4.0 and 14.1.1.0.0; internet-facing instances should be patched or removed from exposure before 2026-06-04.
- Restrict T3/IIOP and administrative interfaces to known management networks; remove direct internet access to WebLogic management endpoints.
- Hunt for anomalous T3/IIOP access, unauthenticated requests to exposed WebLogic services, and unexpected reads of sensitive application data or configuration repositories.
- Review exposed WebLogic inventory against CISA KEV deadlines and treat unpatched externally reachable systems as probable compromise candidates.

### WordPress Steam-Comment C2 Malware

- Search plugin/theme directories for `steamcommunity.com`, invisible Unicode arrays (`U+200C`, `U+200D`, `U+2061-U+2064`), `hash_pbkdf2`, `openssl_decrypt`, `AES-256-CTR`, `CURLOPT_SSL_VERIFYPEER false`, and `asahi-jquery-min-bundle`.
- Hunt web logs for outbound retrieval of Steam Community profile pages from WordPress servers and visitor-facing script loads from `hello-mywordl[.]info`.
- Remove backdoored PHP from plugins/themes, rotate WordPress admin, FTP/SFTP, hosting-panel, and database credentials, and restore from a known-clean backup where possible.
- Detection idea: flag PHP that decodes invisible Unicode from third-party social/gaming profile comments, then emits script tags or accepts base64 PHP via authenticated POST.

## 8. 📊 Trends & Insights

- Dead-drop C2 via trusted public platforms is a clear theme today: Gamaredon uses Telegram/Telegraph/Cloudflare-style DDRs, while WordPress malware hides instructions inside Steam profile comments.
- Attackers are shifting durable configuration away from hardcoded domains and into updateable public-content channels, increasing the value of process-aware egress analytics over static blocklists.
- Enterprise exposure remains dominated by n-day exploitation: CVE-2024-21182 was patched in 2024 but is now a 2026 KEV priority, reinforcing that stale middleware remains an active initial-access surface.
- Mobile targeted exploitation continues to surface through sparse vendor language. Android CVE-2025-48595 should be handled as a targeted-spyware-style risk even without public actor attribution.
- No fresh public ransomware leak-site event with technical IOCs was observed in today's high-confidence source set.
