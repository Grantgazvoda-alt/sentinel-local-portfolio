# SentinelLocal — Portfolio

**Public showcase of engineering work. Implementation remains proprietary.**

SentinelLocal is a defensive, local-only endpoint security suite: real-time file scanning, behavioral monitoring, firewall management, hardening audits, and a resource-governed background service. No cloud, no telemetry, no outbound connections — the only socket it opens is a read-only dashboard on loopback.

## What it demonstrates

- **Honest security engineering** — a full MITRE ATT&CK coverage map that rates roughly three to four techniques as covered, twenty-five as partial, and twenty-plus as gaps. Most security projects publish coverage claims; this one publishes the blind spots next to them.
- **A written threat model** covering assets, trust boundaries, seven adversary classes, and threats to the agent itself — including same-user tampering, rule poisoning, and false-positive weaponization.
- **Safe-response policy by design** — alert-only out of the box. Quarantine is reversible and hash-verified. Kill is off by default with protected-process lists. Firewall changes are dry-run unless an admin explicitly enables an elevated helper behind a marker file. A wrong action must always be cheap to undo.
- **False-positive discipline** — tiered scoring where a single evidence family can never reach the respond tier, allowlists that never override known-bad hashes, planned rule staging (draft → audit-only → enforce), and FP budgets with zero tolerance for quarantining a legitimate file.
- **Production-grade operations** — CPU soft target, RAM soft cap, scan-read pacing, supervised daemon with crash-loop breaker, persisted coverage watermarks so events are never silently dropped, structured rotating JSON logs, and a hash-chained audit log.
- **Signed rule updates** — Ed25519-signed bundles, replay protection, atomic install, rollback. Loose unsigned files run at reduced trust.
- **Verified test surface** — ruff, mypy, pytest including real-daemon integration tests, a real nftables apply-and-rollback test inside an isolated network namespace, negative-control tests that prove the test runner catches deliberately broken rule packs, and a soak test that stresses the daemon through idle, churn, and sixty-files-per-second phases.

## Architecture

See [docs/architecture.md](docs/architecture.md).

## Threat model

See [docs/threat-model.md](docs/threat-model.md).

## Detection methodology

See [docs/detection-methodology.md](docs/detection-methodology.md).

## Response policy

See [docs/response-policy.md](docs/response-policy.md).

## License

See [LICENSE](LICENSE). Implementation source is not included in this public repository.
