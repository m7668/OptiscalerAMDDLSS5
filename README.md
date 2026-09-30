**中文** | [English](README.en.md)


在 **OptiScaler** 上接入 **AMD 神经网络渲染**（DLSS5 on AMD），让 **纯 DLSS / XeSS 游戏** 在 AMD 显卡上跑神经网络降噪；超分辨率仍然由 **FFX/FSR** 完成。

本项目 fork 整合了以下主要上游项目：

- [OptiScaler](https://github.com/optiscaler/OptiScaler)：通用超分辨率与帧生成代理框架，并提供 ASI 插件加载能力；
- [TheAutomatic/dlss-5-amd-project](https://github.com/TheAutomatic/dlss-5-amd-project)：AMD Pre-SR / DLSSNR 整合基底；
- [danielblnc/DLSS-NR-on-AMD](https://github.com/danielblnc/DLSS-NR-on-AMD)：AMD 神经渲染后端，本版本支持 Daniel 0.5.0。

本版本的 DLSSNR 支持通过 Daniel 0.5.0 后端实现；包内 `OptiScaler.ini` 默认仍为 `NrBackend=lmxxf` 且 NR pass 关闭，需按需切换后端并启用。XeMFG（XEFMG）通过 XeFGUnlock ASI 插件接入 OptiScaler。Daniel 的安装器、运行时与权重文件，以及 XeFGUnlock ASI 插件均不随本包分发，需按各自上游说明另行获取。



### lmxxf 配置速查（ini / 菜单）

跨层键与上游同名（`DLSS5_*`）。**Ins 菜单文案不会写入 ini**；优先级：菜单/ini > `native-game-flags.txt` / 环境变量 > 默认值。`DLSS5_STRENGTH` 与 Detail/Colour 为同一组强度，本侧已传入时以菜单为准，无需第二套滑条。

| 菜单位置 | 键 | 说明 |
|---|---|---|
| 顶层 | `TransferStrength` / `ColourStrength` | 网络细节 / 色彩合成（Colour 0–1 保原色，>1 网络色） |
| 顶层 | `DLSS5_FIT_LARGE` | High resolution；大 Color 拟合进网络（含 >1080p） |
| 顶层 | `DLSS5_NETWORK_HEIGHT` | **NR%** auto（默认）或 720 / 900 / 1080 |
| Experimental | `LmxxfAutoExposure` 等 | 无曝光纹理时自动测光；关掉后用 Exposure scale |
| Experimental → Kernels | `DLSS5_HIP_WAVE_OWNED` 等 | 内核 / 显存池 / 字节流 |
| Experimental → Image reuse | `DLSS5_VIT_ADAPTIVE` 等 | 静止帧 ViT 复用（可调，非逐位） |
| Debug / Advanced | `DLSS5_HIP_PDL`、Debug view、增强屏障、early wrap | 排错与兼容开关 |

---

## 安装指南 (Installation Guide)

<details>
<summary><strong>📦 点击展开：压缩包内文件清单</strong></summary>

| 文件/目录 | 作用 |
|---|---|
| `OptiScaler.dll` | 本项目主体（安装时会自动重命名为你选择的代理名称） |
| `OptiScaler.ini` | 核心配置文件（包含 `[DlssNr]` 双后端切换与参数选项） |
| `OptiScaler\` | 核心依赖库（FFX / XeSS / Agility SDK / 插件等） |
| `Setup.bat` / `Setup.ps1` | 交互式图形化安装器（**双击 `Setup.bat` 运行**） |
| `Uninstall_OptiScaler_NR.bat` / `.ps1` | 智能卸载器（安装时自动同步至游戏目录，安全防误删） |
| `tools\` | 内部构建、验证与切换辅助脚本 |
| `Licenses\` | 第三方开源许可证文本 |
| `README.md` / `README.en.md` / `README.es.md` | 本使用文档（中英西三语） |

> **提示**：为遵守各开源协议与版权约束，本压缩包**不随包分发** NVIDIA 专有二进制文件、danielblnc 安装器或未授权模型权重。

</details>

---

### 第一步：准备对应后端的文件

你可以根据需要准备以下任意一种（或两种都准备）：

#### 选项 A：[准备 `lmxxf` 后端文件](https://github.com/lmxxf/dlss5-on-amd-9070xt-porting) 或[点击这里](https://gofile.io/d/RyvcrDxz)获取权重文件
- 准备 `LmxxfNrRuntime.dll`（可从本项目 Release 或 [lmxxf 仓库](https://github.com/lmxxf/dlss5-on-amd-9070xt-porting) 获取）；
- 算子模块目录 `lmxxf-modules\`（官方双架构两层目录结构，包含 `gfx1200` [9060 系列，实验性] 与 `gfx1201` [9070 系列，正式生产] 两个子目录，各含 24 个 `.hsaco` 算子模块、叶子清单与根 `SHA256SUMS` 清单，共 48 个模块；运行时由 D3D12/HIP 设备智能自动匹配，安装器校验完整双包并支持旧版覆盖升级）；
- 着色器目录 `shaders\`（包含 `native_codec_encode.hlsl` 等）；
- 模型权重目录 `native-game-tiled-assets\`（可[点击这里](https://gofile.io/d/RyvcrDxz)直接下载）；
- 将上述文件/文件夹放在与 `Setup.bat` 相同的解压目录下。

升级时直接运行新包的 `Setup.bat` 并选择游戏目录。检测到已有 OptiScaler 后，安装器会建议先卸载，以避免新版文件、模块布局和旧设置冲突：输入 **Y（推荐）**会自动调用新包卸载器，再继续安装；输入 **N** 则直接覆盖安装。卸载会重置 OptiScaler 设置，保留权重和已有备份。覆盖升级会备份旧模块目录；额外 `.hsaco` 保存在安装结束时显示的 `backup-amd-presr-*/lmxxf-modules` 中，不混入新版模块目录，其他兼容的用户文件继续保留。


#### 选项 B：[准备 `danielblnc` 后端文件](https://github.com/danielblnc/DLSS-NR-on-AMD/releases)
- 准备 `dlssnr_on_amd_setup.exe` 与 `nvngx_dlssnr.dll`（推荐，可从 [danielblnc Releases](https://github.com/danielblnc/DLSS-NR-on-AMD/releases) 获取，安装器会自动调用生成 weights）；
- 或者放入已经生成好的 `version.dll` 与 `dlssnr_on_amd_weights.bin`；
- 同样放在与 `Setup.bat` 相同的解压目录下。

---

### 第二步：运行安装器（推荐，一键全自动）

1. 解压本 Release 包到任意临时目录；
2. 将准备好的后端文件与 `Setup.bat` 放在同一目录下；
3. **确认已完全退出游戏**；
4. **双击运行 `Setup.bat`**：
   - 弹出文件夹选择框，选中 **游戏主程序 exe 所在的目录**（例如 `...\Binaries\Win64\`）；
   - 若检测到已有 OptiScaler，输入 **Y** 自动卸载后安装（推荐），或输入 **N** 覆盖安装；
   - 按照提示选择你要注入的 **代理 DLL 名称**（默认为 `dxgi.dll`，推荐；也支持 `winmm.dll`、`d3d12.dll` 等，**不要选 `dinput8.dll`**）；
   - 安装器自动扫描检测你的文件，若同时检测到两个后端，会弹出菜单让你选择安装哪一个，或两者皆装；
   - 安装器自动处理重命名、防双重注入清理、依赖部署，并配置 `OptiScaler.ini`。

---

### 第三步：手动安装（高级玩家）

若你熟悉游戏模组手动放置，可直接将文件拷贝至游戏主程序目录：
1. 将 `OptiScaler.dll` 重命名为你选择的代理名称（如 `dxgi.dll`）放入游戏目录；
2. 将 `OptiScaler.ini` 和 `OptiScaler\` 依赖文件夹复制到游戏目录；
3. **部署后端**：
   - **若使用 `lmxxf`**：将 `LmxxfNrRuntime.dll`、`lmxxf-modules\`、`shaders\`、`native-game-tiled-assets\` 放入游戏目录；
   - **若使用 `danielblnc`**：将 danielblnc 的 `version.dll` 复制三份，分别命名为 `dlssnr_amd_pass1.dll`、`dlssnr_amd_pass2.dll`、`dlssnr_amd_pass3.dll`；将 `dlssnr_on_amd_weights.bin` 放入游戏目录（**切勿保留名为 `version.dll` 的 danielblnc 文件**，以免冲突）；
4. 打开 `OptiScaler.ini`，在 `[DlssNr]` 中设置 `Enabled = true`，并通过 `NrBackend = lmxxf` 或 `NrBackend = daniel` 指定当前生效的后端。

---

### 可选功能：3倍及以上多帧生成（Frame Generation）

<details>
<summary><strong>👉 点击展开：3倍及以上多帧生成方案（Arturs DLSS Enabler / Intel XeFG）</strong></summary>

以下方案为外置可选增强（与 DLSSNR 相互独立），所需文件均不随本包分发，请自行获取。
**注意**：在游戏运行中修改 ini 必须保存并重启游戏生效；保持 `[FrameGen] External=false`；**请勿同时开启两条方案**。

---

#### 方案 1：Arturs（DLSS Enabler）
1. 从 DLSS Enabler 官方发布页获取 `dlss-enabler-headless.dll`（请勿使用第三方整合修改版）：
   [artur-graniszewski/DLSS-Enabler Releases](https://github.com/artur-graniszewski/DLSS-Enabler/releases) 或 [Nexus Mods 757](https://www.nexusmods.com/site/mods/757)
2. 将该 DLL 重命名为 `dlss-enabler-headless.dll`，放入游戏目录中与 `OptiScaler.ini` 并列的 **`OptiScaler\`** 子目录内；
3. 游戏**已有 DLSSG** 时，在 `OptiScaler.ini` 中配置：
   ```ini
   [FrameGen]
   External=false
   Enabled=true
   FGInput=nvngxfg
   FGOutput=auto
   FGNvngxReplacement=Arturs
   ```
   若游戏只有超分没有 DLSSG，使用 `FGInput=upscaler` + `FGOutput=dlssg`；
4. 查看 `OptiScaler.log`，出现 `Artur's initialized` 即代表加载成功。

---

#### 方案 2：Intel XeFG（XeMFG DP4A Unlocker 多倍插帧）
`XeFGUnlock.asi` 与 `XeFGUnlock.ini` 来源于 OptiScaler 社区。
1. 将这两个文件放入游戏目录的 `OptiScaler\plugins\` 子目录中（与 `libxess_fg.dll` 同级），不要加 `-loadlate` 参数；
2. 修改游戏根目录下的 **`OptiScaler.ini`**（非 plugins 内部的 ini）：
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
   - `InterpolationCount`：`1` 代表 2x 插帧，`2` 代表 3x 插帧，以此类推；
   - 实际倍率受硬件能力及插件解锁上限约束；
3. 建议先以 2x 模式跑通，确认无异常后再调高倍率；游戏中可通过 **Page Up** 呼出帧率面板，按 **Page Down** 切换详情观察插帧状态。

</details>

---

##  双后端架构解析与性能实测

本项目目前同时支持两大技术路线的 AMD 神经渲染后端，用户可根据自身硬件与喜好自由选择：

```
                           ┌──► [lmxxf 后端]   ──► 开源 HIP 算子 / 主队列同帧同步 / 深度调优
游戏 DLSS/XeSS 输入 ──► OptiScaler ──┤
                           └──► [daniel 后端] ──► 多槽调度 / 0.3.1 兼容 / 跨系列通用
                                       │
                                       ▼
                             FFX / FSR 超分辨率重建 ──► 游戏画面输出
```

### 一、`danielblnc` 后端：多槽调度（每帧都上 NR）与基准测试

降噪（DLSS5）插在画面渲染路径中：拿到缓冲的那一帧，要等降噪算完才能送去超分出图。原版单槽方案由于每一帧必须等待上一帧降噪完成，存在严重的 GPU 空转挂起（PresentMon 实测约 **MsGPUWait 8.7 ms/帧**）；当算力跟不上时，只能选择整帧跳过降噪，导致画面出现闪烁或间歇性模糊。

本项目在 `danielblnc` 后端上首创了**多槽调度（Multi-slot）**：为每个尚未完成的降噪任务分配独立的并行缓冲槽位，消除了空等上一帧的开销，**做到了尽量每帧都挂上 NR**。

#### 实测对比（鬼武者类，4K FSR 超级性能档 ＝ 720p 渲染；锁 60 帧对照）

| 配置方案 | 帧周期中位 | 大约 FPS | MsGPUWait（GPU空转） | 每帧 NR 状态 |
|---|---:|---:|---:|---|
| **单槽·每帧 NR（旧基线）** | 29.82 ms | **33.5** | **8.69 ms** | 被上一帧卡住，吞吐上不去 |
| **本项目默认多槽** | 22.45 ms | **44.5**（**约 +33%**） | **≈ 0 ms** | **尽量每帧都有 NR** |
| danielblnc 0.3 原生（对照） | 22.35 ms | 44.8 | 0 ms | 原生路径本身不靠跳帧 |

- **收益说明**：在尽量**每帧 NR** 的前提下，相对原版单槽旧基线实测提升约 **+33%**（33.5 → 44.5 FPS）；变快靠的是流水线调度优化，不再空等上一帧，神经网络本身运算耗时未变（`network` 在 720p 下仍约为 12～13 ms）。

#### 槽位选择指南（NR slots：2～5 可调，默认 3）

| 场景测试（4K FSR 超级性能，720p 渲染） | 2 槽 | 3 槽 |
|---|---:|---:|
| **鬼武者** | 19.50 ms，**0 跳过** | 19.49 ms，**0 跳过** |
| **燕云十六声** | 19.05～19.25 ms，**大量跳过 NR 帧**（虽快但无降噪） | 21.78～21.89 ms，**0 跳过** |

- **调参建议**：
  - 在绝大多数常规场景下，**3 槽** 是平衡显存与稳定性的最佳甜点；
  - 《燕云十六声》等高负载游戏极致画质下建议设置为 **≥ 3 槽**；
  - 显存开销极小：每槽仅为渲染分辨率（DLSS 输入）的一张 FP16 纹理（4K 输出配质量档 1440p 渲染仅约 29 MB，原生 4K 仅约 66 MB），按所选数量按需分配。

### 二、`lmxxf` 后端：开源 HIP 算力核心与同帧同步调度

- **双架构硬件支持与自动选择**：
  - **AMD Radeon RX 9070 / 9070 XT (`gfx1201`)**：标准正式生产架构，包含经过完整验证与调优的 24 模块集合；
  - **AMD Radeon RX 9060 (`gfx1200`)**：实验性支持，源码与离线 COMGR 3.0 编译验证完成，硬件实机冒烟与 PDL 表现待后续实机进一步验证；
  - **架构自适应与严格校验**：运行时基于 D3D12 渲染队列绑定与 HIP 设备 LUID 自动匹配对应架构子目录，严格执行 SHA-256 完整性校验与 PDL 孪生符号预检（Preflight）；
- **开源透明**：71 块 ViT 神经网络算子全部由 HIP 实现，针对现代 RDNA 架构进行汇编级优化，引入 LDS 局部作用域栅栏与 C32 CU 模式；
- **主队列同帧同步执行**：OptiScaler 在当前帧的命令列表提交前完成输入录制与外部 Fence 编排，使网络推理与主渲染管线在同一队列周期内紧密衔接，彻底消除外部多进程等待延迟；
- **原生参数支持**：无需重启游戏，可在 Ins 菜单内直接调整细节锐度与色彩校正滑条。

---

##  游戏内设置与控制

1. 启动游戏，进入游戏 3D 渲染画面。
2. 按键盘上的 **Insert (Ins)** 键呼出 OptiScaler 控制菜单。
3. 找到 **DLSS Neural Rendering** 菜单区域，勾选 **Enable NR**。
   - 状态栏将显示当前正在运行的后端：
     - 若为 lmxxf：显示 `AMD NR runtime: lmxxf`；
     - 若为 danielblnc：显示 `AMD NR runtime: 0.3.x`。
4. 画面即时生效：**DLSS 输入拦截 → 神经降噪核心 → FFX/FSR 超分重建**。

### 后端专属调节项说明
- **`lmxxf` 专属**：
  - `Detail strength`：高频细节与亮度增益无级滑条（默认 1.0）；
  - `Colour strength`：色彩饱和与白平衡校正无级滑条（默认 1.0）；
  - `Debug view`：多通道调试可视化（原图、网络输出、差分视图等）。
- **`danielblnc` 专属**：
  - `NR slots`：多槽缓冲数量调节（2～5 槽，默认 3）；
  - `Every-frame`：强制每帧执行 NR 开关；
  - `New wait mode`：0.3.1 状态冻结/恢复新等待模式开关；
  - `Inline same-frame wait`：同帧等待 / async；
  - **Display**：`Tone curve` / `Tone lift` / `Quality`；
  - **Experimental**：`HIP high-priority queue`；
  - **Debug / Advanced**：`dlssnr_on_amd.ini` 额外键说明。

#### daniel 配置键（与 `dlssnr_on_amd.ini` `[DlssNrOnAmd]` 对应）

**优先级：Ins 会话 > `OptiScaler.ini` `[DlssNr]`（Save 后）> `dlssnr_on_amd.ini` / 环境 > 默认。**  
Ins 文案不进 ini；**Save Settings** 才把菜单值写入两侧 ini。

| Ins 菜单 | OptiScaler.ini | daniel 键 | 默认 |
|---|---|---|---|
| New wait mode | `AmdGraphicsWait` | `SpinDraw` | 开 |
| Inline same-frame wait | `AmdInline` | `Async`（0=inline） | 开 |
| NR slots | `AmdSlots` | — | 3 |
| Tone curve | `ToneCurve` | `ToneCurve` | reinhard |
| Tone lift (black) | `ToneLift` | `ToneLift` | 0 |
| Quality | `Quality` | `Quality` | Fast |
| HIP high-priority queue | `QueuePriority` | `QueuePriority` | 关 |
| Style（Pass 1） | `Style` | `Style` | 0 Default |

daniel 自有、未进 Ins 的键（含 **OverlayKey**、`PollSpacing`、`HipDevice` 等）见 `dlssnr_on_amd.ini`；`OverlayKey` 只绑 daniel 自家 overlay。  
高级进程环境变量（无 Ins 开关）：`DLSSNR_NO_REG`、`DLSSNR_CHAIN`、`DLSSNR_NOBLEND`、`DLSSNR_NO_REPACK`、`DLSSNR_WBLOG`。

---

##  排错、日志定位与卸载

### 一、卸载说明
1. 进入**游戏主程序目录**；
2. 双击运行 **`Uninstall_OptiScaler_NR.bat`**；
3. 卸载器会自动列出计划移除的文件与目录，并交互式询问是否保留备份文件夹；输入 `Y` 确认后执行安全清理；
4. **权重保留**：卸载脚本默认设计为保留权重文件夹（`native-game-tiled-assets/` 与 `dlssnr_on_amd_weights.bin`）以及 `nvngx_dlssnr.dll`，避免用户后续重装时需要重复下载大体积资产。

### 二、日志定位与排错

排查问题时，请查看游戏主程序目录（或 XBOX PC 的 `_storage_` 目录）生成的日志：
- `OptiScaler.log`：OptiScaler 核心主日志（检查注入、初始化与各后端创建状态）；
- `amd_bridge.log`：AMD 神经渲染桥接层日志；
- `amd_presr.log`：Pre-SR 调度管线日志；
- `dlssnr_on_amd.log`：Daniel 后端专用运行日志。


---

##  署名与许可 (Attributions & Licenses)

代码链与开源传承（自上而下）：  
[OptiScaler](https://github.com/optiscaler/OptiScaler) → [Dagherbou](https://github.com/Dagherbou/OptiScaler_DLSSNR) → [wilsjo2](https://github.com/wilsjo2/OptiScaler-DLSSNR-PreSR-Multipass) → [Matheus](https://github.com/MatheusGViana/dlss-5-amd-project) → [**本仓库 (TheAutomatic / dlss-5-amd-project)**](https://github.com/TheAutomatic/dlss-5-amd-project)。

- [**OptiScaler**](https://github.com/optiscaler/OptiScaler) — **GPL-3.0 License**：通用超分辨率与神经渲染代理框架；
- [**Dagherbou / OptiScaler_DLSSNR**](https://github.com/Dagherbou/OptiScaler_DLSSNR) — **GPL-3.0 License**：初始接入 DLSS-NR；
- [**wilsjo2 / OptiScaler-DLSSNR-PreSR-Multipass**](https://github.com/wilsjo2/OptiScaler-DLSSNR-PreSR-Multipass) — **GPL-3.0 License**：Pre-SR 超分前执行与 Multi-Pass 架构；
- [**Matheus / dlss-5-amd-project**](https://github.com/MatheusGViana/dlss-5-amd-project) — **GPL-3.0 License**：AMD Pre-SR 桥接方案；
- [**danielblnc / DLSS-NR-on-AMD**](https://github.com/danielblnc/DLSS-NR-on-AMD) — **Custom Non-Commercial / All Rights Reserved**：danielblnc 保留所有权利，禁止未经授权重新分发，本项目不随包分发其二进制，采用外部检测安装方式对接；
- [**lmxxf / dlss5-on-amd-9070xt-porting**](https://github.com/lmxxf/dlss5-on-amd-9070xt-porting) — **MIT License**：开源 HIP 神经渲染算力核心与 71 块网络还原；
- [**RenoDX / clshortfuse**](https://github.com/clshortfuse/renodx) — **MIT License**：`dlssnr.hlsl` 色彩通道合成算法；
- [**本项目 (TheAutomatic / dlss-5-amd-project)**](https://github.com/TheAutomatic/dlss-5-amd-project) — **GPL-3.0 License**：多槽调度架构、主队列同帧同步执行、C-ABI 标准化运行时与 PR 反哺、0.3.1 状态冻结/恢复、双后端共存与智能安装器。

本项目不含 NVIDIA 专有二进制文件、danielblnc 安装工具或未授权分发资产。使用时请遵循各上游开源协议。

