# boot/ — Phase 2

Bootloader and signing setup.

Covers:
- systemd-boot configuration (pairs naturally with the mkosi/systemd image
  pipeline)
- Secure Boot: signing kernel/bootloader/drivers as part of the build
  pipeline
- Image/package signing so an installed image can be verified as coming
  from this project, not a tampered build

See `docs/SCOPE.md` §7.

Status: not started.
