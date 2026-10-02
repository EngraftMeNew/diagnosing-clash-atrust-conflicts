# Diagnose Clash Verge Rev / Mihomo TUN and aTrust conflicts

A Codex Skill for diagnosing DNS, FakeIP, routing, and TUN/VPN traffic conflicts when Clash Verge Rev (Mihomo) and Sangfor aTrust run together. It is a diagnostic workflow, **not** a one-click VPN fix.

## Supported systems

- **macOS:** Read-only interface, route, DNS, proxy, and FakeIP checks; comparison before and after the user connects aTrust.
- **Windows:** Equivalent PowerShell/Windows checks for adapters, route and interface metrics, DNS, proxies, and actual selected paths. Windows interface names, indices, metrics, aTrust behavior, and safe address pools must be discovered on that machine.
- Linux-specific diagnostics are not included.

The Skill begins with aTrust **off** and a read-only baseline. It asks the user to connect aTrust manually, captures the same observations, and compares complete route tables—not just a few sample IPs. It separates measured facts from inferences. It proposes a persistent, minimal change only after the cause is established, shows a new backup and rollback plan, and waits for explicit approval. It never asks Codex to enter VPN/SSH passwords or print a controller secret.

## Install in Codex

The [current official Codex Skill documentation](https://learn.chatgpt.com/docs/build-skills) lists `$HOME/.agents/skills` as a user-level discovery location and states that symlinked Skill folders are supported. It also documents repository-scoped `.agents/skills`. This repository's root **is the Skill folder** (`SKILL.md` is at the root); do not nest it under another copy of the same name.

On macOS, after replacing the repository URL below if the repository is later moved to another owner:

```sh
mkdir -p "$HOME/code" "$HOME/.agents/skills"
git clone https://github.com/EngraftMeNew/diagnosing-clash-atrust-conflicts.git "$HOME/code/diagnosing-clash-atrust-conflicts"
ln -s "$HOME/code/diagnosing-clash-atrust-conflicts" "$HOME/.agents/skills/diagnosing-clash-atrust-conflicts"
```

If that destination already exists, inspect it first; do not overwrite another installed copy. On Windows, clone the repository and copy its root folder to `$HOME\.agents\skills\diagnosing-clash-atrust-conflicts` (or create a directory symlink if your Windows setup allows one). A copied installation must be recopied when updating. If Codex does not notice a new Skill automatically, restart it. Do not install both this copy and another Skill with the same `name` in separate discovery locations.

## Use

Explicitly invoke `$diagnosing-clash-atrust-conflicts`, or describe the symptom in a new Codex task—for example: “aTrust connects and Clash TUN stops working.” The `description` also allows implicit discovery. The first phase should be read-only and should ask for aTrust to remain off until baseline collection finishes.

## Anonymized macOS case (2026)

On one Mac running Clash Verge Rev 2.5.6 and Mihomo v1.19.31, the initial FakeIP pool was `198.18.0.1/16`. Clash TUN used `utun1024`; aTrust used `utun7`. aTrust did not change the default route, but injected **29 more-specific routes** into `198.18.0.0/16`. Longest-prefix matching diverted ordinary FakeIP traffic to aTrust, causing Codex reconnections and Google/GitHub resets. The VPN-only resource still routed correctly through aTrust.

Global `redir-host` and an OpenAI-only real-IP filter did not solve the whole problem. A dedicated per-subscription Clash Verge Rev Merge changed only `dns.fake-ip-range` to `198.19.0.1/16`. A hot reload inherited old in-memory FakeIP mappings, so an explicitly approved, authenticated Mihomo `POST /cache/fakeip/flush` was needed; the temporary loopback-only controller was later disabled. Complete aTrust-on route comparison then found **0 new aTrust routes in `198.19/16`**, versus **29 in the old `198.18/16`**. Clash external traffic, Codex, and the VPN-only resource worked simultaneously; a subscription update retained the Merge.

**`198.19.0.0/16` is not a universal fix.** It remains within RFC 2544's `198.18.0.0/15` benchmarking block. This result means only that one aTrust deployment did not capture it at the time of testing. Future aTrust, Mihomo, subscription, network, or organizational policy changes require a fresh A/B route check. Windows must be diagnosed independently; never apply the Mac range change by analogy.

## Safety and rollback

- Default to read-only checks. The user manually connects and disconnects aTrust.
- Before editing, identify the active subscription's persistent Merge/Override—not generated `clash-verge.yaml`—and show the exact one-variable diff, independent non-overwriting backup, rollback steps, and aTrust-off/on verification plan.
- Stop and roll back an approved experiment if DNS, basic connectivity, or Clash TUN fails. If aTrust dynamically captures the candidate pool, stop pool experiments and escalate to the network administrator or Sangfor compatibility support.
- Never add permanent OS routes, change system DNS or aTrust policy, delete `cache.db`, disable security protections, expose a controller beyond authenticated local access, or silently clear unrelated caches to force a test to pass.
- An SSH `Permission denied` after transport reaches the server is an authentication result, not a VPN-path failure. One stuck Codex task alone is not proof of network failure.

The full procedure is in [SKILL.md](SKILL.md), with separate [macOS](references/macos.md), [Windows](references/windows.md), [controlled experiment](references/controlled-experiment.md), [anonymized case](references/case-macos-2026.md), and [source](references/sources.md) references.

## License

MIT; see [LICENSE](LICENSE).
