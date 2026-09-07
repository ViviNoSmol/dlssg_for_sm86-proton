# DLSSG 独立项目

简体中文 | [English](README.en.md)

Windows x64 / D3D12 的 DLSSG SM86 融合版。代理 DLL 内嵌配套的原版 DLSSG 310.1、模型和计算管线，以及 SM86 后端；默认 `Mode=Bundled`。游戏自带的 DLSSG 版本不参与计算后端匹配。

本目录包含构建所需的源码、第三方头文件/库、运行库和 GPU 资源，可以整体复制到其他位置。构建不依赖外层研究目录、游戏安装目录或旧 CMake 缓存。

## 首个 milestone

当前 **Release（2026-09-07）** 是首个 milestone，基于 RTX 3080 Ti / SM86、驱动 591.86 的离线检查和游戏实测。用户反馈当前游玩相对稳定。

| 用户手动实测 | 关闭插帧 | 2X | 4X |
|---|---:|---:|---:|
| 黑神话：悟空 | 50 FPS | 80 FPS | 150 FPS |
| 赛博朋克 2077，路径追踪 | 35 FPS | 60 FPS | 100 FPS |

发布目录为 `dist/Release/`，部署包为 `dist/dlssg-release-x64.zip`，源码包为 `dist/dlssg-source.zip`，均附 SHA256。部署包使用当前实测 DLL，INI 默认关闭帧标记。两款游戏按统一格式记录在 [测试文档](docs/VALIDATION.md) 中；原始 evidence、日志和截图仅保留在开发工区。

## 一键构建

全新 Windows 机器的安装与构建流程见 [开发环境搭建与构建](docs/DEVELOPMENT.md)。

安装 Visual Studio 的“使用 C++ 的桌面开发”、Windows SDK 和 C++ CMake 工具，然后在本目录执行：

```powershell
.\build.cmd
```

也可以直接运行 `build.ps1`。脚本自动发现 Visual Studio、配套 CMake 和 MSVC x64 工具链，依次完成编译、5 项基础检查、安装到产物目录及 ZIP 打包。RTX 3080 Ti 实测构建使用 MSVC 19.44、Windows SDK 10.0.26100.0 和 Ninja Multi-Config。基础检查包含无需 GPU 的内核选择规则测试。

默认构建无需 Python、CUDA Toolkit 或 GPU 推理环境。内核使用 `assets/kernels` 中已有的 SM86 cubin/PTX；此构建脚本不重新生成内核。

常规构建输出路径如下：

| 输出 | 内容 |
|---|---|
| `dist/Release/version.dll` | 主代理，内嵌运行库和后端 |
| `dist/Release/alternatives/winmm.dll` | 备用代理入口 |
| `dist/Release/dlssg_sm86.ini` | 用户配置 |
| `dist/Release/manifest.json` | 文件哈希、内嵌配对信息和检查结果 |
| `dist/Release/docs/VALIDATION.md` | 游戏实测、执行检查和已知项 |
| `dist/dlssg-release-x64.zip` | 部署包 |
| `dist/dlssg-release-x64.zip.sha256` | 部署包校验值 |

默认构建只记录基础检查通过，GPU 状态为 `performed: false`。使用下面的 `-Validate` 才会执行 GPU 比较并把结果与当前 DLL 哈希绑定；历史验证不会自动算作新二进制的验证。

## 选择 PTX / cubin

在 `dlssg_sm86.ini` 中设置：

```ini
[Compatibility]
KernelImage=PTX
ForceSM86Route=0
SimulateAmpere=0
```

这会让 3080 Ti 等真实 SM86 显卡使用 PTX JIT。默认 `Auto` 在 SM86 上使用预编译 cubin；`Cubin` 显式要求使用 SM86 cubin。此项不单独强制开启其他架构的 SM86 路由，修改后重启游戏。完整行为和日志字段见 [安装说明](docs/INSTALL.md)。

## 可选 GPU 验证

准备 Python 3.10+、NumPy 和 NVIDIA GPU / 驱动环境后运行。RTX 3080 Ti 的实机检查覆盖 Auto、PTX、Cubin 执行及本机输出一致性；验证器另执行固定参考比较：

```powershell
.\build.cmd -Validate -Python "C:\path\to\python.exe"
.\build.cmd -Validate -RequireSM86 -Python "C:\path\to\python.exe"
```

验证器自动查找驱动中的 `_nvngx.dll`，并使用同目录的另一版 `nvngx_dlssg.dll` 做跨版本对照。若当前驱动没有提供不同版本，或者存在多个驱动目录，可以显式指定：

```powershell
.\build.cmd -Validate -Python "C:\path\to\python.exe" `
  -NgxRuntime "C:\path\to\_nvngx.dll" `
  -ComparisonDll "C:\path\to\another-version\nvngx_dlssg.dll"
```

`-RequireSM86` 要求 D3D12 适配器匹配的真实 CUDA 设备为 SM86。结果保存在独立的 `out/validation/<配置>-<时间>/`，各阶段标准输出、错误输出和退出码保存在 `stages/`。验证涵盖能力上报、加载/缓存、图像一致性及四种纹理格式的生成帧标记；数值差异保留在报告中，严格检查失败时停止安装到 dist 和打包。D3D12 debug layer 检查需要 Windows Graphics Tools；未启用时报告明确记为未检查。

2026-09-07 已在 RTX 3080 Ti / SM86、驱动 591.86 上完成离线验证，两个模拟开关均为 0：18 个计算场景及标记场景执行成功，Auto/PTX/Cubin 的 176 张同机输出比较全部位一致。17 项能力、19 项加载、10 项内核配置和 4 种纹理格式检查通过；本机未启用 D3D12 debug layer。结果见 [测试记录](docs/VALIDATION.md)。

两款游戏使用同一版代理，Auto 选择 SM86 cubin，日志均记录了 2X / 4X，路由创建和内核启动失败计数为 0。悟空还覆盖了关闭后重新开启；赛博朋克用户实测条件为路径追踪。上表帧率是用户手动观察值，尚无规范化性能或长期稳定性测量。两款游戏的条件、帧率和执行结果统一见 [测试记录](docs/VALIDATION.md)。

以上结论对应当前 Release 中的实测代理，SHA256 为 `03d445237d519ac48cd9226278a0f07aecd7ac597697697eb64404e1d51b3c5a`。固定参考逐位比较仍未通过，原始 `passed=false` 保留；本次按用户要求接受该已知项，并在 manifest 中分别记录严格检查状态和发布接受状态。常规 `-Validate` 仍在严格比较失败时停止打包。

## 目录

```text
dlssg/
  build.cmd / build.ps1      自动发现工具链、构建、检查和打包
  CMakeLists.txt             独立 CMake 入口
  src/
    loader.cpp              代理导出转发、DLSSG 加载拦截、INI
    bundle.cpp              内嵌配对释放、缓存和哈希校验
    marker.cpp              生成帧标记
    backends/310_1/          与内置运行库匹配的 SM86 后端
    generated/              已生成的代理导出和 GPU 资源索引
  assets/
    runtime/                原版运行库，包含模型/host graph/前后处理
    kernels/                72 组 SM86 cubin/PTX 资源
  config/                   默认 INI
  third_party/              Detours、JSON、DirectX、NGX 头文件/库
  cmake/                    资源嵌入、基础检查准备、打包
  scripts/                  GPU 验证、代理导出、本地工具链入口
  tests/
    native/                 原生检查和 D3D12/NGX 测试
    python/                 能力、加载和图像比较
    reference/              随项目携带的原版输出参考
  docs/                     安装、架构、验证记录与摘要证据
  out/                      构建缓存、测试日志和捕获（不纳入源码）
  dist/                     部署产物（不纳入源码）
```

模型和 GPU 资源作为固定输入参与构建。要更换内置 DLSSG，需同时适配后端的内部地址/结构和内核，再更新运行库哈希约束并重新验证；仅替换 `assets/runtime/nvngx_dlssg.dll` 会被构建检查拒绝。

## 构建选项

```powershell
.\build.cmd -BuildDirectory "out\custom build"
.\build.cmd -Configuration RelWithDebInfo
.\build.cmd -CMake "C:\path\to\cmake.exe" -Generator "Visual Studio 17 2022"
```

相对 `BuildDirectory` 以本项目为基准，脚本可以从其他工作目录调用。CMake 支持 Visual Studio 或已配置 MSVC 环境的 Ninja Multi-Config，支持 Release、Debug 和 RelWithDebInfo；产物分配置存放。M1 的 Release 通过 5 项基础检查。`out/` 和 `dist/` 已写入本目录的 `.gitignore`。当前机器保留了 `out/tools` 中已安装的 MSVC、SDK、Python / NumPy、CMake 和 Ninja，可用 `scripts/build-local.ps1` 构建，或加 `-Validate` 运行物理 SM86 验证。

安装使用见 [安装说明](docs/INSTALL.md)，实现分工见 [架构说明](docs/ARCHITECTURE.md)，当前实测证据见 [验证记录](docs/VALIDATION.md)。构建脚本仅生成部署包，游戏运行使用 DLL + INI。

## AMD 移植研究（暂停）

RX 7900 XTX / RDNA 4 的 WMMA 原型和计划保留在开发工区的 `experimental/amd_wmma`。实现已暂停，当前发布 DLL 尚不支持 AMD；这部分独立研究不包含在 SM86 源码部署快照中。
