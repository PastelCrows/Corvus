# Working rules for Corvus

## Never state a technical fact from memory. Verify it, every time.

Config syntax, section names, default values, package names, CLI flags,
tool behavior — none of it gets presented as fact, written into a config,
or handed to the user as a command until it's checked against a primary
source. "I recall mkosi works like X" is a guess, not an answer, even when
it sounds confident. Guessing from training data and letting the user's
real build be the thing that discovers it's wrong is the single biggest
source of wasted time and wasted iteration on this project so far — stop
doing it.

**What counts as a source of truth, in order of preference:**

1. The actual tool, installed and queried directly — `--help`, `--version`,
   a dry-run/summary/validate command, a live query against the real
   package database (`pacman -Si`), the real system's own man pages.
2. Documentation shipped with the installed tool itself (e.g. mkosi ships
   its full spec as `mkosi.md`/`mkosi.1` inside the installed package —
   read that, not a remembered summary of it). This is version-matched to
   what's actually running, which training-data knowledge is not.
3. Upstream official docs fetched live, if the tool's own bundled docs
   aren't enough.

**What does not count:** recalling how a tool "usually" works, pattern-
matching from a similar tool, or anything not checked against something
version-matched to what's actually being used right now. Tool behavior
drifts across versions constantly — something true of mkosi 20.2 has
already been observed to be false on a newer version in this exact
project.

**If a primary source can't be reached** (blocked network, no man page
installed, etc.), say so explicitly and get the cheapest safe read-only
check run against the real target system instead of substituting a guess.
Never present an unverified claim with the same confidence as a verified
one.

## Don't hand off commands while still making changes

Once a "run this" command block is sent, treat that config as frozen until
the result comes back. Finish all verification/editing first, then send
one command block, then stop — don't keep editing the same files a command
you already sent depends on. If more checking is genuinely needed after a
command's already been sent, say so explicitly before continuing, don't
silently keep working.
