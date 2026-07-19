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
- **Initrd is oversized (~565M UKI) because it's generic (hostonly=no) and
  bundles firmware for effectively every driver.** GPU/wifi/audio firmware
  doesn't need to be in the initrd — it loads on-demand from the real root
  after root is mounted, well past the point the initrd's job is done. Only
  storage/console drivers actually need to be there. `10-esp.conf` is sized
  generously (1024M) to unblock building in the meantime; the real fix is
  constraining what dracut/mkosi's initrd builder pulls in (worth revisiting
  alongside Phase 3's hardware detection work, since it's the same "what
  needs to be available at which boot stage" question).
- No image/package signing yet (Phase 2, `boot/`).

Status: pipeline built successfully partway through on a real Arch VM;
iterating on real build errors as they surface (host tool gaps → tools tree,
repart config syntax, ESP sizing). Not yet a confirmed successful boot.
