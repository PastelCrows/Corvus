# image/ — Phase 1

Base image build pipeline: Arch Linux as the package foundation, built into
versioned, bootable images via `mkosi`.

Covers:
- mkosi build configs producing a bootable Arch + btrfs image
- Bundling `linux-firmware` and proprietary NVIDIA/AMD drivers into the base
  image (not a post-install step)
- Atomic image tooling: build → test → deploy as a unit, boot into it
- btrfs snapshot layout underneath image-level rollback

See `docs/SCOPE.md` §4 and §6.

Status: not started.
