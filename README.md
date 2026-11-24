# Xiaomi Mi 6 (`sagit`) Linux Project — Complete Progress Report

![Linux Boots](https://img.shields.io/badge/Linux%20Boot-Working-brightgreen?style=for-the-badge)
![Display](https://img.shields.io/badge/Display%20Console-Working-brightgreen?style=for-the-badge)
![GPU Rendering](https://img.shields.io/badge/Adreno%20540%20Rendering-Incomplete-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active%20Development-blue?style=for-the-badge)

> **Report Date:** November 25, 2026  
> **Target Hardware:** Xiaomi Mi 6 (`sagit`) — Qualcomm Snapdragon 835 (`MSM8998`) + Adreno 540  
> **Target OS:** Fedora ARM64

---

## Executive Summary

As of **November 25, 2026**, the Xiaomi Mi 6 Linux project has moved well past early kernel bring-up experiments. We now have a functional **Fedora ARM64** installation on internal storage with hardware input, networking, storage access, visible console output, and a GPU stack capable of initializing **EGL 1.5** and submitting real rendering commands.

The primary remaining technical hurdle is getting the **Adreno 540** GPU to reliably **complete rendering jobs** and **present frames** to the display without stalling.

This document separates:
1. **Confirmed capabilities & milestones**
2. **Experiments & patches attempted**
3. **Current bottlenecks & roadmap**

---

## Table of Contents

- [1. Overall Achievement & Architecture](#1-overall-achievement--architecture)
- [2. Booting Linux](#2-booting-linux)
- [3. Hardware Support Matrix](#3-hardware-support-matrix)
- [4. Display Subsystem & KMScon](#4-display-subsystem--kmscon)
- [5. GPU Bring-Up: Adreno 540 (`msm` + Freedreno)](#5-gpu-bring-up-adreno-540-msm--freedreno)
- [6. GPU Patches & Fixes Attempted](#6-gpu-patches--fixes-attempted)
- [7. Breakthrough: EGL Context & Real Command Submission](#7-breakthrough-egl-context--real-command-submission)
- [8. Clocks, Power Management & OPPs](#8-clocks-power-management--opps)
- [9. Device Tree & Firmware](#9-device-tree--firmware)
- [10. The `kmscube` Experiment & Automated Logging](#10-the-kmscube-experiment--automated-logging)
- [11. GRUB Boot Configurations](#11-grub-boot-configurations)
- [12. Graphics Pipeline Diagnosis](#12-graphics-pipeline-diagnosis)
- [13. Action Plan & Priorities](#13-action-plan--priorities)
- [14. Milestone Tracker & Honest Assessment](#14-milestone-tracker--honest-assessment)
- [15. Project Evolution](#15-project-evolution)

---

## 1. Overall Achievement & Architecture

![LineageOS Device Sagit](https://images.openai.com/static-rsc-4/j-EYNgaLjxX95J5gnWEYROXp3pbFN_tp5x_AvZvLKScHvxP2_QZumc2aaJmHr5yitJ8MqkqW7Sco-LbBs1N0T8cr2KL-5Vr-M9COia_pCvW3V4Cvkzu-1vTrnPOTjbU0XzAMLLbW_1yhdt58eoFoEf9pZLszvuS8sn7Dgz5uCW87_N9FJFyHifyKz3Gk216T?purpose=fullsize)
![Xiaomi Mi 6 Motherboard PCB](https://images.openai.com/static-rsc-4/Tp_29ubhlYnkiBkAPJavpEGCIhgwWpf6oOCIGveFNt-WQVYiU-DHrnJgqHt2eokbbS_O9znGcKVfUVEV2Jh6MY7xjH6ByMaDVEiYgw_JCX2qpDodZwFZZfVxgygR7iVjz9f-2qnwc-3IV_iebsJATFFCTu9FFzXu_ZkglKdyzRHM-tnN2WHWI4ZsjHiP5wLY?purpose=fullsize)

Our goal is to turn the **Xiaomi Mi 6 (`sagit`)** into a capable Linux phone.

### Intended System Architecture

```text
[UEFI Firmware]
      │
      ▼
[rEFInd] ──► [GRUB]
                │
                ▼
      [Linux Kernel + Device Tree]
                │
                ▼
      [Fedora ARM64 Userspace (systemd + TTY)]
        ├── Display Stack: DRM/KMS ──► DPU ──► DSI ──► JDI Panel
        └── Graphics Stack: Mesa Freedreno ──► MSM/Adreno ──► Adreno 540
```

---

## 2. Booting Linux

> **Status:** `Working`

Fedora ARM64 boots directly on the Mi 6 hardware using a multi-stage boot chain:

1. Phone firmware / **UEFI**
2. **rEFInd**
3. **GRUB** (with multiple experimental entries alongside a preserved baseline)
4. **Linux kernel** (currently testing `6.0.0-sagit-fedora`; previously explored a Linux 6.18 `sagit` tree)
5. **Fedora ARM64 userspace**
6. **`systemd` & local login terminal**

### What Is Solved
- EFI kernel loading
- Kernel and `initramfs` loading
- Device-tree handoff
- Root filesystem mounting from internal storage
- Fedora userspace startup
- Serial and visible console access
- Local login
- `pstore` / `ramoops` crash debugging enabled and utilized

> **Note:** Having a bootable Linux installation does not yet mean every subsystem is stable or production-ready.

---

## 3. Hardware Support Matrix

| Component | Status | Notes |
| :--- | :---: | :--- |
| **UEFI / GRUB** | ✅ | Boots Linux reliably |
| **Fedora ARM64** | ✅ | Reaches full userspace |
| **Internal Storage** | ✅ | Root filesystem mounted and accessible |
| **USB Host** | ✅ | USB keyboard input confirmed |
| **Touchscreen** | ✅ | Touch input demonstrated |
| **Wi-Fi** | ✅ | Scanning, connecting, and internet access working |
| **Display Console** | ✅ | Visible text output on screen |
| **Adreno EGL** | ✅ | EGL context initializes |
| **3D Rendering** | ⚠️ | Draw commands submit, but job completion hangs |
| **Vulkan** | ❌ | Turnip A540 support not yet demonstrated |
| **Audio** | ❌ | Not established |
| **Camera** | ❌ | Not established |
| **Cellular Modem** | ⚠️ | Remote processors (ADSP, SLPI, modem) start; modem watchdog crashes/restarts under investigation |
| **Battery / Charging** | ⚠️ | Not fully validated |
| **Suspend / Resume** | ⚠️ | Needs proper testing |

---

## 4. Display Subsystem & KMScon

> **Status:** `Visible Console Working`

The display is no longer black—`systemd` logs and a login terminal render directly on the phone's screen.

### Display Pipeline
```text
DRM / KMS (Linux graphics subsystem)
   │
   ▼
Qualcomm DPU / MDSS (Display processing hardware)
   │
   ▼
DSI Controller + DSI PHY
   │
   ▼
JDI FHD-R63452 Panel (Backlight via WLED controller)
```

### Device Tree Findings
- Adreno GPU node at `gpu@5000000` (`status = "okay"`)
- `DSI0` controller at `0xc994000` (`status = "okay"`)
- `DSI1` controller disabled
- `JDI FHD-R63452` panel node defined
- Backlight routed through the **WLED** controller

Full native DRM display mode validation across all graphics modes remains to be completed.

### Resolving the KMScon DRM Master Conflict
While testing graphics from **KMScon**, we hit:

```text
failed to set mode: Permission denied
```

Rather than necessarily indicating a broken GPU, this occurred because KMScon held **DRM master**, blocking other programs from taking display control. Using `/usr/bin/kmscon-launch-gui` allows KMScon to release display ownership while a graphics test runs and reclaim it afterward, letting us test graphics without giving up our terminal.

---

## 5. GPU Bring-Up: Adreno 540 (`msm` + Freedreno)

> **Status:** `Main Remaining Blocker`

![Snapdragon 835 & Adreno 540](https://images.openai.com/static-rsc-4/CiKqIjXTMR7xIVKUflUlfeYj0eTPtswhaTULmL1sa6nmlTDMF2fOYM8bwMSa7phpjc09ScibNLkYXd20nwb3pqQsDUqPYuSsEt01zu2LELy-GMV38L4OBwD-4OTPNYc-uX1sATfaW1gefJQsdVp2VHjw-MzXwKhf3tXlevh2RP0kI9kjrKVmnPX_il2wuMEz?purpose=fullsize)
![Freedreno Project](https://images.openai.com/static-rsc-4/KYDVg7v88JSQpCjfANsiLKRlhv3eK7V5UJNEzrS6K_AMLS8Y-f0GJcRwLH8Mzt92Jn9jP3plLzRwsdrwzPH0SUFGqRmXZx7iwINgaTQgasUEuHPmCMwb-qpc5PAZ_xvMIq5egWzavfhaF82WxoyWTw5pMcvWSOc6v9Df4xuYgCfnVQRKe1lv8gvOviWogHiM?purpose=fullsize)

- **GPU:** Qualcomm Adreno 540
- **Kernel Driver:** `msm`
- **Userspace Driver:** Mesa **Freedreno**
- **Detected Chip ID:** `FD540`

### 5.1 Initial GPU Issues Identified

#### Problem A: Initialization & Power Management
Qualcomm GPUs depend on tightly coupled clocks, regulators, power domains, SMMU translations, and firmware. Early logs showed warnings around:
- Dummy regulators
- GPU `GX` power-domain state (appearing stuck at `off`, raising power-sequencing questions even if physical rails weren't proven off)
- SMMU initialization
- Clock dependencies & deferred probing

#### Problem B: Runtime Suspend / Resume (Hypothesis)
We hypothesized a failure sequence where:
1. GPU renders work.
2. Linux suspends the `GX` power domain.
3. Internal GPU state is lost or stale.
4. Linux powers `GX` back up.
5. Command processor or ringbuffer resumes with stale state and stops completing jobs.

#### Problem C: Rendering Job Hangs
Once command submission was reached, rendering jobs failed to finish—making completion hangs the central GPU issue.

---

## 6. GPU Patches & Fixes Attempted

### 6.1 A5xx Runtime-Resume Reset Patch
We tested an experimental Adreno A5xx patch targeting `REG_A5XX_RBBM_SW_RESET_CMD` during runtime resume and ordering the reset ahead of `devfreq` resumption:
- **Goal:** Clear potentially stale command-processor state after power-domain transitions.
- **Status:** Not yet proven to resolve the hang on its own.

### 6.2 A5xx Preemption Fixes
We examined an upstream four-patch series for A5xx GPU preemption to avoid context-switch stalls:

| Patch | Purpose |
| :--- | :--- |
| **Disable local preemption by default** | Avoid problematic fine-grained preemption |
| **Clear preemption records** | Prevent stale preemption records after resume |
| **Serialize preemption start** | Avoid races between preemption operations |
| **Fix ring-empty detection** | Rely on actual read pointers and completed submissions |

Because our Linux 6.0 tree differed from upstream, the ring-empty logic was adapted to query `gpu->funcs->get_rptr(gpu, ring)` instead of potentially stale ringbuffer memory, alongside `last_seqno[]` tracking and `preempt_start_lock`.

#### Build & Packaging Outcome
- **Status:** Compiled (`a5e316b60bf33bb8b6b546803baa1bf1fbefe9d4`)
- Main source files were restored after building so the base tree stayed clean.
- Packaged into a separate experimental `initramfs` for isolated testing.
- **Caveat:** Experimental backport—semantic equivalence to upstream and effectiveness on `sagit` are not yet confirmed.

---

## 7. Breakthrough: EGL Context & Real Command Submission

We built a surfaceless EGL smoke-test helper using a **GBM** render node to detect the GPU, create an EGL context, compile/link shaders, set up a framebuffer, draw a triangle, call `glFinish()`, and read back pixels.

### Test Results

| Stage | Result | Details |
| :--- | :---: | :--- |
| **GPU Detection** | ✅ Passed | `FD540` detected |
| **EGL Initialization** | ✅ Passed | EGL 1.5 context created |
| **Shader Compilation** | ✅ Passed | Vertex & fragment shaders compiled and linked |
| **Framebuffer** | ✅ Passed | Framebuffer reported complete |
| **Command Submission** | ✅ Confirmed | Real GPU draw commands generated and submitted |
| **GPU Completion** | ❌ Hangs | `glFinish()` timed out after >10 seconds |

### Command Stream Inspection (`.rd`)
Inspecting the captured `.rd` stream confirmed real rendering payloads:
- `494` dwords in the main command buffer
- `283` dwords in an indirect buffer
- `CP_DRAW_INDX_OFFSET` draw command
- `GMEM` resolve commands
- `CACHE_FLUSH_TS` cache flush and timestamp event

### Where Execution Stalls
Execution blocks inside `dma_fence_default_wait` waiting for GPU completion. Possible root causes still under investigation include:
- Command processor lockup
- Ringbuffer read/write pointer mismatch
- Missing fence or timestamp event
- SMMU / memory mapping issue
- Missing GPU interrupt
- Hangcheck / recovery issue
- Remaining clock, regulator, or power-domain sequencing issue

---

## 8. Clocks, Power Management & OPPs

We audited GPU clock dependencies—including interface clock, RBBM timer, memory clock, memory interface clock, RBCPR, and core clock—alongside interconnect and power-domain links and earlier deferred-probe warnings. A dedicated GRUB entry pairs clock adjustments with the A5xx preemption test build.

### Operating Points (OPPs)
Reference `sagit` configurations include higher frequencies (`710 MHz`, `670 MHz`, `596 MHz`, `515 MHz`), which carry risk while regulator definitions remain incomplete. We instead focused conservatively on lower frequencies:
- `257 MHz`
- `342 MHz`
- `414 MHz`

We intentionally avoided chasing stability by bumping clocks before understanding power rails and regulators.

---

## 9. Device Tree & Firmware

We inspected `msm8998-xiaomi-sagit.dtb` as well as a live FDT extracted from the running system to check for discrepancies from UEFI handoff, testing configurations across:
- GPU node status
- Adreno ZAP firmware (`qcom/msm8998/xiaomi/a540_zap.mbn`)
- Reserved memory regions
- GPU power domains
- Display controller (`DPU`/`DSI`) and panel nodes

Whether every live-FDT tweak or ZAP firmware change is strictly required—or resolves the completion hang—remains to be verified.

---

## 10. The `kmscube` Experiment & Automated Logging

To move beyond offscreen rendering toward visible output on the phone display, we ran `/usr/bin/kmscube` via `/usr/bin/kmscon-launch-gui`:

```bash
/usr/bin/kmscon-launch-gui /usr/bin/timeout \
  --signal=TERM --kill-after=3s 15s \
  /usr/bin/kmscube -D /dev/dri/card0 -c 300 -N
```

### Observed Behavior
- The phone froze temporarily.
- Terminal output showed progress past initial device open (including `Using display ... with EGL version 1.5` and Mesa info), but no rotating cube appeared.
- Exact failure stage (surface creation, modesetting, rendering, presentation, or GPU completion) requires inspecting the logs.

### Fixing the Automated Login Hook
An earlier one-shot test failed to trigger because it required `[ "$(tty)" = "/dev/tty1" ]`. We updated the hook to match an interactive local terminal while skipping SSH sessions, checking whether the one-shot test has already run, creating log directories, invoking the launcher, and recording the exit code:

- **Script:** `/home/sagit/.local/bin/sagit-kmscube-once.sh`
- **Test log:** `/home/sagit/.local/state/sagit-kmscube-clockfix.log`
- **Hook log:** `/home/sagit/.cache/sagit-kmscube-hook.log`

Both scripts passed `bash` syntax checks and are awaiting a reboot test to verify execution.

---

## 11. GRUB Boot Configurations

Multiple GRUB entries are maintained side by side so experiments can be compared against a known bootable baseline:

| GRUB Entry | Purpose |
| :--- | :--- |
| `Fedora Linux - Xiaomi Mi 6 - Linux 6.0` | Known bootable baseline |
| `Fedora Linux - Xiaomi Mi 6 - Linux 6.0 (GPU PM test)` | GPU power-management experiment |
| `Fedora Linux - Xiaomi Mi 6 - A5xx Preemption Fix Test` | Experimental A5xx preemption module |
| `Fedora Linux - Xiaomi Mi 6 - Clock fixes + A5xx preemption test` | Combined clock + preemption test |

---

## 12. Graphics Pipeline Diagnosis

Individual layers of the stack already work independently (Fedora boots, text renders, DRM nodes exist, `FD540` is detected, Mesa Freedreno + EGL 1.5 + GLES context + shaders + framebuffer + command submission all work). The gap sits between **job submission**, **completion signaling**, and **frame presentation**.

| Layer | State | Assessment |
| :--- | :---: | :--- |
| **Mesa / Freedreno** | ✅ Working | Builds rendering context and command buffers |
| **MSM / Adreno Driver** | ⚠️ Partial | Accepts and submits GPU jobs |
| **Adreno Command Processor** | 🔍 Investigating | Completion not yet confirmed |
| **DMA Fence / Completion** | ❌ Bottleneck | `glFinish()` times out (`dma_fence_default_wait`) |
| **GBM / DRM Presentation** | ⏳ Unconfirmed | No cube displayed yet |
| **DPU / DSI / Panel** | ⚠️ Partial | Console text works; full graphics presentation pending |

Rather than starting over or swapping drivers at random, we can focus on this narrow middle slice of the pipeline.

---

## 13. Action Plan & Priorities

1. **Priority 1 — Verify Latest Experiment & Collect Logs**
   - Boot the `Clock fixes + A5xx preemption test` entry and log in normally.
   - Let the automated `kmscube` test run, then switch to mass-storage mode.
   - Pull `/home/sagit/.local/state/sagit-kmscube-clockfix.log`, `/home/sagit/.cache/sagit-kmscube-hook.log`, and GPU diagnostics, and confirm the loaded `msm.ko` build ID.
2. **Priority 2 — Diagnose the Hang**
   - Capture GPU interrupt counts before/after rendering, ringbuffer read/write pointers, fence sequence numbers, hangcheck state, GPU power-domain state, SMMU faults, and relevant `dmesg` output.
3. **Priority 3 — Target the Root Cause**
   - Change one variable at a time based on logs (preemption, ringbuffer pointers, runtime PM reset, interrupts, SMMU mappings, or clocks/regulators).
4. **Priority 4 — Achieve First Visible 3D Frame**
   - Validate reliable job completion, `kmscube` rendering, repeated frame presentation, surface modes, and sustained load stability.
5. **Priority 5 — Broader Linux Phone Bring-Up**
   - Desktop environment, audio, battery/charging, thermal monitoring, modem stability, camera, and suspend/resume.

---

## 14. Milestone Tracker & Honest Assessment

| Milestone | Status |
| :--- | :---: |
| Boot Linux | ✅ |
| Reach Fedora userspace | ✅ |
| Use hardware input | ✅ |
| Wi-Fi and internet | ✅ |
| Display text | ✅ |
| Detect Adreno 540 | ✅ |
| Initialize EGL | ✅ |
| Submit actual GPU draw commands | ✅ |
| Complete GPU rendering jobs | ⏳ |
| Show a 3D cube | ⏳ |
| Run a graphical desktop | ⏳ |
| Full daily-driver phone | ⏳ |

### Honest Assessment
We are much further along than when the screen stayed black and the GPU never initialized. Crossing detection, EGL 1.5, shader compilation, and real command submission shows real progress across the graphics stack—even though submitting work isn't the same as completing a frame. The cube isn't guaranteed to be a single patch away, nor is GPU bring-up a lost cause; we now have concrete signals to debug.

---

## 15. Project Evolution

The question has shifted from *"Can we boot Linux on a Xiaomi Mi 6?"* to *"Can we get the Snapdragon 835's native GPU and display pipeline completing and presenting frames reliably under Linux?"*

With Fedora booting, Wi-Fi connected, keyboard and touchscreen input working, internal storage mounted, and console output visible, the next immediate step is retrieving the automated `kmscube` logs before making more blind adjustments—moving steadily toward a hardware-accelerated Linux graphical environment on the Mi 6.
