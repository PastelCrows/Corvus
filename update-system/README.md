# update-system/ — Phase 10

Rollback and versioned update system hardening, plus user-data backup
(distinct from OS-image rollback).

Covers:
- Atomic update flow hardening: build → test → deploy → one-click rollback
- Security updates riding the same pipeline as everything else, batched and
  vetted rather than ad hoc
- User file/document backup in the Settings app, using btrfs snapshots
  scoped to user file locations rather than the OS image

See `docs/SCOPE.md` §4, §15, and §19.

Status: not started.
