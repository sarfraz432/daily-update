# Daily Cybersecurity Intelligence Brief - 2026-05-07

## 1. 🧠 Executive Summary

- **Palo Alto Networks PAN-OS CVE-2026-0300 is under limited in-the-wild exploitation** against internet-exposed User-ID Authentication Portals; unauthenticated attackers can reach root-level RCE on PA-Series and VM-Series firewalls.
- **MuddyWater is masking espionage as Chaos ransomware activity** by using Microsoft Teams social engineering, MFA manipulation, DWAgent/AnyDesk persistence, and a Stagecomp/Darkcomp malware chain with encrypted C2 configuration.
- **CloudZ RAT is abusing Microsoft Phone Link workflows** through the newly documented Pheno plugin to potentially collect SMS/OTP material without compromising the mobile device itself.
- **xlabs_v1 is a newly exposed Mirai-derived DDoS-for-hire botnet** targeting Android/IoT devices with exposed ADB on TCP/5555; Hunt.io recovered C2, token, operator handle, and bandwidth-tiering logic from the malware configuration.
- **X.com review:** public X searches for malware, PAN-OS exploitation, CloudZ/Pheno, and xlabs_v1 were attempted; no independently retrievable, date-valid X-only IOC lead was observed today.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-0300 - Palo Alto PAN-OS User-ID Authentication Portal RCE

- Severity: Critical when exposed to untrusted networks, CVSS 9.3; High when internally restricted, CVSS 8.7.
- Affected systems: PAN-OS on PA-Series and VM-Series firewalls using User-ID Authentication Portal / Captive Portal.
- Affected versions: PAN-OS 12.1 before 12.1.4-h5 / 12.1.7; 11.2 before 11.2.4-h17 / 11.2.7-h13 / 11.2.10-h6 / 11.2.12; 11.1 before 11.1.4-h33 / 11.1.6-h32 / 11.1.7-h6 / 11.1.10-h25 / 11.1.13-h5 / 11.1.15; 10.2 before 10.2.7-h34 / 10.2.10-h36 / 10.2.13-h21 / 10.2.16-h7 / 10.2.18-h6.
- Exploitation status: In-the-wild, limited exploitation reported by Palo Alto Networks against exposed portals.
- Technical root cause: Out-of-bounds write / buffer overflow in User-ID Authentication Portal packet handling, enabling unauthenticated arbitrary code execution as root.
- Patch status: Fixes scheduled by vendor starting 2026-05-13; no full patch available at publication time.

### CVE-2026-22679 - Weaver E-cology Debug API RCE

- Severity: Critical, CVSS 9.8.
- Affected systems: Weaver/Fanwei E-cology 10.0 before 20260312.
- Exploitation status: In-the-wild; still relevant due to exploitation reported from March 2026 and May 5 public coverage.
- Technical root cause: Exposed unauthenticated debug method invocation at `/papi/esearch/data/devops/dubboApi/debug/method`, permitting attacker-controlled Java/Dubbo command execution through `interfaceName` and `methodName` parameters.
- Operational note: Included as carry-forward because it remains an actively exploited enterprise OA RCE; see 2026-05-06 brief for extended IOC set.

### Microsoft Phone Link Abuse by CloudZ RAT / Pheno

- Severity: High.
- Affected systems: Windows 10/11 endpoints with Phone Link present or paired, especially environments where SMS-based OTPs are still accepted.
- Exploitation status: In-the-wild intrusion active since at least January 2026, per Cisco Talos.
- Technical root cause: Not a product CVE; attacker malware accesses synced Phone Link SQLite data and process state after endpoint compromise.
- Impact: Credential theft, browser data theft, Phone Link reconnaissance, and potential OTP interception without implanting the phone.

## 3. 🦠 Malware & Campaign Analysis

### CloudZ RAT with Pheno Phone Link Plugin

- Malware name / family: CloudZ RAT; Pheno plugin.
- Threat actor: Unknown.
- Initial access vector: Undetermined; Talos observed a fake ConnectWise ScreenConnect executable as the initial dropper.
- Execution chain:
  1. Fake ScreenConnect executable executes on the victim host.
  2. Embedded PowerShell sets scheduled-task persistence for a malicious .NET loader.
  3. .NET loader performs hardware/environment checks and deploys CloudZ RAT.
  4. CloudZ decrypts embedded configuration and establishes encrypted socket C2.
  5. Operator issues Base64-encoded commands for metadata collection, browser theft, shell execution, plugin loading, screen recording, and file operations.
  6. Pheno plugin checks for Phone Link state, writes reconnaissance output to `C:\ProgramData\Microsoft\whealth\`, and CloudZ exfiltrates the staged data.
- Persistence technique: Scheduled task executing the malicious .NET loader; plugin staging under `C:\ProgramData\Microsoft\whealth\`.
- C2 communication method: Encrypted socket connection to attacker C2 with Base64-encoded commands.

### MuddyWater / Chaos False-Flag Ransomware Intrusion

- Malware name / family: Stagecomp (`ms_upd.exe`), Darkcomp (`game.exe` / `WebView2.exe`), encrypted config `visualwincomp.txt`; DWAgent and AnyDesk used for remote access.
- Threat actor: MuddyWater / Mango Sandstorm / Seedworm / Static Kitten, Iran-aligned.
- Initial access vector: External Microsoft Teams chat and screen-sharing social engineering; credential harvesting and MFA manipulation.
- Execution chain:
  1. Actor initiates external Teams conversations while impersonating support-style workflows.
  2. Victims are guided through screen sharing and credential entry, including credentials typed into local text files.
  3. Actor uses compromised accounts for discovery, VPN configuration access, and lateral movement.
  4. RDP session downloads `ms_upd.exe` from `172.86.126[.]208` via `curl`.
  5. Stagecomp profiles the host and retrieves `game.exe`, `WebView2Loader.dll`, and `visualwincomp.txt`.
  6. Darkcomp masquerades as Microsoft WebView2, decrypts the config, polls C2 every 60 seconds, and supports command execution, PowerShell, file operations, and interactive shells.
  7. Data is exfiltrated and ransom negotiation occurs, but Rapid7 observed no file encryption in the analyzed intrusion.
- Persistence technique: DWAgent, AnyDesk, RDP access, and bespoke RAT execution.
- C2 communication method: RAT C2 extracted from encrypted `visualwincomp.txt`; Rapid7 reports `uploadfiler[.]com` as C2 and `moonzonet[.]com` as second-stage hosting.

### xlabs_v1 Mirai-Derived ADB Botnet

- Malware name / family: xlabs_v1, Mirai-derived IoT DDoS botnet.
- Threat actor: Unknown; recovered operator handle `Tadashi`.
- Initial access vector: Internet-exposed Android Debug Bridge on TCP/5555 using ADB-shell paste payloads into `/data/local/tmp`.
- Execution chain:
  1. Operator targets Android/IoT hosts with exposed ADB, including Android TV boxes, set-top boxes, smart TVs, residential routers, and IoT-grade ARM hardware.
  2. Bot payload is written into `/data/local/tmp` and launched with an infection-vector tag.
  3. Bot zeroes argv tag, masquerades as `/bin/bash`, daemonizes, and kills competitor bot processes.
  4. ChaCha20 string table decrypts C2 domain, agent tag, operator handle, and authentication token.
  5. Bot registers to C2 and supports 21 DDoS flood variants, including RakNet/Minecraft and OpenVPN-shaped UDP floods.
  6. Bandwidth-profiling routine opens 8,192 parallel TCP sockets to Speedtest infrastructure for 10 seconds and reports Mbps for DDoS-for-hire pricing tiers.
- Persistence technique: No boot persistence observed; fallback re-entry listener opens TCP/26721 with iptables ACCEPT rules if outbound C2 fails.
- C2 communication method: Primary domain `xlabslover[.]lol` resolving to `176.65.139[.]134` over TCP/35342; OpenNIC fallback resolution and victim-side TCP/26721 listener.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "176.65.139[.]134",
    "176.65.139[.]9",
    "176.65.139[.]44",
    "176.65.139[.]42",
    "176.65.139[.]0/24",
    "185.196.10[.]136",
    "77.110.107[.]235",
    "93.123.39[.]127",
    "172.86.126[.]208",
    "116.203.208[.]186"
  ],
  "domains": [
    "xlabslover[.]lol",
    "gate[.]decodo[.]com",
    "pool[.]hashvault[.]pro",
    "calm-wildflower-1349[.]hellohiall[.]workers[.]dev",
    "orange-cell-1353[.]hellohiall[.]workers[.]dev",
    "round-cherry-4418[.]hellohiall[.]workers[.]dev",
    "adm-pulse[.]com",
    "moonzonet[.]com",
    "uploadfiler[.]com",
    "hptqq2o2qjva7lcaaq67w36jihzivkaitkexorauw7b2yul2z6zozpqd[.]onion"
  ],
  "hashes": [
    "65fcd965040fabeb6f092df0a4b6856125018bb3b6a1876342da458139f77dac",
    "ed5de036edbbda52ab0049d2163607038d38a49404a46b6bcfc4bac26b743832",
    "24398b75be2645e6c695e529e62e60deb418143a4bbea13c561d3c361419eb54",
    "5b7284bcf30569ae400e416a62391720cc9081e6047f15816f9d1a04a06eb321",
    "33af554562176eff34598a839051b8e91692b0305edfdbb4d8eb9df0103ffd98",
    "24857fe82f454719cd18bcbe19b0cfa5387bee1022008b7f5f3a8be9f05e4d14",
    "a92d28f1d32e3a9ab7c3691f8bfca8f7586bb0666adbba47eab3e1a8faf7ecc0",
    "1319d474d19eb386841732c728acf0c5fe64aa135101c6ceee1bd0369ecf97b6",
    "3df9dcc45d2a3b1f639e40d47eceeafb229f6d9e7f0adcd8f1731af1563ffb90",
    "c86ab27100f2a2939ac0d4a8af511f0a1a8116ba856100aae03bc2ad6cb0f1e0",
    "a47cd0dc12f0152d8f05b79e5c86bac9231f621db7b0e90a32f87b98b4e82f3a",
    "cd098eddb23f2d2f6c42271ca82803b0d5ac950cb82a9b8ae0928e83945a53df",
    "cf3dfd1d6626fd2129abb7a5983c11827f4b0d497e2dba146a1889bd71f23cd5",
    "a3bac548b5bc91c526b4d6707623ddbd1a675aa952f0d1f9a0aa6f7230f09f23",
    "86e0197389f0573eb83ff53991f337d416124c7c8bd727721ef3d396cd5f65d",
    "bfc1675ee1e358db8356f515aaded7962923e426aa0a0a1c0eddfc4dab053f89"
  ],
  "urls": [
    "hxxps://calm-wildflower-1349[.]hellohiall[.]workers[.]dev/",
    "hxxps://orange-cell-1353[.]hellohiall[.]workers[.]dev/pheno[.]exe",
    "hxxps://round-cherry-4418[.]hellohiall[.]workers[.]dev/?t=1769729309",
    "hxxps://pastebin[.]com/raw/8pYAgF0Z?t=1771833517",
    "hxxps://pastebin[.]com/EBrpRiFi",
    "hxxps://pastebin[.]com/ikjGHALD",
    "hxxps://pastebin[.]com/3jKbe7rN",
    "hxxps://pastebin[.]com/NUrZTmDn",
    "hxxps://pastebin[.]com/RKJcXMAm",
    "hxxps://pastebin[.]com/yUkbaBH3",
    "http://172.86.126[.]208/ms_upd.exe"
  ],
  "mutex": []
}
```

Host artifacts and high-signal strings:

- `C:\ProgramData\Microsoft\whealth\`
- `ms_upd.exe`
- `game.exe`
- `WebView2.exe`
- `WebView2Loader.dll`
- `visualwincomp.txt`
- `dwagent.exe`
- `dwagsvc.exe`
- `dwaglnc.exe`
- `AnyDesk.exe`
- `/data/local/tmp/arm7`
- `/bin/bash` process masquerade on embedded Linux/Android-like hosts
- xlabs_v1 authentication token: `3Wb5WfDEblAOkD0xF3OTPmxWe6cmsR`
- xlabs_v1 operator handle: `Tadashi`
- xlabs_v1 internal codename: `aterna`
- TCP ports: `35342`, `26721`, `24936`, `470`, `1337`, `6767`, `12312`, `25565`, `25567`, `25568`

## 5. 🔗 Technical Resources & Blogs

- The Hacker News: [Mirai-Based xlabs_v1 Botnet Exploits ADB to Hijack IoT Devices for DDoS Attacks](https://thehackernews.com/2026/05/mirai-based-xlabsv1-botnet-exploits-adb.html)
- Hunt.io: [xlabs_v1 DDoS-for-Hire IoT Botnet Exposed](https://hunt.io/blog/xlabs-v1-ddos-for-hire-operation-exposed)
- The Hacker News: [Windows Phone Link Exploited by CloudZ RAT to Steal Credentials and OTPs](https://thehackernews.com/2026/05/windows-phone-link-exploited-by-cloudz.html)
- Cisco Talos: [CloudZ RAT potentially steals OTP messages using Pheno plugin](https://blog.talosintelligence.com/cloudz-pheno-infostealer/)
- Cisco Talos IOCs: [cloudz-pheno-infostealer.txt](https://github.com/Cisco-Talos/IOCs/blob/main/2026/05/cloudz-pheno-infostealer.txt)
- The Hacker News: [MuddyWater Uses Microsoft Teams to Steal Credentials in False Flag Ransomware Attack](https://thehackernews.com/2026/05/muddywater-uses-microsoft-teams-to.html)
- Rapid7: [Muddying the Tracks: The State-Sponsored Shadow Behind Chaos Ransomware](https://www.rapid7.com/blog/post/tr-muddying-tracks-state-sponsored-shadow-behind-chaos-ransomware/)
- The Hacker News: [Palo Alto PAN-OS Flaw Under Active Exploitation Enables Remote Code Execution](https://thehackernews.com/2026/05/palo-alto-pan-os-flaw-under-active.html)
- Palo Alto Networks: [CVE-2026-0300 PAN-OS advisory](https://security.paloaltonetworks.com/CVE-2026-0300)
- Palo Alto Networks mitigation: [Restrict User-ID Authentication Portal access](https://live.paloaltonetworks.com/)

## 6. 🧪 Sandbox / Sample Links

- VirusTotal, Stagecomp: `https://www.virustotal.com/gui/file/24857fe82f454719cd18bcbe19b0cfa5387bee1022008b7f5f3a8be9f05e4d14`
- VirusTotal, Darkcomp: `https://www.virustotal.com/gui/file/1319d474d19eb386841732c728acf0c5fe64aa135101c6ceee1bd0369ecf97b6`
- VirusTotal, CloudZ RAT: `https://www.virustotal.com/gui/file/5b7284bcf30569ae400e416a62391720cc9081e6047f15816f9d1a04a06eb321`
- VirusTotal, Pheno plugin: `https://www.virustotal.com/gui/file/33af554562176eff34598a839051b8e91692b0305edfdbb4d8eb9df0103ffd98`
- Hunt.io infrastructure pivots are embedded in the Hunt.io xlabs_v1 writeup.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.

## 7. 🛡️ Detection & Mitigation

### Palo Alto PAN-OS CVE-2026-0300

- Immediately restrict User-ID Authentication Portal access to trusted internal zones only.
- Disable Response Pages in the Interface Management Profile attached to L3 interfaces where untrusted/internet traffic can ingress; keep Response Pages only where legitimate trusted users ingress.
- Disable User-ID Authentication Portal if not operationally required.
- Enable Palo Alto Threat ID `510019` from Applications and Threats content version `9097-10022` where Threat Prevention is licensed; decoder support requires PAN-OS 11.1 or later.
- Prioritize internet-exposed PA-Series / VM-Series firewalls for isolation until the scheduled vendor fixes are applied.

### CloudZ / Pheno

- Hunt for creation/execution under `C:\ProgramData\Microsoft\whealth\`, fake ScreenConnect binaries, and scheduled tasks launching unsigned or anomalous .NET loaders.
- Alert on suspicious access to Phone Link data stores and process reconnaissance from non-Microsoft binaries.
- Cisco coverage: ClamAV `Win.Packed.Msilheracles-10030690-0`, `Win.Trojan.CloudZRAT-10059935-0`, `Win.Trojan.CloudZRAT-10059959-0`; Snort 2 SIDs `66409`, `66410`, `66408`; Snort 3 SIDs `301492`, `66408`.
- Disable or restrict Phone Link on endpoints handling privileged workflows or SMS OTPs; prefer phishing-resistant MFA over SMS/OTP.

### MuddyWater / Chaos False Flag

- Block external Teams chats unless business-justified; at minimum monitor first-contact external Teams sessions followed by screen sharing, Quick Assist, AnyDesk, DWAgent, or RDP.
- Hunt for `curl` downloads of `ms_upd.exe`, `visualwincomp.txt`, `game.exe`, and unexpected WebView2 sample binaries.
- Detect RAT behavior: 60-second C2 polling, encrypted config reads, spawned `cmd.exe` / PowerShell from WebView2-looking processes, and remote management tool installation after Teams social engineering.
- Reset credentials and revoke sessions for users who participated in suspicious Teams screen-sharing events; review VPN configuration access and cloud session token use.

### xlabs_v1

- Remove internet exposure for ADB on TCP/5555; enforce host firewall blocks on Android TV, set-top box, smart TV, and embedded Linux networks.
- Hunt for outbound TCP/35342 to `xlabslover[.]lol` / `176.65.139[.]134`, inbound TCP/26721 listeners, and unexpected iptables ACCEPT rules for `26721`.
- Detect the token `3Wb5WfDEblAOkD0xF3OTPmxWe6cmsR` in early TCP payloads; Hunt.io assesses this as a high-specificity registration signature.
- On IoT/Android-like hosts, inspect `/data/local/tmp`, suspicious `/bin/bash` process masquerade, and sudden termination of processes listening on TCP/24936.

## 8. 📊 Trends & Insights

- **Legitimate collaboration surfaces are becoming initial-access infrastructure.** Teams, Phone Link, Quick Assist, ScreenConnect lookalikes, AnyDesk, and DWAgent are being chained to reduce malware visibility and exploit user trust.
- **Ransomware branding is increasingly a deception layer.** MuddyWater’s Chaos-themed activity shows extortion artifacts can mask espionage objectives and long-term persistence.
- **IoT DDoS operators are adding commercial telemetry.** xlabs_v1 bandwidth profiling indicates DDoS-for-hire operators are valuing compromised devices by measured upstream capacity, not just bot count.
- **Configuration artifacts remain high-value.** `visualwincomp.txt`, CloudZ embedded config, and xlabs_v1’s weak ChaCha20 string table all exposed C2 or operational secrets; memory/config extraction should be prioritized during triage.
- **Network-edge zero-days still demand immediate exposure reduction.** CVE-2026-0300 is exploitable before fixes are generally available, making access control and feature disablement the primary defense today.
