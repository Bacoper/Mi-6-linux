# Xiaomi Mi 6 (`sagit`) — Native Fedora Linux Project

![OS](https://img.shields.io/badge/OS-Fedora%20Linux%2045%20ARM64-294172?style=for-the-badge&logo=fedora&logoColor=white)
![SoC](https://img.shields.io/badge/SoC-Snapdragon%20835%20(MSM8998)-red?style=for-the-badge)
![GPU](https://img.shields.io/badge/Adreno%20540-Hardware%20Accelerated-brightgreen?style=for-the-badge)
![Kernel](https://img.shields.io/badge/Kernel-6.0.0--sagit--clockfix-blue?style=for-the-badge&logo=linux&logoColor=white)
![Milestone](https://img.shields.io/badge/Milestone-Day%206%20%7C%203D%20Cube%20Rendered-success?style=for-the-badge)

> **Day 6 Milestone Report — November 25, 2026**  
> **Bare-Metal Linux Bring-Up on the 2017 Xiaomi Mi 6 (`sagit`)**

After six days of kernel debugging, device-tree modifications, backporting upstream patches, and hardware validation, we have successfully brought up **native Fedora Linux 45 ARM64** on the **Xiaomi Mi 6 (`sagit`)** with working Wi-Fi, touchscreen, internal UFS storage, native MIPI-DSI display output, and **visible hardware-accelerated 3D graphics on the Adreno 540 GPU**.

This is a **bare-metal Linux installation** running directly on the hardware via EDK2 UEFI and GRUB—not an Android container, chroot, or emulation layer.

<p align="center">
  <img src="docs/images/kmscube-sagit.jpg" alt="Hardware-accelerated kmscube running natively on the Xiaomi Mi 6 display" width="420" />
  <br />
  <em>Physical hardware verification: <code>kmscube</code> running natively on the Xiaomi Mi 6 JDI R63452 panel via Mesa Freedreno (<code>FD540</code>) on November 25, 2026.</em>
</p>

---

## Table of Contents

- [1. Hardware & Software Configuration](#1-hardware--software-configuration)
- [2. Overall Hardware Functionality](#2-overall-hardware-functionality)
- [3. Development History & Bring-Up Phases](#3-development-history--bring-up-phases)
- [4. Qualcomm Firmware & Networking](#4-qualcomm-firmware--networking)
- [5. The Adreno 540 GPU Challenge](#5-the-adreno-540-gpu-challenge)
- [6. Major Kernel & GPU Patches Applied](#6-major-kernel--gpu-patches-applied)
- [7. GPU Progression Summary](#7-gpu-progression-summary)
- [8. Automated GPU Validation Suite](#8-automated-gpu-validation-suite)
- [9. Solving the Final On-Screen `kmscube` Blocker](#9-solving-the-final-on-screen-kmscube-blocker)
- [10. GPU Performance & Power Management](#10-gpu-performance--power-management)
- [11. Kernel Auditing & Initramfs Overlay Engineering](#11-kernel-auditing--initramfs-overlay-engineering)
- [12. Known Limitations & Remaining Work](#12-known-limitations--remaining-work)
- [13. Golden Working Kernel Configuration](#13-golden-working-kernel-configuration)
- [14. Development Roadmap](#14-development-roadmap)
- [15. Project Summary](#15-project-summary)

---

## 1. Hardware & Software Configuration

| Component | Configuration |
| :--- | :--- |
| **Device** | Xiaomi Mi 6 (`sagit`, 2017) |
| **SoC** | Qualcomm Snapdragon 835 (`MSM8998`) |
| **CPU Architecture** | ARM64 / AArch64 |
| **GPU** | Qualcomm Adreno 540 |
| **Operating System** | Fedora Linux 45 ARM64 (Bare-Metal) |
| **Kernel** | Custom Linux `6.0.0-sagit-clockfix` |
| **Kernel Base Source** | MSM8998 downstream mainline tree (`90169e6e9fee29fae14aeac0a905246be5ce6304`) |
| **Boot Firmware & Loaders** | EDK2 UEFI $\rightarrow$ rEFInd $\rightarrow$ GRUB ARM64 EFI |
| **Storage & Filesystems** | Internal UFS (`ext4` rootfs, `FAT32` EFI System Partition) |
| **Graphics Stack** | Linux `drm/msm` + Mesa Freedreno `26.2.3` (`FD540`) |
| **Graphics APIs** | EGL `1.5`, OpenGL ES `3.1` |
| **Display Panel** | JDI `R63452` (`1080 × 1920` @ `60 Hz` over MIPI DSI0) |
| **Current Interface** | `kmscon` DRM terminal with native 3D graphics presentation |

---

## 2. Overall Hardware Functionality

### Subsystem Readiness Overview

```text
Boot & Native Linux Infrastructure   [████████████████████] 100%
Internal UFS Storage & Filesystems   [████████████████████] 100%
Native Display & Touchscreen         [████████████████████] 100%
GPU Hardware Acceleration            [██████████████████░░]  90%
Wi-Fi & Internet Networking          [██████████████████░░]  90%
Graphical Desktop Environment        [███░░░░░░░░░░░░░░░░░]  15%
```
*(Percentages reflect estimated development bring-up progress rather than formal test coverage.)*

### Detailed Hardware Support Matrix

| Subsystem / Feature | Status | Details |
| :--- | :---: | :--- |
| **ARM64 CPU** | ✅ Working | All cores operational |
| **UEFI / rEFInd / GRUB** | ✅ Working | Reliable chainloaded boot with explicit DTB loading |
| **Internal UFS Storage** | ✅ Working | Root filesystem mounted via UUID (`ext4`) |
| **USB Host & Mass Storage** | ✅ Working | USB keyboard and host peripherals functional |
| **Touchscreen** | ✅ Working | Native touch input operational |
| **LCD Display & Backlight** | ✅ Working | Native `1080 × 1920 @ 60 Hz` scanout + WLED backlight |
| **Adreno 540 GPU** | ✅ Working | Detected as `FD540`, hardware 3D rendering confirmed |
| **Physical 3D Display Output** | ✅ Working | Visually confirmed via `kmscube` on screen |
| **GPU Runtime PM** | ✅ Passed | Autosuspend/resume validated across multiple cycles |
| **Wi-Fi (2.4 GHz & 5 GHz)** | ✅ Working | `WCN3990` (`ath10k_snoc`) scanning, connecting, online |
| **Qualcomm Remoteprocs** | ⚠️ Partial | ADSP, SLPI, and Modem start; modem watchdog needs tuning |
| **Bluetooth** | ❌ Errors | Firmware loading and teardown errors present |
| **Battery / Fuel Gauge** | ❌ Errors | Fuel-gauge kernel probe errors |
| **Charging Management** | ⏳ Unverified | Needs dedicated testing |
| **Audio (Speakers / Mic)** | ⏳ Untested | DSP starts, ALSA/UCM audio routing not yet established |
| **Cellular (Calls / SMS / Data)** | ⏳ Untested | Dependent on modem stability work |
| **GPS / Cameras / Sensors** | ⏳ Untested | Fingerprint, accelerometer, gyroscope, and cameras pending |
| **System Deep Sleep** | ⏳ Untested | Full device suspend/resume not yet validated |
| **Vulkan (`Turnip`)** | ❌ Not Ready | OpenGL ES `3.1` working; Vulkan A540 not yet established |
| **Wayland / KDE Plasma** | 🎯 Planned | Next major milestone after `kmscon` / `kmscube` |

---

## 3. Development History & Bring-Up Phases

### Phase 1 — Booting Linux & Solving Device-Tree Handoff
Early testing across `5.15.0-msm8998-tiny`, `6.18.64-msm8998-tiny`, and `6.0.0-sagit-fedora` suffered from black screens, early reboots, and ignored device trees when booting directly via EFI stub:

```text
EFI stub: Ignoring DTB from command line
```

To guarantee deterministic hardware description handoff, we standardized on an explicit GRUB boot chain and settled on a clean Linux `6.0.0` MSM8998 baseline (`90169e6e9fee29fae14aeac0a905246be5ce6304`):

```text
Xiaomi Mi 6 Hardware
  └─► EDK2 UEFI Firmware
        └─► rEFInd Boot Manager
              └─► GRUB ARM64 EFI (explicitly loads DTB + initramfs)
                    └─► Linux 6.0.0-sagit-clockfix
                          └─► Fedora Linux 45 ARM64 (Internal UFS)
```

### Phase 2 — Internal UFS Storage & Standalone Fedora Userspace
Fedora Linux 45 ARM64 was deployed directly to internal UFS partitions so the phone operates completely standalone without a host PC:

| Partition | Role | Filesystem | UUID |
| :--- | :--- | :--- | :--- |
| `sda17` (Partition 17) | EFI System Partition | `FAT32` | `6FB1-B704` |
| `sda18` (Partition 18) | Fedora Root Filesystem | `ext4` | `feb4108f-2ed1-4c46-afdf-32aca365912e` |

To prevent duplicate mount corruption and read-only `ext4` fallbacks during development, all mounting workflows were standardized around explicit UUIDs, health checks, and clean unmounts—culminating in a native login prompt:

```text
Fedora Linux 45
fedora login: sagit
```

### Phase 3 — Native Display & Touchscreen Bring-Up
Moving away from UEFI `simple-framebuffer` / `simpledrm`, we configured the native Qualcomm display pipeline:

| Hardware Block | Role |
| :--- | :--- |
| **Qualcomm MDP / DPU** | Display processing and scanout engine |
| **MIPI DSI0** | Physical display interface (exposed in Linux DRM as connector `DSI-1`) |
| **JDI `R63452`** | `1080 × 1920 @ 60 Hz` LCD panel |
| **WLED & LAB/IBB** | Backlight driver and display power rails |

Resolving endpoints, regulators, and clocks enabled native `1080 × 1920 @ 60 Hz` console output and responsive touchscreen input.

---

## 4. Qualcomm Firmware & Networking

### Wi-Fi (`WCN3990` / `ath10k_snoc`)
Initially, NetworkManager exposed only `lo`. After configuring Qualcomm remoteproc dependencies, QRTR transport, and platform daemons, `wlan0` initialized cleanly with full **2.4 GHz and 5 GHz** support and verified internet access.

### Qualcomm Userspace Daemons

| Service | Function | Status |
| :--- | :--- | :--- |
| `rmtfs` | Remote filesystem access for modem/subsystems | ✅ Running |
| `pd-mapper` | Qualcomm protection-domain mapping | ✅ Running |
| `diag-router` | Diagnostic message routing | ✅ Running |
| `tqftpserv` | Firmware & file transfers over QRTR | ✅ Enabled |

> **Note:** While Wi-Fi is stable, the cellular modem previously exhibited `DOG_HB` watchdog starvation crashes followed by recovery. Cellular stability will be addressed separately from the graphics stack.

### Device-Tree Reserved Memory Regions
Physical memory reservations were adjusted in `msm8998-xiaomi-sagit.dtb` to prevent subsystem collisions:

| Subsystem / Region | Base Address & Size |
| :--- | :--- |
| **ADSP** | `0x8b200000` (expanded to `30 MiB`) |
| **Modem** | `0x8d000000` region |
| **IPA Firmware** | `0xf7800000` |
| **GPU ZAP Shader** | `0xf7900000` (`8 KiB`) |
| **Ramoops Crash Log** | `0xac000000` (`2 MiB`) |

---

## 5. The Adreno 540 GPU Challenge

Early on, Mesa recognized the GPU (`EGL_VENDOR=Mesa Project`, `GL_RENDERER=FD540`), and EGL contexts could compile shaders and submit draw buffers—yet **every rendering workload stalled**, causing `glFinish()` to hang indefinitely alongside IOMMU page faults, clock gating warnings, and DRM master conflicts.

### Verified GPU Firmware Blobs
All required Adreno 540 firmware files were deployed and verified:
- `qcom/a530_pm4.fw`
- `qcom/a530_pfp.fw`
- `qcom/a540_gpmu.fw2`
- `qcom/msm8998/xiaomi/a540_zap.mbn`

---

## 6. Major Kernel & GPU Patches Applied

Eight targeted kernel patches resolved the Adreno 540 execution hangs, clock gating issues, and SMMU faults:

### Patch 1 — A5xx Runtime Power-Management Reset
- **Files:** `drivers/gpu/drm/msm/msm_gpu.c`, `a3xx_gpu.c`, `a4xx_gpu.c`, `a5xx_gpu.c`
- **Fix:** Moved `msm_devfreq_resume(gpu)` into GPU-specific resume callbacks and issued an A5xx software reset sequence prior to resuming `devfreq` so stale command-processor state is cleared across `GX` power transitions:
  ```c
  gpu_write(gpu, REG_A5XX_RBBM_SW_RESET_CMD, 1);
  gpu_read(gpu, REG_A5XX_RBBM_SW_RESET_CMD);
  gpu_write(gpu, REG_A5XX_RBBM_SW_RESET_CMD, 0);

  msm_devfreq_resume(gpu);
  ```

### Patches 2–5 — Upstream A5xx Preemption & Ring-Empty Backport Series
- **Files:** `drivers/gpu/drm/msm/adreno/a5xx_gpu.c`, `a5xx_gpu.h`, `a5xx_preempt.c`
- **Fixes:**
  1. **Disable local preemption by default** (`CP_PREEMPT_ENABLE_LOCAL` set to `0x00` instead of `0x01`).
  2. **Clear stale preemption records** (`preempt[i]->data = 0; preempt[i]->info = 0;`) on hardware init.
  3. **Serialize preemption start** using a dedicated `spinlock_t preempt_start_lock` and trigger post-context-switch checks.
  4. **Fix ring-empty detection** ([upstream `b4fb748f0b73`](https://github.com/torvalds/linux/commit/b4fb748f0b734ce1d2e7834998cc599fcbd25d67)) by reading the live hardware pointer via `gpu->funcs->get_rptr(gpu, ring)` and tracking submitted sequence numbers (`last_seqno[]`).
- **Outcome:** First successful hardware-rendered triangle (`PIXEL_RGBA=[64, 128, 191, 255]`, `RESULT=PASS`), though `iova=00000000` IOMMU faults still occurred.

### Patch 6 — MSM8998 GCC Always-On Graphics Clocks
- **File:** `drivers/clk/qcom/gcc-msm8998.c`
- **References:** Upstream [`f9915d2288cf`](https://github.com/torvalds/linux/commit/f9915d2288cf906340491247b88e56ab9892c97d) & [`0522f2e9c8c4`](https://github.com/torvalds/linux/commit/0522f2e9c8c46e1f826886ee95858384fb54aaca)
- **Fix:** Enabled bit `0` during probe for critical graphics/MMSS clocks (`0x7100c` GPU BIMC, `0x71018` GPU SNOC DVM, `0x9030` MMSS QM AHB, `0x900c` MMSS QM core) to prevent premature unused-clock gating.

### Patch 7 — GPUCC Droop-Detector Register Correction
- **File:** `drivers/clk/qcom/gpucc-msm8998.c`
- **Fix:** Corrected the target register offset for `GPUCC_GPU_DD_WRAP_CTRL`:
  ```c
  -regmap_write_bits(regmap, gfx3d_clk.clkr.enable_reg, BIT(0), BIT(0));
  +regmap_write_bits(regmap, 0x430, BIT(0), BIT(0));
  ```

### Patch 8 — RPTR Shadow Writeback Address Fix (Eliminated All IOMMU Faults)
- **File:** `drivers/gpu/drm/msm/adreno/a5xx_preempt.c`
- **Reference:** Upstream [`32e7083429d46`](https://github.com/torvalds/linux/commit/32e7083429d46f29080626fe387ff90c086b1fbe) (*drm/msm/a5xx: fix context faults during ring switch*)
- **Root Cause:** `rptr_addr` was calculated before shadow memory was initialized, causing the GPU to write ring read-pointers to virtual address `0x00000000` (`gpu fault: iova=00000000, flags=1 (0,0,0,1)`).
- **Fix:** Moved `a5xx_gpu->preempt[i]->rptr_addr = shadowptr(a5xx_gpu, gpu->rb[i]);` into hardware initialization after shadow memory setup.
- **Outcome:** **100% elimination of `iova=00000000` faults and SMMU interrupts** across all 18 offscreen rendering tests.

---

## 7. GPU Progression Summary

```text
[1] Stock MSM8998 Driver      ──► GPU detected (FD540), rendering stalled
[2] + PM Reset Patch          ──► Commands submitted, glFinish() timed out
[3] + 4 Preemption Patches    ──► First triangles rendered, IOVA=0 IOMMU faults remained
[4] + GCC & GPUCC Clock Fixes ──► Clock warnings resolved, IOVA=0 faults remained
[5] + RPTR Shadow Address Fix ──► All 18 offscreen tests passed with 0 IOMMU faults
[6] + KMScon & kmscube Fixes  ──► Live animated 3D cube rendered on physical display!
```

---

## 8. Automated GPU Validation Suite

We developed an automated post-boot diagnostic framework (`display.py`, `suite.py`, `evidence.py`) that runs on local console login and captures kernel journals, interrupt counts, clock states, command streams, and pixel readbacks.

### Validation Metrics (RPTR-Fixed Kernel)

| Metric | Result |
| :--- | :--- |
| **Offscreen Graphics Stages Passed** | **18 / 18** |
| **Native DMA Fences Signaled** | **66 / 66** |
| **Framebuffer Pixel Comparisons** | **131 / 131** |
| **GPU Hardware Interrupts Observed** | **84** |
| **Runtime PM Suspend/Resume Cycles** | **3 / 3** |
| **SMMU / IOMMU Faults** | **0** |

<details>
<summary><strong>Click to expand all 19 verified graphics test cases</strong></summary>

| Test Case | Status |
| :--- | :---: |
| EGL / GBM Initialization | ✅ PASS |
| Hardware Renderer Verification (`FD540`, non-llvmpipe) | ✅ PASS |
| Framebuffer Clearing | ✅ PASS |
| Single Triangle Rendering | ✅ PASS |
| Command-Stream Capture Workload | ✅ PASS |
| Repeated Triangle Rendering | ✅ PASS |
| Small Framebuffer Rendering | ✅ PASS |
| Non-Power-of-Two Framebuffer | ✅ PASS |
| Texture Upload & Sampling | ✅ PASS |
| Depth-Buffer Testing | ✅ PASS |
| Indexed Geometry | ✅ PASS |
| Vertex-Buffer Reuse | ✅ PASS |
| Repeated Context Creation | ✅ PASS |
| Multiple Concurrent EGL Contexts | ✅ PASS |
| Simultaneous Independent GPU Processes | ✅ PASS |
| Runtime PM Resume #1 | ✅ PASS |
| Runtime PM Resume #2 | ✅ PASS |
| Runtime PM Resume #3 | ✅ PASS |
| Bounded Load (`512 × 512`, 6 submissions × 32 indexed draw calls) | ✅ PASS |

</details>

---

## 9. Solving the Final On-Screen `kmscube` Blocker

Even after passing every offscreen GBM test, launching `kmscube` on the physical display initially failed with:

```text
no connected connector!
failed to initialize legacy DRM
```

Two userspace issues were identified and fixed:
1. **DRM Master Handoff:** `kmscon` holds DRM master for the terminal. Using `/usr/bin/kmscon-launch-gui` cleanly releases DRM master to `kmscube` and restores the terminal afterward.
2. **`kmscube` Connector Indexing Bug:** Our test script passed `-n 32` (the DRM connector object ID for `DSI-1`). Disassembling Fedora's ARM64 `kmscube` binary revealed that `-n` actually indexes into the `drmModeRes` connector array (`[0]`), where index `32` is out of bounds. Omitting `-n` allowed `kmscube` to auto-select `DSI-1`:
   ```bash
   kmscube -D /dev/dri/card0 -N -c 180
   ```

**Result:** The animated 3D cube rendered smoothly at native resolution on the Xiaomi Mi 6's physical screen.

---

## 10. GPU Performance & Power Management

Conservative GPU Operating Performance Points (OPPs) remain active while regulator definitions (`vdd` / `vddcx`) are finalized:

- **Enabled OPPs:** `257 MHz`, `342 MHz`, `414 MHz`
- **Disabled OPPs:** `515 MHz`, `596 MHz`, `670 MHz`, `710 MHz`

| Telemetry Metric | Measured Value |
| :--- | :--- |
| **Runtime PM Policy** | `auto` (`250 ms` autosuspend delay) |
| **Confirmed Runtime Resumes** | `3` |
| **Peak Sampled SoC Temperature** | `50.9 °C` |
| **GPU Hangcheck / Recovery Events** | `0` |
| **GPU SMMU Faults** | `0` |

---

## 11. Kernel Auditing & Initramfs Overlay Engineering

- **Full Tree Audit:** Audited **77,999 source files** against the upstream base archive, verifying binary disassembly, SHA-256 hashes, GNU build IDs, and **602 imported symbol CRCs** for `msm.ko`.
- **Lightweight Initramfs Overlay:** Rather than duplicating an `~84 MB` initramfs per test build, we engineered a **`236 KB` initramfs overlay** (`initramfs-6.0.0-sagit-fedora-rptrfix-overlay.img`) that overrides only `msm.ko` at boot while reusing the base initramfs and firmware.
- **EFI Partition Cleanup:** Archived 13 obsolete boot artifacts, reclaiming **`~248 MB`** on the EFI System Partition while preserving known-good fallback entries.

---

## 12. Known Limitations & Remaining Work

> **Important:** Do not modify the working GPU kernel configuration while bringing up secondary peripherals. Address remaining subsystems independently.

- **GPU Regulators & High OPPs:** `vdd` and `vddcx` still use dummy regulators; frequencies above `414 MHz` remain disabled. Early boot logs also show `gpu_gx status stuck at 'off'`, which does not impact runtime rendering.
- **Sustained Thermal / Load Testing:** Short and bounded workloads pass; long-duration 3D stress testing is pending.
- **Modem & Cellular:** `DOG_HB` watchdog crashes observed previously; calls, SMS, and mobile data are untested.
- **Bluetooth & Wi-Fi Shutdown:** Bluetooth firmware loading errors remain; an `ath10k` firmware crash occurred once during system shutdown.
- **Battery & Charging:** Fuel-gauge driver probe errors; charging control is unverified.
- **Audio, Cameras, Sensors & Deep Sleep:** Speakers/mic, cameras, fingerprint reader, IMU sensors, and full system suspend remain to be brought up.

---

## 13. Golden Working Kernel Configuration

Preserve these exact verified artifacts as the golden baseline:

| Artifact | Filename / Identifier |
| :--- | :--- |
| **GRUB Entry** | `Fedora Linux - Xiaomi Mi 6 - Clock fixes + A5xx RPTR fix (full GPU diagnostics)` |
| **Kernel Image** | `Image-6.0.0-sagit-clockfix` |
| **Kernel GNU Build ID** | `04837bae29fb2275c5c784218232dd6b91a0b84d` |
| **Device Tree Blob** | `msm8998-xiaomi-sagit.dtb` |
| **Base Initramfs** | `initramfs-6.0.0-sagit-fedora-preempt-test.img` |
| **RPTR Fix Initramfs Overlay** | `initramfs-6.0.0-sagit-fedora-rptrfix-overlay.img` (`236 KB`) |
| **Patched `msm.ko` Build ID** | `035f367330f30eeaea76aed7b5df653e9fcea6c1` |
| **Patched `msm.ko` SHA-256** | `1afa5235df55ce7ee01863d50ee26667e801a101a6c24de34c6741f69971fe9d` |

---

## 14. Development Roadmap

- [x] **Stage 1 — Native Linux Boot:** EDK2 UEFI, rEFInd, GRUB, UFS storage, and Fedora 45 ARM64 userspace.
- [x] **Stage 2 — Essential Hardware:** Native `1080 × 1920` MIPI-DSI display, touchscreen, USB host, and dual-band Wi-Fi.
- [x] **Stage 3 — GPU Hardware Acceleration:** Adreno 540 (`FD540`) command submission, SMMU/RPTR fix, and on-screen 3D rendering *(Achieved Day 6)*.
- [ ] **Stage 4 — Hardware-Accelerated Wayland:** Bring up Weston / minimal Wayland compositor with touchscreen input and zero-copy scanout.
- [ ] **Stage 5 — Full Mobile Desktop Environment:** KDE Plasma Mobile / desktop with hardware-accelerated apps and proper HiDPI scaling.
- [ ] **Stage 6 — Remaining Hardware Subsystems:** Audio (ALSA/UCM), Bluetooth, battery fuel gauge, charging, sensors, modem, and cameras.
- [ ] **Stage 7 — Power & Thermal Optimization:** Full device suspend/deep sleep, real GPU regulators, and higher OPP validation.
- [ ] **Stage 8 — Reproducible Release:** Clean patch series, automated kernel packaging, and flashable Fedora images for `sagit`.

---

## 15. Project Summary

Over six days of intensive kernel and userspace engineering, the **Xiaomi Mi 6 (`sagit`)** evolved from a device stuck at black screens into a standalone, bare-metal **Fedora Linux 45 ARM64** machine with working Wi-Fi, touchscreen, and **native hardware-accelerated 3D graphics on the Adreno 540**.

With the core GPU pipeline verified end-to-end on the physical display, the project now shifts focus to **Wayland compositing**, **KDE Plasma**, and **remaining smartphone peripherals**.