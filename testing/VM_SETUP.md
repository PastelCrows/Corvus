# Test VM setup (Windows 10 host)

Corvus is built and boot-tested on a separate Arch Linux machine/VM, never
the daily-driver install (`docs/SCOPE.md` §3). This is the setup for doing
that from a Windows 10 host using VirtualBox.

## Why VirtualBox

Free, works on Windows 10 Home or Pro (Hyper-V requires Pro/Enterprise), and
coexists with WSL2 if that's already in use. Nested hardware virtualization
is not required: `mkosi build` only does chroot/mount operations on a disk
image file, and `mkosi qemu` boot-testing works via software emulation
without KVM — slower to boot, but fine for checking that an image comes up.

## VM creation

1. Install VirtualBox: https://www.virtualbox.org/wiki/Downloads
2. Download the Arch install ISO: https://archlinux.org/download/
3. Create the VM:
   - Type Linux, Version "Arch Linux (64-bit)" (fall back to "Other
     Linux (64-bit)" if not offered)
   - RAM: 8192 MB (nvidia-open-dkms compiles at build time and wants
     headroom)
   - CPU: as many cores as available — image builds are CPU-bound
   - Disk: 60GB+, dynamically allocated
   - **Settings → System → Motherboard → "Enable EFI"** — required, since
     systemd-boot is UEFI-only. Easy to miss.
4. Boot the ISO, run `archinstall` (Arch's guided installer) rather than a
   manual pacstrap. Any minimal/no-DE profile is fine — the VM's own root
   filesystem (ext4 is fine here) is unrelated to the btrfs image Corvus
   builds as output.

## Build + boot-test

```sh
sudo pacman -Syu
sudo pacman -S mkosi git qemu-full edk2-ovmf
git clone https://github.com/PastelCrows/Corvus.git
cd Corvus && git checkout claude/corvus-os-scope-1lkv0o
cd image
sudo mkosi build     # produces mkosi.output/corvus.raw.zst
sudo mkosi qemu      # boots the built image
```

`git clone` will prompt for GitHub auth since the repo is private — a
personal access token works as the password.

Re-run `mkosi build` after any change under `image/`; bump `mkosi.version`
for a new versioned image.
