# DAILY CYBERSECURITY INTELLIGENCE BRIEF
Date: 2026-04-22 (IST)
Collection window: 2026-04-20 to 2026-04-22

## 1. 🧠 Executive Summary
- **Ransomware scale-up:** Check Point reported a **SystemBC-linked infrastructure** tied to **The Gentlemen RaaS**, with telemetry from a C2 panel indicating **1,570+ compromised organizations** globally.
- **Mobile financial theft campaign remains active:** ESET documented a **new NGate variant** that trojanizes **HandyPay** to relay NFC card data and steal PINs in Brazil-focused fraud operations.
- **OT-focused malware development observed:** Darktrace analyzed **ZionSiphon**, a politically themed malware with Israel-focused targeting and Modbus-centric sabotage logic for water/desalination environments.
- **Fresh in-the-wild vulnerability pressure:** THN reported **8 newly KEV-listed exploited CVEs** (PaperCut, TeamCity, Kentico, Quest KACE, Zimbra, Cisco SD-WAN Manager).
- **Why this matters:** Attackers are combining mature intrusion playbooks (domain-wide ransomware deployment, proxy C2) with sector-specific malware experimentation (OT and NFC payment abuse), increasing blast radius across enterprise IT and critical systems.

## 2. 🚨 Active Threats & Vulnerabilities

### Threat 1: Multi-vendor KEV additions (Apr 21, 2026)
- **Name / CVE:** CVE-2023-27351, CVE-2024-27199, CVE-2025-2749, CVE-2025-32975, CVE-2025-48700, CVE-2026-20122, CVE-2026-20128, CVE-2026-20133
- **Severity:** Critical/High (mixed set; highest cited CVSS 10.0 for CVE-2025-32975)
- **Affected systems:** PaperCut NG/MF, JetBrains TeamCity, Kentico Xperience, Quest KACE SMA, Synacor Zimbra, Cisco Catalyst SD-WAN Manager
- **Exploitation status:** **In-the-wild** (KEV inclusion indicates active exploitation evidence)
- **Technical root cause:** improper authentication, path traversal, stored/recoverable credentials exposure, XSS, privileged API misuse, sensitive information exposure

### Threat 2: The Gentlemen ransomware operation using SystemBC infrastructure
- **Name / CVE:** The Gentlemen RaaS + SystemBC (no CVE)
- **Severity:** Critical
- **Affected systems:** Windows/Linux/NAS/BSD/ESXi enterprise environments
- **Exploitation status:** **In-the-wild** (incident response case + live C2 victim telemetry)
- **Technical root cause:** credential abuse + post-compromise remote execution; defense impairment and GPO-based ransomware distribution

## 3. 🦠 Malware & Campaign Analysis

### Campaign A: The Gentlemen + SystemBC
- **Malware name / family:** The Gentlemen ransomware, SystemBC proxy malware, Cobalt Strike tooling
- **Threat actor:** The Gentlemen affiliate ecosystem (exact affiliate identity not disclosed)
- **Initial access vector:** Not fully confirmed; evidence points to abuse of internet-facing services and/or compromised credentials
- **Execution chain (step-by-step):**
  1. Domain Controller foothold with domain-admin level activity
  2. Remote payload transfer over ADMIN$ shares via RPC
  3. SystemBC (`socks.exe`) staging and C2 attempts
  4. Cobalt Strike beaconing via `rundll32.exe`
  5. Defender weakening and policy manipulation (`gpupdate /force`)
  6. Broad propagation of ransomware payload (`grand.exe`/`r.exe`)
  7. GPO-triggered near-simultaneous encryption
- **Persistence technique:** AnyDesk installation + RDP enablement; GPO-based execution at scale
- **C2 communication method:** RC4-encrypted SystemBC SOCKS5 tunnels; Cobalt Strike HTTP/HTTPS beaconing

### Campaign B: NGate (trojanized HandyPay variant)
- **Malware name / family:** NGate / NFSkate (Android)
- **Threat actor:** Unknown (single actor likely based on shared infra)
- **Initial access vector:** Social engineering via fake lottery portal and fake Google Play page; manual sideloading
- **Execution chain (step-by-step):**
  1. Victim lured to fake campaign pages
  2. Trojanized HandyPay APK installed from outside Google Play
  3. App prompts victim for PIN and NFC card tap
  4. NFC relay traffic forwarded to operator device
  5. PIN exfiltrated to attacker C2 over HTTP
  6. Cash-out via contactless ATM/payment abuse
- **Persistence technique:** Not observed today beyond normal app presence; campaign relies on fraud workflow rather than deep Android persistence
- **C2 communication method:** HTTP PIN exfiltration to campaign server; NFC relay channel through abused HandyPay workflow

### Campaign C: ZionSiphon (OT-focused malware)
- **Malware name / family:** ZionSiphon
- **Threat actor:** Unknown; politically motivated messaging suggests ideologically driven actor
- **Initial access vector:** Not observed today
- **Execution chain (step-by-step):**
  1. Privilege check and RunAs relaunch via PowerShell
  2. Persistence set in `HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Run`
  3. Target checks for Israel IP ranges and water/desalination artifacts
  4. Local config tampering (chlorine/pressure-related values)
  5. Subnet OT scan for Modbus (502), DNP3 (20000), S7comm (102)
  6. USB propagation via hidden `svchost.exe` + malicious `.lnk`
  7. Self-delete path if target checks fail
- **Persistence technique:** Hidden copy + autorun value `SystemHealthCheck`
- **C2 communication method:** Not observed in analyzed sample; behavior centered on local/OT interaction and propagation

## 4. 🔍 Indicators of Compromise (IOCs)
```json
{
  "ips": [
    "45.86.230.112",
    "91.107.247.163",
    "104.21.91.170",
    "108.165.230.223",
    "172.187.98.211"
  ],
  "domains": [
    "protecaocartao.online",
    "raiffeisen-cz.eu",
    "client.nfcpay.workers.dev",
    "app.mobil-csob-cz.eu",
    "nfc.cryptomaker.info"
  ],
  "hashes": [
    "992c951f4af57ca7cd8396f5ed69c2199fd6fd4ae5e93726da3e198e78bec0a5",
    "025fc0976c548fb5a880c83ea3eb21a5f23c5d53c4e51e862bb893c11adf712a",
    "22b38dad7da097ea03aa28d0614164cd25fafeb1383dbc15047e34c8050f6f67",
    "2ed9494e9b7b68415b4eb151c922c82c0191294d0aa443dd2cb5133e6bfe3d5d",
    "3ab9575225e00a83a4ac2b534da5a710bdcf6eb72884944c437b5fbe5c5c9235",
    "07c3bbe60d47240df7152f72beb98ea373d9600946860bad12f7bc617a5d6f5f",
    "48A0DE6A43FC6E49318AD6873EA63FE325200DBC",
    "A4F793539480677241EF312150E9C02E324C0AA2",
    "94AF94CA818697E1D99123F69965B11EAD9F010C",
    "84361aaf11cde2df075e65fc31082358",
    "7cecbdfdf2e7a7ae7cc226ae26cd3797",
    "8595855eaf9fe0398c8bff7fa06151bf",
    "3c7f107731634fcb7e3f07b693acd4ce",
    "ea6a6666616f6b02c7b679782a676eab",
    "633c3636b646bd08af271584c0e41ff9"
  ],
  "urls": [
    "https://www.virustotal.com/gui/file/07c3bbe60d47240df7152f72beb98ea373d9600946860bad12f7bc617a5d6f5f/details",
    "https://research.checkpoint.com/2026/dfir-report-the-gentlemen/",
    "https://www.welivesecurity.com/en/eset-research/new-ngate-variant-hides-in-a-trojanized-nfc-payment-app/"
  ],
  "mutex": [
    "Not observed today"
  ]
}
```

## 5. 🔗 Technical Resources & Blogs
- THN Threat Intelligence hub: https://thehackernews.com/search/label/Threat%20Intelligence
- Check Point Research (SystemBC + The Gentlemen): https://research.checkpoint.com/2026/dfir-report-the-gentlemen/
- ESET WeLiveSecurity (new NGate variant): https://www.welivesecurity.com/en/eset-research/new-ngate-variant-hides-in-a-trojanized-nfc-payment-app/
- ESET IoC repo (NGate): https://github.com/eset/malware-ioc/tree/master/ngate
- Darktrace (ZionSiphon OT malware analysis): https://www.darktrace.com/blog/inside-zionsiphon-darktraces-analysis-of-ot-malware-targeting-israeli-water-systems
- THN KEV update coverage: https://thehackernews.com/2026/04/cisa-adds-8-exploited-flaws-to-kev-sets.html

## 6. 🧪 Sandbox / Sample Links
- VirusTotal sample (ZionSiphon): https://www.virustotal.com/gui/file/07c3bbe60d47240df7152f72beb98ea373d9600946860bad12f7bc617a5d6f5f/details
- Any.Run: **Not observed today**
- Hybrid Analysis: **Not observed today**
- MalwareBazaar: **Not observed today**

## 7. 🛡️ Detection & Mitigation
- **YARA / Sigma rules:** Not observed today in publicly linked Apr 20–21 reports.
- **Behavioral detection priorities:**
  - Detect domain-wide execution patterns: remote writes to `ADMIN$`, sudden `gpupdate /force` bursts, and GPO-triggered binary launch events.
  - Alert on `rundll32.exe` initiating outbound C2 to rare internet IPs on 80/443 immediately after remote service/task creation.
  - Flag unsanctioned remote-admin tooling install (`AnyDesk`) plus firewall/RDP policy flips in close time proximity.
  - For mobile/BFSI telemetry, monitor Android sideload events + app-default-NFC changes + suspicious HTTP egress from non-Play-distributed finance-related apps.
  - In OT networks, baseline and alert on rapid /24 scans hitting 502/20000/102 and anomalous Modbus write attempts to chlorine/pressure-related registers.
- **Patch / workaround guidance:**
  - Prioritize emergency remediation for the newly KEV-listed CVEs from Apr 21.
  - Restrict administrative shares and GPO-edit privileges; require MFA and PAWs for domain-admin workflows.
  - Block direct outbound C2 from servers/workstations via strict egress controls and proxy allowlists.
  - Enforce Android enterprise controls to block sideloading and require verified app sources.
  - Segment OT from IT, allowlist ICS protocols by zone, and enforce removable-media controls.

## 8. 📊 Trends & Insights
- **RaaS operational maturity is increasing:** The Gentlemen affiliate model shows rapid scaling, layered fallback C2, and domain-wide blast techniques.
- **Fraud malware is converging with legitimate app abuse:** NGate shifted from openly malicious tooling to trojanized legitimate NFC infrastructure, reducing user suspicion.
- **OT malware development is becoming iterative:** ZionSiphon shows partially implemented multi-protocol ICS logic, suggesting staged testing before fully reliable sabotage.
- **Exploit pressure remains broad, not single-vendor:** KEV additions span print management, CI/CD, messaging, patch management, and SD-WAN platforms in one cycle.

