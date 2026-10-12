# BridgedCvGameCore

A Civilization IV: Beyond the Sword GameCore DLL built with Clang and C++23
for the original 32-bit game. It preserves stock gameplay, Python 2.4 and
Boost.Python bindings.

## Build

Requires CMake 3.28+, Ninja, Python 3 with `pefile==2024.8.26`, 7-Zip and BtS.
Run commands from the repository root.

### Linux (Bash)

Use the pinned Linux Clang/LLD 23.1.2 package, ICU 70 and Proton.

```bash
python3 tools/prepare_dependencies.py \
  --proton /path/to/Proton/proton --steam-client /path/to/Steam

BTS_DIR="/path/to/Beyond the Sword"
python3 tools/build_dll.py \
  --bts-dir "$BTS_DIR" \
  --llvm-bin /path/to/llvm-23.1.2/bin \
  --icu70-dir /path/to/icu70

cp tmp/dll-build/vcruntime140.dll "$BTS_DIR/"
```

Steam launch options: `WINEDLLOVERRIDES="msvcr71,msvcp71,vcruntime140=n" %command%`.

Keep Proton/Wine prefixes and test installations outside this checkout when
it is installed as a mod. BtS recursively scans the mod directory before
loading the DLL, including files reached through symlinks. A prefix's
`pfx/dosdevices/z:` link points to `/` and can send that scan through the host
filesystem essentially indefinitely.

### Windows (PowerShell)

Extract the [pinned Windows LLVM 23.1.2 archive](https://github.com/llvm/llvm-project/releases/download/llvmorg-23.1.2/clang+llvm-23.1.2-x86_64-pc-windows-msvc.tar.zst).
Put Python, CMake and Ninja on `PATH`. This build path has been checked under Wine.

```powershell
python tools/prepare_dependencies.py --seven-zip "C:/Program Files/7-Zip/7z.exe"

$BtsDir = "C:/Games/Beyond the Sword"
python tools/build_dll.py `
  --bts-dir "$BtsDir" `
  --llvm-bin "C:/LLVM-23.1.2/bin"

Copy-Item tmp/dll-build/vcruntime140.dll "$BtsDir/"
```

Both builds produce `tmp/dll-build/CvGameCoreDLL.dll`. Copy it into your test
mod's `Assets/`, preserving the original game DLL. `vcruntime140.dll` belongs
beside `Civ4BeyondSword.exe`.

`prepare_dependencies.py` prepares both dependency groups under
`tmp/dependencies/` and reuses files after verifying their checksums. Repeated
runs need neither Proton nor 7-Zip when both groups are complete. Use `--only sdk`
or `--only runtime` to prepare one group, and `--sdk-installer` or
`--runtime-installer` to use a downloaded installer. SDK installation on Linux
uses a temporary Proton prefix outside the checkout.

For a custom dependency location, pass `--output-dir tmp/my-dependencies` to
the preparation script and `--deps-dir tmp/my-dependencies` to the build script.
Existing caches still work through the build's `--vc71`, `--windows-sdk` and
`--exception-runtime` overrides. `--output-dir` selects a custom build directory;
Ninja reuses objects whose inputs have not changed.

`prepare_headers.py` generates the pinned compatibility header view during
CMake configuration. `build_support.py` provides shared hashing and PE metadata.
`dependency-lock.json` pins the Linux tools and stock references, `link-lock.json`
pins additional tools and link inputs, and `windows-toolchain-lock.json` pins
the Windows LLVM package.
