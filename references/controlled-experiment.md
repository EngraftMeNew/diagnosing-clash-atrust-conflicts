# Controlled change: decision gate and verification

Read only after the OS-specific A/B route and DNS evidence is collected. Diagnose first. A range change is a *candidate experiment*, not the default repair. Compare candidate pools against full current LAN, organization/VPN, Tailscale and OS routes. `198.19.0.0/16` is inside RFC 2544's `198.18.0.0/15`; do not infer it is free because `198.18.0.0/16` was occupied. Do not casually use RFC1918, CGNAT, documentation, multicast or other special-use space. If no sufficiently defensible candidate exists, report that and stop. If aTrust dynamically captures a new range, stop changing pools and escalate to network administrator/Sangfor compatibility support.

## Proposal before approval

Show, before touching files:

1. Exact active subscription and persistent Merge/Override source; verify current version's composition order and whether app settings or scripts override the target field. Never edit generated `clash-verge.yaml` for persistence.
2. One-field minimal diff, e.g. `dns.fake-ip-range: <current> -> <candidate>` **only if evidence supports it**. No simultaneous DNS mode, TUN, rule, proxy or system-setting changes.
3. An independent, non-overwriting backup filename, its verification method, and the exact restoration/reload steps. Preserve older backups.
4. Pre-change baseline, aTrust-off checks, aTrust-on checks, expected routes, failure thresholds and user-controlled VPN timing.
5. Cache strategy for the installed Mihomo version. Check `profile.store-fake-ip` and cache provenance; distinguish macOS/Windows resolver cache, Mihomo DNS cache, and FakeIP mapping. A hot reload may clone in-memory mappings even when disk persistence is off. Do not delete `cache.db` or silently clear another cache.

Wait for explicit approval for the specific change. Controller enablement, FakeIP flush and aTrust connection are separate decisions; VPN connection remains the user's action.

## Approved experiment

1. Confirm aTrust off, healthy baseline, exact current composed range and active Merge. Create and verify a fresh backup; abort if impossible.
2. Edit only the approved persistent Merge field, activate/reload via normal app mechanism, and verify final composed config and actual TUN address/routes. Do not assume TUN auto-updates; observe it. If not, stop and roll back.
3. If previously known domains still return old FakeIP after a correct reload, capture that evidence. Check version-specific memory/persistent mapping behavior. With separate approval, use the authenticated Mihomo API `POST /cache/fakeip/flush` against an already secure controller; expect HTTP 204 on versions documenting that response. If the controller is unavailable, a separately approved **Restart Core** through Clash Verge is another test, but it interrupts active connections (including SSH forwards) and may not clear mappings persisted to disk. Verify the old hostname receives a new-pool answer afterward. Do not first call `/cache/dns/flush`, clear the OS resolver, or delete a database. Never open a controller broadly as a workaround. A temporary controller must be authenticated, bound to a supported Unix socket or loopback, and later closed if not needed.
4. With aTrust still off, resolve multiple representative existing and new domains; confirm new pool, actual route via Clash TUN, full route migration, ordinary HTTPS and Codex stability. If anything important fails, stop and restore this experiment's backup/reload; report what was and was not restored. Do not launch aTrust.
5. Only after stage 4 passes, ask the user to connect aTrust. Capture the *complete* before/after route tables and normalized diff immediately. Specifically count all aTrust routes overlapping both old and new ranges and the enclosing special-use block. If aTrust injects more-specific routes into the new pool, declare the pool experiment failed; no need to wait for app stability.
6. If no new-pool capture, verify FakeIP routes via Clash, private resource via aTrust, default route, DNS, ordinary HTTPS, Codex for several minutes, and authorized TCP reachability of the SSH host/port. Treat SSH authentication denial as a distinct layer.
7. Report success/failure and current configuration **before** any further change. If requested, test subscription-update persistence and recommend whether to close a temporary controller. Do not test a third pool without a new decision.

## Stop conditions

Stop automatic repair if baseline network, DNS or TUN breaks; old FakeIP persists after an approved FakeIP flush; aTrust follows the candidate pool; backup/rollback cannot be trusted; repair would require permanent OS routes, system DNS, aTrust internal policy, deleting the full cache database, or disabling a security mechanism. Do not conceal an incomplete rollback.
