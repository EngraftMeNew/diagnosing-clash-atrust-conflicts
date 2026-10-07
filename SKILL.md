---
name: diagnosing-clash-atrust-conflicts
description: Use when Clash Verge Rev or Mihomo TUN stops working, DNS returns suspicious FakeIP addresses, or Codex and other proxy traffic reconnects or resets while Sangfor aTrust is connected on macOS or Windows.
---

# Diagnose Clash–aTrust conflicts

Diagnose first; modify only after evidence and separate user approval. The default mode is read-only. Do not treat a previous machine's FakeIP range, interface names, route count, or workaround as a universal answer.

## Route to the right procedure

- On macOS, read [macOS diagnostics](references/macos.md).
- On Windows, read [Windows diagnostics](references/windows.md). Never transpose macOS commands or interface names to Windows.
- When considering a change, read [controlled experiment and rollback](references/controlled-experiment.md) and [authoritative sources](references/sources.md).
- Use [the 2026 macOS case](references/case-macos-2026.md) only as an example of evidence and a tested local workaround, not as a default prescription.
- Use [the 2026 Windows case](references/case-windows-2026.md) as an example of a FakeIP-specific failure **without observed overlapping aTrust routes**; do not assume its pool choice applies elsewhere.

## Core diagnostic sequence

1. Confirm the OS, Clash Verge Rev/Mihomo versions, active subscription and composed configuration, DNS mode and FakeIP range. Identify the actual TUN and aTrust interfaces rather than assuming names.
2. With aTrust **off**, record interfaces, full IPv4 route table, default route, DNS, proxy settings, representative FakeIP answers, actual routes to those addresses, external connectivity, and a private resource if authorized. Preserve raw evidence in the conversation or an approved diagnostic location; redact credentials and tokens.
3. Ask the user to **manually** start aTrust. Do not start it or enter VPN/SSH passwords. Repeat the same snapshot promptly. Normalize and diff full routes by destination prefix, next hop, interface, flags, and on Windows metrics; retain raw tables. Count and summarize all new routes overlapping the *entire* FakeIP pool and relevant special-use ranges. Check specific FakeIP routes to corroborate, not to infer coverage from a few samples.
4. Separate route conflicts from DNS changes, default-route changes, interface metrics, HTTP/SOCKS proxy changes, IPv4/IPv6 selection, and WebSocket-specific failures. A more-specific prefix can win even if the default route does not change. On Windows, if no route overlap is visible, compare direct FakeIP, direct real-IP, and explicit-proxy paths to the same HTTPS host before inferring a mechanism. Confirm the private resource still uses aTrust.
5. Cross-check any Codex `Reconnecting` with a fresh task, unrelated HTTPS sites, DNS, and routes; one stuck task is not proof of general network failure. `Permission denied` after SSH transport reaches the server is authentication failure, not evidence of VPN path failure.

## Mutation gate

Before **any** change, report current facts, evidence, inference, unknowns, and the proposed next step. For a proposed repair give the exact persistent source file, minimal diff, new independent backup location, rollback procedure, and A/B validation plan. **Wait for explicit approval**; approval to diagnose is not approval to edit. Change one variable per experiment. Prefer the active subscription's Merge/Override after verifying composition order; never edit generated `clash-verge.yaml` as the durable source.

After a change, verify with aTrust off first, then ask the user to enable it and verify again. Stop and roll back the approved experiment if basic networking or DNS degrades, TUN fails, or aTrust captures the new pool. Stop without improvising if reliable backup is impossible, a whole database must be deleted, a permanent system route or DNS change is needed, aTrust policy must change, or a security control must be disabled. Do not blindly test multiple address pools.

Controller/cache maintenance is a **separate approved change**. Never display its secret. If necessary, prefer a supported authenticated Unix socket, otherwise authenticated loopback only; never expose it to LAN/`0.0.0.0`. Use the version's official FakeIP flush only with approval. Do not substitute DNS flush, macOS DNS-cache clearing, or `cache.db` deletion. Disable temporary controller access after verification unless the user wants it retained.

## Stage report contract

Use these fields at every pause: **事实 / 证据 / 推断 / 未确认 / 下一步 / 是否需要确认**. State what was actually measured, including failure-layer distinctions, and do not claim a candidate address pool is safe outside the tested environment.
