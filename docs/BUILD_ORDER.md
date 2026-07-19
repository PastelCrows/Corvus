# Build Order

Tracks the phased build plan from `docs/SCOPE.md` §21. Each phase maps to a
top-level directory. Work through phases roughly in order; a phase doesn't
need to be "done" before starting the next, but later phases assume earlier
ones are at least buildable.

- [ ] **Phase 1 — `image/`** — Base image build pipeline: Arch + mkosi +
      btrfs + firmware/driver bundling + atomic image tooling. *(up next)*
- [ ] **Phase 2 — `boot/`** — Bootloader + signing setup: systemd-boot,
      Secure Boot.
- [ ] **Phase 3 — `hardware/`** — Hardware detection pass, broadened for
      unknown hardware (not just the known dev machine).
- [ ] **Phase 4 — `shell/`** — Desktop shell layer on KWin: taskbar, file
      explorer, launcher, settings app, notifications, login/lock screen.
- [ ] **Phase 5 — `installer/`** — Graphical installer + first-run setup
      wizard.
- [ ] **Phase 6 — `software-center/`** — GUI-first software center (Flatpak
      backend) + networking GUI.
- [ ] **Phase 7 — `gaming/`** — Proton integration.
- [ ] **Phase 8 — `audio/`** — RT kernel tuning + Pipewire RT priority.
- [ ] **Phase 9 — `devtools/`** — Containers (Docker/Podman) + toolchains,
      available through the software center.
- [ ] **Phase 10 — `update-system/`** — Rollback/versioned update system
      hardening.
- [ ] **Phase 11 — `testing/`** — Hardware test matrix pass across GPU
      vendors and laptop/desktop configs before first real release.

## Open decisions

Not blocking Phase 1, but need an explicit answer before they're load-bearing:

- **Privacy/telemetry stance** (SCOPE.md §17) — opt-in/off-by-default vs.
  some telemetry. Needs deciding before any outside install.
- **License** — not yet chosen.
