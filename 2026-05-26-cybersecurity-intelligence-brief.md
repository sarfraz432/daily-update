# Daily Cybersecurity Intelligence Brief - 2026-05-26

Reporting window: last 24 hours ending 2026-05-26 Asia/Kolkata. Sources reviewed include The Hacker News Threat Intelligence section, Socket, QiAnXin XLab, Fox-IT/NCC Group, THN active exploitation reporting, and accessible X.com-indexed security posts. Focus: active exploitation, zero-days, ransomware-enabling access, APT campaigns, major breaches, and malware with configuration/C2 detail.

## 1. 🧠 Executive Summary

- TrapDoor remains the highest-impact malware item observed today: a coordinated npm/PyPI/Crates.io credential stealer targeting crypto, DeFi, AI, and security developers.
- Ghost CMS CVE-2026-26980 is being exploited in the wild to poison 700+ legitimate sites with ClickFix JavaScript loaders and Windows payload delivery.
- The mandatory THN Threat Intelligence review surfaced Lazarus RemotePE: a DPAPI-keyed, memory-only RAT chain with encrypted configuration, actor-in-the-loop C2, ETW suppression, and published YARA/IOCs.
- Microsoft Defender CVE-2026-41091 and CVE-2026-45498 remain active-exploitation priorities; CVE-2026-41091 enables local SYSTEM privilege escalation.
- X.com review was partially limited by rendering/search availability; accessible X-indexed results showed active Ghost CMS discussion, and THN-linked X posts from GitHub/Nx confirmed developer-tooling supply-chain impact, but no unique tweet-only IOC set was independently recovered.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-26980 - Ghost CMS Content API SQL Injection

- Severity: Critical, CVSS 9.4.
- Affected systems: Ghost CMS 3.24.0 through 6.19.0; fixed in 6.19.1.
- Exploitation status: In-the-wild; QiAnXin XLab reports 700+ compromised sites.
- Technical root cause: Unauthenticated SQL injection in Ghost Content API enabling arbitrary database reads and Admin API key theft.
- Operational impact: Stolen Admin API keys are used to bulk-modify article content and inject JavaScript loaders for fake CAPTCHA / ClickFix malware delivery.

### TrapDoor Cross-Registry Credential Stealer

- Severity: High/Critical for developer endpoints, CI/CD systems, and crypto environments.
- Affected systems: Developers installing malicious npm, PyPI, or Crates.io packages themed around crypto, DeFi, AI, Solidity, Sui, Move, and environment auditing.
- Exploitation status: Active campaign; earliest observed package upload was May 22, 2026, with 34+ packages and 384+ versions/artifacts reported.
- Technical root cause: Abuse of package lifecycle execution: npm `postinstall`, Rust `build.rs`, and Python import-time remote JavaScript execution.
- Operational impact: Theft of SSH keys, cloud credentials, GitHub tokens, browser data, environment variables, Sui/Solana/Aptos wallet data, and API keys.

### CVE-2026-41091 / CVE-2026-45498 - Microsoft Defender

- Severity: High for CVE-2026-41091; Medium for CVE-2026-45498.
- Affected systems: Microsoft Defender Antimalware Platform before 1.1.26040.8 and Defender platform before 4.18.26040.7.
- Exploitation status: In-the-wild; Microsoft and CISA KEV-listed.
- Technical root cause: CVE-2026-41091 is improper link resolution before file access, enabling local privilege escalation. CVE-2026-45498 is a Defender denial-of-service flaw.
- Operational impact: Local attacker can gain SYSTEM via CVE-2026-41091; CVE-2026-45498 can disrupt endpoint protection availability.

### CVE-2026-45584 - Microsoft Defender Heap-Based Buffer Overflow

- Severity: High, CVSS 8.1.
- Affected systems: Microsoft Defender Antimalware Platform before 1.1.26040.8.
- Exploitation status: No in-the-wild exploitation observed today.
- Technical root cause: Heap-based buffer overflow allowing unauthenticated remote code execution.
- Operational impact: High-risk adjacent patch priority because it affects security tooling, but current reporting does not confirm exploitation.

## 3. 🦠 Malware & Campaign Analysis

### TrapDoor Crypto / Developer Credential Stealer

- Malware name / family: TrapDoor; shared npm payload `trap-core.js`.
- Threat actor: Unknown.
- Initial access vector: Malicious packages posing as security scanners, crypto wallet auditors, AI/model utilities, DeFi tooling, Solidity helpers, and Sui/Move build helpers.
- Execution chain:
  1. Victim installs/imports malicious npm, PyPI, or Crates.io package.
  2. npm packages execute `trap-core.js` through `postinstall`.
  3. Rust crates execute malicious `build.rs`, search local keystores, XOR-encrypt data with `cargo-build-helper-2026`, and exfiltrate to GitHub Gists.
  4. PyPI packages auto-run on import, fetch JavaScript from `ddjidd564.github[.]io`, and execute it with `node -e`.
  5. Payload scans developer secrets, validates AWS/GitHub credentials via API calls, and attempts SSH-based lateral movement.
  6. The campaign plants `.cursorrules` and `CLAUDE.md` files with hidden instructions to influence AI coding assistants.
- Persistence technique: `.cursorrules`, `CLAUDE.md`, Git hooks, shell hooks, systemd services, cron jobs, SSH propagation.
- C2 communication method: GitHub Pages payload/config delivery; GitHub Gist exfiltration for Rust keystore theft; attacker GitHub account `ddjidd564`.

### Ghost CMS ClickFix Delivery Campaign

- Malware name / family: ClickFix / fake CAPTCHA loader chain; final payload includes a modified Grape Electron client.
- Threat actor: At least two unidentified clusters.
- Initial access vector: Exploitation of Ghost CMS CVE-2026-26980 to steal Admin API keys.
- Execution chain:
  1. Attacker reads Admin API key material via Ghost Content API SQL injection.
  2. Admin API is used to bulk-edit legitimate articles.
  3. Injected JavaScript loader retrieves a traffic-distribution script from `clo4shara[.]xyz/11z77u3.php` or related infrastructure.
  4. Cloaking layer fingerprints browser/user attributes and filters scanners.
  5. Selected visitors see fake CAPTCHA instructions that coerce execution of a Base64 command through Windows Run.
  6. Command downloads ZIP/script content, invokes PowerShell, retrieves DLL/JavaScript payloads, and launches `rundll32.exe` or an Inno Setup installer.
  7. Modified Grape Electron application installs under `%appdata%\local\SuperMaxionQuickMaxlite` and polls for tasking.
- Persistence technique: Electron `setLoginItemSettings` autostart.
- C2 communication method: HTTP POST polling every 30 seconds to `web-telegram[.]ug` for JavaScript or executable tasking.

### Lazarus RemotePE Memory-Only RAT

- Malware name / family: DPAPILoader, RemotePELoader, RemotePE.
- Threat actor: Lazarus subgroup overlapping AppleJeus, Citrine Sleet, UNC4736, and Gleaming Pisces.
- Initial access vector: Social engineering against financial/crypto personnel; prior case used Telegram impersonation and fake meeting domains.
- Execution chain:
  1. DPAPILoader DLL is installed as `C:\Windows\System32\Iassvc.dll` under a masqueraded "Internet Authentication Service" service.
  2. Loader searches `C:\ProgramData\Microsoft\Windows\DeviceMetadataStore\en-US*.*` for non-Cabinet payload/config blobs.
  3. Payload/config are decrypted with DPAPI and XORed with `0x8D`.
  4. RemotePELoader applies TartarusGate/Hell's Gate direct syscalls and patches `EtwEventWrite` with `48 33 c0 c3`.
  5. Loader reads configuration containing up to three C2 URLs, proxy fields, user-agent, reconnect interval, sleep-min/max, and sleep-until epoch.
  6. Loader checks in over HTTP POST with Microsoft-like cookie fields and receives AES-GCM/Base64 PE payload in `odata.metadata`.
  7. RemotePE runs entirely in memory, processes command batches, and returns compressed AES-GCM encrypted output in `armAuthorization`.
- Persistence technique: Windows service persistence via `Iassvc.dll`; alternate samples include `sspicli.dll` sideloading and `wmiclnt.dll`.
- C2 communication method: HTTP POST, cookie/session metadata, AES-GCM encrypted payloads, Microsoft-like JSON/cookie naming.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "144.31.236[.]66"
  ],
  "domains": [
    "ddjidd564[.]github[.]io",
    "clo4shara[.]xyz",
    "com-apps[.]cc",
    "cloud-verification[.]com",
    "jalwat[.]com",
    "taketwolabs[.]com",
    "web-telegram[.]ug",
    "staticcloudflare[.]pro",
    "script-dev[.]digital",
    "cdnupdatenews[.]top",
    "wl[.]gl",
    "amazonbusketss-535659318049-us-west-1-an.s3.us-west-1.amazonaws[.]com",
    "aes-secure[.]net",
    "livedrivefiles[.]com",
    "azureglobalaccelerator[.]com",
    "msdeliverycontent[.]com",
    "akamaicloud[.]com",
    "intelcloudinsights[.]com",
    "devicelinkintel[.]com"
  ],
  "hashes": [
    "5659292833ec421da11ebde005d9c9a8",
    "d30cc10d54ebc967c8538ff74f442eee",
    "ec5dfee13abf94e08d0f94e90b527db0",
    "18a7251ddde77ed24bc54700d84d9be1",
    "9434fe686801742ef7d6da248fb0b900dc32208a",
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
    "https://ddjidd564[.]github[.]io/defi-security-best-practices/",
    "https://clo4shara[.]xyz/11z77u3.php",
    "https://com-apps[.]cc/11z77u3.php",
    "https://staticcloudflare[.]pro/api/css.js",
    "https://script-dev[.]digital/api/css.js",
    "https://cdnupdatenews[.]top/dl?fid=38",
    "https://wl[.]gl/sup.exe",
    "https://amazonbusketss-535659318049-us-west-1-an.s3.us-west-1.amazonaws[.]com/UtilifySetup.exe"
  ],
  "mutex": [
    "554D5C1F-AABE-49E4-AB57-994D22ECED28"
  ]
}
```

Notes:
- RemotePE hashes are SHA256 values from Fox-IT. Ghost CMS payload hashes include MD5 and SHA1 values published by XLab.
- No new ransomware leak-site-only IOCs were validated from primary sources today.
- Registry keys for TrapDoor and Ghost ClickFix payloads were not observed today.

## 5. 🔗 Technical Resources & Blogs

- The Hacker News Threat Intelligence: Lazarus RemotePE memory-only RAT - https://thehackernews.com/2026/05/lazarus-deploys-remotepe-memory-only.html
- Fox-IT/NCC Group: RemotePE technical analysis, YARA, samples, IOCs - https://blog.fox-it.com/2026/05/22/remotepe-the-lazarus-rat-that-lives-in-memory/
- The Hacker News: TrapDoor cross-registry credential stealer - https://thehackernews.com/2026/05/trapdoor-supply-chain-attack-spreads.html
- Socket: TrapDoor crypto stealer supply-chain attack - https://socket.dev/blog/trapdoor-crypto-stealer-npm-pypi-crates
- Socket TrapDoor campaign tracker - https://socket.dev/supply-chain-attacks/trapdoor-crypto-stealer
- The Hacker News: Ghost CMS CVE-2026-26980 exploitation - https://thehackernews.com/2026/05/ghost-cms-cve-2026-26980-exploited-to.html
- QiAnXin XLab: Ghost CMS mass compromise via CVE-2026-26980 - https://blog.xlab.qianxin.com/ghost-cms-mass-compromised-via-cve-2026-26980-now-fueling-clickfix-attacks/
- Microsoft Defender exploitation reporting - https://thehackernews.com/2026/05/microsoft-warns-of-two-actively.html
- CISA KEV catalog - https://www.cisa.gov/known-exploited-vulnerabilities-catalog

## 6. 🧪 Sandbox / Sample Links

- VirusTotal: XLab notes `UtilifySetup.exe` had 0 detections at analysis time, but no direct VirusTotal URL was published in accessible reporting.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.
- GitHub PoC: No reliable public PoC repository was confirmed today for CVE-2026-26980 or the Defender CVEs.

## 7. 🛡️ Detection & Mitigation

### TrapDoor

- Block the known malicious npm, PyPI, and Crates.io package names from Socket's report at package proxy/mirror level.
- Hunt for `trap-core.js`, campaign marker `P-2024-001`, GitHub account `ddjidd564`, `ddjidd564.github[.]io`, and XOR key `cargo-build-helper-2026`.
- Search endpoints and repos for unauthorized `.cursorrules`, `CLAUDE.md`, Git hook, shell profile, systemd, cron, and SSH authorized-key changes.
- Treat affected developer workstations as credential-compromised: rotate GitHub, AWS/cloud, npm/PyPI/crates, SSH, wallet, and CI/CD secrets.
- Disable lifecycle scripts in CI unless explicitly required; enforce dependency installs through a scanning internal registry.

### Ghost CMS CVE-2026-26980

- Upgrade Ghost to 6.19.1 or later immediately.
- Rotate Admin API keys, Content API keys, administrator passwords, and sessions.
- Audit article bodies, themes, and Code Injection settings for `sj.ssc/ipa/`, `ghost_once_footer_`, `atob(` plus `appendChild`, `btoa(a.origin)`, and domains listed in the IOC block.
- Review Ghost Admin API logs for abnormal or bulk `PUT /ghost/api/admin/posts/:id/` requests.
- Notify users who visited contaminated pages during exposure windows; the infection path moves from browser JavaScript to local Windows execution through ClickFix.

### RemotePE

- Deploy Fox-IT YARA rules: `Lazarus_DPAPILoader_Hunting`, `Lazarus_RemotePE_C2_strings`, `Lazarus_RemotePE_class_strings`, and `Lazarus_RemotePE_DPAPI_Encrypted_config`.
- Hunt for Windows service masquerading: `Ias`, display name `Internet Authentication Service`, `servicedll=%SystemRoot%\system32\Iassvc.dll`.
- Inspect `C:\ProgramData\Microsoft\Windows\DeviceMetadataStore\en-US*.*` for non-Cabinet DPAPI blobs, especially files over 50 KiB or config-sized files under 20 KiB.
- Detect ETW patching of `EtwEventWrite` with bytes `48 33 c0 c3` and direct-syscall remapping from `\KnownDlls`.
- Network hunt for cookie fields `MSCC`, `MicrosoftApplicationsTelemetryDeviceId`, `MSFPC`, `HASH`, `LV`, `LU`, `MS0`, `MUID`, `at_check`, `ai_session`, and JSON keys `odata.metadata` / `armAuthorization`.

### Microsoft Defender CVEs

- Verify Antimalware Platform is at least 1.1.26040.8 and Defender platform is at least 4.18.26040.7.
- Prioritize CVE-2026-41091 for endpoint triage where local code execution is plausible; hunt for suspicious symlink/link-following abuse around Defender-scanned file paths.
- Monitor for Defender service crashes or repeated protection-disable events as possible CVE-2026-45498 exploitation.

## 8. 📊 Trends & Insights

- Developer endpoints are now a primary intrusion surface: TrapDoor combines registry abuse, AI-assistant prompt files, credential validation, and SSH lateral movement.
- Legitimate sites remain high-leverage delivery infrastructure: Ghost exploitation turns trusted content into ClickFix launch points.
- APT tooling is increasingly configuration-aware and forensic-resistant: RemotePE combines DPAPI environmental keying, memory-only execution, ETW suppression, and encrypted C2.
- Security tooling is still being targeted directly; Defender exploitation creates a path for privilege escalation or disabling protection before later-stage payloads.
- Ransomware-specific new incidents were not observed today in high-confidence last-24-hour primary sources, but Ghost ClickFix and developer credential theft are credible ransomware-enabling access paths.
