# Daily Cybersecurity Intelligence Brief - 2026-05-25

Reporting window: last 24 hours from 2026-05-25 Asia/Kolkata. Sources reviewed include The Hacker News Threat Intelligence section, Fox-IT/NCC Group, Socket, QiAnXin XLab, LiteSpeed/CVE enrichment, SecurityWeek/BleepingComputer, and accessible X.com-indexed reporting. Focus: active exploitation, zero-days, ransomware-enabling paths, major breaches, APT campaigns, and malware with configuration/C2 detail.

## 1. 🧠 Executive Summary

- Lazarus-linked RemotePE was disclosed in The Hacker News Threat Intelligence feed: a DPAPI-keyed, memory-only RAT chain targeting financial and cryptocurrency organizations.
- TrapDoor is an active cross-ecosystem supply-chain campaign across npm, PyPI, and Crates.io, stealing developer secrets, crypto wallets, cloud credentials, SSH keys, browser data, and GitHub tokens.
- Ghost CMS CVE-2026-26980 exploitation has compromised 700+ sites for ClickFix delivery, using stolen Admin API keys to inject JavaScript loaders into legitimate content.
- LiteSpeed User-End cPanel Plugin CVE-2026-48172 remains a maximum-severity active exploitation event enabling root-level script execution from cPanel context.
- X.com review surfaced Socket's May 24 TrapDoor alert as the most actionable malware/security item; no unique tweet-only IOC set was independently recoverable today.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-26980 - Ghost CMS Content API SQL Injection

- Severity: Critical, CVSS 9.4.
- Affected systems: Ghost CMS 3.24.0 through 6.19.0; fixed in 6.19.1.
- Exploitation status: In-the-wild; 700+ websites reported compromised.
- Technical root cause: SQL injection in Ghost Content API enabling unauthenticated database reads and Admin API key theft.
- Impact: Attackers use stolen Admin API keys to bulk-modify articles and inject malicious JavaScript loaders for ClickFix/FakeCaptcha malware delivery.

### CVE-2026-48172 - LiteSpeed User-End cPanel Plugin Privilege Escalation

- Severity: Critical, CVSS 10.0.
- Affected systems: LiteSpeed User-End cPanel Plugin before 2.4.5, with vendor guidance now recommending WHM Plugin 5.3.1.0 bundled with cPanel plugin 2.4.7 or later.
- Exploitation status: In-the-wild; acknowledged by LiteSpeed and reported by THN.
- Technical root cause: Incorrect privilege assignment in `lsws.redisAble` allowing cPanel users, including compromised accounts, to run arbitrary scripts as root.
- Impact: Root-level server compromise on affected hosting infrastructure.

### TrapDoor Cross-Registry Supply-Chain Malware

- Severity: High/Critical for developer endpoints and CI/CD environments.
- Affected systems: npm, PyPI, and Crates.io consumers installing malicious crypto, DeFi, AI, Solidity, Sui, or Move-themed packages.
- Exploitation status: Active package publication and updates began May 22, 2026, with reporting over May 24-25.
- Technical root cause: Abuse of package lifecycle execution paths: npm `postinstall`, Rust `build.rs`, and Python import-time execution of remote JavaScript.
- Impact: Developer secret theft, credential validation, SSH lateral movement, wallet theft, and persistence through developer tooling files and OS schedulers.

### RemotePE / DPAPILoader / RemotePELoader

- Severity: High for targeted financial and crypto organizations.
- Affected systems: Windows endpoints in high-value financial, DeFi, and cryptocurrency environments.
- Exploitation status: Used in intrusions; Fox-IT reports actor-in-the-loop delivery and active C2 retrieval during investigations.
- Technical root cause: Malware chain, not a software CVE; environmental keying with DPAPI plus memory-only final-stage execution.
- Impact: Long-term stealth access, plugin-based post-exploitation, command execution, file operations, process control, and potential financial theft.

## 3. 🦠 Malware & Campaign Analysis

### RemotePE Memory-Only RAT

- Malware name / family: DPAPILoader, RemotePELoader, RemotePE.
- Threat actor: Lazarus subgroup overlapping AppleJeus, Citrine Sleet, UNC4736, and Gleaming Pisces.
- Initial access vector: Social engineering; prior incident involved Telegram impersonation of a trading-company employee and fake Calendly/Picktime meeting domains.
- Execution chain:
  1. DPAPILoader DLL is installed as `C:\Windows\System32\Iassvc.dll` under a masqueraded "Internet Authentication Service" Windows service.
  2. DPAPILoader locates non-cabinet files under `C:\ProgramData\Microsoft\Windows\DeviceMetadataStore\en-US*.*`.
  3. Payload is decrypted with Windows DPAPI, XORed with `0x8D`, and reflectively loaded.
  4. RemotePELoader applies TartarusGate/Hell's Gate direct syscalls and patches `EtwEventWrite` to suppress ETW.
  5. RemotePELoader reads DPAPI+XOR encrypted config with up to three C2 URLs, proxy data, user-agent, reconnect, sleep-min/max, and sleep-until fields.
  6. Loader checks in over HTTP POST with Microsoft-like cookies, receives AES-GCM/Base64 PE payload in `odata.metadata`, and loads RemotePE in memory.
  7. RemotePE polls C2 for command batches and returns compressed, AES-GCM encrypted output through `armAuthorization`.
- Persistence technique: Windows service persistence via `Iassvc.dll`; sideloading variants include `sspicli.dll` and `wmiclnt.dll`.
- C2 communication method: HTTP POST, Cookie-based host/session metadata, AES-GCM encrypted message bodies, JSON keys that resemble Microsoft ecosystem fields.

### TrapDoor Crypto Stealer

- Malware name / family: TrapDoor; shared npm payload `trap-core.js`.
- Threat actor: Unknown.
- Initial access vector: Malicious packages masquerading as developer helpers, wallet auditors, crypto security tools, AI prompt/model utilities, Solidity tools, and Sui/Move build helpers.
- Execution chain:
  1. Developer installs or imports a malicious npm/PyPI/Crates.io package.
  2. npm packages execute `trap-core.js` through postinstall hooks.
  3. Rust crates execute malicious `build.rs`, search local wallet keystores, encrypt data with hardcoded XOR key `cargo-build-helper-2026`, and exfiltrate to GitHub Gists.
  4. PyPI packages auto-execute on import, fetch JavaScript from `ddjidd564.github[.]io`, and run it with `node -e`.
  5. Payload scans local files, environment variables, cloud credentials, browser data, crypto wallets, SSH material, AWS/GitHub tokens, and validates stolen tokens through APIs.
  6. Malware attempts SSH-based lateral movement and plants developer-environment prompt files to influence AI coding assistants.
- Persistence technique: `.cursorrules`, `CLAUDE.md`, Git hooks, shell hooks, systemd services, cron jobs, and SSH persistence.
- C2 communication method: GitHub Pages remote payload delivery; GitHub Gist exfiltration observed for Rust package data.

### Ghost CMS ClickFix Delivery Campaign

- Malware name / family: ClickFix/FakeCaptcha delivery chain; final payload includes modified Grape Electron client.
- Threat actor: At least two unidentified clusters per XLab.
- Initial access vector: Exploitation of Ghost CMS CVE-2026-26980 to steal Admin API keys.
- Execution chain:
  1. Attacker exploits SQL injection in Ghost Content API and reads Admin API key material.
  2. Stolen Admin API keys are used to bulk-modify Ghost articles.
  3. Injected JavaScript loader is appended to article pages.
  4. Loader retrieves second-stage traffic distribution script from `clo4shara[.]xyz/11z77u3.php`.
  5. Cloaking script fingerprints browser traits and filters scanners/crawlers.
  6. Selected visitors receive a fake CAPTCHA page instructing them to run a Base64 command through Windows Run.
  7. Command downloads a ZIP, runs a batch script, invokes PowerShell, downloads DLL or JavaScript payload, and launches payload via `rundll32.exe` or installer flow.
  8. Modified Grape Electron client persists and polls C2 every 30 seconds.
- Persistence technique: Electron `setLoginItemSettings` autostart in the modified Grape client.
- C2 communication method: POST polling to `web-telegram[.]ug` every 30 seconds for JavaScript or executable tasking.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [],
  "domains": [
    "aes-secure[.]net",
    "livedrivefiles[.]com",
    "azureglobalaccelerator[.]com",
    "msdeliverycontent[.]com",
    "akamaicloud[.]com",
    "intelcloudinsights[.]com",
    "devicelinkintel[.]com",
    "ddjidd564[.]github[.]io",
    "clo4shara[.]xyz",
    "web-telegram[.]ug"
  ],
  "hashes": [
    "4f6ae0110cf652264293df571d66955f7109e3424a070423b5e50edc3eb43874",
    "aa4a2d1215f864481994234f13ab485b95150161b4566c180419d93dda7ac039",
    "159471e1abc9adf6733af9d24781fbf27a776b81d182901c2e04e28f3fe2e6f3",
    "7a05188ab0129b0b4f38e2e7599c5c52149ce0131140db33feb251d926428d68",
    "37f5afb9ed3761e73feb95daceb7a1fdbb13c8b5fc1a2ba22e0ef7994c7920ef",
    "6b33d20196267b0d64bca815ca863558d26b17cee77caf62a6cce8eae555ac8d",
    "62e040a32aac2d2faa8d2bffa2cf7ab662228cebf9bb78eaa0a633c0b729d119",
    "710f15302859c7af1c1e25219d704841b3fdbc48f16a5a574d5ab6cf4f4842e8"
  ],
  "urls": [
    "https://clo4shara[.]xyz/11z77u3.php",
    "https://ddjidd564[.]github[.]io/defi-security-best-practices/"
  ],
  "mutex": [
    "554D5C1F-AABE-49E4-AB57-994D22ECED28"
  ]
}
```

Notes:
- RemotePE sample hashes are SHA256 values published by Fox-IT.
- No reliable attacker IP list was observed today in accessible primary reporting.
- Windows registry keys for these campaigns were not observed today.

## 5. 🔗 Technical Resources & Blogs

- The Hacker News Threat Intelligence: Lazarus RemotePE memory-only RAT - https://thehackernews.com/2026/05/lazarus-deploys-remotepe-memory-only.html
- Fox-IT/NCC Group: RemotePE technical analysis, YARA, samples, IOCs - https://blog.fox-it.com/2026/05/22/remotepe-the-lazarus-rat-that-lives-in-memory/
- Socket: TrapDoor crypto stealer supply-chain attack - https://socket.dev/blog/trapdoor-crypto-stealer-npm-pypi-crates
- Socket TrapDoor campaign tracker - https://socket.dev/supply-chain-attacks/trapdoor-crypto-stealer
- QiAnXin XLab: Ghost CMS mass compromise via CVE-2026-26980 - https://blog.xlab.qianxin.com/ghost-cms-mass-compromised-via-cve-2026-26980-now-fueling-clickfix-attacks/
- The Hacker News: Ghost CMS CVE-2026-26980 exploitation - https://thehackernews.com/2026/05/ghost-cms-cve-2026-26980-exploited-to.html
- The Hacker News: LiteSpeed cPanel Plugin CVE-2026-48172 - https://thehackernews.com/2026/05/litespeed-cpanel-plugin-cve-2026-48172.html
- LiteSpeed vendor advisory - https://blog.litespeedtech.com/2026/05/21/security-update-for-litespeed-cpanel-plugin/

## 6. 🧪 Sandbox / Sample Links

- VirusTotal: Fox-IT reported RemotePELoader and RemotePE were not present on VirusTotal before publication; no direct VT sample links were observed today.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.
- GitHub PoC: No reliable PoC repository was confirmed today for CVE-2026-26980 or CVE-2026-48172. Treat unauthenticated Ghost exploit claims as high-risk until independently validated.

## 7. 🛡️ Detection & Mitigation

### RemotePE

- Deploy Fox-IT YARA rules for `Lazarus_DPAPILoader_Hunting` and related RemotePE artifacts.
- Hunt for suspicious Windows services with `servicedll=%SystemRoot%\system32\Iassvc.dll`, service name `Ias`, and execution through `svchost.exe -k netsvcs -p`.
- Search for non-cabinet, DPAPI-encrypted blobs in `C:\ProgramData\Microsoft\Windows\DeviceMetadataStore\en-US*.*`, especially files over 50 KiB or config-sized files under 20 KiB.
- Detect ETW patching where `EtwEventWrite` is overwritten with `48 33 c0 c3`, and direct-syscall unhooking of `\KnownDlls` mappings.
- Network hunt for RemotePE cookie fields: `MSCC`, `MicrosoftApplicationsTelemetryDeviceId`, `MSFPC`, `HASH`, `LV`, `LU`, `MS0`, `MUID`, `at_check`, and `ai_session`, especially with `odata.metadata` and `armAuthorization` JSON keys.
- Block and retro-hunt RemotePE C2 domains; avoid IP-only blocking because Fox-IT observed Namecheap shared hosting.

### TrapDoor

- Remove and block known malicious package names listed in the Socket report across npm, PyPI, and Crates.io.
- Treat hosts that installed affected packages as compromised: rotate GitHub, AWS, cloud, npm, SSH, wallet, and API credentials.
- Hunt for `trap-core.js`, marker `P-2024-001`, GitHub account `ddjidd564`, domain `ddjidd564.github[.]io`, and XOR key `cargo-build-helper-2026`.
- Search for unauthorized `.cursorrules`, `CLAUDE.md`, Git hook, shell profile, systemd, cron, and SSH authorized-key changes.
- Enforce dependency installation through internal mirrors with package behavior scanning; block lifecycle scripts in CI unless explicitly required.

### Ghost CMS CVE-2026-26980

- Upgrade Ghost to 6.19.1 or later immediately.
- Rotate Admin API keys and assume keys generated before remediation may be compromised.
- Audit articles/themes for injected JavaScript, especially loader patterns containing `atob(`, `appendChild`, `btoa(a.origin)`, or references to `clo4shara[.]xyz`.
- Review Ghost Admin API logs for bulk article modification from unfamiliar IPs.
- Notify users who visited compromised pages during contamination windows, since the browser-side payload transitions to local Windows execution via ClickFix.

### LiteSpeed CVE-2026-48172

- Upgrade to LiteSpeed WHM Plugin 5.3.1.0 with cPanel plugin 2.4.7 or later.
- If patching is not immediate, uninstall the user-end plugin:
  ```bash
  /usr/local/lsws/admin/misc/lscmctl cpanelplugin --uninstall
  ```
- Hunt cPanel logs with:
  ```bash
  grep -rE "cpanel_jsonapi_func=redisAble" /var/cpanel/logs /usr/local/cpanel/logs/ 2>/dev/null
  ```
- Treat non-legitimate hits as possible root compromise and perform full host triage, not only plugin cleanup.

## 8. 📊 Trends & Insights

- Financially motivated and state-aligned actors are prioritizing developer workstations, package registries, and endpoint security products because these paths create high-trust execution.
- Malware tradecraft is converging with AI-assisted developer workflows: TrapDoor plants hidden instructions for Claude/Cursor contexts, while Ghost exploitation was discovered from an AI-found CMS flaw.
- RemotePE shows mature APT emphasis on environmental keying, memory-only final payloads, ETW suppression, and C2 that blends with Microsoft-like telemetry fields.
- ClickFix continues to scale through legitimate-site compromise rather than only malvertising, increasing user trust and bypassing perimeter reputation controls.
- Ransomware-specific new events were not observed today in high-confidence last-24-hour sources, but LiteSpeed root compromise and Ghost ClickFix delivery are ransomware-enabling access paths.
