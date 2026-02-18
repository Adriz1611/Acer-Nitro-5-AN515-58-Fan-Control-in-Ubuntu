# NitroSense Linux Controller (Acer AN515-58)

This repository contains a Linux-native control stack for Acer Nitro laptops:

- `nitrosense`: EC + power tuning backend script
- `nitrosense-gui`: GTK desktop frontend for profile control
- `nitrosense-admin`: single helper utility for packaging and sudoers setup

The intent is to keep behavior close to NitroSense-style profiles while staying fully scriptable.

## What The Backend Can Do

`nitrosense` supports profile and maintenance actions through compact flags:

- Power profiles: `q` (Quiet), `d` (Default), `p` (Performance)
- Fan profiles: `a` (Auto), `c` (Custom), `m` (Max)
- Debug and service actions:
  - `r` or `x`: dump EC bytes
  - `e`: print Intel RAPL `energy_uj` values
  - `n`: restart `nvidia-powerd`
- Maintenance:
  - `w`: only ensure EC write path is armed

It accepts grouped flags like `da`, `pm`, `pc` and optional custom fan percentage for `c`.

## System Prerequisites

Kernel command line should include:

```bash
ec_sys.write_support=1 msr.allow_writes=on
```

Recommended packages:

```bash
sudo apt install bash python3 python3-gi gir1.2-gtk-3.0 gir1.2-ayatanaappindicator3-0.1 policykit-1 sudo
```

Optional module check:

```bash
find /lib/modules -type f -iname "*ec_sys.ko*"
```

## Backend Usage

Show command help:

```bash
sudo bash nitrosense
```

Common examples:

```bash
sudo bash nitrosense da
sudo bash nitrosense pm
sudo bash nitrosense pc 50
sudo bash nitrosense r
sudo bash nitrosense e
```

## GUI Usage

Run directly from repo:

```bash
./nitrosense-gui
```

After `.deb` install, launch from Ubuntu app menu using:

- **NitroSense Controller**

To start hidden manually (this is what autostart uses):

```bash
nitrosense-gui --start-hidden
```

GUI features:

- power/fan profile cards
- custom fan slider (0-100)
- multi-point fan curve presets (`Silent`, `Balanced`, `Gaming`) in Custom fan mode
- quick actions (`n`, `e`, `r`)
- automatic AC/Battery switching (AC -> `Performance + Auto`, Battery -> `Default + Auto`)
- tray hide/show with live state, checked mode items, and one-click `Restore Safe Mode`
- tray indicator label shows average CPU temperature (refresh every 3 seconds)
- timestamped command output and status feedback
- auto-registers as a startup app in `~/.config/autostart/nitrosense-gui.desktop`
- applies startup profile automatically using AC/Battery rule (or `da` fallback if power source is unavailable)
- includes in-app controls to disable/re-enable startup registration

### Optional: No Password Prompt Every Time

For repository-local runs, install a narrowly scoped sudoers rule manually:

```bash
./nitrosense-admin sudoers-install
```

Remove later with:

```bash
./nitrosense-admin sudoers-remove
```

For `.deb` installs, passwordless execution is enabled by default via:

- `/etc/sudoers.d/nitrosense-gui`
- rule scope: `/bin/bash /usr/lib/nitrosense/nitrosense *` (for `%sudo` users)

## Build Debian Package

Build package:

```bash
./nitrosense-admin build-deb 0.1.0
```

Output file:

```bash
dist/nitrosense-gui_0.1.0_$(dpkg --print-architecture).deb
```

Install locally:

```bash
sudo apt install ./dist/nitrosense-gui_0.1.0_$(dpkg --print-architecture).deb
```

After installation, app actions run without password prompts for users in `sudo` group.

Installed paths are:

- `/usr/bin/nitrosense-gui`
- `/usr/bin/nitrosense-admin`
- `/usr/lib/nitrosense/nitrosense`
- `/usr/share/applications/nitrosense-gui.desktop`
- `/usr/share/icons/hicolor/scalable/apps/nitrosense-gui.svg`

## Automatic Debian Builds On GitHub

This repository includes GitHub Actions workflow:

- `.github/workflows/deb-build.yml`

Behavior:

- Push to `main`: auto-builds a `.deb` with version format:
  - `0.0.0+gitYYYYMMDDHHMMSS.<shortsha>`
  - Uploaded to the workflow run as an artifact.
- Push tag `vX.Y.Z`: builds version `X.Y.Z` and attaches `.deb` to GitHub Release for that tag.

Tag-and-release example:

```bash
git tag v0.2.0
git push origin v0.2.0
```

## Notes

- The backend script now supports `x` as an alias for `r`.
- `nitrosense-gui` can resolve backend from repo-local path or `/usr/lib/nitrosense/nitrosense` (for packaged install).
- Thermal and EC behavior is firmware-dependent; test profiles incrementally.
