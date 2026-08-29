# BSCP workspace manifest

[简体中文](README.zh-CN.md) | English

This is the canonical entry point for BSCP: a cross-platform, security-isolated Android compute
platform for Linux/KVM, macOS/HVF, and Windows/WHPX. Microdroid is the primary guest and release
baseline. A Cuttlefish-derived full Android workflow is available as an optional compatibility
path for framework, graphics, and device testing.

The manifest owns repository composition and pinned branch selection. The root repository owns
build orchestration and operations; component source remains in independent repositories so every
change has one Git owner and auditable provenance.

## Author's perspective and long-term direction

> This section records the author's technical outlook and intended direction. It is not a claim
> that every capability is complete in the current release. Component documentation and evidence
> produced on each target platform remain authoritative for support status and limitations.

As Android devices continue to gain CPU, GPU, NPU, memory, and energy efficiency, they are evolving
from single-purpose application hosts into general compute nodes capable of running multiple
isolated workloads. Once hardware virtualization, IOMMU support, verified boot, and protected
memory become platform primitives, virtual machines are a natural workload boundary for improving
utilization, fault containment, software-supply-chain consistency, and the protection of sensitive
tasks.

BSCP aims to provide a common control plane, a stable guest contract, and portable VMM adapters so
Microdroid workloads can span Windows, Linux, and macOS and eventually extend to Android hosts that
provide the Android Virtualization Framework. The objective is not to maintain a different
application system for every host. It is to run the same signed and verified Microdroid
guest/payload delivery without changing guest content when these conditions are met:

- the host and guest instruction sets match, or the release provides equivalent architecture
  variants;
- the kernel, initrd, partitions, virtio devices, and Binder/vsock interfaces follow the same
  versioned guest ABI;
- the KVM, HVF, WHPX, or Android AVF adapter supplies the required CPU, memory, interrupt, and
  virtual-device semantics;
- host-specific permission, key, network, graphics, and lifecycle differences remain in the host
  layer instead of leaking into application payloads.

Accordingly, “one guest artifact set” primarily means one source, configuration, security policy,
and reproducible release process. x86_64 and arm64 still require architecture-specific binary
images. Where the CPU architecture and guest ABI match, the goal is to reuse the exact signed
artifact rather than customize the guest for each host OS.

On Android devices with pKVM support, BSCP may evolve toward protected VMs. With verified pvmfw,
AVB/Verified Boot, protected memory, trusted key and attestation services, a restricted device
exposure policy, and constrained host/VMM process boundaries, Microdroid could become a
hardware-hardened execution environment. This path is conditional: availability depends on the
SoC, kernel, firmware, Android release, and device-vendor configuration. Virtualization extensions
alone are not evidence of pVM-grade security.

Graphics and general-purpose compute are a planned next stage. The author intends to add controlled
Android Vulkan support to Microdroid by connecting the guest Vulkan loader and driver,
virtio-gpu, rutabaga/gfxstream, host Vulkan/Metal/D3D backends, and explicit memory allocation,
synchronization, quota, and failure-isolation policies. Once these pieces pass cross-platform
correctness, security, and performance validation, Microdroid can move beyond headless payloads
and operate as a schedulable cross-platform compute endpoint for graphics, media, AI inference,
and other GPU or general compute workloads.

The long-term objective is for BSCP to serve as compute infrastructure: immutable and verifiable
guest images as delivery units, VMs as tenant and failure boundaries, and a common control plane
for scheduling, quotas, observability, audit, upgrades, and rollback. Success is not merely the
ability to boot a VM. It is the ability to reproduce a workload across platforms, bound its
resources, contain its failures, retain auditable evidence, and explicitly reject unsupported
security requirements instead of silently degrading them.

## Choose a branch

| Branch | Purpose |
| --- | --- |
| `main` | Canonical Microdroid-first release entry point |
| `bscp-android-15.0.0_r14` | Versioned alias of the current Android baseline |
| `hd-feature` | Optional product integration; adds the HD repository and matching crosvm/gfxstream branches |

The main branches contain no HD product code or repository entry. Do not merge `hd-feature` into
main; promote generic fixes through their owning component branch instead.

## Initialize and synchronize

Install the AOSP `repo` launcher, choose an empty directory, and initialize over HTTPS:

```bash
mkdir bscp-workspace && cd bscp-workspace
repo init -u https://github.com/miyu-xu/bscp-manifest-repo.git -b main
repo sync -c -j4 --fail-fast
repo status
```

For the isolated optional product branch, replace `-b main` with `-b hd-feature`. SSH users may
configure URL rewriting in their own Git configuration; the manifest remains HTTPS-based for
portable, credential-neutral checkout.

## Workspace projects

- `.` (`bscp-root`): build, packaging, runtime wrappers, tests, and bilingual operations docs.
- `packages/modules/Virtualization`: `vm`, `virtmgr`, libvmclient, payload and Microdroid lifecycle.
- `external/crosvm`: KVM/HVF/WHPX virtual machine monitor and virtio devices.
- `frameworks/native`: portable binder RPC support.
- `hardware/google/gfxstream` and `hardware/google/aemu`: optional accelerated graphics host stack.
- `system/core`: pinned Android core compatibility subset.
- flatbuffers, lz4, minijail, and rutabaga_gfx: pinned supporting dependencies.

Each project retains its own license and README. Root and modified component repositories provide
English and Simplified Chinese integration documentation.

## Host environment

Common requirements are Git, Python 3, CMake, Ninja, rustup, and sufficient disk space for Android
artifacts and per-instance writable images.

- Linux: KVM access and a supported C/C++ toolchain; TAP permissions are optional and needed only
  for network scenarios.
- macOS: Apple Silicon, Xcode command-line tools, Hypervisor.framework entitlement, and an arm64
  `com.android.virt` APEX tree.
- Windows: WHPX enabled, PowerShell, MinGW-w64, CMake/Ninja, the pinned GNU-hosted Rust toolchain,
  and discoverable `libclang.dll`.

After synchronization, follow the root [English README](https://github.com/miyu-xu/bscp-root) or
its `README.zh-CN.md`. The primary flow is:

```bash
./build_all.sh
./scripts/vm_linux.sh --command validate-prereqs
./scripts/vm_linux.sh --command run-microdroid
```

On Windows use `build_all.bat`, `vm_windows.ps1 -Command validate-prereqs`, and
`run_microdroid_windows.ps1`. On macOS provide the arm64 APEX tree and use `vm_macos.sh`.

## Security and release discipline

- Pin every project to a reviewed revision before tagging a release.
- Treat debug policy, unrestricted networking, host mounts, and unencrypted storage as development
  features, never implicit production defaults.
- Keep signing keys and credentials outside the workspace; do not commit `.repo/`, `out/`, logs,
  guest data, or generated binaries.
- Run the Microdroid platform regression before any optional full Android validation.
- Review `repo status`, project commit identities, weekend publication timestamps, and bilingual
  documentation together before publishing rewritten branches.

History in this workspace has been rewritten for release hygiene. Existing clones must reinitialize
or fetch and reset explicitly after the corresponding remote branches are force-updated.

## License and commercial use

BSCP-original manifest configuration and documentation are available under the
[PolyForm Noncommercial License 1.0.0](LICENSE). Commercial use requires prior written
authorization under a separate [commercial license](COMMERCIAL_LICENSING.md).

This restriction makes BSCP-original material source-available, not OSI-approved open-source
software. Every repository selected by the manifest retains its own license and notices. See the
[workspace license policy](LICENSE_POLICY.md) before building, packaging, or redistributing the
combined workspace.
