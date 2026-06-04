# Daily Cybersecurity Intelligence Brief - 2026-06-04

## 1. 🧠 Executive Summary

- CISA-driven urgency increased for actively exploited Android Framework CVE-2025-48595 and Linux kernel CVE-2022-0492; federal remediation deadline is 2026-06-05.
- Oracle WebLogic CVE-2024-21182 remains an active exploitation risk with a 2026-06-04 federal deadline and likely internet-facing enterprise blast radius.
- Kaspersky disclosed Argamal, a new RAT hidden in trojanized adult-game packages, with COM hijacking persistence, delayed payload retrieval, UDP/TCP C2, and rich remote-control commands.
- Proofpoint published TA4922 global expansion: Atlas RAT, RomulusLoader, SilentRunLoader, DLL sideloading, RMM delivery, and Chrome credential theft against Japan, Southeast Asia, Europe, and Africa.
- The Hacker News Threat Intelligence review was completed; newest malware/configuration item in that section remains SideCopy/Xeno RAT and Gamaredon/Gamma ecosystem, while the highest-fidelity new malware configuration today is Kaspersky Argamal.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2025-48595 - Android Framework Integer Overflow

- Severity: High, CVSS 8.4
- Affected systems: Android 14, 15, 16, and 16 QPR2 beta builds
- Exploitation status: In-the-wild; Google reported limited targeted exploitation and CISA KEV added 2026-06-02
- Technical root cause: Integer overflow / wraparound enabling local code execution and privilege escalation without user interaction
- Operational note: Prioritize mobile fleets with delayed OEM patch channels; patch levels 2026-06-01 and 2026-06-05 address the issue.

### CVE-2022-0492 - Linux Kernel cgroups v1 Privilege Escalation

- Severity: High
- Affected systems: Linux kernel branches 2.6-4.20 and 5.5-5.17, especially containerized hosts using cgroups v1 with elevated capabilities
- Exploitation status: In-the-wild; added to CISA KEV with remediation due 2026-06-05
- Technical root cause: Insufficient authentication checks in `cgroup_release_agent_write()` allow namespace isolation bypass, local privilege escalation, and potential container escape to host root
- Operational note: Highest priority for Kubernetes/container hosts, CI runners, multi-tenant Linux servers, and legacy cloud images.

### CVE-2024-21182 - Oracle WebLogic Server

- Severity: High, CVSS 7.5
- Affected systems: Oracle WebLogic Server deployments exposed over T3/IIOP; patched by Oracle in July 2024
- Exploitation status: In-the-wild; CISA KEV addition reported 2026-06-02, federal due date 2026-06-04
- Technical root cause: Unspecified unauthenticated network-accessible vulnerability allowing compromise of susceptible WebLogic servers
- Operational note: No public exploit-chain telemetry was observed today; historical WebLogic exploitation commonly leads to botnets, cryptomining, web shells, and ransomware staging.

## 3. 🦠 Malware & Campaign Analysis

### Argamal RAT

- Malware name / family: Argamal RAT, detected as `Trojan.Win32.Termixia.*`, `Trojan.Win32.Agent.*`, `HEUR:Trojan.Win32.Argamal.gen`, `HEUR:Trojan-Downloader.Win32.Argamal.gen`
- Threat actor: Unknown; Kaspersky assesses with medium confidence that the downloader-chain developer is Spanish-speaking
- Initial access vector: Trojanized adult-game archives distributed via dedicated websites, PixelDrain redirects, AniRena/torrent trackers, and gaming-forum cheat lures
- Execution chain:
  1. Victim launches a bundled game containing a modified `ffmpeg.dll`.
  2. Modified DLL imports `DllGetClassObject` from `natives2_blob.bin`.
  3. `natives2_blob.bin` executes Base64 PowerShell Stage1.
  4. Stage1 checks for Sandboxie/Procmon64, sets `MI_V`/`MI_V2`, writes COM registry staging keys, and schedules Stage2 to execute three days later.
  5. Stage2 downloads encrypted `zaesdl.dat` from GitHub using `bitsadmin.exe`, saves it as `settings.dat`, decrypts with AES-CBC key/IV `zbcd1j9234r670eh`, then installs the decrypted DLL as a COM object.
  6. RAT sends UDP heartbeats and can switch to extended TCP RAT mode.
- Persistence technique: COM hijacking via `HKCU\SOFTWARE\Classes\CLSID\{B210D694-C8DF-490D-9576-9E20CDBC20BD}\InprocServer32`, triggered by `\Microsoft\Windows\WindowsColorSystem\Calibration`; temporary staging via `{722D0F89-B69C-4700-AE8C-4A44350E4876}`.
- C2 communication method: UDP heartbeats to port 57441; payload update via UDP 63559; extended mode over TCP 3747 with substitution-cipher encrypted commands. Current C2 includes `Winst0[.]kozow[.]com` and `asper1[.]freeddns[.]org` resolving to `186[.]158.223.35`.

### TA4922 Atlas RAT / RomulusLoader / SilentRunLoader

- Malware name / family: Atlas RAT, RomulusLoader, SilentRunLoader, ValleyRAT/Winos4.0 ecosystem
- Threat actor: TA4922, suspected Chinese-speaking financially motivated cluster; overlaps partially with Silver Fox / Void Arachne reporting
- Initial access vector: Localized phishing emails using HR, payroll, tax, invoicing, benefits, and compliance lures; GoFile, LimeWire, MediaFire, and shortened URLs
- Execution chain:
  1. Recipient opens ZIP/RAR/IMG-hosted executable plus malicious DLL.
  2. Legitimate executable side-loads malicious DLL (`libcef.dll`, `vulkan-1.dll`, `teamspeak_control.dll`).
  3. Atlas RAT loader performs anti-sandbox checks and executes shellcode using direct syscalls via SysWhispers.
  4. Shellcode decodes C2 and retrieves Atlas RAT core; core writes hex-encoded config to Documents and connects over TCP 886.
  5. RomulusLoader retrieves follow-on tooling, including AnyDesk or SyncFuture RMM, from first-stage infrastructure.
  6. SilentRunLoader steals Chrome credentials/cookies/browsing data and exfiltrates via HTTP POST.
- Persistence technique: Not consistently observed in public reporting; durable access is achieved through RAT/RMM deployment and modular plugin loading.
- C2 communication method: Atlas RAT TCP 886 with ChaCha-encrypted system information; RomulusLoader TCP 1234/first-stage payload hosting; SilentRunLoader HTTP POST to `ws[.]ztts88[.]cyou/upload[.]php`.

### SideCopy Operation XENOFISCAL / Xeno RAT

- Malware name / family: Xeno RAT 1.8.7
- Threat actor: SideCopy under Transparent Tribe/APT36 umbrella
- Initial access vector: Spear-phishing ZIP with Pashto-language LNK targeting Afghanistan Ministry of Finance and provincial finance/revenue entities
- Execution chain:
  1. Victim opens malicious LNK from ZIP.
  2. LNK launches `mshta.exe`.
  3. `mshta.exe` fetches remote HTA from a compromised Afghan education domain.
  4. HTA runs obfuscated JavaScript in memory.
  5. DLL-based loader drops Xeno RAT and decoy document.
- Persistence technique: Registry persistence masquerading as Microsoft Edge; Xeno RAT can also launch via scheduled task.
- C2 communication method: TCP-based RAT server; supports SOCKS5 tunneling, external DLL modules, file operations, keylogging, screenshot, clipboard, webcam/microphone, and persistence removal.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "186.158.223.35",
    "181.116.218.56",
    "206.238.115.58",
    "154.211.86.110",
    "43.156.77.97",
    "103.214.172.33",
    "18.139.83.110"
  ],
  "domains": [
    "asper1.freeddns.org",
    "Winst0.kozow.com",
    "country1.ignorelist.com",
    "ws.ztts88.cyou",
    "nwphotoblog.com"
  ],
  "hashes": [
    "42add9475e67a1ccc6a6af94b5475d3defc01b85",
    "edce72f59e4c1d136cd1946af70d334c19df858d",
    "76253fb55aed707440e808ea78e7101318436b1c",
    "1405a3c5e0aeb08012484134e16cdec4ab29b4a4",
    "535f4337f261b6da20a3c614eb13270bed2d533a",
    "d2cb0d7a9ad2b5d4ea7c2da8aec62beb37cf36d6",
    "e05f1767c2a337910ed75e90288838d6d0541164",
    "dad26f61da7b8bccc78364411812be74c025b475",
    "29f1d346a6e71774c7dad25b90f446b2974393df",
    "e815a9b418d09c2d4bcd074c2c0bc21406eeb22f",
    "17f8f8f34dfa737f36182fed7ff9e9814a114058",
    "954722b0c9c678b1313d1f8b204e102842dc5889",
    "69331cfdac792dc79240e6a6bb6e803eabd70beb",
    "901cfa97b1baaf908fd4a02bb52d970f576c4193",
    "5f1f3689bcf23de1b280b5f35712946da0f7978f",
    "c2d9d48b3b10bd58cdf5df9463e3ffcd60533ff3",
    "2423a5bf0fa7cb9ec09211630a5488629499691b",
    "ae4601a19d28332a3ec6ac31b385cdf53be53450",
    "9803604ec45f31f9ef75bcca1e1310d8ac1fc3a6",
    "02819d200d1424882af81cb504b3e8614b32397a",
    "a648db354820ea4d02940cb1702b35974513b7aae83f6dffaacaac4ba31f9295",
    "584a9448dda46bd590d7a2f86228100d2ae6e0d6d990c1a4459ed5ee28e07ae8",
    "66a3836b9a17771bce2161f6b73cbc2494a91e49d6aa30d2d53711e8d10de60d",
    "4fcfa88fffacbce30bbe2136753c9ab5a4c092940d2406fd9d44d5118e745b9d",
    "a75eab31d7ff06b6864960ad7e633be3f9730ff3d3873e4539c8f425fc632dad",
    "40b41979b317406f8abc601677a3b93aaf6ef8ab8ac188b8f383735e388f13b5",
    "8c9b6542f73c5c7fe455b52f5101314407da4f65ff48e7ebf6896605e607c8d0",
    "3119cf37b8267db8a2dcd11d9a83d5237d7ef1e42388e7c9afa2831b91da8a2d",
    "314f4b59535d1b783e1c20c2be00f9e30f8ed27b2e21fad06a73b47ea43279ef",
    "2d2a251a88632f010fd9671789746908eeccaa5bc5c0a5d25e4649efe4f5b15d",
    "0857148fb0bc4aa7adf967ede2307bdb4fc427065d5b6a6db132688a5a8e1eb8",
    "e0a6a71c605d9a4076147e9537f82f79f1e1eccadc874595160aa4637ff4088c",
    "de82998ad5fcd63deae030803388e0fb4290d6223fda82368fd25b99b823f0d2",
    "9d0a55c545c4147956db2c2667c4ed931a2875309147548b1dfdd216228f5f73"
  ],
  "urls": [
    "hxxps://github[.]com/gmz159/u",
    "hxxps://github[.]com/DnyP/files",
    "hxxps://github[.]com/mgzv/p",
    "hxxps://ws.ztts88[.]cyou/file/cg[.]exe",
    "hxxps://ws.ztts88[.]cyou/upload[.]php",
    "hxxps://nwphotoblog[.]com"
  ],
  "mutex": [
    "Not observed today"
  ]
}
```

## 5. 🔗 Technical Resources & Blogs

- The Hacker News Threat Intelligence page: https://thehackernews.com/search/label/Threat%20Intelligence
- THN - SideCopy / Xeno RAT: https://thehackernews.com/2026/06/pakistan-linked-sidecopy-targets.html
- THN - Gamaredon GammaPhish/GammaLoad/GammaWorm/GammaSteel: https://thehackernews.com/2026/06/gamaredon-exploits-winrar-to-deliver.html
- THN - Oracle WebLogic CVE-2024-21182 KEV: https://thehackernews.com/2026/06/oracle-weblogic-cve-2024-21182-added-to.html
- THN - Miasma / Mini Shai-Hulud supply chain worm: https://thehackernews.com/2026/06/miasma-supply-chain-attack-compromises.html
- Kaspersky Securelist - Argamal RAT technical analysis: https://securelist.com/argamal-rat-distributed-with-hentai-games/119999/
- Kaspersky press release - Argamal: https://www.kaspersky.com/about/press-releases/kaspersky-discovers-argamal-a-new-malware-hidden-in-games-for-adults
- Proofpoint - TA4922 global campaign: https://www.proofpoint.com/us/blog/threat-insight/ta4922-suspected-chinese-crime-group-going-global
- BleepingComputer - CISA Android/Linux active exploitation: https://www.bleepingcomputer.com/news/security/cisa-warns-of-active-attacks-exploiting-android-linux-bugs/
- NVD - CVE-2025-48595: https://nvd.nist.gov/vuln/detail/CVE-2025-48595
- Android Security Bulletin 2026-06-01: https://source.android.com/docs/security/bulletin/2026/2026-06-01
- X.com review: Direct X rendering was inaccessible/blank for source-linked posts reviewed today; no independently verifiable tweet-only IOCs were recovered.

## 6. 🧪 Sandbox / Sample Links

- Kaspersky OpenTIP Argamal samples are linked from the Securelist IOC section:
  - https://opentip.kaspersky.com/76253fb55aed707440e808ea78e7101318436b1c
  - https://opentip.kaspersky.com/edce72f59e4c1d136cd1946af70d334c19df858d
- VirusTotal: Not observed today.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.

## 7. 🛡️ Detection & Mitigation

- Argamal YARA/Sigma: Not observed today.
- Argamal detection ideas:
  - Alert on user-writable COM hijack writes to `HKCU\SOFTWARE\Classes\CLSID\{B210D694-C8DF-490D-9576-9E20CDBC20BD}\InprocServer32`.
  - Hunt for `MI_V` or `MI_V2` user environment variables containing Base64 PowerShell.
  - Hunt for `bitsadmin.exe` retrieving `zaesdl.dat` from GitHub and saving `settings.dat` under random `%LOCALAPPDATA%` subdirectories.
  - Detect UDP egress to ports 57441 and 63559, and TCP 3747 to Argamal C2 domains/IPs.
  - Alert when games load modified `ffmpeg.dll` plus `natives2_blob.bin` from user download/game directories.
- TA4922 detection ideas:
  - Alert on side-loaded DLL names `libcef.dll`, `vulkan-1.dll`, `teamspeak_control.dll` adjacent to recently downloaded ZIP/RAR/IMG payloads.
  - Hunt for Atlas RAT TCP 886 traffic to `206.238.115.58` or `154.211.86.110` and the loader check-in string `SFuck` followed by null bytes.
  - Monitor Chrome credential-store access followed by HTTP POST to `ws.ztts88.cyou/upload.php`.
  - Block or heavily inspect unsolicited GoFile, LimeWire, MediaFire, `srt.tw`, and payroll/tax-themed download flows.
- CVE-2025-48595 mitigation: Deploy Android 2026-06-01/2026-06-05 patch levels; prioritize devices in executive, government, journalism, defense, and high-risk mobile user groups.
- CVE-2022-0492 mitigation: Upgrade Linux kernels to fixed branches; disable cgroups v1 where possible; remove unnecessary container capabilities; audit privileged containers and CI runners.
- CVE-2024-21182 mitigation: Confirm Oracle July 2024 CPU or later; restrict T3/IIOP exposure; inspect WebLogic access logs for anomalous unauthenticated traffic and post-exploitation web shells.

## 8. 📊 Trends & Insights

- Malware operators are leaning into delayed execution and user-context persistence: Argamal’s three-day scheduled delay plus COM hijacking reduces immediate sandbox visibility.
- Supply-chain and developer-tool compromise remains a dominant pattern: Miasma, GitHub workflow abuse, Claude/VS Code persistence artifacts, and cloud identity collectors show adversaries moving from secret theft to identity graph expansion.
- Chinese-speaking criminal tooling continues to globalize: TA4922 moved from regional East Asia targeting to Europe/Africa with localized lures and a broader RAT/loader/RMM stack.
- Active exploitation pressure is shifting toward privilege-escalation bugs that amplify post-compromise access: Android Framework and Linux cgroups v1 flaws are not remote entry points, but materially improve implant durability and container breakout potential.
- Tweet-only enrichment was not observed today due to X access/rendering limits; source-linked X references around Miasma were reviewed but did not expose unique extractable indicators.
