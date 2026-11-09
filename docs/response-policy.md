# Response Policy

Principle: a wrong action must always be cheap to undo. The agent informs and contains; the user decides.

## Allowed versus forbidden automatic actions

| Action | Policy |
| --- | --- |
| Alert | Always allowed |
| Quarantine | Automatic only at High tier or above; never for allowlisted or system-owned paths; reversible and hash-verified |
| Restore | User-initiated only; hash-verified; non-executable; refuses overwrite unless forced |
| Delete | Never automatic. Purge is an explicit command touching only the quarantine directory |
| Kill | Off by default; requires both a config flag and an action-list entry; refuses PID four and below, self, ancestors, protected names, privileged accounts, PID-reuse mismatches |
| Firewall change | Dry-run by default; execute requires interactive confirmation of the shown plan; snapshot before apply; one-command rollback; dead-man timer option |
| Hardening fix | Same plan-confirm-snapshot-apply-undo cycle |
| Network isolation, account changes, registry or system-file modification | Not supported |

## Confidence tiers

Score is the sum of independent rule scores per event, or highest detection severity from the scanner. Defaults: alert at thirty, respond at seventy.

- **Low** — single weak heuristic. Log only.
- **Medium** — one strong or multiple weak heuristics. Alert only, with evidence and a why.
- **High** — corroborated: two or more independent indicators, or a known-bad hash. Alert plus quarantine of the candidate file only.
- **Critical** — known-bad hash and active behavior. Alert plus quarantine plus a prompt for user-approved kill; kill is never silent.

Rules: a single heuristic family can never reach High alone unless it is a documented exact match. Audit-only staged rules cap at Medium regardless of score. An allowlist match downgrades to log-only but is itself logged.

## Quarantine requirements

Same-filesystem move preferred; otherwise copy, verify SHA-256, remove the original only after verify succeeds. Preserve original path, owner, mode, mtime, detections, rule ID, and alert ID. Quarantine directory is user-owned, mode zero-seven-zero-zero, excluded from scanning and watch, never executed from. Capacity guard refuses quarantine when disk would drop below a floor. Interpreters, system binaries, package-managed files, OS directories, and the agent's own tree are never quarantined.

## Rollback

Quarantine restores by hash. Firewall rolls back from a pre-apply snapshot via fixed in-code command tables, never from editable files. Hardening fixes back up the previous value per fix. Rules back up before import. Config keeps a last-known-good and refuses to start invalid.

## Fail-safes

If scanner or rule load fails, keep the previous ruleset and alert — never fail open silently. If a response action errors, alert with the error and do not retry into a loop. Event storms degrade by dropping low tiers first, never by degrading the user's system. The agent never acts on its own quarantine directory, logs, or config.

## Open items

Hash-chained audit log, user-confirm prompt for kill, quarantine minimum-age purge, dead-man firewall revert, desktop notification channel.
