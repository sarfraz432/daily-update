# Daily Cybersecurity Intelligence Brief - 2026-05-12

## 1. 🧠 Executive Summary

- AI and model/distribution ecosystems are now an active malware delivery surface: a fake Hugging Face `Open-OSS/privacy-filter` repo delivered a Rust infostealer and reached ~244K downloads before takedown.
- Google Threat Intelligence Group reported the first observed cybercrime zero-day exploit assessed as AI-assisted, targeting 2FA logic in an open-source web administration tool for planned mass exploitation.
- Internet-facing security infrastructure remains a priority target: PAN-OS CVE-2026-0300 exploitation enabled root-level RCE, nginx shellcode injection, log cleanup, and tunneling tool deployment.
- Developer and CI/CD trust chains remain under pressure: TeamPCP returned against Checkmarx via a malicious Jenkins AST plugin after prior KICS, VS Code, GitHub Actions, and package ecosystem compromises.
- X.com review: direct live X search was not reliably retrievable in this environment today; source-linked X artifacts around TeamPCP/Checkmarx were reviewed, but no unique new IOCs were recovered from X.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-0300 - Palo Alto Networks PAN-OS User-ID Authentication Portal RCE

- Severity: Critical
- Affected systems: Palo Alto Networks PAN-OS deployments exposing User-ID Authentication Portal/Captive Portal to untrusted networks.
- Exploitation status: In-the-wild; Unit 42 tracks the activity as CL-STA-1132, a suspected state-sponsored cluster of unknown provenance.
- Technical root cause: Buffer overflow in the User-ID Authentication Portal service enabling unauthenticated remote code execution as root via crafted packets.
- Observed activity: Exploitation achieved RCE, injected shellcode into an nginx worker process, cleared crash artifacts, performed Active Directory enumeration, and deployed EarthWorm and ReverseSocks5.
- Defensive priority: Restrict or disable the User-ID Authentication Portal until patched; fixes were expected to begin May 13, 2026.

### AI-Assisted 2FA Bypass Zero-Day - Open-Source Web Administration Tool

- Severity: High
- Affected systems: Undisclosed popular open-source, web-based system administration tool.
- Exploitation status: Planned mass exploitation disrupted through responsible disclosure; no public CVE or affected vendor name disclosed today.
- Technical root cause: Semantic authorization/2FA logic flaw caused by a hard-coded trust assumption; valid credentials are required before bypass.
- Why it matters: GTIG assesses with high confidence that an AI model supported discovery and weaponization based on code structure, generated-style docstrings, and a hallucinated CVSS score in the exploit script.

### Checkmarx Jenkins AST Plugin Supply-Chain Compromise

- Severity: High
- Affected systems: Checkmarx Jenkins AST plugin users who installed malicious marketplace versions after December 17, 2025.
- Exploitation status: Active supply-chain compromise disclosed by Checkmarx; malicious publication path not fully disclosed.
- Technical root cause: Unauthorized modification/publication of a trusted Jenkins plugin, likely enabled by incomplete credential rotation or retained access after prior TeamPCP activity.
- Defensive priority: Downgrade to `2.0.13-829.vc72453fa_1c16` or earlier if exposed, or validate Checkmarx's fixed `2.0.13-848.v76e89de8a_053` release and rotate CI/CD secrets.

### CVE-2026-22679 - Weaver E-cology Debug API RCE

- Severity: Critical
- Affected systems: Weaver/Fanwei E-cology 10.0 versions prior to build `20260312`.
- Exploitation status: In-the-wild exploitation observed; earliest public reporting places exploitation activity in March 2026 and new analysis remains relevant today.
- Technical root cause: Unauthenticated debug endpoint exposure at `/papi/esearch/data/devops/dubboApi/debug/method`, allowing attacker-controlled `interfaceName` and `methodName` parameters to invoke command-execution helpers.
- Observed activity: RCE checks, failed payload drops, attempted MSI implant `fanwei0324.msi`, PowerShell payload retrieval attempts, and discovery commands including `whoami`, `ipconfig`, and `tasklist`.

## 3. 🦠 Malware & Campaign Analysis

### Rust Infostealer via Fake Hugging Face Privacy Filter

- Malware name / family: Unnamed Rust-based infostealer; infrastructure overlap with ValleyRAT/Winos 4.0 activity.
- Threat actor: Unknown; broader infrastructure linkage suggests possible open-source ecosystem supply-chain operator, with ValleyRAT activity historically attributed to Silver Fox.
- Initial access vector: Typosquatted Hugging Face model repository `Open-OSS/privacy-filter` impersonating OpenAI's legitimate Privacy Filter model; users were instructed to run `start.bat` or `loader.py`.
- Execution chain:
  1. Victim clones the fake Hugging Face repository and runs `loader.py` or `start.bat`.
  2. `loader.py` executes decoy model code, disables SSL verification, decodes `jsonkeeper[.]com/b/AVNNE`, and retrieves a JSON `cmd` field.
  3. Hidden PowerShell downloads `update.bat` from `api[.]eth-fastscan[.]org`.
  4. `update.bat` requests elevation, adds Microsoft Defender exclusions, downloads the Rust payload, and creates an Edge-themed scheduled task.
  5. Scheduled task launches the payload as SYSTEM, then the task and runner artifacts self-delete.
  6. Infostealer collects Chromium/Gecko browser data, Discord tokens/master keys, cryptocurrency wallets, FileZilla/SSH/VPN files, system metadata, and screenshots.
- Persistence technique: No reboot persistence observed; scheduled task used as a one-shot SYSTEM-context launcher.
- C2 communication method: HTTPS/WinHTTP POST of gzip JSON with bearer authorization to `recargapopular[.]com`; dead-drop payload resolver through JSON Keeper.

### PROMPTSPY Android Backdoor AI-Augmented Operations

- Malware name / family: PROMPTSPY
- Threat actor: Not publicly attributed in today's accessible reporting.
- Initial access vector: Not observed today in public reporting.
- Execution chain:
  1. Android backdoor abuses Accessibility API to serialize visible UI hierarchy.
  2. Malware sends UI state and goals to Gemini `2.5-flash-lite` in JSON mode.
  3. Model response returns structured actions and coordinates.
  4. Malware simulates gestures such as clicks and swipes for autonomous device interaction.
  5. Malware can capture PIN/pattern gestures and replay authentication actions.
- Persistence technique: Invisible overlay blocks uninstall button touches; Firebase Cloud Messaging can relaunch the backdoor when inactive.
- C2 communication method: Runtime-rotatable C2 configuration; Gemini API keys and VNC relay server can be updated dynamically through the C2 channel.

### CL-STA-1132 PAN-OS Post-Exploitation

- Malware name / family: EarthWorm, ReverseSocks5, injected shellcode.
- Threat actor: Suspected state-sponsored cluster, unknown provenance.
- Initial access vector: CVE-2026-0300 exploitation against exposed PAN-OS User-ID Authentication Portal.
- Execution chain:
  1. Crafted network traffic triggers unauthenticated RCE.
  2. Actor injects shellcode into an nginx worker process.
  3. Actor deletes crash kernel messages, nginx crash entries, crash records, and core dump files.
  4. Actor performs AD enumeration.
  5. Actor stages EarthWorm and ReverseSocks5 on a second device for tunneling.
- Persistence technique: Not observed today in public reporting.
- C2 communication method: SOCKS/tunneling through EarthWorm and ReverseSocks5.

### TeamPCP Checkmarx Jenkins AST Plugin Compromise

- Malware name / family: Credential-stealing supply-chain malware; exact payload from the Jenkins AST incident not fully disclosed today.
- Threat actor: TeamPCP.
- Initial access vector: Malicious Checkmarx Jenkins AST plugin distributed via Jenkins Marketplace.
- Execution chain:
  1. TeamPCP allegedly obtains or retains access to Checkmarx plugin/repository controls.
  2. Modified plugin version appears in Jenkins Marketplace.
  3. Trusted CI/CD execution context exposes build secrets, tokens, and downstream supply-chain access.
  4. Actor defaces/renames GitHub repository and claims secret rotation failure.
- Persistence technique: Trusted plugin execution inside CI/CD workflows; persistent host mechanism not disclosed.
- C2 communication method: Not observed today.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "89.124.93.110"
  ],
  "domains": [
    "api.eth-fastscan.org",
    "recargapopular.com",
    "welovechinatown.info",
    "jsonkeeper.com"
  ],
  "hashes": [
    "6db01158b044f178c45754666e2cbc0365f394e953fbf99ec34aa5304d5b79b1",
    "6d5b1b7b9b95f2074094632e3962dc21432c2b7dccfbbe2c7d61f724ffcfea7c",
    "c1b59cc25bdc1fe3f3ce8eda06d002dda7cb02dea8c29877b68d04cd089363c7"
  ],
  "urls": [
    "https://thehackernews.com/search/label/Threat%20Intelligence",
    "https://thehackernews.com/2026/05/fake-openai-privacy-filter-repo-hits-1.html",
    "https://www.hiddenlayer.com/research/malware-found-in-trending-hugging-face-repository-open-oss-privacy-filter",
    "https://huggingface.co/Open-OSS/privacy-filter",
    "https://huggingface.co/anthfu/Bonsai-8B-gguf",
    "https://huggingface.co/anthfu/Qwen3.6-35B-A3B-APEX-GGUF",
    "https://huggingface.co/anthfu/DeepSeek-V4-Pro",
    "https://huggingface.co/anthfu/Qwopus-GLM-18B-Merged-GGUF",
    "https://huggingface.co/anthfu/Qwen3.6-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-GGUF",
    "https://huggingface.co/anthfu/supergemma4-26b-uncensored-gguf-v2",
    "https://jsonkeeper.com/b/AVNNE",
    "https://api.eth-fastscan.org/update.bat",
    "https://api.eth-fastscan.org/sefirah"
  ],
  "mutex": []
}
```

Additional host artifacts:

- `%TEMP%\update.bat`
- `%TMP%\runner.ps1`
- `MicrosoftEdgeUpdateTaskCore[a-z0-9]{8}`
- `cmd.exe /k %TEMP%\update.bat`
- `fanwei0324.msi`

## 5. 🔗 Technical Resources & Blogs

- [The Hacker News - Threat Intelligence section](https://thehackernews.com/search/label/Threat%20Intelligence)
- [Fake OpenAI Privacy Filter repo delivers infostealer - The Hacker News](https://thehackernews.com/2026/05/fake-openai-privacy-filter-repo-hits-1.html)
- [HiddenLayer technical analysis of Open-OSS/privacy-filter](https://www.hiddenlayer.com/research/malware-found-in-trending-hugging-face-repository-open-oss-privacy-filter)
- [GTIG AI Threat Tracker - AI-assisted exploitation and PROMPTSPY](https://cloud.google.com/blog/topics/threat-intelligence/ai-vulnerability-exploitation-initial-access)
- [PAN-OS CVE-2026-0300 exploitation - The Hacker News](https://thehackernews.com/2026/05/pan-os-rce-exploit-under-active-use.html)
- [Unit 42 PAN-OS CVE-2026-0300 threat brief](https://unit42.paloaltonetworks.com/captive-portal-zero-day/)
- [TeamPCP Checkmarx Jenkins AST compromise - The Hacker News](https://thehackernews.com/2026/05/teampcp-compromises-checkmarx-jenkins.html)
- [Checkmarx security updates](https://checkmarx.com/blog/ongoing-security-updates/)
- [Weaver E-cology CVE-2026-22679 active exploitation - The Hacker News](https://thehackernews.com/2026/05/weaver-e-cology-rce-flaw-cve-2026-22679.html)
- [Vega analysis of CVE-2026-22679 exploitation](https://blog.vega.io/posts/cve-2026-22679-weaver-ecology-exploitation/)
- [CVE-2026-22679 detection repository](https://github.com/keraattin/CVE-2026-22679)

## 6. 🧪 Sandbox / Sample Links

- VirusTotal: [o0q2l47f.exe](https://www.virustotal.com/) was referenced by public reporting as a related `api[.]eth-fastscan[.]org` payload; direct stable VT object URL was not fully available through accessible source text today.
- Triage: HiddenLayer reports sandbox execution with exfiltration to `recargapopular[.]com`; direct public report URL was not observed today.
- Any.Run / Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.
- GitHub: [CVE-2026-22679 detection script](https://github.com/keraattin/CVE-2026-22679)

## 7. 🛡️ Detection & Mitigation

- Hugging Face infostealer: Hunt for execution of `loader.py` from AI/model repositories, PowerShell launched by Python, `jsonkeeper[.]com/b/AVNNE`, `api[.]eth-fastscan[.]org/update.bat`, `cmd.exe /k` with `%TEMP%\update.bat`, Edge-themed scheduled tasks, Defender exclusions in `%TEMP%`, `%LOCALAPPDATA%`, or `%APPDATA%`, and outbound POSTs to `recargapopular[.]com`.
- Credential response: Treat affected hosts as fully compromised; reimage rather than clean up, rotate browser-stored credentials, cookies, OAuth tokens, SSH keys, FTP credentials, Discord sessions, cloud tokens, and crypto wallet material.
- PAN-OS: Restrict User-ID Authentication Portal to trusted zones or disable it; disable Response Pages on L3 interfaces receiving untrusted ingress; enable Advanced Threat Prevention Threat ID `510019` with current content; hunt for nginx worker anomalies, crash/core cleanup, AD enumeration from firewall networks, EarthWorm, and ReverseSocks5.
- Checkmarx Jenkins AST: Inventory plugin versions, remove suspect versions, rotate Jenkins credentials and secrets accessible to affected jobs, review build logs for unexpected network egress, and validate plugin provenance before re-enabling jobs.
- Weaver E-cology: Patch to build `20260312` or later; block public access to `/papi/esearch/data/devops/dubboApi/debug/method`; hunt for `fanwei0324.msi`, `whoami`, `ipconfig`, `tasklist`, and PowerShell payload retrieval shortly after API POST requests.
- PROMPTSPY: Monitor Android fleets for Accessibility abuse by untrusted apps, invisible overlay behavior during uninstall flows, Firebase Cloud Messaging relaunch patterns, VNC relay configuration changes, and outbound Gemini API use from non-approved mobile applications.
- YARA/Sigma: No vendor-published YARA/Sigma rules were observed today in accessible reporting; detections should be behavioral and egress-led until sample-specific rules mature.

## 8. 📊 Trends & Insights

- Attackers are laundering trust through AI ecosystems. Model cards, inflated likes/downloads, and cloned descriptions are now functioning like package typosquatting for ML users.
- One-shot privilege mechanisms are replacing classic persistence in some stealers. The Hugging Face infostealer used scheduled tasks for SYSTEM execution and deleted them before reboot, reducing persistence artifacts.
- AI is shifting from lure generation to exploit discovery and autonomous malware control. GTIG's 2FA-bypass case and PROMPTSPY show attacker value in semantic reasoning and UI-state interpretation.
- Edge appliances remain high-value espionage footholds because they combine privileged network position with weak endpoint telemetry.
- Supply-chain incident response quality is becoming a key risk factor. The Checkmarx repeat compromise suggests credential rotation and retained-access eradication are now primary containment tests, not cleanup details.
