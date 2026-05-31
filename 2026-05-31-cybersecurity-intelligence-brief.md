# Daily Cybersecurity Intelligence Brief - 2026-05-31

Reporting window: last 24 hours ending 2026-05-31 12:47 IST / 2026-05-31 07:17 UTC. Sources reviewed include The Hacker News homepage and Threat Intelligence section, Palo Alto Networks, Rapid7, Sysdig, Arctic Wolf Labs, WithSecure Labs, Socket, Microsoft Security Blog, BleepingComputer, and accessible X.com-indexed security search results. The strict last-24-hour window contained limited new high-impact reporting; newest THN Threat Intelligence malware/configuration items from May 29 are retained where necessary for the required malware-family/configuration review.

## 1. 🧠 Executive Summary

- PAN-OS GlobalProtect CVE-2026-0257 moved into active exploitation: successful VPN session establishment was observed against unpatched edge devices with vulnerable authentication-override configuration.
- Marimo CVE-2026-39987 exploitation shows a material post-exploitation shift: Sysdig observed an LLM-agent-driven intrusion pivoting from exposed notebook RCE to AWS credential theft, SSH key retrieval, bastion access, and PostgreSQL exfiltration.
- FortiClient EMS CVE-2026-35616 remains a high-impact malware delivery path: attackers abuse trusted endpoint-management configuration to push EKZ Infostealer as a fake Fortinet patch.
- The mandatory THN Threat Intelligence review surfaced malware/config-heavy items: GREYVIBE's PhantomRelay/FallSpy/LegionRelay ecosystem, EKZ Infostealer, and malicious Sicoob.Sdk / npm supply-chain credential theft.
- X.com review found indexed discussion around PAN-OS CVE-2026-0257, EKZ Infostealer, Sicoob.Sdk, and vpmdhaj npm packages, but no unique last-24-hour tweet-only IOCs were independently recoverable.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-0257 - PAN-OS GlobalProtect Authentication Bypass

- Severity: High operational priority; vendor CVSS 7.8.
- Affected systems: Palo Alto Networks PAN-OS and Prisma Access GlobalProtect portals/gateways with authentication override enabled and reused authentication override cookie certificate conditions.
- Exploitation status: In-the-wild. Palo Alto Networks reported limited exploit attempts on unpatched devices; Rapid7 observed successful exploitation across customers, with earliest activity on 2026-05-17 and a second wave on 2026-05-21.
- Technical root cause: Authentication bypass in GlobalProtect portal/gateway authentication override handling, allowing unauthorized VPN connection establishment under specific certificate/configuration conditions.
- Impact: Edge VPN access can place attackers inside trusted network paths without valid user authentication.

### CVE-2026-39987 - Marimo Pre-Authentication RCE

- Severity: Critical.
- Affected systems: Marimo notebook deployments up to and including 0.20.4; fixed in 0.23.0.
- Exploitation status: In-the-wild. Sysdig documented exploitation of an internet-reachable Marimo instance followed by AI-agent-assisted post-exploitation.
- Technical root cause: Pre-authenticated command execution through vulnerable Marimo terminal/WebSocket exposure.
- Impact: Initial RCE enabled credential discovery, AWS Secrets Manager access, SSH private key retrieval, bastion compromise, and internal PostgreSQL database exfiltration in under an hour.

### CVE-2026-35616 - FortiClient EMS Abuse to Deploy EKZ Infostealer

- Severity: Critical; CVSS 9.1 in public reporting.
- Affected systems: Fortinet FortiClient EMS 7.4.5 and 7.4.6; managed Windows endpoints receiving EMS-controlled VPN/profile scripts.
- Exploitation status: In-the-wild. Fortinet disclosed active exploitation earlier; Arctic Wolf observed May 2026 attacks delivering EKZ Infostealer.
- Technical root cause: Improper access control / pre-authentication API access bypass allowing unauthenticated privileged EMS configuration changes.
- Impact: Compromised EMS becomes a trusted malware distribution tier across managed endpoints.

### Sicoob.Sdk / vpmdhaj Supply-Chain Credential Theft

- Severity: High.
- Affected systems: .NET developers using malicious NuGet `Sicoob.Sdk` versions 2.0.0-2.0.4; npm users installing vpmdhaj OpenSearch/ElasticSearch/DevOps typosquats.
- Exploitation status: Real-world malicious packages published and removed/blocked; active downstream compromise not confirmed in accessible reporting.
- Technical root cause: Registry package impersonation, source-to-package mismatch, preinstall hook execution, and hardcoded exfiltration endpoints.
- Impact: Theft of banking PFX certificates/passwords, client IDs, AWS metadata credentials, Vault tokens, npm tokens, and CI/CD secrets.

## 3. 🦠 Malware & Campaign Analysis

### EKZ Infostealer via FortiClient EMS

- Malware name / family: EKZ Infostealer; delivered as `p.exe`, installed/executed as `FortiEndpoint_Patch.exe`.
- Threat actor: Unknown cluster observed by Arctic Wolf.
- Initial access vector: CVE-2026-35616 exploitation against FortiClient EMS.
- Execution chain:
  1. Attacker sends crafted unauthenticated requests to EMS endpoints processed as privileged administrative actions.
  2. EMS configuration is modified, including remote access profile and endpoint policy updates.
  3. Managed endpoints establish VPN/IPsec tunnel.
  4. `fortitray.exe` or `ipsec.exe` launches GUID-named `.cmd` files under `C:\Program Files\Fortinet\FortiClient\logs\Trace\scripts\`.
  5. `.cmd` invokes Base64 PowerShell.
  6. PowerShell downloads `hxxp[:]//83.138.53[.]110/dl/p.exe`, executes it, waits, and POSTs harvested output to attacker infrastructure.
  7. EKZ extracts Chromium/Edge/Firefox/Gecko browser credentials, cookies, autofill data, credit-card data, addresses, and phone numbers.
- Persistence technique: No standalone EKZ persistence confirmed; malicious persistence/effect lives in EMS-managed VPN script/profile configuration until removed.
- C2 communication method: HTTP download and exfiltration via `83.138.53[.]110`; EKZ itself stages data locally and PowerShell performs network exfiltration.

### GREYVIBE PhantomRelay / FallSpy / LegionRelay

- Malware name / family: PhantomRelay, PhantomRelayV1/V2, FallSpy Android spyware, LegionRelay PowerShell RAT.
- Threat actor: GREYVIBE, a Russian-speaking Russia-nexus group assessed by WithSecure as state-aligned with cybercrime overlap.
- Initial access vector: Spear-phishing email archives, fake CAPTCHA/ClickFix pages, fake Zoom/LAPAS domains, fraudulent Ukrainian adult-club sites, and charity/drone-themed lures.
- Execution chain:
  1. Victim receives spear-phish or visits themed lure site.
  2. JavaScript/PowerShell loaders execute decoys and retrieve PhantomRelay/LegionRelay, or Android lures install FallSpy.
  3. LegionRelay communicates by REST API and runs operator-issued PowerShell commands.
  4. Operators stage scripts for file enumeration, exfiltration, screenshots, browser-data theft, Telegram/WhatsApp data theft, and RDP setup.
  5. FallSpy collects contacts, call logs, installed apps, SIM-linked numbers, network/device data, Wi-Fi SSID, location, public IP, and media files.
- Persistence technique: PhantomRelayV1 uses a custom watchdog persistence mechanism; GREYVIBE also rotates custom obfuscators/loaders including LOOKVALPS, LOOKVALJS, DAYLIGHT, and TEASOUP.
- C2 communication method: LegionRelay REST API C2; WithSecure published GREYVIBE YARA and IOC CSV resources.

### Sicoob.Sdk / vpmdhaj Registry Malware

- Malware name / family: Malicious NuGet banking SDK; vpmdhaj npm cloud/CI credential harvester.
- Threat actor: Unknown; npm cluster used maintainer alias `vpmdhaj` and email `a39155771@gmail.com`.
- Initial access vector: Developer installs malicious registry package through NuGet or npm.
- Execution chain:
  1. `Sicoob.Sdk` presents as an official C# SDK for Sicoob banking APIs.
  2. Constructor receives client ID, PFX path, and PFX password during normal production-mode initialization.
  3. Malicious DLL reads PFX file, Base64-encodes it, and sends client ID/password/cert data to hardcoded Sentry DSN.
  4. Separate Sentry path captures raw Boleto API responses.
  5. vpmdhaj npm packages execute through `preinstall`/setup scripts.
  6. Gen-1 chain: `node -> preinstall.js -> HTTP C2 -> payload.bin`.
  7. Gen-2 chain: `node -> setup.mjs -> Bun runtime -> bundled stage-2`.
  8. Payload hunts AWS IMDS/ECS metadata, Vault, Secrets Manager, and npm publish tokens.
- Persistence technique: Not observed today; execution is installer/lifecycle-script based.
- C2 communication method: Sentry ingest DSN for Sicoob.Sdk; `aab.sportsontheweb[.]net` HTTP stage-1 endpoint for vpmdhaj npm packages.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "104.207.144[.]154",
    "146.19.216[.]119",
    "146.19.216[.]120",
    "146.19.216[.]125",
    "157.66.54[.]26",
    "104.28.162[.]160",
    "104.28.165[.]251",
    "104.28.165[.]169",
    "104.28.157[.]50",
    "83.138.53[.]110",
    "185.220.101[.]15",
    "192.42.116[.]14",
    "169.254.169[.]254",
    "169.254.170[.]2"
  ],
  "domains": [
    "aab.sportsontheweb[.]net",
    "o4511335034847232[.]ingest[.]de[.]sentry[.]io"
  ],
  "hashes": [
    "0da123adf9251957a4b850a3f6bd6a753dd4892be176a84a18450e899534cc5e",
    "17e771c78430cc67e71d4547f8996a1a488e9d3f",
    "338662fd0c4d750a0ba203a32b59f081",
    "8c5b72906e8183037532afc3f4639931"
  ],
  "urls": [
    "hxxp[:]//83.138.53[.]110/dl/p.exe",
    "hxxp[:]//83.138.53[.]110/service/save.php",
    "hxxps[:]//d565e3f03d0b1a7c8935d7ff94237316@o4511335034847232[.]ingest[.]de[.]sentry[.]io/4511337546317904",
    "hxxp[:]//aab.sportsontheweb[.]net/x.php"
  ],
  "mutex": []
}
```

Notes: PAN-OS IOCs include Rapid7-observed source IPs, hostnames `DESKTOP-GP01` and `GP-CLIENT`, and spoofed MAC `aa:bb:cc:dd:ee:ff`; these are excluded from the JSON where they do not match requested IOC categories. Registry keys and mutexes were not observed today.

## 5. 🔗 Technical Resources & Blogs

- The Hacker News Threat Intelligence section: https://thehackernews.com/search/label/Threat%20Intelligence
- THN: PAN-OS GlobalProtect CVE-2026-0257 under active exploitation: https://thehackernews.com/2026/05/pan-os-globalprotect-authentication.html
- Palo Alto Networks advisory: https://security.paloaltonetworks.com/CVE-2026-0257
- Rapid7: Observed CVE-2026-0257 exploitation and IOCs: https://www.rapid7.com/blog/post/etr-rapid7-observed-exploitation-of-pan-os-globalprotect-authentication-bypass-vulnerability-cve-2026-0257/
- Sysdig: LLM-agent post-exploitation after Marimo CVE-2026-39987: https://www.sysdig.com/blog/ai-agent-at-the-wheel-how-an-attacker-used-llms-to-move-from-a-cve-to-an-internal-database-in-4-pivots
- Arctic Wolf: EKZ Infostealer via FortiClient EMS CVE-2026-35616: https://arcticwolf.com/resources/blog/forticlient-ems-exploited-via-cve-2026-35616-to-deliver-ekz-infostealer-disguised-as-a-fortinet-patch/
- THN: FortiClient EMS exploitation with credential stealer: https://thehackernews.com/2026/05/threat-actors-exploit-critical.html
- THN: GREYVIBE AI-powered campaigns: https://thehackernews.com/2026/05/new-russian-linked-greyvibe-targets.html
- WithSecure Labs: GREYVIBE technical report: https://labs.withsecure.com/publications/greyvibe
- WithSecure GREYVIBE YARA/IOC repository: https://github.com/WithSecureLabs/iocs/tree/master/GREYVIBE/
- Socket: Malicious Sicoob.Sdk NuGet package: https://socket.dev/blog/malicious-nuget-package-impersonates-sicoob-sdk
- Microsoft: vpmdhaj npm typosquats stealing cloud and CI/CD secrets: https://www.microsoft.com/en-us/security/blog/2026/05/28/typosquatted-npm-packages-used-steal-cloud-ci-cd-secrets/

## 6. 🧪 Sandbox / Sample Links

- VirusTotal: Not observed today in accessible primary reporting.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.
- GitHub/YARA: WithSecure GREYVIBE YARA and IOC repository - https://github.com/WithSecureLabs/iocs/tree/master/GREYVIBE/
- PoC: No new last-24-hour PoC was observed for PAN-OS CVE-2026-0257 or Marimo CVE-2026-39987 in accessible sources today.

## 7. 🛡️ Detection & Mitigation

### PAN-OS CVE-2026-0257

- Patch urgently to fixed PAN-OS/Prisma Access builds listed by Palo Alto Networks/Rapid7.
- Disable GlobalProtect authentication override where feasible, or generate a new certificate used exclusively for authentication override.
- Hunt GlobalProtect logs for cookie authentication to local admin accounts, spoofed MAC `aa:bb:cc:dd:ee:ff`, hostnames `DESKTOP-GP01` / `GP-CLIENT`, and source IPs `104.207.144[.]154`, `146.19.216[.]119`, `146.19.216[.]120`, `146.19.216[.]125`.
- Treat unexpected VPN IP assignment without corresponding normal identity controls as a containment trigger.

### Marimo CVE-2026-39987

- Upgrade Marimo to 0.23.0 or later.
- Remove internet exposure from Marimo notebooks and block terminal/WebSocket access from untrusted networks.
- Rotate AWS keys, SSH keys, and database credentials accessible from Marimo processes.
- Detect command streams that chain `.env`, `/proc/*/environ`, `~/.aws/credentials`, `~/.pgpass`, `~/.ssh/id_*`, AWS Secrets Manager reads, and rapid multi-IP SSH fan-out.
- Monitor for structured command output delimiters (`---`) and abnormal `psql` table enumeration/dump sequences from bastion hosts.

### FortiClient EMS / EKZ

- Upgrade FortiClient EMS to 7.4.7 or later.
- Review EMS logs for `Certificate not found in request header` followed by unexpected `fortinet-ca2` Fabric updates.
- Hunt for `fortitray.exe` or `ipsec.exe` spawning `cmd.exe` and Base64 PowerShell from `C:\Program Files\Fortinet\FortiClient\logs\Trace\scripts\{GUID}.cmd`.
- Block and investigate `83.138.53[.]110`, `/dl/p.exe`, and `/service/save.php`.
- Search endpoints for `FortiEndpoint_Patch.exe`, `p.exe`, `C:\ProgramData\log.txt`, and browser credential-store access immediately after VPN tunnel establishment.

### Sicoob.Sdk / vpmdhaj Packages

- Remove `Sicoob.Sdk` versions 2.0.0-2.0.4; rotate Sicoob client IDs, PFX certificates, and PFX passwords.
- Audit Sicoob API and Boleto transaction logs for unexpected access after package installation.
- Block npm maintainer alias `vpmdhaj`, package names listed in Microsoft's advisory, `aab.sportsontheweb[.]net`, and `hxxp://aab.sportsontheweb[.]net/x.php`.
- Detect npm lifecycle execution spawning `node`, `bun`, downloaded binaries, AWS IMDS (`169.254.169[.]254`), ECS metadata (`169.254.170[.]2`), Vault, or Secrets Manager access during package installation.

## 8. 📊 Trends & Insights

- Edge identity infrastructure remains the fastest route to enterprise impact: PAN-OS and FortiClient EMS both convert perimeter-management weaknesses into trusted access or trusted execution.
- Malware delivery is moving into management planes and developer workflows; attackers are abusing FortiClient EMS, NuGet, npm, Bun, Sentry, and package metadata rather than relying only on conventional droppers.
- AI-assisted operations are now observable in command telemetry, not just lure quality: the Marimo intrusion showed adaptive schema discovery, value handoff between commands, and machine-oriented output formatting.
- Supply-chain attackers are shifting from obvious typo names to "manufactured legitimacy": plausible package names, spoofed metadata, clean GitHub facades, inflated versions, and domain-specific credential capture.
- X.com did not provide independently verifiable unique IOCs today through accessible indexed results; primary-source technical reporting remains the defensible basis for this brief.
