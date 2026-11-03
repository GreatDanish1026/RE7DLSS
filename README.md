# RE7DLSS 1.0.0

ReShade add-on for **Resident Evil 7** (current DirectX 12 version) that replaces the game's TAA
with **NVIDIA DLSS**. The game only has FSR 1 (a spatial upscaler); this gives it DLAA at native
resolution and DLSS upscaling from Quality to Performance, with the game's bloom, tonemapping (and
RenoDX HDR) and film look on top of the DLSS image.

NVIDIA RTX graphics card. ReShade 6.8 or later with add-on support, and REFramework for the DLSS
mode dropdown.

## What it does

- **DLSS in place of TAA** (F9 to switch on and off): DLSS Super Resolution 310.9.16 takes the
  game's own colour, depth, motion vectors and camera jitter.
- **DLSS mode**: DLAA (100%), Quality (67%), Balanced (58%) or Performance (50%) set the game's
  render scale (its Image Quality setting) while DLSS is on, and keep it there if the in-game
  slider moves. "Game's Image Quality slider" leaves it to the menu. This needs REFramework: the
  bundled script `reframework\autorun\RE7DLSS.lua` sets the render scale.
- **Upscaling**: below 100%, DLSS renders the full output resolution, and the game's bloom,
  tonemapping/colour grading (and RenoDX's HDR, if installed) and film look run on it at full
  resolution. The HUD is drawn as usual. The game's FidelityFX CAS sharpening isn't applied to the
  upscaled image (at 100%/DLAA it is).
- **Sharper textures at lower render sizes** (on by default): a mip bias of log2(render scale) on
  the game's material textures, as NVIDIA recommends for DLSS. Set when the game starts, so after
  changing the DLSS mode, restart the game for the textures to follow.
- **DLSS model**: K (transformer, default), J, or NVIDIA's default.

Settings are in ReShade's Add-ons tab (Home key) under RE7DLSS, saved in `ReShade.ini` under
`[RE7DLSS]`.

## Install

**With Fluffy Mod Manager**: put `RE7DLSS-1.0.0.zip` in Fluffy Mod Manager's `Games\RE7\Mods`
folder (or drag it onto the manager) and enable it. It installs into the game folder:

- `RE7DLSS.addon64` (the add-on) and `nvngx_dlss.dll` (NVIDIA's DLSS runtime, 310.9.16)
- `reframework\autorun\RE7DLSS.lua` (sets the render scale for the DLSS mode)
- `RE7DLSS-README.md`, `RE7DLSS-THIRD_PARTY_NOTICES.txt`, `RE7DLSS-licenses\`

**By hand**: copy the contents of the zip's `RE7DLSS` folder into the game folder (where
`re7.exe` is), leaving out `modinfo.ini`.

Then:

1. ReShade 6.8 or later with full add-on support must be installed for `re7.exe` (DirectX 12).
2. REFramework must be installed for the DLSS mode dropdown (without it, use the game's Image
   Quality slider: 0.67 = Quality, 0.58 = Balanced, 0.5 = Performance).
3. In the game, keep Anti-Aliasing on TAA (DLSS replaces that pass) and FidelityFX Super
   Resolution off.
4. Open ReShade (Home), Add-ons tab, RE7DLSS: DLSS is on by default; pick a DLSS mode.

If the NVIDIA App has a DLSS model override for the game, it takes precedence over the model here.

## Notes

- Tested at 3840x2160 with HDR (RenoDX) on an NVIDIA RTX card, at every DLSS mode offered.
- A few frames during loading may show the game's own upscaling; that's expected.
- The render scale the DLSS mode sets is the game's Image Quality setting, so the game saves it in
  `re7_config.ini`. With DLSS off (F9) or the mode on "Game's Image Quality slider", the script
  puts the game's previous value back.
- `RE7DLSS.log` next to `re7.exe` records what the add-on did and why it fell back, if it did.

## Building

`build.ps1` builds `bin\RE7DLSS.addon64` with Visual Studio 2022's C++ tools (MSVC): `-Install`
copies it, `nvngx_dlss.dll` and the REFramework script into the game folder; `-Package` makes the
Fluffy Mod Manager zip in `bin\release`. `nvngx_dlss.dll` isn't in the repository: put NVIDIA's
signed 310.9.16 file (from the Streamline SDK 2.14.16 release, `bin\x64`) in `third_party\bin`.
`docs\FINDINGS.md` describes how the game renders and what the add-on changes (forked from
RE2DLSS and VillageDLSS; their analyses are in `docs\RE2-FINDINGS.md` and
`docs\VILLAGE-FINDINGS.md`).

## Licence

MIT (see `LICENSE`). Third-party code and NVIDIA's runtime: `THIRD_PARTY_NOTICES.txt`.
