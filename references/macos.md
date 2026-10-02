# macOS: read-only baseline and A/B diagnostics

Use this reference only on macOS. These commands read state; they do not change routes, DNS, caches, or VPN settings. Do not run commands that might disclose a controller secret or proxy credentials verbatim in user-visible output. Ask before saving raw snapshots, since they can contain private network details.

## 1. aTrust off: establish a baseline

Identify the running product/core version, active subscription, DNS mode, `dns.fake-ip-range`/`fake-ip-range6`, `profile.store-fake-ip`, TUN settings, and configuration sources. Locate Clash Verge Rev data using the app's UI and process/config paths; on common installs the data directory is under `~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/`. Inspect `profiles.yaml` or UI association to identify the *active subscription's* Merge/Override. Read the generated config only to verify the composed result; redact `secret`, tokens, proxy credentials and subscription URLs. Also inspect DNS/TUN fields that may explicitly refer to the pool.

Read-only baseline commands:

```sh
ifconfig
netstat -rn -f inet
netstat -rn -f inet6
route -n get default
scutil --dns
scutil --nwi
scutil --proxy
networksetup -listallnetworkservices
```

For the active service, `networksetup -getinfo 'Wi-Fi'` and `networksetup -getdnsservers 'Wi-Fi'` are read-only. Record relevant proxy environment variable *names and redacted values*, not embedded credentials. Identify TUN ownership from configuration, address, routes and process; `utun1024`/`utun7` were names in one case only. Check whether aTrust has actually exited, not merely that its window is closed.

Use the system's configured resolver path and a direct DNS query as complementary observations, e.g. `dscacheutil -q host -a name chatgpt.com` and `dig +short A chatgpt.com`. `dig` may not reproduce application-specific macOS resolver behavior; note which resolver answered. Query several representative proxy domains and an unqueried control name. For each returned A address run `route -n get <IPv4>` and record `interface`, gateway and flags. Record a private destination only with user authorization, e.g. `route -n get <VPN_RESOURCE_IPV4>`. Check a known HTTPS site and Codex state without equating an app task hang with a network outage.

## 2. User manually connects aTrust

Ask the user to connect and keep aTrust on long enough for a second read-only snapshot. Repeat the same commands and DNS queries promptly; include all `utun` addresses, the complete IPv4 table, default route, resolver ordering, proxy state, and relevant `route -n get` results. If Codex disconnects, do not alter configuration; preserve available evidence and let the user exit aTrust.

## 3. Normalize and analyze

- Compare route entries as destination/prefix + gateway + interface + flags, not by display order or transient expiry. Keep every raw row. Expand abbreviated destinations before CIDR arithmetic; do not mistake host routes or interface-scope routes for broad prefixes.
- Extract *every* new aTrust-owned route overlapping the configured FakeIP range and its enclosing special-use block (often `198.18.0.0/15`). Count entries, calculate the union of their CIDRs, and separately report uncovered portions. Inspect other special-use ranges such as `100.64.0.0/10` if present. A few sampled `route get` results cannot establish coverage of a whole `/16`.
- Compare default route, DNS resolver list/order, IPv4 vs IPv6, HTTP/SOCKS/system proxy settings, and interface ownership. To distinguish WebSocket-specific failure, compare ordinary HTTPS and an actual Codex connection while the user observes the app; do not infer transport solely from `Reconnecting`.
- Confirm whether the VPN-only resource IP still routes through aTrust. Network reachability of SSH TCP port can be tested separately from authentication; avoid password entry or repeated login attempts.

If the evidence does not establish route capture, do not prescribe a FakeIP-range change. If it does, route to [controlled experiment](controlled-experiment.md) only after presenting evidence and waiting for approval.
