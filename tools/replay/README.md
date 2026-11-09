# RA1 replay package integration

This package uses the repaired Westwood executable and ReplayLauncher adapter
from [CO2's ra-patches replay contribution](https://github.com/CO2-code/ra-patches/pull/9).
The included overlay was built from source revision
`8111764b44e1c11aeb7ab7d8d54f0b9f893133d1`. The client binaries and client source
are unchanged: existing Record Replays, extension/directory settings, replay list,
import/delete, storage pruning and Play are reused.

The adapter currently targets Windows/.NET Framework 4.8. The Unix launcher is
unchanged; integrating recording/playback into that launch path is separate work.

The checked-in overlay contains:

- `package/ra95.dat`: normal replay build, without diagnostic tracing;
- `package/ra95.dat.json`: input/output hashes, patch audit, source revision and
  launcher hash;
- `package/ReplayLauncher.exe` and `.config`;
- `package/Resources/GameLoadingWindow.ini`: existing INI theming hides unsupported
  YR viewer controls.

ClientDefinitions enables replay support, selects `.rarp` and the new launcher,
and requires its engine manifest. MainMenu exposes Load Game. Recording is not
forced: it follows **Options → Game → Record Replays**. Watch through **Load Game
→ Replays → Play**. Saved-game and campaign starts continue without recording.
The initial viewer uses the recorded player's house/shroud, free camera and a
visible read-only sidebar; pause/speed/observer/seek controls are not implemented.

## Rebuild the overlay

Requirements: Python 3, .NET SDK, NASM and GNU ld with i386pe on PATH, plus the
ra-patches replay checkout. The existing client supplies System.Memory.dll.

```powershell
python tools/replay/build_overlay.py --ra-patches ../ra-patches --out ../replay-overlay
```

Output must be a new directory. The tool reads the pristine ra95.dat from this
repository's base Git object `cab197416e7d4934d548419496d99918a7a3886a`, checks its
SHA-256, builds into temporary directories and creates only the overlay. It does
not overwrite package/ or download executables. Fetch that base commit if using a
shallow clone, or supply `--original-game <pristine.dat>`. Unknown binaries are
rejected. Inspect the output manifest, then copy the overlay into package/.

The source revision must match the intended ra-patches contribution. Rebuild after
changing that source. `--trace` produces the separate diagnostic variant used for
frame/RNG/CRC comparisons; do not accidentally publish it as the normal build.

See ra-patches' `tools/replay/CLIENT-INTEGRATION.md` for the RA1-tagged metadata
envelope and payload limits. These files are client-readable RA1 recordings, not
YR command streams. They require matching installed engine/assets/rules.

Scratch game copies are removed after success. `ReplayKeepTestSnapshots=yes` is
available for development; retained debug folders are outside replay storage
limits. No debug snapshot flag is enabled in this package.

## Validation

The existing package client reader parsed the portable metadata and extracted the
setup. Twenty-one client/container checks passed against that reader. Native
diagnostic builds reproduced 11,522 samples through the client's Play button, with
earlier singleplayer and two-peer network fixtures also matching. Actual launcher
tests cover recording enabled/disabled and temporary-copy cleanup. The normal
binary passes 19 emitted-x86 tests, skipping the diagnostic-only probe test.
The exact normal package overlay also recorded and replayed a 302-record match
with deployment/speed events, without diagnostic traces, and removed its scratch
copies after both runs.

The additive game build targets this exact package's original executable. A full
legacy ra-patches build and broad real-network/custom-map regression remain review
work; this integration is submitted as a draft with its engine dependency.
