# Resident Evil 7 frame analysis

Game: Steam, re7.exe (DirectX 12 version), 3840x2160 HDR10 (rgb10a2, colour space 3), ReShade
6.8.0, REFramework, RenoDX (same add-on as Village and RE2). re7_config.ini has ImageQualityRate,
AntiAliasing=TAA, SharpnessSetting, UpscaleType. RE2DLSS's shader ids don't match RE7, except
CAS_CS (csa2614017, identical). See RE2-FINDINGS.md and VILLAGE-FINDINGS.md.

## Post chain (captures: frame-1 = 1.0, frame-2 = 0.5, 1920x1080 -> 3840x2160)

| Pass | Shader | Reads | Writes |
|---|---|---|---|
| **TAA** | **ps0e421d9f** (vs PreTonemapFullScreenTriangleVS d6d34fc5) | **t0 depth** r32 (as r32f), **t1 colour**, **t2 history**, **t3 velocity** rgba16snorm; root b0, b1 | rgba16f #0bc8d |
| history copy | copy | TAA output | #0be58 |
| bloom down/blur | ps75c6c7df, ps20c86918, ps3c48910d | | 480x270 .. 30x16 |
| bloom up | **ps50600766** (one shader at every size) | t0 smaller level | the last one, 1920x1080, **into the TAA output** #0bc8d |
| **tonemap** | **ps0b3afad3** (vs00b71385, as RE2) | **t0 colour** | #0bbf4 rgba16f |
| sharpen? | **ps43abceef** (vs FullScreenTriangleTexVS) | t0 tonemapped | #0bc8d |
| **stretch** | **csa2614017** CAS_CS (same as RE2) | t0 | u0 3840x2160 rgba16f |
| output | ps40b49e09 | | back buffer |

(No HUD in these frames.) Plan: RE2DLSS's design, with the TAA registers of Village (0/1/3), the
bloom redirect only for ps50600766's draw into the TAA output, and ps43abceef redirected to output
size like Village's Sharpness_PS. Next: DumpShaders bytecode to confirm names and conventions.

## Shaders (docs/shaders) and 0.1.0

ps0e421d9f = PreTonemap2_PS (ReadonlyDepth t0, HDRImage t1, PrevHDRImage t2, Velocity t3; same
velocity maths as Village/RE2), ps50600766 = Integration (bloom upsample/add), ps0b3afad3 =
LDRPostProcess_WithTonemap, ps43abceef = PS_RetroFilm (film look after the tonemap: LDRImage t0,
texture coordinates, SceneInfo screenSize, EnvironmentInfo, cbNoiseParam). SceneInfo layout as
Village's (renderOutputId at 520).

0.1.0: RE2DLSS's design with RE7's ids: TAA registers 0/1/3; bloom redirect only for Integration's
draw into the TAA output; tonemap in place (t0); PS_RetroFilm redirected in place to output size
(t0 = tonemapped Y -> Z); CAS_CS replaced by our blit of the final image. No sharpening switch.

Confirmed by the user (2026-11-03): looks good. Log: every DLSS mode via the dropdown
(RE7DLSS.lua works in RE7): 2561x1440, 2227x1252, 1920x1080 -> 3840x2160 and DLAA 3840x2160, no
fallbacks; texture mip bias -1.00 on 3030 material samplers (game started at 0.5).
