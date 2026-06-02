# Daily Cybersecurity Intelligence Brief - 2026-06-02

Reporting window: last 24 hours ending 2026-06-02 11:02 IST / 2026-06-02 05:32 UTC. Sources reviewed include The Hacker News Threat Intelligence section, The Hacker News supply-chain reporting, Snyk, BleepingComputer, Microsoft MSRC, and accessible X.com-indexed security search results. The mandatory The Hacker News Threat Intelligence review surfaced the China-aligned "Dragon Weave" activity with AZUREVEIL malware and AdaptixC2 tradecraft; THN also carried a same-window Miasma supply-chain report with malicious npm configuration behavior.

## 1. 🧠 Executive Summary

- Miasma is today's highest-impact developer supply-chain event: compromised Red Hat-related npm packages executed preinstall malware that harvested credentials and targeted cloud/API secrets from build environments.
- A malicious package impersonating an OpenAI Codex UI component was published to npm and designed to steal Anthropic API keys, indicating active targeting of AI developer workflows and LLM credentials.
- Microsoft confirmed exploitation of CVE-2026-41089, a critical Windows Netlogon RPC RCE, making domain controllers and Windows identity infrastructure a priority patch surface.
- THN Threat Intelligence highlighted Operation Dragon Weave, where China-aligned operators used DLL side-loading, a Rust loader tracked as AZUREVEIL, AdaptixC2 shellcode, and Cloudflare R2 dead-drop infrastructure.
- X.com review showed discussion around the same supply-chain and malware themes, but no independently verifiable tweet-only IOCs were recovered today.

## 2. 🚨 Active Threats & Vulnerabilities

### CVE-2026-41089 - Windows Netlogon Remote Code Execution

- Severity: Critical; CVSS 9.8.
- Affected systems: Microsoft Windows systems exposing the Netlogon Remote Protocol, with highest operational risk on domain controllers.
- Exploitation status: In-the-wild. Microsoft updated the advisory to mark exploitation detected; public reporting on 2026-06-01 amplified the change.
- Technical root cause: RPC protocol handling flaw in Windows Netlogon enabling unauthenticated remote code execution over a reachable network path.
- Impact: Potential domain-controller compromise, credential theft, lateral movement, and enterprise-wide identity takeover if exploited successfully.

### Miasma npm Supply-Chain Compromise

- Severity: High.
- Affected systems: Developer workstations, CI/CD runners, and build environments installing compromised `@redhat-cloud-services/*` npm packages.
- Exploitation status: In-the-wild supply-chain compromise. Malicious package versions were published to npm, then removed after disclosure.
- Technical root cause: Package maintainer/account compromise or publishing pipeline abuse leading to malicious `preinstall` script execution during dependency installation.
- Impact: Theft of environment variables, npm tokens, Git credentials, GitHub/GitLab/Bitbucket tokens, AWS credentials, SSH private keys, and developer-shell history.

### Malicious npm Package Impersonating Codex UI

- Severity: High.
- Affected systems: Developers installing `codexui-android` from npm.
- Exploitation status: In-the-wild malicious package. BleepingComputer reported 1,300+ downloads before removal.
- Technical root cause: Typosquat/brand impersonation and install-time credential exfiltration through package lifecycle hooks.
- Impact: Theft of Anthropic API keys from local developer environments, enabling unauthorized LLM API use and potential downstream exposure of development secrets.

## 3. 🦠 Malware & Campaign Analysis

### Miasma npm Credential Stealer

- Malware name / family: Miasma supply-chain stealer.
- Threat actor: Unknown; Snyk assessed the operation as a targeted npm supply-chain attack against Red Hat cloud-service packages.
- Initial access vector: Installation of trojanized npm package versions under the `@redhat-cloud-services` namespace.
- Execution chain:
  1. Attacker publishes malicious package versions containing a `preinstall` lifecycle script.
  2. `preinstall` executes automatically during dependency installation.
  3. The payload checks host context and collects environment variables.
  4. It searches for `.npmrc`, Git configuration, cloud credentials, SSH private keys, shell history, and source-control tokens.
  5. It fingerprints the host using identifiers such as hostname, username, OS, architecture, and working directory.
  6. Stolen data is packaged for outbound exfiltration.
- Persistence technique: Not observed today; execution is install-time and depends on package installation or CI dependency resolution.
- C2 communication method: HTTP(S) exfiltration to attacker-controlled infrastructure was reported, but stable destination domains/IPs were not exposed in accessible public reporting.

### AZUREVEIL / AdaptixC2 in Operation Dragon Weave

- Malware name / family: AZUREVEIL Rust loader with AdaptixC2 payload delivery.
- Threat actor: China-aligned intrusion activity; specific public attribution remains unresolved in accessible reporting.
- Initial access vector: Spear-phishing and execution of trojanized application bundles using legitimate signed binaries.
- Execution chain:
  1. Victim opens a malicious archive or lure containing a legitimate executable and malicious DLL.
  2. The legitimate binary side-loads the attacker's DLL.
  3. The loader decrypts and launches embedded shellcode.
  4. AZUREVEIL retrieves or stages next payload logic, including AdaptixC2 components.
  5. AdaptixC2 provides interactive command execution and post-exploitation control.
- Persistence technique: Not fully observed in public reporting; campaign tradecraft emphasizes side-loading and memory-resident loader execution over documented persistence.
- C2 communication method: AdaptixC2 over HTTP(S); Cloudflare R2 was used as dead-drop/storage infrastructure for payload retrieval.

### codexui-android npm API-Key Stealer

- Malware name / family: Malicious npm credential stealer; no unique family name published.
- Threat actor: Unknown.
- Initial access vector: Developer installs `codexui-android`, a malicious npm package impersonating an OpenAI Codex UI library.
- Execution chain:
  1. Package is installed from npm.
  2. Install-time code scans the local environment for Anthropic API credentials.
  3. The payload validates and attempts exfiltration of keys.
  4. BleepingComputer reported the package attempted to contact `api.anthropic.com:443/v1/api`, an unusual path that appears designed to blend with legitimate Anthropic API traffic while handling stolen credentials.
- Persistence technique: Not observed today.
- C2 communication method: HTTPS request behavior involving `api.anthropic.com:443/v1/api`; no attacker-owned exfiltration host was published in accessible reporting.

## 4. 🔍 Indicators of Compromise (IOCs)

```json
{
  "ips": [],
  "domains": [
    "api.anthropic.com"
  ],
  "hashes": [],
  "urls": [
    "https://api.anthropic.com:443/v1/api"
  ],
  "mutex": []
}
```

Additional observables: malicious npm package `codexui-android`; compromised npm namespace/package family `@redhat-cloud-services/*`; Azure/Cloudflare R2 dead-drop usage in Dragon Weave. File hashes, attacker IPs, registry keys, and mutex values were not observed today in accessible public reporting.

## 5. 🔗 Technical Resources & Blogs

- The Hacker News Threat Intelligence section: https://thehackernews.com/search/label/Threat%20Intelligence
- THN: China-Aligned Groups Ramp Up Attacks on Tibetan, Uyghur, Taiwanese Groups: https://thehackernews.com/2026/06/china-aligned-groups-ramp-up-attacks-on.html
- THN: Miasma Supply Chain Attack Compromises Red Hat Cloud Services npm Packages: https://thehackernews.com/2026/06/miasma-supply-chain-attack-compromises.html
- Snyk: Miasma supply-chain attack against Red Hat cloud-services npm packages: https://snyk.io/blog/miasma-supply-chain-attack-malicious-code-redhat-cloud-services-npm-packages/
- BleepingComputer: Critical Windows Netlogon remote code execution flaw now exploited in attacks: https://www.bleepingcomputer.com/news/microsoft/critical-windows-netlogon-remote-code-execution-flaw-now-exploited-in-attacks/
- Microsoft MSRC CVE-2026-41089: https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-41089
- BleepingComputer: OpenAI Codex npm package downloads Anthropic API-key stealer: https://www.bleepingcomputer.com/news/security/openai-codex-npm-package-downloads-anthropic-api-key-stealer/

## 6. 🧪 Sandbox / Sample Links

- VirusTotal links: Not observed today.
- Any.Run links: Not observed today.
- Hybrid Analysis links: Not observed today.
- MalwareBazaar sample links: Not observed today.
- GitHub PoC repositories: Not observed today for CVE-2026-41089 in accessible sources.

## 7. 🛡️ Detection & Mitigation

### CVE-2026-41089

- Apply Microsoft security updates for CVE-2026-41089 immediately, prioritizing domain controllers and exposed Windows server segments.
- Restrict Netlogon/RPC reachability to domain-joined administrative and server subnets only; block untrusted inbound access to domain-controller RPC services.
- Hunt for unusual Netlogon RPC activity from non-domain-controller hosts, unauthenticated connection attempts, and domain-controller process crashes or unexpected child processes.
- Treat any suspicious activity on domain controllers as credential-compromise scope; rotate privileged credentials after containment.

### Miasma / npm Supply Chain

- Audit dependency lockfiles and package caches for malicious `@redhat-cloud-services/*` versions installed during the reporting window.
- Rebuild CI runners and developer hosts that installed affected versions; do not rely only on package removal.
- Rotate npm tokens, GitHub/GitLab/Bitbucket tokens, AWS credentials, SSH keys, and service-account secrets exposed to affected build jobs.
- Detection ideas: npm lifecycle script execution spawning shell, curl/wget/node network clients during install, access to `.npmrc`, `.gitconfig`, `.ssh/id_*`, `~/.aws/credentials`, and shell history immediately after `npm install`.
- Sigma/YARA: Not observed today in public reporting.

### Dragon Weave / AZUREVEIL

- Hunt for legitimate signed binaries loading unexpected sibling DLLs from user-writable directories.
- Detect Rust executables or loaders that decrypt embedded shellcode and spawn network connections to cloud object-storage endpoints.
- Monitor for Cloudflare R2 object retrieval followed by in-memory execution, especially where the parent process is a side-loaded desktop utility.
- Block known campaign object URLs if available internally; public reporting did not expose stable public C2 domains or hashes today.

### codexui-android

- Remove `codexui-android` from all package manifests, lockfiles, local npm caches, and artifact repositories.
- Search logs for install-time requests to `https://api.anthropic.com:443/v1/api` and unexpected Anthropic API traffic from CI runners.
- Rotate Anthropic API keys present on any system that installed the package.
- Add package allowlisting or provenance checks for AI/LLM SDK dependencies; flag npm package lifecycle scripts in newly introduced dependencies.

## 8. 📊 Trends & Insights

- Attackers are converging on developer identity and AI workflow credentials: npm install-time execution is being used to harvest source-control, cloud, npm, SSH, and LLM API secrets before code reaches production.
- The same-day appearance of Miasma and the Codex-themed package reinforces that dependency trust and brand impersonation are now part of active credential-theft operations, not only commodity nuisance campaigns.
- APT tradecraft continues to normalize cloud object storage as staging infrastructure; Dragon Weave's Cloudflare R2 dead-drop usage gives defenders fewer classic C2 indicators and shifts detection toward side-loading plus cloud-fetch behavior.
- Identity infrastructure remains exposed to critical RCE risk. CVE-2026-41089 exploitation against Netlogon is strategically severe because domain controllers compress attacker objectives into one compromise point.
- Public sample hashes and sandbox links were sparse today; detection should emphasize behavior, package-install telemetry, network egress during dependency resolution, and domain-controller RPC anomalies.
