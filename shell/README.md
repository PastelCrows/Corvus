# shell/ — Phase 4

Custom desktop shell layer, built on top of KWin (kept as-is underneath for
multi-monitor, DPI scaling, and Wayland-native GPU negotiation).

Covers:
- Taskbar, file explorer, app launcher
- Notification system (implements the freedesktop notification spec so
  third-party apps like Discord integrate without special-casing)
- Settings app, login/lock screen
- Accessibility (screen reader compatibility, scaling, high-contrast) and
  i18n structure

Display server: Wayland, with XWayland enabled for X11 app compatibility.

See `docs/SCOPE.md` §9 and §16.

Status: not started.
