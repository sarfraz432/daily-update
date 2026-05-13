# Daily Cybersecurity Intelligence Brief - 2026-05-13

## 1. 🧠 Executive Summary

- TeamPCP's Mini Shai-Hulud campaign is the highest-impact event today: 170+ npm/PyPI packages, valid SLSA provenance abuse, CI/CD token theft, and worm-like propagation into developer ecosystems.
- Operation SilentCanvas shows enterprise intrusion tradecraft using a fake JPEG PowerShell loader, on-host C# compilation, UAC bypass, trojanized ScreenConnect, encrypted C2, credential interception, and RMM persistence.
- CRPx0 is a newly reported cross-platform malware/ransomware campaign using OnlyFans lures for crypto theft, data exfiltration, and file encryption across Windows/macOS, with Linux capability in development.
- Microsoft patched a critical Outlook/Word zero-click UAF RCE (CVE-2026-40361); no exploitation is reported, but the attack path is email preview/rendering with "exploitation more likely."
- Foxconn confirmed a North American cyberattack claimed by Nitrogen ransomware, with alleged 8 TB/11M-document theft; operational recovery is underway, public technical IOCs are not yet available.

## 2. 🚨 Active Threats & Vulnerabilities

### Mini Shai-Hulud / TeamPCP Supply-Chain Worm

- **Severity:** Critical
- **Affected systems:** npm and PyPI consumers; CI/CD runners; GitHub Actions workflows; packages under TanStack, UiPath, Mistral AI, OpenSearch, Squawk, Guardrails AI, and others.
- **Exploitation status:** Active in the wild; malicious versions published on May 11-12, 2026.
- **Technical root cause:** CI/CD trust-boundary abuse: `pull_request_target` workflow exposure, GitHub Actions cache poisoning, runtime extraction of OIDC token material from runner process memory, and malicious package publication with valid Sigstore/SLSA provenance.
- **Impact:** Credential theft, cloud/API token theft, GitHub/NPM propagation, persistent token-monitoring daemon, destructive logic in some locale conditions.

### CVE-2026-40361 - Microsoft Outlook/Word Zero-Click RCE

- **Severity:** Critical
- **Affected systems:** Microsoft Word rendering components used by Word and Outlook; enterprise Outlook/Exchange environments are high-value.
- **Exploitation status:** PoC reported by researcher; no in-the-wild exploitation observed today.
- **Technical root cause:** Use-after-free in email/document rendering path; Outlook preview/read path can trigger without link/attachment interaction.
- **Notes:** Microsoft rates exploitation as more likely. Plain-text email rendering is a partial mitigation where immediate patching is delayed.

### CVE-2026-44277 / CVE-2026-26083 - Fortinet FortiAuthenticator and FortiSandbox RCE

- **Severity:** Critical
- **Affected systems:** FortiAuthenticator 6.5.0-6.5.6, 6.6.0-6.6.8, 8.0.0/8.0.2; FortiSandbox/FortiSandbox Cloud/FortiSandbox PaaS WEB UI affected versions.
- **Exploitation status:** No exploitation reported by Fortinet/SecurityWeek today.
- **Technical root cause:** Improper access control (CVE-2026-44277) and missing authorization (CVE-2026-26083), reachable remotely via crafted requests.

### Foxconn / Nitrogen Ransomware Claim

- **Severity:** High
- **Affected systems:** Foxconn North American factory infrastructure; exact systems not public.
- **Exploitation status:** Confirmed cyberattack; ransomware group claim publicly reported.
- **Technical root cause:** Not publicly disclosed.
- **Impact:** Foxconn says affected factories are resuming normal operations. Nitrogen alleges theft of 8 TB and over 11 million documents.

## 3. 🦠 Malware & Campaign Analysis

### Mini Shai-Hulud

- **Malware/family:** Mini Shai-Hulud supply-chain worm / credential stealer.
- **Threat actor:** TeamPCP.
- **Initial access vector:** Compromised package release pipelines and GitHub Actions workflow abuse.
- **Execution chain:**
  1. Attacker opens PR/fork path that triggers a vulnerable `pull_request_target` workflow.
  2. Attacker-controlled code poisons GitHub Actions cache across fork/base boundary.
  3. Later legitimate workflow restores poisoned cache.
  4. Implant extracts OIDC token material from runner memory.
  5. Malicious packages are published with trusted identity and valid SLSA provenance.
  6. `router_init.js` / Python payloads steal credentials and attempt propagation.
- **Persistence technique:** Persistent daemon polling GitHub for token revocation; repository commits using spoofed Claude Code app identity; package re-publication where tokens permit.
- **C2 communication method:** `git-tanstack[.]com`, GitHub dead-drop repositories, and Session network (`*.getsession[.]org`) encrypted exfiltration.

### Operation SilentCanvas

- **Malware/family:** Trojanized ConnectWise ScreenConnect intrusion framework.
- **Threat actor:** Unknown.
- **Initial access vector:** Social engineering with `sysupdate.jpeg` fake image file; likely phishing, fake updates, or deceptive file-sharing.
- **Execution chain:**
  1. `sysupdate.jpeg` executes PowerShell, not image content.
  2. Loader creates `C:\Systems` staging directories.
  3. Payload downloads secondary components from `legitserver.theworkpc[.]com` over TCP/5443.
  4. PowerShell uses AMSI bypass and in-memory execution.
  5. `csc.exe`/`cvtres.exe` compile `uds.exe` on host.
  6. `ComputerDefaults.exe` plus `HKCU\Software\Classes\ms-settings\shell\open\command` performs fileless UAC bypass.
  7. Trojanized ScreenConnect runs from `C:\ProgramData\OneDriveServer\...`.
- **Persistence technique:** Windows service masquerading as `OneDriveServers`; hidden desktop; hidden local account management; process watchdog.
- **C2 communication method:** Encrypted ScreenConnect-like sessions to `legitserver.theworkpc[.]com:8041`, internally mapped to `45.138.16[.]64`; PBKDF2/HMAC-SHA256 derived session keys.

### CRPx0

- **Malware/family:** CRPx0 cross-platform malware/ransomware.
- **Threat actor:** Unknown.
- **Initial access vector:** `OnlyfansAccounts.zip` lure with `Onlyfans Accounts.lnk`.
- **Execution chain:**
  1. Victim downloads ZIP promising free OnlyFans credentials.
  2. LNK displays decoy `Accounts.txt` content.
  3. Malware fingerprints host, establishes C2, and periodically updates itself.
  4. Clipboard hijacker swaps cryptocurrency wallet addresses.
  5. Operator-selected files are exfiltrated.
  6. `crypter.py` is downloaded and executed via local Python interpreter.
  7. Files are encrypted with `.crpx0`; ransom notes dropped in English, Russian, and Chinese.
- **Persistence technique:** Periodic C2 check-in/update behavior; exact autorun mechanism not observed in public reporting today.
- **C2 communication method:** Operator-controlled C2; specific endpoints not observed today.

### Foxconn / Nitrogen

- **Malware/family:** Nitrogen ransomware operation.
- **Threat actor:** Nitrogen; reporting links the operation to ALPHV/BlackCat lineage and Conti 2 codebase.
- **Initial access vector:** Not observed today.
- **Execution chain:** Not observed today.
- **Persistence technique:** Not observed today.
- **C2 communication method:** Not observed today.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "45.138.16.64",
    "83.142.209.194"
  ],
  "domains": [
    "legitserver.theworkpc.com",
    "git-tanstack.com",
    "*.getsession.org"
  ],
  "hashes": [
    "7adffc1c0b3fdcba46e8d0a81203c955976d4ef39893c98d0b2dbfbb8d6a8ec3",
    "ecd5ed16975d556d1d17bc980f248f8a5262bed11df9d9cf999efd9c273c11df",
    "cea1d85967d2c456fccecae3a70ff2adfe4c113aacf9d18c35906c2ed24ca9b4",
    "e4c9f3bb4a65c640795bfc1a56c0b56485b849ccd97027eed7ad9aa78a732a4f",
    "ee3d776cdaf82335e4293e19ee313cc35eee49cde9963b96766a8f9c89d44a79",
    "4d8ac85c5b98c69ba44146df61183e9bf613edd796aa516c3ae73611b7d77c06",
    "A635F0C94C98B658AE799978994F0D0A292567CD97B8A19068A8423D1297652A",
    "MD5:7DD05336097E5A833F03A63D3221494F"
  ],
  "urls": [
    "https://www.cyfirma.com/research/operation-silentcanvas-jpeg-based-multistage-powershell-intrusion/",
    "https://www.securityweek.com/tanstack-mistral-ai-uipath-hit-in-fresh-supply-chain-attack/",
    "https://www.securityweek.com/free-onlyfans-lure-used-to-spread-cross-platform-crpx0-malware/",
    "https://www.bleepingcomputer.com/news/security/electronics-giant-foxconn-confirms-cyberattack-on-north-american-factories/",
    "https://www.securityweek.com/microsoft-patches-critical-zero-click-outlook-vulnerability-threatening-enterprises/"
  ],
  "mutex": []
}
```

## 5. 🔗 Technical Resources & Blogs

- THN Threat Intelligence feed reviewed: https://thehackernews.com/search/label/Threat%20Intelligence. Latest relevant malware/config posts in feed include the May 11 fake OpenAI/Hugging Face Rust infostealer and May 8 TCLBANKER/PamDOORa items; these were not re-promoted because they were already covered or outside the strict 24-48h focus.
- Mini Shai-Hulud overview: https://www.securityweek.com/tanstack-mistral-ai-uipath-hit-in-fresh-supply-chain-attack/
- Snyk TanStack analysis: https://snyk.io/blog/tanstack-npm-packages-compromised/
- Wiz Mini Shai-Hulud analysis: https://www.wiz.io/blog/mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised
- CYFIRMA Operation SilentCanvas: https://www.cyfirma.com/research/operation-silentcanvas-jpeg-based-multistage-powershell-intrusion/
- CRPx0 reporting with Aryaka analysis reference: https://www.securityweek.com/free-onlyfans-lure-used-to-spread-cross-platform-crpx0-malware/
- Microsoft Outlook CVE-2026-40361: https://www.securityweek.com/microsoft-patches-critical-zero-click-outlook-vulnerability-threatening-enterprises/
- Fortinet/Ivanti patch reporting: https://www.securityweek.com/fortinet-ivanti-patch-critical-vulnerabilities/
- Foxconn/Nitrogen ransomware: https://www.bleepingcomputer.com/news/security/electronics-giant-foxconn-confirms-cyberattack-on-north-american-factories/
- X.com review: indexed X-linked reporting surfaced Haifei Li's post on CVE-2026-40361 impact; live unauthenticated X search did not yield additional unique IOCs today.

## 6. 🧪 Sandbox / Sample Links

- VirusTotal: Not observed today for the above samples.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.

## 7. 🛡️ Detection & Mitigation

- **Mini Shai-Hulud:** Inventory installs from May 11-12 for affected namespaces/packages; isolate CI runners that installed suspect versions; rotate GitHub, npm, PyPI, cloud, Vault, Kubernetes, password-manager, and AI-tool secrets exposed to build hosts; audit `pull_request_target`, cache restore paths, and OIDC federation policies; do not treat SLSA provenance alone as benign.
- **SilentCanvas:** Alert on `.jpeg` files lacking JPEG magic bytes that invoke PowerShell; block/monitor `csc.exe`, `cvtres.exe`, and `ComputerDefaults.exe` spawned from PowerShell; detect writes to `HKCU\Software\Classes\ms-settings\shell\open\command`; hunt for `C:\Systems`, `uds.exe`, `OneDriveServers`, `C:\ProgramData\OneDriveServer`, outbound `:5443`/`:8041`, and unauthorized ScreenConnect services.
- **CYFIRMA YARA:** Use the published `APT_ScreenConnect_Dropper`, `APT_CSC_CompiledLauncher`, and `APT_UDS_CompiledDropper` rules from the SilentCanvas report; tune for local path variants and compile-after-delivery behavior.
- **CRPx0:** Detect LNK execution from ZIP archives, fake credential text-file decoys, clipboard wallet address substitution, Python execution of `crypter.py`, `.crpx0` extension creation, and ransom notes in multiple languages.
- **CVE-2026-40361:** Apply Microsoft May 2026 updates quickly; consider temporary plain-text email rendering for high-risk Outlook users until patch coverage is verified.
- **Fortinet critical RCEs:** Patch FortiAuthenticator to 6.5.7/6.6.9/8.0.3 or later and apply FortiSandbox/FortiSandbox Cloud/FortiSandbox PaaS fixes; restrict management interfaces from untrusted networks.
- **Foxconn/Nitrogen:** Monitor for Nitrogen leak-site claims touching suppliers/customers; no actionable technical IOCs observed today.

## 8. 📊 Trends & Insights

- Software supply-chain attacks are moving from stolen-package-token abuse to trusted-build compromise: valid provenance now proves where malicious code was built, not that the code is safe.
- RMM abuse remains a preferred post-compromise substrate; SilentCanvas shows attackers blending signed enterprise tooling with unsigned/tampered DLLs and local encrypted configuration.
- Attackers are converging credential theft, exfiltration, and ransomware into one modular kill chain, visible in both CRPx0 and Mini Shai-Hulud.
- X.com-derived signal today was useful for vulnerability triage (Outlook zero-click risk) but did not provide unique malware IOCs beyond primary research and news sources.
