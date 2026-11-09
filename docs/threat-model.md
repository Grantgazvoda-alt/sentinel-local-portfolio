# Threat Model

Scope: one user, one workstation. Linux first-class; Windows paths exist but are untested on real hardware. Local-only: no cloud, no telemetry, no auto-update. Python user-space daemon — no kernel driver, no ETW/eBPF sensor.

## Assets protected

User documents and backups, credentials and session material (browser profiles, SSH keys, cloud CLI creds, tokens), the dev environment, system integrity (autoruns, cron, systemd units, authorized_keys), compute and bandwidth, and SentinelLocal itself.

## Trust boundaries

1. Internet to host — everything arriving is untrusted.
2. Unprivileged user processes to SentinelLocal — v1 runs as the same user it watches. Same-user malware sits on the same side of the boundary. This is the biggest structural limit and it is stated, not implied away.
3. User to root — elevation is per-command through a fixed-table helper, never permanent.
4. Local files to the scanner — hostile input (archives, scripts, rule files) is parsed; parser bugs are an attack surface.
5. Rule import to active ruleset — operator-supplied files cross into code-equivalent trust.
6. Loopback dashboard to local processes — read-only, Host-header checked.
7. Quarantine directory to the rest of the filesystem — held files are hostile content.

## Adversaries in scope

Commodity droppers and trojans, infostealers, ransomware, malicious scripts and LOLBins, persistence implants, credential theft, malicious browser extensions, and supply-chain or dev-tooling attacks.

Explicitly out of scope: kernel and firmware rootkits, bootkits, targeted nation-state implants, physical access, zero-click chains, malicious insiders with root.

## Key scenarios and residual risk

| Scenario | Mitigation | Residual |
| --- | --- | --- |
| Dropper lands in Downloads | Realtime scan + quarantine + opt-in pre-exec guard | Scans after file settles; guard is opt-in; unknown samples pass |
| Office or web server spawns a shell | Process-tree and cmdline rules | Two-second polling misses short-lived processes |
| Ransomware encrypts documents | FS-burst, entropy, rename, note heuristics | Detects during encryption; some files are lost; backups remain the real defense |
| New persistence | Baseline diff of autorun paths | Misses WMI subscriptions, services, browser-extension persistence, DLL hijack |
| Infostealer reads cookies and keys | Mostly a gap — no file-open telemetry in user space | Partial via exfil-ish network heuristics |
| Malicious package install hook | Cmdline and network heuristics on children | Weak; no package-manager awareness yet |
| Malicious browser extension | Gap in v1 | Candidate: watch extension dirs and manifest permissions |

## Threats to the agent

Tampering with binaries or config, alert and audit log truncation, rule and update poisoning, allowlist abuse, privilege escalation via the agent, parser attacks (zip bombs, ReDoS), resource exhaustion, false-positive weaponization, dashboard abuse via DNS-rebinding, local info disclosure through alert cmdlines, and unsafe self-remediation locking the user out.

Mitigations present: supervisor restart, directory permissions, hash-chained audit log, bounded archive inspection, governor caps, quarantine-only reversible responses, loopback-only dashboard. Mitigations partial or absent: integrity self-check, privilege separation, signed audit anchors, signature verification on rule bundles, secrets redaction in logs.

## Structural limitations, stated plainly

User-space polling gives no kernel telemetry and no tamper protection against same-user or root malware. The signature set is a small hand-written starter — not comparable to a commercial feed. The realistic edge is stability, transparency, low false positives, safe response, and behavioral heuristics. Claims of beating consumer AV on signature coverage would be false, and the project does not make them.
