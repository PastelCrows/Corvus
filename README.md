# Corvus

A daily-driveable, Arch-based Linux operating system built end-to-end rather
than re-skinned: atomic/immutable image updates (mkosi + btrfs, Silverblue
/SteamOS-style), sandboxed apps, GUI-first UX with terminal access never
required for basic tasks, and out-of-the-box driver/hardware support.

Corvus is a standalone project — independent of, and with no dependency in
either direction on, the corvid-suite.

- Full architecture and design rationale: [`docs/SCOPE.md`](docs/SCOPE.md)
- Phased build plan and current status: [`docs/BUILD_ORDER.md`](docs/BUILD_ORDER.md)

## Layout

Each directory corresponds to a build phase in `docs/BUILD_ORDER.md`:

| Directory | Covers |
|---|---|
| `image/` | mkosi build pipeline, base Arch+btrfs image, firmware/driver bundling |
| `boot/` | systemd-boot, Secure Boot signing |
| `hardware/` | first-boot hardware detection |
| `shell/` | custom desktop shell on KWin |
| `installer/` | graphical installer + first-run wizard |
| `software-center/` | GUI software center (Flatpak) + networking GUI |
| `gaming/` | Proton integration |
| `audio/` | RT kernel + Pipewire tuning |
| `devtools/` | container/toolchain support |
| `update-system/` | atomic update + rollback tooling |
| `testing/` | hardware test matrix, QA notes |

Currently in Phase 1 (`image/`).
