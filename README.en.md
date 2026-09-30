[中文](README.md) | **English**


This release package combines work from these primary upstream projects:

- [OptiScaler](https://github.com/optiscaler/OptiScaler): the upscaling and frame-generation proxy, including ASI plugin loading;
- [TheAutomatic/dlss-5-amd-project](https://github.com/TheAutomatic/dlss-5-amd-project): the AMD Pre-SR / DLSSNR integration base;
- [danielblnc/DLSS-NR-on-AMD](https://github.com/danielblnc/DLSS-NR-on-AMD): the AMD neural-rendering backend, with Daniel 0.5.0 supported by this version.

This version supports DLSSNR through the Daniel 0.5.0 backend. The bundled `OptiScaler.ini` still defaults to `NrBackend=lmxxf` with the NR pass disabled; switch backends and enable it as needed. XeMFG (XEFMG) integration uses the XeFGUnlock ASI plugin through OptiScaler. Daniel's installer, runtime and weights, and the XeFGUnlock ASI plugin are not included; obtain them from their upstream sources and follow their respective licenses.



**In-game menu**
- Menu layout cleanup with lmxxf / daniel feature toggles

**Known issues**
- Using danielblnc 0.4.3 / 0.5.0 as the backend may crash in the Wo Long 2 Demo; not yet determined whether this project or the upstream backend is at fault.
- Baldur's Gate 3: (not confirmed whether this is DX11-related) you may sometimes need to toggle the in-game DLSS quality level and the Ins-menu top-left DX11-to-DX12 FSR 4.1.1 upscaler back and forth before NR takes effect.

### lmxxf config map (ini / Ins menu)

Cross-layer keys use the same `DLSS5_*` names as upstream. **Ins labels are never written to the ini.** Priority: menu/ini > `native-game-flags.txt` / environment > defaults. `DLSS5_STRENGTH` is the same pair as Detail/Colour strength; when the host sends those fields the menu wins — no second slider.

| Menu location | Key | Notes |
|---|---|---|
| Top | `TransferStrength` / `ColourStrength` | Detail / colour mix (Colour 0–1 keeps game colour) |
| Top | `DLSS5_FIT_LARGE` | High resolution; fit large Color into the network (incl. >1080p) |
| Top | `DLSS5_NETWORK_HEIGHT` | **NR%** auto (default) or 720 / 900 / 1080 |
| Experimental | `LmxxfAutoExposure`, paper white | Meter when no usable exposure texture |
| Experimental → Kernels | `DLSS5_HIP_WAVE_OWNED`, … | Kernels / shared pool / byte stream |
| Experimental → Image reuse | `DLSS5_VIT_ADAPTIVE`, … | Static-frame ViT reuse (tunable) |
| Debug / Advanced | `DLSS5_HIP_PDL`, debug view, enhanced barriers, early wrap | Diagnostics |

---



### 🚀 Key Highlights

1. **Unreal Engine 5 (UE5) Compatibility Fixes for lmxxf (*Neverness to Everness*, *Palworld*, etc.) (1.9.0.3)**
   - **Render Queue Binding (*Neverness to Everness*)**: Correctly binds to the game's actual Direct rendering queue executing DLSS-NR commands, avoiding crashes and session invalidation caused by viewport render queue vs. Swapchain present queue mismatch (`QueueContract: targetQueue != sessionQueue`).
   - **Queue Safety Guard**: Adds COM identity checks during command list execution callbacks to skip HIP evaluation gracefully when unexpected command lists are dispatched.
   - **GPU Draining on Migration**: Flushes the GPU prior to queue migration to reduce VRAM leak risks from session recreation.
   - **Command List Split & Startup Fixes (*Palworld*)**: Hardens split eligibility checks, adjusts default log level to 2 (Information) to remove startup hashing delays, and introduces log rate-limiting.

2. **New `lmxxf` Neural Rendering Backend**
   - **Open-Source Compute Core**: In addition to maintaining compatibility with the existing `danielblnc` backend, integrates the open-source HIP neural rendering core.
   - **Same-Frame Queue Execution**: Embeds input recording, HIP asynchronous inference, and output barrier synchronization within the game's primary command queue before upscaling (Pre-SR).
   - **DLSS / XeSS Proxy Support for lmxxf**: Enables the `lmxxf` backend to intercept DLSS and XeSS inputs before upscaling (Pre-SR), allowing games without native FSR to use the lmxxf denoiser.
   - **Dual-Backend Support**: Seamlessly supports both `lmxxf` and `danielblnc` backends. Switch between them anytime in `OptiScaler.ini` via `NrBackend=lmxxf` or `NrBackend=daniel`.
   - **Memory & Stability Hardening**: Optimizes `fast_prefix` mode to bypass the redundant 201MB noise buffer allocation, reducing host memory footprint and startup overhead. Enhances GPU LUID matching in Fake NVAPI to prevent cross-adapter crashes in multi-GPU or spoofed environments.
   - **⚠️ Resolution Recommendation**: The current `lmxxf` model architecture is optimized for **pre-upscale render resolution ≤ 1080p**:
     - **4K Output**: Recommended to use **FSR Performance** (1080p render) or Ultra Performance (720p render).
     - **1440p (2K) Output**: Recommended to use **FSR Quality / Balanced / Performance** (all render at or below 1080p).
     - **1080p Output**: Supports **Native 1080p** or any FSR scaling mode.

3. **Installer Update**
   - Fixed installer interaction logic, supporting dual-backend selection and safe coexistence/overwrites.

4. **Menu (Ins Menu) Polish & Real-Time Parameter Sliders**
   - **Context-Aware Menu**: Automatically hides Daniel-specific options (e.g. slots, passes, new wait) when in `lmxxf` mode to eliminate confusion.
   - **Layout Fixes**: Resolves layout clumping between `Enable NR` and `AMD processing`, restoring clear vertical structure.
   - **Live Sliders**: Introduces continuous sliders for `Detail strength` and `Colour strength`, along with a real-time `Debug view` channel selector for live visual diagnostics.

---


##  Installation Guide

<details>
<summary><strong>📦 Click to expand: Package Contents</strong></summary>

| File / Directory | Purpose |
|---|---|
| `OptiScaler.dll` | Main binary (renamed during installation to your chosen proxy name) |
| `OptiScaler.ini` | Core configuration file (contains `[DlssNr]` dual-backend options) |
| `OptiScaler\` | Core dependencies (FFX, XeSS, Agility SDK, plugins) |
| `Setup.bat` / `Setup.ps1` | Interactive installer (**Double-click `Setup.bat`**) |
| `Uninstall_OptiScaler_NR.bat` / `.ps1` | Safe uninstaller (automatically placed in game directory) |
| `tools\` | Internal build and verification utilities |
| `Licenses\` | Third-party open-source licenses |
| `README.md` / `README.en.md` / `README.es.md` | Documentation (Chinese / English / Spanish) |

> **Note**: To comply with upstream licenses and distribution policies, this package **does not bundle** NVIDIA proprietary binaries, danielblnc installer tools, or unauthorized model weights.

</details>

---

### Step 1: Prepare Backend Files

Prepare either backend (or both for side-by-side coexistence):

#### Option A: [Prepare `lmxxf` Backend Files](https://github.com/lmxxf/dlss5-on-amd-9070xt-porting) or [Click Here](https://gofile.io/d/RyvcrDxz) to download weights
- `LmxxfNrRuntime.dll` (from project release or [upstream lmxxf repository](https://github.com/lmxxf/dlss5-on-amd-9070xt-porting));
- Module folder `lmxxf-modules\` (official dual-architecture layout containing `gfx1200` [9060 series, experimental] and `gfx1201` [9070 series, production] subfolders, with 24 `.hsaco` compute modules each, leaf manifests, and root `SHA256SUMS` for a total of 48 modules; automatically matched by the runtime based on D3D12/HIP GPU architecture; the installer validates the complete bundle and supports overwriting older flat installs);
- Shader folder `shaders\` (with `native_codec_encode.hlsl`);
- Weights folder `native-game-tiled-assets\` (can be downloaded [here](https://gofile.io/d/RyvcrDxz));
- Place these in the same extracted folder as `Setup.bat`.

To upgrade, run the new package's `Setup.bat` and select the game folder. When OptiScaler is detected, Setup recommends uninstalling first to avoid conflicts between the new files, module layout, and old settings. Choose **Y (Recommended)** to run the new uninstaller automatically and continue installing, or **N** to overwrite the existing installation. Uninstall resets OptiScaler settings but keeps weights and existing backups. An overwrite upgrade backs up the old module folder; extra `.hsaco` files are preserved under `backup-amd-presr-*/lmxxf-modules` at the path shown when installation finishes, while other compatible user files remain in place.


#### Option B: [Prepare `danielblnc` Backend Files](https://github.com/danielblnc/DLSS-NR-on-AMD/releases)
- `dlssnr_on_amd_setup.exe` and `nvngx_dlssnr.dll` (from [danielblnc Releases](https://github.com/danielblnc/DLSS-NR-on-AMD/releases); installer generates weights automatically);
- Or pre-generated `version.dll` and `dlssnr_on_amd_weights.bin`;
- Place in the same extracted folder as `Setup.bat`.

---

### Step 2: Run the Installer (Recommended)

1. Extract this release to any temporary folder;
2. Place your backend files alongside `Setup.bat`;
3. **Ensure the game is not running**;
4. **Double-click `Setup.bat`**:
   - Select your game's executable directory (e.g. `...\Binaries\Win64\`);
   - If OptiScaler is already installed, choose **Y** to uninstall automatically before installing (recommended), or **N** to overwrite;
   - Select your proxy DLL name (default `dxgi.dll`, recommended; `winmm.dll`, `d3d12.dll` also supported; **do not use `dinput8.dll`**);
   - If both backends are detected, choose which to install or install both;
   - The installer sets up proxies, clears conflicting duplicate files, and configures `OptiScaler.ini`.

---

### Step 3: Manual Installation

If you prefer manual file placement:
1. Rename `OptiScaler.dll` to your proxy name (e.g. `dxgi.dll`) and copy it to the game directory;
2. Copy `OptiScaler.ini` and the `OptiScaler\` folder into the game directory;
3. **Deploy Backend Files**:
   - **For `lmxxf`**: Copy `LmxxfNrRuntime.dll`, `lmxxf-modules\`, `shaders\`, and `native-game-tiled-assets\` into the game directory;
   - **For `danielblnc`**: Duplicate `version.dll` into `dlssnr_amd_pass1.dll`, `dlssnr_amd_pass2.dll`, `dlssnr_amd_pass3.dll`; copy `dlssnr_on_amd_weights.bin` into the game directory (**do not leave a file named `version.dll`** to prevent double injection);
4. In `OptiScaler.ini`, set `Enabled = true` under `[DlssNr]` and set `NrBackend = lmxxf` or `NrBackend = daniel`.

---

### Optional: 3x+ Frame Generation

<details>
<summary><strong>👉 Click to expand: 3x+ Frame Generation (Arturs DLSS Enabler / Intel XeFG)</strong></summary>

These options are independent of DLSSNR. Required files are not bundled; obtain them separately.
**Note**: Game restarts are required when changing INI settings. Keep `[FrameGen] External=false`. **Do not enable both simultaneously**.

---

#### Option 1: Arturs (DLSS Enabler)
1. Obtain `dlss-enabler-headless.dll` from the official author:
   [artur-graniszewski/DLSS-Enabler Releases](https://github.com/artur-graniszewski/DLSS-Enabler/releases) or [Nexus Mods 757](https://www.nexusmods.com/site/mods/757)
2. Place `dlss-enabler-headless.dll` into the **`OptiScaler\`** subfolder in the game directory;
3. If the game has **native DLSSG**, configure in `OptiScaler.ini`:
   ```ini
   [FrameGen]
   External=false
   Enabled=true
   FGInput=nvngxfg
   FGOutput=auto
   FGNvngxReplacement=Arturs
   ```
   If the game only has upscaling without DLSSG, use `FGInput=upscaler` + `FGOutput=dlssg`;
4. Check `OptiScaler.log` for `Artur's initialized`.

---

#### Option 2: Intel XeFG (XeMFG DP4A Unlocker Multi-Frame Generation)
1. Place `XeFGUnlock.asi` and `XeFGUnlock.ini` into `OptiScaler\plugins\` (alongside `libxess_fg.dll`);
2. Configure `OptiScaler.ini` in the game root:
   ```ini
   [Plugins]
   LoadAsiPlugins=true

   [FrameGen]
   External=false
   Enabled=true
   FGInput=dlssg
   FGOutput=xefg

   [XeFG]
   InterpolationCount=1
   ```
   - `InterpolationCount`: `1` for 2x, `2` for 3x, etc.;
3. Test with 2x first before increasing multipliers. Press **Page Up** for FPS overlay and **Page Down** for detailed stats.

</details>

---

## 3. Dual-Backend Architecture & Benchmarks

This project supports two distinct AMD Neural Rendering backend technologies:

```
                          ┌──► [lmxxf Backend]   ──► Open-source HIP / Same-frame queue / Deep tuning
Game DLSS/XeSS Inputs ──► OptiScaler ──┤
                          └──► [daniel Backend] ──► Multi-slot scheduling / 0.3.1 compat / Universal
                                      │
                                      ▼
                            FFX / FSR Upscaling ──► Final Game Output
```

### 1. `danielblnc` Backend: Multi-Slot Scheduling (NR on Every Frame)

Denoising (DLSS5) is inserted directly into the frame rendering pipeline: a frame must finish denoising before passing to upscaling. In single-slot setups, each frame must wait for the preceding frame's denoising to complete, causing severe GPU idle stalls (**MsGPUWait ~8.7 ms/frame** in PresentMon). Under heavy load, frames are forced to skip denoising entirely, causing visible shimmering or blur.

This project introduced **Multi-Slot Scheduling**: allocating independent parallel buffers (slots) so each frame can proceed without waiting for the previous frame's GPU completion.

#### Benchmark (Onimusha-type workload, 4K FSR Ultra Performance = 720p render; locked 60 fps comparison)

| Configuration | Median Frame Time | Approx FPS | MsGPUWait (GPU Stall) | Per-Frame NR Status |
|---|---:|---:|---:|---|
| **Single-slot · per-frame NR (old baseline)** | 29.82 ms | **33.5** | **8.69 ms** | Blocked by previous frame |
| **Our Multi-slot default** | 22.45 ms | **44.5** (**+33%**) | **≈ 0 ms** | **NR on virtually every frame** |
| Upstream 0.3 native (baseline) | 22.35 ms | 44.8 | 0 ms | Native pipeline does not drop frames |

- **Key Takeaway**: Delivers a **+33%** throughput increase (33.5 → 44.5 FPS) by optimizing pipeline scheduling rather than compromising denoising quality; the neural network computation itself remains unchanged (~12–13 ms @ 720p).

#### Slot Count Recommendations (NR slots: 2–5, default 3)

| Test Scene (4K FSR Ultra Performance, 720p render) | 2 Slots | 3 Slots |
|---|---:|---:|
| **Onimusha** | 19.50 ms, **0 skipped** | 19.49 ms, **0 skipped** |
| **Where Winds Meet** | 19.05–19.25 ms, **Frequent skips** | 21.78–21.89 ms, **0 skipped** |

- **Recommendations**:
  - **3 slots** is the ideal sweet spot for most titles;
  - Heavy scenes like *Where Winds Meet* on max settings benefit from **≥ 3 slots**;
  - VRAM cost is minimal: each slot is one FP16 render-resolution texture (~29 MB at 1440p render; ~66 MB at native 4K).

### 2. `lmxxf` Backend: Open-Source HIP Compute & Same-Frame Queue Execution

- **Dual-Architecture Support & Auto-Selection**:
  - **AMD Radeon RX 9070 / 9070 XT (`gfx1201`)**: Standard verified production architecture with 24 tuned compute modules;
  - **AMD Radeon RX 9060 (`gfx1200`)**: Experimental support, verified through COMGR 3.0 compilation; real-device smoke test and PDL speedup pending hardware verification;
  - **Adaptive Architecture & Strict Verification**: Automatically selects matching arch subfolder based on D3D12 queue binding and HIP device LUID, with SHA-256 integrity verification and PDL twin symbol preflight;
- **Open Source & Hardware Optimized**: All 71 ViT neural network modules are implemented in HIP, tuned for modern RDNA architectures with LDS workgroup fences and C32 CU mode;
- **Same-Frame Queue Execution**: OptiScaler schedules input recording, HIP inference, and barrier synchronization on the main queue before command list close, eliminating external cross-process synchronization delays;
- **Dynamic Parameter Controls**: Real-time continuous sliders for detail/brightness enhancement and color calibration directly in the Ins menu.

---

##  In-Game Settings & Controls

1. Launch the game and enter 3D rendering.
2. Press **Insert (Ins)** to open the OptiScaler overlay menu.
3. Locate the **DLSS Neural Rendering** section and check **Enable NR**.
   - The status line indicates the active runtime:
     - `AMD NR runtime: lmxxf` for lmxxf backend;
     - `AMD NR runtime: 0.3.x` for danielblnc backend.
4. Active pipeline: **DLSS Inputs → Neural Denoising → FFX/FSR Upscaling**.

### Backend Controls
- **`lmxxf` Specific**:
  - `Detail strength`: Continuous slider for detail and brightness enhancement (default 1.0);
  - `Colour strength`: Continuous slider for color saturation and balance (default 1.0);
  - `Debug view`: Live visualization of inputs, network output, and difference buffers.
- **`danielblnc` Specific**:
  - `NR slots`, `Every-frame`, `New wait mode`, `Inline same-frame wait`;
  - **Display**: `Tone curve` / `Tone lift` / `Quality`;
  - **Experimental**: `HIP high-priority queue`;
  - **Debug / Advanced**: extra `dlssnr_on_amd.ini` keys.

**Priority:** Ins session > `OptiScaler.ini` `[DlssNr]` (after Save) > `dlssnr_on_amd.ini` / env > defaults.  
Ins labels are not written to ini; **Save Settings** persists menu values to both inis.  
Unlisted daniel keys (`OverlayKey`, `PollSpacing`, ...) stay in `dlssnr_on_amd.ini`; `OverlayKey` binds only daniel's own overlay.  
Advanced process env (no Ins toggle): `DLSSNR_NO_REG`, `DLSSNR_CHAIN`, `DLSSNR_NOBLEND`, `DLSSNR_NO_REPACK`, `DLSSNR_WBLOG`.

---

##  Troubleshooting, Logs & Uninstallation

### 1. Uninstallation
1. Open the **game directory**;
2. Run **`Uninstall_OptiScaler_NR.bat`**;
3. Review the proposed deletion list, choose whether to keep backup folders, and confirm with `Y`;
4. **Preserved Weights**: The script is designed to preserve user weight files (`native-game-tiled-assets/` and `dlssnr_on_amd_weights.bin`) and `nvngx_dlssnr.dll` by default, avoiding repeated multi-gigabyte downloads.

### 2. Log Locations & Diagnostics

Inspect the following logs in the game directory (or `_storage_` for Microsoft Store / XBOX PC games):
- `OptiScaler.log`: Main initialization, hooking, and backend creation log;
- `amd_bridge.log`: AMD bridge layer log;
- `amd_presr.log`: Pre-SR dispatch log;
- `dlssnr_on_amd.log`: danielblnc runtime log.


---

##  Attributions & Licenses

Codebase heritage (top to bottom):  
[OptiScaler](https://github.com/optiscaler/OptiScaler) → [Dagherbou](https://github.com/Dagherbou/OptiScaler_DLSSNR) → [wilsjo2](https://github.com/wilsjo2/OptiScaler-DLSSNR-PreSR-Multipass) → [Matheus](https://github.com/MatheusGViana/dlss-5-amd-project) → [**This Repository (TheAutomatic / dlss-5-amd-project)**](https://github.com/TheAutomatic/dlss-5-amd-project).

- [**OptiScaler**](https://github.com/optiscaler/OptiScaler) — **GPL-3.0 License**: Universal upscaling proxy framework;
- [**Dagherbou / OptiScaler_DLSSNR**](https://github.com/Dagherbou/OptiScaler_DLSSNR) — **GPL-3.0 License**: Initial DLSS-NR integration;
- [**wilsjo2 / OptiScaler-DLSSNR-PreSR-Multipass**](https://github.com/wilsjo2/OptiScaler-DLSSNR-PreSR-Multipass) — **GPL-3.0 License**: Pre-SR and Multi-Pass architecture;
- [**Matheus / dlss-5-amd-project**](https://github.com/MatheusGViana/dlss-5-amd-project) — **GPL-3.0 License**: AMD Pre-SR bridge;
- [**danielblnc / DLSS-NR-on-AMD**](https://github.com/danielblnc/DLSS-NR-on-AMD) — **Custom Non-Commercial / All Rights Reserved**: Author retains all rights; redistribution prohibited; integrated via external detection;
- [**lmxxf / dlss5-on-amd-9070xt-porting**](https://github.com/lmxxf/dlss5-on-amd-9070xt-porting) — **MIT License**: Open-source HIP neural rendering core and 71-block network recovery;
- [**RenoDX / clshortfuse**](https://github.com/clshortfuse/renodx) — **MIT License**: Color compositing algorithms in `dlssnr.hlsl`;
- [**This Project (TheAutomatic / dlss-5-amd-project)**](https://github.com/TheAutomatic/dlss-5-amd-project) — **GPL-3.0 License**: Multi-slot scheduling, same-frame queue execution, C-ABI runtime creation and upstream PR, 0.3.1 state freeze/restore, dual-backend coexistence, and smart installer.

This distribution contains no NVIDIA proprietary binaries, danielblnc installer tools, or unauthorized model weights. Please respect all upstream licenses.

