# Daily Cybersecurity Intelligence Brief - 2026-04-14

## 1. 🧠 Executive Summary
- Active in-the-wild exploitation remained centered on **Adobe Acrobat/Reader CVE-2026-34621** and **marimo CVE-2026-39987**, with exploitation timelines measured in hours after disclosure.
- A high-impact software supply-chain campaign (Axios npm compromise) continues to drive downstream enterprise response activity, including certificate rotation by impacted vendors.
- Major extortion/data-theft pressure persisted against large consumer platforms (e.g., Rockstar-linked Snowflake/Anodot data exposure claims; Booking.com and Basic-Fit breach disclosures on April 13).
- Developer and CI/CD ecosystems remain a top target: attacker tradecraft leveraged package manager trust, postinstall execution, and token/certificate exposure windows.
- Why this matters: current campaigns combine rapid weaponization, stealthy credential theft, and third-party trust transitivity, reducing detection/patch response margins.

## 2. 🚨 Active Threats & Vulnerabilities

### Threat 1: CVE-2026-34621 (Adobe Acrobat/Reader)
- Severity: **Critical** (CVSS 8.6)
- Affected systems: Acrobat DC/Reader DC `<= 26.001.21367`; Acrobat 2024 `<= 24.001.30356` (Windows/macOS)
- Exploitation status: **In-the-wild** (confirmed by Adobe)
- Technical root cause: Prototype pollution (`CWE-1321`) enabling arbitrary code execution; exploit chain abused privileged JS APIs (`util.readFileIntoStream`, `RSS.addFeed`) for fingerprinting/data theft and follow-on payload staging.

### Threat 2: CVE-2026-39987 / GHSA-2679-6mx9-h9xc (marimo)
- Severity: **Critical** (CVSS 9.3)
- Affected systems: `marimo <= 0.20.4` (patched in `0.23.0`)
- Exploitation status: **In-the-wild** (first exploitation observed within ~10 hours of advisory)
- Technical root cause: Missing authentication (`CWE-306`) on `/terminal/ws` WebSocket endpoint; endpoint accepts unauthenticated sessions and spawns PTY shell (`pty.fork`), resulting in pre-auth RCE.

### Threat 3: Axios npm supply-chain compromise (no CVE assigned in source)
- Severity: **High**
- Affected systems: Environments that installed compromised `axios` versions (`1.14.1`, `0.30.4`) containing malicious `plain-crypto-js`
- Exploitation status: **In-the-wild**
- Technical root cause: Maintainer account compromise + malicious dependency injection + `postinstall` execution path (`setup.js`) in npm lifecycle.

## 3. 🦠 Malware & Campaign Analysis

### Campaign A: WAVESHAPER.V2 via Axios supply-chain compromise
- Malware name/family: **SILKBELL dropper** + **WAVESHAPER.V2 RAT**
- Threat actor: **UNC1069** (North Korea nexus), per Google Threat Intelligence Group attribution
- Initial access vector: Compromise of axios maintainer account and poisoning of package versions with `plain-crypto-js`
- Execution chain:
  1. Victim installs compromised axios package
  2. npm runs `postinstall` hook (`node setup.js`)
  3. `setup.js` fingerprints OS and pulls stage payloads
  4. Platform-specific backdoor executes (PowerShell/Mach-O/Python)
  5. Beaconing to C2 over TCP/8000 at 60s intervals
- Persistence technique: Windows `Run` key (`HKCU\Software\Microsoft\Windows\CurrentVersion\Run`) + hidden batch (`%PROGRAMDATA%\system.bat`)
- C2 communication method: HTTP POST + polling to `sfrclak[.]com:8000`; base64-encoded JSON beaconing; fixed user-agent string
- MITRE ATT&CK mapping (inferred from observed behavior): `T1195.001` (Supply Chain Compromise), `T1059.007` (JavaScript), `T1059.001` (PowerShell), `T1547.001` (Registry Run Keys), `T1071.001` (Web Protocols), `T1105` (Ingress Tool Transfer)

### Campaign B: Opportunistic marimo credential-theft exploitation
- Malware name/family: **Not observed today** (hands-on terminal abuse)
- Threat actor: Unknown
- Initial access vector: Direct WebSocket connect to unauthenticated `/terminal/ws`
- Execution chain:
  1. Attacker validates code execution with PoC markers
  2. Reconnects for manual host discovery (`pwd`, `whoami`, `ls`)
  3. Targets `.env` and SSH-related paths
  4. Exfiltrates credentials/secrets
- Persistence technique: **Not observed today** (researcher observed no miner/backdoor persistence attempt)
- C2 communication method: **Not observed today**
- MITRE ATT&CK mapping (inferred): `T1190` (Exploit Public-Facing App), `T1059` (Command and Scripting Interpreter), `T1083` (File and Directory Discovery), `T1552.001` (Credentials in Files)

### Campaign C: Adobe Reader exploit chain (CVE-2026-34621)
- Malware name/family: **Not observed today** (exploit document campaign)
- Threat actor: Unknown (targeting pattern indicates likely espionage-oriented fingerprinting workflow)
- Initial access vector: Malicious PDF opening in vulnerable Reader/Acrobat
- Execution chain:
  1. PDF executes obfuscated JS
  2. Abuses privileged APIs to read local files and collect host/profile data
  3. Sends telemetry to attacker server
  4. Optionally retrieves additional JS for follow-on exploitation
- Persistence technique: **Not observed today**
- C2 communication method: HTTP(S) requests triggered through abused Reader API flow
- MITRE ATT&CK mapping (inferred): `T1204.001` (User Execution: Malicious File), `T1059.007` (JavaScript), `T1005` (Data from Local System), `T1041` (Exfiltration Over C2 Channel)

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "49.207.56.74",
    "142.11.206.73",
    "23.254.167.216",
    "169.40.2.68",
    "188.214.34.20"
  ],
  "domains": [
    "sfrclak.com"
  ],
  "hashes": [
    "e10b1fa84f1d6481625f741b69892780140d4e0e7769e7491e5f4d894c2e0e09",
    "fcb81618bb15edfdedfb638b4c08a2af9cac9ecfa551af135a8402bf980375cf",
    "92ff08773995ebc8d55ec4b8e1a225d0d1e51efa4ef88b8849d0071230c9645a",
    "617b67a8e1210e4fc87c92d1d1da45a2f311c08d26e89b12307cf583c900d101",
    "ed8560c1ac7ceb6983ba995124d5917dc1a00288912387a6389296637d5f815c",
    "f7d335205b8d7b20208fb3ef93ee6dc817905dc3ae0c10a0b164f4e7d07121cd",
    "58401c195fe0a6204b42f5f90995ece5fab74ce7c69c67a24c61a057325af668",
    "65dca34b04416f9a113f09718cbe51e11fd58e7287b7863e37f393ed4d25dde7"
  ],
  "urls": [
    "http://sfrclak.com:8000",
    "http://sfrclak.com:8000/6202033",
    "https://www.virustotal.com/gui/file/65dca34b04416f9a113f09718cbe51e11fd58e7287b7863e37f393ed4d25dde7"
  ]
}
```

## 5. 🔗 Technical Resources & Blogs
- Adobe bulletin (APSB26-43): https://helpx.adobe.com/security/products/acrobat/apsb26-43.html
- Marimo advisory (GHSA-2679-6mx9-h9xc): https://github.com/marimo-team/marimo/security/advisories/GHSA-2679-6mx9-h9xc
- Sysdig marimo exploitation analysis: https://www.sysdig.com/blog/marimo-oss-python-notebook-rce-from-disclosure-to-exploitation-in-under-10-hours
- Google Threat Intelligence (Axios campaign): https://cloud.google.com/blog/topics/threat-intelligence/north-korea-threat-actor-targets-axios-npm-package
- OpenAI response advisory (Axios exposure handling): https://openai.com/index/axios-developer-tool-compromise/
- BleepingComputer Adobe exploitation coverage: https://www.bleepingcomputer.com/news/security/adobe-rolls-out-emergency-fix-for-acrobat-reader-zero-day-flaw/
- BleepingComputer marimo exploitation coverage: https://www.bleepingcomputer.com/news/security/critical-marimo-pre-auth-rce-flaw-now-under-active-exploitation/
- BleepingComputer Rockstar/Anodot breach linkage: https://www.bleepingcomputer.com/news/security/stolen-rockstar-games-analytics-data-leaked-by-extortion-gang/
- BleepingComputer Booking.com breach disclosure: https://www.bleepingcomputer.com/news/security/new-bookingcom-data-breach-forces-reservation-pin-resets/
- BleepingComputer Basic-Fit breach disclosure: https://www.bleepingcomputer.com/news/security/european-gym-giant-basic-fit-data-breach-affects-1-million-members/

## 6. 🧪 Sandbox / Sample Links
- VirusTotal sample (Adobe exploit file): https://www.virustotal.com/gui/file/65dca34b04416f9a113f09718cbe51e11fd58e7287b7863e37f393ed4d25dde7
- GTI IOC collection (Axios campaign): https://www.virustotal.com
- Any.Run links: **Not observed today**
- Hybrid Analysis links: **Not observed today**
- MalwareBazaar sample links: **Not observed today**

## 7. 🛡️ Detection & Mitigation
- YARA/Sigma rules:
  - eSentire-referenced detections for STX RAT were cited in related reporting, but campaign-specific rule content for the April 12-14 focus set was **not publicly enumerated in source detail today**.
  - For this reporting window, direct YARA/Sigma publication for Adobe/marimo exploitation chains was **Not observed today**.
- Detection ideas (behavioral):
  - Alert on outbound connections to `sfrclak[.]com` / `142.11.206.73` / `23.254.167.216`, especially from Node/npm build hosts.
  - Hunt for npm lockfiles or package trees containing `axios@1.14.1`, `axios@0.30.4`, `plain-crypto-js@4.2.0/4.2.1`.
  - Detect anomalous `postinstall`-initiated child processes (`node -> curl/powershell/bash/zsh/python`).
  - For marimo, monitor and block unsolicited WebSocket sessions to `/terminal/ws`; flag shell command execution from notebook service contexts.
  - For Adobe environments, inspect endpoint/network telemetry for Reader-initiated suspicious file reads and unusual feed/API invocation patterns tied to untrusted PDFs.
- Patch / workaround guidance:
  - Adobe: patch to APSB26-43 fixed builds immediately.
  - marimo: upgrade to `0.23.0+`; if delayed, hard-block `/terminal/ws`, remove internet exposure, and rotate secrets.
  - Axios compromise response: pin safe versions (`<=1.14.0` or `<=0.30.3`), clear package caches, and rotate credentials on any host where malicious dependency executed.

## 8. 📊 Trends & Insights
- Exploitation velocity continues to compress: marimo was operationally exploited in under 10 hours after disclosure, reinforcing “advisory-to-weaponization” as a same-day risk.
- Supply-chain attacks are increasingly multi-platform by default, with one package-level compromise cascading into macOS/Windows/Linux payload delivery.
- Credential theft is favored over noisy disruption in early-stage intrusions (e.g., marimo sessions prioritized `.env`/SSH data with minimal dwell and no immediate persistence).
- Third-party integration trust remains a structural weakness in 2026 breach patterns, with token exposure and delegated access repeatedly enabling downstream tenant impacts.
