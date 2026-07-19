# image/ — Phase 1

Base image build pipeline: Arch Linux as the package foundation, built into
versioned, bootable disk images via [`mkosi`](https://github.com/systemd/mkosi).

See `docs/SCOPE.md` §4 and §6.

## Layout

- `mkosi.conf` — top-level image settings (distro, output format, hostname,
  locked root account)
- `mkosi.version` — current image version
- `mkosi.conf.d/10-base-packages.conf` — core system packages
- `mkosi.conf.d/20-graphics-drivers.conf` — NVIDIA/AMD/Intel drivers, bundled
  into the image itself rather than installed post-boot
- `mkosi.conf.d/30-kernel.conf` — kernel command line
- `mkosi.repart/` — systemd-repart partition definitions: ESP (vfat) + root
  (btrfs). Root splits `/home` and `/var/log` into their own subvolumes so
  that rolling back the OS image doesn't touch user files or logs — see the
  comment in `mkosi.repart/20-root.conf`.
- `mkosi.postinst.chroot` — enables core services (NetworkManager,
  systemd-resolved, systemd-boot auto-update) at build time, so the image
  boots with networking working out of the box

## Requirements to actually build

This pipeline is authored and syntax-validated (`mkosi summary`) in the dev
sandbox, but **cannot be built or boot-tested here** — building an Arch
rootfs needs `pacman` and network access to Arch mirrors, and boot-testing
needs `/dev/kvm`, neither of which this sandbox has. Per `docs/SCOPE.md` §3,
build/test is meant to happen on your separate Arch test machine/VM anyway,
never on the daily-driver install.

On that machine, with `mkosi` installed and running as root:

```sh
cd image
mkosi build       # builds mkosi.output/corvus.raw.zst
mkosi qemu        # boots the built image in QEMU to sanity-check it
```

Re-running `mkosi build` after bumping `mkosi.version` produces the next
versioned image; the atomic update / rollback tooling that deploys these as
a unit belongs in `update-system/` (Phase 10) — this phase only covers
producing the image itself.

## Known gaps / next steps

- `nvidia-open-dkms` compiles against `linux-headers` at build time; worth
  confirming DKMS build time is acceptable per image build, or whether to
  pin/cache built modules.
- `mkosi.repart/20-root.conf`'s `Subvolumes=` directive needs
  `systemd-repart >= 255` on the build host — unverified here, confirm on
  the real build machine.
- No image/package signing yet (Phase 2, `boot/`).

Status: pipeline authored and config-validated; not yet built or boot-tested
on real hardware/VM.
