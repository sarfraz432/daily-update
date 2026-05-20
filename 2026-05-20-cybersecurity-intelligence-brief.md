# Daily Cybersecurity Intelligence Brief - 2026-05-20

## 1. 🧠 Executive Summary

- Mini Shai-Hulud returned on 2026-05-19 with a high-speed npm supply-chain wave affecting 323+ legitimate packages, primarily the AntV visualization ecosystem and adjacent high-download packages.
- Developer workstations, CI/CD runners, cloud environments, and AI coding tool configurations are the primary targets; the payload steals secrets, persists through Claude/Codex/VS Code hooks, and can self-propagate with npm/OIDC credentials.
- ChromaDB Python FastAPI deployments expose a pre-auth RCE path via attacker-controlled Hugging Face model loading; 73% of internet-exposed Chroma instances observed by HiddenLayer were in the vulnerable version range.
- Microsoft disrupted Fox Tempest, a malware-signing-as-a-service operation enabling signed Oyster, Lumma, Vidar, Rhysida, Akira, INC, Qilin, and BlackByte payload delivery.
- X.com review: direct X search/status rendering was unreliable in this environment; relevant X-linked intelligence observed through reporting includes Microsoft-shared Mini Shai-Hulud endpoint `t.m-kosche[.]com`.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-45829 - ChromaDB Python FastAPI Pre-Auth RCE

- Severity: Critical
- Affected systems: ChromaDB Python FastAPI server introduced in 1.0.0 and reported unpatched as of 1.5.8; Rust frontend not affected.
- Exploitation status: Public technical disclosure with working demo; in-the-wild exploitation not confirmed today.
- Technical root cause: Authentication-ordering flaw plus unsafe dynamic model loading. The `create_collection` path instantiates client-supplied embedding function settings and can load a Hugging Face model with `trust_remote_code: true` before authentication is enforced.
- Impact: Unauthenticated HTTP attacker can execute code in the ChromaDB server process, exposing environment variables, API keys, mounted secrets, and local vector-store data.
- Mitigation: Prefer the Rust deployment path, block public access to Python FastAPI Chroma ports, restrict to trusted clients, scan model artifacts, and monitor for outbound Hugging Face fetches immediately preceding failed unauthenticated collection creation requests.
- Source: HiddenLayer, "ChromaToast Served Pre-Auth" - https://www.hiddenlayer.com/research/chromatoast-served-pre-auth

### Mini Shai-Hulud / AntV npm Supply-Chain Compromise

- Severity: Critical
- Affected systems: npm projects installing compromised `@antv/*`, `size-sensor`, `echarts-for-react`, `timeago.js`, `canvas-nest.js`, and related packages published by compromised maintainer accounts between 2026-05-19 01:39 and ~02:56 UTC.
- Exploitation status: Active supply-chain compromise; malicious package versions were published to npm and consumed by developer/CI environments.
- Technical root cause: Compromised npm maintainer credentials plus malicious lifecycle scripts, Bun-based payload execution, OIDC-to-npm token exchange, Sigstore provenance abuse, GitHub dead-drop exfiltration, and CI workflow injection.
- Impact: Theft of GitHub, npm, cloud, Kubernetes, Vault, Docker, database, SSH, CI/CD, and AI-tool credentials; persistence through local hooks; worm-like republishing to additional packages.
- Mitigation: Remove persistence before revoking tokens, reinstall from known-good pre-May 19 versions with `--ignore-scripts`, rotate npm/GitHub/cloud/SSH credentials, audit GitHub repos/workflows, and inspect lockfiles for affected versions.
- Sources: BleepingComputer - https://www.bleepingcomputer.com/news/security/new-shai-hulud-malware-wave-compromises-600-npm-packages/; JFrog - https://research.jfrog.com/post/shai-hulud-here-we-go-again-may19/; Snyk - https://snyk.io/blog/mini-shai-hulud-antv-npm-supply-chain-attack/; Cremit - https://www.cremit.io/blog/antv-mini-shai-hulud-2026

### Fox Tempest Malware-Signing-as-a-Service

- Severity: High
- Affected systems: Windows environments exposed to signed malware installers impersonating Teams, AnyDesk, PuTTY, Webex, VPN clients, and similar enterprise software brands.
- Exploitation status: In-the-wild ecosystem enabler; Microsoft reports ransomware and malware operators used Fox Tempest-signed payloads in active intrusions.
- Technical root cause: Abuse of Microsoft Artifact Signing to obtain short-lived fraudulent certificates, making malicious binaries appear trusted for approximately 72 hours.
- Impact: Increased execution success and control bypass for Oyster, Lumma Stealer, Vidar, Rhysida, INC, Qilin, BlackByte, Akira-linked activity, and other ransomware operations.
- Mitigation: Ensure Microsoft Defender cloud-delivered protection and SmartScreen coverage, hunt for Fox Tempest certificate hashes, and treat short-lived Microsoft-issued signatures on unexpected installers as suspicious.
- Source: Microsoft Threat Intelligence - https://www.microsoft.com/en-us/security/blog/2026/05/19/exposing-fox-tempest-a-malware-signing-service-operation/

### Storm-2949 Microsoft 365 / Azure Data Theft

- Severity: High
- Affected systems: Microsoft Entra ID, Microsoft 365, OneDrive, SharePoint, Azure subscriptions, Azure App Services, Key Vaults, SQL databases, and production cloud assets tied to privileged identities.
- Exploitation status: In-the-wild cloud intrusion campaign.
- Technical root cause: Social engineering of privileged users into approving Self-Service Password Reset/MFA flows, followed by password reset, MFA removal, attacker authenticator enrollment, Graph API enumeration, and Azure RBAC abuse.
- Impact: Large-scale file theft, credential discovery, production Azure asset enumeration, and possible pivot from cloud to endpoint network using VPN and operational files.
- Mitigation: Restrict SSPR for privileged roles, require phishing-resistant MFA for administrators, alert on MFA method removal/enrollment, audit Graph API enumeration spikes, and review OneDrive/SharePoint bulk downloads.
- Source: Microsoft/BleepingComputer reporting - https://www.bleepingcomputer.com/news/security/microsoft-self-service-password-reset-abused-in-azure-data-theft-attacks/

## 3. 🦠 Malware & Campaign Analysis

### Mini Shai-Hulud - AntV / npm / PyPI Wave

- Malware name / family: Mini Shai-Hulud / Shai-Hulud "Here We Go Again" variant.
- Threat actor: TeamPCP attribution is reported by Snyk; JFrog notes attribution is complicated because Shai-Hulud code leaked and copycat use is active.
- Initial access vector: Compromised npm maintainer account `atool` and additional publisher accounts; later related PyPI compromise observed in `durabletask` 1.4.1-1.4.3.
- Execution chain:
  1. Attacker publishes legitimate packages with root-level obfuscated `index.js` payload and `package.json` `preinstall` hook: `bun run index.js`.
  2. Fallback execution path uses `optionalDependencies` pointing to `github:antvis/G2#1916faa365f2788b6e193514872d51a242876569`.
  3. Payload daemonizes, enumerates developer/CI/cloud secrets, and queries cloud metadata services.
  4. Stolen data is serialized, compressed, AES-256-GCM encrypted, RSA-OAEP wrapped, and exfiltrated via fake OpenTelemetry endpoint and/or GitHub dead-drop repositories.
  5. In CI, payload exchanges GitHub Actions OIDC material for npm publish tokens and republishes infected tarballs.
  6. Payload injects GitHub workflow `.github/workflows/codeql.yml` on branch `chore/add-codeql-static-analysis` to dump secrets.
- Persistence technique: `gh-token-monitor.sh`, `kitty-monitor` daemon, Claude/Codex `SessionStart` hooks, VS Code `folderOpen` tasks, GitHub commit-search C2, and malicious workflow injection.
- C2 communication method: `t.m-kosche[.]com:443` fake OpenTelemetry endpoint; GitHub dead-drop repositories; Session P2P upload path `filev2.getsession[.]org/file/` reported in related coverage.

### Oyster / Signed Ransomware Delivery via Fox Tempest

- Malware name / family: Oyster/Broomstick backdoor; downstream Rhysida and other ransomware payloads.
- Threat actor: Fox Tempest service operator; Vanilla Tempest and other ransomware affiliates as customers/users.
- Initial access vector: Malvertising, SEO poisoning, and fraudulent download pages for trusted enterprise tools.
- Execution chain:
  1. User searches for legitimate software such as Microsoft Teams.
  2. Purchased ads or poisoned results redirect to attacker-controlled download page.
  3. Victim runs fraudulently signed installer.
  4. Installer deploys Oyster loader/backdoor.
  5. Operator establishes C2, collects host information, and stages additional payloads.
  6. In observed cases, ransomware such as Rhysida follows.
- Persistence technique: Oyster establishes persistent remote access; specific persistence artifact not newly published today.
- C2 communication method: Oyster backdoor C2; exact live C2 endpoints not newly observed today.

### THN Threat Intelligence Malware Review

- Required source reviewed: https://thehackernews.com/search/label/Threat%20Intelligence
- New THN Threat Intelligence malware/configuration post in the last 24 hours: Not observed today.
- Closest recent THN malware/configuration-relevant posts remain outside the strict 24h window and were excluded from primary analysis to preserve recency constraints.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [
    "169.254.169.254",
    "169.254.170.2"
  ],
  "domains": [
    "t.m-kosche.com",
    "filev2.getsession.org",
    "signspace.cloud"
  ],
  "hashes": [
    "a68dd1e6a6e35ec3771e1f94fe796f55dfe65a2b94560516ff4ac189390dfa1c",
    "dc0acb01e3086ea8a9cb144a5f97810d291020ce",
    "7e6d9dac619c04ae1b3c8c0906123e752ed66d63",
    "f0668ce925f36ff7f3359b0ea47e3fa243af13cd6ad9661dfccc9ff79fb4f1cc",
    "11af4566539ad3224e968194c7a9ad7b596460d8f6e423fc62d1ea5fc0724326",
    "f0a6b89ec7eee83274cd484cea526b970a3ef28038799b0a5774bb33c5793b55"
  ],
  "urls": [
    "github:antvis/G2#1916faa365f2788b6e193514872d51a242876569",
    "https://registry.npmjs.org/-/npm/v1/oidc/token/exchange/package/<package-name>",
    "https://thehackernews.com/search/label/Threat%20Intelligence"
  ],
  "mutex": []
}
```

Notes: `169.254.169.254` and `169.254.170.2` are cloud metadata endpoints, included as behavioral indicators when accessed from package-install contexts or non-cloud developer hosts. `dc0ac...` and `7e6d...` are Microsoft-reported SignerSha-1 certificate indicators, not file hashes.

## 5. 🔗 Technical Resources & Blogs

- BleepingComputer - New Shai-Hulud wave: https://www.bleepingcomputer.com/news/security/new-shai-hulud-malware-wave-compromises-600-npm-packages/
- JFrog - Shai-Hulud returns, npm/PyPI analysis: https://research.jfrog.com/post/shai-hulud-here-we-go-again-may19/
- Snyk - Mini Shai-Hulud AntV compromise: https://snyk.io/blog/mini-shai-hulud-antv-npm-supply-chain-attack/
- Cremit - AntV npm compromise timeline and IOCs: https://www.cremit.io/blog/antv-mini-shai-hulud-2026
- HiddenLayer - ChromaToast CVE-2026-45829: https://www.hiddenlayer.com/research/chromatoast-served-pre-auth
- Microsoft - Fox Tempest malware-signing service: https://www.microsoft.com/en-us/security/blog/2026/05/19/exposing-fox-tempest-a-malware-signing-service-operation/
- BleepingComputer - Storm-2949 SSPR/Azure theft: https://www.bleepingcomputer.com/news/security/microsoft-self-service-password-reset-abused-in-azure-data-theft-attacks/
- The Hacker News Threat Intelligence section reviewed: https://thehackernews.com/search/label/Threat%20Intelligence

## 6. 🧪 Sandbox / Sample Links

- VirusTotal links: Not observed today for the May 19 Shai-Hulud payloads in reviewed sources.
- Any.Run / Hybrid Analysis links: Not observed today.
- MalwareBazaar sample links: Not observed today.
- Public package/advisory pivots: Snyk advisories and OSV malicious package IDs `MAL-2026-3845` through `MAL-2026-4161` are available via Snyk/OSV references in the sources above.

## 7. 🛡️ Detection & Mitigation

- YARA/Sigma: No authoritative new YARA or Sigma rules observed today for Mini Shai-Hulud, ChromaToast, or Fox Tempest. Use package, process, filesystem, and egress detections below.
- Mini Shai-Hulud behavioral detections:
  - Package metadata: `preinstall` script `bun run index.js`; Bun dependency unexpectedly added to visualization/utility packages; optional dependency `github:antvis/G2#1916faa365f2788b6e193514872d51a242876569`.
  - Payload hash: `a68dd1e6a6e35ec3771e1f94fe796f55dfe65a2b94560516ff4ac189390dfa1c`.
  - Persistence artifacts: `.claude/settings.json` `SessionStart`, `.codex` package hooks, `.vscode/tasks.json` with `folderOpen`, `~/.local/share/kitty/cat.py`, `~/.local/bin/gh-token-monitor.sh`, `kitty-monitor` systemd user service or macOS LaunchAgent `com.user.kitty-monitor`.
  - GitHub artifacts: repos/descriptions containing reversed `Shai-Hulud: Here We Go Again`, branch `chore/add-codeql-static-analysis`, workflow `.github/workflows/codeql.yml`, workflow name `Run Copilot`, artifact `format-results.txt`, commit message `fix: ci`.
  - Network: egress to `t.m-kosche.com:443`, `filev2.getsession.org/file/`, cloud metadata IPs from package-install processes, and GitHub API calls using `python-requests/2.31.0` from unexpected contexts.
- Mini Shai-Hulud response order:
  - Isolate affected hosts first.
  - Stop/remove `gh-token-monitor` and `kitty-monitor` persistence before revoking GitHub tokens because JFrog reports destructive token-invalidation logic.
  - Remove AI/IDE hooks and malicious workflows.
  - Rotate npm, GitHub, cloud, SSH, Vault, Docker, database, and CI/CD secrets.
  - Reinstall dependencies from pre-May 19 known-good versions with `npm install --ignore-scripts`.
- ChromaDB:
  - Block unauthenticated/public access to the Python FastAPI server.
  - Alert on unauthenticated `create_collection` attempts with embedding-function `kwargs` containing `trust_remote_code`.
  - Hunt for Chroma server outbound pulls from Hugging Face followed by HTTP 500 responses.
- Fox Tempest:
  - Hunt for the Microsoft-reported SignerSha-1 values and SHA-256 file hashes.
  - Flag short-lived Microsoft Artifact Signing certificates on unexpected installers.
  - Correlate signed installer execution from ad/SEO referral chains with Oyster/Lumma/Vidar detections.
- Storm-2949:
  - Alert on SSPR initiation for privileged users followed by MFA method removal, password reset, authenticator enrollment, Graph API enumeration, and bulk OneDrive/SharePoint download.
  - Review Azure App Service Kudu/FTP/Web Deploy enablement by newly reset privileged accounts.

## 8. 📊 Trends & Insights

- Supply-chain malware is moving from package poisoning to full developer-environment persistence, with explicit hooks into AI coding tools and IDE task systems.
- Provenance and signing are now attacker targets, not sufficient trust signals: Mini Shai-Hulud abuses OIDC/Sigstore paths, while Fox Tempest abuses Microsoft Artifact Signing.
- Attackers continue to prefer credentials and control-plane access over single-host malware impact: npm tokens, GitHub OIDC, cloud metadata, SSPR, Graph API, Azure RBAC, and code-signing infrastructure all featured today.
- The operational blast radius of one stolen maintainer credential is no longer bounded by one package namespace; worm logic rapidly expands through any publisher token reachable from the infected host.
