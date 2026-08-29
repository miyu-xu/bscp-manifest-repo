# BSCP 工作区 Manifest

简体中文 | [English](README.md)

本仓库是 BSCP 的规范主入口。BSCP 是面向 Linux/KVM、macOS/HVF 和 Windows/WHPX 的跨平台、
安全隔离 Android 通用算力平台。Microdroid 是首要 Guest 与发布基线；基于 Cuttlefish 的完整
Android 工作流仅作为 Framework、图形和设备验证的附带兼容路径。

manifest 负责仓库组合和固定分支选择；根仓库负责构建编排与运维；组件源码保留在独立仓库，
保证每个修改只有一个 Git 所有者，并可追溯发布来源。

## 作者观点与发展愿景

> 本节描述项目作者对技术演进的判断和长期方向，不等同于当前版本已经完成的支持承诺。
> 实际能力、限制和发布状态以各组件文档及目标平台验证结果为准。

随着 Android 设备的 CPU、GPU、NPU、内存容量和能效持续提升，移动终端正在从单一应用载体
演进为能够承载多租户任务的通用计算节点。在硬件虚拟化、IOMMU、可信启动和内存保护逐步
成为平台基础能力之后，以虚拟机作为工作负载边界，是提升资源利用率、故障隔离、软件供应链
一致性和敏感任务安全性的自然方向。

BSCP 希望建立一套由统一控制面、稳定 Guest 契约和可移植 VMM 适配层组成的运行框架，使
Microdroid 工作负载能够覆盖 Windows、Linux、macOS，并进一步延伸到具备 Android
Virtualization Framework 的 Android 主机。其核心目标不是为每个平台维护不同的业务系统，
而是让同一份经过签名和验证的 Microdroid Guest/Payload 交付物，在满足以下前提时无需修改
Guest 内容即可运行：

- Host 与 Guest 的 CPU 指令集匹配，或发布集中提供内容一致的架构变体；
- kernel、initrd、分区、virtio 设备和 Binder/vsock 接口遵循同一版本化 Guest ABI；
- KVM、HVF、WHPX 或 Android AVF 适配层能够提供所需的 CPU、内存、中断和虚拟设备语义；
- 平台特有的权限、密钥、网络、图形和生命周期差异由 Host 层吸收，不泄漏进业务 Payload。

因此，“同一套 Guest 产物”更准确地指同一源码、配置、安全策略和可复现发布流程；当 x86_64
与 arm64 指令集不同时，仍需要分别构建对应的二进制镜像。对于 CPU 架构相同且 Guest ABI
兼容的平台，目标是复用同一份签名产物，而不是为不同 Host 修改 Guest。

在支持 pKVM 的 Android 设备上，BSCP 未来可以探索 protected VM。若同时具备受验证的
pvmfw、AVB/Verified Boot、受保护内存、可信密钥与证明服务、严格的设备暴露策略，以及受限
的 Host/VMM 进程边界，Microdroid 可进一步成为具备硬件强化隔离能力的执行环境。这是一条
有条件的演进路径；是否可用取决于 SoC、内核、固件、Android 版本和设备厂商配置，不能仅凭
存在虚拟化扩展就宣称已经达到 pVM 安全等级。

图形与通用计算是下一阶段重点。当前 Host 侧的 virtio-gpu、rutabaga/gfxstream 及平台图形
后端已经具备支撑 Microdroid Vulkan 离屏渲染的基础能力，不需要重新设计 Host 图形架构。
对于已经按现有图形 profile 构建并装入 gfxstream/ANGLE runtime 的 Host 发行物，剩余代码
改动主要集中在 Guest：在 Microdroid 镜像中启用并打包 Vulkan loader、Guest ICD/驱动及相关
运行库，配置设备访问、SELinux/权限和 VM GPU profile，再完成跨平台一致性、安全性、资源
配额与性能门禁。未包含可选图形 runtime 的默认 Host 包只需启用现有构建/打包配置，不代表
需要开发新的 Host 后端。

离屏路径不依赖窗口或显示 scanout；Payload 可以把 Vulkan image/buffer 作为计算与渲染目标，
再通过受控 readback 或资源导出获取结果。上述 Guest 能力和端到端证据完成后，Microdroid 将
不再局限于无图形 Payload，而可作为可调度的跨平台算力端点，承载图形、媒体、AI 推理和其他
GPU/通用计算任务。

长期来看，BSCP 希望成为算力基础设施的一部分：以不可变、可验证的 Guest 镜像作为交付单元，
以虚拟机作为租户和故障边界，并在统一控制面中提供调度、配额、可观测性、审计、升级和回滚。
项目评价标准不是“能否启动一个虚拟机”，而是同一工作负载能否跨平台复现、资源能否被明确
约束、故障能否被隔离、运行证据能否被审计，以及安全能力不足时系统能否显式拒绝而非静默
降级。

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

## 许可证与商业使用

BSCP 原创 manifest 配置和文档采用
[PolyForm Noncommercial License 1.0.0](LICENSE)。商业使用必须事先取得书面授权，并另行
签订或获得[商业许可证](COMMERCIAL_LICENSING.zh-CN.md)。

由于限制商业用途，BSCP 原创材料属于 source-available，而不是 OSI 批准的开源软件。manifest
选择的每个独立仓库继续保留自身许可证和声明。构建、打包或再分发组合工作区之前，请阅读
[工作区许可证策略](LICENSE_POLICY.zh-CN.md)。
