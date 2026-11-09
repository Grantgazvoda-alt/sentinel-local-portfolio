# Detection Methodology

## Layers

1. **Hash reputation** — SHA-256 lists. Known-bad hashes are never overridden by allowlists.
2. **Byte-pattern signatures** — validated at load: no empty, no short, no ReDoS-prone patterns.
3. **YARA** — optional (yara-python); each rule file compiled separately with its own namespace; a broken file is skipped with an error rather than failing startup. Twenty-six rules across seven categories: self-test, ransomware, script, LOLBin, persistence, webshell, credential access.
4. **Bounded archive inspection** — zip, gzip, tar up to three nesting levels. Past the budget, the result is reported incomplete, never clean.
5. **Streaming scan** — large files are not loaded wholesale; read pacing caps at twenty-five megabytes per second.
6. **Behavioral rules** — sixty-two JSON rules across ransomware, persistence, process trees, and LOLBin command lines. Scores zero to one hundred, built from families of evidence.
7. **Persistence diffing** — autoruns, cron, systemd user units, shell rc files, authorized_keys, ld.so.preload; Windows Run keys, startup folders, scheduled tasks.
8. **Ransomware heuristic** — mass modification plus rename-to-same-extension plus ransom-note plus high-entropy sampling. File-activity-only evidence cannot reach the respond tier.
9. **Pre-execution guard** — opt-in wrapper that scans a program and its script argument before launch and refuses with exit one twenty-six.
10. **Hardening auditor** — CIS-style checks for SSH, sysctl, permissions, accounts, exposed services, mounts, MAC, disk encryption, updates on Linux; firewall, UAC, SMBv1, RDP, Defender, BitLocker, LSA, WDigest on Windows. Guided fixes with undo data holding only id and old value.

## MITRE ATT&CK coverage (honest)

Of the techniques mapped: roughly three to four covered, twenty-five partial, twenty-plus gaps. Strongest areas: ransomware burst behavior, naive reverse shells and download-and-exec one-liners, Linux persistence-file changes, EICAR and test content. Biggest gaps: credential theft via cookies, keys, or LSASS memory access; browser extensions; supply-chain and dev-tooling hooks; exfiltration volume accounting; process injection; kernel-level evasion; anything encrypted or packed.

Windows rows are written from canned-output unit tests only and are unvalidated until run on a real host.

The coverage matrix is generated from rule metadata tagged with ATT&CK IDs, so it stays in sync with the rules rather than drifting as hand-maintained prose.

## Test corpus

- Positive fixtures: EICAR variants assembled in memory at test time (never stored on disk), synthetic strings matching each rule, synthetic file-event streams, synthetic autorun diffs.
- Negative fixtures: realistic benign content — dev-tool builds, package installs, backup and sync churn, office docs, security reading material that mentions credential tools — proving the runner does not false-positive on documentation about attacks.
- Negative controls: twenty-nine deliberately broken copies of the rule pack that the test runner must all catch.
- Hygiene: no EICAR on disk anywhere in the repository; reserved hosts only; no executables in fixtures; every regex timed on adversarial four-kilobyte inputs against a cost budget.

## What is not claimed

No detection rate against real malware families. No production readiness. Thresholds are placeholders to tune. The project says so in its own STATUS.md.
