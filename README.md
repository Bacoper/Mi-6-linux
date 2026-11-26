# Xiaomi Mi 6 (`sagit`) — Native Fedora Linux Project

> **Day 7 Milestone Report — November 26, 2026**
>
> **Bare-Metal Fedora Linux 45 ARM64 + KDE Plasma 6.8.1 Wayland on the Xiaomi Mi 6 (`sagit`)**

![Xiaomi Mi 6 running kmscube on native Fedora Linux](docs/images/kmscube-sagit.jpg)

Seven days ago, we were struggling to get anything displayed on the phone—facing black screens, unexpected reboots, missing drivers, unresolved clock dependencies, and an Adreno 540 GPU that couldn't reliably execute rendering commands.

Today, we have **native Fedora Linux 45 ARM64**, a full **KDE Plasma 6.8.1 Wayland desktop**, **hardware-accelerated Adreno 540 graphics**, **stabilized Wi-Fi**, and **live battery monitoring through our own custom kernel module**—overcoming major limitations inherited from the original MSM8998 Linux port.

This is a **bare-metal Linux installation** running directly on the hardware via EDK2 UEFI and GRUB—not an Android container, chroot, or emulation layer.

---

## Table of Contents

- [Device & Stack Overview](#device--stack-overview)
- [1. Complete Current Hardware Status](#1-complete-current-hardware-status)
- [2. The Complete Development Journey](#2-the-complete-development-journey)
- [3. Kernel, Bootloader & Storage Bring-Up](#3-kernel-bootloader--storage-bring-up)
- [4. Device-Tree Corrections](#4-device-tree-corrections)
- [5. Adreno 540 GPU Bring-Up](#5-adreno-540-gpu-bring-up)
- [6. Qualcomm Remote Processors & Firmware](#6-qualcomm-remote-processors--firmware)
- [7. Wi-Fi — From Nonexistent to Stable Connectivity](#7-wi-fi--from-nonexistent-to-stable-connectivity)
- [8. Bluetooth Bring-Up](#8-bluetooth-bring-up)
- [9. KDE Plasma 6.8.1 Wayland Desktop](#9-kde-plasma-681-wayland-desktop)
- [10. Custom PMI8998 Battery Monitoring Module](#10-custom-pmi8998-battery-monitoring-module)
- [11. Charging & USB Role-Switching Investigation](#11-charging--usb-role-switching-investigation)
- [12. Audio Subsystem Status](#12-audio-subsystem-status)
- [13. Automated Diagnostic Infrastructure](#13-automated-diagnostic-infrastructure)
- [14. Known-Working Golden Baseline](#14-known-working-golden-baseline)
- [15. Project Documentation](#15-project-documentation)
- [16. Remaining Problems & Future Roadmap](#16-remaining-problems--future-roadmap)
- [17. Project Progress Summary](#17-project-progress-summary)
- [18. Day 7 Achievements Recap](#18-day-7-achievements-recap)

---

## Device & Stack Overview

| Specification / Layer | Details |
| :--- | :--- |
| **Device** | Xiaomi Mi 6 (`sagit`, 2017) |
| **SoC & Memory** | Qualcomm Snapdragon 835 (`MSM8998`) · Adreno 540 · `6 GB` RAM |
| **Display** | 5.15-inch JDI `R63452` (`1080 × 1920` @ `60 Hz`) |
| **Operating System** | Fedora Linux 45 ARM64 (Bare-Metal) |
| **Desktop Environment** | KDE Plasma `6.8.1` (KWin Wayland) |
| **Kernel** | Custom Linux `6.0.0-sagit-fedora` (`Image-6.0.0-sagit-clockfix`) |
| **Graphics Stack** | Linux `drm/msm` `1.9.0` + Mesa Freedreno `26.2.3` (`FD540`) |
| **Project Duration** | 7 Days (November 20 – November 26, 2026) |
| **Latest Milestone** | Live battery telemetry via custom `sagit_pmi8998_monitor.ko` |

---

## 1. Complete Current Hardware Status

> **Note:** 🟢 indicates working within the scope of current testing, not necessarily production-ready. Several subsystems still require extended qualification.

| Subsystem / Component | Status | Notes |
| :--- | :---: | :--- |
| **Native ARM64 Linux Boot** | 🟢 Working | Bare-metal boot on internal UFS |
| **EDK2 UEFI / rEFInd / GRUB** | 🟢 Working | Chainloaded EFI boot with explicit DTB loading |
| **Internal UFS & `ext4` Rootfs** | 🟢 Working | Mounted reliably via partition UUID |
| **All 8 CPU Cores** | 🟢 Detected | Kryo 280 octa-core online |
| **CPU Frequency Scaling** | 🟡 Partial | Needs full OPP/governor qualification |
| **LCD Display & Backlight** | 🟢 Working | Native MIPI DSI0 (`DSI-1`) `1080 × 1920 @ 60 Hz` + WLED |
| **Touchscreen** | 🟢 Working | Native multi-touch operational in KDE Plasma |
| **USB Host & Keyboard** | 🟢 Working | External HID peripherals verified |
| **USB Peripheral Mode / Role Switch** | 🟡 Investigating | Suspected stuck in host role; affects charging |
| **Adreno 540 GPU (`FD540`)** | 🟢 Working | Hardware OpenGL ES `3.1` & EGL `1.5` verified |
| **GPU Runtime PM & Preemption** | 🟢 Working | Autosuspend/resume and preemption patches validated |
| **GPU IOMMU Mappings** | 🟢 Working | `iova=00000000` write fault resolved via RPTR patch |
| **Physical 3D Rendering** | 🟢 Working | Verified on-screen (`kmscube` & KWin Wayland) |
| **GPU Maximum Frequency** | 🟡 Limited | Capped at `414 MHz` pending real `vdd`/`vddcx` regulators |
| **Hardware Vulkan (`Turnip`)** | 🔴 Unsupported | OpenGL ES working; Turnip A540 not established |
| **KDE Plasma `6.8.1` Wayland** | 🟢 Working | Boots directly to graphical session (`sagit.desktop=plasma`) |
| **Desktop Animations & Window Drag** | 🟡 Partial | Animations improved; window drag choppiness/artifacts remain |
| **Wi-Fi (`2.4 GHz` & `5 GHz`)** | 🟢 Working | `WCN3990` (`ath10k_snoc`) scanning, DHCP, internet verified |
| **Wi-Fi Firmware Stability** | 🟢 Working | Zero crashes observed since power-saving fix |
| **Bluetooth Controller & Firmware** | 🟢 Detected | Uncompressed `crbtfw21.tlv` / `crnv21.bin` installed; pairing untested |
| **Battery Detection & Percentage** | 🟢 Verified | Live SOC exposed via `/sys/class/power_supply/sagit-battery` |
| **Battery Voltage, Current & Temp** | 🟢 Verified | Register decoding validated against PMI8998 v2.1 hardware |
| **Battery Charging under Linux** | 🔴 Not Working | Under active investigation (DWC3 role switch / SMB2) |
| **ADSP & SLPI Remote Processors** | 🟢 Starts | Remoteprocs boot cleanly |
| **Modem Remote Processor (MPSS)** | 🟡 Partial | Starts; previous `DOG_HB` watchdog instability observed |
| **Qualcomm Userspace Services** | 🟢 Running | `rmtfs`, `pd-mapper`, `diag-router`, `tqftpserv` active |
| **Internal Speaker** | 🔴 Not Working | ADSP/APR up, ASoC/codec/amplifier routing pending |
| **Microphone, Calls, SMS & Mobile Data** | ⚪ Untested | Pending audio and modem qualification |
| **GPS, Cameras, Fingerprint & IMU** | ⚪ Untested | Sensor and camera bring-up planned in Phase 2 |
| **Full System Suspend / Deep Sleep** | ⚪ Untested | GPU runtime suspend works; full SoC sleep untested |

---

## 2. The Complete Development Journey

```
Phase 1 ──► Almost Nothing Worked      Black screens, 15s reboots, ignored DTBs, no usable TTY
Phase 2 ──► Fedora ARM64 TTY           Explicit GRUB DTB handoff + clean Linux 6.0.0 baseline
Phase 3 ──► GPU Comes Alive            drm/msm + Adreno firmware + FD540 runtime PM wake-up
Phase 4 ──► The Invisible Triangle     First offscreen triangle rendered, but IOVA=0 IOMMU faults remained
Phase 5 ──► Day 6: The Spinning Cube   RPTR shadow address fix + kmscube on-screen 3D rendering
Phase 6 ──► Day 7: Full KDE Plasma     Plasma 6.8.1 Wayland booted on first try + Wi-Fi stabilized
Phase 7 ──► Day 7: Battery Telemetry   Custom sagit_pmi8998_monitor.ko reporting live SOC in KDE
Phase 8 ──► Current Objective          USB-C DWC3 role switching & safe battery charging
```

---

## 3. Kernel, Bootloader & Storage Bring-Up

### Resolving Early Boot & Device-Tree Failures
Initial experiments across Linux `5.15.0-msm8998-tiny` and `6.18.64-msm8998-tiny` suffered from black screens, automatic reboots ~15 seconds into boot, deferred driver probes, and ignored device trees when booting directly via EFI stub (`EFI stub: Ignoring DTB from command line`).

We standardized on the MSM8998 mainline downstream source tree (`90169e6e9fee29fae14aeac0a905246be5ce6304`), built as `6.0.0-sagit-fedora`, and moved DTB loading into GRUB:

| Kernel Build | Role in Project |
| :--- | :--- |
| `5.15.0-msm8998-tiny` | Early EFI boot experiments |
| `6.18.64-msm8998-tiny` | Display and GPU bring-up experiments |
| `6.0.0-sagit-fedora` | Clean Fedora ARM64 baseline |
| `Image-6.0.0-sagit-clockfix` | Current working clock-corrected kernel image |

### Working Boot Chain

```
Xiaomi Mi 6 Hardware
  └─► EDK2 UEFI Firmware
        └─► rEFInd EFI Boot Manager
              └─► GRUB ARM64 (explicitly loads kernel, DTB, and initramfs overlays)
                    └─► Linux 6.0.0-sagit-clockfix + RPTR GPU Module
                          └─► Fedora 45 ARM64 Rootfs (systemd → Plasma Login Manager)
                                └─► KDE Plasma 6.8.1 Wayland
```

### Internal UFS Storage Layout
To avoid duplicate mount corruption and read-only `ext4` fallbacks across reboots, all host and target mount workflows strictly use partition UUIDs:

| Partition | Filesystem | UUID |
| :--- | :--- | :--- |
| **EFI System Partition (ESP)** | `FAT32` | `6FB1-B704` |
| **Fedora Root Filesystem** | `ext4` | `feb4108f-2ed1-4c46-afdf-32aca365912e` |

---

## 4. Device-Tree Corrections

We audited and corrected the live device tree (`msm8998-xiaomi-sagit.dtb`) across display endpoints, GPU clocks, SMMU mappings, and remoteproc memory reservations:

| Hardware Block / Region | Address / Configuration |
| :--- | :--- |
| **Adreno 540 GPU** | `0x05000000` |
| **GPU SMMU** | `0x05040000` |
| **MSM Display Controller (MDP/DPU)** | `0x0c901000` |
| **MIPI DSI0 Controller** | `0x0c994000` (exposed in DRM as `DSI-1`) |
| **LCD Panel** | JDI `R63452` (`1080 × 1920 @ 60 Hz`) |
| **ADSP Reserved Memory** | `0x8b200000` (expanded to `30 MiB`) |
| **Modem Reserved Memory** | `0x8d000000` region |
| **GPU ZAP Reserved Memory** | `0xf7900000` (`8 KiB`) |
| **Ramoops Crash Storage** | `0xac000000` (`2 MiB`) |

> **Key Finding:** The physical LCD panel is wired to hardware **DSI0**, even though Linux DRM labels the connector `DSI-1`. This is purely a naming difference rather than a hardware mismatch.

---

## 5. Adreno 540 GPU Bring-Up

### Stage A & B — DRM Initialization & Firmware
Once clock and power-domain dependencies were satisfied, `drm/msm` `1.9.0` initialized `/dev/dri/card0` and `/dev/dri/renderD128` with Mesa reporting `FD540` using:
- `qcom/a530_pm4.fw` (Command processor)
- `qcom/a530_pfp.fw` (Prefetch parser)
- `qcom/a540_gpmu.fw2` (GPU power management)
- `qcom/msm8998/xiaomi/a540_zap.mbn` (Protected GPU initialization)

### Stage C–F — Eight Kernel Patches That Made Adreno 540 Reliable
1. **A5xx Runtime PM Reset (`msm_gpu.c`, `a3xx/a4xx/a5xx_gpu.c`):** Added an `REG_A5XX_RBBM_SW_RESET_CMD` reset sequence on runtime resume before calling `msm_devfreq_resume(gpu)` so stale command-processor state is cleared across `GX` power transitions.
2. **Disable Local Preemption (`a5xx_gpu.c`):** Changed `CP_PREEMPT_ENABLE_LOCAL` from `0x01` to `0x00` during normal submissions.
3. **Clear Stale Preemption Records (`a5xx_preempt.c`):** Zeroed `preempt[i]->data` and `preempt[i]->info` during hardware init.
4. **Serialize Preemption Start (`a5xx_gpu.h`, `a5xx_preempt.c`):** Added `spinlock_t preempt_start_lock` and post-context-switch triggers to eliminate ring-switch race conditions.
5. **Fix Ring-Empty Detection (`a5xx_gpu.c`, `a5xx_gpu.h`, `a5xx_preempt.c`):** Backported [upstream `b4fb748f0b73`](https://github.com/torvalds/linux/commit/b4fb748f0b734ce1d2e7834998cc599fcbd25d67) to read the live hardware pointer (`gpu->funcs->get_rptr(gpu, ring)`) and track `last_seqno[]`. This produced our first offscreen triangle (`RGBA: [64, 128, 191, 255]`), though `iova=00000000` faults remained.
6. **MSM8998 GCC Always-On Graphics Clocks (`gcc-msm8998.c`):** Enabled bit `0` during probe for GPU BIMC (`0x7100c`), GPU SNOC DVM (`0x71018`), MMSS QM AHB (`0x9030`), and MMSS QM core (`0x900c`) ([`f9915d2288cf`](https://github.com/torvalds/linux/commit/f9915d2288cf906340491247b88e56ab9892c97d), [`0522f2e9c8c4`](https://github.com/torvalds/linux/commit/0522f2e9c8c46e1f826886ee95858384fb54aaca)).
7. **GPUCC Droop-Detector Register Fix (`gpucc-msm8998.c`):** Fixed `GPUCC_GPU_DD_WRAP_CTRL` write offset to `0x430`.
8. **RPTR Shadow Writeback Address Fix (`a5xx_preempt.c`):** Backported [upstream `32e7083429d46`](https://github.com/torvalds/linux/commit/32e7083429d46f29080626fe387ff90c086b1fbe) (*drm/msm/a5xx: fix context faults during ring switch*), moving `preempt[i]->rptr_addr = shadowptr(a5xx_gpu, gpu->rb[i])` into hardware init after shadow memory mapping—**eliminating 100% of `gpu fault: iova=00000000` IOMMU write faults**.

### Stage G & H — Automated GPU Validation & On-Screen `kmscube`
- **18 / 18** offscreen test stages passed (textures, depth buffer, indexed geometry, multi-context, concurrent processes, bounded `512 × 512` load).
- **131 / 131** framebuffer pixel checks passed and **66 / 66** native DMA fences signaled with **0 SMMU/IOMMU faults**.
- Fixed `kmscube` connector indexing (Fedora's `kmscube -n` treats the argument as an array index rather than DRM connector ID `32`) and `kmscon-launch-gui` DRM master handoff, achieving **180 successful page flips at `30.14 fps`** on the physical panel.
- Conservative GPU OPPs (`257 MHz`, `342 MHz`, `414 MHz`) remain active while higher frequencies (`515–710 MHz`) stay disabled pending real `vdd` / `vddcx` regulator definitions.

---

## 6. Qualcomm Remote Processors & Firmware

| Subsystem / Service | Role | Status |
| :--- | :--- | :--- |
| **SLPI & ADSP** | Low-power sensor & audio DSP remoteprocs | 🟢 Starts cleanly |
| **MPSS** | Cellular modem remoteproc | 🟡 Starts (previous `DOG_HB` watchdog resets noted) |
| **QRTR / GLINK / SMEM / RPM** | Interprocessor IPC, shared memory & power management | 🟢 Operational |
| **`rmtfs` & `pd-mapper`** | Remote filesystem & protection-domain mapping | 🟢 Running |
| **`diag-router` & `tqftpserv`** | Diagnostic routing & QRTR firmware file transfer | 🟢 Running / Enabled |

---

## 7. Wi-Fi — From Nonexistent to Stable Connectivity

The `WCN3990` wireless subsystem (`ath10k_snoc`) progressed from showing only `lo` to full **2.4 GHz and 5 GHz** scanning, WPA authentication, DHCP, and internet connectivity.

During the first KDE Plasma boot, the Wi-Fi firmware experienced seven service exits during normal operation before recovering. Investigation ruled out missing board calibration data and pointed toward wireless power-management behavior. Applying a **reversible, per-interface NetworkManager power-saving configuration** (while preserving firmware and saved credentials) completely stopped the crashes during testing.

---

## 8. Bluetooth Bring-Up

The Qualcomm UART Bluetooth controller attempted to load `qca/crbtfw21.tlv`, which existed on Fedora only in compressed form unsupported by our `6.0.0` kernel. After matching the `WCN3990` controller product and ROM version against the firmware patch header, we deployed the uncompressed firmware files:
- `/usr/lib/firmware/qca/crbtfw21.tlv`
- `/usr/lib/firmware/qca/crnv21.bin`

Physical Bluetooth device discovery, pairing, and audio streaming remain to be verified.

---

## 9. KDE Plasma 6.8.1 Wayland Desktop

On Day 7, we switched the boot target from `multi-user.target` to `graphical.target` with `sagit.desktop=plasma` and retired the auto-launching `kmscube` hook. **KDE Plasma `6.8.1` (KWin Wayland) booted on the very first attempt without requiring a new kernel build:**

| Boot Milestone | Elapsed Time |
| :--- | :---: |
| MSM DRM initialized | `7s` |
| `graphical.target` reached | `18s` |
| Plasma Wayland login screen running | `23s` |
| User desktop session launched | `37s` |
| KWin & Plasma startup | `39s` |
| Full graphical desktop ready | `43s` |

### Desktop Optimizations Applied
- Disabled heavy blur and translucency effects and shortened animation durations.
- Disabled background Baloo file indexing and reduced diagnostic polling overhead.
- Fixed root-owned directory permissions inside `/home/sagit` (resolving KDE Wallet, WirePlumber, and config persistence issues).
- Configured landscape display orientation.

### Current Desktop Observations
- **Smooth:** Spinning 3D benchmarks and general desktop navigation.
- **Improved:** Window/UI animations.
- **Needs Investigation:** Choppy window dragging, occasional brief visual artifacts, and a recurring Mesa warning (`Failed to set BO metadata with DRM_MSM_GEM_INFO: -22`) likely tied to Mesa `26.2.3` running against kernel `6.0.0`.

---

## 10. Custom PMI8998 Battery Monitoring Module

### Overcoming a Longstanding Upstream Port Limitation
Across 25 historical boots, the legacy `qcom-fg` driver consistently failed to probe on `sagit`:

```text
qcom-fg 800f000.spmi:pmic@2:fuel-gauge@4000: Couldn't read CHG base address from dt
qcom-fg: probe failed with error -22
```

Because the legacy driver expected SMB2 charger registers (`0x1000`) and `usb-plugin` interrupts inside the fuel-gauge node—and patching the DT blindly would have triggered unsafe register writes—we developed a dedicated, **read-only** kernel module: **`sagit_pmi8998_monitor.ko`**.

### Architecture & First Physical Validation

```
Xiaomi Mi 6 Battery
  └─► Qualcomm PMI8998 v2.1 Fuel-Gauge Registers (Read-Only Poll every 20s)
        └─► sagit_pmi8998_monitor.ko (validates PMIC subtype, rev, SOC readiness)
              └─► /sys/class/power_supply/sagit-battery
                    └─► UPower ──► KDE PowerDevil (15% · Discharging)
```

Registered at `14.96s` after boot with zero probe errors, matching raw PMI8998 SOC register readbacks (`0x2c → 17%`, `0x2a → 16%`, `0x28 → 16%`, `0x26 → 15%`):

| Telemetry Parameter | First Physical Boot Reading |
| :--- | :--- |
| **PMIC Hardware Revision** | `v2.1` |
| **Last Sampled Voltage** | `3.486 V` |
| **Last Sampled Discharge Current** | `660.6 mA` |
| **Last Sampled Temperature** | `31.3 °C` |
| **Power Trend (`15s → 30s → 60s → 120s`)** | `4.29 W` $\rightarrow$ `3.48 W` $\rightarrow$ `2.86 W` $\rightarrow$ `2.30 W` |

---

## 11. Charging & USB Role-Switching Investigation

While battery discharge telemetry now works in KDE Plasma, the phone does not yet charge under Linux. Our working hypothesis is that the DWC3 USB controller remains locked in **host mode** (supplying VBUS rather than sinking current). With live battery current monitoring now available, we are passively investigating:
- USB-C CC source/sink detection and DWC3 dual-role switching (`host` vs. `peripheral`).
- VBUS regulator control and PMI8998 SMB2 charger integration—strictly preserving hardware charging protections.

---

## 12. Audio Subsystem Status

KDE Plasma currently reports no audio output devices. While ADSP firmware boots, Qualcomm APR communication is active, and PipeWire + WirePlumber run cleanly in userspace, ASoC sound card registration, SLIMbus routing, codec configuration, and speaker amplifier initialization remain to be brought up.

---

## 13. Automated Diagnostic Infrastructure

We built a suite of automated post-boot collectors (`sagit-gpu-diag`, `sagit-gpu-diag-v2`, `sagit-rptr-diag`, `sagit-gpu-full`, `sagit-kde-diag`, and `sagit-battery-diag`) that gather kernel logs, live FDTs, module build IDs, GPU faults, interrupt counters, and PMIC register dumps without manual console typing:

- **First KDE Archive:** `28` boots · `1,036` archived files.
- **First Battery Archive:** `120` files · `83` verified journal files · **36 / 36** target checksum checks passed.

---

## 14. Known-Working Golden Baseline

All core graphics and battery artifacts are preserved under dedicated GRUB entries:

| Artifact | Filename / GNU Build ID |
| :--- | :--- |
| **Kernel Image** | `Image-6.0.0-sagit-clockfix` (`04837bae29fb2275c5c784218232dd6b91a0b84d`) |
| **Baseline Device Tree** | `msm8998-xiaomi-sagit.dtb` |
| **Base Initramfs** | `initramfs-6.0.0-sagit-fedora-preempt-test.img` |
| **RPTR MSM Module Overlay** | `initramfs-6.0.0-sagit-fedora-rptrfix-overlay.img` (`035f367330f30eeaea76aed7b5df653e9fcea6c1`) |
| **Battery Test Device Tree** | `msm8998-xiaomi-sagit-battery-test.dtb` |
| **Battery Monitor Module** | `sagit_pmi8998_monitor.ko` (`a4b6e188737c7faf029ce235743fea5b6de67a6a`) |

---

## 15. Project Documentation

| Document | Scope |
| :--- | :--- |
| `AGENTS.md` | Project constraints and safe-testing rules |
| `PROJECT_STATUS.md` | Current subsystem status and historical milestones |
| `SESSION_REPORT.md` | Chronological engineering logs |
| `TEST_PLAN.md` | Reproducible hardware validation procedures |
| `GPU_ANALYSIS.md` | Adreno 540 fault analysis, patches, and `kmscube` metrics |
| `GOLDEN_BASELINE.md` | Preserved working kernel, DTB, and initramfs identities |
| `KDE_BRINGUP.md` | KDE Plasma 6.8.1 Wayland configuration and optimizations |
| `KDE_WIFI_BT_REPORT.md` | Desktop tuning, `ath10k` stability fix, and Bluetooth firmware |
| `BATTERY_BRINGUP.md` | PMI8998 fuel-gauge root cause, custom module, and telemetry validation |
| `CHARGING_BRINGUP.md` | Ongoing USB-C role-switching and SMB2 charging investigation |

---

## 16. Remaining Problems & Future Roadmap

### Phase 1 — Core Daily Usability
- [ ] **Enable safe battery charging** *(Critical — Hard)*
- [ ] **Verify USB-C DWC3 dual-role switching** *(High — Moderate–Hard)*
- [ ] **Merge `sagit_pmi8998_monitor.ko` into default KDE boot entry** *(High — Easy)*
- [ ] **Eliminate KWin window-dragging choppiness & visual artifacts** *(High — Moderate–Hard)*
- [ ] **Verify CPU frequency scaling & optimize idle power draw** *(High — Moderate–Hard)*
- [ ] **Verify Bluetooth pairing/audio & qualify long-term Wi-Fi stability** *(Medium — Easy–Moderate)*

### Phase 2 — Full Smartphone Hardware Support
- [ ] Safely validate GPU regulator supplies (`vdd`/`vddcx`) and higher OPPs (`515–710 MHz`)
- [ ] Bring up ASoC audio (internal speaker amplifier, microphone, ALSA UCM)
- [ ] Enable IMU sensors (accelerometer/gyroscope) and automatic screen rotation
- [ ] Validate full system suspend / deep sleep
- [ ] Bring up cellular data, calls, SMS, GPS, cameras, and fingerprint reader

### Phase 3 — Reproducible Distribution & Upstreaming
- [ ] Build a reproducible, flashable Fedora ARM64 image for `sagit`
- [ ] Rebase the clean patch series onto a newer maintained mainline kernel and share fixes with the broader MSM8998 / postmarketOS community

---

## 17. Project Progress Summary

| Milestone | Status |
| :--- | :--- |
| Native ARM64 Linux Boot & Internal UFS Storage | ✅ Achieved |
| Native `1080 × 1920 @ 60 Hz` Display & Touchscreen | ✅ Achieved |
| Hardware-Accelerated Adreno 540 3D Rendering | ✅ Achieved |
| Full KDE Plasma `6.8.1` Wayland Desktop | ✅ Achieved |
| Stabilized Dual-Band Wi-Fi Connectivity | ✅ Achieved |
| Live PMI8998 Battery Telemetry | ✅ Achieved |
| Charging & USB-C Dual-Role Power Management | 🔄 In Progress |
| Audio, Sensors, Modem & Cameras | ⏳ Pending |

---

## 18. Day 7 Achievements Recap

- ✅ Booted full **KDE Plasma `6.8.1` Wayland** on the first attempt.
- ✅ Verified normal graphical startup (`43s` cold boot to desktop) without rebuilding the kernel.
- ✅ Applied desktop compositor optimizations and fixed `/home/sagit` ownership issues.
- ✅ Deployed uncompressed `WCN3990` Bluetooth firmware (`crbtfw21.tlv` / `crnv21.bin`).
- ✅ Eliminated Wi-Fi firmware crashes via NetworkManager power-saving configuration.
- ✅ Expanded automated diagnostic archives (`1,036` KDE files + `120` battery evidence files).
- ✅ Engineered and validated **`sagit_pmi8998_monitor.ko`**, bringing live battery percentage, voltage, current, and temperature into KDE PowerDevil.
