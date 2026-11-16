# v0.1.0 — first public release

A native Windows x64 static-recompilation port of **Digimon Story: Sunburst
(Japan, Rev 1)**. Not an emulator: the game's ARM9/ARM7 code is translated to C
at build time and compiled for your PC.

## Downloads

| Asset | Size | What it is |
|---|---|---|
| `sunburst-0.1.0-windows-x64-sources.zip` | ~4.7 MiB | Sources + vendored framework. First start downloads the private toolchain and builds locally. **Recommended.** |
| `sunburst-0.1.0-windows-x64.zip` | ~79 MiB | Optional prebuilt Windows runtime. Larger, no compiler download. |

Both need your own cartridge dump. Neither contains the ROM, BIOS, firmware,
saves or compiled game code beyond what is built locally.

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

- Boot → title → continue → field route, on FreeBIOS with no BIOS dump.
- Battery save (64 KiB EEPROM): write, reload, restart recovery.
- Save states in all 12 slots.
- Settings menu, volume, settings persistence after restart.
- Display filters (colour curve, LCD grid), integer scaling.
- Fast forward (2×–4×, held Left Shift).
- 900-frame deterministic boot on a clean extract: 0 audio underruns, frame
  digest identical to the pre-release build.

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
  interpreter (loader-generated or rewritten code). The measured route runs at
  about 2× realtime.

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
