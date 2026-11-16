# Building

Two supported ways to get a runnable port:

| Path | Who it is for | What it needs |
|---|---|---|
| **Release zip + `play.bat`** | Players | Windows, your ROM, internet on first run |
| **Checkout + `sunburst.py`** | Contributors | Windows (or a POSIX host), your own C toolchain, Python 3.10+ |

Both produce the same executable. Neither needs BIOS dumps.

---

## Path A — release zip (recommended)

1. Download `sunburst-<version>-windows-x64-sources.zip` from Releases and unzip
   it into a writable directory. Do not use `Program Files`; the build writes
   beside itself.
2. Copy your dump to `rom\Digimon Story - Sunburst (Japan) (Rev 1).nds`. The
   SHA-1 must be `2fa1c272974fdef6b8f2acc3f9f778dd4111a587`.
3. Run `play.bat`.

`play.bat` hands over to `setup_kit.py`, which:

1. finds or downloads the private kit under `kit\` (LLVM-MinGW, CMake, Ninja,
   SDL3, ndspy), verifying every download by SHA-256;
2. verifies the ROM size and hash;
3. runs the build phases `prepare` → `recompiler` → `banks` (and `promote` only
   if coverage manifests are present, which a release never ships) → `build`;
4. copies the freshly built runner and its three DLLs into `runtime\`;
5. launches the game.

Everything is logged to `game\build.log`. Later launches compare
`.ship-id` / `.build-id` and start the existing executable immediately.

Useful switches:

```bat
python setup_kit.py --dry-run --no-dialog     :: show the plan, build nothing
python setup_kit.py --rom D:\dumps\sunburst.nds --no-dialog
python setup_kit.py --offline                 :: never download; fail if kit absent
python setup_kit.py --adopt-runtime           :: use a shipped runtime\ if it matches
python setup_kit.py --rebuild                 :: ignore caches, build again
python setup_kit.py --jobs 8                  :: limit parallel compile jobs
```

The kit layout under `kit\` is also what a checkout build expects, so you can
point a checkout at it with `SUNBURST_TOOLCHAINS=<unzip>\kit` instead of
installing a separate toolchain.

---

## Path B — build from a checkout

### 1. Checkout and ROM

```bat
git clone https://github.com/aoxzh/digimon-story-sunburst-recomp.git
cd digimon-story-sunburst-recomp
```

Put the dump in the repository root, keeping the exact name:

```
Digimon Story - Sunburst (Japan) (Rev 1).nds
```

`game.toml` points at it as `../Digimon Story - Sunburst (Japan) (Rev 1).nds`
from `sunburst-recomp\`. `*.nds` is git-ignored; never commit it.

### 2. Python and ndspy

`prepare` needs Python 3.10+ and [ndspy](https://github.com/RoadrunnerWMC/ndspy):

```bat
python -m pip install ndspy
```

### 3. Toolchain

The build needs CMake, Ninja, LLVM-MinGW (Windows) or a host clang, and the
SDL3 development package. Discovery order is:

1. explicit environment variables: `SUNBURST_CMAKE`, `SUNBURST_NINJA`,
   `LLVM_MINGW_BIN`, `SDL3_ROOT`;
2. `SUNBURST_TOOLCHAINS` (a directory laid out like the release `kit\`);
3. `<repository root>\toolchains`;
4. `PATH`.

On Windows the release kit is the tested combination (LLVM-MinGW 20261020,
CMake 4.4.4, Ninja 1.13.2, SDL3 3.4.18). A stock LLVM/Clang install usually
works but produces a different binary.

### 4. Build

From `sunburst-recomp\`:

```bat
python tools\sunburst.py prepare      :: verify ROM, extract ARM9/ARM7/overlays
python tools\sunburst.py recompiler   :: build nds_recompile
python tools\sunburst.py banks        :: emit the title and system banks
python tools\sunburst.py build        :: build nds_runner
```

Outputs:

| Artifact | Location |
|---|---|
| Recompiler | `generated\windows\recompiler\nds_recompile.exe` |
| Runner | `generated\windows\runner\nds_runner.exe` |
| Extracted inputs | `generated\windows\inputs\` |
| Generated banks | `generated\windows\recomp\`, `system\` |

Run it interactively:

```bat
generated\windows\runner\nds_runner.exe --interactive ^
  --rom "..\Digimon Story - Sunburst (Japan) (Rev 1).nds" ^
  --config game.toml --generated-firmware --boot direct
```

Useful flags: `--force` on `recompiler`/`build` to rebuild, `--jobs N` to limit
parallelism, `--software-3d` on `build` when the machine has no usable OpenGL
compute path.

`python tools\sunburst.py status` prints toolchain versions, inputs and artifact
state.

### 5. Mods (optional)

```bat
python tools\sunburst.py mods list
python tools\sunburst.py mods apply --patch mods\translation\my.ips --out ..\patched.nds
python tools\sunburst.py prepare --patched-rom ..\patched.nds --accept-patched-identity
```

A patched build only accepts that exact patched ROM; the retail dump is refused
afterwards. See [MODS.md](../sunburst-recomp/docs/MODS.md).

---

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| `ROM ... does not match` | Not the Japan Rev 1 dump, or patched/trimmed. Check the SHA-1. |
| `no Python` on first run | `play.bat` falls back to `bootstrap_python.ps1`; allow it, or install Python 3.10+ and delete `kit\python`. |
| `missing VCRUNTIME140.dll` | A system Python was used instead of the kit's. Install the Microsoft VC++ 2015–2022 x64 redistributable, or delete `kit\python` so the bootstrap fetches a self-contained copy. |
| SmartScreen warning | The executable is unsigned. "More info" → "Run anyway", or build it yourself from source. |
| Build stops on a missing compiler | Install the kit (run the release zip once) and set `SUNBURST_TOOLCHAINS`, or install CMake/Ninja/LLVM/SDL3 and put them on `PATH`. |
| Antivirus quarantines the runner | Common with fresh unsigned binaries and self-built executables; allow the folder or build from source. |
| Build fails after moving the folder | Generated build files contain absolute paths. Delete `sunburst-recomp\generated\` and `game\build\` (release shape), then rebuild. |
| No sound | Check `settings.ini` (`audio.enabled=1`, `audio.volume=100`) and the Esc menu. Audio is the frame clock; if the device fails to open, the game still runs but pacing changes. |
| Stutter in the first seconds | Expected: the audio queue and asset streaming warm up during boot. |

Logs from the first-start kit are in `game\build.log`; runtime logs are in
`logs\`. A bug report is much easier to act on with the ROM SHA-1, the toolchain
versions from `python tools\sunburst.py status`, and the log around the failure.

---

## Notes

- `generated\` is build output and is never committed. Do not hand-edit it;
  fix the recompiler, the runtime or the config instead.
- A built `nds_runner.exe` links GPL-3.0-or-later framework components; it is a
  GPL combined work. See [LICENSE](../LICENSE) and
  [THIRD_PARTY_NOTICES](../THIRD_PARTY_NOTICES.md).
- The vendored framework is pinned by this repository; updating it means
  updating `framework/` and re-running the build, not editing generated code.
