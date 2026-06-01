# Daily Cybersecurity Intelligence Brief - 2026-06-01

Reporting window: last 24 hours ending 2026-06-01 11:02 IST / 2026-06-01 05:32 UTC. Sources reviewed include The Hacker News Threat Intelligence section, BleepingComputer, Wordfence/Defiant, Palo Alto Networks, Rapid7, Arctic Wolf Labs, and accessible X.com-indexed security search results. No newly published THN Threat Intelligence malware-configuration article was observed inside the strict last-24-hour window; the newest relevant THN malware/configuration coverage remains EKZ Infostealer and related May 2026 malware reporting, retained only as context where noted.

## 1. 🧠 Executive Summary

- PAN-OS GlobalProtect CVE-2026-0257 remains the top edge-access risk: exploitation has been observed against vulnerable authentication-override configurations, enabling unauthorized VPN establishment.
- WP Maps Pro CVE-2026-8732 is under active exploitation against WordPress sites, with attackers creating administrator accounts through an unauthenticated privilege escalation flaw.
- No new high-confidence, last-24-hour ransomware outbreak, APT malware family, or malware sample set with fresh configuration was observed in accessible primary reporting today.
- The mandatory THN Threat Intelligence review did not surface a new malware-configuration post in the window; recent EKZ Infostealer reporting remains relevant because FortiClient EMS abuse turns endpoint management into a trusted malware delivery channel.
- X.com review via accessible indexed results showed discussion of PAN-OS, WordPress exploitation, and recent infostealer campaigns, but no independently verifiable tweet-only IOCs were recovered today.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-0257 - PAN-OS GlobalProtect Authentication Bypass

- Severity: High; vendor CVSS 7.8, operationally critical for exposed VPN infrastructure.
- Affected systems: Palo Alto Networks PAN-OS / Prisma Access GlobalProtect portals and gateways using authentication override with vulnerable cookie-certificate configuration.
- Exploitation status: In-the-wild. Palo Alto Networks reported limited exploitation attempts; Rapid7 reported successful exploitation across multiple customers beginning 2026-05-17 with additional activity from 2026-05-21 onward.
- Technical root cause: Authentication bypass in GlobalProtect authentication override handling, allowing attacker-controlled VPN session establishment when preconditions are met.
- Impact: Unauthorized VPN access can place an attacker directly inside enterprise network trust boundaries without valid user authentication.

### CVE-2026-8732 - WP Maps Pro Unauthenticated Privilege Escalation

- Severity: Critical; public reporting cites CVSS 9.8.
- Affected systems: WordPress sites running WP Maps Pro plugin versions 5.0.0 through 5.0.4.
- Exploitation status: In-the-wild. Wordfence observed over 3,600 exploit attempts starting 2026-05-30 and peaking on 2026-05-31.
- Technical root cause: Missing authorization / improper capability checks in temporary access functionality, allowing unauthenticated creation of administrator-level users.
- Impact: Full WordPress administrative takeover, followed by plugin/theme modification, webshell deployment, SEO spam, credential theft, or malware staging.

### CVE-2026-35616 - FortiClient EMS Abuse Delivering EKZ Infostealer

- Severity: Critical; public reporting cites CVSS 9.1.
- Affected systems: Fortinet FortiClient EMS 7.4.5 and 7.4.6, and managed Windows endpoints receiving EMS policy/script updates.
- Exploitation status: In-the-wild. Not newly disclosed today, but still relevant in the current active-exploitation window because it has recent confirmed malware delivery.
- Technical root cause: Improper access control / pre-authentication API access bypass enabling unauthenticated privileged EMS configuration changes.
- Impact: Compromised EMS can distribute malicious VPN scripts and payloads through trusted endpoint-management workflows.

## 3. 🦠 Malware & Campaign Analysis

### EKZ Infostealer via FortiClient EMS

- Malware name / family: EKZ Infostealer; payload observed as `p.exe` and `FortiEndpoint_Patch.exe`.
- Threat actor: Unknown financially motivated cluster tracked by Arctic Wolf; no public APT attribution.
- Initial access vector: Exploitation of CVE-2026-35616 against FortiClient EMS.
- Execution chain:
  1. Attacker sends unauthenticated requests to vulnerable EMS endpoints.
  2. EMS remote access profiles and endpoint policy artifacts are modified.
  3. Managed endpoints establish VPN/IPsec flows and execute EMS-delivered scripts.
  4. `fortitray.exe` or `ipsec.exe` launches GUID-named `.cmd` scripts from `C:\Program Files\Fortinet\FortiClient\logs\Trace\scripts\`.
  5. Scripted Base64 PowerShell downloads `p.exe` from attacker infrastructure.
  6. Payload runs as fake Fortinet patch tooling and steals browser credentials, cookies, autofill data, credit-card data, addresses, and phone numbers.
  7. Harvested data is staged locally and exfiltrated by PowerShell over HTTP.
- Persistence technique: No independent EKZ persistence confirmed in accessible reporting; attacker durability comes from malicious EMS-managed script/profile state until administrators clean it.
- C2 communication method: HTTP download and exfiltration through `83.138.53[.]110`, including `/dl/p.exe` and `/service/save.php`.

### New Malware Family / Fresh Configurations

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
    "104.207.144[.]154",
    "146.19.216[.]119",
    "146.19.216[.]120",
    "146.19.216[.]125",
    "83.138.53[.]110"
  ],
  "domains": [],
  "hashes": [],
  "urls": [
    "hxxp[:]//83.138.53[.]110/dl/p.exe",
    "hxxp[:]//83.138.53[.]110/service/save.php"
  ],
  "mutex": []
}
```

Additional non-JSON observables: Rapid7 reported GlobalProtect client hostnames `DESKTOP-GP01` and `GP-CLIENT`, plus spoofed MAC address `aa:bb:cc:dd:ee:ff`, in CVE-2026-0257 exploitation telemetry. WordPress CVE-2026-8732 reporting did not provide stable attacker IPs in accessible sources today.

## 5. 🔗 Technical Resources & Blogs

- The Hacker News Threat Intelligence section: https://thehackernews.com/search/label/Threat%20Intelligence
- BleepingComputer: WP Maps Pro bug exploited to create admin accounts on WordPress sites: https://www.bleepingcomputer.com/news/security/wp-maps-pro-bug-exploited-to-create-admin-accounts-on-wordpress-sites/
- Wordfence vulnerability details for WP Maps Pro CVE-2026-8732: https://www.wordfence.com/threat-intel/vulnerabilities/wordpress-plugins/wp-maps-pro/wp-maps-pro-500-504-unauthenticated-privilege-escalation
- Palo Alto Networks advisory for CVE-2026-0257: https://security.paloaltonetworks.com/CVE-2026-0257
- Rapid7 observed exploitation of PAN-OS GlobalProtect CVE-2026-0257: https://www.rapid7.com/blog/post/etr-rapid7-observed-exploitation-of-pan-os-globalprotect-authentication-bypass-vulnerability-cve-2026-0257/
- Arctic Wolf: FortiClient EMS exploited via CVE-2026-35616 to deliver EKZ Infostealer: https://arcticwolf.com/resources/blog/forticlient-ems-exploited-via-cve-2026-35616-to-deliver-ekz-infostealer-disguised-as-a-fortinet-patch/
- THN: FortiClient EMS exploitation with credential stealer: https://thehackernews.com/2026/05/threat-actors-exploit-critical.html

## 6. 🧪 Sandbox / Sample Links

- VirusTotal links: Not observed today.
- Any.Run links: Not observed today.
- Hybrid Analysis links: Not observed today.
- MalwareBazaar sample links: Not observed today.
- GitHub PoC repositories: Not observed today for CVE-2026-0257 or CVE-2026-8732 in accessible sources.

## 7. 🛡️ Detection & Mitigation

### PAN-OS CVE-2026-0257

- Patch PAN-OS / Prisma Access to fixed versions listed in Palo Alto Networks guidance.
- Disable GlobalProtect authentication override where feasible; otherwise issue a new certificate used only for authentication override.
- Hunt GlobalProtect logs for successful sessions sourced from `104.207.144[.]154`, `146.19.216[.]119`, `146.19.216[.]120`, and `146.19.216[.]125`.
- Prioritize events using hostnames `DESKTOP-GP01` or `GP-CLIENT`, spoofed MAC `aa:bb:cc:dd:ee:ff`, and cookie-based authentication paths without expected identity assurance.
- Treat successful VPN assignment without corresponding normal MFA/IdP telemetry as a containment event.

### WP Maps Pro CVE-2026-8732

- Upgrade WP Maps Pro to a fixed version beyond 5.0.4, or disable/remove the plugin until patched.
- Review WordPress users created since 2026-05-30, especially administrators without a business owner or expected change record.
- Hunt web logs for unauthenticated requests to WP Maps Pro temporary access endpoints followed by `wp-login.php`, `/wp-admin/`, plugin editor, theme editor, or file-upload activity.
- Search for newly modified PHP files under `wp-content/plugins/`, `wp-content/themes/`, and writable upload paths.
- Rotate WordPress administrator passwords and invalidate sessions on any site where unauthorized administrator creation is suspected.

### FortiClient EMS / EKZ Infostealer

- Upgrade FortiClient EMS to 7.4.7 or later.
- Review EMS logs for suspicious unauthenticated API activity and `Certificate not found in request header` patterns preceding policy/profile changes.
- Detect `fortitray.exe` or `ipsec.exe` spawning `cmd.exe` or PowerShell from `C:\Program Files\Fortinet\FortiClient\logs\Trace\scripts\`.
- Block `83.138.53[.]110`, `/dl/p.exe`, and `/service/save.php`; search for `FortiEndpoint_Patch.exe`, `p.exe`, and `C:\ProgramData\log.txt`.
- Investigate browser credential-store access immediately after FortiClient VPN tunnel setup.

## 8. 📊 Trends & Insights

- Active exploitation is concentrated on identity-adjacent edge systems and administrator surfaces: VPN authentication override and WordPress admin creation both convert a single flaw into immediate privileged access.
- Management-plane abuse remains a key malware delivery pattern. FortiClient EMS exploitation shows attackers prefer trusted endpoint-control paths over noisy first-stage loaders when available.
- Plugin and appliance telemetry should be treated as identity telemetry, not only application logs; the highest-signal detections today are successful session creation and privileged account creation anomalies.
- Public sample repositories and sandbox links were sparse today, indicating defenders should lean on behavioral detection and vendor/IR telemetry rather than sample-derived static rules.
- X.com did not add unique, verifiable IOCs in accessible results today; primary technical writeups remain the defensible source base for this brief.
