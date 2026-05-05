# Daily Cybersecurity Intelligence Brief - 2026-05-05

## 1. 🧠 Executive Summary

- **VENOMOUS#HELPER / STAC6405** is abusing signed SimpleHelp and ScreenConnect RMM tooling in an ongoing phishing campaign affecting 80+ organizations, mainly in the U.S.; tradecraft is consistent with IAB or ransomware precursor activity.
- **Progress MOVEit Automation CVE-2026-4670** introduces unauthenticated authentication bypass risk on managed file-transfer automation servers; no exploitation was reported today, but MOVEit remains a proven extortion target class.
- **Silver Fox ABCDoor** remains the most relevant malware-family/configuration item from The Hacker News Threat Intelligence feed: tax-themed phishing delivers RustSL, ValleyRAT/Winos 4.0, then a Python HTTPS backdoor.
- Current attacker pattern: replace custom first-stage malware with legitimate admin tooling, compromised hosting, signed binaries, and language runtimes to reduce static detection value.
- X.com review: direct X search was attempted for the last 24 hours, but no independently retrievable recent posts were accessible in this environment.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-4670 - Progress MOVEit Automation Authentication Bypass

- Severity: Critical, CVSS 9.8.
- Affected systems: MOVEit Automation 2025.0.0 through 2025.0.8, 2024.0.0 through 2024.1.7, and versions prior to 2024.0.0.
- Exploitation status: Not observed today; Progress and Help Net Security reporting state there is no public mention of in-the-wild exploitation.
- Technical root cause: Improper authentication / authentication bypass through backend command-port service interfaces.
- Impact: Unauthorized access, administrative control, data exposure, and potential movement into adjacent managed file-transfer workflows.
- Fixed versions: 2025.1.5, 2025.0.9, and 2024.1.8.

### CVE-2026-5174 - Progress MOVEit Automation Privilege Escalation

- Severity: High, CVSS 7.7.
- Affected systems: Same MOVEit Automation branches as CVE-2026-4670.
- Exploitation status: Not observed today.
- Technical root cause: Improper input validation that can enable privilege escalation through service backend command-port interfaces.
- Impact: Post-auth or chained administrative escalation; most concerning when paired with CVE-2026-4670.

### VENOMOUS#HELPER Dual-RMM Campaign

- Severity: High.
- Affected systems: Windows endpoints where users execute SSA-themed lure executables; U.S., Western Europe, and Latin America are listed target regions.
- Exploitation status: In-the-wild phishing campaign active since at least April 2025; Securonix reports 80+ impacted organizations.
- Technical root cause: Social engineering plus legitimate signed RMM software abuse, not a software vulnerability.
- Impact: Persistent silent remote access, SYSTEM-context execution, file transfer, interactive desktop control, security-product discovery, and likely ransomware staging.

## 3. 🦠 Malware & Campaign Analysis

### VENOMOUS#HELPER / STAC6405 RMM Intrusion Chain

- Malware name / family: Customized SimpleHelp 5.0.1 RAT deployment plus ConnectWise ScreenConnect relay.
- Threat actor: Unattributed; assessed as financially motivated IAB / ransomware precursor. Overlaps with Sophos STAC6405 and Red Canary tracking.
- Initial access vector: SSA-themed phishing email directs victims to compromised `gruta[.]com.mx`, then to payload delivery from `server.cubatiendaalimentos[.]com.mx`.
- Execution chain:
  1. Victim receives SSA impersonation email and clicks a compromised Mexican business site link.
  2. Landing page harvests email and serves `statement5648.exe` from a compromised cPanel-hosted path.
  3. User approves UAC prompt showing verified publisher `SimpleHelp Ltd`.
  4. JWrapper bootstrap decrypts an embedded config overlay, extracts an EOL Oracle Java runtime, and installs silently.
  5. `SimpleService.exe` registers `Remote Access Service`, adds Safe Mode persistence, and starts a watchdog.
  6. `SimpleGatewayService.exe` launches `customer.jar` and beacons to SimpleHelp C2.
  7. Operator installs ScreenConnect as a redundant interactive-access channel.
- Persistence technique: Windows service `Remote Access Service`, `HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot\Network\Remote Access Service`, JWrapper liveness file `sgalive`, and ScreenConnect enrollment.
- C2 communication method: `udp://84.200.205[.]233:5555` for SimpleHelp and `relay://sslzeromail[.]run.place:8041/` for ScreenConnect.

### Silver Fox ABCDoor / ValleyRAT Campaign

- Malware name / family: ABCDoor Python backdoor, ValleyRAT / Winos 4.0, modified RustSL loader.
- Threat actor: Silver Fox, China-based cybercrime/APT-style cluster.
- Initial access vector: Tax-themed phishing emails impersonating Indian and Russian tax authorities; PDF links or archive attachments deliver disguised executables.
- Execution chain:
  1. Victim opens tax-themed PDF/archive and executes a disguised loader.
  2. Modified RustSL performs country geofencing plus VM/sandbox checks.
  3. Loader unpacks encrypted payloads and downloads ValleyRAT/Winos 4.0 components.
  4. ValleyRAT retrieves custom modules, including ABCDoor.
  5. ABCDoor runs from `%LOCALAPPDATA%\appclient` using bundled Python and supports screenshot capture, remote keyboard/mouse control, process and file operations, clipboard theft, updates, removal, and exfiltration.
- Persistence technique: `HKCU:\Software\Microsoft\Windows\CurrentVersion\Run:AppClient`, scheduled task `AppClient` every minute, and `HKCU\Environment\UserInitMprLogonScript` in JavaScript-loader variants.
- C2 communication method: ABCDoor communicates over HTTPS using Socket.IO-style async handlers; ValleyRAT uses configured IP/port C2.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "84.200.205[.]233",
    "213.136.71[.]246",
    "154.82.81[.]205",
    "45.118.133[.]203:5000"
  ],
  "domains": [
    "gruta[.]com.mx",
    "server.cubatiendaalimentos[.]com.mx",
    "sslzeromail[.]run.place",
    "abc.haijing88[.]com",
    "abc.fetish-friends[.]com",
    "mcagov[.]cc"
  ],
  "hashes": [
    "810a99a7d6696a36491530e286476b4cf8a819a47fb5e3801fdfecfdb2dc6193",
    "641230a9f3091bdd38d04c6df96062bfc82dfc4ff6f663ceb522d3881d6af53a",
    "dbdddea03c3fc4c2574ce4221450ec86221ebc615c4915c4c4eb3f2a5e3f5b25",
    "9369d7194ab03362e9e7af022a48bc6d4e7d91a6ab7c4b5cf5d90abbcd8c7012",
    "97f801e750cfc2d4558020fb246782e034fd6101d75a59d8915b4f2b2b50ebd9",
    "11914d10b51b5a96606ae606b5ab70d79550e36c1cce94a86134107c59075e0c",
    "76d85124db2778baecee24cc5ad56c9a3060c41c5b3c1b5cdc7f0435e0f77cac",
    "d953dfbe8d91dc9fafad0a6117e1276fa636d4ae1b6a4d81616ff2446cf09234",
    "3e4b3559fdbe584e19a1ff9b3142b429c6fb91aaa63b5c922c8c5b32c38e426a",
    "5b998a5bc5ad1c550564294034d4a62c",
    "c50c980d3f4b7ed970f083b0d37a6a6a",
    "de8f0008b15f2404f721f76fac34456a",
    "9bf9f635019494c4b70fb0a7c0fb53e4",
    "a543b96b0938de798dd4f683dd92a94a",
    "fa08b243f12e31940b8b4b82d3498804",
    "13669b8f2bd0af53a3fe9ac0490499e5"
  ],
  "urls": [
    "server.cubatiendaalimentos[.]com.mx/~tiendazoycom/sns/",
    "server.cubatiendaalimentos[.]com.mx/~tiendazoycom/sns/statement5648.exe",
    "http[:]//84.200.205[.]233:5555/access/",
    "udp://84.200.205[.]233:5555",
    "relay://sslzeromail[.]run.place:8041/",
    "hxxp://154.82.81[.]205/YD20251001143052.zip",
    "hxxps://mcagov[.]cc/download.php?type=exe",
    "hxxps://abc.fetish-friends[.]com/uploads/appclient.zip",
    "hxxps://abc.fetish-friends[.]com/setup/install"
  ],
  "mutex": []
}
```

Host artifacts:

- `C:\ProgramData\JWrapper-Remote Access\`
- `C:\ProgramData\JWrapper-Remote Access\JWAppsSharedConfig\sgalive`
- `C:\Windows\System32\wbem\wmic.exe.bak`
- `HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot\Network\Remote Access Service`
- `HKCU:\Software\Microsoft\Windows\CurrentVersion\Run:AppClient`
- `HKCU:\Software\CarEmu:FirstInstallTime`
- `%LOCALAPPDATA%\appclient\`
- `%LOCALAPPDATA%\applogs\device.log`

## 5. 🔗 Technical Resources & Blogs

- The Hacker News: [Phishing Campaign Hits 80+ Orgs Using SimpleHelp and ScreenConnect RMM Tools](https://thehackernews.com/2026/05/phishing-campaign-hits-80-orgs-using.html)
- Securonix: [VENOMOUS#HELPER Dual-RMM Phishing Campaign](https://www.securonix.com/blog/venomous-helper-phishing-campaign)
- The Hacker News: [Progress Patches Critical MOVEit Automation Bug Enabling Authentication Bypass](https://thehackernews.com/2026/05/progress-patches-critical-moveit.html)
- Help Net Security: [Critical MOVEit Automation auth bypass vulnerability fixed](https://www.helpnetsecurity.com/2026/05/04/critical-moveit-automation-auth-bypass-vulnerability-fixed-cve-2026-4670/)
- Progress advisory: [MOVEit Automation Critical Security Alert Bulletin](https://community.progress.com/s/article/MOVEit-Automation-Critical-Security-Alert-Bulletin-April-2026-CVE-2026-4670-CVE-2026-5174)
- The Hacker News: [Silver Fox Deploys ABCDoor Malware via Tax-Themed Phishing in India and Russia](https://thehackernews.com/2026/05/silver-fox-deploys-abcdoor-malware-via.html)
- Kaspersky Securelist: [Analyzing the Silver Fox tax campaign and the new ABCDoor backdoor](https://securelist.com/silver-fox-tax-notification-campaign/119575/)
- GitHub repos / exploit code: Not observed today for the prioritized vulnerabilities; RustSL is referenced in reporting but no verified direct repository URL was retrieved.

## 6. 🧪 Sandbox / Sample Links

- Kaspersky OpenTIP links are embedded in the Securelist IOC table for ABCDoor, RustSL loaders, scripts, SFX archives, and ZIP archives.
- VirusTotal: Not observed today.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.
- Public PoC for MOVEit Automation CVE-2026-4670/CVE-2026-5174: Not observed today.

## 7. 🛡️ Detection & Mitigation

### VENOMOUS#HELPER

- YARA/Sigma: Public standalone rules not observed today.
- Detection ideas:
  - Alert on `statement*.exe` or document-lure executables spawning JWrapper, Java 1.7 paths, or `C:\ProgramData\JWrapper-Remote Access\`.
  - Hunt `Remote Access.exe` spawning `wmic.exe`, `wmic.exe.bak`, `netsh.exe`, `cacls.exe`, `cmd.exe`, or PowerShell.
  - Detect repeated process cadence: `netsh wlan show interfaces` around 15s, mouse polling around 23s, and `root\SecurityCenter2` WMI queries around 67s.
  - Treat `C:\Windows\System32\wbem\wmic.exe.bak` as high-confidence compromise evidence.
  - Monitor outbound connections to `84.200.205[.]233:5555` and `sslzeromail[.]run.place:8041`.
- Mitigation:
  - Block listed infrastructure and revoke RMM allowlisting for unsigned/unmanaged SimpleHelp and ScreenConnect tenants.
  - Remove `Remote Access Service`, SafeBoot key, JWrapper directory, `wmic.exe.bak`, and ScreenConnect client; then validate no hands-on-keyboard second stage occurred.
  - Reset credentials used on affected hosts because the campaign enables silent desktop access and bidirectional file transfer.

### MOVEit Automation CVE-2026-4670 / CVE-2026-5174

- YARA/Sigma: Not observed today.
- Detection ideas:
  - Review MOVEit Automation service/backend command-port logs for failed or anomalous authentication transitions, unexpected admin actions, and new workflow/schedule changes.
  - Hunt new administrative users, modified file-transfer tasks, unexpected outbound destinations, and bulk file staging immediately before or after patch windows.
  - Prioritize internet-exposed Automation servers and any deployments adjacent to MOVEit Transfer or sensitive MFT repositories.
- Mitigation:
  - Upgrade to MOVEit Automation 2025.1.5, 2025.0.9, or 2024.1.8.
  - Progress states there are no workarounds that fully resolve the issues; restrict network access to backend command-port interfaces until patching is complete.
  - After patching, rotate credentials and API secrets used by automation workflows if exposure is suspected.

### ABCDoor / Silver Fox

- YARA/Sigma: Public standalone rules not observed today.
- Detection ideas:
  - Alert on `%LOCALAPPDATA%\appclient\python\pythonw.exe -m appclient`, scheduled task `AppClient`, and `HKCU:\Software\Microsoft\Windows\CurrentVersion\Run:AppClient`.
  - Hunt PowerShell/curl downloads from raw IPs into `%LOCALAPPDATA%\appclient\111.zip`.
  - Detect suspicious use of geolocation APIs by non-browser processes and RustSL payload markers `<RSL_START>` / `<RSL_END>`.
  - Inspect tax-themed archives containing executables disguised as PDF/Excel files.
- Mitigation:
  - Block ABCDoor/ValleyRAT infrastructure; quarantine affected hosts and assume clipboard, screenshot, and file-system data exposure.
  - Add detonation for tax-lure PDFs and archives with India/Russia tax terminology.

## 8. 📊 Trends & Insights

- **RMM abuse is replacing commodity RAT loaders:** VENOMOUS#HELPER shows actors favoring signed admin tools because the detection problem shifts from hash reputation to behavior and tenant legitimacy.
- **Managed file transfer remains an extortion pressure point:** MOVEit Automation has no reported exploitation today, but the combination of authentication bypass plus sensitive workflow access deserves emergency patch handling.
- **Runtime-heavy malware continues to fragment visibility:** Silver Fox moves through Rust, Node/Python, ValleyRAT, and ABCDoor layers, forcing defenders to correlate process lineage rather than hunt one binary family.
- **Compromised legitimate hosting is a recurring delivery choice:** Both cPanel-hosted paths and established business domains are being used to bypass reputation filters.
- **X.com-derived signal:** Not observed today due to retrieval failure; no tweet-sourced IOC or sample was included without independent verification.
