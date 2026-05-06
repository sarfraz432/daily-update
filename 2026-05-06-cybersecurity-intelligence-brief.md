# Daily Cybersecurity Intelligence Brief - 2026-05-06

## 1. 🧠 Executive Summary

- **DAEMON Tools Windows installers are actively compromised** in a signed software supply-chain attack; Kaspersky reports trojanized versions 12.5.0.2421-12.5.0.2434 distributed from the legitimate site since 2026-04-08.
- **Weaver E-cology CVE-2026-22679 is under active exploitation** via an unauthenticated debug API RCE path, with payload drops and PowerShell retrieval attempts observed.
- **ScarCruft expanded BirdCall into Android supply-chain delivery** through trojanized APKs hosted on `sqgame[.]net`, targeting ethnic Koreans in China's Yanbian region and likely North Korean defector communities.
- **China-nexus UAT-8302 is reusing shared APT malware** including NetDraft/NosyDoor, CloudSorcerer, SNOWRUST/VShell, Deed RAT/ZingDoor, and Draculoader against government entities.
- **X.com review:** public X search was attempted for malware/security posts in the last 24 hours; no independently retrievable, date-valid X-only IOC lead was available in this environment.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-22679 - Weaver E-cology Debug API RCE

- Severity: Critical, CVSS 9.8.
- Affected systems: Weaver/Fanwei E-cology 10.0 before 20260312.
- Exploitation status: In-the-wild; Shadowserver observed exploitation from 2026-03-31, Vega reports earlier exploitation from 2026-03-17.
- Technical root cause: Unauthenticated exposed debug functionality at `/papi/esearch/data/devops/dubboApi/debug/method` permits attacker-controlled `interfaceName` / `methodName` command execution through Java/Dubbo helpers.
- Observed activity: Synchronous command execution through `java.exe`, `whoami`, `ipconfig`, `tasklist`, failed payload drops, MSI staging as `fanwei0324.msi`, and PowerShell payload retrieval attempts.

### CVE-2026-31431 - Linux Kernel "Copy Fail" LPE

- Severity: High, CVSS 7.8.
- Affected systems: Linux kernels released from 2017 until fixed vendor kernels; Microsoft lists Ubuntu 24.04 LTS, Amazon Linux 2023, RHEL 10.1, SUSE 16, Debian, Fedora, and Arch-like distributions as impacted classes.
- Exploitation status: CISA KEV-listed; Microsoft reports limited exploitation mainly in PoC/preliminary testing, with high likelihood of broader weaponization.
- Technical root cause: Logic flaw in the Linux AF_ALG `algif_aead` crypto interface and `splice()` interaction allows an unprivileged local attacker to perform a controlled 4-byte page-cache overwrite and corrupt privileged binaries in memory.
- Impact: Root privilege escalation, container escape risk, multi-tenant host compromise, and lateral movement enablement after any local/container foothold.

### DAEMON Tools Signed Installer Supply-Chain Compromise

- Severity: High.
- Affected systems: Windows DAEMON Tools Lite installations, versions 12.5.0.2421-12.5.0.2434.
- Exploitation status: Active supply-chain compromise as of Kaspersky's 2026-05-05 reporting.
- Technical root cause: Vendor-distributed signed binaries `DTHelper.exe`, `DiscSoftBusServiceLite.exe`, and `DTShellHlp.exe` were trojanized; malicious startup code activates a backdoor thread during normal application/service startup.
- Impact: Profiling of thousands of infected systems globally, selective second-stage delivery to retail, scientific, government, manufacturing, and education targets.

### MOVEit Automation CVE-2026-4670 / CVE-2026-5174

- Severity: Critical / High.
- Affected systems: Progress MOVEit Automation versions before 2025.1.5, 2025.0.9, and 2024.1.8.
- Exploitation status: Not observed today.
- Technical root cause: Authentication bypass and improper input validation through backend command-port service interfaces.
- Impact: Administrative access, data exposure, and high extortion risk due to managed file-transfer workflow sensitivity.

## 3. 🦠 Malware & Campaign Analysis

### DAEMON Tools Backdoor / QUIC RAT Supply Chain

- Malware name / family: DAEMON Tools implant, `envchk.exe` profiler, minimalist backdoor, QUIC RAT.
- Threat actor: Unattributed; Kaspersky notes Chinese-language artifacts but does not attribute.
- Initial access vector: Trusted signed Windows installers from the official DAEMON Tools distribution channel.
- Execution chain:
  1. User installs trojanized DAEMON Tools Lite 12.5.0.2421-12.5.0.2434.
  2. Startup execution of `DTHelper.exe`, `DiscSoftBusServiceLite.exe`, or `DTShellHlp.exe` launches injected backdoor logic.
  3. Implant sends HTTP GET to `env-check.daemontools[.]cc/2032716822411?s=<hostname>`.
  4. C2 returns `cmd.exe` / PowerShell download commands for payloads from `38.180.107[.]76`.
  5. `envchk.exe` collects MAC, hostname, DNS domain, process list, installed software, and locale.
  6. Selected hosts receive `cdg.exe` / `cdg.tmp` shellcode loader and a minimalist backdoor; a Russian education victim received QUIC RAT.
- Persistence technique: Trojanized signed DAEMON Tools binaries run at system startup through normal product service/helper execution.
- C2 communication method: HTTP GET/POST for profiling and payload retrieval; QUIC RAT supports HTTP, UDP, TCP, WSS, QUIC, DNS, and HTTP/3, with injection into `notepad.exe` and `conhost.exe`.

### ScarCruft BirdCall Android/Windows Supply Chain

- Malware name / family: BirdCall Android backdoor, Windows BirdCall/RokRAT chain.
- Threat actor: ScarCruft, North Korea-aligned state-sponsored actor.
- Initial access vector: Compromised Yanbian-themed gaming platform `sqgame[.]net` / `sqgame.com[.]cn` serving trojanized Android APKs and previously a trojanized Windows update DLL.
- Execution chain:
  1. Victim downloads trojanized Android game APKs `ybht.apk` or `sqybhs.apk` from the compromised gaming site.
  2. APK runs as a legitimate-looking game while BirdCall collects contacts, SMS, call logs, media, documents, screenshots, and ambient audio.
  3. Windows chain, observed historically, used trojanized `mono.dll` downloader with anti-analysis checks.
  4. Windows downloader retrieved shellcode leading to RokRAT, then BirdCall installation.
- Persistence technique: Android app persistence through malicious installed application; Windows persistence details not fully published in open reporting.
- C2 communication method: Abuse of legitimate cloud storage services, including Dropbox, pCloud, Yandex Disk, and Zoho WorkDrive.

### UAT-8302 China-Nexus Government Intrusions

- Malware name / family: NetDraft/NosyDoor, CloudSorcerer v3, SNOWRUST/SNOWLIGHT, VShell, Deed RAT/Snappybee, ZingDoor, Draculoader, Stowaway, SoftEther VPN.
- Threat actor: UAT-8302, China-nexus APT cluster tracked by Cisco Talos.
- Initial access vector: Not confirmed; Talos suspects exploitation of zero-day and N-day web-application vulnerabilities.
- Execution chain:
  1. Initial compromise of government networks in South America and southeastern Europe.
  2. Reconnaissance and automated scanning using open-source tools such as `gogo`.
  3. Lateral movement and deployment of shared China-aligned malware families.
  4. SNOWRUST downloads and executes VShell from remote infrastructure.
  5. Stowaway and SoftEther VPN provide alternate backdoor/proxy access.
- Persistence technique: Custom backdoors plus proxy/VPN tooling for redundant operator access.
- C2 communication method: HTTPS and direct IP infrastructure listed in Talos IOCs; VShell and NetDraft-family C2 over attacker-controlled web infrastructure.

### Microsoft Code-of-Conduct AiTM Phishing

- Malware name / family: Not malware; credential/token theft campaign using AiTM phishing.
- Threat actor: Unattributed.
- Initial access vector: Legitimate email delivery services sending conduct-review themed lures with PDF attachments.
- Execution chain:
  1. Email impersonates internal conduct/compliance processes.
  2. PDF attachment links to attacker-controlled infrastructure.
  3. Victim passes through CAPTCHA and intermediate pages.
  4. AiTM sign-in page harvests Microsoft credentials and session tokens in real time.
- Persistence technique: Session-token theft and cloud account access; no endpoint persistence observed.
- C2 communication method: HTTPS phishing infrastructure.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "38.180.107[.]76",
    "205.209.116[.]54",
    "161.132.49[.]114",
    "141.11.89[.]42",
    "132.243.172[.]2",
    "152.32.173[.]138",
    "39.106.249[.]68",
    "211.239.117[.]117",
    "114.108.128[.]157",
    "221.143.43[.]214",
    "222.231.2[.]20",
    "222.231.2[.]23",
    "222.231.2[.]41",
    "85.209.156[.]3",
    "185.238.189[.]41",
    "103.27.108[.]55",
    "38.54.32[.]244"
  ],
  "domains": [
    "env-check.daemontools[.]cc",
    "sqgame.com[.]cn",
    "sqgame[.]net",
    "1980food.co[.]kr",
    "inodea[.]com",
    "www.lawwell.co[.]kr",
    "colorncopy.co[.]kr",
    "swr.co[.]kr",
    "sejonghaeun[.]com",
    "cndsoft.co[.]kr",
    "www.drivelivelime[.]com",
    "msiidentity[.]com",
    "trafficmanagerupdate[.]com",
    "image.update-kaspersky[.]workers[.]dev",
    "update-kaspersky[.]workers[.]dev",
    "compliance-protectionoutlook[.]de",
    "acceptable-use-policy-calendly[.]de",
    "cocinternal[.]com",
    "gadellinet[.]com",
    "harteprn[.]com"
  ],
  "hashes": [
    "9ccd769624de98eeeb12714ff1707ec4f5bf196d",
    "50d47adb6dd45215c7cb4c68bae28b129ca09645",
    "0c1d3da9c7a651ba40b40e12d48ebd32b3f31820",
    "28b72576d67ae21d9587d782942628ea46dcc870",
    "46b90bf370e60d61075d3472828fdc0b85ab0492",
    "6325179f442e5b1a716580cd70dea644ac9ecd18",
    "bd8fbb5e6842df8683163adbd6a36136164eac58",
    "15ed5c3384e12fe4314ad6edbd1dcccf5ac1ee29",
    "2d4eb55b01f59c62c6de9aacba9b47267d398fe4",
    "9dbfc23ebf36b3c0b56d2f93116abb32656c42e4",
    "295ce86226b933e7262c2ce4b36bdd6c389aaaef",
    "147ac3f24b2b63544d65070007888195a98d30e380f2d480edffb3f07a78377f",
    "1139b39d3cc151ddd3d574617cf113608127850197e9695fef0b6d78df82d6ca",
    "ee56c49f42522637f401d15ac2a2b6f3423bfb2d5d37d071f0172ce9dc688d4b",
    "51f0cf80a56f322892eed3b9f5ecae45f1431323600edbaea5cd1f28b437f6f2",
    "35b2a5260b21ddb145486771ec2b1e4dc1f5b7f2275309e139e4abc1da0c614b",
    "199bd156c81b2ef4fb259467a20eacaa9d861eeb2002f1570727c2f9ff1d5dab",
    "071e662fc5bc0e54bcfd49493467062570d0307dc46f0fb51a68239d281427c6",
    "01A33066FBC6253304C92760916329ABD50C3191",
    "03E3ECE9F48CF4104AAFC535790CA2FB3C6B26CF",
    "95BDB94F6767A3CCE6D92363BBF5BC84B786BDB0",
    "5DB1ECBBB2C90C51D81BDA138D4300B90EA5EB2885CCE1BD921D692214AECBC6"
  ],
  "urls": [
    "https://env-check.daemontools[.]cc/2032716822411?s=<full_computer_name>",
    "http://38.180.107[.]76/env_check_script",
    "http://38.180.107[.]76/09505aca4f538bd",
    "http://205.209.116[.]54:2013/vsgbt.exe",
    "http://205.209.116[.]54:2013/hjchhb.exe",
    "http://161.132.49[.]114/config.js",
    "http://141.11.89[.]42/fanwei0324.msi",
    "http://132.243.172[.]2/config/xx.ps1",
    "http://132.243.172[.]2/w-2026/x.ps1",
    "http://152.32.173[.]138/U<16hex>.<8hex>",
    "hxxps://www.drivelivelime[.]com/x",
    "hxxps://www.drivelivelime[.]com/pw",
    "hxxps://msiidentity[.]com/pw",
    "hxxp://trafficmanagerupdate[.]com/index[.]php",
    "hxxp://85.209.156[.]3:8080/wagent.exe",
    "hxxp://85.209.156[.]3:8082/wagent.exe",
    "hxxp://103.27.108[.]55:48265/",
    "hxxp://38.54.32[.]244/Rar.exe",
    "sqgame.com[.]cn/ybht.apk",
    "sqgame.com[.]cn/sqybhs.apk"
  ],
  "mutex": []
}
```

Host artifacts:

- `C:\Program Files\DAEMON Tools Lite\DTHelper.exe`
- `C:\Program Files\DAEMON Tools Lite\DiscSoftBusServiceLite.exe`
- `C:\Program Files\DAEMON Tools Lite\DTShellHlp.exe`
- `C:\Windows\Temp\envchk.exe`
- `C:\Windows\Temp\cdg.exe`
- `C:\Windows\Temp\imp.tmp`
- `C:\Windows\Temp\piyu.exe`
- `fanwei0324.msi`
- `vsgbt.exe`
- `hjchhb.exe`
- `nvm.exe`
- `2.txt` renamed PowerShell executable
- `xx.ps1`
- `x.ps1`

## 5. 🔗 Technical Resources & Blogs

- The Hacker News: [DAEMON Tools Supply Chain Attack Compromises Official Installers with Malware](https://thehackernews.com/2026/05/daemon-tools-supply-chain-attack.html)
- Kaspersky Securelist: [DAEMON Tools software infected - supply chain attack ongoing since April 8, 2026](https://securelist.com/tr/daemon-tools-backdoor/119654/)
- The Hacker News: [Weaver E-cology RCE Flaw CVE-2026-22679 Actively Exploited via Debug API](https://thehackernews.com/2026/05/weaver-e-cology-rce-flaw-cve-2026-22679.html)
- Vega: [Ping, Payload, PowerShell: Active Exploitation of CVE-2026-22679 in Weaver E-cology](https://blog.vega.io/posts/cve-2026-22679-weaver-ecology-exploitation/)
- Microsoft: [CVE-2026-31431: Copy Fail vulnerability enables Linux root privilege escalation](https://www.microsoft.com/en-us/security/blog/2026/05/01/cve-2026-31431-copy-fail-vulnerability-enables-linux-root-privilege-escalation/)
- Cisco Talos: [UAT-8302 and its box full of malware](https://blog.talosintelligence.com/uat-8302/)
- The Hacker News: [ScarCruft Hacks Gaming Platform to Deploy BirdCall Malware on Android and Windows](https://thehackernews.com/2026/05/scarcruft-hacks-gaming-platform-to.html)
- ESET: [A rigged game: ScarCruft compromises gaming platform in a supply-chain attack](https://www.welivesecurity.com/en/eset-research/rigged-game-scarcruft-compromises-gaming-platform-supply-chain-attack/)
- Microsoft: [Multi-stage code of conduct phishing campaign leads to AiTM token compromise](https://www.microsoft.com/en-us/security/blog/2026/05/04/breaking-the-code-multi-stage-code-of-conduct-phishing-campaign-leads-to-aitm-token-compromise/)
- GitHub PoC/detection: [CVE-2026-22679 Weaver scanner](https://github.com/kerem0x00/CVE-2026-22679), [ESET malware-ioc repository](https://github.com/eset/malware-ioc), [Cisco Talos IOCs](https://github.com/Cisco-Talos/IOCs)

## 6. 🧪 Sandbox / Sample Links

- Kaspersky OpenTIP sample links are embedded in the Securelist IOC table for DAEMON Tools installers, modified binaries, `envchk.exe`, and backdoor components.
- ESET sample references are available through the linked ESET malware IOC GitHub repository.
- Cisco Talos sample hashes are available through the linked Talos IOC GitHub repository.
- VirusTotal: Not observed today.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.

## 7. 🛡️ Detection & Mitigation

### DAEMON Tools Supply Chain

- YARA/Sigma: Public standalone YARA/Sigma not observed today; Kaspersky published product detections and EDR/MDR detection logic.
- Detection ideas:
  - Hunt DAEMON Tools helper binaries spawning `cmd.exe` or PowerShell.
  - Alert on outbound traffic to `env-check.daemontools[.]cc` or `38.180.107[.]76`.
  - Detect PowerShell `WebClient.DownloadFile` from DAEMON Tools process ancestry into `%TEMP%`.
  - Hunt code injection from Temp/AppData/Public-staged executables into `notepad.exe` or `conhost.exe`.
  - Compare installed DAEMON Tools helper hashes against vendor-clean baselines and the listed malicious SHA1 values.
- Mitigation:
  - Isolate hosts with affected DAEMON Tools versions 12.5.0.2421-12.5.0.2434.
  - Remove affected builds, block IOCs, and run endpoint sweeps for second-stage payloads.
  - Treat profiled hosts in sensitive sectors as possible selective follow-on targets even if no QUIC RAT is found.

### Weaver E-cology CVE-2026-22679

- YARA/Sigma: Not observed today.
- Detection ideas:
  - Alert on POST requests to `/papi/esearch/data/devops/dubboApi/debug/method`.
  - Hunt `java.exe` spawning `cmd.exe`, `powershell.exe`, `whoami`, `ipconfig`, `tasklist`, MSI installers, or renamed PowerShell binaries.
  - Review web logs for debug endpoint response bodies returning command stdout/stderr.
  - Block and hunt downloads from the Vega-listed payload IPs and URLs.
- Mitigation:
  - Upgrade Weaver E-cology 10.0 to 20260312 or later.
  - Restrict external access to E-cology administrative/debug endpoints.
  - If exploitation is detected, assume no persistent shell is required because the exposed debug API itself acts as the shell.

### Linux Copy Fail CVE-2026-31431

- YARA/Sigma: Microsoft published Defender detection names: `Exploit:Linux/CopyFailExpDl.A`, `Exploit:Python/CopyFail.A`, `Exploit:Linux/CVE-2026-31431.A`, and `Behavior:Linux/CVE-2026-31431`.
- Detection ideas:
  - Hunt low-privileged processes invoking AF_ALG crypto sockets and `splice()` patterns followed by privileged binary execution such as `/usr/bin/su`.
  - Treat any untrusted container code execution on vulnerable kernels as potential host compromise.
  - Monitor CI/CD runners, shared Kubernetes nodes, and multi-tenant Linux hosts for unexpected UID 0 transitions.
- Mitigation:
  - Patch distribution kernels immediately; where patching is delayed, block AF_ALG socket creation or disable affected crypto functionality if operationally viable.
  - Recycle Kubernetes nodes after suspected exploitation; disk integrity alone is insufficient because exploitation can be in-memory.

### ScarCruft BirdCall

- YARA/Sigma: ESET published hashes and ATT&CK mapping; standalone YARA not observed today.
- Detection ideas:
  - Block trojanized APK downloads from `sqgame.com[.]cn/ybht.apk` and `sqgame.com[.]cn/sqybhs.apk`.
  - Hunt Android apps requesting contacts, SMS, call log, microphone, screenshot/media access while communicating with cloud storage APIs.
  - On Windows, detect `mono.dll` replacement/update anomalies and downloader behavior that checks VM/debugger tools before fetching shellcode.
- Mitigation:
  - Remove APKs sourced from the compromised gaming platform; reset credentials and review mobile data exposure for targeted users.
  - Monitor Dropbox, pCloud, Yandex Disk, and Zoho WorkDrive API usage from unmanaged mobile applications.

### UAT-8302

- YARA/Sigma: Cisco Talos published ClamAV detections and Snort SIDs 66055, 66054, 301437-301431, 66052-66040.
- Detection ideas:
  - Hunt for VShell, Stowaway, SoftEther VPN, `gogo`, and `naabu` usage inside government networks.
  - Alert on traffic to `drivelivelime[.]com`, `msiidentity[.]com`, `trafficmanagerupdate[.]com`, and the listed direct IPs/ports.
  - Correlate web-app exploit telemetry with rapid scanner deployment and multiple China-nexus backdoors.
- Mitigation:
  - Patch internet-facing web applications aggressively; initial access is suspected to rely on zero-day/N-day web exploitation.
  - Treat any one listed malware family as a marker for possible shared-access partner tooling and hunt laterally for the full toolset.

## 8. 📊 Trends & Insights

- **Software supply-chain targeting is accelerating:** DAEMON Tools joins recent trusted-software compromises where signed binaries and official delivery channels defeat perimeter trust assumptions.
- **Cloud/SaaS services remain favored C2 substrate:** ScarCruft's BirdCall and UAT-linked tooling continue the shift toward legitimate cloud storage, worker domains, and normal HTTPS infrastructure.
- **Debug/admin surfaces are being treated as shells:** Weaver exploitation shows operators can conduct discovery and payload delivery through synchronous API responses without deploying a traditional agent.
- **Local privilege bugs are now cloud-impact bugs:** Copy Fail is local-only, but container, CI/CD, and shared kernel environments make it highly relevant to post-exploitation paths.
- **X.com signal was not independently consumable today:** No X-only malware sample, config, or IOC was included without primary-source corroboration.
