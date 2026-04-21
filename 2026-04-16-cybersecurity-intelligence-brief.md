# DAILY CYBERSECURITY INTELLIGENCE BRIEF — 2026-04-16

## 1. 🧠 Executive Summary
- **UAC-0247** is actively targeting Ukrainian government and municipal healthcare entities with a new **AGINGFLY** malware chain, using humanitarian-aid phishing lures, LNK/HTA execution, and staged loaders.
- **n8n webhook abuse** has scaled sharply (Talos observed a 686% rise vs. Jan 2025 baseline), enabling phishing-to-malware delivery and recipient fingerprinting from trusted `*.app.n8n.cloud` infrastructure.
- **CVE-2026-33032 (nginx-ui / MCPwn)** remains high-risk with active exploitation reporting; unauthenticated MCP endpoint access enables full Nginx config takeover and traffic interception.
- Attackers continue to blend **trusted services + LOLBins + post-exploitation tunneling/RMM tooling** (mshta, PowerShell, Datto/ITarian, Ligolo/Chisel), reducing static-detection effectiveness.
- Immediate priorities: patch/contain exposed nginx-ui, hard-block risky script/shortcut execution paths, and detect abnormal webhook-driven delivery chains.

## 2. 🚨 Active Threats & Vulnerabilities

### Threat 1: CVE-2026-33032 (nginx-ui MCP auth bypass, “MCPwn”)
- **Severity:** Critical (CVSS 9.8)
- **Affected systems:** `nginx-ui` deployments exposing MCP endpoints (`/mcp`, `/mcp_message`), especially internet-facing instances
- **Exploitation status:** **In-the-wild** (reported active exploitation)
- **Technical root cause:** Authentication gap + fail-open IP allowlist behavior on `/mcp_message`; request routing allows unauthenticated tool execution to modify/reload Nginx config

### Threat 2: UAC-0247 / AGINGFLY campaign
- **Severity:** High
- **Affected systems:** Ukrainian public-sector Windows endpoints (government, municipal clinics, emergency hospitals; possible defense-linked targets)
- **Exploitation status:** **In-the-wild** targeted operations (March–April 2026 activity disclosed Apr 16)
- **Technical root cause:** Multi-stage social engineering and execution abuse (phishing -> LNK -> `mshta.exe` -> HTA/loader), plus credential theft tooling and encrypted C2

### Threat 3: n8n webhook abuse for malware delivery and fingerprinting
- **Severity:** High
- **Affected systems:** Email recipients and organizations trusting `*.app.n8n.cloud` URLs without behavioral controls
- **Exploitation status:** **In-the-wild** campaign activity observed since Oct 2025; fresh Talos reporting Apr 15, 2026
- **Technical root cause:** Abuse of legitimate webhook infrastructure as trusted content relay; JavaScript-mediated payload retrieval and tracking-pixel telemetry evasion

## 3. 🦠 Malware & Campaign Analysis

### Campaign A: AGINGFLY / UAC-0247
- **Malware family:** AGINGFLY (with SILENTLOOP, RAVENSHELL stagers)
- **Threat actor:** UAC-0247 (origin not yet attributed)
- **Initial access vector:** Spear-phishing (humanitarian aid pretext), redirection to compromised/fake pages, LNK payload delivery
- **Execution chain (step-by-step):**
  1. Phishing email lures target to click aid-themed link.
  2. Redirect to compromised (XSS-abused) or attacker-controlled page.
  3. LNK triggers `mshta.exe`, pulling remote HTA.
  4. HTA shows decoy and launches staged loaders; shellcode injected into legitimate process.
  5. AGINGFLY + SILENTLOOP deployed; credential/data theft tooling launched (ChromElevator, ZAPiDESK).
  6. Recon/lateral movement via RustScan/Ligolo/Chisel; optional cryptomining observed.
- **Persistence technique:** Scheduled task creation; staged loader architecture; RAT capability expansion at runtime
- **C2 communication method:** WebSockets (AGINGFLY), reverse TCP (RAVENSHELL-like), C2 discovery/update via Telegram-assisted logic; encrypted C2 traffic reported

### Campaign B: n8n webhook malware delivery
- **Malware family:** Modified RMM-backdoor chains (Datto RMM / ITarian Endpoint Management droppers)
- **Threat actor:** Unknown (multiple clusters)
- **Initial access vector:** Phishing emails embedding n8n webhook URLs (often OneDrive-style lure + CAPTCHA)
- **Execution chain (step-by-step):**
  1. User opens phishing email with n8n webhook URL.
  2. Webhook serves dynamic HTML/JS, often with CAPTCHA gate.
  3. JS pulls external executable or MSI payload.
  4. Payload installs modified RMM tool; PowerShell/msiexec-based setup runs.
  5. Host establishes outbound management/C2 channel; attacker gains persistent remote access.
- **Persistence technique:** Scheduled tasks and managed-agent style foothold via modified legitimate tools
- **C2 communication method:** Outbound connections through RMM relay infrastructure and attacker-controlled web resources

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [],
  "domains": [
    "onedrivedownload.zoholandingpage.com",
    "majormetalcsorp.com",
    "pagepoinnc.app.n8n.cloud",
    "monicasue.app.n8n.cloud",
    "centrastage.net"
  ],
  "hashes": [
    "93a09e54e607930dfc068fcbc7ea2c2ea776c504aa20a8ca12100a28cfdcc75a",
    "7f30259d72eb7432b2454c07be83365ecfa835188185b35b30d11654aadf86a0"
  ],
  "urls": [
    "https://onedrivedownload.zoholandingpage.com/my-workspace/DownloadedOneDrive",
    "https://majormetalcsorp.com/Openfolder",
    "https://pagepoinnc.app.n8n.cloud/webhook/downloading-1a92cb4f-cff3-449d-8bdd-ec439b4b3496",
    "https://monicasue.app.n8n.cloud/webhook/download-file-92684bb4-ee1d-4806-a264-50bfeb750dab"
  ],
  "mutex": []
}
```

- IP-level and mutex-level indicators: **Not observed today** in high-confidence public reporting.

## 5. 🔗 Technical Resources & Blogs
- The Hacker News Threat Intelligence (category): https://thehackernews.com/search/label/Threat%20Intelligence
- UAC-0247 / AGINGFLY coverage: https://thehackernews.com/2026/04/uac-0247-targets-ukrainian-clinics-and.html
- Additional AGINGFLY technical recap: https://www.bleepingcomputer.com/news/security/new-agingfly-malware-used-in-attacks-on-ukraine-govt-hospitals/
- Cisco Talos n8n abuse analysis: https://blog.talosintelligence.com/the-n8n-n8mare/
- Talos IOC repository (April 2026): https://github.com/Cisco-Talos/IOCs/tree/main/2026/04
- nginx-ui advisory (GHSA-h6c2-x2m2-mwhf): https://github.com/0xJacky/nginx-ui/security/advisories/GHSA-h6c2-x2m2-mwhf
- Pluto Security technical analysis (CVE-2026-33032): https://pluto.security/blog/mcp-bug-nginx-security-vulnerability-cvss-9-8/
- nginx-ui patched release v2.3.4: https://github.com/0xJacky/nginx-ui/releases/tag/v2.3.4

## 6. 🧪 Sandbox / Sample Links
- VirusTotal:
  - https://www.virustotal.com/gui/file/93a09e54e607930dfc068fcbc7ea2c2ea776c504aa20a8ca12100a28cfdcc75a
  - https://www.virustotal.com/gui/file/7f30259d72eb7432b2454c07be83365ecfa835188185b35b30d11654aadf86a0
- Any.Run links: **Not observed today**
- Hybrid Analysis links: **Not observed today**
- MalwareBazaar sample links: **Not observed today**

## 7. 🛡️ Detection & Mitigation
- **Network/IDS signatures:** Cisco Talos Snort rule update includes `1:66274` (`POLICY-OTHER n8n webhook request attempt`) for monitoring suspicious n8n webhook traffic patterns.
- **Behavioral detections (high-value):**
  - Alert on Office/email client parent spawning browser -> downloaded executable/MSI -> `msiexec.exe`/PowerShell chain.
  - Alert on `mshta.exe` execution from user contexts and LNK/HTA/JS child-process trees.
  - Detect new scheduled tasks immediately following browser-driven downloads.
  - Hunt for abnormal outbound WebSocket/TCP beacons from user workstations to unapproved infrastructure.
  - For nginx-ui, monitor unauthenticated requests to `/mcp_message` and config mutations followed by reload/restart actions.
- **Patch/workarounds:**
  - Upgrade nginx-ui to **v2.3.4+** immediately.
  - If patch lag exists: disable MCP exposure, enforce auth on `/mcp_message`, and restrict access to trusted management networks.
  - Apply strict attachment/script controls: block or constrain LNK/HTA/JS execution and harden PowerShell execution policy/logging.

## 8. 📊 Trends & Insights
- **Trusted-platform abuse is accelerating:** adversaries are shifting delivery and tracking to legitimate cloud automation and SaaS infrastructure instead of disposable attacker domains.
- **Execution-chain modularity is increasing:** AGINGFLY’s runtime handler compilation and staged components reflect a move toward slimmer initial payloads and adaptive post-compromise logic.
- **Control-plane takeover risk is rising with MCP integrations:** security-control asymmetry around new AI/MCP endpoints is now an active exploitation pathway with direct service-impact potential.
- **Detection pressure point:** static URL/domain blocking alone is insufficient; organizations need correlation across email telemetry, process ancestry, and post-download behavior.
