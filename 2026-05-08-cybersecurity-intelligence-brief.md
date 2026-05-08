# Daily Cybersecurity Intelligence Brief - 2026-05-08

## 1. 🧠 Executive Summary

- **Ivanti EPMM CVE-2026-6973 is under limited zero-day exploitation**; exploitation requires an authenticated admin account, making credential rotation after prior EPMM compromises a priority.
- **PCPJack is a newly disclosed cloud credential-theft worm** targeting exposed Docker, Kubernetes, Redis, MongoDB, RayML, and vulnerable web apps while evicting TeamPCP artifacts.
- **ZiChatBot is a newly documented PyPI-delivered malware family** abusing Zulip REST APIs for C2 across Windows and Linux.
- **TCLBanker shows LATAM banking malware adopting worm-like spread** through hijacked WhatsApp Web sessions and Outlook COM automation.
- **X.com review:** public X searches for new malware and security leads in the last 24 hours were attempted; no independently retrievable, date-valid X-only IOC lead was observed today. Source-linked X share/reference links were reviewed where available.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-6973 - Ivanti Endpoint Manager Mobile RCE

- Severity: High, CVSS 7.2.
- Affected systems: Ivanti Endpoint Manager Mobile on-prem before 12.6.1.1, 12.7.0.1, and 12.8.0.1.
- Exploitation status: In-the-wild; Ivanti reports a very limited number of exploited customers.
- Technical root cause: Improper input validation enabling remote code execution by an authenticated administrator.
- Operational note: Risk is materially higher where admin credentials were not rotated after prior EPMM exploitation of CVE-2026-1281 and CVE-2026-1340.

### CVE-2026-5786 / CVE-2026-5787 / CVE-2026-5788 / CVE-2026-7821 - Ivanti EPMM Companion Fixes

- Severity: High.
- Affected systems: Ivanti EPMM on-prem; vendor states Ivanti Neurons for MDM, Ivanti EPM, Ivanti Sentry, and other Ivanti products are not affected.
- Exploitation status: Not observed today for these four companion CVEs.
- Technical root cause: Improper access control and improper certificate validation issues enabling admin access, Sentry host impersonation, arbitrary method invocation, or restricted device enrollment/information disclosure.

### PCPJack Cloud Worm Exploit Set

- Severity: High.
- Affected systems: Exposed Linux cloud workloads and services running Docker, Kubernetes, Redis, MongoDB, RayML, WordPress, CentOS Web Panel, React/Next.js, or related web applications.
- Exploitation status: In-the-wild malware framework observed by SentinelLABS.
- Technical root cause: Worm propagation through exposed management surfaces plus exploitation of known flaws including CVE-2025-29927, CVE-2025-55182, CVE-2026-1357, CVE-2025-9501, and CVE-2025-48703.

### ClickFix / Vidar Targeting Australian Infrastructure

- Severity: High.
- Affected systems: Windows endpoints browsing compromised WordPress sites; Australian organizations and infrastructure entities are explicitly targeted.
- Exploitation status: In-the-wild, per ASD ACSC.
- Technical root cause: Social engineering-driven execution rather than a CVE; compromised WordPress infrastructure injects fake verification prompts that copy obfuscated PowerShell to the user clipboard.

## 3. 🦠 Malware & Campaign Analysis

### PCPJack

- Malware name / family: PCPJack cloud credential-theft framework; includes Python worm modules and Sliver ELF beacons.
- Threat actor: Unknown; SentinelLABS assesses possible familiarity with, or former membership in, TeamPCP due to explicit TeamPCP eviction logic.
- Initial access vector: Internet-exposed cloud services, vulnerable web apps, and laterally reachable Docker/Kubernetes/Redis/RayML/MongoDB/SSH surfaces.
- Execution chain:
  1. `bootstrap.sh` creates `/var/lib/.spm/`, installs Python dependencies, and pulls six modules from attacker-controlled S3.
  2. Script removes TeamPCP/PCPCat artifacts, deploys `monitor.py`, and establishes persistence.
  3. `monitor.py` harvests `.env`, config files, environment variables, SSH keys, AWS IMDS credentials, Kubernetes service account tokens, Docker secrets, crypto wallets, and git history secrets.
  4. `lateral.py` propagates through Kubernetes APIs, Docker sockets, Redis cron rewrite, RayML jobs, MongoDB scraping, and SSH.
  5. `cloud_scan.py` expands externally using cloud IP ranges and Common Crawl parquet-derived targets.
  6. Separate `check.sh` deploys garbled Sliver binaries as `/var/tmp/apt-daily-upgrade`.
- Persistence technique: systemd `sys-monitor.service` when root; otherwise crontabs every five minutes. Redis cron rewrite is used during lateral movement.
- C2 communication method: Telegram channels for the Python worm; HTTPS exfiltration to `cdn[.]cloudfront-js[.]com:8443/u` for the Sliver/credential harvester toolset.

### ZiChatBot PyPI Supply-Chain Campaign

- Malware name / family: ZiChatBot.
- Threat actor: Unknown; Kaspersky notes 64% dropper similarity to an OceanLotus/APT32 dropper, but attribution remains tentative.
- Initial access vector: Malicious PyPI wheel packages `uuid32-utils`, `colorinal`, and `termncolor`.
- Execution chain:
  1. Developer installs a malicious wheel package from PyPI.
  2. Windows package extracts `terminate.dll`; Linux package extracts `terminate.so`.
  3. Import/load event executes the dropper.
  4. Windows dropper writes autorun registry persistence and deletes itself.
  5. Linux dropper writes payload under `/tmp/obsHub/obs-check-update` and configures crontab.
  6. ZiChatBot receives shellcode from C2 and signals successful command execution with a heart response.
- Persistence technique: Windows autorun registry key; Linux crontab.
- C2 communication method: Zulip public team-chat REST APIs instead of dedicated attacker C2.

### TCLBanker / REF3076

- Malware name / family: TCLBanker, major evolution of Maverick/Sorvepotel-style Brazilian banking malware.
- Threat actor: REF3076.
- Initial access vector: Trojanized MSI installer masquerading as Logitech AI Prompt Builder; phishing infrastructure is still being expanded.
- Execution chain:
  1. Victim executes trojanized MSI/ZIP delivery package.
  2. Malware loads via DLL side-loading in the legitimate Logitech application context.
  3. Environment-gated decryption and anti-debug watchdog block common tools including x64dbg, IDA, dnSpy, Frida, ProcessHacker, Ghidra, and de4dot.
  4. Browser URL monitor checks for 59 banking, fintech, and cryptocurrency targets.
  5. WebSocket C2 session enables live screen streaming, keylogging, clipboard hijacking, shell execution, file operations, and remote mouse/keyboard control.
  6. WPF overlays present fake credential, PIN, phone-number, support, and Windows Update screens.
  7. WhatsApp Web IndexedDB/session data and Outlook COM automation are abused to send phishing messages from the victim’s trusted accounts.
- Persistence technique: Scheduled task behavior documented by Elastic; local process/watchdog logic keeps banker components active.
- C2 communication method: Cloudflare Workers and HTTPS/WebSocket endpoints, including `/api/campaign`, `/api/control`, and `/api/progress`.

### Beagle Backdoor via Fake Claude Site

- Malware name / family: Beagle backdoor; DonutLoader first stage; related chains show PlugX/AdaptixC2-like sideloading patterns.
- Threat actor: Unknown; Sophos notes possible PlugX operator experimentation or imitation.
- Initial access vector: Fake Claude site `claude-pro[.]com` serving `Claude-Pro-windows-x64.zip` with a malicious MSI.
- Execution chain:
  1. Victim downloads fake Claude-Pro Relay installer.
  2. Startup folder receives `NOVupdate.exe`, `NOVupdate.exe.dat`, and `avk.dll`.
  3. Signed G DATA updater `NOVupdate.exe` side-loads malicious `avk.dll`.
  4. DLL decrypts `NOVupdate.exe.dat` and executes DonutLoader in memory.
  5. DonutLoader deploys Beagle, which supports command execution, upload/download, file management, and uninstall.
- Persistence technique: Startup folder placement of `NOVupdate.exe`, `NOVupdate.exe.dat`, and `avk.dll`.
- C2 communication method: TCP/443 and UDP/8080 to `license[.]claude-pro[.]com`, AES-protected JSON messages with types 10 and 11.

### Vidar Stealer via ClickFix

- Malware name / family: Vidar Stealer.
- Threat actor: Unknown.
- Initial access vector: Compromised WordPress websites serving fake Cloudflare/CAPTCHA verification prompts.
- Execution chain:
  1. Victim visits compromised WordPress site.
  2. Injected payload domain loads external JavaScript and overwrites the page with a fake verification prompt.
  3. JavaScript copies an obfuscated PowerShell command to clipboard.
  4. User is instructed to execute the command with administrative privileges.
  5. PowerShell retrieves and launches Vidar from the same payload domain.
  6. Vidar deletes its initial executable and operates primarily in memory.
  7. Malware retrieves C2 via dead-drop URLs on Telegram bots and Steam profiles, then exfiltrates over HTTP/S POST.
- Persistence technique: Not observed today; ACSC highlights self-deletion and in-memory operation rather than durable persistence.
- C2 communication method: Dead-drop resolver to obtain C2, followed by web POST exfiltration.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "8.217.190[.]58",
    "38.242.204[.]245",
    "38.242.237[.]196",
    "38.242.245[.]147",
    "83.171.249[.]231",
    "161.97.129[.]25",
    "161.97.135[.]154",
    "161.97.163[.]87",
    "161.97.186[.]175",
    "161.97.187[.]42",
    "193.187.129[.]143",
    "213.136.80[.]73",
    "191.96.224[.]96",
    "192.252.186[.]62"
  ],
  "domains": [
    "cdn[.]cloudfront-js[.]com",
    "lastpass-login-help[.]com",
    "spm-cdn-assets-dist-2026[.]s3[.]us-east-2[.]amazonaws[.]com",
    "claude-pro[.]com",
    "license[.]claude-pro[.]com",
    "licence[.]claude-pro[.]com",
    "gouvvbo[.]top",
    "update-trellix[.]com",
    "update-crowdstrike[.]com",
    "update-sentinelone[.]com",
    "campanha1-api.ef971a42[.]workers.dev",
    "documents.ef971a42[.]workers.dev",
    "mxtestacionamentos[.]com",
    "arquivos-omie[.]com",
    "documentos-online[.]com",
    "afonsoferragista[.]com",
    "doccompartilhe[.]com",
    "recebamais[.]com"
  ],
  "hashes": [
    "005587975a483876c1fa26b64b418931019be38f",
    "01cebc48016395e284ac76afc1816f143ee3e7b6",
    "0b86434ca5145636d745222f7e49c903ce6ef538",
    "2cd2c5268e41cdece1b0506bcda3b9eba2998119",
    "2fab324eb0d927846c8744dc0e217beea65138e0",
    "339cbf61c80f757085c5afb7304d69f323bdf87a",
    "6060da100b5cd587131a1c11a20d6e0108604744",
    "848ef1f638807826586802428a7ebafdc710915c",
    "9c7ab48c9fdbbeecdad8433529bdab38584f0e25",
    "a20a9924d92c2b06d82b79c0fe87451c650cabec",
    "c2dd8051d89c4efa71bd67d2df7d9b4bc3e67810",
    "fed52a4bbac7b5b6ae4f76cab3eadd67e79227e3",
    "701d51b7be8b034c860bf97847bd59a87dca8481c4625328813746964995b626",
    "8a174aa70a4396547045aef6c69eb0259bae1706880f4375af71085eeb537059",
    "668f932433a24bbae89d60b24eee4a24808fc741f62c5a3043bb7c9152342f40",
    "63beb7372098c03baab77e0dfc8e5dca5e0a7420f382708a4df79bed2d900394"
  ],
  "urls": [
    "hxxps://cdn[.]cloudfront-js[.]com:8443/u",
    "hxxps://spm-cdn-assets-dist-2026[.]s3[.]us-east-2[.]amazonaws[.]com",
    "hxxps://campanha1-api.ef971a42[.]workers.dev/api/campaign",
    "hxxps://campanha1-api.ef971a42[.]workers.dev/api/control",
    "hxxps://campanha1-api.ef971a42[.]workers.dev/api/progress",
    "hxxps://claude-pro[.]com",
    "hxxps://license[.]claude-pro[.]com"
  ],
  "mutex": []
}
```

Host artifacts:

- `/var/lib/.spm/`
- `/var/lib/.spm/monitor.py`
- `/var/lib/.spm/_cr/ranges.json`
- `/var/tmp/apt-daily-upgrade`
- `sys-monitor.service`
- `worm.py`, `monitor.py`, `parser.py`, `utils.py`, `lateral.py`, `_lat.py`, `crypto_util.py`, `_cu.py`, `cloud_ranges.py`, `_cr.py`, `cloud_scan.py`, `_csc.py`
- `/tmp/obsHub/obs-check-update`
- `terminate.dll`
- `terminate.so`
- `NOVupdate.exe`
- `NOVupdate.exe.dat`
- `avk.dll`
- `%TEMP%\\oc<guid>.ps1`

## 5. 🔗 Technical Resources & Blogs

- The Hacker News: [Threat Intelligence section](https://thehackernews.com/search/label/Threat%20Intelligence)
- The Hacker News: [Ivanti EPMM CVE-2026-6973 RCE Under Active Exploitation](https://thehackernews.com/2026/05/ivanti-epmm-cve-2026-6973-rce-under.html)
- Ivanti: [May 2026 Security Advisory - EPMM Multiple CVEs](https://hub.ivanti.com/s/article/May-2026-Security-Advisory-Ivanti-Endpoint-Manager-Mobile-EPMM-Multiple-CVEs?language=en_US)
- CISA KEV: [Known Exploited Vulnerabilities Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- The Hacker News: [PCPJack Credential Stealer Exploits 5 CVEs](https://thehackernews.com/2026/05/pcpjack-credential-stealer-exploits-5.html)
- SentinelLABS: [PCPJack - Cloud Worm Evicts TeamPCP and Steals Credentials at Scale](https://www.sentinelone.com/labs/cloud-worm-evicts-teampcp-and-steals-credentials-at-scale/)
- The Hacker News: [PyPI Packages Deliver ZiChatBot Malware via Zulip APIs](https://thehackernews.com/2026/05/pypi-packages-deliver-zichatbot-malware.html)
- Kaspersky Securelist: [OceanLotus suspected of distributing ZiChatBot malware via wheel packages in PyPI](https://securelist.com/oceanlotus-suspected-pypi-zichatbot-campaign/119603/)
- BleepingComputer: [New TCLBanker malware self-spreads over WhatsApp and Outlook](https://www.bleepingcomputer.com/news/security/new-tclbanker-malware-self-spreads-over-whatsapp-and-outlook/)
- Elastic Security Labs: [TCLBANKER - Brazilian Banking Trojan Spreading via WhatsApp and Outlook](https://www.elastic.co/security-labs/tclbanker-brazilian-banking-trojan)
- BleepingComputer: [Fake Claude AI website delivers new Beagle Windows malware](https://www.bleepingcomputer.com/news/security/fake-claude-ai-website-delivers-new-beagle-windows-malware/)
- Sophos: [Donuts and Beagles - Fake Claude site spreads backdoor](https://www.sophos.com/en-us/blog/donuts-and-beagles-fake-claude-site-spreads-backdoor)
- BleepingComputer: [Australia warns of ClickFix attacks pushing Vidar Stealer malware](https://www.bleepingcomputer.com/news/security/australia-warns-of-clickfix-attacks-pushing-vidar-stealer-malware/)
- ASD ACSC: [ClickFix distributing Vidar Stealer via WordPress targeting Australian infrastructure](https://www.cyber.gov.au/about-us/view-all-content/alerts-and-advisories/clickfix-distributing-vidar-stealer-via-wordpress-targeting-australian-infrastructure)

## 6. 🧪 Sandbox / Sample Links

- VirusTotal, PCPJack `bootstrap.sh`: `https://www.virustotal.com/gui/file/a20a9924d92c2b06d82b79c0fe87451c650cabec`
- VirusTotal, PCPJack `check.sh`: `https://www.virustotal.com/gui/file/339cbf61c80f757085c5afb7304d69f323bdf87a`
- VirusTotal, Beagle/Sophos samples: Sophos links February, March, and April sample pivots directly from the technical writeup.
- Sophos GitHub IOCs: linked from the Sophos report; direct repository path was not fully exposed in the crawled page.
- ASD ACSC ClickFix/Vidar IOC CSV: attached as `Clickfix-Vidar-IOC.csv` on the ACSC advisory page.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.

## 7. 🛡️ Detection & Mitigation

### Ivanti EPMM

- Upgrade on-prem EPMM to 12.6.1.1, 12.7.0.1, or 12.8.0.1 immediately.
- Rotate EPMM administrator credentials, especially in environments exposed to CVE-2026-1281/CVE-2026-1340 exploitation earlier this year.
- Hunt for anomalous admin-authenticated actions followed by appliance command execution, new device enrollment anomalies, or unexpected Sentry certificate issuance.
- Federal remediation deadline for CVE-2026-6973 in CISA KEV is 2026-05-10.

### PCPJack

- Hunt for `/var/lib/.spm/`, `sys-monitor.service`, five-minute crontabs invoking `monitor.py`, and `/var/tmp/apt-daily-upgrade`.
- Alert on cloud workloads downloading from `spm-cdn-assets-dist-2026[.]s3[.]us-east-2[.]amazonaws[.]com` or posting to `cdn[.]cloudfront-js[.]com:8443/u`.
- Enforce IMDSv2, remove unauthenticated Docker API exposure on 2375/2376, restrict Kubernetes service account tokens/RBAC, and block Redis cron rewrites.
- Patch the exploited web stack: Next.js middleware auth bypass, React/Next.js Server Actions deserialization, WPVivid Backup upload, W3 Total Cache PHP injection, and CentOS Web Panel shell injection.

### ZiChatBot

- Remove `uuid32-utils`, `colorinal`, and `termncolor` from developer environments; rebuild affected virtual environments/containers from trusted lockfiles.
- Hunt for `terminate.dll`, `terminate.so`, `/tmp/obsHub/obs-check-update`, new crontab entries, and unexpected autorun registry entries after Python package install.
- Monitor for Zulip REST API traffic from hosts that do not normally use Zulip.
- Treat package install hosts as potentially exposed developer-secret systems; rotate tokens present in `.env`, shell history, package config, and CI credentials.

### TCLBanker

- Elastic YARA: `Windows.Trojan.TCLBanker` is available from Elastic’s GitHub-linked rule in the report.
- Hunt for Logitech AI Prompt Builder MSI anomalies, `screen_retriever_plugin.dll`, WPF overlay behavior, hidden Chromium instances accessing WhatsApp Web IndexedDB, and Outlook COM email bursts.
- Block known Cloudflare Worker C2 domains and monitor `/api/campaign`, `/api/control`, `/api/progress` endpoints.
- Detect process-kill behavior against Task Manager and anti-analysis process enumeration.

### Beagle

- Hunt Startup folders for `NOVupdate.exe`, `NOVupdate.exe.dat`, and `avk.dll`.
- Monitor for connections to `claude-pro[.]com`, `license[.]claude-pro[.]com`, and `8.217.190[.]58` over TCP/443 or UDP/8080.
- Detect signed G DATA updater execution from user-writable or Startup paths with adjacent unsigned DLLs.
- Sophos detections include `ATK/DonutLdr-B`, `Troj/Loader-OT`, `Troj/Beagldr-A`, `Troj/Loader-OX`, `ATK/AdaptixC2-R`, and `Troj/LnkRun-EY`.

### Vidar / ClickFix

- Block JavaScript clipboard writes from untrusted websites where feasible.
- Restrict PowerShell outbound network access and enforce script signing.
- Detect user-driven PowerShell launched shortly after browser interaction with fake CAPTCHA/Cloudflare prompts.
- Hunt Vidar dead-drop resolver behavior through Telegram bots and Steam profile retrieval followed by HTTP/S POST exfiltration.
- For WordPress administrators, patch core/plugins/themes and remove unused or unsupported plugins/themes.

## 8. 📊 Trends & Insights

- **Commodity malware is converging with worm behavior:** PCPJack spreads across cloud services, TCLBanker self-propagates through trusted messaging/email accounts, and Vidar delivery abuses user-assisted ClickFix flows.
- **Attackers continue to exploit trust in legitimate platforms:** Zulip, Telegram, Steam profiles, Cloudflare Workers, S3, WhatsApp, Outlook, and signed updater binaries are all part of today’s observed chains.
- **Developer and cloud secrets remain priority targets:** PCPJack and ZiChatBot both focus on environments likely to hold API keys, CI secrets, SSH keys, package credentials, and cloud metadata tokens.
- **AI-brand impersonation is a live lure class:** Beagle’s fake Claude delivery shows attackers adapting malware distribution to current AI adoption patterns.
- **Ransomware-specific new activity:** Not observed today in the reviewed 24-48 hour window beyond extortion-adjacent credential theft and previously covered MuddyWater/Chaos activity from 2026-05-06.
