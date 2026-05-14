# Daily Cybersecurity Intelligence Brief - 2026-05-14

## 1. 🧠 Executive Summary

- Mini Shai-Hulud remains the highest-impact active campaign: 170+ npm/PyPI packages, valid SLSA provenance abuse, CI/CD identity theft, IDE persistence, and destructive token-revocation logic.
- RubyGems saw fresh registry abuse via GemStuffer: 150+ gems used as an exfiltration/storage channel for scraped U.K. local-government portal content.
- Exim disclosed CVE-2026-45185, a severe GnuTLS/BDAT use-after-free with potential unauthenticated RCE against exposed mail transfer agents.
- cPanel CVE-2026-41940 exploitation continues at scale, with Mr_Rot13 deploying Filemanager backdoor, PHP web shells, SSH keys, credential-stealing JavaScript, and Telegram exfiltration.
- X.com review surfaced Microsoft Security Intelligence activity around the malicious `mistralai` PyPI package; no additional unique last-24-hour malware IOCs were observed beyond primary vendor reporting.

## 2. 🚨 Active Threats & Vulnerabilities

### Mini Shai-Hulud / CVE-2026-45321

- **Severity:** Critical
- **Affected systems:** npm/PyPI consumers; CI/CD runners; GitHub Actions trusted publishing; TanStack, Mistral AI, Guardrails AI, UiPath, OpenSearch, and related developer ecosystems.
- **Exploitation status:** In-the-wild supply-chain compromise; malicious packages were published with valid provenance attestations.
- **Technical root cause:** GitHub Actions trust-boundary failure involving `pull_request_target`, cache poisoning, OIDC token extraction from runner process memory, and repository-level trusted-publisher scoping that allowed attacker code to mint publish tokens.

### CVE-2026-45185 - Exim Dead.Letter

- **Severity:** Critical/High
- **Affected systems:** Exim 4.97 through 4.99.2 built with `USE_GNUTLS=yes`, with STARTTLS and CHUNKING/BDAT exposed.
- **Exploitation status:** PoC-level exploitability reported; in-the-wild exploitation not observed today.
- **Technical root cause:** Use-after-free in BDAT message-body parsing during GnuTLS TLS shutdown; stale callback path can write into freed heap memory and enable code execution.

### CVE-2026-41940 - cPanel & WHM Authentication Bypass

- **Severity:** Critical
- **Affected systems:** Internet-facing cPanel & WHM / WP Squared systems missing April 2026 security updates.
- **Exploitation status:** In-the-wild, mass exploitation; XLab reports 2,000+ attacker source IPs involved globally.
- **Technical root cause:** Authentication/session-management bypass; public technical analyses describe CRLF/newline injection into session/log handling as the exploitation primitive.

### Microsoft May 2026 Patch Tuesday

- **Severity:** Critical
- **Affected systems:** Windows DNS Client, Windows domain controllers/Netlogon, Hyper-V, Dynamics 365 on-premises, Azure-adjacent services, Office Word.
- **Exploitation status:** Microsoft reports none of the 138 patched CVEs are publicly known or under active attack.
- **Technical root cause:** Includes heap-based buffer overflow in Windows DNS (`CVE-2026-41096`), stack-based buffer overflow in Netlogon (`CVE-2026-41089`), Hyper-V use-after-free (`CVE-2026-40402`), and Office Word UAF/type-confusion issues.

## 3. 🦠 Malware & Campaign Analysis

### Mini Shai-Hulud

- **Malware/family:** Mini Shai-Hulud supply-chain worm / credential stealer / conditional wiper.
- **Threat actor:** TeamPCP.
- **Initial access vector:** Compromised CI/CD release workflows and trusted-publishing paths.
- **Execution chain:**
  1. Attacker stages malicious payload in GitHub fork/orphaned commit.
  2. Vulnerable workflow restores poisoned cache or runs attacker-controlled code.
  3. Malware extracts OIDC material and mints package-publishing tokens.
  4. Compromised packages execute `router_init.js`, `tanstack_runner.js`, `setup.mjs`, or Python artifacts during install/import.
  5. Stealer profiles environment, queries cloud/CI metadata, harvests GitHub/npm/cloud/Vault secrets, and exfiltrates encrypted data.
  6. Worm enumerates publishable packages and republishes itself where token permissions allow.
- **Persistence technique:** Claude Code SessionStart hook in `.claude/settings.json`, VS Code `folderOpen` task in `.vscode/tasks.json`, dropped runtime/setup scripts, and `gh-token-monitor` service.
- **C2 communication method:** Session network and GitHub fallback; `filev2.getsession[.]org`, `api.masscan[.]cloud`, `git-tanstack[.]com`, attacker GitHub repositories, and Microsoft-reported `83.142.209[.]194` for the malicious `mistralai` PyPI chain.

### GemStuffer

- **Malware/family:** GemStuffer registry-abuse/exfiltration campaign.
- **Threat actor:** Unknown.
- **Initial access vector:** Ruby scripts packaged as junk RubyGems; no mass developer compromise observed.
- **Execution chain:**
  1. Script captures local execution context.
  2. Fetches U.K. ModernGov council portal pages using Ruby `Net::HTTP`.
  3. Crawls agenda/committee links and stores responses in `lib/result.txt` or `README`.
  4. Builds a valid `.gem` archive locally or in memory.
  5. Pushes the archive to RubyGems with embedded API credentials or direct RubyGems API POST.
  6. Attacker retrieves data later via normal `gem fetch` tooling.
- **Persistence technique:** Not observed today.
- **C2 communication method:** RubyGems as public data-drop/exfiltration channel; `https://rubygems.org/api/v1/gems`.

### Mr_Rot13 / Filemanager

- **Malware/family:** Filemanager cross-platform backdoor plus PHP web shell and credential-stealing JavaScript.
- **Threat actor:** Mr_Rot13.
- **Initial access vector:** Exploitation of cPanel CVE-2026-41940.
- **Execution chain:**
  1. Exploit cPanel/WHM auth bypass for administrative control.
  2. Download Go infector `Update` from `cp.dene[.]de[.]com`.
  3. Change root password to a hardcoded value and plant SSH public key.
  4. Drop `cpanel.py` web shell under `/usr/local/cpanel/cgi-sys/`.
  5. Inject `login.js`/`login.tmpl` credential harvester into the cPanel login path.
  6. Deploy Filemanager via `wpsock[.]com/cpanel/install.sh`.
- **Persistence technique:** SSH authorized key, PHP web shell, modified login template/JavaScript, and Filemanager backdoor.
- **C2 communication method:** `wrned[.]com` credential endpoint hidden with ROT13, `cp.dene[.]de[.]com/collect.php`, and Telegram bot/group exfiltration.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "83.142.209.194",
    "169.254.169.254",
    "169.254.170.2",
    "127.0.0.1:8200"
  ],
  "domains": [
    "api.masscan.cloud",
    "filev2.getsession.org",
    "seed1.getsession.org",
    "git-tanstack.com",
    "vault.svc.cluster.local",
    "rubygems.org",
    "moderngov.lambeth.gov.uk",
    "democracy.wandsworth.gov.uk",
    "moderngov.southwark.gov.uk",
    "wrned.com",
    "cp.dene.de.com",
    "wpsock.com"
  ],
  "hashes": [
    "SHA256:ab4fcadaec49c03278063dd269ea5eef82d24f2124a8e15d7b90f2fa8601266c",
    "SHA256:2ec78d556d696e208927cc503d48e4b5eb56b31abc2870c2ed2e98d6be27fc96",
    "SHA256:7c12d8614c624c70d6dd6fc2ee289332474abaa38f70ebe2cdef064923ca3a9b",
    "MD5:2286f126ab4740ccf2595ad1fa0c615c",
    "MD5:2de27ca8d97124adaf604b18161a441e",
    "MD5:29222f5e73dd10088fcf1204aa21f87f",
    "MD5:fb1bc3f935fdeb3555465070ba2db33c",
    "MD5:9305b4ebbb4d39907cf36b62989a6af3",
    "MD5:45fc93426cf08f91c9f9de5f04a12263",
    "MD5:711afb014f64c97d7b31685709c34ce7",
    "MD5:22613c952459e65ce09fb6b5c1c03d47",
    "MD5:e49f68a363c867608972680799389daf",
    "MD5:e1ec6ebb96cf87c785ee6a7da677c059",
    "MD5:02a5990b11293236e01f174f5999df20",
    "MD5:bae1f1bce7c82fa86f05b12e2e254cfc"
  ],
  "urls": [
    "http://filev2.getsession.org/file/",
    "http://169.254.169.254/latest/meta-data/iam/security-credentials/",
    "http://169.254.170.2",
    "https://registry.npmjs.org/-/npm/v1/tokens",
    "https://rubygems.org/api/v1/gems",
    "https://cp.dene.de.com/cpanel.py",
    "https://cp.dene.de.com/login.js",
    "https://cp.dene.de.com/adminer.php",
    "https://cp.dene.de.com/Update",
    "https://cp.dene.de.com/collect.php",
    "https://wpsock.com/cpanel/install.sh",
    "https://wpsock.com/cpanel/dist/filemanager-linux-386",
    "https://wpsock.com/cpanel/dist/filemanager-linux-amd64",
    "https://wpsock.com/cpanel/dist/filemanager-linux-armv7",
    "https://wpsock.com/cpanel/dist/filemanager-linux-arm64",
    "https://wpsock.com/cpanel/dist/filemanager-windows-386.exe"
  ],
  "mutex": []
}
```

## 5. 🔗 Technical Resources & Blogs

- THN Threat Intelligence feed reviewed: https://thehackernews.com/search/label/Threat%20Intelligence
- THN Mini Shai-Hulud analysis: https://thehackernews.com/2026/05/mini-shai-hulud-worm-compromises.html
- Aikido Mini Shai-Hulud IOCs: https://www.aikido.dev/blog/mini-shai-hulud-is-back-tanstack-compromised
- StepSecurity Mini Shai-Hulud deep dive: https://www.stepsecurity.io/blog/mini-shai-hulud-is-back-a-self-spreading-supply-chain-attack-hits-the-npm-ecosystem
- THN GemStuffer writeup: https://thehackernews.com/2026/05/gemstuffer-abuses-150-rubygems-to.html
- Socket GemStuffer analysis and package tracker: https://socket.dev/blog/gemstuffer
- THN Exim CVE-2026-45185: https://thehackernews.com/2026/05/new-exim-bdat-vulnerability-exposes.html
- BleepingComputer Exim RCE coverage: https://www.bleepingcomputer.com/news/security/new-critical-exim-mailer-flaw-allows-remote-code-execution/
- THN cPanel/Filemanager coverage: https://thehackernews.com/2026/05/cpanel-cve-2026-41940-under-active.html
- XLab Mr_Rot13 analysis: https://blog.xlab.qianxin.com/mr_rot13-the-elusive-6-year-hacker-group-weaponizing-critical-cpanel-flaws-for-backdoor-deployment/
- THN Microsoft Patch Tuesday: https://thehackernews.com/2026/05/microsoft-patches-138-vulnerabilities.html
- Microsoft Security Intelligence X post referenced by THN for `mistralai` PyPI activity: https://x.com/MsftSecIntel/status/2054041471280423424

## 6. 🧪 Sandbox / Sample Links

- VirusTotal: THN links public VT references for Mr_Rot13 `wrned[.]com` and `helper.php`; no new direct sample pages were independently validated today.
- Any.Run: Not observed today.
- Hybrid Analysis: Not observed today.
- MalwareBazaar: Not observed today.

## 7. 🛡️ Detection & Mitigation

- **Mini Shai-Hulud:** Isolate build hosts and developer systems that installed affected packages; image before revoking npm tokens because destructive logic may trigger on token revocation. Hunt for `router_init.js`, `router_runtime.js`, `tanstack_runner.js`, `.claude/settings.json`, `.vscode/tasks.json`, `.claude/router_runtime.js`, `.claude/setup.mjs`, `.vscode/setup.mjs`, `gh-token-monitor`, and GitHub repos containing `Shai-Hulud: Here We Go Again`.
- **CI/CD controls:** Scope OIDC trusted publishing to protected branches and exact workflow files, disable broad `pull_request_target` trust, pin GitHub Action permissions to least privilege, and alert on package tarball size anomalies plus root-level unexpected payload files.
- **GemStuffer:** Alert on Ruby processes creating `/tmp/gemhome/.gem/credentials`, setting `ENV['HOME']` to `/tmp/gemhome`, invoking `gem build`/`gem push` from temporary directories, or posting `application/octet-stream` archives to `rubygems.org/api/v1/gems` with embedded API keys.
- **Exim:** Upgrade to Exim 4.99.3 immediately where GnuTLS builds expose STARTTLS and BDAT. No complete mitigation is available short of patching; temporary risk reduction is to disable CHUNKING/BDAT or restrict SMTP exposure where operationally possible.
- **cPanel/Filemanager:** Patch cPanel/WHM, then hunt for `/usr/local/cpanel/cgi-sys/cpanel.py`, modified login templates, injected `login.js`, root password changes, `cpanel-updater` SSH key, outbound traffic to `cp.dene[.]de[.]com`, `wpsock[.]com`, `wrned[.]com`, and Telegram bot API use. Rotate credentials only after backdoors are removed.
- **Microsoft Patch Tuesday:** Prioritize exposed DNS/Netlogon paths and domain controllers for `CVE-2026-41096` and `CVE-2026-41089`; validate Hyper-V and Office Word patch coverage separately.

## 8. 📊 Trends & Insights

- Trusted-build compromise is now central to supply-chain intrusion: provenance proves the attacker reached the build pipeline, not that the artifact is benign.
- Package registries are being abused both as execution channels and as data drops, expanding detection needs from dependency install behavior to package publishing behavior.
- Malware operators are increasingly adding defensive punishment logic, such as Mini Shai-Hulud's token-revocation-triggered wipe path.
- Control-panel exploitation remains operationally valuable because one auth bypass can yield hosted-site compromise, database credentials, SSH persistence, and downstream ransomware opportunity.
