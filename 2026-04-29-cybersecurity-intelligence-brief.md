# Daily Cybersecurity Intelligence Brief - 2026-04-29

Time window: 2026-04-27 to 2026-04-29 IST. Focus: high-impact active exploitation, ransomware, supply-chain malware, and newly reported malware/configuration-bearing campaigns.

## 1. 🧠 Executive Summary

- Microsoft updated CVE-2026-32202 to active exploitation: Windows Shell auto-parses malicious LNK/CPL metadata and can leak Net-NTLMv2 material over SMB without a click.
- VECT 2.0 is being marketed as ransomware but behaves operationally as a wiper for files larger than 128 KB because it discards required ChaCha20-IETF nonces.
- GlassWorm v2 continues to weaponize Open VSX extensions against developers, with loader extensions installing GitHub-hosted VSIX payloads across VS Code-family IDEs.
- Infoblox documented fake CAPTCHA IRSF fraud and Keitaro abuse using commercial TDS infrastructure, cookie-based targeting, back-button hijacking, and SMS C2-style control.
- X.com review: retrievable search results did not expose independently verifiable last-24h tweet content or usable IOCs in this environment; no tweet-only intelligence is included.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-32202 - Windows Shell Spoofing / NTLM Authentication Coercion

- Severity: Medium by CVSS (4.3), operationally High where outbound SMB/NTLM is permitted.
- Affected systems: Microsoft Windows Shell; patched in April 2026 Patch Tuesday.
- Exploitation status: In-the-wild. Microsoft revised the advisory on 2026-04-27/28 to mark exploitation active.
- Technical root cause: Incomplete remediation of CVE-2026-21510 left an early path-resolution flaw. Explorer renders a folder containing a malicious LNK, shell32 resolves a UNC path embedded in CPL/Control Panel IDList metadata, and `PathFileExistsW` triggers SMB authentication before trust verification.
- Impact: Net-NTLMv2 hash exposure enabling offline cracking or NTLM relay. Prior chain associated with APT28 targeting Ukraine and EU entities.
- Sources: [The Hacker News](https://thehackernews.com/2026/04/microsoft-confirms-active-exploitation.html), [Akamai](https://www.akamai.com/blog/security-research/incomplete-patch-apt28s-zero-day-cve-2026-32202), [MSRC](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-32202).

### VECT 2.0 Ransomware / Wiper-by-Implementation

- Severity: Critical.
- Affected systems: Windows, Linux, VMware ESXi; default ESXi targeting includes `/vmfs/volumes`.
- Exploitation status: Publicly observed ransomware operation; VECT claims victims and has affiliate access via BreachForums/TeamPCP partnership.
- Technical root cause: Cryptographic implementation flaw. VECT uses raw ChaCha20-IETF with 32-byte keys and 12-byte nonces, not ChaCha20-Poly1305. For files greater than 131,072 bytes, it encrypts four chunks with four generated nonces but appends only the final nonce, permanently losing decryption material for the first three chunks.
- Impact: Paying ransom cannot restore most enterprise files. VM disks, databases, backups, documents, and mailboxes typically exceed the 128 KB boundary.
- Sources: [The Hacker News](https://thehackernews.com/2026/04/vect-20-ransomware-irreversibly.html), [Check Point Research](https://research.checkpoint.com/2026/vect-ransomware-by-design-wiper-by-accident/).

## 3. 🦠 Malware & Campaign Analysis

### VECT 2.0

- Malware family: VECT 2.0 ransomware, functionally destructive for large files.
- Threat actor: VECT RaaS operators; affiliates likely recruited via BreachForums and TeamPCP-linked supply-chain compromise fallout.
- Initial access vector: Not fully disclosed for current victims; operator ecosystem is explicitly tied to TeamPCP supply-chain credential/data abuse.
- Execution chain:
  1. Affiliate builds Windows/Linux/ESXi payload from VECT panel.
  2. Payload enumerates local/removable/network drives or ESXi datastores.
  3. Windows variant stops processes, deletes shadow copies, clears logs, disables Defender controls, and optionally configures Safe Mode execution.
  4. Linux/ESXi variants stop services, shut down VMs, wipe logs/history, and encrypt target paths.
  5. Large files are partially encrypted with lost nonce material and renamed with `.vect`.
- Persistence technique: Windows Safe Mode persistence via `bcdedit /set {default} safeboot minimal` and registry service-load path when `--force-safemode` is used; Linux/ESXi write login/ransom-note artifacts such as `/etc/motd`, `/etc/issue`, `/etc/profile.d/vector_notice.sh`.
- C2 communication method: No public C2 endpoints observed today. Operator panel/leak site exists, but Check Point published sample hashes rather than live C2.

### GlassWorm v2 Open VSX Supply-Chain Malware

- Malware family: GlassWorm v2, developer-focused stealer/RAT loader.
- Threat actor: Unknown.
- Initial access vector: Malicious and sleeper Open VSX extensions impersonating legitimate VS Code extensions; recent activity includes transitive dependency abuse and GitHub-hosted VSIX payload retrieval.
- Execution chain:
  1. Developer installs cloned/sleeper Open VSX extension.
  2. Extension activates as an obfuscated JavaScript loader.
  3. Loader retrieves secondary VSIX payload from GitHub.
  4. Payload installs into VS Code, Cursor, Windsurf, VSCodium, and other detected IDEs using `--install-extension`.
  5. Malware performs Russian-system avoidance, sensitive-data theft, RAT deployment, and rogue Chromium extension installation.
- Persistence technique: Socket reports macOS persistence via LaunchAgents in earlier/current GlassWorm activity.
- C2 communication method: HTTP exfiltration to actor-controlled endpoints; prior reporting includes Solana memo-based C2 discovery for related waves. No fresh IP/domain C2 was independently observed today.
- Sources: [The Hacker News](https://thehackernews.com/2026/04/researchers-uncover-73-fake-vs-code.html), [Socket GlassWorm v2 tracker](https://socket.dev/supply-chain-attacks/glassworm-v2).

### Fake CAPTCHA IRSF + Keitaro TDS Abuse

- Malware family: Not malware; TDS-enabled fraud/social-engineering infrastructure with C2-like server-side controls.
- Threat actor: Unknown SMS scam actor; Keitaro abuse also observed across multiple actors including TA2726 in broader reporting.
- Initial access vector: Typosquat domains, malicious ads, affiliate/TDS redirection chains, fake CAPTCHA pages.
- Execution chain:
  1. User hits typosquat or ad-driven entry point.
  2. Traffic flows through commercial TDS infrastructure, then actor-controlled TDS nodes.
  3. Fake CAPTCHA asks trivial questions and calls `makeTrackerDownload.php`.
  4. Server returns destination phone numbers, SMS content, and control parameters.
  5. JavaScript launches Android/iOS SMS app with pre-filled messages to many international numbers.
  6. Back-button hijacking and cookies trap/route users based on campaign suitability.
- Persistence technique: Browser push-notification prompts observed on related content sites; cookie-based tracking controls repeat targeting.
- C2 communication method: HTTPS API endpoints on fake CAPTCHA domains; campaign parameters include `clientId`, `productId`, `publisher_id`, `af`, and `groupds`.
- Sources: [The Hacker News](https://thehackernews.com/2026/04/fake-captcha-irsf-scam-and-120-keitaro.html), [Infoblox](https://www.infoblox.com/blog/threat-intelligence/hold-the-phone-international-revenue-share-fraud-driven-by-fake-captchas/).

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [],
  "domains": [
    "colnsdital.com",
    "hotnow.sweeffg.online",
    "zawsterris.com",
    "megaplaylive.com",
    "chat.matchnewtoday.com",
    "vids.chatorizon.com",
    "d.fufecarrol.top",
    "d.herbosfinx.com",
    "d.marraheltin.com",
    "d.panzozerrot.com",
    "d.remotesbuffalo.top",
    "d.ruelomamuy.com",
    "d.santafebuno.top",
    "d.vistertransit.com",
    "d.zerrotmamil.com",
    "r.buffalosolpe.top",
    "r.carrolvassin.top",
    "r.transitcaxip.com",
    "claimandwins.com",
    "4lifetips.com",
    "verifysuper.com"
  ],
  "hashes": [
    "a7eadcf81dd6fda0dd6affefaffcb33b1d8f64ddec6e5a1772d028ef2a7da0f2",
    "58e17dd61d4d55fa77c7f2dd28dd51875b0ce900c1e43b368b349e65f27d6fdd",
    "e1fc59c7ece6e9a7fb262fc8529e3c4905503a1ca44630f9724b2ccc518d0c06",
    "8ee4ec425bc0d8db050d13bbff98f483fff020050d49f40c5055ca2b9f6b1c4d",
    "9c745f95a09b37bc0486bf0f92aad4a3d5548a939c086b93d6235d34648e683f",
    "e512d22d2bd989f35ebaccb63615434870dc0642b0f60e6d4bda0bb89adee27a"
  ],
  "urls": [
    "https://d.ruelomamuy.com/makeTrackerDownload.php",
    "http://megaplaylive.com/?enc=<base64_campaign_payload>",
    "https://verifysuper.com/cl/i/wopmej?aff_sub4=<affiliate_id>&aff_sub5=US"
  ],
  "mutex": []
}
```

Additional package indicators:

- Open VSX malicious extensions reported by The Hacker News: `outsidestormcommand.monochromator-theme`, `keyacrosslaud.auto-loop-for-antigravity`, `krundoven.ironplc-fast-hub`, `boulderzitunnel.vscode-buddies`, `cubedivervolt.html-code-validate`, `winnerdomain17.version-lens-tool`.
- Recent Socket-tracked GlassWorm package artifacts include `sremovik.dendron-deep-hub`, `dranaven.flask-live-craft`, `tralarin.firefox-rich-lens`, `veltarik.duplicate-fast-helper`, `brixovik.es7-quick-hub`, `krosaven.dot-live-forge`, and `drovenko.data-live-suite`.

## 5. 🔗 Technical Resources & Blogs

- [Check Point Research - VECT: Ransomware by design, Wiper by accident](https://research.checkpoint.com/2026/vect-ransomware-by-design-wiper-by-accident/)
- [Akamai - Incomplete Patch of APT28 Zero-Day Leads to CVE-2026-32202](https://www.akamai.com/blog/security-research/incomplete-patch-apt28s-zero-day-cve-2026-32202)
- [Socket - GlassWorm v2 tracker](https://socket.dev/supply-chain-attacks/glassworm-v2)
- [Infoblox - International Revenue Share Fraud Driven by Fake CAPTCHAs](https://www.infoblox.com/blog/threat-intelligence/hold-the-phone-international-revenue-share-fraud-driven-by-fake-captchas/)
- [Infoblox IOC GitHub repository](https://github.com/infobloxopen/threat-intelligence)
- [The Hacker News Threat Intelligence section](https://thehackernews.com/search/label/Threat%20Intelligence)
- PoC GitHub repos: Not observed today for the highlighted active exploitation items.

## 6. 🧪 Sandbox / Sample Links

- VECT: [VirusTotal sample referenced by Check Point](https://www.virustotal.com/) for earlier ESXi variant; use the SHA-256 values above for lookup.
- Fake CAPTCHA: Infoblox reports an [ANY.RUN](https://app.any.run/) capture for `megaplaylive.com` redirection; direct public task ID not exposed in retrieved text.
- GlassWorm v2: Use Socket package artifact links and Open VSX package pages for sample acquisition; MalwareBazaar/Hybrid Analysis links not observed today.

## 7. 🛡️ Detection & Mitigation

- CVE-2026-32202:
  - Apply April 2026 Windows updates and verify deployment coverage.
  - Block outbound SMB/NTLM to the internet; monitor egress TCP/445 and WebDAV-like coercion paths.
  - Hunt for `.lnk` files embedding UNC paths and Control Panel CLSID `26EE0668-A00A-44D7-9371-BEB064C98683`.
  - Alert on Explorer-initiated SMB authentication to untrusted hosts.
- VECT 2.0:
  - Hunt for `.vect` extension creation, `!!!READ_ME!!!.txt`, `dvm3_wall.bmp`, `bcdedit /set {default} safeboot minimal`, `vssadmin delete shadows /all /quiet`, and `wevtutil cl`.
  - Monitor ESXi/Linux for mass service stops, `/var/log` wiping, `/vmfs/volumes` traversal, `scp`/`ssh` fan-out, and ransom-note writes to `/etc/motd`, `/etc/issue`, `/etc/profile.d/vector_notice.sh`.
  - Treat incidents as destructive; prioritize isolation and offline backup restoration over ransom negotiation.
- GlassWorm v2:
  - Inventory Open VSX extensions across VS Code, Cursor, Windsurf, VSCodium.
  - Block or review `--install-extension` executions initiated by extensions or shell scripts.
  - Monitor macOS LaunchAgents created by IDE-related processes and HTTP exfiltration of browser, wallet, keychain, VPN, Notes, or CI-secret artifacts.
  - Remove listed extensions and rotate developer tokens, Git credentials, CI/CD secrets, and wallet keys where exposure is plausible.
- Fake CAPTCHA / Keitaro TDS:
  - Block listed domains and patterns at DNS/proxy layer; monitor `makeTrackerDownload.php`, `clientId`, `productId`, `publisher_id`, `groupds`, and suspicious `af` affiliate parameters.
  - Detect mobile users redirected from typosquat domains to TDS chains, especially `*.sweeffg.online`, `zawsterris.com`, and fake CAPTCHA subdomains.
  - For telecom/fraud teams, correlate premium international SMS bursts to the country-code set listed in Infoblox reporting.
- YARA/Sigma:
  - Public YARA/Sigma rules for these exact 24-48h reports were not observed today.

## 8. 📊 Trends & Insights

- Ransomware operators are lowering affiliate friction before technical maturity catches up. VECT’s BreachForums model increases deployment risk even though its crypto implementation is flawed.
- Developer supply-chain attacks continue shifting from obvious typosquats to sleeper extensions, transitive dependency abuse, and payload installation across multiple IDE ecosystems.
- Windows credential theft remains viable via shell parsing and authentication coercion, even after RCE-class bugs are patched.
- TDS infrastructure is converging fraud, malware delivery, wallet drainers, and adtech-style victim filtering; DNS telemetry is increasingly decisive for early campaign detection.
- Not observed today: credible public PoC for CVE-2026-32202, fresh actor-owned C2 for VECT, public YARA/Sigma specific to VECT 2.0 or GlassWorm v2, and high-confidence X.com-only IOCs.
