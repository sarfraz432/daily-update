# Daily Cybersecurity Intelligence Brief - 2026-05-27

Reporting window: last 24 hours ending 2026-05-27 11:02 IST / 2026-05-27 05:32 UTC. Sources reviewed include The Hacker News Threat Intelligence section, Google/Mandiant, Check Point Research, Palo Alto Unit 42, Broadcom/Symantec reporting surfaced by THN, BleepingComputer, and accessible X.com-indexed security search results. Focus: active exploitation, zero-days, APT campaigns, malware with configuration/C2 detail, ransomware-enabling access, and major breaches.

## 1. 🧠 Executive Summary

- Active exploitation of KnowledgeDeliver CVE-2026-5426 was disclosed by Google/Mandiant and THN: shared ASP.NET machine keys enabled unauthenticated ViewState deserialization RCE, BLUEBEAM/Godzilla web shell deployment, JavaScript tampering, and Cobalt Strike delivery.
- The mandatory THN Threat Intelligence review surfaced two Iran-linked APT campaigns: MuddyWater/Seedworm DLL side-loading with ChromElevator credential theft, and Nimbus Manticore/Screening Serpens deploying the newly documented MiniFast/MiniUpdate RAT.
- MiniFast is today's most complete malware-analysis item: Check Point and Unit 42 published infection chains, AppDomain hijacking details, C2 endpoints, opcodes, persistence, domains, and SHA256 hashes.
- Targeting is concentrated on aviation, software, telecom, defense, manufacturing, education, public sector, finance, and Japanese LMS deployments.
- Ransomware-specific new activity and unique X.com-only IOCs were not observed today from high-confidence accessible sources.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-5426 - Digital Knowledge KnowledgeDeliver ViewState RCE

- Severity: High, CVSS 7.5.
- Affected systems: Digital Knowledge KnowledgeDeliver LMS deployments created before 2026-02-24 using standardized `web.config` with shared ASP.NET `machineKey` values.
- Exploitation status: In-the-wild; exploited as a zero-day before patch/public disclosure.
- Technical root cause: Hard-coded/pre-shared ASP.NET machine keys allow attackers to sign malicious `__VIEWSTATE` payloads and trigger unauthenticated deserialization/RCE.
- Operational impact: BLUEBEAM/Godzilla in-memory web shell, web-root permission tampering, remote JavaScript injection, and targeted Cobalt Strike Beacon workstation infection.

### Nimbus Manticore MiniFast / MiniUpdate Campaign

- Severity: High.
- Affected systems: Windows endpoints of aviation, software, defense, telecom, oil and gas, and Middle East/U.S./European organizations targeted by phishing or SEO-poisoned software installers.
- Exploitation status: Active APT campaign; not vulnerability-driven.
- Technical root cause: Social engineering plus AppDomainManager hijacking and DLL side-loading through trusted .NET/installer execution paths.
- Operational impact: Long-term RAT access, command execution, file exfiltration, DLL loading, scheduled-task persistence, and adaptive C2 beacon timing.

### MuddyWater / Seedworm DLL Side-Loading Campaign

- Severity: High.
- Affected systems: Industrial/electronics manufacturing, education, public sector, financial services, professional services, airport, and telecom-adjacent targets across nine countries.
- Exploitation status: In-the-wild APT activity observed in Q1 2026 and disclosed in the reporting window.
- Technical root cause: Abuse of signed `fmapp.exe` and `sentinelmemoryscanner.exe` to side-load malicious `fmapp.dll` and `sentinelagentcore.dll`.
- Operational impact: Chromium credential/cookie/payment-card theft via ChromElevator, SAM hive theft, screenshots, reconnaissance, privilege escalation, and SOCKS5 reverse-proxy tunneling.

### Trend Micro Apex One CVE-2026-34926

- Severity: Medium, operationally high because it targets endpoint security infrastructure.
- Affected systems: Trend Micro Apex One on-premise servers.
- Exploitation status: In-the-wild; Trend Micro observed exploitation attempts and CISA KEV-listed the flaw with a 2026-06-04 federal deadline.
- Technical root cause: Directory traversal allowing a pre-authenticated local attacker with admin credentials on the Apex One server to modify a key table and inject malicious code for deployment to agents.
- Operational impact: Compromised security server can become a malware distribution point to managed Windows endpoints.

## 3. 🦠 Malware & Campaign Analysis

### MiniFast / MiniUpdate RAT

- Malware name / family: MiniFast, also tracked by Unit 42 as MiniUpdate; related MiniJunk V2.
- Threat actor: Nimbus Manticore / Screening Serpens / UNC1549, IRGC-affiliated.
- Initial access vector: Fake career opportunities, fake meeting/Zoom installer lures, OnlyOffice-hosted ZIP archives, and SEO poisoning for `getsqldeveloper[.]com`.
- Execution chain:
  1. Victim downloads a ZIP or trojanized installer impersonating an airline hiring portal, Zoom installer, or SQL Developer.
  2. `Setup.exe.config` abuses AppDomainManager hijacking to load attacker-controlled DLLs before normal application logic.
  3. Loader displays fake installer/error UI and may launch a legitimate installer to reduce suspicion.
  4. Malware waits for the legitimate Zoom scheduled task `ZoomUpdateTaskUser-<SID>` and hijacks it.
  5. Second-stage files are staged under `C:\Users\<USER>\AppData\Local\Zoom\bin\update`.
  6. `Update.exe` loads `Updater.dll`, which validates `update.exe` with parent `svchost.exe` before invoking `UpdateChecker.dll`.
  7. MiniFast/MiniUpdate beacons host identity, registers a token/session, polls for tasks, executes opcodes, and returns results.
- Persistence technique: Scheduled task hijacking of Zoom updater flow; MiniFast can also create/update a scheduled task named `WindowsSecurityUpdate`.
- C2 communication method: HTTP API-style JSON and Base64 tasking with Chrome-like user agent. Endpoints include `/rg`, `/agent/init`, `/agent/poll?token=`, `/agent/result`, `/upload/`, and `/files/`. Operator-controlled `pollInterval` and `jitterTime` are supported.

### KnowledgeDeliver BLUEBEAM / Cobalt Strike Chain

- Malware name / family: BLUEBEAM / Godzilla web shell; Cobalt Strike Beacon follow-on.
- Threat actor: Unknown.
- Initial access vector: CVE-2026-5426 unauthenticated ViewState deserialization against internet-facing KnowledgeDeliver LMS.
- Execution chain:
  1. Attacker obtains or derives shared ASP.NET `machineKey` values.
  2. Malicious `__VIEWSTATE` payload is sent to the LMS and deserialized server-side.
  3. BLUEBEAM/Godzilla runs in-memory inside `w3wp.exe` and accepts encrypted HTTP POST command bodies.
  4. Attacker uses `icacls` to grant broad write access to the web application directory.
  5. Application JavaScript is modified to show a fake security alert and silently load attacker-controlled script.
  6. Visitors are convinced to install a fake "security authentication plugin."
  7. The fake installer deploys Cobalt Strike Beacon encrypted with a key derived from the victim organization name.
- Persistence technique: BLUEBEAM in-memory web shell plus malicious web-root JavaScript modifications; Cobalt Strike persistence not publicly detailed.
- C2 communication method: Encrypted HTTP POST body traffic to BLUEBEAM/Godzilla; Cobalt Strike C2 indicators are available via Google Threat Intelligence collection, but full public infrastructure was not exposed in accessible text.

### MuddyWater / Seedworm ChromElevator Side-Loading

- Malware name / family: Malicious `fmapp.dll` / `sentinelagentcore.dll` loaders with embedded ChromElevator; Node.js/PowerShell implant chain.
- Threat actor: MuddyWater / Seedworm / Mango Sandstorm / Static Kitten, Iran-linked.
- Initial access vector: Not publicly confirmed for the May 26 disclosure.
- Execution chain:
  1. Node.js-based implant infrastructure launches PowerShell for discovery, screenshots, SAM hive theft, privilege escalation, and SOCKS5 tunneling.
  2. Signed Fortemedia `fmapp.exe` side-loads malicious `fmapp.dll`.
  3. Signed SentinelOne `sentinelmemoryscanner.exe` side-loads malicious `sentinelagentcore.dll`.
  4. Both DLL paths run ChromElevator to bypass Chromium App-Bound Encryption and steal browser credentials, cookies, and payment-card data.
  5. Stolen data is staged through `sendit[.]sh` in at least one observed intrusion.
- Persistence technique: Re-execution of side-loaded binaries and Run-key style persistence were reported in related Broadcom/Symantec context; exact value names for the current disclosure were not public.
- C2 communication method: SOCKS5 reverse proxy tunneling; one previously reported `fmapp.dll` connected to `157.20.182[.]49`.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "157.20.182[.]49"
  ],
  "domains": [
    "business-startup[.]org",
    "business-startup.azurewebsites[.]net",
    "businessstartup.azurewebsites[.]net",
    "buisness-centeral.azurewebsites[.]net",
    "buisness-centeral-transportation.azurewebsites[.]net",
    "buisness-centeral-transportation[.]com",
    "licencemanagers.azurewebsites[.]net",
    "licencesupporting.azurewebsites[.]net",
    "peerdistsvcmanagers.azurewebsites[.]net",
    "themesmanagers.azurewebsites[.]net",
    "themesprovidermanagers.azurewebsites[.]net",
    "nanomatrix.azurewebsites[.]net",
    "quantumweave.azurewebsites[.]net",
    "elementshift.azurewebsites[.]net",
    "premierhealthadvisory[.]com",
    "premierhealthadvisory.azurewebsites[.]net",
    "premier-healthadvisory.azurewebsites[.]net",
    "ramiltonsfinance[.]com",
    "ramiltonsfinance.azurewebsites[.]net",
    "ramiltons-finance.azurewebsites[.]net",
    "globalitconsultants.azurewebsites[.]net",
    "globalit-consultants.azurewebsites[.]net",
    "global-it-consultants.azurewebsites[.]net",
    "global-it-checkers.azurewebsites[.]net",
    "global-it-checkbusiness.azurewebsites[.]net",
    "global-check-itbusiness.azurewebsites[.]net",
    "global-check-business-it.azurewebsites[.]net",
    "globalbusiness-checkers-it.azurewebsites[.]net",
    "getsqldeveloper[.]com",
    "docspace-y4cumb.onlyoffice[.]com",
    "docspace-twpf0e.onlyoffice[.]com",
    "sendit[.]sh"
  ],
  "hashes": [
    "10fd541674adadfbba99b54280f7e59732746faf2b10ce68521866f737f1e46d",
    "eee657ffdb2af8ed6412221e7d5fbf4f5742f2ac2c88f43f12db46af0697de71",
    "781605ce9d4a9869e846f6c9657d71437cb6240ab27ffbc4cd550c0e06996690",
    "2c214494fd0bad31473ca8adce78a4f50847876584571e66aadeae70827ec2dc",
    "f08b17856616d66492a24dced27f788e235f35f42fa7cd10f315000d3a2f4c03",
    "a57ffb819fe8d98ff925c5d7b239598fe302acf5a13193d7a535040a71298fdf",
    "44f4f7aca7f1d9bfdaf7b3736934cbe19f851a707662f8f0b0c49b383e054250",
    "332ba2f0297dfb1599adecc3e9067893e7cf243aa23aedce4906a4c480574c17",
    "0db36a04d304ad96f9e6f97b531934594cd95a5cea9ff2c9af249201089dc864",
    "38bd137c672bd58d08c4f0502f993a6561e2c3411773d1ae57ee0151a0a9d11d",
    "d4a7e9f107fe40c1a5d0139c6c6e25bf6bf57f61feff090bee28f476bb3cc3c2",
    "bc3b44154518c5794ce639108e7b9c5fecb0c189607a26de1aaed518d890c7ad",
    "74882085db2088356ed7f72f01e0404a0a98cda88ef56fb15ce74c1f36b26d27",
    "9cf029daca89523d917dafed0568d11d00e45ec96b5b90b4a1f7fd4018c7da84",
    "b19e06da580cf91691eda066ac9ee4b09c6e5dc26c367af12660fe1f9306eec4",
    "8808c794c24367438f183e4be941876f1d3ecd0c8d2eb43b10d2380841d2283b",
    "43dc62cef52ebdd69e79f10015b3e13890f26c058325c0ff139c70f8d8eadcfa",
    "9e4a658e6d831c9e9bdfe11884a75b7c64812ed0a80e8495ddf6b316505acac1",
    "7c1f99dca8e5a7897892f9d224a6495023a2cfd2671697d229d355978c415ed2"
  ],
  "urls": [
    "hxxps://docspace-y4cumb.onlyoffice[.]com/storage/files/root/folder_3602000/file_3601577/v1/content.zip",
    "hxxps://docspace-twpf0e.onlyoffice[.]com/storage/files/root/folder_3765000/file_3764519/v1/content.zip",
    "hxxps://2117.filemail[.]com/api/file/get?filekey=T0EnWQ6NugHkW_kLfDxPBEw_um6NSkg9ZwNRQ_5lrKrLLUo35pV8m3TKv1LqF3zZzdUm"
  ],
  "mutex": []
}
```

Notes:
- Hashes are SHA256 values from Check Point, Unit 42, and Google/Mandiant reporting.
- No new registry-key or mutex indicators were observed today in accessible primary reporting.

## 5. 🔗 Technical Resources & Blogs

- The Hacker News Threat Intelligence: MuddyWater DLL side-loading - https://thehackernews.com/2026/05/muddywater-uses-dll-side-loading-in.html
- Broadcom/Symantec reporting surfaced by THN: Seedworm Korean electronics/global campaign - https://www.security.com/threat-intelligence/iran-seedworm-electronics
- The Hacker News Threat Intelligence: Nimbus Manticore MiniFast/MiniJunk V2 - https://thehackernews.com/2026/05/iranian-hackers-deploy-minifast-and.html
- Check Point Research: Fast and Furious - Nimbus Manticore Operations During the Iranian Conflict - https://research.checkpoint.com/2026/fast-and-furious-nimbus-manticore-operations-during-the-iranian-conflict/
- Palo Alto Unit 42: Tracking Iranian APT Screening Serpens' 2026 Espionage Campaigns - https://unit42.paloaltonetworks.com/tracking-iran-apt-screening-serpens/
- The Hacker News Threat Intelligence: KnowledgeDeliver LMS flaw exploited - https://thehackernews.com/2026/05/knowledgedeliver-lms-flaw-exploited-to.html
- Google Cloud/Mandiant: Exploitation of KnowledgeDeliver via ViewState Deserialization - https://cloud.google.com/blog/topics/threat-intelligence/knowledgedeliver-viewstate-deserialization-vulnerability/
- BleepingComputer: Trend Micro Apex One CVE-2026-34926 active exploitation - https://www.bleepingcomputer.com/news/security/trend-micro-warns-of-apex-one-zero-day-exploited-in-attacks/
- CISA KEV catalog - https://www.cisa.gov/known-exploited-vulnerabilities-catalog

## 6. 🧪 Sandbox / Sample Links

- VirusTotal / Google Threat Intelligence: KnowledgeDeliver IOC collection requires registered GTI access - https://www.virustotal.com/
- VirusTotal sample references for MiniUpdate were noted by Unit 42 metadata, but direct public VT URLs for every sample were not exposed in accessible text.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.
- GitHub PoC: Not observed today for CVE-2026-5426 or CVE-2026-34926.

## 7. 🛡️ Detection & Mitigation

### KnowledgeDeliver CVE-2026-5426

- Apply Digital Knowledge remediation and ensure every KnowledgeDeliver instance uses unique, cryptographically strong ASP.NET `machineKey` values.
- Hunt Windows Application Event ID 1316 / ASP.NET Event code `4009` for `Viewstate verification failed` messages. `Viewstate was invalid` with successful integrity checks is higher-confidence.
- Monitor `w3wp.exe` spawning `cmd.exe /c`, `whoami`, `powershell.exe`, or `icacls`.
- File-integrity monitor `.js`, `.aspx`, and `.config` under web roots for unauthorized remote script loaders.
- Search for BLUEBEAM/Godzilla sample `7c1f99dca8e5a7897892f9d224a6495023a2cfd2671697d229d355978c415ed2`.
- Restrict LMS exposure to known IP ranges where feasible and perform endpoint triage for users who installed the fake plugin.

### MiniFast / MiniUpdate

- Detect `.config` files that set custom `appDomainManagerAssembly` / `appDomainManagerType` near signed binaries such as `Setup.exe` or `Update.exe`.
- Hunt for .NET configuration directives used for evasion: `<etwEnable enabled="false"/>`, `<bypassTrustedAppStrongNames enabled="true"/>`, and `<publisherPolicy apply="no"/>`.
- Monitor creation/modification of `ZoomUpdateTaskUser-<SID>` and suspicious task `WindowsSecurityUpdate`.
- Hunt staging path `C:\Users\<USER>\AppData\Local\Zoom\bin\update` containing `Update.exe`, `Updater.dll`, or `UpdateChecker.dll` outside normal Zoom update activity.
- Network detections: HTTP POST/GET to `/rg`, `/agent/init`, `/agent/poll?token=`, `/agent/result`, `/upload/`, `/files/`; Chrome UA `Chrome/146.0.0.0` on unusual Azure/App Service hosts.
- Block listed MiniUpdate/MiniFast domains and rotate credentials for users exposed to fake hiring/meeting lures.

### MuddyWater / Seedworm

- Detect `fmapp.exe` loading `fmapp.dll` and `sentinelmemoryscanner.exe` loading `sentinelagentcore.dll` from user-writable or per-user staging directories.
- Hunt for Node.js spawning PowerShell plus `reg save` of SAM/SECURITY/SYSTEM hives, screenshot tooling, and SOCKS5 proxy creation.
- Monitor Chromium App-Bound Encryption bypass tooling/strings associated with ChromElevator and abnormal browser credential database access.
- Block or investigate traffic to `157.20.182[.]49` and data staging to `sendit[.]sh`.
- Review Run keys and scheduled logon persistence for long random value names pointing to side-loaded binaries.

### Trend Micro Apex One CVE-2026-34926

- Patch Apex One on-premise servers per Trend Micro guidance; federal deadline is 2026-06-04.
- Treat Apex One server administrator credential compromise as prerequisite and rotate server admin credentials.
- Audit Apex One server key tables/configuration changes and agent deployment tasks for unauthorized code distribution.
- Hunt managed endpoints for unexpected code pushed from Apex One immediately before or after suspicious server-side changes.

## 8. 📊 Trends & Insights

- Iranian APT operators are accelerating tool refresh cycles: MiniFast/MiniUpdate shows AI-assisted coding indicators, SEO poisoning adoption, and operationally resilient Azure-heavy C2 segmentation.
- Shared deployment secrets remain a systemic exploit class; KnowledgeDeliver repeats the Sitecore/ASP.NET machine-key pattern where one leaked key can compromise an ecosystem.
- Security tooling continues to be abused both as a target and as cover: Apex One can be turned into a distribution path, while SentinelOne/Fortemedia signed binaries are used for side-loading.
- Credential theft is converging with stealthy post-exploitation: MuddyWater's ChromElevator use pairs browser credential theft with SOCKS5 tunneling and SAM hive collection.
- X.com review did not produce independently verifiable tweet-only malware IOCs today; accessible search results were either stale, unrelated, or pointed back to the same public research.
