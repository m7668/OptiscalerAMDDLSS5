# OptiScaler AMD Pre-SR — 1.9.9.1

m7668 维护的 Windows 整合发行包，基于 OptiScaler 与 AMD Pre-SR 社区项目，为支持的游戏提供 OptiScaler 安装、超分辨率组件，以及可选的 DLSS-NR Pre-SR 和 XeFG/XeMFG 接入。本项目是社区个人整合包，不代表上游官方发布或背书。

## 基础支持项目

本项目综合并基于以下项目与实现：

- [OptiScaler](https://github.com/optiscaler/OptiScaler)：图形代理、超分辨率和帧生成框架。
- [DLSS-NR-on-AMD](https://github.com/danielblnc/DLSS-NR-on-AMD)：Daniel 的 AMD DLSS-NR 后端；本项目的安装器与配置为兼容版本（包括 0.5.0）提供接入路径。
- [dlss-5-amd-project](https://github.com/TheAutomatic/dlss-5-amd-project)：AMD Pre-SR 神经渲染集成及相关调度实现的基础项目。

XeFG/XeMFG 流程可接入 OptiScaler 社区的 XeFGUnlock ASI 插件；该插件为单独的第三方组件。

## 1.9.9.1 包含内容

OptiScaler 代理、配置、安装器/卸载器；AMD FidelityFX、Intel XeSS、DirectX Agility 依赖及随包许可；lmxxf NR 运行时、gfx1200/gfx1201 模块、着色器、清单，以及中英西三语说明。

本版本保留配置原值：DLSS-NR 默认关闭（[DlssNr] Enabled=false），默认后端为 lmxxf。若使用 Daniel 0.5.0，需从官方发布渠道取得并安装外部运行文件；本包本身不会启用 Daniel 后端。

## 外部依赖与分发范围

公开 ZIP 不包含 Daniel 的代理/运行文件、安装器或模型权重。Daniel 的许可禁止重新分发其软件的全部或部分内容，包括再次上传或捆入其他工具。请从[官方 Releases](https://github.com/danielblnc/DLSS-NR-on-AMD/releases)获取，并遵守[许可](https://github.com/danielblnc/DLSS-NR-on-AMD/blob/master/LICENSE)。安装器说明需由用户另行提供 Daniel 的 version.dll；权重可按其说明生成或提供。

公开 ZIP 也不包含 NVIDIA nvngx_dlssnr.dll，或 XeFGUnlock 的 ASI/INI 文件。XeFGUnlock 的独立再分发许可未随本项目提供；请从作者或获授权的 OptiScaler 社区来源获取。

## 安装与配置

1. 下载并解压本页的 OptiScaler-AMD-PreSR-1.9.9.1.zip。
2. 运行 Setup.bat。无参数启动时会打开游戏目录选择器和代理 DLL 选择菜单。
3. 要启用 Daniel NR 或 XeMFG，请先从对应权利人/授权渠道取得外部文件，再按上游与安装器说明配置。不要将这些文件重新上传至本仓库。

XeFGUnlock 配置示例（建议先从 2× 测试）：

    [Plugins]
    LoadAsiPlugins=true

    [FrameGen]
    External=false
    Enabled=true
    FGInput=dlssg
    FGOutput=xefg

    [XeFG]
    InterpolationCount=1

InterpolationCount=1 表示 2×。实际兼容性取决于游戏、驱动、硬件及插件版本。更多说明见 [TheAutomatic/dlss-5-amd-project](https://github.com/TheAutomatic/dlss-5-amd-project)。

## 许可与免责声明

SHA256SUMS.txt 列出公开 ZIP 内文件的 SHA-256；许可和归属文本位于 Licenses/。各组件保留其自身许可，本整合包不授予所有第三方组件统一许可，也不代表 AMD、Intel、Microsoft、NVIDIA 或上游作者背书。

本包按现状提供。安装前请备份游戏文件和配置。反馈问题时，请附游戏、显卡、驱动版本和必要日志，并移除个人信息。