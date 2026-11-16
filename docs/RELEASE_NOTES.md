# v0.1.0 — first public release

A native Windows x64 static-recompilation port of **Digimon Story: Sunburst
(Japan, Rev 1)**. Not an emulator: the game's ARM9/ARM7 code is translated to C
at build time and compiled for your PC.

## Downloads

| Asset | Size | What it is |
|---|---|---|
| `sunburst-0.1.0-windows-x64-sources.zip` | ~4.0 MiB | Sources + vendored framework. First start downloads the private toolchain and builds locally. This is the only shape published; there is no prebuilt executable. |

The archive needs your own cartridge dump. It contains no ROM, BIOS, firmware,
saves or pre-translated game code; the runner is built from source on your
machine.

## Requirements

- Windows 10/11 x64.
- *Digimon Story: Sunburst (Japan) (Rev 1)*, 64 MiB,
  SHA-1 `2fa1c272974fdef6b8f2acc3f9f778dd4111a587`.
- Internet on first start (only to download the toolchain); about 1 GB free
  disk for the kit.
- No BIOS dump required — the port uses FreeBIOS and a generated firmware.

## Quick start

1. Unzip into a writable directory (not `Program Files`).
2. Put the dump at `rom\Digimon Story - Sunburst (Japan) (Rev 1).nds`.
3. Run `play.bat`. First start builds the runner (2–5 minutes on a modern
   machine); later starts are instant.
4. On the prebuilt archive, `play.bat` adopts the shipped runtime directly.

Saves go to `saves\sunburst.sav`, savestates to `savestates\`, settings to
`settings.ini`, logs to `logs\`.

## Verified in this release

- Clean extract of this exact archive builds with the pinned kit
  (prepare 0 s, recompiler 4 s, banks 2 s, build 27 s on the test machine) and
  starts without a BIOS dump.
- Boot → title → continue → field route, 900 frames, exit code 0, frame digest
  identical to the development build.
- FreeBIOS + generated firmware, direct cartridge boot.

The remaining feature checks (battery save, 12 save-state slots, settings menu,
volume, filters, fast forward, restart recovery) were verified on the
development build of the same runtime. The release archive is the base-bank
build; it boots the same route, but promotes fewer code regions.

## Known limitations

- Only the opening chapter has been played end to end. Later chapters are
  **unverified**, not "supported".
- Wi-Fi / local wireless, microphone, IR and DS download play are not
  implemented.
- 3D-heavy scenes are unexercised by the tested route; the rasterizer is
  melonDS-derived but not guaranteed pixel-exact.
- Save compatibility with other emulators is not guaranteed. Back up saves.
- Linux and macOS are not supported by this release.
- A few percent of executed ARM9 instructions still use the bounded Tier-3
  interpreter (loader-generated or rewritten code) on the development build;
  the sources release ships the base-bank build, which promotes fewer regions
  and can spend more time interpreting long scripted stretches. The development
  build's measured route runs at about 2× realtime.

## Building from source

See `docs/BUILDING.md`. The sources archive builds with the bundled kit; a
checkout build needs CMake, Ninja, a linker and SDL3, plus `ndspy` for the ROM
step.

## License

Original tooling and documentation are MIT. The vendored ndsrecomp framework is
PolyForm Noncommercial 1.0.0 with MIT and GPL-3.0-or-later portions; recomp-ui
is MIT; FreeBIOS is BSD-2-Clause. A built `nds_runner.exe` is a
GPL-3.0-or-later combined work. See `LICENSE` and `THIRD_PARTY_NOTICES.md`.

This is an unofficial fan project, not affiliated with or endorsed by Bandai
Namco, Nintendo or the Digimon rights holders. Game assets remain the property
of their owners; you must supply your own dump.
