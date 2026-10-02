# Case study: macOS Clash Verge Rev 2.5.6 + aTrust (2026)

This is an **anonymized local observation**, not a generic recipe. The machine had Mihomo v1.19.31, Clash TUN `utun1024`, DNS `enhanced-mode: fake-ip`, initial `fake-ip-range: 198.18.0.1/16`, and aTrust `utun7`. A VPN-only SSH service is represented as `<VPN_RESOURCE_IPV4>:2222`; its real address is intentionally omitted.

## Observed mechanism

With aTrust off, ordinary proxy domains mapped to `198.18.x.x`, and those FakeIP addresses routed through `utun1024`. Starting aTrust did **not** change the default route, but it injected 29 more-specific routes into `198.18.0.0/16`, covering nearly the whole `/16`. Longest-prefix matching sent matching FakeIP traffic to `utun7`. Codex showed `Reconnecting`; Google/GitHub reset. The VPN-only service `<VPN_RESOURCE_IPV4>` correctly routed through `utun7`. Sangfor documents a Mac + Clash FakeIP segment conflict as a known third-party-software issue (see [sources](sources.md)).

Two narrower experiments did not solve the whole problem: global `fake-ip -> redir-host` caused real-DNS/HTTP anomalies and was rolled back; a real-IP filter for OpenAI/ChatGPT allowed Codex to bypass FakeIP, but left ordinary Google/GitHub FakeIP captured by aTrust.

## Locally successful workaround

The active subscription's **dedicated Merge** carried only:

```yaml
dns:
  fake-ip-range: 198.19.0.1/16
```

The generated `clash-verge.yaml` was not edited. On hot reload, the composed range and TUN address changed (`utun1024` to `198.19.0.1/30`), yet previously queried domains still returned their exact old `198.18.x.x` mappings. In this installation `profile.store-fake-ip` was absent/false; source inspection and behavior implicated copied **in-memory** FakeIP mappings, not necessarily persistent `cache.db`. An authenticated external controller was temporarily enabled at `127.0.0.1:9097`; approved `POST /cache/fakeip/flush` returned HTTP 204. No DNS flush, OS DNS-cache clear or database deletion was used. New answers then came from `198.19.x.x`; the controller was disabled after the experiment.

With aTrust on, complete route-table comparison found **0 new `utun7` routes in `198.19/16`**, while the old `198.18/16` still had **29 `utun7` routes**. Representative `198.19.x.x` FakeIPs used Clash `utun1024`; `<VPN_RESOURCE_IPV4>` used aTrust `utun7`. Google/GitHub returned HTTP 200, Codex remained stable, and TCP to SSH port 2222 was reachable. SSH `Permission denied` was an authentication issue. A manual update of the active subscription preserved the dedicated Merge and the final `198.19` composed config.

**Boundary:** `198.19/16` is still part of RFC 2544 `198.18.0.0/15`. It was not taken over in *this* aTrust environment at the time of testing; no vendor guarantees future safety. Recheck after Clash, Mihomo, aTrust, subscription, VPN policy, Tailscale or local-network changes. Do not prescribe this range on another machine without its own full A/B route evidence.
