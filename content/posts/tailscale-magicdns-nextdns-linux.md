---
title: "Split DNS for Tailscale MagicDNS + NextDNS on Linux"
date: 2026-01-12
---

*Disclaimer: This post was written by Claude with my direction!  We're all in the midst of recalibrating our ethical compass when it comes to programming and writing with LLMs.  Since this post is intended as a reference for future me, it felt okay that I was simply a collaborator and editor.*

This document describes a robust DNS configuration for Tailscale on Linux that provides:

- MagicDNS for tailnet hostname resolution
- NextDNS for all other queries (with DNS-over-TLS encryption)
- Resilience to network transitions (suspend/resume, wifi changes)
- Protection against tailscaled's DNS queue exhaustion

## The problem with `--accept-dns=true`

### How it works

When `accept-dns=true` (the default), tailscaled configures systemd-resolved via D-Bus:

```
SetLinkDNS(tailscale0, [100.100.100.100])
SetLinkDomains(tailscale0, ["tail4a82f0.ts.net", "~."])
SetLinkDefaultRoute(tailscale0, true)
```

The `~.` routing domain means **all** DNS queries flow through tailscaled:

```
Application → 127.0.0.53 (resolved) → 100.100.100.100 (tailscaled) → NextDNS (DoH)
```

### The queue exhaustion problem

tailscaled has an internal DNS query queue with a hard limit:

```go
const maxActiveQueries = 256
```

When this queue is full, tailscaled immediately returns SERVFAIL:

```go
if n := atomic.AddInt32(&m.activeQueriesAtomic, 1); n > maxActiveQueries {
    return nil, errFullQueue  // instant SERVFAIL
}
```

### Why network transitions trigger failure

During suspend/resume or wifi reconnection:

1. **Network goes down** --- tailscaled cannot reach upstream DNS (NextDNS via DoH)
2. **Queries accumulate** --- applications and system services make DNS requests
3. **DoH has slow failure** --- each query waits for TLS handshake timeout (~5-30 seconds)
4. **Queue fills** --- 256 in-flight queries waiting to timeout
5. **New queries rejected** --- instant SERVFAIL responses
6. **Network recovers** --- but queue is still full of timing-out queries
7. **Death spiral** --- resolved sees instant failures, retries aggressively

### Why this differs from normal DNS server failure

| Scenario | Behavior | Result |
|----------|----------|--------|
| Remote DNS unreachable | Slow timeout (5+ sec) | Rate-limited retries, fallback works |
| tailscaled queue full | Instant SERVFAIL | Aggressive retries, no backoff |

With `~.` routing domain, resolved has no fallback---all queries must go to 100.100.100.100. The combination of instant failure + no fallback = complete DNS outage.

### Evidence from logs

```
Jan 12 20:00:23 tailscaled: monitor: time jumped (probably wake from sleep)
Jan 12 20:00:27 tailscaled: [RATELIMIT] (48 dropped)
Jan 12 20:00:27 tailscaled: dns udp query: dial tcp [2a07:a8c0::d6:3326]:443: network is unreachable
Jan 12 20:00:32 systemd-resolved: Using degraded feature set TCP instead of UDP for DNS server 100.100.100.100
Jan 12 20:01:19 tailscaled: dns: tcp query: request queue full
```

## The problem with `--accept-dns=false`

### Expected behavior

With `accept-dns=false`, tailscaled should not manage DNS at all, allowing manual split DNS configuration.

### Actual behavior

tailscaled still calls D-Bus methods to configure the tailscale0 interface, but with empty values:

```go
// From resolved.go - setConfigOverDBus is called regardless of accept-dns setting
SetLinkDNS(tailscale0, [])           // clears DNS servers
SetLinkDomains(tailscale0, [])       // clears routing domains
SetLinkDefaultRoute(tailscale0, true) // sets as default route
SetLinkLLMNR(tailscale0, "no")
SetLinkMulticastDNS(tailscale0, "no")
FlushCaches()
```

This happens on network events like daemon startup, `LinkChange` events (network up/down, IP changes), and control plane config updates.

### Impact on manual configuration

Any manual split DNS configuration:

```bash
resolvectl dns tailscale0 100.100.100.100
resolvectl domain tailscale0 tail4a82f0.ts.net
resolvectl default-route tailscale0 false
```

Gets overwritten on network events by tailscaled's empty config.

### Evidence from logs

```
Jan 16 12:37:55 tailscale-dns.service sets DNS to 100.100.100.100
Jan 16 12:37:58 tailscaled resets: "Bus client reset DNS server list"
Jan 16 12:37:58 tailscaled resets: "Bus client set default route setting: yes"
```

This behavior is arguably intentional---tailscaled manages its own interface regardless of whether it's handling DNS for the whole system. But it means we need a workaround to maintain split DNS.

## Understanding systemd-resolved DNS routing

### DNS server priority

systemd-resolved uses this priority order:

1. **Per-link with matching routing domain** --- e.g., tailscale0 with `tail4a82f0.ts.net`
2. **Per-link with `DefaultRoute=yes`** --- e.g., `wlp0s20f3` from DHCP
3. **Global DNS** --- from `/etc/systemd/resolved.conf`

### The `~.` routing domain

A domain prefixed with `~` is a routing domain (not search domain):

- `tail4a82f0.ts.net` → search domain, appended to single-label names
- `~tail4a82f0.ts.net` → routing domain, queries for `*.tail4a82f0.ts.net` go here
- `~.` → routing domain for root, captures **all** queries

### Why Global DNS is rarely used

With typical wifi/ethernet connections:

```
wlp0s20f3: DefaultRoute=yes, DNS=192.168.31.1 (from DHCP)
Global: DNS=45.90.28.0#nextdns (from resolved.conf)
```

The per-link DHCP DNS wins because `DefaultRoute=yes` has higher priority than Global. Without intervention, the `resolved.conf` NextDNS config is essentially unused.

## The `DNSOverTLS=yes` trick

### How it works

In `/etc/systemd/resolved.conf`:

```ini
[Resolve]
DNS=45.90.28.0#d63326.dns.nextdns.io
DNS=45.90.30.0#d63326.dns.nextdns.io
DNSOverTLS=yes
```

`DNSOverTLS=yes` **requires** DNS-over-TLS for all queries.

### Effect on DNS routing

When resolved tries to use DHCP-provided DNS (e.g., `192.168.31.1`):

1. Attempts DoT connection to `192.168.31.1:853`
2. Home router doesn't support DoT → connection fails
3. Falls back to Global DNS (NextDNS)
4. NextDNS supports DoT → succeeds

### Result

`DNSOverTLS=yes` acts as a filter that bypasses all non-DoT DNS servers:

- NextDNS (supports DoT) → **used**
- DHCP DNS (no DoT) → bypassed
- Coffee shop DNS (no DoT) → bypassed

This provides consistent NextDNS usage regardless of network, without configuring `ignore-auto-dns` on every connection.

## The complete solution

### Step 1: Configure NextDNS in systemd-resolved

Edit `/etc/systemd/resolved.conf`:

```ini
[Resolve]
DNS=45.90.28.0#YOUR-CONFIG-ID.dns.nextdns.io
DNS=45.90.30.0#YOUR-CONFIG-ID.dns.nextdns.io
DNS=2a07:a8c0::#YOUR-CONFIG-ID.dns.nextdns.io
DNS=2a07:a8c1::#YOUR-CONFIG-ID.dns.nextdns.io
DNSOverTLS=yes
```

Replace `YOUR-CONFIG-ID` with your NextDNS configuration ID (found at https://my.nextdns.io).

Restart resolved:

```bash
sudo systemctl restart systemd-resolved
```

### Step 2: Prevent NetworkManager from managing tailscale0

Create `/etc/NetworkManager/conf.d/99-unmanaged-tailscale.conf`:

```ini
[keyfile]
unmanaged-devices=interface-name:tailscale*
```

### Step 3: Disable Tailscale DNS override

```bash
sudo tailscale set --accept-dns=false
```

This prevents tailscaled from setting the `~.` routing domain (which would route all queries through its queue).

### Step 4: Create the DNS fix script

Create `/usr/local/bin/tailscale-dns-fix.sh`:

```bash
#!/bin/bash
sleep 2
if ip link show tailscale0 &>/dev/null; then
    resolvectl dns tailscale0 100.100.100.100
    resolvectl domain tailscale0 YOUR-TAILNET.ts.net
    resolvectl default-route tailscale0 false
    logger -t tailscale-dns-fix "Applied MagicDNS config to tailscale0"
fi
```

Replace `YOUR-TAILNET.ts.net` with your actual tailnet domain.

```bash
sudo chmod +x /usr/local/bin/tailscale-dns-fix.sh
```

### Step 5: Create the watcher service

Create `/etc/systemd/system/tailscale-dns-watch.service`:

```ini
[Unit]
Description=Watch tailscaled DNS changes and apply MagicDNS config
After=tailscaled.service
Requires=tailscaled.service

[Service]
Type=simple
ExecStart=/bin/bash -c 'journalctl -u tailscaled -f -n0 --grep="dns: Set:" | while read line; do /usr/local/bin/tailscale-dns-fix.sh; done'
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now tailscale-dns-watch.service
```

This monitors tailscaled logs and re-applies split DNS whenever tailscaled resets it.

## How it works

```
                        Application
                            │
                            ▼
                    127.0.0.53 (resolved stub)
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
    *.YOUR-TAILNET.ts.net           Everything else
      (routing domain)
              │                           │
              ▼                           ▼
         tailscale0               wlp0s20f3 (DHCP DNS)
       100.100.100.100            192.168.31.1 - NO DoT
              │                           │
              ▼                           ▼ (DoT required, fails)
          MagicDNS                  Global DNS (NextDNS)
       (tailnet hosts)            45.90.28.0 - DoT works
```

| Query | Route | DNS Server | Transport |
|-------|-------|------------|-----------|
| `mydevice.YOUR-TAILNET.ts.net` | tailscale0 | 100.100.100.100 | UDP (local) |
| `mydevice` (single-label) | tailscale0 (search domain) | 100.100.100.100 | UDP (local) |
| `google.com` | Global (DoT fallback) | NextDNS | DoT |
| `any-website.com` | Global (DoT fallback) | NextDNS | DoT |

## Why this solution is robust

**Protection against queue exhaustion:**
- Regular DNS queries never touch tailscaled---they go directly to NextDNS via DoT
- Only tailnet queries (`*.YOUR-TAILNET.ts.net`) go through 100.100.100.100
- Network transitions don't cause queue buildup for regular DNS

**Automatic recovery:**
- `tailscale-dns-watch.service` detects when tailscaled resets DNS config
- Re-applies split DNS within ~2 seconds
- Works across tailscaled restarts and network changes

**Consistent NextDNS usage:**
- `DNSOverTLS=yes` bypasses any DHCP-provided DNS
- Works on any network without per-network configuration
- Encrypted DNS everywhere

**MagicDNS functionality preserved:**
- Tailnet hostnames resolve via 100.100.100.100
- Search domain allows `ping mydevice` instead of `ping mydevice.YOUR-TAILNET.ts.net`
- Split DNS means only tailnet queries have queue risk (minimal traffic)

## Verification

```bash
# Check resolved config
resolvectl status

# Check tailscale0 specifically
resolvectl status tailscale0

# Test NextDNS is working
# Visit https://test.nextdns.io in a browser

# Test MagicDNS
ping mydevice.YOUR-TAILNET.ts.net

# Or just the short name (search domain)
ping mydevice
```

## Quick reference

| Task | Command |
|------|---------|
| Check DNS status | `resolvectl status` |
| Check tailscale0 DNS config | `resolvectl status tailscale0` |
| Test specific DNS server | `dig @100.100.100.100 hostname` |
| Restart resolved | `sudo systemctl restart systemd-resolved` |
| Check watcher service | `sudo systemctl status tailscale-dns-watch` |
| Manually re-apply DNS fix | `sudo /usr/local/bin/tailscale-dns-fix.sh` |
| Check tailscale DNS mode | `tailscale debug prefs \| grep -i dns` |

## Known limitations

1. **2-second window after tailscaled changes** --- brief period where MagicDNS may not work
2. **Relies on log monitoring** --- if journald is unavailable, watcher won't trigger
3. **Workaround for tailscaled behavior** --- ideally `accept-dns=false` would leave tailscale0's resolved config alone

## See also

- [Tailscale: Linux DNS](https://tailscale.com/kb/1188/linux-dns)
- [Tailscale Blog: The Sisyphean Task of DNS Client Config on Linux](https://tailscale.com/blog/sisyphean-dns-client-linux)
