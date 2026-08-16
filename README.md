# BSCP workspace manifest

[简体中文](README.zh-CN.md) | English

This is the canonical entry point for BSCP: a cross-platform, security-isolated Android compute
platform for Linux/KVM, macOS/HVF, and Windows/WHPX. Microdroid is the primary guest and release
baseline. A Cuttlefish-derived full Android workflow is available as an optional compatibility
path for framework, graphics, and device testing.

The manifest owns repository composition and pinned branch selection. The root repository owns
build orchestration and operations; component source remains in independent repositories so every
change has one Git owner and auditable provenance.

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
