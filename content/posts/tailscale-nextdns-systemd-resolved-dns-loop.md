---
title: "Fixing Tailscale + NextDNS + systemd-resolved DNS loops on Linux"
date: 2026-01-12
---

*Disclaimer: This post was written by Claude with my direction!  We're all in the midst of recalibrating our ethical compass when it comes to programming and writing with LLMs.  Since this post is intended as a reference for future me, it felt okay that I was simply a collaborator and editor.*

## Tailscale goes wild

Tailscale commands were hanging on my Arch Linux system. I wasn't using the stub-resolv.conf symlink at the time, so I tried switching to it [per Tailscale's recommendation](https://tailscale.com/kb/1188/linux-dns) as a potential fix. After connecting to WiFi, the journal filled with `dns udp query: request queue full` and DNS resolution stopped working entirely!

## The problem

When using Tailscale with NextDNS configured in the Tailscale admin console, combined with systemd-resolved on Linux, you may encounter a DNS loop that causes complete DNS failure. The symptom is tailscaled spamming the journal with:

```
dns udp query: request queue full
dns tcp query: request queue full
[RATELIMIT] format("dns udp query: %v") (106 dropped)
```

The system can ping IP addresses but cannot resolve any hostnames. Killing tailscaled restores DNS.

## Why it happens

The root cause is that Tailscale's internal DNS forwarder can end up with `100.100.100.100` (itself) in the upstream list, causing queries to loop infinitely until the queue is exhausted.

### The loop mechanism

Once 100.100.100.100 is in the upstream list:

```
Query arrives at 100.100.100.100 (Tailscale)
  → Not a MagicDNS query, forward upstream
  → Upstream includes 100.100.100.100
  → Query arrives at 100.100.100.100
  → Forward upstream...
  → ∞
```

This exponentially floods the queue until DNS stops working entirely.

### Known bugs that cause this

- [GitHub Issue #7816](https://github.com/tailscale/tailscale/issues/7816)---In "direct" mode, Tailscale's resolv.conf backup can become polluted with its own address
- [GitHub Issue #4842](https://github.com/tailscale/tailscale/issues/4842)---After network disruptions (WiFi reconnect, router reboot), Tailscale's internal routing can misconfigure to point at itself
- [GitHub Issue #7655](https://github.com/tailscale/tailscale/issues/7655)---Related DNS loop symptoms

See also: [Tailscale Blog: The Sisyphean Task of DNS Client Config on Linux](https://tailscale.com/blog/sisyphean-dns-client-linux)

## The solution

Use NextDNS directly in systemd-resolved, bypass Tailscale's DNS override, and configure split DNS so MagicDNS still works for tailnet hostnames.

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

### Step 2: Disable Tailscale DNS override

```bash
sudo tailscale set --accept-dns=false
```

This prevents Tailscale from taking over global DNS routing.

### Step 3: Configure split DNS for MagicDNS

So you can still resolve tailnet hostnames like `mydevice.tail123abc.ts.net`, configure the `tailscale0` interface to route `.ts.net` queries to Tailscale's DNS. Create a systemd service that triggers when the interface appears:

```bash
sudo tee /etc/systemd/system/tailscale-dns.service << 'EOF'
[Unit]
Description=Configure DNS routing for Tailscale MagicDNS
After=sys-subsystem-net-devices-tailscale0.device
BindsTo=sys-subsystem-net-devices-tailscale0.device

[Service]
Type=oneshot
ExecStart=/usr/bin/resolvectl dns tailscale0 100.100.100.100
ExecStart=/usr/bin/resolvectl domain tailscale0 tail123abc.ts.net
ExecStart=/usr/bin/resolvectl default-route tailscale0 false
RemainAfterExit=yes

[Install]
WantedBy=sys-subsystem-net-devices-tailscale0.device
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now tailscale-dns.service
```

Replace `tail123abc.ts.net` with your actual tailnet domain.

The domain setting serves dual purpose:

- **Search domain**: `ping mydevice` expands to `ping mydevice.tail123abc.ts.net`
- **Routing domain**: queries ending in `.tail123abc.ts.net` are sent to the interface's DNS server (100.100.100.100)

The `default-route false` ensures only tailnet queries go to Tailscale's DNS---everything else goes to NextDNS

## Verification

```bash
# Check resolved config
resolvectl status

# Test NextDNS is working
# Visit https://test.nextdns.io in a browser

# Test MagicDNS
ping mydevice.tail123abc.ts.net

# Or just the short name (search domain)
ping mydevice
```

## How it works

`/etc/resolv.conf` symlinks to systemd-resolved's stub, so all DNS queries go through resolved.

**Resolving `mydevice`:**

`mydevice` → 127.0.0.53 (resolved) → append search domain → `mydevice.tail123abc.ts.net` → domain match on tailscale0 → 100.100.100.100 (Tailscale) → `100.x.y.z`

**Resolving `mydevice.tail123abc.ts.net`:**

`mydevice.tail123abc.ts.net` → 127.0.0.53 (resolved) → domain match on tailscale0 → 100.100.100.100 (Tailscale) → `100.x.y.z`

**Resolving `google.com`:**

`google.com` → 127.0.0.53 (resolved) → no routing domain match → NextDNS via DoT → `142.250.x.x`

Since `--accept-dns=false` prevents Tailscale from registering as the global handler, there's no path for 100.100.100.100 to end up in the upstream list---no loop.

## Quick reference

| Task | Command |
|------|---------|
| Check DNS status | `resolvectl status` |
| Check tailscale0 DNS config | `resolvectl status tailscale0` |
| Test specific DNS server | `dig @100.100.100.100 hostname` |
| Restart resolved | `sudo systemctl restart systemd-resolved` |
| Restart tailscale DNS service | `sudo systemctl restart tailscale-dns` |
| Check tailscale DNS mode | `tailscale debug prefs \| grep -i dns` |
