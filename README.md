# Ludus Fedora Kernel

Fedora 44 RPM builds of the Ludus gaming kernel, based on
[linux-tkg](https://github.com/Frogging-Family/linux-tkg).

This repository is Fedora-only. It does not build or publish Debian packages.

## Variants

Every kernel release contains ten builds: five CPU targets and two toolchains.

| CPU target | Minimum/target CPU |
| --- | --- |
| `x86-64` | Generic x86-64 (v1) |
| `x86-64-v2` | x86-64-v2 |
| `x86-64-v3` | x86-64-v3 |
| `x86-64-v4` | x86-64-v4 |
| `znver3` | AMD Zen 3 |

The `llvm-thin` build uses LLVM and ThinLTO. The `gcc` build uses GCC without
LTO. BORE, BBR, the performance governor, 1000 Hz ticks and the repository's
`modprobed/generic/modprobed.db` are enabled by default.

Each variant publishes two packages:

- `kernel`: kernel image and modules.
- `kernel-devel`: files required by akmods and external kernel modules.

The generated `kernel-headers` package is intentionally not published because
Fedora provides the userspace UAPI headers.

The RPMs are produced by linux-tkg through the upstream kernel `binrpm-pkg`
target. They are installable Fedora packages, but they do not use Fedora's full
official `kernel.spec` package split.

## Automation

`watch-kernel.yml` checks kernel.org every three hours. A build is dispatched
only when all of the following are true:

- The complete GitHub release does not already exist.
- No other Fedora kernel build is queued or running.
- The pinned linux-tkg commit supports the detected kernel series.

`build-fedora-kernel.yml` runs the ten builds in `fedora:44` containers on
GitHub-hosted Ubuntu runners. RPMs are published only after every matrix job
succeeds and the release contains exactly twenty packages.

Automatic releases use tags such as
`ludus-kernel-7.1.3-bore-modprobed`, keeping different schedulers and module
strategies isolated from one another.

linux-tkg is pinned by full commit SHA in `.github/linux-tkg-ref.txt`. Updating
that file is required before the watcher can build a kernel series unsupported
by the current pin.

Compiler objects are cached independently for each CPU and toolchain. Each
cache is limited to 1 GiB so the ten variants can fit within the usual GitHub
Actions repository cache quota.

## Manual Builds

Run the `Build Fedora 44 RPM kernels` workflow and select:

- A kernel tag such as `v7.1.3`, or a series such as `7.1-latest`.
- The module strategy (`modprobed`, `diet` or `full`).
- A scheduler supported by that kernel series.
- An optional built-in kernel command line and release tag.

## Installation

Install the matching `kernel` and `kernel-devel` packages from exactly one CPU
and toolchain variant:

```bash
sudo dnf install ./kernel-<variant>.x86_64.rpm ./kernel-devel-<variant>.x86_64.rpm
```

Keep an official Fedora kernel installed as a fallback. CPU-specific packages
are still identified as `x86_64` RPMs, so DNF cannot prevent installation on an
incompatible processor.

These kernels and their modules are not signed for Fedora Secure Boot. Secure
Boot must be disabled or a separate kernel/module signing and MOK enrollment
workflow must be provided.

The default built-in command line disables CPU vulnerability mitigations and
split-lock detection for performance. This is a deliberate security tradeoff.
