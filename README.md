# linux-okhsunrog-lts

Personal Arch Linux kernel package — fork of [`linux-cachyos-lts`](https://aur.archlinux.org/packages/linux-cachyos-lts) with two extra patches and ZFS built-in by default.

## Why

Intel Meteor Lake (Core Ultra series) iGPUs have a known firmware-level bug that causes random GPU hangs (`GUC: TLB invalidation response timed out` → `GPU HANG: ecode 12:0:00000000` → full system freeze). See [drm/i915 issue #14469](https://gitlab.freedesktop.org/drm/i915/kernel/-/issues/14469).

The only reliable workaround is disabling RC6. Intel removed the `i915.enable_rc6` modparam years ago, so a small kernel patch is needed to re-add it. This package bundles that patch (for both `i915` and `xe` drivers) on top of the CachyOS LTS kernel.

ZFS is also built-in (rather than DKMS) since the kernel is rebuilt anyway on each update.

## What's on top of upstream

- `0001-drm-i915-Add-modparam-for-rc6.patch` — the i915 patch posted by Vinay Belgaumkar @ Intel ([patchwork](https://patchwork.freedesktop.org/patch/666117/))
- `0001-drm-xe-Add-modparam-for-rc6.patch` — same idea ported to the `xe` driver
- `_build_zfs=yes` defaulted on
- `pkgbase` renamed to `linux-okhsunrog-lts` so it doesn't collide with the upstream AUR package

Everything else (BORE scheduler, Cachy patch stack, configurable env vars) is unchanged from CachyOS upstream.

## Build / install

```sh
makepkg -si
```

Then add `i915.enable_rc6=0` (and/or `xe.enable_rc6=0`) to the kernel cmdline, or:

```sh
echo "options i915 enable_rc6=0" | sudo tee /etc/modprobe.d/i915-rc6.conf
echo "options xe enable_rc6=0"   | sudo tee /etc/modprobe.d/xe-rc6.conf
```

## Updating from upstream

When CachyOS bumps their AUR PKGBUILD, sync via:

```sh
git clone https://aur.archlinux.org/linux-cachyos-lts.git upstream
diff upstream/PKGBUILD PKGBUILD
# manually merge bumps (pkgver, _minor, b2sums of source tarball + config) into PKGBUILD
```

The two `.patch` files and rc6-related metadata stay; only the upstream-sourced bits move.

## License

Same as Linux kernel: GPL-2.0-only.
