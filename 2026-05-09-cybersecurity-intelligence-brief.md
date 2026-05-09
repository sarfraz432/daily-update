# Daily Cybersecurity Intelligence Brief - 2026-05-09

## 1. 🧠 Executive Summary

- Active exploitation remains concentrated on perimeter and admin-plane systems: PAN-OS User-ID Authentication Portal RCE, Ivanti EPMM authenticated RCE, Linux kernel Dirty Frag LPE, and recently weaponized cPanel/WHM flaws.
- Developer and AI supply-chain users are high-priority targets: fake Hugging Face/OpenAI repositories, QLNX Linux RAT, and PCPJack all focus on stealing cloud, package registry, SSH, and CI/CD credentials.
- New malware families observed in the last 24-48 hours include TCLBANKER, QLNX, PamDOORa, PCPJack, and Beagle; TCLBANKER and the fake Hugging Face infostealer include detailed configuration/C2 behavior.
- Initial access trends: typosquatted AI/ML assets, trojanized installers, compromised WordPress ClickFix chains, exposed cloud services, and exploitation of internet-facing security appliances.
- X.com review: public search showed limited accessible posts, but indexed X activity around Dirty Frag disclosure and security vendor posts aligned with the same issues; no unique IOC-only X thread was independently observable today.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-0300 - PAN-OS User-ID Authentication Portal RCE

- Severity: Critical
- Affected systems: Palo Alto Networks PA-Series and VM-Series firewalls with internet-exposed User-ID Authentication Portal/Captive Portal.
- Exploitation status: In-the-wild, likely state-sponsored activity tracked by Unit 42 as CL-STA-1132.
- Technical root cause: Buffer overflow enabling unauthenticated remote code execution as root.
- Notes: Exploitation attempts reportedly began April 9, 2026; successful compromise led to shellcode injection, log cleanup, and deployment of EarthWorm/ReverseSocks5 tunneling tools.
- Source: [Unit 42](https://unit42.paloaltonetworks.com/captive-portal-zero-day/), [BleepingComputer](https://www.bleepingcomputer.com/news/security/pan-os-firewall-rce-zero-day-exploited-in-attacks-since-april-9/), [The Hacker News](https://thehackernews.com/2026/05/pan-os-rce-exploit-under-active-use.html)

### CVE-2026-6973 - Ivanti EPMM RCE

- Severity: High
- Affected systems: Ivanti Endpoint Manager Mobile on-prem versions 12.8.0.0 and earlier.
- Exploitation status: Limited in-the-wild exploitation; added to CISA KEV with a May 10, 2026 federal remediation deadline.
- Technical root cause: Improper input validation allowing remotely authenticated admin users to execute arbitrary code.
- Source: [BleepingComputer](https://www.bleepingcomputer.com/news/security/cisa-gives-feds-four-days-to-patch-ivanti-flaw-exploited-as-zero-day/), [The Hacker News](https://thehackernews.com/2026/05/ivanti-epmm-cve-2026-6973-rce-under.html)

### Dirty Frag - Linux Kernel LPE

- Severity: High
- Affected systems: Major Linux distributions including Ubuntu, RHEL, CentOS Stream, AlmaLinux, openSUSE Tumbleweed, and Fedora.
- Exploitation status: Public PoC available; no confirmed in-the-wild exploitation observed today.
- Technical root cause: Deterministic page-cache write bug class chaining xfrm-ESP and RxRPC page-cache write flaws to modify protected files in memory and gain root.
- Source: [BleepingComputer](https://www.bleepingcomputer.com/news/security/new-linux-dirty-frag-zero-day-with-poc-exploit-gives-root-privileges/), [The Hacker News](https://thehackernews.com/2026/05/linux-kernel-dirty-frag-lpe-exploit.html), [Dirty Frag GitHub](https://github.com/v4bel/dirty-frag)

### CVE-2026-29201 / CVE-2026-29202 / CVE-2026-29203 - cPanel/WHM

- Severity: Medium to High
- Affected systems: cPanel and WHM, WP Squared.
- Exploitation status: No exploitation observed for these three today; related cPanel CVE-2026-41940 was recently weaponized for Mirai variants and Sorry ransomware.
- Technical root cause: Insufficient input validation leading to arbitrary file read and arbitrary Perl code execution; unsafe symlink handling causing DoS or privilege escalation.
- Source: [The Hacker News](https://thehackernews.com/2026/05/cpanel-whm-patch-3-new-vulnerabilities.html), [BleepingComputer on CVE-2026-41940/Sorry](https://www.bleepingcomputer.com/news/security/critrical-cpanel-flaw-mass-exploited-in-sorry-ransomware-attacks/)

## 3. 🦠 Malware & Campaign Analysis

### TCLBANKER / REF3076

- Malware family: TCLBANKER, Brazilian banking trojan; assessed as major Maverick/SORVEPOTEL evolution.
- Threat actor: Unknown; ecosystem overlap with Brazilian banking trojans and Water Saci lineage.
- Initial access vector: ZIP containing malicious MSI abusing signed Logitech AI Prompt Builder.
- Execution chain: MSI installs Logitech binary -> DLL sideloading of `screen_retriever_plugin.dll` -> anti-analysis/environment hash checks -> payload decryption -> banking trojan + worm modules -> WebSocket operator session when target financial URL is detected.
- Persistence technique: Scheduled task.
- C2 method: HTTP POST beacon with host data; WebSocket command loop for remote control and social engineering overlays.
- Notable config: Hard-coded list of 59 banking/fintech/crypto targets; Brazil locale/timezone/keyboard checks; server-delivered WhatsApp message templates.
- Source: [The Hacker News](https://thehackernews.com/2026/05/tclbanker-banking-trojan-targets.html), [Elastic Security Labs](https://www.elastic.co/security-labs/tclbanker-brazilian-banking-trojan)

### Fake OpenAI Privacy Filter on Hugging Face / Rust Infostealer

- Malware family: Rust-based infostealer; infrastructure overlaps with WinOS 4.0 typosquatting activity.
- Threat actor: Unknown supply-chain actor.
- Initial access vector: Typosquatted Hugging Face repository `Open-OSS/privacy-filter`, artificially boosted to trending status.
- Execution chain: User runs `loader.py` or `start.bat` -> Base64 URL decodes to JsonKeeper paste -> hidden PowerShell downloads `update.bat` -> UAC self-elevation -> downloads `sefirah` payload -> Defender exclusions -> one-shot SYSTEM scheduled task -> Rust infostealer execution.
- Persistence technique: No durable persistence observed; scheduled task is used as a SYSTEM launcher and deleted.
- C2 method: HTTPS download infrastructure and WinHTTP POST exfiltration with gzip JSON and bearer authorization to `recargapopular[.]com`.
- Source: [BleepingComputer](https://www.bleepingcomputer.com/news/security/fake-openai-repository-on-hugging-face-pushes-infostealer-malware/), [HiddenLayer](https://www.hiddenlayer.com/research/malware-found-in-trending-hugging-face-repository-open-oss-privacy-filter)

### Quasar Linux RAT / QLNX

- Malware family: QLNX, previously undocumented Linux RAT/rootkit.
- Threat actor: Unknown.
- Initial access vector: Not observed today.
- Execution chain: In-memory Linux implant -> process masquerading as kernel thread -> host profiling/container detection -> log wiping -> credential harvesting -> raw TCP/HTTPS/HTTP C2 loop.
- Persistence technique: Multiple redundant methods including systemd, crontab, `.bashrc` shell injection, PAM inline-hook backdoor, LD_PRELOAD userland rootkit, and eBPF concealment.
- C2 method: Raw TCP, HTTPS, HTTP; supports 58 commands, SOCKS proxies, TCP tunnels, BOFs, and P2P mesh.
- Source: [The Hacker News](https://thehackernews.com/2026/05/quasar-linux-rat-steals-developer.html), [Trend Micro](https://www.trendmicro.com/)

### PCPJack

- Malware family: PCPJack credential-stealing cloud worm.
- Threat actor: Unknown; SentinelLabs assesses possible former TeamPCP operator/affiliate.
- Initial access vector: Exposed cloud services and vulnerable applications targeting Docker, Kubernetes, Redis, MongoDB, RayML, and web apps.
- Execution chain: `bootstrap.sh` -> hidden working directory -> Python dependencies -> module download -> persistence -> `monitor.py` orchestrator -> TeamPCP artifact removal -> credential harvesting and lateral movement.
- Persistence technique: Installed Linux services/processes; exact mechanism varies by module.
- C2 method: Telegram exfiltration after X25519 ECDH + ChaCha20-Poly1305 encryption and chunking into 2800-byte messages.
- Source: [The Hacker News](https://thehackernews.com/2026/05/pcpjack-credential-stealer-exploits-5.html), [SentinelLabs](https://www.sentinelone.com/labs/cloud-worm-evicts-teampcp-and-steals-credentials-at-scale/)

### PamDOORa

- Malware family: PamDOORa Linux PAM backdoor.
- Threat actor: Advertised by `darkworm` on Rehub forum; operational use not observed today.
- Initial access vector: Post-exploitation deployment after attacker obtains root by other means.
- Execution chain: Root access -> malicious PAM module deployment -> OpenSSH authentication interception -> magic-password/magic-port backdoor -> credential capture -> anti-forensic log tampering.
- Persistence technique: PAM module persistence in SSH authentication path.
- C2 communication method: Not observed today.
- Source: [The Hacker News](https://thehackernews.com/2026/05/new-linux-pamdoora-backdoor-uses-pam.html)

### Beagle Backdoor via Fake Claude-Pro

- Malware family: Beagle Windows backdoor; distinct from historical Bagle/Beagle worm.
- Threat actor: Unknown; PlugX-style sideloading overlap noted.
- Initial access vector: Fake `claude-pro[.]com` site offering `Claude-Pro-windows-x64.zip`.
- Execution chain: MSI installer -> Startup folder artifacts `NOVupdate.exe`, `NOVupdate.exe.dat`, `avk.dll` -> signed G Data updater DLL sideloading -> DonutLoader -> Beagle backdoor.
- Persistence technique: Startup folder artifacts.
- C2 communication method: Not observed today in accessible source details.
- Source: [BleepingComputer](https://www.bleepingcomputer.com/news/security/fake-claude-ai-website-delivers-new-beagle-windows-malware/), [Malwarebytes](https://www.malwarebytes.com/blog/scams/2026/04/fake-claude-site-installs-malware-that-gives-attackers-access-to-your-computer)

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "89.124.93.110"
  ],
  "domains": [
    "api.eth-fastscan.org",
    "recargapopular.com",
    "welovechinatown.info",
    "jsonkeeper.com",
    "claude-pro.com"
  ],
  "hashes": [
    "6db01158b044f178c45754666e2cbc0365f394e953fbf99ec34aa5304d5b79b1",
    "6d5b1b7b9b95f2074094632e3962dc21432c2b7dccfbbe2c7d61f724ffcfea7c",
    "4fba92a34fd9338293de53444bc9f05c278897d903a24efb95fde0522b3d50c0",
    "04f0569971ac7ff81c8656e8453a69189d8870040044909dad45c04c567e7564",
    "ba67720dd115293ec5a12d08be6b0ee982227a4c5e4662fb89269c76556df6e0",
    "c1b59cc25bdc1fe3f3ce8eda06d002dda7cb02dea8c29877b68d04cd089363c7"
  ],
  "urls": [
    "https://huggingface.co/Open-OSS/privacy-filter",
    "https://huggingface.co/anthfu/Bonsai-8B-gguf",
    "https://huggingface.co/anthfu/Qwen3.6-35B-A3B-APEX-GGUF",
    "https://huggingface.co/anthfu/DeepSeek-V4-Pro",
    "https://huggingface.co/anthfu/Qwopus-GLM-18B-Merged-GGUF",
    "https://huggingface.co/anthfu/Qwen3.6-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-GGUF",
    "https://huggingface.co/anthfu/supergemma4-26b-uncensored-gguf-v2",
    "https://jsonkeeper.com/b/AVNNE",
    "https://api.eth-fastscan.org/update.bat",
    "https://api.eth-fastscan.org/sefirah"
  ],
  "mutex": []
}
```

Host artifacts:

- `%TMP%\node.b64`
- `%TMP%\runner.ps1`
- Scheduled task regex: `MicrosoftEdgeUpdateTaskCore[a-z0-9]{8}$`
- TCLBANKER DLL: `screen_retriever_plugin.dll`
- Beagle chain: `NOVupdate.exe`, `NOVupdate.exe.dat`, `avk.dll`

## 5. 🔗 Technical Resources & Blogs

- [The Hacker News Threat Intelligence section](https://thehackernews.com/search/label/Threat%20Intelligence)
- [TCLBANKER - The Hacker News](https://thehackernews.com/2026/05/tclbanker-banking-trojan-targets.html)
- [TCLBANKER - Elastic Security Labs](https://www.elastic.co/security-labs/tclbanker-brazilian-banking-trojan)
- [QLNX Linux RAT - The Hacker News](https://thehackernews.com/2026/05/quasar-linux-rat-steals-developer.html)
- [PamDOORa - The Hacker News](https://thehackernews.com/2026/05/new-linux-pamdoora-backdoor-uses-pam.html)
- [PCPJack - SentinelLabs](https://www.sentinelone.com/labs/cloud-worm-evicts-teampcp-and-steals-credentials-at-scale/)
- [PAN-OS CVE-2026-0300 - Unit 42](https://unit42.paloaltonetworks.com/captive-portal-zero-day/)
- [Fake OpenAI Hugging Face repo - HiddenLayer](https://www.hiddenlayer.com/research/malware-found-in-trending-hugging-face-repository-open-oss-privacy-filter)
- [Dirty Frag GitHub PoC/documentation](https://github.com/v4bel/dirty-frag)

## 6. 🧪 Sandbox / Sample Links

- HiddenLayer referenced a sandbox execution: [Triage sample context](https://tria.ge/)
- VirusTotal links: Not observed today in accessible source details.
- Any.Run / Hybrid Analysis links: Not observed today.
- MalwareBazaar sample links: Not observed today.

## 7. 🛡️ Detection & Mitigation

- PAN-OS: Restrict User-ID Authentication Portal exposure to trusted zones or disable if unused; disable Response Pages on L3 interfaces accepting untrusted traffic; monitor for nginx crash/core cleanup, crash kernel message clearing, EarthWorm/ReverseSocks5 artifacts, and unexpected SOCKS tunneling.
- Ivanti EPMM: Upgrade to 12.6.1.1, 12.7.0.1, or 12.8.0.1; audit admin accounts; rotate admin credentials, especially if exposed to January EPMM exploitation.
- Dirty Frag: Treat public PoC as high-risk. Prioritize kernel vendor advisories, restrict local shell access, monitor for unexpected privileged file modifications and root-owned file changes following unprivileged process activity.
- TCLBANKER: Hunt for Logitech AI Prompt Builder abuse, `screen_retriever_plugin.dll` sideloading, ETW/ntdll unhooking attempts, scheduled task creation, browser URL polling via UI Automation, WebSocket sessions after visits to financial sites, and Outlook/WhatsApp Web mass messaging.
- Hugging Face infostealer: Block listed domains/URLs; hunt for `powershell.exe -ExecutionPolicy Bypass -WindowStyle Hidden`, `cmd.exe /k %TEMP%\update.bat`, Defender exclusions in `%TEMP%`, `%LOCALAPPDATA%`, `%APPDATA%`, and deleted `MicrosoftEdgeUpdateTaskCore*` tasks.
- QLNX: Detect kernel-thread name masquerading by userland processes, LD_PRELOAD anomalies, suspicious eBPF hiding behavior, PAM module changes, log wiping, and access to `.npmrc`, `.pypirc`, `.aws/credentials`, `.kube/config`, `.docker/config.json`, `.env`, and GitHub CLI tokens.
- PCPJack: Hunt for `bootstrap.sh`, `monitor.py`, hidden Python working directories, Telegram exfiltration, deletion of TeamPCP artifacts, cloud metadata access, and credential file enumeration across Docker/Kubernetes/Redis/MongoDB/RayML hosts.
- YARA/Sigma: No vendor-published YARA/Sigma rules observed today in accessible reports; behavioral detections above are recommended until validated rules are released.

## 8. 📊 Trends & Insights

- AI/ML ecosystems are now a primary malware delivery surface: attackers are typosquatting model repositories, inflating engagement metrics, and exploiting developer trust in loader scripts.
- Linux credential theft is converging with rootkit and PAM persistence: QLNX and PamDOORa show increasing focus on SSH, package registry, cloud, and CI/CD credentials.
- Worm-like propagation is returning in targeted forms: TCLBANKER abuses trusted victim-owned WhatsApp and Outlook sessions; PCPJack spreads laterally across exposed cloud services while evicting competing TeamPCP implants.
- Perimeter devices remain strategic initial access: PAN-OS and Ivanti EPMM exploitation underline continued targeting of externally reachable security and mobile-management infrastructure.
- Public PoC timing is compressing defender response windows: Dirty Frag moved from embargo break to public exploit quickly, creating immediate local privilege escalation risk before broad patch availability.

