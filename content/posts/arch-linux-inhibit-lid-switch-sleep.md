---
title: "Arch Linux: inhibiting sleep triggered by lid switch"
date: 2026-01-10
---

Sometimes I want to run long jobs overnight on my laptop, or even charge my phone while the laptop is closed in my backpack (getting USB to have power when suspended is another longstanding problem).

For quite a while, I tried different variations of `gnome-session-inhibit` and `systemd-inhibit`.  But they never worked properly!

Finally, I found this gem on the Arch Linux forum: [Systemd-inhibit doesn't prevent sleep](https://bbs.archlinux.org/viewtopic.php?id=216236)

> Turns out in /etc/systemd/logind.conf, LidSwitchIgnoreInhibited defaults to yes.
>
> Uncomment the line, set it to no, and restart fixes the issue for me.

Note that after making this configuration change, you must restart your system.

## Inhibit suspend

Then, either of these commands can be used to inhibit sleep:

```bash
systemd-inhibit --what=sleep --mode=block sleep inf
```

```bash
gnome-session-inhibit --inhibit suspend --inhibit-only
```

The first talks to systemd-logind at the system level, the second talks to the GNOME session manager.  Both achieve the same result on a GNOME desktop, but `systemd-inhibit` is more portable across desktop environments.

## Inhibit lock screen

To also prevent the screen from locking, add `idle` to the GNOME command (systemd's idle inhibition doesn't prevent screen lock):

```bash
gnome-session-inhibit --inhibit suspend:idle --inhibit-only
```
