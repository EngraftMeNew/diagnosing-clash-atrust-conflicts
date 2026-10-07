# Case study: Windows 11 Clash Verge Rev 2.5.7 + aTrust (2026)

This is an **anonymized observation on one Windows machine**, not a generic repair recipe. Clash Verge Rev 2.5.7 used Mihomo v1.19.32, FakeIP DNS with `198.18.0.1/16`, and TUN with automatic routing. The user manually connected aTrust. Private VPN addresses, subscription details, SSH identity, and proxy-provider details are omitted.

## Evidence before changing the pool

With aTrust off and TUN on, `api.openai.com` resolved to `198.18.x.x`. Direct HTTP to an ordinary test site returned 200, and direct HTTPS to the OpenAI API returned 401 without credentials. The same status codes were returned through the local explicit Clash proxy. The 401 was an expected network reachability result, **not** an authenticated Codex API test.

With aTrust on and TUN on, Codex Desktop repeatedly showed `Reconnecting`. Repeated direct requests through TUN failed with Windows `10053`/`10054` connection abort/reset errors, while the explicit Clash proxy continued to return HTTP 200 and HTTPS 401. With TUN off and the Clash system proxy on, Codex and private SSH access worked. Changing only the TUN stack from `mixed` to `gvisor` did **not** fix the TUN failure.

The captured Windows IPv4 route tables showed the Clash TUN default route and aTrust routes for its private VPN subnet and DNS endpoint. They did **not** show new aTrust routes overlapping `198.18.0.0/16`. Therefore the macOS case's more-specific-route mechanism was **not established** on this Windows machine. A driver/filter interaction or another mechanism remained possible; the route table alone could not identify it.

The decisive traffic comparison used the **same HTTPS hostname and SNI**. Its FakeIP was obtained through the Windows resolver; a real A record was obtained through DNS-over-HTTPS over the working explicit proxy. Both addresses returned 401 with aTrust off. With aTrust on, the `198.18.x.x` connection failed with `10053`, while the real-IP connection still returned 401. Both test sockets were direct connections, without an application proxy. This showed a failure specific to the old FakeIP path, without proving which Windows or aTrust component dropped it.

## Controlled change and result

After checking the current route table for the candidate pool, the active subscription's dedicated Merge was backed up and changed by one field:

```yaml
dns:
  fake-ip-range: 198.19.0.1/16
```

The generated `clash-verge.yaml` was **not** edited. Reactivating the subscription produced `198.19.0.1/16` in the composed config, but a previously queried hostname still returned its old `198.18.x.x` FakeIP; newly queried names received `198.19.x.x`. The local controller was not listening, so no FakeIP API flush was performed. A restart using Clash Verge's **Restart Core** control changed the old hostname's answer to `198.19.x.x`. No OS DNS flush, DNS-cache API flush, or cache-database deletion was used. The exact source of the retained old mapping was not established.

With aTrust connected, TUN enabled, and the new pool active, direct HTTP and HTTPS again returned 200 and 401, and the explicit proxy still worked. The user reported Codex stable with `gvisor`. The TUN stack was then restored to its original `mixed` value; four consecutive direct and explicit-proxy probes passed, and the user again reported Codex stable. The Windows default route remained on the Clash TUN address in `198.19.0.0/16`, while the private aTrust route remained present. SSH to the authorized private resource and an SSH reverse forward to the local Clash proxy were also verified. The reverse-forward session had to be restarted after the TUN stack change interrupted its SSH connection.

**Boundary:** `198.19.0.0/16` is still inside the RFC 2544 `198.18.0.0/15` block. This result establishes only that it worked on this machine in this observed aTrust session. It does not prove aTrust's internal mechanism, future compatibility, subscription-refresh persistence, or behavior after a reboot. On another Windows host, reproduce the route and FakeIP-versus-real-IP tests before considering a pool change. If it fails, restore the saved Merge, reactivate the profile, restart the core only if old mappings persist, and verify the resulting DNS and routes.
