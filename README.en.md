# DLSSG Standalone

[简体中文](README.md) | English

A Windows x64 / D3D12 DLSSG package with an SM86 backend. Each proxy DLL embeds the matching original DLSSG 310.1 runtime, its models and processing pipeline, and the SM86 backend. The default `Mode=Bundled` uses this embedded pair, so backend matching does not depend on the game's own DLSSG version.

The project contains the source code, third-party headers and libraries, runtime, and GPU resources required to build it. It can be copied to another location without the original research workspace, a game installation, or an existing CMake cache.

## Current release

**Release, September 7, 2026** is the first milestone. Hardware validation was performed on an **NVIDIA GeForce RTX 3080 Ti, 12 GiB, physical SM86**, with driver **591.86**.

| User-reported gameplay results | Frame generation off | 2X | 4X |
|---|---:|---:|---:|
| Black Myth: Wukong | 50 FPS | 80 FPS | 150 FPS |
| Cyberpunk 2077, path tracing | 35 FPS | 60 FPS | 100 FPS |

These are approximate FPS observations from manual gameplay. The tester described the current experience as relatively stable. They are not standardized average FPS, percentile FPS, latency, or long-duration stability measurements. In-game 2X / 4X settings do not guarantee the same multiplier over the measured frame rate with frame generation off.

The release directory is `dist/Release/`. The deployment archive is `dist/dlssg-release-x64.zip`, and the source archive is `dist/dlssg-source.zip`; both have SHA256 sidecars. Deployment and source packages exclude raw evidence, logs, screenshots, and captures. Those remain in the development workspace.

Both games use the same tested proxy. Their test conditions, results, and limitations are recorded in the [validation document](docs/VALIDATION.md) (Chinese).

## Installation

1. Exit the game completely.
2. Copy `version.dll` and `dlssg_sm86.ini` from the release directory beside the game's actual rendering executable.
3. Start the game normally.

If the game does not import `version.dll`, use `alternatives/winmm.dll` instead. Enable only one proxy. An existing mod with the same DLL name requires resolving the entry-point conflict; arbitrary proxy chaining is not implemented.

Key defaults in the supplied INI are:

```ini
[FrameGeneration]
MaxGeneratedFrames=3

[Logging]
Level=2

[Debug]
MarkGeneratedFrames=0

[Compatibility]
KernelImage=Auto
ForceSM86Route=0
SimulateAmpere=0

[Runtime]
Mode=Bundled
```

`MaxGeneratedFrames=3` advertises a maximum of three additional generated frames, corresponding to 4X mode. The game chooses the actual count; the setting does not independently add menu options or force 4X presentation. `1` allows up to 2X, `2` up to 3X, and `0` preserves the runtime capability.

The proxy extracts and verifies its embedded runtime and backend under `%LOCALAPPDATA%\DlssgSm86\bundles\<bundle-id>`. Subsequent launches reuse the cache. No Python or PowerShell launcher is needed to run the game.

Restart the game after changing the INI. To uninstall, exit the game and remove the added proxy and INI. See the [installation and configuration guide](docs/INSTALL.md) (Chinese) for all options.

## Building

Install Visual Studio or Build Tools with **Desktop development with C++**, the Windows SDK, and **C++ CMake tools for Windows**. The project requires MSVC, an x64 target, C++20, and CMake 3.22 or later.

Run this from the project root in PowerShell:

```powershell
.\build.cmd
```

The entry point calls `build.ps1`, discovers the Visual Studio installation and its CMake tools, builds the project, runs five basic CTest checks, and creates the deployment package. The tested build used MSVC 19.44, Windows SDK 10.0.26100.0, CMake 3.31.6, and Ninja Multi-Config.

**A normal build and the five basic checks do not require Python, a GPU, or the CUDA Toolkit.** The build embeds the existing SM86 cubin/PTX resources in `assets/kernels`; it does not regenerate the kernels or models.

| Output | Contents |
|---|---|
| `dist/Release/version.dll` | Main proxy with embedded runtime and backend |
| `dist/Release/alternatives/winmm.dll` | Alternative proxy |
| `dist/Release/dlssg_sm86.ini` | User configuration |
| `dist/Release/README.md` | Installation instructions |
| `dist/Release/docs/VALIDATION.md` | Gameplay results and known issues |
| `dist/Release/manifest.json` | File hashes, bundle identity, and validation status |
| `dist/dlssg-release-x64.zip` | Deployment archive |
| `dist/dlssg-release-x64.zip.sha256` | Archive checksum |

A normal build records GPU validation as `performed=false`. Previous hardware results do not automatically apply to a newly compiled DLL.

Common build options:

```powershell
.\build.cmd -Configuration Debug
.\build.cmd -Configuration RelWithDebInfo
.\build.cmd -BuildDirectory "out\custom build"
.\build.cmd -CMake "C:\path\to\cmake.exe" -Generator "Visual Studio 17 2022"
```

Relative `BuildDirectory` paths are resolved against the project root. Outputs under `dist` remain grouped by configuration. Use a new build directory after changing generators or moving the source tree. Ninja Multi-Config requires an MSVC x64 environment and Ninja on PATH.

The [fresh-machine setup and build guide](docs/DEVELOPMENT.md) (Chinese) covers installation from scratch. The existing development workspace also has a prepared toolchain under `out/tools`, used by `scripts/build-local.ps1`; that toolchain is not included in the source package, and the helper does not install missing tools.

## PTX and cubin selection

On a physical SM86 GPU such as the RTX 3080 Ti, `KernelImage=Auto` selects the precompiled SM86 cubin. To use the driver's PTX JIT path on the same GPU, set:

```ini
[Compatibility]
KernelImage=PTX
ForceSM86Route=0
SimulateAmpere=0
```

`KernelImage=Cubin` explicitly requires physical SM86 when the SM86 route is active. `KernelImage` chooses the kernel format; it does not independently force routing on another GPU architecture. Keep both compatibility switches at `0` on the RTX 3080 Ti. Restart the game after changing these settings.

## Optional GPU validation

Prepare Python 3.10 or later, NumPy, and an NVIDIA GPU with its driver, then run:

```powershell
.\build.cmd -Validate -Python "C:\path\to\python.exe"
.\build.cmd -Validate -RequireSM86 -Python "C:\path\to\python.exe"
```

`-RequireSM86` requires the physical CUDA device matched to the D3D12 adapter to be SM86. It must be used with `-Validate`.

The validator discovers the driver's `_nvngx.dll` and uses a different `nvngx_dlssg.dll` beside it for cross-version checks. If discovery fails or multiple driver installations are present, specify the files explicitly:

```powershell
.\build.cmd -Validate -RequireSM86 -Python "C:\path\to\python.exe" `
  -NgxRuntime "C:\path\to\_nvngx.dll" `
  -ComparisonDll "C:\path\to\another-version\nvngx_dlssg.dll"
```

The comparison DLL must differ from the embedded `assets/runtime/nvngx_dlssg.dll`. Each run writes a new `out/validation/<configuration>-<timestamp>/` directory. Reports retain numerical differences; `stages/` contains each stage's command, stdout, stderr, and exit status.

The RTX 3080 Ti run completed 18 compute scenarios and a marker scenario with both architecture simulation switches disabled. All 176 comparisons between Auto, PTX, and Cubin outputs on the same GPU were bit-exact. Five basic checks, 17 capability cases, 19 loading cases, 10 kernel-configuration cases, and four marker texture formats passed.

Both games selected SM86 cubin and executed 2X / 4X. Wukong also covered disabling frame generation and enabling it again. Neither session recorded failed SM86 routed kernel creation or failed kernel launches. Game PTX and 3X were not tested; their execution coverage comes from the offline matrix.

Comparisons against the fixed references still show small differences in some generated frames, with a maximum observed channel difference of **3/255**. The exact numerical cause has not been located. The raw strict result remains `passed=false`; this release accepts that known issue based on the recorded RTX 3080 Ti checks and gameplay. The manifest records release acceptance separately from strict validation. **The normal `-Validate` workflow still stops installation and packaging when strict validation fails.**

The tested `version.dll` SHA256 is:

```text
03d445237d519ac48cd9226278a0f07aecd7ac597697697eb64404e1d51b3c5a
```

D3D12 debug-layer validation requires Windows Graphics Tools. It was not enabled for the recorded run; `d3d12_debug_errors=null` means it was not checked. Frame pacing, latency, temporal artifacts, and long-duration stability have not been measured separately.

## Project layout

```text
dlssg/
  build.cmd / build.ps1   Toolchain discovery, build, checks, and packaging
  CMakeLists.txt          Standalone CMake project
  src/
    loader.cpp           Proxy exports, DLSSG loading hooks, and INI handling
    bundle.cpp           Embedded payload extraction, caching, and hash checks
    marker.cpp           Optional generated-frame markers
    backends/310_1/      Backend matched to the embedded runtime
    generated/           Generated proxy exports and GPU resource indices
  assets/
    runtime/             Original runtime, models, host graph, and processing
    kernels/             72 groups of SM86 cubin/PTX resources
  config/                Default INI
  third_party/           Detours, JSON, DirectX, and NGX headers/libraries
  cmake/                 Resource embedding, check setup, and packaging
  scripts/               GPU validation, export generation, and local build helper
  tests/
    native/              Native checks and D3D12/NGX tests
    python/              Capability, loading, and image comparisons
    reference/           Bundled reference inputs and outputs
  docs/                  Installation, architecture, and validation documentation
  out/                   Tooling, build caches, and validation output
  dist/                  Deployment artifacts
```

Models and GPU resources are fixed build inputs. Updating the embedded DLSSG version requires matching backend addresses, structures, kernels, and runtime hash constraints, followed by new validation. Replacing only `assets/runtime/nvngx_dlssg.dll` is rejected by the build checks.

Implementation details are in the [architecture guide](docs/ARCHITECTURE.md) (Chinese). License terms are in [LICENSE.md](LICENSE.md), with third-party attribution in [THIRD_PARTY_NOTICES.txt](THIRD_PARTY_NOTICES.txt).

## AMD research status

The RX 7900 XTX / RDNA 4 WMMA prototype and plan remain in the development workspace under `experimental/amd_wmma`. Development is paused. The current release does not support AMD GPUs, and this independent research is not part of the SM86 source distribution.
