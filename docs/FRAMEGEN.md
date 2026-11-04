# FSR and frame generation (1.1.0, 2026-11-04)

Ported from RE4DLSS 1.2.0 (`src/fsr.cpp`, `src/framegen.cpp`, `src/fsrfg.cpp`; RE4DLSS's
docs/FSR.md and docs/FRAMEGEN.md have the background). What's specific to RE7:

## FSR 3.1 / FSR 4

Same hook as DLSS: the TAA draw (PreTonemap2_PS ps0e421d9f). Inputs t0 depth (depth buffer, state
depth read + shader resource 0xE0), t1 colour, t3 velocity (rgba16snorm NDC offsets, y up: scale
(0.5 w, -0.5 h)), output X (the DLSS output texture, UAV). Camera: near = SceneInfo
cameraNearPlane (byte 392), FOV = 2 atan(1 / projElement[0].y), far infinite (reversed Z),
metres. No reactive mask (RE7 has no FSR 2 passes).

## Frame generation

- Inputs at the TAA draw, whoever upscales (also the game's TAA): depth t0 (0xE0, copied),
  velocity t3 (0xC0), DLSS-G mvecScale (0.5, -0.5), FSR FG motion scale (0.5 w, -0.5 h);
  SceneInfo bound right there (no search as in RE4); reset on a camera cut.
- `FrameGenTag` now takes the depth and motion states and the motion scale separately (RE4: r32f
  depth, UV motion), and the Streamline project id comes from version.h.
- Picture before the HUD: the HUD is drawn into the output-size texture the output pass
  (ps40b49e09) writes to the back buffer. That texture is copied right after the scene goes into
  it: after the game's CAS_CS (u0), after our blit (upscaling, OnFinalCompute), or after ps0cbe90b1
  (native size). After the output pass, the copy goes back into it and the output pass is repeated
  into our back-buffer-format target. Pending work runs at the next barrier / pipeline bind /
  render target bind on the list (while the output pass's state is still bound).
- Exclusive full screen (needed for the game's HDR): in the first test, the game's first
  SetFullscreenState(TRUE) calls through Streamline's swapchain failed with
  DXGI_ERROR_NOT_CURRENTLY_AVAILABLE and the game stayed in a 1280x1024 window. The proxy's
  SetFullscreenState is hooked: on that error the game window is brought to the front and the call
  retried (3 s budget), and the log names the window that had the focus. In later tests the first
  call succeeded (retry not needed).

Tested 2026-11-04 (RTX, 4K HDR, exclusive full screen, REFramework + RenoDX): DLSS upscaling +
DLSS FG 2x, DLSS + FSR FG 2x, picture before the HUD in use, clean exits.
