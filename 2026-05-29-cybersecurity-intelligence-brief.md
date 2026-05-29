# Daily Cybersecurity Intelligence Brief - 2026-05-29

Reporting window: last 24 hours ending 2026-05-29 14:07 IST / 2026-05-29 08:37 UTC. Sources reviewed include The Hacker News Threat Intelligence section, Arctic Wolf Labs, Wiz Research, ENKI WhiteHat, Kaspersky Securelist, Rapid7, CISA/KEV-indexed reporting, BleepingComputer/SecurityWeek coverage, and accessible X.com-indexed security search results. Focus: active exploitation, zero-days, APT campaigns, malware with configuration/C2 detail, ransomware-enabling access, and major breaches.

## 1. 🧠 Executive Summary

- The mandatory The Hacker News Threat Intelligence review surfaced Kimsuky/Velvet Chollima activity deploying a new HTTPSpy variant, HelloDoor, HttpMalice, AppleSeed, VS Code tunnels, Cloudflare tunnels, and DWAgent against South Korean military, corporate, public, and private-sector targets.
- CVE-2026-35616 FortiClient EMS exploitation remains operationally high impact: Arctic Wolf observed attackers abusing trusted endpoint-management paths to push a previously unreported Windows browser credential stealer, EKZ Infostealer, as a fake Fortinet patch.
- Wiz disclosed JINX-0164, a financially motivated actor targeting cryptocurrency organizations and developers with recruiter/meeting lures, AUDIOFIX macOS infostealer/RAT, MiniRAT, and CI/CD/source-code tampering.
- Rapid7 disclosed an unpatched critical Gogs authenticated RCE with public Metasploit support; no CVE was assigned at publication, but default-configured internet-facing Gogs instances are exposed to repo-wide compromise.
- X.com review found indexed discussion/amplification around FortiClient EMS, Gogs RCE, and JINX-0164, but no unique last-24-hour tweet-only IOCs were independently recoverable from accessible public search.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-35616 - Fortinet FortiClient EMS Abuse to Deploy EKZ Infostealer

- Severity: Critical, CVSS 9.1 per THN/Arctic Wolf reporting; CISA KEV-listed.
- Affected systems: Fortinet FortiClient EMS 7.4.5 and 7.4.6; managed Windows endpoints receiving VPN/profile policy from compromised EMS.
- Exploitation status: In-the-wild; Fortinet/CISA reported active exploitation in April, and Arctic Wolf observed May 2026 follow-on intrusions delivering EKZ.
- Technical root cause: Improper access control / pre-authentication API access bypass enabling unauthenticated privileged EMS configuration changes via crafted requests.
- Operational impact: Compromised EMS becomes a malware distribution mechanism to managed endpoints through FortiClient VPN on-connect script execution.

### Gogs Authenticated Rebase RCE - No CVE Assigned

- Severity: Critical, CVSS 9.4 per Rapid7.
- Affected systems: Gogs 0.14.2 and 0.15.0+dev on Windows, Linux, and macOS; default configurations with user registration/repository creation materially increase risk.
- Exploitation status: Public Metasploit module / PoC available; active in-the-wild exploitation not confirmed in accessible reporting.
- Technical root cause: Argument injection into `git rebase --exec` through malicious branch names during "Rebase before merging"; affected code path uses raw `process.ExecDir` instead of hardened git-module APIs.
- Operational impact: Authenticated users can execute arbitrary OS commands, read private repositories across tenants, dump secrets, tamper with hosted code, and pivot from the Git server.

### Microsoft Public Zero-Day Disclosures - BlueHammer / RedSun / UnDefend

- Severity: High to Critical depending on component and local exposure.
- Affected systems: Microsoft Windows components including Defender and BitLocker-related attack surfaces; exact supported-build coverage varies by advisory.
- Exploitation status: THN reports BlueHammer (CVE-2026-33825), RedSun (CVE-2026-41091), and UnDefend (CVE-2026-45498) have come under active exploitation after uncoordinated disclosures.
- Technical root cause: Public exploit details for multiple Windows component flaws; full root-cause detail remains fragmented because disclosures and account removals disrupted public repositories.
- Operational impact: Elevated endpoint compromise risk where PoCs are available before complete Microsoft servicing coverage.

### Kimsuky HTTPSpy / HelloDoor / VS Code Tunnel Campaign

- Severity: High.
- Affected systems: South Korean military and corporate entities; Kaspersky also reports South Korean public/private-sector, defense, military, government, medical, machinery, and energy-sector targeting.
- Exploitation status: In-the-wild APT activity observed March-April 2026 and disclosed in the reporting window.
- Technical root cause: Social engineering with fake security-software installers, fake Webex meeting pages, malicious JSE/PIF/SCR/EXE droppers, and legitimate remote-access tunnel abuse.
- Operational impact: Remote shell, file transfer, screenshot capture, DLL injection, command execution, GPKI certificate extraction, and covert access through VS Code tunnels or RMM tooling.

## 3. 🦠 Malware & Campaign Analysis

### EKZ Infostealer via FortiClient EMS

- Malware name / family: EKZ Infostealer; payload filename `FortiEndpoint_Patch.exe`, delivered as `p.exe`.
- Threat actor: Unknown threat cluster tracked by Arctic Wolf; no confident attribution.
- Initial access vector: Exploitation of CVE-2026-35616 against FortiClient EMS management servers, followed by malicious EMS configuration changes.
- Execution chain:
  1. Attacker sends crafted unauthenticated requests to FortiClient EMS privileged API endpoints.
  2. EMS logs show `Certificate not found in request header`; real-world exploitation also showed `Certificate user: fortinet-ca2 FortiGate: Fabric device (SN=fortinet-ca2) successfully updated`.
  3. Threat actor modifies `remind_upgrade_after`, Remote Access Profile configuration, and endpoint policy.
  4. Managed endpoints establish IPsec/VPN tunnel and FortiClient executes GUID-named `.cmd` scripts from `C:\Program Files\Fortinet\FortiClient\logs\Trace\scripts\`.
  5. `fortitray.exe` or `ipsec.exe` spawns `cmd.exe`, which starts Base64 PowerShell.
  6. PowerShell downloads `hxxp[:]//83.138.53[.]110/dl/p.exe`, runs `FortiEndpoint_Patch.exe`, sleeps, reads staged output, POSTs results to `hxxp[:]//83.138.53[.]110/service/save.php`, then deletes artifacts.
  7. EKZ extracts Chrome/Edge/Chromium and Firefox/Gecko browser credentials, cookies, autofill data, credit-card data, addresses, and phone data into `C:\ProgramData\log.txt`.
- Persistence technique: No malware-native persistence confirmed; persistence/effect is achieved through EMS-managed VPN profile/script configuration until cleaned.
- C2 communication method: Raw HTTP to `83.138.53[.]110`; payload download from `/dl/p.exe`, exfiltration to `/service/save.php`.

### JINX-0164 AUDIOFIX / MiniRAT Campaign

- Malware name / family: AUDIOFIX Python macOS infostealer/RAT; MiniRAT Go backdoor.
- Threat actor: JINX-0164, previously undocumented financially motivated cluster. Wiz notes tradecraft resemblance to some North Korean crypto-theft clusters but reports no infrastructure overlap sufficient for attribution.
- Initial access vector: LinkedIn/recruiter social engineering, fake virtual meeting portals, fake technical-error pages, and a prior poisoned npm package `@velora-dex/sdk`.
- Execution chain:
  1. Actor uses credible LinkedIn/recruiter personas to steer crypto/developer targets to fake meeting domains.
  2. Victim downloads a fake meeting client or "audio/camera fix."
  3. Bash script from `apple.driver-store[.]com` or related driver-update infrastructure detects Intel vs Apple Silicon.
  4. Script downloads a payload masquerading as `coreaudiod`, saves it as `ChromeUpdater`, marks it executable, and runs it with `launchctl`.
  5. AUDIOFIX steals password manager, browser, iCloud Keychain, local admin, SSH, configuration, console history, crypto-extension, wallet-address, Discord, Slack, and Telegram session data.
  6. Actor laterally accesses CI/CD and code distribution systems, injects AUDIOFIX into build/source paths, and attempts downstream wallet credential theft.
  7. MiniRAT, seen in the `@velora-dex/sdk` supply-chain case, provides shell execution, file upload/download, compression, and payload retrieval.
- Persistence technique: macOS LaunchAgents such as `~/Library/LaunchAgents/com.microsoft.teams.coreaudiod.plist`, `io.aircall.workspace.helper.plist`, and `com.electron.dialpad.helper.plist`.
- C2 communication method: HTTPS/Dropbox variants for AUDIOFIX; embedded C2 domains include `cloud-sync[.]online`, `datahub[.]ink`, and `byte-io[.]us`.

### Kimsuky HTTPSpy / HelloDoor / HttpMalice

- Malware name / family: HTTPSpy RAT, HelloDoor Rust-based PebbleDash variant, HttpMalice, HttpTroy, AppleSeed/HappyDoor.
- Threat actor: Kimsuky / Velvet Chollima, North Korea-linked.
- Initial access vector: Fake South Korean B2B messaging security software installers, fake Cisco Webex meeting page, and malicious JSE/PIF/SCR/EXE droppers.
- Execution chain:
  1. Victim visits fake security software or fake Webex page.
  2. Fake installers `nos-setup.exe` or `astx-setup.exe` masquerade as nProtect Online Security/AhnLab Safe Transaction.
  3. Installer launches `MemLoader.dll` via `regsvr32.exe`; batch cleanup removes on-disk traces.
  4. `MemLoader.dll` creates scheduled-task persistence and polls C2 for selectively delivered payloads.
  5. In the Webex chain, `fix-camera.jse` retrieves `mTSTCv8.mdxm` by PowerShell.
  6. Downloader performs anti-analysis checks and fetches `engine.dat` or `spyInster.dll`.
  7. Final DLL drops `cacheMon.dat`, which launches HTTPSpy; fake meeting page redirects to a legitimate Webex meeting to preserve credibility.
- Persistence technique: Scheduled tasks, VS Code Remote Tunneling, Cloudflare Quick Tunnels, and DWAgent remote management in related Kaspersky-observed clusters.
- C2 communication method: HTTP C2 for HTTPSpy/HttpMalice/HttpTroy; legitimate VS Code tunnels can remove the need for conventional malware C2.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "83.138.53[.]110",
    "185.220.101[.]15",
    "192.42.116[.]14",
    "185.100.85[.]250",
    "84.32.83[.]250",
    "163.172.53[.]20",
    "185.100.85[.]98",
    "153.92.126[.]84",
    "45.45.217[.]242",
    "89.36.224[.]5",
    "208.115.220[.]17",
    "185.175.59[.]85"
  ],
  "domains": [
    "apple.driver-store[.]com",
    "driver-store[.]com",
    "windows.driver-store[.]com",
    "driver-update[.]io",
    "apple.driver-update[.]io",
    "windows.driver-update[.]io",
    "driver-updater[.]net",
    "driver-hub[.]net",
    "apple.driver-hub[.]net",
    "drvstore[.]com",
    "apple.drvstore[.]com",
    "cloud-sync[.]online",
    "datahub[.]ink",
    "byte-io[.]us",
    "live[.]us[.]org",
    "team[.]live[.]us[.]org",
    "teams[.]live[.]us[.]org",
    "teams[.]cam",
    "teamicrosoft[.]com",
    "bitget-meeting[.]com",
    "us03-slack[.]online",
    "slktest[.]live",
    "live[.]ong",
    "teams[.]us[.]org",
    "retesta[.]live"
  ],
  "hashes": [
    "0da123adf9251957a4b850a3f6bd6a753dd4892be176a84a18450e899534cc5e",
    "17e771c78430cc67e71d4547f8996a1a488e9d3f",
    "338662fd0c4d750a0ba203a32b59f081",
    "0a8ab3d16b12d3a453ee5a3208fe04744ad54514ef8ea27bb8fe32679efad270",
    "0b028b781950641818800fee2b4bf68e4ef2bcee53fe71a21755275ba108783d",
    "a35d2b67fa478a7174e308b43ce30bf69b3bc6f44fa76197fdf95fc2fbc1cf5b",
    "65cba741fe30fa4799fb9002ea8de6d96042a59159dd7c3419c766af24c835e6",
    "0b1a36a31b952341a534fe24890f1ed2921ee259773cff46e4f6273b8c4d5d21",
    "e8ee6f5145c9d503c5130bfc6585567f6e19d409158c3c0ca0b259f1875b15f4",
    "3e3901519c2305fbe9d5483b7234c25c6d2b562512916481d96f26b849c39fdb",
    "9c2ce925133a3bf5a924063bbef8df49918d5b7258695c1894cd18c75970157a",
    "402625ec79e3573a80b6de9b33fc1e503e3c7803603cd958ddd515fb0549007c",
    "b6cab0b3aa8e56e2427f486c74588d598ae58bb0cbc0eda6939fe171cb0aed17",
    "d4e863f9818bfb2f1dd932df6441dff204e6142c3bdb55b298cb08dc7b6a0c62",
    "c6ef82d2864dfd26f117a1ef5602679153423f2742970a7949cec72722f0a01e",
    "2a10ffe0367bb1b26ba2c3bc600892c21074725c0b8c9dc9161e6ceb33915460",
    "995a0a49ae4b244928b3f67e2bfd7a6e",
    "52f1ff082e981cbdfd1f045c6021c63f",
    "9fe43e08c8f446554340f972dac8a68c",
    "8e15c4d4f71bdd9dbc48cd2cabc87806",
    "65fc9f06de5603e2c1af9b4f288bb22c",
    "c19aeaedbbfc4e029f7e9bdface495b9",
    "8983ffa6da23e0b99ccc58c17b9788c7",
    "a7f0a18ac87e982d6f32f7a715e12532",
    "f4465403f9693939fe9c439f0ab33610",
    "5c373c2116ab4a615e622f577e22e9be",
    "d1ec20144c83bba921243e72c517da5e",
    "58ac2f65e335922be3f60e57099dc8a3",
    "f73ba062116ea9f37d072aa41c7f5108",
    "7e0825019d0de0c1c4a1673f94043ddb",
    "08160acf08fccecde7b34090db18b321",
    "94faed9af49c98a89c8acc55e97276c9",
    "c42ae004badddd3017adadbdd1421e00",
    "9ca5f93a732f404bbb2cee848f5bbda0"
  ],
  "urls": [
    "hxxp[:]//83.138.53[.]110/dl/p.exe",
    "hxxp[:]//83.138.53[.]110/service/save.php",
    "hxxps[:]//apple.driver-store[.]com/mac/arm/driver/coreaudiod",
    "hxxps[:]//apple.driver-store[.]com/mac/intel/driver/coreaudiod",
    "hxxps[:]//apple.driver-update[.]io/troubleshoot/mac/audio-issue-fix.sh"
  ],
  "mutex": []
}
```

Notes:
- EKZ SHA256/SHA1/MD5 are from Arctic Wolf's payload table.
- JINX-0164 SHA256, infrastructure, and host artifacts are from Wiz Research.
- Kimsuky hashes are MD5 values published by Kaspersky Securelist. Registry-key and mutex values were not observed today in accessible primary reporting.

## 5. 🔗 Technical Resources & Blogs

- The Hacker News Threat Intelligence: Kimsuky HTTPSpy / HelloDoor - https://thehackernews.com/2026/05/kimsuky-deploys-httpspy-expands-arsenal.html
- ENKI WhiteHat: Kimsuky JSONPing, Webex spoofing, new HTTPSpy variant - https://www.enki.co.kr/en/media-center/blog/kimsuky-s-advanced-attack-techniques-jsonping-webex-spoofing-and-a-new-httpspy-variant
- Kaspersky Securelist: Kimsuky PebbleDash / AppleSeed tooling - https://securelist.com/kimsuky-appleseed-pebbledash-campaigns/119785/
- Arctic Wolf Labs: FortiClient EMS exploited via CVE-2026-35616 to deliver EKZ Infostealer - https://arcticwolf.com/resources/blog/forticlient-ems-exploited-via-cve-2026-35616-to-deliver-ekz-infostealer-disguised-as-a-fortinet-patch/
- The Hacker News: FortiClient EMS exploitation with credential stealer - https://thehackernews.com/2026/05/threat-actors-exploit-critical.html
- Canadian Centre for Cyber Security: FortiClient EMS CVE-2026-35616 advisory and CISA KEV context - https://www.cyber.gc.ca/en/alerts-advisories/al26-007-vulnerability-impacting-fortinet-forticlientems-cve-2026-35616
- Wiz Research: JINX-0164 targets crypto organizations - https://www.wiz.io/blog/threat-actors-target-crypto-orgs
- The Hacker News: JINX-0164 macOS malware campaign - https://thehackernews.com/2026/05/jinx-0164-targets-cryptocurrency-firms.html
- Rapid7: Authenticated RCE via argument injection in Gogs - https://www.rapid7.com/blog/post/ve-authenticated-rce-via-argument-injection-gogs-unfixed/
- Rapid7 Metasploit PR for Gogs rebase RCE - https://github.com/rapid7/metasploit-framework/pull/21515
- The Hacker News: Microsoft response to public zero-day disclosures - https://thehackernews.com/2026/05/microsoft-slams-public-zero-day.html

## 6. 🧪 Sandbox / Sample Links

- Kaspersky OpenTIP hash references for Kimsuky samples are linked directly from the Securelist IOC table: https://securelist.com/kimsuky-appleseed-pebbledash-campaigns/119785/
- VirusTotal direct links for EKZ, AUDIOFIX, and MiniRAT samples were not exposed in accessible primary text today.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.
- GitHub PoC: Rapid7 Metasploit module PR for Gogs RCE - https://github.com/rapid7/metasploit-framework/pull/21515

## 7. 🛡️ Detection & Mitigation

### FortiClient EMS CVE-2026-35616 / EKZ

- Upgrade FortiClient EMS to 7.4.7 or later, or apply Fortinet hotfixes for 7.4.5/7.4.6. Treat exposed EMS as compromised until logs and endpoint artifacts are reviewed.
- Hunt EMS logs for `Certificate not found in request header` followed closely by `Certificate user: fortinet-ca2 FortiGate: Fabric device (SN=fortinet-ca2) successfully updated`.
- Audit EMS Remote Access Profile and endpoint policy XML for unexpected `on_connect` / `script` directives.
- Endpoint process detection:
  - `fortitray.exe` or `ipsec.exe` -> `cmd.exe` -> `powershell.exe` -> `FortiEndpoint_Patch.exe`
  - GUID-named scripts under `C:\Program Files\Fortinet\FortiClient\logs\Trace\scripts\{GUID}.cmd`
  - execution/staging of `C:\ProgramData\log.txt`
- Network detection: raw HTTP download from `/dl/p.exe` and POST exfiltration to `/service/save.php` on `83.138.53[.]110`.
- Rotate browser-stored credentials and invalidate sessions for endpoints that executed the payload; EKZ targets cookies and saved credentials that may bypass MFA through session reuse.

### JINX-0164 AUDIOFIX / MiniRAT

- Hunt macOS LaunchAgents:
  - `~/Library/LaunchAgents/com.microsoft.teams.coreaudiod.plist`
  - `~/Library/LaunchAgents/io.aircall.workspace.helper.plist`
  - `~/Library/LaunchAgents/com.electron.dialpad.helper.plist`
- Detect `curl` from fake driver domains saving `coreaudiod` as `$HOME/Library/Application Support/Google/ChromeUpdater`, followed by `chmod +x` and `launchctl`.
- Search for host artifacts `/audio.lock`, `/helper.log`, `/clip`, `/tokens.txt`, and `~/.zsh_cache`.
- Monitor developer endpoints for access to CI/CD, source-code distribution, package-publishing systems, and crypto wallet material after recruiter/meeting-lure exposure.
- Block JINX-0164 meeting spoofing and driver-update domains, with special priority for `apple.driver-store[.]com`, `driver-update[.]io`, `driver-hub[.]net`, `drvstore[.]com`, `cloud-sync[.]online`, and `datahub[.]ink`.

### Kimsuky HTTPSpy / HelloDoor

- Detect fake installer names `nos-setup.exe`, `astx-setup.exe`, `fix-camera.jse`, `mTSTCv8.mdxm`, `engine.dat`, `spyInster.dll`, and `cacheMon.dat`.
- Alert on `regsvr32.exe` loading `MemLoader.dll` from user-writable paths and scheduled-task creation immediately after fake security-software installer execution.
- Hunt for fake Webex/camera-error pages that redirect users to script downloads while later opening a legitimate Webex meeting.
- Monitor for local JSONP/JSONPing behavior where a web page queries a local service to verify whether malware is installed.
- Detect VS Code Remote Tunnel usage and Cloudflare Quick Tunnel execution on endpoints where developers or administrators do not normally use those features.
- Kaspersky published no YARA/Sigma in the accessible text; use IOC hash/domain blocking and behavior-based detections above.

### Gogs Rebase RCE

- Until patched, set `DISABLE_REGISTRATION = true` and `MAX_CREATION_LIMIT = 0` in `app.ini`; audit all repositories with rebase merging enabled.
- Disable "Rebase before merging" globally where operationally feasible and review all newly created accounts/repos since disclosure.
- Monitor Gogs logs for HTTP 500 responses associated with repository creation/deletion, PR creation, and merge/rebase actions.
- Hunt OS process logs for Gogs spawning `git rebase` with attacker-controlled branch names or unexpected child processes.
- Review hosted repository integrity and secrets if an untrusted authenticated account had repo-creation or write privileges.

### Microsoft Public Zero-Day Disclosures

- Prioritize Microsoft Defender, BitLocker, and Windows endpoint telemetry for anomalies tied to recently disclosed PoCs: BlueHammer/CVE-2026-33825, RedSun/CVE-2026-41091, UnDefend/CVE-2026-45498, YellowKey/CVE-2026-45585, GreenPlasma, and MiniPlasma.
- Apply Microsoft mitigations/security updates as released; restrict local admin rights and block untrusted PoC execution in developer/test environments.
- Treat public exploit repositories and mirrors as hostile; monitor EDR for renamed PoC binaries and scripting around Defender disablement/evasion.

## 8. 📊 Trends & Insights

- Endpoint-management platforms are being turned into distribution systems: the FortiClient EMS case shows attackers no longer need independent endpoint access if the management plane can push scripts.
- Developer and crypto organizations remain high-value targets because one compromised workstation can unlock CI/CD, package distribution, wallet, and downstream supply-chain access.
- Legitimate remote-access channels are increasingly central to APT persistence: Kimsuky use of VS Code tunnels, Cloudflare Quick Tunnels, and DWAgent reduces dependence on obvious custom C2.
- Public exploit release cadence is compressing defender timelines. Gogs now has a Metasploit module before a vendor patch, while Microsoft-related uncoordinated disclosures reportedly triggered active exploitation of multiple Windows flaws.
- No new ransomware family with high-confidence last-24-hour technical detail was observed today; ransomware risk is indirect through credential theft, management-plane compromise, and public RCE weaponization.
