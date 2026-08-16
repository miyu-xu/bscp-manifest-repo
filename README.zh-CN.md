# BSCP 工作区 Manifest

简体中文 | [English](README.md)

本仓库是 BSCP 的规范主入口。BSCP 是面向 Linux/KVM、macOS/HVF 和 Windows/WHPX 的跨平台、
安全隔离 Android 通用算力平台。Microdroid 是首要 Guest 与发布基线；基于 Cuttlefish 的完整
Android 工作流仅作为 Framework、图形和设备验证的附带兼容路径。

manifest 负责仓库组合和固定分支选择；根仓库负责构建编排与运维；组件源码保留在独立仓库，
保证每个修改只有一个 Git 所有者，并可追溯发布来源。

## 分支选择

| 分支 | 用途 |
| --- | --- |
| `main` | 以 Microdroid 为主的规范发布入口 |
| `bscp-android-15.0.0_r14` | 当前 Android 基线的版本化别名 |
| `hd-feature` | 可选产品集成；增加 HD 仓库并选择匹配的 crosvm/gfxstream 分支 |

主分支不包含 HD 产品代码或仓库入口。不要把 `hd-feature` 直接合入主分支；通用修复应先进入
其所属组件主分支。

## 初始化与同步

安装 AOSP `repo`，在空目录中通过 HTTPS 初始化：

```bash
mkdir bscp-workspace && cd bscp-workspace
repo init -u https://github.com/miyu-xu/bscp-manifest-repo.git -b main
repo sync -c -j4 --fail-fast
repo status
```

可选产品工作区将 `-b main` 替换为 `-b hd-feature`。SSH 用户可以在个人 Git 配置中做 URL
重写；manifest 保持 HTTPS，便于跨平台和无凭据默认检出。

## 工作区项目

- `.` (`bscp-root`)：构建、打包、运行包装器、测试和中英文运维文档。
- `packages/modules/Virtualization`：`vm`、`virtmgr`、libvmclient、载荷和 Microdroid 生命周期。
- `external/crosvm`：KVM/HVF/WHPX 虚拟机监控器与 virtio 设备。
- `frameworks/native`：可移植 binder RPC 支持。
- `hardware/google/gfxstream` 与 `hardware/google/aemu`：可选图形加速主机栈。
- `system/core`：固定的 Android core 兼容子集。
- flatbuffers、lz4、minijail 与 rutabaga_gfx：固定的支撑依赖。

每个项目保留自身许可证和 README；根仓库及有修改的组件提供英文与简体中文集成说明。

## 主机环境

通用依赖包括 Git、Python 3、CMake、Ninja、rustup，以及 Android 产物和实例级可写镜像所需
磁盘空间。

- Linux：KVM 权限与受支持 C/C++ 工具链；只有网络场景需要 TAP 权限。
- macOS：Apple Silicon、Xcode Command Line Tools、Hypervisor.framework entitlement 和
  arm64 `com.android.virt` APEX 树。
- Windows：启用 WHPX、PowerShell、MinGW-w64、CMake/Ninja、固定的 GNU 主机 Rust 工具链
  和可发现的 `libclang.dll`。

同步后进入根仓库，先构建并运行 `validate-prereqs`，再运行 `run-microdroid`。Windows 使用
`build_all.bat` 与 PowerShell 包装器；macOS 先提供 arm64 APEX 树。详细命令以根仓库 README
为准。

## 安全与发布纪律

- 发布标签前把每个项目固定到已审查 revision。
- 调试策略、无限制网络、主机挂载和未加密存储只能是显式开发能力，不能作为生产默认值。
- 签名密钥与凭据保存在工作区之外；不得提交 `.repo/`、`out/`、日志、Guest 数据或生成二进制。
- 任何完整 Android 验证之前，必须先通过 Microdroid 平台回归。
- 发布重写分支前，同时审查 `repo status`、提交身份、周末发布时间和中英文文档。

本工作区历史已按发布要求重写。远端强制更新相应分支后，旧检出必须重新初始化，或显式获取
并重置到新历史。
