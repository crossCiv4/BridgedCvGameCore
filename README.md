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
python3 tools/s2_deps.py \
  --proton /path/to/Proton/proton --steam-client /path/to/Steam
python3 tools/s2_runtime_deps.py

BTS_DIR="/path/to/Beyond the Sword"
python3 tools/s3.py \
  --bts-dir "$BTS_DIR" \
  --llvm-bin /path/to/llvm-23.1.2/bin \
  --icu70-dir /path/to/icu70 \
  --vc71 "tmp/s2-deps/installed/Microsoft Visual C++ Toolkit 2003" \
  --windows-sdk tmp/s2-deps/installed/WindowsSDK \
  --exception-runtime tmp/s2-runtime-deps/minimum/vcruntime140.dll_x86

cp tmp/s3-build/vcruntime140.dll "$BTS_DIR/"
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
python tools/s2_deps.py
python tools/s2_runtime_deps.py --seven-zip "C:/Program Files/7-Zip/7z.exe"

$BtsDir = "C:/Games/Beyond the Sword"
python tools/s3.py `
  --bts-dir "$BtsDir" `
  --llvm-bin "C:/LLVM-23.1.2/bin" `
  --vc71 "tmp/s2-deps/installed/Microsoft Visual C++ Toolkit 2003" `
  --windows-sdk tmp/s2-deps/installed/WindowsSDK `
  --exception-runtime tmp/s2-runtime-deps/minimum/vcruntime140.dll_x86

Copy-Item tmp/s3-build/vcruntime140.dll "$BtsDir/"
```

Both builds produce `tmp/s3-build/CvGameCoreDLL.dll`. Copy it into your test
mod's `Assets/`, preserving the original game DLL. `vcruntime140.dll` belongs
beside `Civ4BeyondSword.exe`.
