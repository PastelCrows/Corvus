# testing/ — Phase 11

Hardware test matrix pass across GPU vendors and laptop/desktop configs
before the first real release.

Covers:
- GPU vendor coverage (NVIDIA/AMD/Intel — genuinely different driver
  behavior)
- Laptop vs. desktop (power management is laptop-specific)
- Common peripheral classes (wireless input devices, USB audio interfaces)
- VM testing vs. real hardware testing (GPU passthrough/driver behavior
  doesn't fully replicate in a VM)

QA grows organically with whatever hardware is actually accessible — not a
planned matrix that has to be filled out before starting.

See `docs/SCOPE.md` §18.

Status: not started.
