# Xiaomi Mi 6 (`sagit`) — Native Fedora Linux Project

![OS](https://img.shields.io/badge/OS-Fedora%20Linux%2045%20ARM64-51A2DA?style=for-the-badge&logo=fedora&logoColor=white)
![Desktop](https://img.shields.io/badge/Desktop-KDE%20Plasma%206.8.1%20Wayland-1D99F3?style=for-the-badge&logo=kde&logoColor=white)
![Kernel](https://img.shields.io/badge/Kernel-Linux%206.0.0--sagit--fedora-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![GPU](https://img.shields.io/badge/GPU-Adreno%20540%20%28FD540%29-success?style=for-the-badge)
![Audio](https://img.shields.io/badge/Audio-TFA9888%20Speaker%20Working-brightgreen?style=for-the-badge)
![Battery](https://img.shields.io/badge/Battery-PMI8998%20Telemetry%20%2B%20Charging-brightgreen?style=for-the-badge)

> **Day 8 Milestone Report — November 27, 2026 | Physical Audio Breakthrough**
>
> **Bare-Metal Fedora Linux 45 ARM64 on the Xiaomi Mi 6 (`sagit`) — Graphics, Wi-Fi, Custom Battery Telemetry, Charging & Physical Speaker Output**

<p align="center">
  <img src="docs/images/kmscube-sagit.jpg" alt="Hardware-accelerated 3D rendering on the Xiaomi Mi 6 physical LCD display" width="520" />
  <br />
  <em>Hardware-accelerated OpenGL ES rendering running natively on the Xiaomi Mi 6 (Snapdragon 835 / Adreno 540 / JDI R63452 1080×1920 panel).</em>
</p>

Eight days ago, we were struggling to get a Linux kernel to boot reliably on the Xiaomi Mi 6 (`sagit`). Today, we have a **native Fedora 45 ARM64 desktop** with **hardware-accelerated Adreno 540 graphics**, **stabilized dual-band Wi-Fi**, **live battery telemetry**, **working USB charging**, and **audible physical speaker output** through a reconstructed Qualcomm ADSP + WCD9335 + NXP TFA9888 audio pipeline.

This is no longer just a kernel boot experiment—we are building a usable, pocket-sized bare-metal Linux computer running directly via EDK2 UEFI and GRUB (no Android containers, chroots, or emulation).

---

## Table of Contents

- [1. Overall Hardware Status](#1-overall-hardware-status)
- [2. Device Specifications](#2-device-specifications)
- [3. Bootloader & Kernel Bring-Up](#3-bootloader--kernel-bring-up)
- [4. Internal Storage & Development Infrastructure](#4-internal-storage--development-infrastructure)
- [5. Adreno 540 — Hardware Graphics Acceleration](#5-adreno-540--hardware-graphics-acceleration)
- [6. Display & KDE Plasma 6.8.1 Wayland](#6-display--kde-plasma-681-wayland)
- [7. Wi-Fi, Bluetooth & Qualcomm Firmware](#7-wi-fi-bluetooth--qualcomm-firmware)
- [8. Battery Monitoring — Custom PMI8998 Driver](#8-battery-monitoring--custom-pmi8998-driver)
- [9. Charging & USB-C Role Management](#9-charging--usb-c-role-management)
- [10. Physical Audio — Day 8 Breakthrough](#10-physical-audio--day-8-breakthrough)
- [11. Project Documentation & Preservation](#11-project-documentation--preservation)
- [12. Project Timeline (Days 1–8)](#12-project-timeline-days-18)
- [13. Future Development Roadmap](#13-future-development-roadmap)
- [14. Power Consumption & Optimization Priorities](#14-power-consumption--optimization-priorities)
- [15. Project Vision & Day 8 Summary](#15-project-vision--day-8-summary)

---

## 1. Overall Hardware Status

| Subsystem / Feature | Status | Notes |
| :--- | :---: | :--- |
| **UEFI / rEFInd / GRUB** | 🟢 Working | Chainloaded ARM64 EFI boot with explicit DTB loading |
| **Linux Kernel Boot** | 🟢 Working | Custom `6.0.0-sagit-fedora` baseline (`Image-6.0.0-sagit-clockfix`) |
| **Internal UFS 2.1 Storage** | 🟢 Working | Native `ext4` rootfs and `FAT32` ESP mounted by UUID |
| **CPU & RAM (8-core / 6 GB)** | 🟢 Working | Kryo 280 octa-core + full 6 GB LPDDR4X active |
| **Display & Backlight** | 🟢 Working | Native MIPI DSI0 (`DSI-1`) `1920 × 1080 @ 60 Hz` + WLED |
| **Touchscreen** | 🟢 Working | Multi-touch operational in KDE Plasma Wayland |
| **Adreno 540 Acceleration** | 🟢 Working | Mesa Freedreno `26.2.3` (`FD540`) OpenGL ES `3.1` / EGL `1.5` |
| **KDE Plasma Wayland** | 🟢 Working | Full Plasma `6.8.1` graphical desktop session |
| **2.4 GHz & 5 GHz Wi-Fi** | 🟢 Working | Qualcomm `WCN3990` (`ath10k_snoc`) scanning, DHCP, internet |
| **Battery Percentage** | 🟢 Working | Live SOC reporting via custom `sagit_pmi8998_monitor.ko` |
| **Battery Voltage / Current / Temp** | 🟢 Working | Validated against raw PMI8998 v2.1 fuel-gauge registers |
| **USB Charging** | 🟢 Working | Confirmed `~2.5 W` (`5 V × 500 mA`) input & positive net battery current |
| **Digital Audio (`WCD9335`)** | 🟢 Working | ALSA sound card registered, `48 kHz` stereo playback verified |
| **Physical Speaker (`TFA9888`)** | 🟢 Working | **First audible sound achieved on Day 8** (`tfa98xx_aac.cnt`) |
| **Bluetooth** | 🟡 Partial | Uncompressed firmware installed; pairing & audio need validation |
| **Speaker Volume & Profiles** | 🟡 Polishing | Quiet output volume & PipeWire auto-profile selection need tuning |
| **USB OTG / Host Mode** | 🟡 Intermittent | Works in early boots; DWC3 dual-role coordination needs completion |
| **GPU Maximum Frequency** | 🟡 Limited | Capped at `414 MHz` pending real `vdd`/`vddcx` regulator validation |
| **Power Management** | 🟡 Optimizing | Idle drain reduction, CPU idle states & GPU runtime PM tuning pending |
| **Hardware Video Decoding** | ⚪ Untested | Venus video accelerator unverified |
| **Microphone** | ⚪ Untested | Capture routing pending |
| **System Suspend / Resume** | ⚪ Unverified | GPU runtime suspend works; full SoC deep sleep unverified |
| **Cellular Calls & Mobile Data** | 🔴 Unfinished | MPSS remoteproc starts; modem stability & voice/data unfinished |
| **Cameras** | 🔴 Unfinished | Sensor and ISP bring-up not yet started |
| **Vulkan (`Turnip`)** | 🔴 Unavailable | Not supported in current `6.0.0` / `FD540` stack |

---

## 2. Device Specifications

| Component | Specification |
| :--- | :--- |
| **Device** | Xiaomi Mi 6 (`sagit`, released 2017) |
| **SoC** | Qualcomm MSM8998 Snapdragon 835 (`AArch64`) |
| **CPU** | 8-core Kryo 280 (up to `2.45 GHz`) |
| **GPU** | Qualcomm Adreno 540 |
| **RAM** | `6 GB` LPDDR4X |
| **Storage** | Internal UFS 2.1 |
| **Display** | `5.15"` JDI `R63452` (`1920 × 1080`, `60 Hz`) |
| **Wireless** | Qualcomm `WCN3990` (Wi-Fi 802.11ac + Bluetooth) |
| **Audio Codec** | Qualcomm `WCD9335` |
| **Speaker Amplifier** | NXP `TFA9888` (rev `0x2c88`, `AAC` speaker variant) |
| **PMIC / Fuel Gauge** | Qualcomm `PMI8998` (rev `v2.1`) |

Our goal is to unlock as much of this hardware as possible using native Linux drivers rather than relying on Android userspace blobs or containers.

---

## 3. Bootloader & Kernel Bring-Up

### Boot Chain Architecture

```text
Xiaomi Mi 6 Hardware
  └─► EDK2 UEFI Firmware
        └─► rEFInd Boot Manager
              └─► GRUB ARM64 EFI (explicit kernel + DTB + initramfs overlay handoff)
                    └─► Linux 6.0.0-sagit-fedora (Image-6.0.0-sagit-clockfix)
                          └─► Internal ext4 Root Filesystem
                                └─► Fedora Linux 45 ARM64 (KDE Plasma 6.8.1 Wayland)
```

### Overcoming Early Boot Issues

Early experiments with Linux `5.15` and `6.18` builds suffered from black screens, automatic reboots ~15 seconds after startup, ignored DTBs (`EFI stub: Ignoring DTB from command line`), deferred driver probes, and broken clock dependencies.

We standardized on the MSM8998 mainline downstream Linux `6.0` tree (`90169e6e9fee29fae14aeac0a905246be5ce6304`), built as `6.0.0-sagit-fedora`, delivering:

- Reliable ARM64 UEFI boot chain with explicit DTB loading via GRUB.
- Native Fedora 45 ARM64 rootfs on internal UFS storage.
- Modular kernel builds and lightweight initramfs overlays for safe driver testing.
- Independent GRUB entries preserving known-good baselines alongside experimental builds.

*(Newer `6.18` kernel trees remain a future rebase target once hardware enablement stabilizes.)*

---

## 4. Internal Storage & Development Infrastructure

We established a reproducible host-to-phone workflow via UEFI USB mass-storage mode using UUID-based mounting to prevent partition-letter drift or duplicate-mount `ext4` corruption:

| Partition | UUID | Host Mount |
| :--- | :--- | :--- |
| **Root Filesystem (`ext4`)** | `feb4108f-2ed1-4c46-afdf-32aca365912e` | `/mnt/mi6` |
| **EFI System Partition (`FAT32`)** | `6FB1-B704` | `/mnt/esp` |

Every test cycle enforces duplicate-mount checks, `fsck`, SHA256 artifact checksums, kernel module ABI/CRC verification, and clean `sync` + unmount procedures—paired with on-phone automated diagnostic collectors so complex kernel iterations require zero manual touchscreen typing.

---

## 5. Adreno 540 — Hardware Graphics Acceleration

Bringing up the Adreno 540 (`Mesa Freedreno 26.2.3`, renderer `FD540`) required solving clock gating, runtime power management, preemption races, and IOMMU zero-address faults across eight targeted patches:

| Fix / Patch | Purpose |
| :--- | :--- |
| **GCC Clock Fixes** | Keep critical GPU BIMC, SNOC DVM, and MMSS QM clocks enabled during probe |
| **GPUCC Droop-Detector Fix** | Write `GPUCC_GPU_DD_WRAP_CTRL` to the correct register offset (`0x430`) |
| **A5xx Runtime PM Reset** | Issue `REG_A5XX_RBBM_SW_RESET_CMD` on resume before `msm_devfreq_resume()` |
| **Disable Local Preemption** | Set `CP_PREEMPT_ENABLE_LOCAL` to `0x00` in standard ring submissions |
| **Preemption Record & Race Fixes** | Zero stale preemption records on init and serialize triggers via `preempt_start_lock` |
| **Ring-Empty Detection Fix** | Read live hardware `rptr` and track `last_seqno[]` (backported from upstream `b4fb748f0b73`) |
| **RPTR Shadow-Address Fix** | Assign `preempt[i]->rptr_addr` after shadow memory mapping (backported from upstream `32e7083429d46`), eliminating `gpu fault: iova=00000000` |
| **DT & KMS Connector Fixes** | Configure display/GPU dependencies and fix `kmscube` connector indexing + DRM master handoff |

### Verified Rendering Metrics

- **18 / 18** offscreen rendering test stages passed (textures, depth buffer, indexed geometry, multi-context, concurrent processes).
- **66 / 66** native DMA fences signaled cleanly.
- **131 / 131** rendered pixel comparisons matched expected output.
- **0** `iova=00000000` SMMU/IOMMU faults observed.
- **On-screen `kmscube` (Day 6):** ~`30.14 FPS` with `181` buffer swaps and `180` page flips on the physical LCD.

### Conservative GPU Operating Points

| Frequency | Status | Notes |
| :---: | :---: | :--- |
| `257 MHz` | 🟢 Enabled | Active conservative OPP |
| `342 MHz` | 🟢 Enabled | Active conservative OPP |
| `414 MHz` | 🟢 Enabled | Current maximum enabled OPP |
| `515 MHz` – `710 MHz` | 🔴 Disabled | Held back until `vdd`/`vddcx` regulators and thermal limits are validated |

---

## 6. Display & KDE Plasma 6.8.1 Wayland

Following the `kmscube` milestone, we switched the default boot target to `graphical.target` (`sagit.desktop=plasma`), booting **Fedora 45 KDE Plasma 6.8.1 (KWin Wayland)** on the first attempt (`~43s` from boot to graphical session).

- **Active Display Stack:** `drm/msm` $\rightarrow$ Qualcomm MDP/DPU $\rightarrow$ MIPI DSI0 (`DSI-1`) $\rightarrow$ JDI `R63452` (`1920 × 1080 @ 60 Hz`) + WLED backlight.
- **Desktop Optimizations Applied:** Fixed `/home/sagit` directory ownership (resolving KDE Wallet and WirePlumber issues), disabled blur/translucency effects, shortened animations, disabled Baloo indexing, and trimmed diagnostic polling overhead.
- **Remaining Polish:** Window dragging still exhibits occasional choppiness and brief visual artifacts, pointing to KWin scheduling, compositor synchronization, or devfreq ramp-up behavior to tune next.

---

## 7. Wi-Fi, Bluetooth & Qualcomm Firmware

### Wi-Fi (`WCN3990` / `ath10k_snoc`)

Both **2.4 GHz and 5 GHz** Wi-Fi scanning, authentication, DHCP, and internet connectivity are working. After earlier boots showed periodic Wi-Fi firmware restarts, applying a reversible per-interface NetworkManager power-saving configuration eliminated the crashes during testing.

### Qualcomm Remote Processors & Services

| Service / Subsystem | Status | Role |
| :--- | :---: | :--- |
| `rmtfs` | 🟢 Running | Remote processor filesystem access |
| `pd-mapper` | 🟢 Running | Qualcomm protection-domain mapping |
| `diag-router` | 🟢 Running | Diagnostic packet routing |
| `tqftpserv` | 🟢 Enabled | QRTR firmware and remote file transfer |
| **QRTR / GLINK / SMEM** | 🟢 Operational | Interprocessor communication & shared memory |
| **ADSP & SLPI** | 🟢 Operational | Audio and sensor DSP remote processors |
| **Modem (MPSS)** | 🟡 Caveats | Boots cleanly; earlier `DOG_HB` watchdog resets need long-term testing |

### Bluetooth

Uncompressed `WCN3990` Bluetooth firmware files (`qca/crbtfw21.tlv` and `qca/crnv21.bin`) matching the controller's hardware/ROM version have been deployed to `/usr/lib/firmware/qca/`. Device discovery, pairing, and Bluetooth audio remain to be validated.

---

## 8. Battery Monitoring — Custom PMI8998 Driver

The upstream `qcom-fg` driver consistently failed on `sagit` (`Couldn't read CHG base address from dt`, error `-22`), and modifying the DT to force the legacy driver risked unsafe PMIC register writes.

Instead, we built a dedicated, **read-only** kernel module—**`sagit_pmi8998_monitor.ko`** (`compatible = "xiaomi,sagit-pmi8998-battery-monitor"`)—that validates the `PMI8998 v2.1` subtype, decodes raw fuel-gauge telemetry every 20 seconds without touching charging or calibration registers, and exposes `/sys/class/power_supply/sagit-battery` directly to UPower and KDE PowerDevil:

- **Battery Percentage (SOC):** Live reporting in KDE (`0x2c → 17%`, `0x2a → 16%`, `0x28 → 16%`, `0x26 → 15%` verified against raw registers).
- **Battery Voltage, Current & Temperature:** Live hardware readouts (`3.486 V`, `31.3 °C` verified).
- **Charging State Detection:** Accurately switches between `Discharging` and `Charging` in KDE.

---

## 9. Charging & USB-C Role Management

Using our custom battery monitor, we confirmed active USB charging under Linux:

- **Observed USB Input Power:** `~2.5 W` (`~5 V × 500 mA`).
- **Measured Net Battery Charging Current:** `+182 mA` (Test 1), `+270 mA` (Test 2), `+177 mA` (Test 3), and `+167 mA` (Test 4), accompanied by verified battery percentage increases.

### Remaining USB-C Blocker

While USB host mode (external keyboard) worked in earlier boots and charging works in recent boots, **dual-role switching** between host mode (sourcing VBUS to peripherals) and peripheral/charging mode (sinking power) is not yet coordinated automatically. RRADC voltage-sensing tests also exposed stale readings, so safe coordination across DWC3 role switching, PMI8998 boost regulator control, and VBUS discharge is our next major target.

---

## 10. Physical Audio — Day 8 Breakthrough

> **November 27, 2026 (Day 8) — First Audible Physical Speaker Sound Achieved on Native Fedora**

Reconstructing audio on `sagit` required rebuilding the entire pipeline from Xiaomi's production device tree, Android kernel sources, HAL mixer paths, and speaker firmware:

```text
KDE Applications
  └─► PipeWire / WirePlumber
        └─► ALSA / UCM (Internal Speaker Test profile)
              └─► Qualcomm ADSP (Q6ASM audio processing)
                    └─► Quaternary MI2S Digital Audio Transport
                          └─► NXP TFA9888 Smart Amplifier (CoolFlux DSP + tfa98xx_aac.cnt)
                                └─► Physical Speaker (AUDIBLE OUTPUT!)
```

### Stage 1 — Digital Audio & WCD9335 Route

We fixed ADSP reserved memory, enabled SLIMbus and audio frontend dependencies, resolved channel-map bugs, and added the missing `divclk1` clock provider. Linux registered a working ALSA sound card, passing `48 kHz` stereo playback tests and verifying the `WCD9335` route without `RX FIFO` errors.

### Stage 2 & 3 — Hardware Identification & Factory Calibration

Hardware inspection and GPIO100 identification confirmed that `sagit` uses the **NXP TFA9888** smart amplifier (chip revision `0x2c88`) paired with the **AAC speaker variant** (`tfa98xx_aac.cnt`), rather than the `TFA9890` assumed in older ports.

We preserved the device's genuine factory speaker impedance calibration values (confirmed via DSP readback) and kept speaker thermal/excursion protection active:

| Channel | Factory Impedance Calibration |
| :--- | :--- |
| **Channel 1** | `29,722 mΩ` |
| **Channel 2** | `7,588 mΩ` |

### Stage 4 — Five-Build Physical Speaker Progression

| Build | Change / Discovery | Result |
| :---: | :--- | :--- |
| **1. Initial Speaker Test** | Queried impedance before DSP init completed; SpeakerBoost returned `0` and rejected playback | 🔇 Silent |
| **2. DSP Sequencing Fix** | Fixed DSP config order, exposing premature MI2S clock-stability checks | 🔇 Silent (new blocker found) |
| **3. MI2S Timing Fix** | Fixed reset timing; all `21` attempts reached stable clocks, but DSP-owned amp activation was misread as a fault | 🔇 Silent (init progressed) |
| **4. DSP-Owned Output** | Allowed CoolFlux DSP to drive the amp; factory calibration passed, AAC filters loaded, and protected unmute succeeded for `~2 ms` before a standby shutdown timeout cut the tone | 🔇 Silent (protected unmute reached) |
| **5. Standby Sequencing Fix** | Fixed `SBSL` shutdown sequencing, verified Class-D disengagement, and allowed guarded recovery from verified muted standby timeouts | 🔊 **AUDIBLE SOUND!** |

Selecting the **Internal Speaker Test** profile in KDE Plasma produced **real, audible sound from the phone's physical speaker**.

### Remaining Audio Polish

- Retrieve and archive the latest successful boot logs and preserve the exact working audio build.
- Confirm continuous speaker protection across extended playback.
- Investigate quiet output volume and stabilize automatic PipeWire profile switching.

---

## 11. Project Documentation & Preservation

All technical investigations, patches, diagnostic archives, and known-good artifacts are documented in dedicated engineering reports:

| Document | Purpose |
| :--- | :--- |
| `AGENTS.md` | Project constraints and safe hardware testing rules |
| `HANDOFF.md` | Context transfer for ongoing development sessions |
| `PROJECT_STATUS.md` | Overall hardware status and milestone tracking |
| `SESSION_REPORT.md` | Chronological engineering logs |
| `GPU_ANALYSIS.md` | Adreno 540 fault analysis, patches, and `kmscube` metrics |
| `GOLDEN_BASELINE.md` | Verified kernel, DTB, initramfs, and module build IDs |
| `TEST_PLAN.md` | Structured hardware validation procedures |
| `KDE_BRINGUP.md` | KDE Plasma 6.8.1 Wayland configuration and optimizations |
| `KDE_WIFI_BT_REPORT.md` | Desktop tuning, `ath10k` stability fix, and Bluetooth firmware |
| `BATTERY_BRINGUP.md` | Custom `sagit_pmi8998_monitor.ko` design and telemetry validation |
| `CHARGING_BRINGUP.md` | USB charging current tests and power path analysis |
| `USB_ROLE_SWITCH_BRINGUP.md` | DWC3 dual-role switching and VBUS safety investigation |
| `AUDIO_BRINGUP.md` | ADSP, WCD9335, and TFA9888 AAC speaker bring-up logs |

---

## 12. Project Timeline (Days 1–8)

| Day | Date | Milestone | Key Achievements |
| :---: | :--- | :--- | :--- |
| **Days 1–3** | Nov 20–22 | **Boot & Storage Foundation** | EDK2 UEFI, rEFInd, GRUB DTB handoff, internal UFS Fedora rootfs, working console |
| **Day 4** | Nov 23 | **Wireless Breakthrough** | `WCN3990` dual-band Wi-Fi, Qualcomm IPC services (`rmtfs`, `pd-mapper`, `qrtr`) |
| **Day 5** | Nov 24 | **GPU Bring-Up & Debugging** | Runtime PM reset, preemption patches, clock fixes, offscreen triangle rendering |
| **Day 6** | Nov 25 | **THE SPINNING CUBE** | RPTR shadow-address fix (`0` IOMMU faults) + live `kmscube` on physical LCD |
| **Day 7** | Nov 26 | **THE DESKTOP & BATTERY** | Full KDE Plasma `6.8.1` Wayland desktop + custom `sagit_pmi8998_monitor.ko` driver |
| **Day 8** | Nov 27 | **THE FIRST SOUND** | Digital audio, `TFA9888` AAC speaker firmware, factory calibration & physical sound |

---

## 13. Future Development Roadmap

### Phase 1 — Native Linux Foundation (`4 / 4` Complete)
- [x] Working ARM64 kernel boot (`6.0.0-sagit-fedora`)
- [x] Internal UFS storage & `ext4` rootfs
- [x] Native DSI display & touchscreen
- [x] Wi-Fi & Qualcomm remote processor services

### Phase 2 — Core Desktop Functionality (`6 / 6` Complete)
- [x] Hardware-accelerated Adreno 540 graphics (`FD540`)
- [x] Visible on-screen 3D rendering (`kmscube`)
- [x] Full KDE Plasma `6.8.1` Wayland desktop
- [x] Live battery telemetry (`sagit_pmi8998_monitor.ko`)
- [x] Working USB battery charging
- [x] Physical speaker audio output (`TFA9888`)

### Phase 3 — Audio Stabilization (`0 / 5`)
- [ ] Archive and lock in the successful Day 8 audio build
- [ ] Verify continuous speaker protection throughout playback
- [ ] Increase/normalize speaker output volume
- [ ] Stabilize automatic PipeWire / UCM audio profile detection
- [ ] Validate repeated and long-duration audio playback

### Phase 4 — USB-C Dual-Role & Power (`0 / 4`)
- [ ] Restore reliable USB host mode alongside charging
- [ ] Implement safe VBUS boost regulator control
- [ ] Enable automatic USB-C data and power role switching
- [ ] Validate reliable charging and external peripheral handling

### Phase 5 — Performance & Power Optimization (`0 / 9`)
- [ ] Reduce idle battery drain and profile subsystem power draw
- [ ] Optimize CPU frequency scaling and idle (`cpuidle`) states
- [ ] Safely validate GPU regulators (`vdd`/`vddcx`) and higher OPPs (`515–710 MHz`)
- [ ] Eliminate KWin window-dragging choppiness and visual artifacts
- [ ] Improve overall KDE Plasma responsiveness
- [ ] Optimize thermal management under sustained load
- [ ] Test Venus hardware video decoding
- [ ] Validate full system suspend and resume
- [ ] Reduce cold-boot time to desktop

### Phase 6 — Public Release (`0 / 6`)
- [ ] Consolidate and clean up the kernel patch series
- [ ] Build a reproducible, flashable Fedora ARM64 image for `sagit`
- [ ] Write clear installation and bootloader setup instructions
- [ ] Document recovery and mass-storage backup procedures
- [ ] Update the postmarketOS Xiaomi Mi 6 (`sagit`) wiki with hardware findings
- [ ] Publish kernel sources, DTB, custom modules, and compatibility documentation

---

## 14. Power Consumption & Optimization Priorities

While battery charging and live current telemetry now work, the phone currently exhibits elevated battery drain under Fedora. Key suspects to profile once USB-C role switching is stabilized include:

- Always-on clocks and incomplete power-domain gating.
- Incomplete GPU runtime suspend transitions and conservative OPP behavior.
- Suboptimal CPU idle states and background modem/diagnostic service wakeups.
- KWin Wayland compositing overhead.

Systematic power profiling using our custom PMI8998 current monitor will guide both battery-life improvements and safe higher-frequency GPU tuning.

---

## 15. Project Vision & Day 8 Summary

The end goal of this project is a downloadable, reproducible **Fedora Linux ARM64** build that lets anyone with an unlocked Xiaomi Mi 6 turn a 2017 smartphone into a genuine pocket-sized Linux computer—capable of desktop web browsing, SSH, programming, media playback, lightweight gaming, and hardware experimentation.

### Day 8 Status Snapshot

- **Day 1:** Attempting basic kernel boot
- **Day 4:** Working dual-band Wi-Fi
- **Day 6:** Hardware-accelerated spinning 3D cube on the physical LCD
- **Day 7:** Native KDE Plasma Wayland desktop & custom PMI8998 battery driver
- **Day 8:** **Physical speaker playback via NXP TFA9888 + Qualcomm ADSP**
- **Next Major Milestone:** Reliable USB-C dual-role switching & charging coordination
- **Following Milestone:** System-wide performance and battery-life optimization

> **Eight days in:** A 2017 smartphone that previously had incomplete Linux hardware support is now running bare-metal Fedora 45, hardware-accelerated KDE Plasma Wayland, dual-band Wi-Fi, a custom battery-monitoring kernel driver, active USB charging, and a working physical speaker.