# Daily Cybersecurity Intelligence Brief - 2026-05-11

## 1. 🧠 Executive Summary

- No fresh, high-confidence malware family with new public IOCs was observed in the exact May 10-11 accessible source window; the most relevant THN Threat Intelligence malware/configuration item remains PCPJack from May 7, still operationally relevant for cloud responders.
- Active exploitation priority remains perimeter/admin infrastructure: PAN-OS CVE-2026-0300, Ivanti EPMM CVE-2026-6973, and Linux Dirty Frag/CVE-2026-43284 are the highest-impact defensive actions.
- Targets: cloud/container environments, internet-facing security appliances, MDM infrastructure, Linux multi-user/container hosts, and managed file transfer operators.
- Why it matters: current campaigns favor credential harvesting, appliance compromise, and root-level local escalation over noisy destructive payloads.
- X.com review: direct X search was not reliably retrievable in this environment; indexed social mirrors and vendor reposts aligned with PCPJack, Dirty Frag, and PAN-OS exploitation but did not add unique IOCs.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-0300 - Palo Alto Networks PAN-OS User-ID Authentication Portal RCE

- Severity: Critical
- Affected systems: PA-Series and VM-Series firewalls with User-ID Authentication Portal/Captive Portal exposed to untrusted networks.
- Exploitation status: In-the-wild; Unit 42 tracks related activity as CL-STA-1132, a suspected state-sponsored cluster of unknown provenance.
- Technical root cause: Buffer overflow in PAN-OS User-ID Authentication Portal enabling unauthenticated RCE as root via crafted packets.
- Current status: Fixes expected starting May 13, 2026; CISA added the flaw to KEV on May 6 with a May 9 federal mitigation deadline.
- Post-exploitation: shellcode injection into an nginx worker process, crash/core log cleanup, Active Directory enumeration, EarthWorm and ReverseSocks5 deployment.

### CVE-2026-6973 - Ivanti Endpoint Manager Mobile RCE

- Severity: High
- Affected systems: Ivanti EPMM on-prem before 12.6.1.1, 12.7.0.1, and 12.8.0.1.
- Exploitation status: Limited in-the-wild exploitation; CISA KEV due date was May 10, 2026.
- Technical root cause: Improper input validation allowing a remotely authenticated administrative user to execute arbitrary code.
- Current status: Ivanti also patched CVE-2026-5786, CVE-2026-5787, CVE-2026-5788, and CVE-2026-7821; these are not reported exploited today.

### Dirty Frag - CVE-2026-43284 and CVE-2026-43500 Linux Kernel LPE Chain

- Severity: High
- Affected systems: Major Linux distributions, including Ubuntu 24.04.4, RHEL 10.1, CentOS Stream 10, AlmaLinux 10, openSUSE Tumbleweed, and Fedora 44.
- Exploitation status: Public PoC available; Microsoft reports limited in-the-wild LPE-like activity using `su` after SSH access.
- Technical root cause: Deterministic page-cache write bug class chaining xfrm-ESP and RxRPC page-cache write primitives; no race condition required.
- Impact: Local root privilege escalation and possible container escape in environments running arbitrary third-party workloads.

### CVE-2026-23918 - Apache HTTP Server HTTP/2 Double-Free

- Severity: High/Critical
- Affected systems: Apache HTTP Server 2.4.66 with `mod_http2`; fixed in 2.4.67.
- Exploitation status: PoC-level RCE demonstrated in lab; no confirmed active exploitation observed today.
- Technical root cause: Double-free in HTTP/2 stream cleanup path when HEADERS is followed by RST_STREAM before stream registration.
- Impact: Trivial unauthenticated DoS; RCE is practical under specific APR mmap allocator conditions, including Debian-derived systems and official httpd Docker images.

### CVE-2026-4670 / CVE-2026-5174 - MOVEit Automation

- Severity: Critical / High
- Affected systems: MOVEit Automation <= 2025.1.4, <= 2025.0.8, <= 2024.1.7.
- Exploitation status: No exploitation observed today.
- Technical root cause: Authentication bypass and improper input validation through service backend command port interfaces.
- Impact: Unauthorized access, administrative control, and data exposure; risk elevated because MOVEit has prior ransomware exploitation history.

## 3. 🦠 Malware & Campaign Analysis

### PCPJack Cloud Credential-Stealing Worm

- Malware name / family: PCPJack
- Threat actor: Unknown; SentinelLabs assesses possible former TeamPCP operator, rival, or actor directly modeling TeamPCP tradecraft.
- Initial access vector: Exposed Docker, Kubernetes, Redis, MongoDB, RayML, vulnerable web apps, and exploit paths for CVE-2025-29927, CVE-2025-55182, CVE-2026-1357, CVE-2025-9501, and CVE-2025-48703.
- Execution chain:
  1. `bootstrap.sh` creates `/var/lib/.spm/`.
  2. Checks public IP against operator blocklist.
  3. Removes TeamPCP/PCPCat processes, paths, services, and containers.
  4. Installs Python 3.6+, virtualenv, `requests`, `cryptography`, and `pyarrow`.
  5. Downloads `worm.py`, `parser.py`, `lateral.py`, `crypto_util.py`, `cloud_ranges.py`, and `cloud_scan.py`.
  6. Renames modules to `monitor.py`, `utils.py`, `_lat.py`, `_cu.py`, `_cr.py`, and `_csc.py`.
  7. Harvests secrets and propagates internally/externally.
- Persistence technique: Root installs systemd service such as `sys-monitor.service`/`spm-worker.service`; non-root uses crontabs; Redis compromise can rewrite cron to run `bootstrap.sh` every five minutes.
- C2 communication method: Telegram channel C2 and exfiltration; credentials encrypted with X25519 ECDH plus ChaCha20-Poly1305, falling back to plaintext if cryptography support is absent.
- Configuration notes: String constants are hex/XOR-obfuscated with an MD5-derived key; Sliver second-stage uses `/var/tmp/apt-daily-upgrade` and exfiltrates to `cdn[.]cloudfront-js[.]com:8443/u`.

### CL-STA-1132 PAN-OS Post-Exploitation

- Malware name / family: EarthWorm, ReverseSocks5, injected shellcode
- Threat actor: Suspected state-sponsored cluster, unknown provenance.
- Initial access vector: CVE-2026-0300 exploitation against internet-exposed PAN-OS User-ID Authentication Portal.
- Execution chain:
  1. Crafted packets trigger unauthenticated RCE.
  2. Shellcode is injected into an nginx worker process.
  3. Actor clears crash kernel messages, nginx crash entries, and core dumps.
  4. Actor performs AD enumeration.
  5. EarthWorm and ReverseSocks5 are staged on a second device for tunneling.
- Persistence technique: Not observed today in public reporting.
- C2 communication method: SOCKS/tunneling via EarthWorm and ReverseSocks5.

### Dirty Frag Follow-On Intrusion Pattern

- Malware name / family: Not observed today
- Threat actor: Unknown
- Initial access vector: External SSH access followed by local privilege escalation attempt.
- Execution chain:
  1. External connection obtains SSH shell.
  2. ELF artifact `./update` is staged/executed.
  3. Privilege escalation is triggered through `su`-related activity consistent with Dirty Frag or Copy Fail techniques.
  4. Actor modifies GLPI LDAP authentication file.
  5. Actor enumerates GLPI/system configuration, accesses PHP session files, deletes/wipes some sessions.
- Persistence technique: GLPI authentication modification observed; durable persistence not confirmed.
- C2 communication method: Not observed today.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "38.242.204.245",
    "38.242.237.196",
    "38.242.245.147",
    "83.171.249.231",
    "161.97.129.25",
    "161.97.135.154",
    "161.97.163.87",
    "161.97.186.175",
    "161.97.187.42",
    "193.187.129.143",
    "213.136.80.73",
    "67.206.213.86",
    "136.0.8.48",
    "146.70.100.69",
    "149.104.66.84"
  ],
  "domains": [
    "cdn.cloudfront-js.com",
    "lastpass-login-help.com",
    "spm-cdn-assets-dist-2026.s3.us-east-2.amazonaws.com"
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
    "fed52a4bbac7b5b6ae4f76cab3eadd67e79227e3"
  ],
  "urls": [
    "https://thehackernews.com/search/label/Threat%20Intelligence",
    "https://thehackernews.com/2026/05/pcpjack-credential-stealer-exploits-5.html",
    "https://www.sentinelone.com/labs/cloud-worm-evicts-teampcp-and-steals-credentials-at-scale/",
    "https://cdn.cloudfront-js.com:8443/u",
    "https://spm-cdn-assets-dist-2026.s3.us-east-2.amazonaws.com"
  ],
  "mutex": []
}
```

Host/file artifacts:

- `/var/lib/.spm/`
- `/etc/systemd/system/spm-worker.service`
- `/var/tmp/apt-daily-upgrade`
- `/tmp/.origin`
- `harvest.jsonl`
- `monitor.py`, `utils.py`, `_lat.py`, `_cu.py`, `_cr.py`, `_csc.py`
- `—-WebKitFormBoundaryx8jO2oVc6SWP3Sad`
- `6d4imqQ/s/GfQCVcybdcjfTe/PMYHtZN8ZGHnEXSbRo=`

## 5. 🔗 Technical Resources & Blogs

- [The Hacker News - Threat Intelligence](https://thehackernews.com/search/label/Threat%20Intelligence)
- [PCPJack credential stealer - The Hacker News](https://thehackernews.com/2026/05/pcpjack-credential-stealer-exploits-5.html)
- [PCPJack technical analysis - SentinelLabs](https://www.sentinelone.com/labs/cloud-worm-evicts-teampcp-and-steals-credentials-at-scale/)
- [PAN-OS CVE-2026-0300 exploitation - The Hacker News](https://thehackernews.com/2026/05/pan-os-rce-exploit-under-active-use.html)
- [Palo Alto CVE-2026-0300 advisory](https://security.paloaltonetworks.com/)
- [Dirty Frag analysis - The Hacker News](https://thehackernews.com/2026/05/linux-kernel-dirty-frag-lpe-exploit.html)
- [Dirty Frag PoC/writeup](https://github.com/v4bel/dirty-frag)
- [Apache HTTP Server CVE-2026-23918 - The Hacker News](https://thehackernews.com/2026/05/critical-apache-http2-flaw-cve-2026.html)
- [MOVEit Automation vulnerabilities - The Hacker News](https://thehackernews.com/2026/05/progress-patches-critical-moveit.html)
- [Ivanti EPMM CVE-2026-6973 - The Hacker News](https://thehackernews.com/2026/05/ivanti-epmm-cve-2026-6973-rce-under.html)

## 6. 🧪 Sandbox / Sample Links

- VirusTotal: SentinelLabs located the initial PCPJack script through VirusTotal hunting, but no stable public sample URL was observed in accessible details today.
- Any.Run / Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.
- GitHub PoC/sample: [Dirty Frag](https://github.com/v4bel/dirty-frag)

## 7. 🛡️ Detection & Mitigation

- PAN-OS: Restrict User-ID Authentication Portal to trusted zones or disable it; disable Response Pages on L3 interfaces receiving untrusted traffic; enable Advanced Threat Prevention Threat ID `510019` with content version `9097-10022`; hunt for nginx shellcode injection, crash/core cleanup, EarthWorm, ReverseSocks5, and AD enumeration from firewall-adjacent hosts.
- Ivanti EPMM: Upgrade to fixed versions immediately; rotate administrative credentials, especially if exposed to January EPMM exploitation; audit admin sessions and unexpected appliance-side command execution.
- Dirty Frag: Apply distro kernel fixes as released; until then blocklist `esp4`, `esp6`, and `rxrpc` where operationally acceptable; monitor unprivileged users spawning `su`, staging ELF files such as `./update`, and altering GLPI/PHP session or authentication files.
- Apache HTTP Server: Upgrade to 2.4.67; disable HTTP/2 where patching is delayed; prioritize Debian-derived and official httpd Docker deployments using threaded MPM and APR mmap allocator.
- MOVEit Automation: Upgrade to 2025.1.5, 2025.0.9, or 2024.1.8; there are no complete workarounds; restrict backend command port access and review administrative activity.
- PCPJack behavioral detections: Hunt for `/var/lib/.spm`, `monitor.py`, `bootstrap.sh`, `spm-worker.service`, repeated cloud metadata access, Telegram API traffic from servers, Common Crawl parquet retrieval, Docker socket abuse, Redis cron rewrite, RayML job submission on port 8265, and unauthenticated Kubernetes secret reads.
- YARA/Sigma: No vendor-published YARA/Sigma rules were observed today in accessible reporting; prioritize behavioral detections and network controls above.

## 8. 📊 Trends & Insights

- Cloud malware is moving from opportunistic miners to credential-first frameworks. PCPJack deliberately avoids miner payloads and focuses on secrets, API keys, crypto wallets, enterprise SaaS tokens, and resale/extortion value.
- Actor-on-actor displacement is now operationally relevant. PCPJack removes TeamPCP/PCPCat artifacts before installing its own tooling, complicating incident scoping and making “single actor” assumptions unsafe.
- Edge devices remain the espionage access layer of choice. PAN-OS exploitation shows continued interest in appliances with privileged network visibility and weaker endpoint telemetry.
- Linux LPE reliability is improving. Dirty Frag’s deterministic page-cache write chain reduces exploit fragility compared with race-condition LPEs and may accelerate post-SSH privilege escalation.
- Exact 24-hour malware novelty was limited in accessible public sources today; the actionable shift is exploitation and mitigation urgency rather than a new IOC-rich family.
