# Daily Cybersecurity Intelligence Brief - 2026-05-18

Scope: high-impact activity observed from 2026-05-17 09:48 UTC through 2026-05-18. The Hacker News Threat Intelligence section was reviewed; direct X.com search pages did not render usable tweet text, so X-derived items are limited to indexed/feed-visible references.

## 1. 🧠 Executive Summary

- NGINX CVE-2026-42945 is now under in-the-wild exploitation, with honeypot activity showing attempts against an 18-year-old heap buffer overflow that can crash workers and may reach RCE on weakly hardened hosts.
- openDCIM exploitation is being automated from a China-hosted source using a Vulnhuntr-like scanner and dropping PHP web shells after chaining authz, SQLi, and command-injection flaws.
- Tycoon2FA has expanded into OAuth device-code phishing against Microsoft 365, using Trustifi click-tracking, Cloudflare Workers, layered JavaScript, and anti-analysis checks to steal OAuth tokens.
- MiniPlasma is a newly public Windows local privilege escalation PoC that gives SYSTEM on fully patched Windows 11; exploitation in the wild is not yet confirmed for MiniPlasma, but related leaked Windows zero-days have already been abused.
- THN Threat Intelligence had no new last-24-hour malware-family/configuration post; the latest relevant malware-configuration item remains Turla/Secret Blizzard's Kazuar modular P2P botnet, reviewed as carry-over because of the explicit requirement.

## 2. 🚨 Active Threats & Vulnerabilities

### NGINX CVE-2026-42945

- Name / CVE: CVE-2026-42945, NGINX `ngx_http_rewrite_module` heap buffer overflow.
- Severity: Critical, CVSS 9.2.
- Affected systems: NGINX Plus and NGINX Open versions 0.6.27 through 1.30.0 with vulnerable rewrite-module configuration.
- Exploitation status: In-the-wild exploitation observed by VulnCheck honeypots.
- Technical root cause: Heap buffer overflow in rewrite processing; unauthenticated crafted HTTP requests can crash worker processes and may permit RCE where exploitability constraints are met.
- Operational note: Reliable RCE appears configuration-dependent and significantly easier where ASLR is disabled; worker-crash DoS is practical enough to treat as urgent.

### openDCIM chained exploitation

- Name / CVE: CVE-2026-28515, CVE-2026-28516, CVE-2026-28517.
- Severity: Critical, CVSS 9.3 for CVE-2026-28515 and CVE-2026-28517.
- Affected systems: openDCIM data center infrastructure management deployments.
- Exploitation status: In-the-wild exploitation observed; activity reportedly originates from a single Chinese IP.
- Technical root cause: Missing authorization in LDAP configuration access, SQL injection, and OS command injection in `report_network_map.php` via unsanitized `dot` parameter.
- Impact: Five-request exploit chain can reach remote command execution and PHP web-shell deployment.

### Funnel Builder by FunnelKit checkout skimming

- Name / CVE: Funnel Builder by FunnelKit arbitrary script injection; CVE not assigned as of reporting.
- Severity: Critical.
- Affected systems: WordPress/WooCommerce sites using Funnel Builder before 3.15.0.3.
- Exploitation status: In-the-wild exploitation; Sansec observed injected payment skimmers on checkout pages.
- Technical root cause: Public checkout endpoint allowed unauthenticated callers to invoke internal methods without capability checks or method allow-listing, enabling writes to global External Scripts settings.
- Impact: Persistent client-side payment skimmer injection into checkout flows.

### MiniPlasma Windows zero-day

- Name / CVE: MiniPlasma; related historical bug CVE-2020-17103, no new CVE observed today.
- Severity: High.
- Affected systems: Fully patched Windows 11 Pro confirmed by BleepingComputer; broader affected Windows versions not fully established.
- Exploitation status: Public PoC released; in-the-wild exploitation not observed today.
- Technical root cause: Windows Cloud Filter driver `cldflt.sys` / `HsmOsBlockPlaceholderAccess` logic allows registry-key creation in the `.DEFAULT` hive without proper access checks via undocumented Cloud Files API behavior.
- Impact: Local standard user can obtain SYSTEM privileges.

## 3. 🦠 Malware & Campaign Analysis

### Tycoon2FA Microsoft 365 device-code phishing

- Malware name / family: Tycoon2FA phishing-as-a-service kit.
- Threat actor: Tycoon2FA operators and customers; specific affiliate not identified.
- Initial access vector: Invoice-themed phishing email with Trustifi click-tracking URL.
- Execution chain:
  1. Victim clicks Trustifi tracking URL.
  2. Traffic redirects through Trustifi, Cloudflare Workers, and multiple obfuscated JavaScript layers.
  3. Victim lands on fake Microsoft CAPTCHA page.
  4. Phishing page retrieves a Microsoft OAuth device code from attacker backend.
  5. Victim is instructed to enter the code at `microsoft.com/devicelogin` and complete MFA.
  6. Microsoft issues OAuth access and refresh tokens to the attacker-controlled device.
- Persistence technique: Long-lived OAuth refresh tokens and rogue device registration; mailbox/cloud access persists until token/device revocation.
- C2 communication method: Web delivery through redirect infrastructure; login infrastructure uses Node.js/axios user agents in observed eSentire IOCs.

### FunnelKit Magecart-style payment skimmer

- Malware name / family: Unnamed JavaScript payment skimmer; Magecart-style tradecraft.
- Threat actor: Unknown.
- Initial access vector: Exploitation of vulnerable Funnel Builder checkout endpoint.
- Execution chain:
  1. Attacker sends unauthenticated request to vulnerable Funnel Builder endpoint.
  2. Plugin settings are modified to include fake GTM/analytics-looking JavaScript.
  3. Checkout pages load `https://analytics-reports[.]com/wss/jquery-lib.js`.
  4. Loader opens `wss://protect-wss[.]com/ws`.
  5. C2 streams storefront-tailored skimmer code.
  6. Skimmer captures card numbers, CVVs, billing addresses, and personal data.
- Persistence technique: Stored script in Funnel Builder External Scripts settings.
- C2 communication method: HTTPS loader plus WebSocket C2.

### Kazuar modular P2P botnet review

- Malware name / family: Kazuar backdoor, modular P2P botnet variant.
- Threat actor: Turla / Secret Blizzard, linked by Microsoft to Russia's FSB-associated activity.
- Initial access vector: Not newly observed today; historical deployment via Turla intrusion chains.
- Execution chain:
  1. Kernel module coordinates tasks and elects an internal leader.
  2. Non-leader implants stay silent and communicate internally.
  3. Bridge module proxies leader traffic to external C2.
  4. Worker module performs collection: keylogging, screenshots, filesystem harvest, Outlook/MAPI collection, reconnaissance, and command execution.
  5. Data is encrypted, staged locally, and exfiltrated through the Bridge path.
- Persistence technique: Modular stateful implant with configurable scheduling and security-bypass options; specific persistence artifact not newly observed today.
- C2 communication method: HTTP, WebSockets, or Exchange Web Services; internal IPC via Windows Messaging, Mailslots, and named pipes with AES-encrypted Protobuf messages.

### Ransomware

- Malware name / family: Not observed today.
- Threat actor: Not observed today.
- Initial access vector: Not observed today.
- Execution chain: Not observed today.
- Persistence technique: Not observed today.
- C2 communication method: Not observed today.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "193.228.131.161",
    "166.1.241.247",
    "138.249.139.6",
    "166.1.255.233",
    "142.252.86.104",
    "130.49.117.205",
    "170.168.215.64",
    "50.118.198.26",
    "172.120.234.223",
    "45.39.175.12",
    "142.252.171.135",
    "166.88.219.45",
    "172.120.57.92",
    "172.121.59.99"
  ],
  "domains": [
    "analytics-reports.com",
    "protect-wss.com"
  ],
  "hashes": [],
  "urls": [
    "https://analytics-reports.com/wss/jquery-lib.js",
    "wss://protect-wss.com/ws",
    "https://thehackernews.com/2026/05/nginx-cve-2026-42945-exploited-in-wild.html",
    "https://thehackernews.com/2026/05/funnel-builder-flaw-under-active.html",
    "https://sansec.io/research/funnelkit-woocommerce-vulnerability-exploited",
    "https://www.bleepingcomputer.com/news/security/tycoon2fa-hijacks-microsoft-365-accounts-via-device-code-phishing/",
    "https://www.bleepingcomputer.com/news/microsoft/new-windows-miniplasma-zero-day-exploit-gives-system-access-poc-released/",
    "https://www.microsoft.com/en-us/security/blog/2026/05/14/kazuar-anatomy-of-a-nation-state-botnet/"
  ],
  "mutex": []
}
```

## 5. 🔗 Technical Resources & Blogs

- The Hacker News: NGINX CVE-2026-42945 exploited in the wild - https://thehackernews.com/2026/05/nginx-cve-2026-42945-exploited-in-wild.html
- The Hacker News: Funnel Builder flaw under active exploitation - https://thehackernews.com/2026/05/funnel-builder-flaw-under-active.html
- Sansec: Critical FunnelKit vulnerability threatens 40,000+ WooCommerce checkouts - https://sansec.io/research/funnelkit-woocommerce-vulnerability-exploited
- BleepingComputer: Tycoon2FA hijacks Microsoft 365 accounts via device-code phishing - https://www.bleepingcomputer.com/news/security/tycoon2fa-hijacks-microsoft-365-accounts-via-device-code-phishing/
- eSentire Tycoon2FA IOC repository - https://github.com/eSentire/iocs/blob/main/Tycoon2FA/Tycoon2fa-iocs-03-23-2026.txt
- BleepingComputer: MiniPlasma zero-day PoC - https://www.bleepingcomputer.com/news/microsoft/new-windows-miniplasma-zero-day-exploit-gives-system-access-poc-released/
- Microsoft: Kazuar anatomy of a nation-state botnet - https://www.microsoft.com/en-us/security/blog/2026/05/14/kazuar-anatomy-of-a-nation-state-botnet/
- THN Threat Intelligence section reviewed - https://thehackernews.com/search/label/Threat%20Intelligence

## 6. 🧪 Sandbox / Sample Links

- VirusTotal: Not observed today for Tycoon2FA, FunnelKit skimmer, NGINX exploitation, or MiniPlasma.
- Any.Run / Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.
- GitHub PoC: MiniPlasma PoC source/executable was reported as published by the researcher; direct repository URL not included here because it is weaponized local privilege escalation exploit code.

## 7. 🛡️ Detection & Mitigation

- NGINX: Patch to the fixed F5/NGINX releases immediately. Hunt for abnormal worker crashes, crafted rewrite-module requests, and unexpected segmentation faults around external HTTP probes. Confirm ASLR is enabled across exposed Linux hosts.
- openDCIM: Patch CVE-2026-28515/28516/28517, restrict openDCIM to trusted admin networks, and hunt for requests to LDAP config and `report_network_map.php` with attacker-controlled `dot` values. Review web roots for newly created PHP shells.
- FunnelKit: Upgrade Funnel Builder to 3.15.0.3 or later. Review `Settings > Checkout > External Scripts` for unfamiliar GTM/analytics snippets, base64-encoded loaders, `analytics-reports.com`, `protect-wss.com`, and checkout-page WebSocket activity.
- Tycoon2FA: Disable OAuth device-code flow where not required. Enforce admin consent for third-party apps, Continuous Access Evaluation, compliant-device conditional access, and token revocation for suspicious `deviceCode` authentications. Hunt Entra logs for Microsoft Authentication Broker, Node.js/axios user agents, and the eSentire IP set.
- MiniPlasma: Treat local untrusted code execution on Windows endpoints as SYSTEM-risk until Microsoft publishes guidance. Restrict execution from user-writable paths, prioritize EDR telemetry on `cldflt.sys` abuse, registry writes into `.DEFAULT`, and unexpected elevated `cmd.exe`/PowerShell spawned by standard users.
- Kazuar: Build behavior detections for encrypted internal IPC over named pipes/Mailslots, Outlook/MAPI bulk collection, Bridge-like hosts making low-volume EWS/HTTP/WebSocket C2 traffic, and AMSI/ETW/WLDP bypass attempts.
- YARA / Sigma: No new public YARA or Sigma rules observed today for the NGINX exploit activity, FunnelKit skimmer, Tycoon2FA device-code campaign, or MiniPlasma.

## 8. 📊 Trends & Insights

- Exploit windows remain compressed: NGINX weaponization appeared within days of disclosure, while openDCIM exploitation shows AI-assisted vulnerability discovery tooling being operationalized for web-shell deployment.
- Identity attacks are shifting from password capture to OAuth token issuance; Tycoon2FA's device-code workflow turns legitimate Microsoft authentication into attacker device enrollment.
- E-commerce skimmers continue to hide as analytics and tag-management code, making configuration review and checkout-page network telemetry more valuable than file-only scanning.
- Windows public zero-day disclosures are creating a rolling LPE risk surface; even when PoCs are local-only, they materially raise post-compromise impact for phishing, loader, and ransomware operators.
