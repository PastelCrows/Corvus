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
mkosi build       # builds mkosi.output/corvus.raw
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

Status (as of last session): build succeeds end-to-end. Boot-testing via
`mkosi vm --console=interactive` gets past firmware and finds/loads the
bootloader (the `CopyFiles=/efi:/` fix in `mkosi.repart/10-esp.conf` was the
real fix for the earlier "no bootable option" failure). Currently blocked on
`Out of resources` when firmware tries to load the ~565M UKI into the VM's
default 2G RAM — next thing to try is `--ram=8G` on the `mkosi vm` command
line (cheap, no rebuild needed) to confirm that's really the cause before
deciding whether to just give the test VM more RAM permanently or finally
fix the oversized-initrd root cause above. Soft-lockup watchdog spam during
slow boots is likely a symptom of the same memory pressure, not a separate
issue — `nowatchdog` in `30-kernel.conf` alone didn't stop it.

Useful for next time: no `/dev/kvm` in the test VM (nested virt greyed out
in VirtualBox, likely Hyper-V on the Windows host claiming VT-x), so boot
tests run under slow software emulation. In `--console=interactive` mode
(the default), exit a stuck/slow boot with **Ctrl+A then X** — this reliably
works, unlike the graphical-mode Ctrl+Alt+2 monitor escape, which never did.
