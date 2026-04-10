# ngmt-codec

`ngmt-codec` is the NextGenMediaTransport packaging of the **VMX** software video codec (forked from upstream **libvmx**). VMX targets very high encode/decode performance in software: low-latency intra-frame operation, 4:2:2:4 with alpha, 10-bit paths, and heavily optimized SIMD (including AVX2 on x86_64 and NEON on AArch64 via sse2neon).

VMX began as the “vMix Video Codec” used for the vMix Instant Replay feature. It has since been expanded with alpha channel support and is used in the Open Media Transport ecosystem.

## Features

- Low latency, intra-frame only
- 4:2:2:4 with alpha channel support
- 10-bit support
- Highly optimized AVX2 paths on x86_64 (e.g. 2160p60 on a single Intel core class CPU)
- Cross-platform: Windows, Linux, and macOS; ARM NEON support via sse2neon

## License

VMX / ngmt-codec is distributed under a permissive MIT license. See [LICENSE.txt](LICENSE.txt).

## System requirements

### x86_64

- Minimum: 64-bit CPU with SSE4.2 and SSSE3
- Recommended: AVX2 (e.g. Intel Haswell, 2013, and newer)

### AArch64 (ARM64)

- 64-bit ARM CPU with NEON (ARMv8)

## Documentation

Open Media Transport documentation: https://www.openmediatransport.org/docs/

## Building with CMake

Requirements: **CMake 3.15+**, a **C++17** compiler, and **Ninja** or **Make** (or Visual Studio on Windows). The build selects **x86_64** or **AArch64** sources from the **host** architecture (one architecture per build). On **x86_64**, GCC and Clang builds add **`-msse4.2`**, **`-mbmi`**, and **`-mlzcnt`** to the main VMX translation units so compiler intrinsics match the documented CPU baseline (AVX2 remains limited to `vmxcodec_avx2.cpp` via `-mavx2`).

From the repository root:

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --parallel
```

On Windows with a multi-configuration generator (e.g. Visual Studio), use:

```bash
cmake -B build
cmake --build build --config Release --parallel
```

Outputs (names may vary slightly by platform):

- **Shared library:** `libngmt-codec` / `ngmt-codec.dll` / `libngmt-codec.dylib`
- **CMake install** (optional): installs the library and public headers `vmxcodec.h`, `thread_tasks.h`.

```bash
cmake --install build --prefix /path/to/install
```

### Consuming the library (C/C++)

- Add the install prefix (`include`) to your include path and include **`vmxcodec.h`**.
- Link against the **`ngmt-codec`** library artifact produced above.
- Exported entry points remain the historical **`VMX_*`** C API (see `exports.def` on Windows).

### C# / VB.NET

Functions can be called via `DllImport`. Use `IntPtr` for the instance handle; you do not need to define native structs.

## Coding style

Formatting uses the repository’s **`.clang-format`** (LLVM-based, aligned with `ngmt-core`).

## Changelog

See [CHANGELOG.md](CHANGELOG.md).
