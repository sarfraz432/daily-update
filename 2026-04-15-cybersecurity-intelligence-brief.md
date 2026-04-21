# Daily Cybersecurity Intelligence Brief — 2026-04-15

## 1. 🧠 Executive Summary
- Active exploitation pressure remained high on **April 14–15, 2026**, led by exploitation of **ShowDoc CVE-2025-0520** (unauthenticated file upload to RCE) and newly cataloged **CISA KEV** flaws across Fortinet, Adobe, and Microsoft.
- Mobile threat activity escalated with **Mirax Android RAT** campaigns (200K+ social-ad reach), combining banking malware behavior with **SOCKS5 residential-proxy monetization**.
- Financial-sector targeting in LATAM continued with **JanelaRAT** (THN Threat Intelligence), showing mature operator tradecraft: title-bar targeting, staged C2, and anti-fraud-aware interaction hijacking.
- Social engineering remained an APT initial-access accelerator: **APT37/ScarCruft** used Facebook-to-Telegram trust chains to deliver **RokRAT** through a trojanized PDF toolchain.
- Defensive priority: patch KEV-tracked assets immediately, block high-risk installer paths, and detect C2 over atypical channels (WebSockets/cloud storage/overlay-driven fraud workflows).

## 2. 🚨 Active Threats & Vulnerabilities

### Threat: CVE-2025-0520 (ShowDoc)
- Severity: **Critical**
- Affected systems: ShowDoc versions prior to 2.8.7 (legacy deployments still exposed)
- Exploitation status: **In-the-wild**
- Technical root cause: **Unauthenticated unrestricted file upload** enabling attacker web-shell upload and server-side RCE.

### Threat: CVE-2026-21643 (Fortinet FortiClient EMS)
- Severity: **Critical**
- Affected systems: FortiClient EMS
- Exploitation status: **In-the-wild** (CISA KEV inclusion)
- Technical root cause: **SQL injection / improper input handling** in management interfaces leading to unauthorized command/code execution.

### Threat: CVE-2026-34621 (Adobe Acrobat/Reader)
- Severity: **Critical/High** (vendor-scored critical impact in exploitation context)
- Affected systems: Adobe Acrobat/Reader (Windows/macOS vulnerable builds prior to patched versions)
- Exploitation status: **In-the-wild**
- Technical root cause: **Prototype pollution** enabling JavaScript object/property manipulation and subsequent code execution.

### Threat: Microsoft KEV-added vulnerabilities (batch)
- Name/CVEs: CVE-2023-21529, CVE-2023-36424, CVE-2025-60710, CVE-2012-1854
- Severity: **High**
- Affected systems: Microsoft Exchange, Windows CLFS path, Windows link handling, legacy Office/VBE components
- Exploitation status: **In-the-wild** (KEV evidence threshold met)
- Technical root cause: **Deserialization of untrusted data, out-of-bounds read, link following/path abuse, insecure library loading**.

## 3. 🦠 Malware & Campaign Analysis

### Campaign: Mirax Android RAT
- Malware name/family: **Mirax RAT (Android, private MaaS model)**
- Threat actor: **Financially motivated cybercrime operators (exact cluster unconfirmed)**
- Initial access vector: Meta ad-driven social engineering to sideload fake IPTV/streaming apps
- Execution chain (step-by-step):
  1. Victim clicks malicious social ad and lands on mobile-gated dropper page.
  2. Dropper APK is fetched (often via GitHub Releases) and installed outside trusted app stores.
  3. Embedded encrypted payload (.dex/.apk) is decrypted/unpacked (RC4/XOR staging).
  4. Victim is coerced into granting Accessibility/abuse-critical permissions.
  5. Implant initiates WebSocket C2 channels for control, data theft, and optional proxy tunneling.
- Persistence technique: Accessibility abuse, anti-uninstall controls, and overlay-driven user deception
- C2 communication method: **WebSockets** (`/control`, `/data`, optional `/tunnel`) with SOCKS5 relay/proxy support.

### Campaign: JanelaRAT (THN Threat Intelligence)
- Malware name/family: **JanelaRAT** (BX RAT derivative)
- Threat actor: **Financially motivated LATAM banking-focused operators**
- Initial access vector: Phishing (invoice lure) -> ZIP/MSI-based staged loader chain
- Execution chain (step-by-step):
  1. Victim receives invoice-themed lure and downloads staged archive/installer.
  2. Loader executes multi-stage scripts and uses **DLL side-loading** to run payload.
  3. Malware establishes startup persistence and performs victim profiling.
  4. Title-bar matching against hard-coded financial targets triggers interactive fraud modules.
  5. Operator receives telemetry and executes remote interaction hijacking/overlay actions.
- Persistence technique: Startup-folder **LNK** persistence + staged loaders
- C2 communication method: TCP socket channels, including conditional secondary channel activation.

### Campaign: APT37 (ScarCruft) delivering RokRAT
- Malware name/family: **RokRAT**
- Threat actor: **APT37 / ScarCruft (North Korea-linked)**
- Initial access vector: Facebook trust-building -> Messenger -> Telegram-delivered archive with tampered PDF viewer
- Execution chain (step-by-step):
  1. Threat personas build rapport on Facebook.
  2. Victim receives Telegram archive containing decoys and trojanized installer.
  3. Installer executes embedded shellcode and retrieves staged payload masquerading as image data.
  4. Final RokRAT payload deploys and begins collection/exfiltration.
- Persistence technique: Trojanized legitimate software execution chain and staged payloading
- C2 communication method: Compromised website + cloud abuse (including Zoho WorkDrive-backed operations).

## 4. 🔍 Indicators of Compromise (IOCs)
```json
{
  "ips": [],
  "domains": [
    "japanroom[.]com"
  ],
  "hashes": [],
  "urls": [
    "hxxp://japanroom[.]com/1288247428101.jpg"
  ],
  "mutex": []
}
```
Note: High-confidence public IP/hash indicators were **not consistently disclosed** in April 14–15 source reporting.

## 5. 🔗 Technical Resources & Blogs
- The Hacker News Threat Intelligence hub: https://thehackernews.com/search/label/Threat%20Intelligence
- ShowDoc active exploitation (CVE-2025-0520): https://thehackernews.com/2026/04/showdoc-rce-flaw-cve-2025-0520.html
- CISA KEV additions coverage (Fortinet/Microsoft/Adobe): https://thehackernews.com/2026/04/cisa-adds-6-known-exploited-flaws-in.html
- JanelaRAT campaign (THN, citing Kaspersky): https://thehackernews.com/2026/04/janelarat-malware-targets-latin.html
- Mirax technical analysis (Cleafy): https://www.cleafy.com/cleafy-labs/mirax-a-new-android-rat-turning-infected-devices-into-potential-residential-proxy-nodes
- APT37 RokRAT campaign: https://thehackernews.com/2026/04/north-koreas-apt37-uses-facebook-social.html

## 6. 🧪 Sandbox / Sample Links
- VirusTotal: **Not observed today** (no verified, newly disclosed sample hash links in source set)
- Any.Run / Hybrid Analysis: **Not observed today**
- MalwareBazaar: **Not observed today**

## 7. 🛡️ Detection & Mitigation
- Prioritize emergency patching for KEV-listed CVEs (especially internet-exposed FortiClient EMS, Exchange, and document-handling endpoints).
- Detection ideas (behavioral):
  - Alert on **unauthenticated file upload + PHP/web-shell creation** patterns on ShowDoc hosts.
  - Hunt for unusual **msiexec/rundll32/side-loading chains** and Startup-folder LNK creation tied to invoice-themed lures.
  - Monitor Android/mobile telemetry for Accessibility abuse + WebSocket C2 + rapid APK hash churn from identical app labels.
  - Flag browser/mobile sessions with overlay injection behavior and post-login anomalous transaction sequences.
  - Detect social-engineering kill chain pivots: Facebook/Messenger -> Telegram -> archive installer delivery.
- YARA / Sigma: Public rules were not consistently bundled in the cited April 14–15 releases; generate local detections from observed chain artifacts and ATT&CK behaviors.
- Workarounds:
  - Restrict sideloading and enforce managed app stores/mobile threat defense.
  - Enforce application allowlisting and signature validation for document viewers/installers.
  - Block known malicious domains/paths and tighten egress controls for abnormal WebSocket destinations.

## 8. 📊 Trends & Insights
- Campaigns are converging on a **fraud + infrastructure dual-use model** (e.g., Mirax stealing credentials while monetizing infected devices as proxy nodes).
- **Interactive fraud malware** is maturing: title-bar/overlay-driven execution windows indicate strong operator feedback loops and victim-aware timing.
- **Trust-abuse initial access** remains dominant: social platforms, legit cloud hosting, and tampered legitimate software reduce static detections.
- Exploitation velocity remains high for both legacy and current enterprise software, reflected by continued KEV expansion and rapid operationalization.
