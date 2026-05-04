# Daily Cybersecurity Intelligence Brief - 2026-05-04

## 1. 🧠 Executive Summary

- Silver Fox is deploying the newly documented Python backdoor **ABCDoor** through tax-themed phishing against organizations in India, Russia, Indonesia, South Africa, and Japan, with ValleyRAT/Winos 4.0 used as the intermediate backdoor layer.
- **CVE-2026-31431 / Copy Fail** moved into CISA KEV after active exploitation evidence; the Linux kernel LPE is especially relevant to cloud, CI/CD, Kubernetes, and container-hosting environments where any foothold can become host root.
- **CVE-2026-41940** cPanel/WHM authentication bypass is being mass-exploited in **Sorry ransomware** attacks; Shadowserver telemetry cited by BleepingComputer indicates at least 44,000 cPanel IPs compromised.
- Attacker tradecraft today continues to emphasize developer/admin trust paths: signed-looking tax documents, hosting control planes, CI/CD secrets, and container escape from low-privilege contexts.
- X.com review: direct X search was attempted for the last 24 hours, but no independently retrievable recent posts were accessible in this environment.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-41940 - cPanel & WHM Authentication Bypass

- Severity: Critical, CVSS 9.8.
- Affected systems: cPanel & WHM versions after 11.40, DNSOnly, WP Squared; public-facing cPanel/WHM/Webmail services are highest risk.
- Exploitation status: In-the-wild mass exploitation; BleepingComputer reported use in Sorry ransomware attacks and cited Shadowserver telemetry of at least 44,000 compromised cPanel IPs.
- Technical root cause: Authentication bypass in login/session handling, described in public reporting as CRLF injection in session loading/writing paths that enables unauthenticated administrative access.
- Operational impact: Root-equivalent WHM access, hosted-site compromise, database/mailbox theft, ransomware deployment, and persistence via unauthorized root accounts or SSH backdoors.

### CVE-2026-31431 - Linux Kernel "Copy Fail"

- Severity: High, CVSS 7.8.
- Affected systems: Linux kernels shipped since 2017 across major distributions, including cloud and container hosts when the vulnerable AF_ALG / algif_aead path is reachable.
- Exploitation status: Added to CISA KEV on 2026-05-01; Microsoft observed preliminary testing activity and warned of likely increased exploitation. Working Python PoC exists, with Go/Rust ports observed in public repositories.
- Technical root cause: Logic flaw in the Linux kernel cryptographic subsystem; improper in-place handling around AF_ALG and `splice()` allows controlled page-cache corruption of readable files, including setuid binaries.
- Operational impact: Local unprivileged user or compromised container can escalate to UID 0; not remotely exploitable alone, but high-impact when chained after SSH, CI runner, webshell, or container foothold.

## 3. 🦠 Malware & Campaign Analysis

### Silver Fox ABCDoor / ValleyRAT Campaign

- Malware name / family: ABCDoor Python backdoor, ValleyRAT / Winos 4.0, modified RustSL loader, JavaScript loader.
- Threat actor: Silver Fox, China-based cybercrime/APT-style cluster.
- Initial access vector: Spear-phishing emails impersonating tax authorities; PDF files contain archive download links, or RAR/ZIP archives attach executables disguised with PDF/Excel icons.
- Execution chain:
  1. Victim opens tax-themed PDF or archive such as `ITD.-.rar`, `GST.pdf`, or Russian tax-lure ZIP content.
  2. User executes a disguised RustSL loader binary.
  3. Modified RustSL performs geofencing and sandbox/VM checks, then extracts/decrypts payloads using custom XOR-style payload packing with `<RSL_START>` / `<RSL_END>` markers.
  4. Shellcode downloads encrypted ValleyRAT / Winos 4.0 modules, including Online and Login modules.
  5. ValleyRAT retrieves and executes additional modules, including the ABCDoor loader/plugin.
  6. ABCDoor contacts HTTPS C2 and supports screenshot capture, remote keyboard/mouse control, file operations, process management, clipboard theft, update/removal, and data exfiltration.
- Persistence technique: Phantom Persistence in some RustSL samples; JavaScript delivery chain also sets `HKCU\Environment\UserInitMprLogonScript` after launching `python/pythonw.exe -m appclient`.
- C2 communication method: ValleyRAT TCP C2 configured with IP/port pairs such as `207.56.138[.]28:6666`; ABCDoor uses HTTPS to actor-controlled domains and `45.118.133[.]203:5000`.
- Targeting: Industrial, consulting, retail, and transportation organizations; highest telemetry in India, Russia, and Indonesia, with Japan added to newer RustSL geofencing.

### Sorry Ransomware via cPanel CVE-2026-41940

- Malware name / family: Sorry ransomware.
- Threat actor: Not publicly attributed today.
- Initial access vector: Internet-exposed cPanel/WHM instances vulnerable to CVE-2026-41940.
- Execution chain:
  1. Attacker sends crafted unauthenticated request to bypass cPanel/WHM authentication.
  2. Attacker obtains WHM administrative control and root-level host access.
  3. Post-exploitation reportedly includes unauthorized UID 0 accounts, SSH backdoors on non-standard ports, and file encryption.
  4. Files are encrypted with `.sorry` extension and ransom notes direct victims to qTox contact.
- Persistence technique: Unauthorized root-level accounts and SSH backdoors reported by victims; exact standardized ransomware persistence not observed today.
- C2 communication method: Not observed today.

### Harvester Linux GoGra Backdoor

- Malware name / family: Linux GoGra backdoor.
- Threat actor: Harvester.
- Initial access vector: Social engineering with ELF binaries disguised as PDF documents.
- Execution chain:
  1. Victim executes ELF masquerading as document.
  2. Dropper displays a lure document while running GoGra.
  3. GoGra polls Microsoft Graph API / Outlook mailbox folder `Zomato Pizza` every two seconds.
  4. C2 tasks arrive as email subjects beginning with `Input`; commands are Base64-decoded/decrypted and executed via `/bin/bash`.
  5. Results are sent back through email subjects beginning with `Output`, then tasking messages are deleted.
- Persistence technique: Not observed today.
- C2 communication method: Microsoft Graph API and Outlook mailbox as covert C2.
- Note: Published 2026-04-22, included only because it remains a top The Hacker News threat-intel story and matches the malware/configuration requirement; no new 2026-05-04 activity was observed.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "45.118.133[.]203:5000",
    "108.187.37[.]85",
    "108.187.42[.]63",
    "207.56.138[.]28",
    "108.187.41[.]221",
    "154.82.81[.]192",
    "139.180.128[.]251",
    "192.229.115[.]229",
    "207.56.119[.]216",
    "192.163.167[.]14",
    "45.192.219[.]60",
    "192.238.205[.]47",
    "45.32.108[.]178",
    "57.133.212[.]106",
    "154.82.81[.]205"
  ],
  "domains": [
    "abc.fetish-friends[.]com",
    "abc.3mkorealtd[.]com",
    "abc.sudsmama[.]com",
    "abc.woopami[.]com",
    "abc.ilptour[.]com",
    "abc.petitechanson[.]com",
    "abc.doublemobile[.]com",
    "mcagov[.]cc",
    "roldco[.]com",
    "vnc.kcii2[.]com",
    "abc.haijing88[.]com"
  ],
  "hashes": [
    "1AA72CD19E37570E14D898DFF3F2E380",
    "79CD56FC9ABF294B9BA8751E618EC642",
    "0B9B420E3EDD2ADE5EDC44F60CA745A2",
    "6611E902945E97A1B27F322A50566D48",
    "84E54C3602D8240ED905B07217C451CD",
    "2B92E125184469A0C3740ABCAA10350C",
    "043E457726F1BBB6046CB0C9869DBD7D",
    "6495C409B59DEB72CFCB2B2DA983B3BB",
    "B500E0A8C87DFFE6F20C6E067B51AFBF",
    "90257AA1E7C9118055C09D4A978D4BEE",
    "F8371097121549FEB21E3BCC2EEEA522",
    "814032EEC3BC31643F8FAA4234D0E049",
    "B53E3CC11947E5645DFBB19934B69833",
    "0C3B60FFC4EA9CCCE744BFA03B1A3556",
    "039E93B98EF5E329F8666A424237AE73",
    "B6DF7C59756AB655CA752B8A1B20CFFA",
    "2C5A1DD4CB53287FE0ED14E0B7B7B1B7",
    "E6362A81991323E198A463A8CE255533"
  ],
  "urls": [
    "hxxps://abc.haijing88[.]com/uploads/фнс/фнс.zip",
    "hxxps://abc.haijing88[.]com/uploads/印度邮箱/CBDT.rar",
    "hxxps://mcagov[.]cc/download.php?type=exe",
    "hxxps://abc.fetish-friends[.]com/uploads/appclient.zip",
    "hxxps://abc.fetish-friends[.]com/setup?channel=jiqi_0819",
    "hxxps://abc.fetish-friends[.]com/setup/install?channel=whatsapp_0826",
    "hxxps://abc.fetish-friends[.]com/setup/install?channel=dianhua-0903"
  ],
  "mutex": []
}
```

Registry / persistence artifacts:

- `HKCU\Environment\UserInitMprLogonScript`
- `HKCU:\Software\CarEmu:InstallChannel`
- `%USERPROFILE%\.node\node.exe`
- `%USERPROFILE%\AppData\Local\appclient`
- `cmd /c start /min python/pythonw.exe -m appclient`

## 5. 🔗 Technical Resources & Blogs

- The Hacker News: [Silver Fox Deploys ABCDoor Malware via Tax-Themed Phishing in India and Russia](https://thehackernews.com/2026/05/silver-fox-deploys-abcdoor-malware-via.html)
- Kaspersky Securelist: [Silver Fox uses the new ABCDoor backdoor to target organizations in Russia and India](https://securelist.com/silver-fox-tax-notification-campaign/119575/)
- The Hacker News: [CISA Adds Actively Exploited Linux Root Access Bug CVE-2026-31431 to KEV](https://thehackernews.com/2026/05/cisa-adds-actively-exploited-linux-root.html)
- Microsoft Security Blog: [CVE-2026-31431: Copy Fail vulnerability enables Linux root privilege escalation across cloud environments](https://www.microsoft.com/en-us/security/blog/2026/05/01/cve-2026-31431-copy-fail-vulnerability-enables-linux-root-privilege-escalation/)
- NVD: [CVE-2026-31431](https://nvd.nist.gov/vuln/detail/CVE-2026-31431)
- Theori PoC: [copy-fail-CVE-2026-31431](https://github.com/theori-io/copy-fail-CVE-2026-31431)
- BleepingComputer: [Critical cPanel flaw mass-exploited in Sorry ransomware attacks](https://www.bleepingcomputer.com/news/security/critrical-cpanel-flaw-mass-exploited-in-sorry-ransomware-attacks/)
- cPanel advisory: [Security: CVE-2026-41940 - cPanel & WHM / WP2 Security Update](https://support.cpanel.net/hc/en-us/articles/40073787579671-Security-CVE-2026-41940-cPanel-WHM-WP2-Security-Update-04-28-2026)
- Rapid7: [CVE-2026-41940: cPanel & WHM Authentication Bypass](https://www.rapid7.com/blog/post/etr-cve-2026-41940-cpanel-whm-authentication-bypass/)
- The Hacker News: [Harvester Deploys Linux GoGra Backdoor in South Asia Using Microsoft Graph API](https://thehackernews.com/2026/04/harvester-deploys-linux-gogra-backdoor.html)

## 6. 🧪 Sandbox / Sample Links

- Kaspersky OpenTIP sample/enrichment links are embedded in the Securelist IOC table for ABCDoor, ValleyRAT, RustSL loaders, PDFs, SFX archives, and scripts.
- VirusTotal: Not observed today.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.

## 7. 🛡️ Detection & Mitigation

### ABCDoor / Silver Fox

- YARA/Sigma: Public standalone YARA/Sigma rules not observed today.
- Detection ideas:
  - Alert on `cmd.exe` launching hidden `pythonw.exe -m appclient` from `%USERPROFILE%\AppData\Local\appclient`.
  - Hunt for `HKCU\Environment\UserInitMprLogonScript` creation or modification by script engines, SFX installers, NodeJS, Python, or unsigned binaries.
  - Detect non-browser processes calling public IP geolocation services: `ip-api.com`, `ipwho.is`, `ipinfo.io`, `ipapi.co`, `www.geoplugin.net`.
  - Inspect archives containing executable files with PDF/Excel icons and sibling payload files using extensions `.png`, `.htm`, `.md`, `.log`, `.xlsx`, `.ico`, `.cfg`, `.map`, `.xml`, or `.old`.
  - Hunt for payload markers `<RSL_START>` and `<RSL_END>` in suspicious local files.
- Mitigation:
  - Block listed ABCDoor/ValleyRAT infrastructure.
  - Quarantine hosts with RustSL/ValleyRAT indicators; assume credential, clipboard, and file-system exposure.
  - Add mail-gateway detonation for tax-themed PDFs containing archive links.

### CVE-2026-31431 / Copy Fail

- YARA/Sigma: Not observed today; Microsoft Defender detections include `Exploit:Linux/CopyFailExpDl.A`, `Exploit:Python/CopyFail.A`, `Exploit:Linux/CVE-2026-31431.A`, and `Behavior:Linux/CVE-2026-31431`.
- Detection ideas:
  - Prioritize hunting in CI runners, build agents, Kubernetes nodes, SSH bastions, multi-tenant Linux hosts, and internet-exposed workloads with recent low-privilege code execution.
  - Treat container compromise on vulnerable kernels as potential host compromise because the exploit targets shared kernel page cache.
  - Monitor suspicious AF_ALG socket creation, `splice()`-heavy short-lived processes, unexpected setuid binary execution after local script execution, and UID 0 process creation from unprivileged parent processes.
- Mitigation:
  - Patch to fixed vendor kernels; upstream fixed versions include 6.18.22, 6.19.12, and 7.0.
  - If patching is delayed, restrict local code execution paths, isolate vulnerable nodes, block AF_ALG socket creation where feasible, and recycle potentially exposed Kubernetes/container nodes.
  - FCEB due date in CISA KEV: 2026-05-15.

### CVE-2026-41940 / cPanel & WHM

- YARA/Sigma: Not observed today.
- Detection ideas:
  - Review cPanel/WHM access logs for anomalous unauthenticated admin-session creation, unusual newline/CRLF patterns in login/session requests, and new root sessions from unfamiliar IPs.
  - Hunt for unauthorized UID 0 users, modified SSH daemon configuration, new SSH listeners on `2222`, `8080`, `22000`, and newly encrypted `.sorry` files.
  - Treat any exposed unpatched cPanel server as potentially compromised, not merely vulnerable.
- Mitigation:
  - Update immediately to fixed cPanel & WHM versions: 11.110.0.97, 11.118.0.63, 11.126.0.54, 11.130.0.18, 11.132.0.29, 11.136.0.5, or 11.134.0.20; WP Squared 136.1.7.
  - If update cannot complete, isolate or shut down public cPanel/WHM/Webmail ports `2083`, `2087`, `2095`, and `2096` until patching and compromise assessment are complete.
  - Rebuild from known-good media if ransomware or UID 0 backdoors are confirmed.

## 8. 📊 Trends & Insights

- **Foothold-to-root compression:** Copy Fail gives attackers a reliable post-exploitation escalation step after otherwise low-grade footholds such as CI job execution, SSH access, or container compromise.
- **Administrative surfaces remain ransomware accelerators:** cPanel/WHM exploitation shows that hosting panels provide immediate scale: one unauthenticated control-plane bug can expose many websites, databases, and mailboxes per host.
- **Malware families are blending loaders and legitimate runtimes:** Silver Fox combines RustSL, NodeJS, Python, ValleyRAT, and custom ABCDoor modules to fragment detection and make each stage look different.
- **Geofencing is becoming more operationally deliberate:** Silver Fox RustSL selectively executes in target countries and has expanded configuration from India/Russia/Indonesia/South Africa to Japan.
- **X.com-derived signal:** Not observed today due to retrieval failure; no tweet-sourced IOC or sample was included without independent verification.
