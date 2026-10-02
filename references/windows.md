# Windows: read-only baseline and A/B diagnostics

Use PowerShell/Windows commands on Windows; do not assume `utun`, `route -n get`, `ifconfig`, or macOS paths apply. The commands below are read-only. Redact credentials, controller secret and subscription URLs from any displayed configuration or environment data.

## 1. aTrust off: baseline

Determine Windows, Clash Verge Rev and Mihomo versions, running processes (`Get-Process` by name without dumping command lines), active subscription and composed config. Locate the real configuration directory from the running app/UI; do not assume a fixed `%APPDATA%` path. Identify the associated profile Merge/Override, DNS mode and FakeIP pool, TUN device and aTrust adapter by addresses and routing, not names alone.

```powershell
Get-NetAdapter -IncludeHidden
Get-NetIPInterface
Get-NetRoute -AddressFamily IPv4
Get-DnsClientServerAddress
ipconfig /all
route print -4
netsh winhttp show proxy
Resolve-DnsName chatgpt.com -Type A
Resolve-DnsName google.com -Type A
Resolve-DnsName github.com -Type A
```

Check system/user proxy configuration and relevant environment variables with credentials redacted. For each returned IPv4, use `Find-NetRoute -RemoteIPAddress <IPv4>` where available to identify the selected local interface and route, and check the full route table for overlapping prefixes. Use `Test-NetConnection -ComputerName <host> -Port <port>` only for intended, authorized endpoints. Record a private resource's route if in scope. Collect an HTTPS/Codex baseline without exposing traffic content.

## 2. User manually connects aTrust

Ask the user to start aTrust. Immediately repeat the same snapshot, including all adapters (hidden ones too), `ifIndex`, interface and route metrics, DNS servers, system proxy, complete IPv4 routes, FakeIP answers, and selected route for each FakeIP and private resource. Do not enter VPN or SSH credentials.

## 3. Normalized diff and interpretation

Normalize `Get-NetRoute` rows by `DestinationPrefix`, `NextHop`, `InterfaceIndex`/alias, `RouteMetric`, and route store; retain both raw tables. Join with `Get-NetIPInterface` to inspect interface metrics. Compute overlap and CIDR union for *all* new aTrust routes within the configured FakeIP range and the encompassing special-use range, not only a few example IPs. Windows chooses a matching route by prefix specificity before comparing effective metrics among comparable routes; confirm the actual selected path with `Find-NetRoute`. Detect DNS changes, default-route changes, IPv4/IPv6 differences, proxy changes and TUN/Wintun ownership separately. Test VPN-only/private traffic separately from external traffic.

Do not conclude that `198.19/16` is suitable for Windows merely because it worked on a Mac. If an address-pool conflict is demonstrated, use [controlled experiment](controlled-experiment.md) with Windows-specific backup, cache and route verification. If aTrust dynamically follows a new pool, stop pool experiments and refer to administrator/Sangfor compatibility guidance.
