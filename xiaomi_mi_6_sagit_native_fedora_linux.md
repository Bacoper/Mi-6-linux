<div align="center">

# 📱 Xiaomi Mi 6 (`sagit`) — Native Fedora Linux

### *Transforming a 2017 Snapdragon 835 flagship into a bare-metal, hardware-accelerated pocket Linux computer*

[![Status](https://img.shields.io/badge/Status-Active_Development-brightgreen?style=for-the-badge&logo=linux)](https://github.com/Bacoper)
[![Day](https://img.shields.io/badge/Milestone-Development_Day_9-blueviolet?style=for-the-badge)](https://github.com/Bacoper)
[![Updated](https://img.shields.io/badge/Updated-Nov_28,_2026-blue?style=for-the-badge)](https://github.com/Bacoper)
[![OS](https://img.shields.io/badge/OS-Fedora_Linux_45_ARM64-51A2DA?style=for-the-badge&logo=fedora&logoColor=white)](https://fedoraproject.org/)
[![Desktop](https://img.shields.io/badge/DE-KDE_Plasma_6_Wayland-1D99F3?style=for-the-badge&logo=kde&logoColor=white)](https://kde.org/)
[![Kernel](https://img.shields.io/badge/Kernel-6.0.0--sagit--fedora-orange?style=for-the-badge&logo=linux&logoColor=white)](https://kernel.org/)
[![GPU](https://img.shields.io/badge/GPU-Adreno_540_%7C_Freedreno_FD540-red?style=for-the-badge)](https://docs.mesa3d.org/drivers/freedreno.html)

<br />

<!-- Replace or add additional photos in docs/images/ as you capture KDE Plasma, battery telemetry, and audio tests -->
<img src="docs/images/kmscube-sagit.jpg" alt="Xiaomi Mi 6 running hardware-accelerated Linux graphics" width="460" style="border-radius: 12px;" />

<p>
  <em>Bare-metal Fedora 45 ARM64 executing directly on Xiaomi Mi 6 hardware via EDK2 UEFI &amp; GRUB — zero Android containers, chroots, or virtualization.</em>
</p>

---

[About](#-about-the-project) •
[Specifications](#-device--software-specifications) •
[Hardware Matrix](#-hardware-compatibility-matrix) •
[Audio Profiles](#-audio-configurations) •
[Achievements](#-major-development-achievements) •
[Known Issues](#-known-issues) •
[Roadmap](#-development-roadmap)

</div>

---

## 🚀 About the Project

This project aims to deliver a fully functional, native **Fedora Linux** desktop experience on the **Xiaomi Mi 6 (`sagit`)**, powered by a custom MSM8998 Linux kernel, hardware-accelerated **Adreno 540** graphics via **Mesa Freedreno**, and **KDE Plasma 6 Wayland**.

The long-term objective is to give the Xiaomi Mi 6 a second life as a genuine, pocket-sized ARM64 Linux computer—complete with reliable daily hardware support, safe battery management, internal audio, and native desktop applications.

> **100% Bare-Metal Execution:** Fedora boots directly from internal UFS storage via EDK2 UEFI, rEFInd, and GRUB. Android, Halium, emulation, virtual machines, and remote desktop streaming are not used.

---

## 📊 Device & Software Specifications

| Category | Component | Details |
| :--- | :--- | :--- |
| **Hardware** | **Device** | Xiaomi Mi 6 (`sagit`, released 2017) |
| | **SoC** | Qualcomm Snapdragon 835 (`MSM8998`, `ARM64` / `AArch64`) |
| | **CPU** | 8-core Kryo 280 (up to `2.45 GHz`) |
| | **GPU** | Qualcomm Adreno 540 |
| | **Memory** | `6 GB` LPDDR4X |
| | **Storage** | Internal UFS 2.1 (`EXT4` root filesystem) |
| | **Display** | `5.15"` JDI `R63452` (`1920 × 1080`, `60 Hz`, MIPI DSI) |
| | **Wireless** | Qualcomm `WCN3990` (2.4 / 5 GHz Wi-Fi + Bluetooth) |
| | **Audio Stack** | Qualcomm `WCD9335` Codec + NXP `TFA9888` Smart Amplifier (`AAC`) |
| | **PMIC** | Qualcomm `PMI8998` |
| **Software** | **Operating System** | Fedora Linux 45 ARM64 |
| | **Desktop Environment** | KDE Plasma 6 (`Wayland` session via Plasma Login Manager) |
| | **Graphics Driver** | Linux `drm/msm` + Mesa `Freedreno` (`FD540` renderer) |
| | **Kernel** | Custom `6.0.0-sagit-fedora` |
| | **Boot Chain** | `EDK2 UEFI` $\rightarrow$ `rEFInd` $\rightarrow$ `GRUB ARM64` $\rightarrow$ `Linux + DTB` |

---

## 🧩 Hardware Compatibility Matrix

### Status Legend
- ✅ **Working** — Verified functional on physical hardware
- 🟡 **Working with limitations** — Functional, but requires tuning, guardrails, or stability work
- 🧪 **Experimental / Under development** — Actively being built and validated across test boots
- ❌ **Currently unavailable** — Not yet supported in the current kernel/userspace stack
- ⚪ **Not yet verified** — Untested or pending subsystem bring-up

<details open>
<summary><b>🖥️ Core System & Boot Infrastructure</b></summary>
<br />

| Feature | Status | Notes |
| :--- | :---: | :--- |
| **Native Linux Boot** | ✅ | Boots bare-metal directly on the Xiaomi Mi 6 |
| **UEFI** | ✅ | EDK2 firmware |
| **rEFInd** | ✅ | Graphical EFI boot manager |
| **GRUB** | ✅ | Explicit kernel, DTB, and initramfs overlay loading |
| **ARM64 CPU** | ✅ | Native `AArch64` application execution |
| **RAM** | ✅ | Full `6 GB` system memory available |
| **Internal UFS Storage** | ✅ | Reliable root filesystem access |
| **EXT4 Filesystem** | ✅ | Full read/write support |
| **Initramfs** | ✅ | Custom Qualcomm firmware & module overlay configuration |
| **Systemd** | ✅ | Normal userspace service initialization |
| **Console / TTY** | ✅ | KMScon / TTY available for recovery and diagnostics |
| **Mass-Storage Mode** | ✅ | Exposed via UEFI for host-side filesystem maintenance |
| **Shutdown & Reboot** | ✅ | Clean shutdown and reboot supported |

</details>

<details open>
<summary><b>🎨 Graphics & Display</b></summary>
<br />

| Feature | Status | Notes |
| :--- | :---: | :--- |
| **Adreno 540 GPU** | ✅ | Hardware command submission and rendering verified |
| **Mesa Freedreno** | ✅ | `FD540` hardware renderer active |
| **DRM / KMS** | ✅ | Native Qualcomm DPU/MDP display output |
| **DSI Display** | ✅ | Internal `1920 × 1080 @ 60 Hz` panel working |
| **Hardware-Accelerated 3D** | ✅ | Physically verified on-screen (`kmscube` & Wayland) |
| **OpenGL ES & EGL** | ✅ | Hardware context creation and shader execution functional |
| **Display Backlight** | ✅ | WLED backlight functional |
| **GPU Runtime PM** | 🟡 | Basic suspend/resume transitions tested |
| **GPU Frequency Scaling** | 🟡 | Capped at conservative operating points (`257–414 MHz`) |
| **Manual Brightness Control** | 🟡 | Functional; desktop slider integration needs further testing |
| **Automatic Brightness** | ❌ | Ambient light sensor integration pending |
| **Automatic Rotation** | ❌ | Accelerometer/gyroscope integration pending |
| **Vulkan (`Turnip`)** | ❌ | Not supported by the current `6.0.0` / `FD540` configuration |
| **Hardware Video Decoding** | ⚪ | Venus VPU requires testing |

</details>

<details open>
<summary><b>🪟 Desktop Environment (KDE Plasma 6 Wayland)</b></summary>
<br />

| Feature | Status | Notes |
| :--- | :---: | :--- |
| **KDE Plasma Desktop** | ✅ | Native Fedora ARM64 Plasma 6 environment |
| **Wayland Session** | ✅ | KWin Wayland compositor active |
| **Plasma Login Manager** | ✅ | Graphical login screen operational |
| **Desktop Applications** | ✅ | Native ARM64 Linux apps running directly on device |
| **Touchscreen** | ✅ | Functional multi-touch input |
| **Window Management** | ✅ | Functional window tiling, switching, and controls |
| **Screen Locking** | ✅ | Functional lock screen |
| **Display Dimming & Sleep** | ✅ | Automatic screen dimming and display sleep observed working |
| **Power Button Wake** | ✅ | Verified waking display from sleep |
| **Desktop Animations** | 🟡 | Tuned, further compositor optimization needed |
| **Desktop Responsiveness** | 🟡 | Occasional stuttering during window dragging |
| **On-Screen Keyboard** | ⚪ | Further Wayland virtual keyboard integration required |
| **Deep System Suspend** | ⚪ | Full SoC sleep/resume not yet qualified |

</details>

<details open>
<summary><b>📡 Networking & Wireless</b></summary>
<br />

| Feature | Status | Notes |
| :--- | :---: | :--- |
| **2.4 GHz & 5 GHz Wi-Fi** | ✅ | Dual-band networks detected via `WCN3990` (`ath10k_snoc`) |
| **NetworkManager** | ✅ | Native wireless management and DHCP |
| **Bluetooth** | ✅ | Basic controller operation confirmed with uncompressed firmware |
| **Wi-Fi Stability** | 🟡 | Functional, though intermittent connection drops still occur |
| **Internet Connectivity** | 🟡 | Successfully demonstrated; tracking intermittent limited-connectivity states |
| **Bluetooth Audio & Peripherals** | ⚪ | Requires audio playback and extended peripheral testing |
| **GPS / GNSS** | ⚪ | Not yet verified |
| **Cellular Modem** | 🧪 | Remoteproc firmware and QRTR/IPC bring-up underway |
| **Mobile Data, SMS & Calls** | ⚪ | Not yet verified |

</details>

<details open>
<summary><b>🔋 Battery, Charging & Power Management</b></summary>
<br />

| Feature | Status | Notes |
| :--- | :---: | :--- |
| **Battery Detection** | ✅ | Custom read-only `PMI8998` kernel monitoring module |
| **Percentage / Voltage / Current / Temp** | ✅ | Live hardware fuel-gauge telemetry verified |
| **KDE Battery Indicator** | ✅ | Integrated via UPower and KDE PowerDevil |
| **Charging Detection** | ✅ | Charging vs. discharging state reported accurately |
| **USB Charging** | 🟡 | Works (`~5 V @ 500 mA`) under specific boot configurations |
| **Remaining Battery Capacity** | 🟡 | Percentage accurate; learned capacity counters limited |
| **Thermal Management** | 🟡 | Safe operating temperatures observed; requires full qualification |
| **Battery Endurance** | 🟡 | Elevated idle drain; power optimization in progress |
| **System Power Optimization** | 🧪 | CPU idle, clock gating, and background service tuning underway |
| **Automatic VBUS Control** | 🧪 | Driver safety validation in progress |
| **Fast Charging** | ❌ | Not implemented (focusing on safe standard charging first) |
| **Battery Health & Time Remaining** | ⚪ | Further telemetry development required |

</details>

<details open>
<summary><b>🔌 USB & Connectivity</b></summary>
<br />

| Feature | Status | Notes |
| :--- | :---: | :--- |
| **USB Keyboard** | ✅ | Physically verified in host mode |
| **USB Host Controller & OTG** | 🟡 | Previously demonstrated; behavior varies across boots |
| **USB Charging** | 🟡 | Verified under charging-capable boot configurations |
| **USB Type-C Attachment Detection** | 🟡 | Hardware cable detection demonstrated |
| **Automatic USB Role Switching** | ❌ | Dual-role host/peripheral coordination under active development |
| **USB Mouse & Flash Drives** | ⚪ | Requires testing once role switching stabilizes |
| **USB Peripheral Mode** | ⚪ | Native Linux gadget configuration incomplete |
| **USB Display Output** | ⚪ | Not verified |

</details>

<details open>
<summary><b>🔊 Audio Subsystem</b></summary>
<br />

| Feature | Status | Notes |
| :--- | :---: | :--- |
| **Qualcomm ADSP** | ✅ | Firmware loaded and Q6DSP IPC operational |
| **WCD9335 Audio Codec** | ✅ | Successfully initialized over SLIMbus |
| **ALSA & Quaternary MI2S** | ✅ | Sound card, PCM interfaces, and MI2S transport verified |
| **PipeWire & WirePlumber** | ✅ | Digital playback and session management operational |
| **KDE Audio Controls** | ✅ | Audio profiles and volume sliders integrated |
| **AAC Speaker Firmware** | ✅ | Matching `tfa98xx_aac.cnt` firmware identified |
| **Factory Speaker Calibration** | ✅ | Original impedance calibration preserved and verified via readback |
| **Speaker Fault Detection** | ✅ | Hardware/DSP protective shutdown verified working |
| **TFA9888 Smart Amplifier** | 🟡 | Physical speaker playback achieved; stability tuning ongoing |
| **SpeakerBoost DSP** | 🟡 | Initialized; channel-specific validation pending |
| **Bottom Loudspeaker** | 🟡 | Playback demonstrated; signal-dependent protection faults observed |
| **Upper Earpiece (Mono Mode)** | 🧪 | Independent left-channel playback mode under active validation |
| **Stereo Playback** | 🧪 | Experimental dual-speaker configuration preserved for testing |
| **Microphone & USB-C Audio** | ⚪ | Not yet verified |

</details>

<details>
<summary><b>🧭 Sensors & Additional Hardware</b></summary>
<br />

| Feature | Status | Notes |
| :--- | :---: | :--- |
| **Power Button** | ✅ | Functional (screen sleep/wake & power menu) |
| **Volume Buttons** | ⚪ | Desktop keymap integration requires verification |
| **Accelerometer & Gyroscope** | ⚪ | `iio-sensor-proxy` & auto-rotation planned |
| **Ambient Light Sensor** | ⚪ | Automatic display brightness planned |
| **Proximity Sensor & Magnetometer** | ⚪ | Not yet verified |
| **Vibration Motor, NFC & IR Blaster** | ⚪ | Not yet verified |
| **Camera Flash & Notification LED** | ⚪ | Not yet verified |
| **Fingerprint Reader** | ❌ | Not currently supported |
| **Front & Rear Cameras** | ❌ | Not currently supported |

</details>

---

## 🔊 Audio Configurations

Because the Xiaomi Mi 6 uses an asymmetric dual-speaker setup driven by an **NXP TFA9888** smart amplifier (upper earpiece + bottom main loudspeaker), three distinct boot/driver profiles are maintained to balance daily reliability, hardware safety, and ongoing debugging:

### 1. Configuration 1 — Single-Speaker Mode *(Recommended Daily Target)*
* **Status:** 🧪 **Under Development / Intended Default**
* **Architecture:** Routes mono audio exclusively to the **upper earpiece** while keeping the bottom loudspeaker electrically disabled.
* **Key Features:**
  * Retains native ALSA, PipeWire, and KDE volume integration.
  * Preserves genuine `AAC` speaker firmware and factory impedance calibration.
  * Uses modified `TFA9888` driver logic for independent channel operation with active fault monitoring and conservative playback limits.
* **Next Step:** Complete live left-channel DSP protection validation for sustained daily playback.

### 2. Configuration 2 — Experimental Dual-Speaker Mode *(Stereo)*
* **Status:** 🧪 **Experimental / Physical Stereo Demonstrated**
* **Architecture:** Enables both the **upper earpiece** and **bottom loudspeaker** simultaneously using the `AAC` `music_dual` firmware profile and full `SpeakerBoost` DSP processing.
* **Known Issue:** During heavy bass transients (specifically right before high-energy beat drops), the right amplifier channel (bottom loudspeaker) repeatedly triggers a **speaker-protection fault**, automatically muting and shutting down the amplifier until reboot.

> ⚠️ **Hardware Safety Warning:**
> Dual-speaker mode is strictly experimental. Do **not** repeatedly trigger speaker-protection faults or bypass amplifier thermal/excursion safeguards. Different Mi 6 units may exhibit different speaker tolerances.

### 3. Configuration 3 — Speakers Disabled *(Safe Recovery Default)*
* **Status:** ✅ **Working Recovery Configuration**
* **Architecture:** Holds the `TFA9888` amplifier in hardware reset, preventing speaker activation while keeping the rest of the OS and desktop completely functional. Currently set as the default development boot while mono earpiece validation completes.

---

## 🏆 Major Development Achievements

### 1. Native Bare-Metal Fedora ARM64 Boot
Established a rock-solid UEFI boot pipeline (`EDK2` $\rightarrow$ `rEFInd` $\rightarrow$ `GRUB`) on internal UFS storage with an `EXT4` root filesystem, explicit DTB handoff, custom initramfs overlays, and isolated recovery entries.

### 2. Adreno 540 Hardware Acceleration (`Mesa Freedreno FD540`)
Overcame Qualcomm clock gating, runtime power management sequencing, A5xx preemption races, and `iova=00000000` SMMU write faults (via the ringbuffer `RPTR` shadow-address initialization fix):
* ✅ **18 / 18** offscreen OpenGL ES rendering stages passed
* ✅ **66 / 66** native DMA fence checks signaled cleanly
* ✅ **131 / 131** exact rendered pixel comparisons verified
* ✅ **3 / 3** explicit GPU runtime suspend/resume cycles passed
* ✅ **0** zero-address IOMMU faults or SMMU context-fault interrupts
* ✅ **~30 FPS** physical on-screen `kmscube` rendering achieved

### 3. KDE Plasma 6 Wayland Desktop
Booted a full native KDE Plasma Wayland session directly on the device with touchscreen support, graphical login, screen sleep/wake, and desktop responsiveness tuning (disabling heavy blur/translucency, Baloo indexing, and background diagnostic overhead).

### 4. Custom Read-Only `PMI8998` Battery Monitoring Driver
Bypassed the broken upstream `qcom-fg` probe (`Couldn't read CHG base address from dt`) by developing a dedicated, non-destructive read-only kernel module (`sagit_pmi8998_monitor.ko`) that exposes live **SOC percentage, voltage, current, temperature, and charging status** directly to `UPower` and KDE PowerDevil without altering factory PMIC calibration.

### 5. Verified USB Charging
Demonstrated real physical battery charging under Linux (`~5 V @ 500 mA`), confirmed via positive net battery charging current telemetry in the custom fuel-gauge driver.

### 6. Physical Speaker Audio Bring-Up
Reconstructed the full MSM8998 audio chain—ADSP remoteproc, `WCD9335` codec over SLIMbus, Quaternary MI2S routing, ALSA/PipeWire profiles, `TFA9888` startup/standby sequencing, and factory impedance calibration—achieving audible physical speaker output on Day 8.

---

## ⚠️ Known Issues

| Issue | Severity | Summary & Investigation Focus |
| :--- | :---: | :--- |
| **Stereo Speaker Fault** | 🟡 Medium | Right channel (bottom speaker) trips speaker-protection shutdown on heavy bass transients; mono earpiece mode is being finalized as default. |
| **USB-C Role Conflicts** | 🟡 Medium | Host mode (VBUS out) and charging (power sink) are not yet coordinated automatically across DWC3, Type-C detection, and PMIC VBUS control. |
| **Wi-Fi Instability** | 🟡 Medium | Intermittent connection drops or "limited connectivity" states despite active association; power management and WCN3990 firmware behavior under review. |
| **Desktop Stuttering** | 🟡 Low–Med | KWin Wayland window dragging shows occasional frame pacing hiccups; compositor sync and GPU/CPU frequency scaling need tuning. |
| **Elevated Idle Drain** | 🟡 Medium | Battery discharge rate is higher than desired; requires `cpuidle` tuning, clock gating audit, and deeper GPU/modem power management. |
| **Missing Sensors & Suspend** | ⚪ Low | Auto-rotation, auto-brightness, and full SoC deep suspend are not yet implemented. |
| **Linux 6.0 Baseline** | ⚪ Info | Currently running a patched `6.0.0` tree; rebasing onto a newer mainline kernel is planned once hardware support matures. |

---

## 🗺️ Development Roadmap

<details open>
<summary><b>Phase 1 — Core Linux Foundation (Complete)</b></summary>

- [x] Native ARM64 Linux boot (`6.0.0-sagit-fedora`)
- [x] Internal UFS storage & `EXT4` root filesystem
- [x] EDK2 UEFI, rEFInd, and GRUB integration
- [x] Console / TTY recovery environment
- [x] Systemd userspace initialization
- [x] Core SoC hardware initialization

</details>

<details open>
<summary><b>Phase 2 — Graphics & Desktop</b></summary>

- [x] Internal MIPI DSI display (`1920 × 1080 @ 60 Hz`)
- [x] Native DRM/KMS pipeline
- [x] Adreno 540 hardware acceleration (`Mesa Freedreno FD540`)
- [x] Physical on-screen 3D rendering
- [x] KDE Plasma 6 Wayland desktop
- [ ] Extended GPU benchmark qualification
- [ ] Further KWin Wayland compositor & frame-pacing optimization

</details>

<details open>
<summary><b>Phase 3 — Connectivity</b></summary>

- [x] Dual-band Wi-Fi bring-up (`2.4 GHz` & `5 GHz`)
- [x] Internet connectivity via NetworkManager
- [x] Basic Bluetooth controller initialization
- [ ] Resolve intermittent Wi-Fi connection drops
- [ ] Extended Bluetooth pairing & A2DP audio testing
- [ ] GPS / GNSS support
- [ ] Optional cellular modem stability & data/voice qualification

</details>

<details open>
<summary><b>Phase 4 — Audio</b></summary>

- [x] ALSA & PipeWire / WirePlumber integration
- [x] `WCD9335` codec & Quaternary MI2S initialization
- [x] `TFA9888` amplifier bring-up & physical speaker playback
- [x] Experimental dual-speaker stereo playback
- [x] Factory impedance calibration preservation & fault shutdown verification
- [x] Independent channel configuration implemented
- [ ] Validate sustained upper-earpiece mono playback
- [ ] Enable reliable mono audio by default
- [ ] Diagnose and resolve bottom loudspeaker protection faults
- [ ] Stabilize stereo playback & bring up internal microphone

</details>

<details>
<summary><b>Phase 5 — USB-C & Charging</b></summary>

- [x] Physical USB charging demonstrated (`~5 V @ 500 mA`)
- [x] Physical USB host mode & keyboard input demonstrated
- [x] Custom `PMI8998` battery telemetry driver
- [ ] Reliable USB OTG across reboots
- [ ] Automatic USB-C data and power role switching
- [ ] Safe automatic VBUS regulator control
- [ ] Reliable charging across all boot modes & USB peripheral expansion

</details>

<details>
<summary><b>Phase 6 — Power & Performance</b></summary>

- [ ] Optimize Kryo 280 CPU frequency scaling (`cpufreq`)
- [ ] Improve deep CPU idle (`cpuidle`) residency
- [ ] Safely validate higher Adreno 540 operating points (`> 414 MHz`)
- [ ] Reduce idle battery power consumption
- [ ] Improve thermal management under sustained load
- [ ] Qualify reliable full-system suspend and resume
- [ ] Eliminate KWin desktop stuttering

</details>

<details>
<summary><b>Phase 7 — Sensors & Usability</b></summary>

- [ ] Accelerometer & gyroscope integration (`iio-sensor-proxy`)
- [ ] Automatic screen rotation in KDE Plasma Wayland
- [ ] Ambient light sensor & automatic brightness adjustment
- [ ] Enhanced touchscreen gestures & on-screen keyboard integration
- [ ] Hardware volume button keymap verification

</details>

<details>
<summary><b>Phase 8 — Public Release</b></summary>

- [ ] Complete hardware compatibility qualification
- [ ] Finalize unified daily-driver GRUB boot configuration
- [ ] Write step-by-step installation and UEFI flashing documentation
- [ ] Document mass-storage recovery & backup procedures
- [ ] Clean up and publish kernel patches & device trees
- [ ] Audit firmware redistribution compliance
- [ ] Release reproducible Fedora ARM64 image for `sagit`

</details>

---

## 🛠️ Development Approach & Philosophy

* **Pocket Computer First:** The primary goal is delivering a responsive, reliable native Linux computing environment (desktop apps, web browsing, SSH, development tools, media playback, and hardware experimentation). Smartphone-specific peripherals like cameras and fingerprint sensors remain secondary to core system stability.
* **Safe, Reproducible Iteration:** Every test cycle preserves known-good kernel images, DTBs, initramfs overlays, and boot logs so broken experiments never risk bricking the working environment.
* **Hardware-First Verification:** All patches are validated on a physical Xiaomi Mi 6 with factory hardware calibration and thermal/electrical protections kept intact.

---

## 🙏 Credits & Acknowledgments

This independent community project stands on the shoulders of the open-source and mainline mobile Linux ecosystem:

* **MSM8998 Mainline Linux & EDK2 UEFI Contributors** — foundational Snapdragon 835 kernel, device-tree, and bootloader support
* **Mesa Freedreno Developers** — open-source Adreno `FD540` graphics stack
* **postmarketOS Developers & Maintainers** — mobile Linux device bring-up research and hardware documentation
* **Fedora Linux & KDE Plasma Teams** — upstream ARM64 userspace and Wayland desktop stack
* **ALSA, PipeWire & Qualcomm Linux Contributors** — ADSP, SLIMbus, and audio infrastructure
* **Xiaomi & LineageOS Contributors** — reference device configurations aiding hardware reverse-engineering

*Disclaimer: This is an independent community development project and is not affiliated with or endorsed by Xiaomi, Fedora, or KDE.*

---

## ⚠️ Disclaimer

This software and kernel configuration are **experimental** and under active development. Certain hardware subsystems remain incomplete or unstable, and experimental builds may cause kernel panics, unexpected shutdowns, or elevated power draw. Always maintain complete backups of your partitions and understand UEFI mass-storage recovery procedures before testing.

---

<div align="center">

### 🎯 Final Objective
**Give the Xiaomi Mi 6 a second life as a fully usable, hardware-accelerated, pocket-sized Linux computer.**

</div>