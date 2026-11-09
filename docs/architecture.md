# Architecture

## Purpose

SentinelLocal protects one workstation. It is not a commercial AV replacement — it is a transparent, controllable extra layer with no cloud dependency.

## Components

```text
sentinel-local/
├── src/sentinel_local/
│   ├── scanner/        # signatures, YARA engine, ruleset, hash reputation
│   ├── behavior/       # process trees, LOLBins, persistence, ransomware, network
│   ├── realtime.py      # file-event watcher, scan-on-close
│   ├── response.py      # tiered response actions
│   ├── quarantine.py    # reversible, hash-verified containment
│   ├── firewall.py      # nftables/ufw/netsh, dry-run default
│   ├── hardening.py     # CIS-style audit + guided fixes
│   ├── governor.py      # CPU/RAM soft caps, I/O pacing
│   ├── supervisor.py    # crash-loop breaker, heartbeat, coverage watermarks
│   ├── daemon.py        # background service
│   ├── dashboard.py     # read-only loopback HTML, no JavaScript
│   ├── rules_update.py  # Ed25519-signed bundle import/rollback
│   ├── guard.py         # pre-execution scan for launched commands
│   ├── simulate.py      # benign EICAR/ransomware/dropper tests
│   └── cli.py          # sentinel command surface
├── detections/       # 26 YARA + 62 behavior rules, MITRE matrix, test corpus
├── design/           # threat model, attack coverage, FP controls, response policy, reviews
├── packaging/        # systemd unit, installers
├── tools/            # soak test, release builder, FP scanner
└── tests/           # unit, integration, negative controls
```

## Data flow

1. A file settles in a watched folder (Downloads, Desktop, Documents, temp).
2. The realtime watcher queues a scan within about one second.
3. The scanner checks SHA-256 reputation, byte signatures, and optional YARA — bounded archive inspection up to three nesting levels, streaming for large files.
4. Concurrently, the behavior monitor polls processes and connections every two to five seconds and diffs persistence locations.
5. Scores accumulate from independent evidence families. Alert threshold is thirty; respond threshold is seventy, and no single family can reach respond alone.
6. At High tier, the candidate file (script or binary in a user-writable path) is quarantined — moved, XOR-defanged, hash-pinned, restorable. Nothing is deleted automatically.
7. Every decision appends to a structured JSON audit log. Elevated actions (firewall, system hardening) require an admin-created marker file and run through a tiny root-owned helper that validates requests against fixed command tables.

## Design decisions

- **User-space, no kernel hooks.** Honest about what polling misses: short-lived processes, kernel-level threats, same-user tampering.
- **Alert-first.** The product informs and contains; the user decides. Destroy and kill are explicit, gated commands.
- **Offline by rule.** No runtime network code. Rule updates are imported as signed bundles over an out-of-band channel.
- **Resource-governed.** Soft CPU target, soft RAM cap, read pacing, pause under system load. The agent must not become the attack.
- **Coverage watermarks.** Events that cannot be queued are re-walked after overflow, trim, or crash — never silently dropped.
- **Windows paths unit-tested with canned output only.** Not validated on a live Windows host. Stated in STATUS.md, not hidden.

## What is deliberately absent

- No cloud reputation, no sandbox detonation, no memory scanning, no AMSI-equivalent hooks.
- No signatures named for specific threat groups — matches are generic, never attribution.
- No secrets, credentials, or customer data in the tree.
