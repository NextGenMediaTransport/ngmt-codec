# Changelog

All notable changes to this project are documented in this file.

## 2026-04-10 — Phase 2: CMake build standardization

- **Linux CI (GCC):** apply `-fdeclspec` only for **Clang** (`AppleClang` / `Clang`). GCC does not accept that flag; Ubuntu runners use g++ by default.

- Initial publish to the **NextGenMediaTransport** organization (`main`); foundational Phase 2 infrastructure commit.
- Added a root `CMakeLists.txt` using CMake 3.15+ with target-based configuration, C++17, and architecture-specific source selection (x86_64 vs AArch64).
- Windows: link `exports.def`, include `libvmx.rc` and `dllmain.cpp`; AVX2 compile flags apply only to `src/vmxcodec_avx2.cpp`.
- Removed legacy Visual Studio projects (`libvmx.sln`, `VMXCodec.vcxproj`, `VMXCodec.vcxproj.filters`) and the old shell/CMD build scripts under `build/`.
- Added `.clang-format` (LLVM-based, consistent with `ngmt-core`) and CI workflow for Ubuntu, Windows, and macOS.
- Documented the CMake-centric build in `README.md`.
