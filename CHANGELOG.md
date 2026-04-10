# Changelog

All notable changes to this project are documented in this file.

## 2026-04-10 — Phase 2: CMake build standardization

- **Linux CI (GCC):** for **x86_64**, CMake passes **`-msse4.2;-mbmi`** to `src/vmxcodec.cpp` and `src/vmxcodec_x86.cpp` on GCC/Clang so SSE4.1 intrinsics (`smmintrin.h`) and BMI `_tzcnt_u64` match the translation unit ISA (default generic `x86-64` is SSE2-only and caused “target specific option mismatch” on Ubuntu).

- **Linux CI (GCC):** MSVC `__declspec(align(N))` in VMX sources is replaced with `VMX_DECLSPEC_ALIGN(N)` in `vmxcodec.h` — MSVC keeps `__declspec`, GCC/Clang use `__attribute__((aligned(N)))`. Removed the CMake `-fdeclspec` workaround (GCC never supported that flag; without it, raw `__declspec` did not compile on Ubuntu).

- Initial publish to the **NextGenMediaTransport** organization (`main`); foundational Phase 2 infrastructure commit.
- Added a root `CMakeLists.txt` using CMake 3.15+ with target-based configuration, C++17, and architecture-specific source selection (x86_64 vs AArch64).
- Windows: link `exports.def`, include `libvmx.rc` and `dllmain.cpp`; AVX2 compile flags apply only to `src/vmxcodec_avx2.cpp`.
- Removed legacy Visual Studio projects (`libvmx.sln`, `VMXCodec.vcxproj`, `VMXCodec.vcxproj.filters`) and the old shell/CMD build scripts under `build/`.
- Added `.clang-format` (LLVM-based, consistent with `ngmt-core`) and CI workflow for Ubuntu, Windows, and macOS.
- Documented the CMake-centric build in `README.md`.
