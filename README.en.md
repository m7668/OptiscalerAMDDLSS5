# OptiScaler AMD Pre-SR — 1.9.9.1

A Windows integration package maintained by m7668. It combines OptiScaler with community AMD Pre-SR work to provide an installer, upscaling components, and optional DLSS-NR Pre-SR and XeFG/XeMFG integration for supported games. This is a community package, not an official release or endorsement by the upstream projects.

## Foundation projects

This project combines and builds on:

- [OptiScaler](https://github.com/optiscaler/OptiScaler): graphics proxy, upscaling, and frame-generation framework.
- [DLSS-NR-on-AMD](https://github.com/danielblnc/DLSS-NR-on-AMD): Daniel's AMD DLSS-NR backend; this package's installer and configuration provide an integration path for compatible versions, including 0.5.0.
- [dlss-5-amd-project](https://github.com/TheAutomatic/dlss-5-amd-project): a foundation for AMD Pre-SR neural-rendering integration and related scheduling work.

The XeFG/XeMFG path can use the separate XeFGUnlock ASI plugin from the OptiScaler community.

## Included in 1.9.9.1

OptiScaler proxy, configuration, installer/uninstaller; AMD FidelityFX, Intel XeSS, and DirectX Agility dependencies with accompanying notices; lmxxf NR runtime, gfx1200/gfx1201 modules, shaders, manifests, and Chinese/English/Spanish documentation.

This release preserves the supplied configuration defaults: DLSS-NR is disabled ([DlssNr] Enabled=false), and the selected backend is lmxxf. To use Daniel 0.5.0, obtain and install its external runtime from an official source; this package does not enable Daniel's backend by itself.

## External dependencies and redistribution

The public ZIP does not include Daniel's proxy/runtime files, installer, or model weights. Daniel's license prohibits redistribution of the software in whole or in part, including re-uploading it or bundling it with another tool. Obtain files from the [official Releases](https://github.com/danielblnc/DLSS-NR-on-AMD/releases) and follow the [license](https://github.com/danielblnc/DLSS-NR-on-AMD/blob/master/LICENSE). The installer documentation requires users to provide Daniel's version.dll; weights may be generated or supplied as described upstream.

The public ZIP also omits NVIDIA's nvngx_dlssnr.dll and the XeFGUnlock ASI/INI files. No separate redistribution license for XeFGUnlock was included with this project; obtain it from the author or an authorized OptiScaler community source.

## Installation and configuration

1. Download and extract OptiScaler-AMD-PreSR-1.9.9.1.zip from this release.
2. Run Setup.bat. With no arguments, it opens a game-folder picker and proxy-DLL selection menu.
3. To enable Daniel NR or XeMFG, obtain the external files from their rights holders or authorized channels and follow upstream and installer instructions. Do not re-upload those files here.

Example XeFGUnlock settings (start by testing 2×):

    [Plugins]
    LoadAsiPlugins=true

    [FrameGen]
    External=false
    Enabled=true
    FGInput=dlssg
    FGOutput=xefg

    [XeFG]
    InterpolationCount=1

InterpolationCount=1 corresponds to 2×. Compatibility depends on the game, driver, hardware, and plugin version. See [TheAutomatic/dlss-5-amd-project](https://github.com/TheAutomatic/dlss-5-amd-project) for integration notes.

## Licensing and disclaimer

SHA256SUMS.txt lists SHA-256 hashes for the files in the public ZIP. License and attribution notices are in Licenses/. Each component retains its own terms; this bundle does not grant a single license for all third-party components and is not endorsed by AMD, Intel, Microsoft, NVIDIA, or upstream maintainers.

Provided as-is. Back up game files and configuration before installation. When reporting issues, include the game, GPU, driver version, and relevant logs after removing personal information.