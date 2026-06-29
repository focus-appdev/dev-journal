# systemd Boot Hang Fix — disable vs. mask

**Date:** 2026-06-29
**Category:** Linux / Ubuntu Server / systemd

## What I Did
proton was hanging at boot and I finally tracked it down to
`systemd-networkd-wait-online.service` — it was sitting there waiting for
the network interface to fully come online before letting the system continue.
I was SSHing in from PuTTY and kept thinking the machine had just locked up.

First thing I tried was disabling the service:

```bash
sudo systemctl disable systemd-networkd-wait-online.service
```

Didn't fix it. Rebooted and it hung in the same spot. Spent more time than
I'd like to admit staring at that before I figured out what was actually
going on.

Turns out `systemd-networkd` pulls `systemd-networkd-wait-online` in as a
dependency, so disabling it doesn't stick — systemd just re-enables it
indirectly at boot anyway. The fix that actually worked was masking it:

```bash
sudo systemctl mask systemd-networkd-wait-online.service
```

Rebooted clean after that.

## What Was Confusing
I assumed disable and mask were basically the same thing with different
severity. They're not. Disable removes it from the normal startup chain,
but if another active service lists it as a dependency, systemd will pull
it back in regardless. That's exactly what was happening here.

## What I Now Understand
- `disable` = won't start on its own, but dependencies can still drag it in
- `mask` = hard block — creates a symlink to `/dev/null`, nothing can start it
- If a service keeps coming back after disable, a dependency is the likely culprit
- mask is the right tool when you need to suppress something unconditionally
