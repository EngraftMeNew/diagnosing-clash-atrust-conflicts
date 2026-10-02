# Authoritative references and version checks

Prefer the installed versions' documentation and source. Current online docs may have changed; verify tags before applying version-sensitive guidance. These references support analysis, **not** permission to modify a machine.

- [Mihomo DNS and FakeIP](https://wiki.metacubex.one/en/config/dns/): `enhanced-mode`, `fake-ip-range`, `fake-ip-filter`, and TUN default IPv4 relation.
- [Mihomo TUN](https://wiki.metacubex.one/en/config/inbound/tun/): auto-route, device, DNS hijack and platform-specific notes.
- [Mihomo general configuration](https://wiki.metacubex.one/en/config/general/): `profile.store-fake-ip` is for persisting FakeIP mappings; `store-selected` is a different setting.
- [Mihomo API](https://wiki.metacubex.one/en/api/): `POST /cache/fakeip/flush` and separate `POST /cache/dns/flush`, with documented HTTP 204 success.
- [Mihomo v1.19.31 FakeIP pool source](https://github.com/MetaCubeX/mihomo/blob/v1.19.31/component/fakeip/pool.go) (`CloneFrom`, `FlushFakeIP`, disk-state handling) and [v1.19.31 cache route source](https://github.com/MetaCubeX/mihomo/blob/v1.19.31/hub/route/cache.go). Verify the installed tag when interpreting reload/cache behavior.
- [Clash Verge Rev v2.5.6 composition source](https://github.com/clash-verge-rev/clash-verge-rev/blob/v2.5.6/src-tauri/src/enhance/mod.rs) and [Merge implementation](https://github.com/clash-verge-rev/clash-verge-rev/blob/v2.5.6/src-tauri/src/enhance/merge.rs): inspect active-profile/global extensions and fields enforced by app settings. [Project extension guide](https://github.com/clash-verge-rev/clash-verge-rev.github.io/blob/main/docs/guide/extend.md) explains profile versus global extensions; recheck when versions differ.
- [Sangfor aTrust known conflicting software](https://bbs.sangfor.com.cn/atrustdeveloper/aTrustClient/docs/9.%E7%AC%AC%E4%B8%89%E6%96%B9%E8%BD%AF%E4%BB%B6%E5%86%B2%E7%AA%81/9.2%E5%B7%B2%E7%9F%A5%E5%86%B2%E7%AA%81%E7%B1%BB%E8%BD%AF%E4%BB%B6.html): Mac + Clash FakeIP segment conflict is listed. Vendor acknowledgment does not specify the routes of every deployment.
- [IANA IPv4 Special-Purpose Address Registry](https://www.iana.org/assignments/iana-ipv4-special-registry/): RFC 2544 benchmarking block is `198.18.0.0/15`, encompassing both `198.18/16` and `198.19/16`.
- [Tailscale address-space documentation](https://tailscale.com/docs/concepts/tailscale-ip-addresses): Tailscale uses `100.64.0.0/10`; inspect it before considering CGNAT space for FakeIP.
- [Microsoft Find-NetRoute](https://learn.microsoft.com/en-us/powershell/module/nettcpip/find-netroute?view=windowsserver2025-ps) and [NetTCPIP module](https://learn.microsoft.com/en-us/powershell/module/nettcpip/?view=windowsserver2025-ps): read-only Windows route and interface inspection. Confirm availability on the host.

Do not elevate forum anecdotes to facts. Record source URL, product version, timestamp and what was actually measured on the user's machine. If a source or version is unavailable, say so explicitly.
