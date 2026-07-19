# Corvus — Architecture & Planning Doc

This is the canonical scope/architecture reference for Corvus. It is the source
of truth for design decisions; subsystem READMEs should link back here rather
than restate it.

## 1. Goal & Design Philosophy

A daily-driveable, Linux-based operating system that:

- Works out of the box the way Windows does (no terminal required for basic
  tasks, drivers pre-integrated)
- Does everything well — coding, gaming, 3D/design, Discord/YouTube, music
  production — because the user genuinely uses their machine for everything
- Does not fragment/break under normal update cycles the way rolling-release
  distros do
- Is built and owned end-to-end, not a re-skin of an existing distro
- Is designed to eventually be usable by other people, on other hardware, not
  locked to one machine

Core principle: don't fight Linux's actual weak points (driver polish, update
fragility, terminal-first UX). Design them out architecturally rather than
hoping a base distro's defaults are good enough.

## 2. Project Scope Note

This OS is a fully standalone project, independent of the corvid-suite.
Neither depends on the other in either direction. The corvid-suite continues
to be built for Windows on its existing timeline; it only gets versioned
toward Linux once both the suite and this OS are separately complete. Any
integration between the two is a possible future add-on, not part of this
build.

## 3. Target Scope

- Not locked to one machine. Intended to eventually support other people's
  hardware, not just the current build — this changes the hardware/driver
  strategy from "tune for known hardware" to "detect and adapt to unknown
  hardware," and makes installer UX and QA across configurations real
  requirements, not nice-to-haves.
- Development machine: built and tested on a separate test machine/VM.
  Current Windows install stays untouched throughout development — no
  dual-boot risk to the daily driver while this is unstable.

## 4. Base Layer

Base distro: Arch Linux (best hardware/driver breadth, AUR gives near-total
software coverage, no philosophical restriction against proprietary
drivers/firmware).

Image pipeline: `mkosi` (built by the systemd developers) — produces
bootable, versioned images from build configs/scripts, layered on top of the
existing Arch/pacman/AUR foundation. Chosen over a NixOS-style approach
specifically to avoid abandoning the Arch foundation and avoid taking on
Nix's separate declarative config language as its own learning curve.

Filesystem: btrfs — chosen over ext4 specifically because it pairs with the
atomic image model: filesystem-level snapshots give a finer-grained safety
net underneath image-level rollback, which matters more once this runs on
hardware/configs beyond your own.

Update model: immutable/atomic images, not rolling package-by-package
updates.

- Core OS (kernel, DE, system libraries) ships as a single versioned,
  read-only image — modeled on Fedora Silverblue / SteamOS's approach
- Updates are atomic: build + test a new image, deploy it as a unit, boot
  into it
- One-click rollback to the last known-good image if something's wrong,
  backed by btrfs snapshots underneath
- You control the update cadence — nothing pulls in upstream changes
  automatically; you batch and test before releasing

This single decision eliminates the primary failure mode reported by Linux
daily-drivers: an unrelated package update breaking the DE/taskbar/file
explorer because they shared a library.

## 5. Application Model

Apps run sandboxed/self-contained (Flatpak-style), not against shared system
libraries.

- Each app bundles its own dependencies — mirrors how Windows apps behave,
  avoids "install 7 dependencies to run one app/game"
- System core and apps update independently and safely, since apps aren't
  relying on system-wide shared library versions

## 6. Hardware & Driver Strategy

Now that the OS is meant to run on hardware beyond your own, this shifts from
a one-time personal tuning pass to a real detection/compatibility layer:

- Ship full `linux-firmware`, not a trimmed set
- Bundle proprietary NVIDIA/AMD GPU drivers into the base image itself — not
  a post-install step
- Real first-boot hardware detection (GPU, wifi, audio interfaces, input
  devices), not just a config pass tuned to known hardware — needs to
  identify and adapt to hardware it hasn't seen before
- Laptop support is now a real question, not just desktop: power
  management, sleep/hibernate, battery reporting, and lid/brightness
  controls all need explicit handling if laptops are a supported target
- Peripheral support (mouse/keyboard/audio interfaces/etc.) tested across a
  range of common configurations, not just the current build machine

## 7. Boot & Security Foundations

- Bootloader: systemd-boot pairs naturally with the mkosi/systemd-based
  pipeline already chosen
- Secure Boot: supported. Requires signing your own kernel/bootloader/drivers
  as part of the build pipeline — extra setup work, but avoids forcing
  anyone installing this on a modern default-config PC to go disable a BIOS
  security feature first
- Package/image signing: once other people are installing this, verifying
  that an image actually came from you (not a tampered build) becomes a
  real trust requirement, not optional

## 8. Networking

- NetworkManager (or equivalent) with a GUI front end as the default — no
  terminal required to join wifi, manage VPNs, or handle network profiles
- VPN client support baked in as a standard capability, not an afterthought

## 9. Desktop Shell Layer

Middle-path approach, decided:

- Keep KWin (KDE Plasma's compositor/window manager) underneath — it already
  solves the genuinely hard problems (multi-monitor handling, DPI scaling,
  Wayland-native GPU negotiation, drag/resize, virtual desktops) without
  needing to be rewritten
- Build a fully custom shell layer on top: taskbar, file explorer, app
  launcher, notification system, settings app, login/lock screen
- Display server: Wayland, with XWayland enabled underneath for
  compatibility with older apps that haven't moved off X11 — avoids
  silently breaking legacy software
- Notification protocol: should implement the standard freedesktop
  notification spec so third-party apps (Discord, etc.) integrate with the
  custom notification system without needing special-casing

UX principle: GUI-first for everything a normal user touches daily — package
installs, settings, file management, network config — with terminal access
still available underneath for dev work, never required for basic tasks.

## 10. First Boot & Installer Experience

Now a real requirement since this isn't just for your own pre-configured
machine:

- Graphical installer (partitioning, user account creation, hardware
  detection) — this is most new users' first impression and needs to not
  feel like an Arch install
- First-run setup wizard: basic account/preferences setup, no terminal
  touched
- Branding/theming applied out of the box (this is also where the eventual
  OS name/identity lands)

## 11. Developer Tooling

Since you personally code as a primary use case, and this may attract other
developer users too:

- Containerization support (Docker/Podman) for dev workflows
- Standard dev toolchain availability (compilers, language runtimes, version
  control) easily installable through the GUI software center, not
  requiring manual setup

## 12. Gaming Layer

- Proton preconfigured and integrated out of the box — this is the one
  category that's already close to solved; inherits Valve's compatibility
  work rather than rebuilding it

## 13. Audio / Music Production Layer

The hardest category — real-time audio needs explicit tuning, it won't work
well by default:

- Low-latency/RT-tuned kernel
- Pipewire configured with proper real-time priority out of the box

## 14. Software Coverage Strategy

1. Open-source software that doesn't already run on Linux — treated as a
   bounded porting problem (swap Win32/DirectX calls for Vulkan/cross-platform
   equivalents), not a rewrite-from-scratch problem.
2. Closed-source Windows-only software — the real, unavoidable ceiling in
   general, but not treated as a pre-build risk here: most software currently
   relied on either has an existing Linux-using precedent among friends, or
   an existing Linux equivalent already. No dedicated pre-audit needed.

## 15. Backup & Recovery (User Data)

Distinct from OS-image rollback — this covers actual user files/documents,
which the image/snapshot system doesn't protect on its own:

- Decided: built into the Settings app directly, using the existing btrfs
  snapshot infrastructure but scoped to user file locations (docs, project
  folders, etc.) rather than the OS image itself

## 16. Accessibility & Localization

Worth deciding now rather than retrofitting later if this is ever used by
anyone else:

- Basic accessibility support (screen reader compatibility, scaling,
  high-contrast options) in the custom shell layer
- Localization/i18n structure, even if English-only at first — much easier
  to build in from the start than bolt on later

## 17. Privacy & Telemetry Stance

Matters once other people are installing this on their own hardware: decide
and state plainly whether the OS phones home at all (crash reports, usage
data) or is fully opt-in/off by default — this becomes part of the OS's
identity and trustworthiness for outside users.

**Status: undecided.** Needs an explicit decision before the first outside
install; tracked in `docs/BUILD_ORDER.md`.

## 18. Testing/QA Strategy

Informal and organic early on, not a formal upfront process — grows with
whatever hardware is actually accessible (own machine, friends willing to
test), rather than a planned matrix that has to be filled out before
starting:

- Dimensions worth being aware of as testing naturally expands: GPU vendor
  (NVIDIA/AMD/Intel — genuinely different driver behavior), laptop vs.
  desktop (power management is laptop-specific), common peripheral classes
  (wireless input devices, USB audio interfaces)
- VM testing covers a lot but not everything (GPU passthrough/driver
  behavior especially) — real hardware testing matters more as the pool of
  testers grows

## 19. Security & Maintenance Model

- Security updates ride the same atomic-image pipeline as everything else —
  tested, batched, rollback-capable — rather than ad hoc patches landing
  independently
- No auto-pulling of upstream changes; each image release is built and
  vetted directly

## 20. Naming

Working name: **Corvus** (the biological genus covering crows, ravens, rooks,
jackdaws, etc. — sits above the suite's species-level naming without being
part of it). Placeholder, can change any time during the build with no
architectural impact.
