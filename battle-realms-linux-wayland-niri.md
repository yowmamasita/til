# Running Battle Realms: Zen Edition on Linux with Niri (Wayland)

**Date:** 2026-02-08
**Tags:** `linux`, `gaming`, `wayland`, `niri`, `proton`, `steam`, `gamescope`, `nvidia`

## Problem

Battle Realms: Zen Edition (Steam app 1025600) has several issues on Niri/Wayland with an NVIDIA GPU:

1. **Black screen** without gamescope — the game can't do fullscreen through XWayland on Niri
2. **Animation stuttering** with DXVK (Vulkan) — switching to WineD3D (OpenGL) fixes it
3. **Cursor jittering left-right** when moving the mouse — caused by gamescope 3.15+ on NVIDIA
4. **Broken/loud audio** — PulseAudio buffer too small for Wine

## Setup

- Niri 25.11 (Wayland compositor)
- NVIDIA GTX 1660 (proprietary drivers)
- Steam with Proton
- Display: 3840x2160 @ 1.25x scale

## Fix 1: Gamescope (required)

The game only works through gamescope on Niri. Without it, you get a black screen regardless of fullscreen/windowed settings.

## Fix 2: WineD3D instead of DXVK

The game stutters with DXVK (Vulkan). Forcing WineD3D (OpenGL) via `PROTON_USE_WINED3D=1` fixes the animation stutter.

## Fix 3: Gamescope 3.14.24 for cursor fix (NVIDIA-specific)

Gamescope 3.15+ and 3.16+ have broken cursor handling on NVIDIA GPUs — the cursor visibly jitters left and right when moving the mouse. This is a known upstream issue. Downgrading to **gamescope 3.14.24** fixes it.

On Arch/CachyOS:

```bash
# Download from Arch Linux Archive
curl -LO "https://archive.archlinux.org/packages/g/gamescope/gamescope-3.14.24-1-x86_64.pkg.tar.zst"

# Install (replaces current gamescope)
sudo pacman -U gamescope-3.14.24-1-x86_64.pkg.tar.zst
```

You may want to add gamescope to `IgnorePkg` in `/etc/pacman.conf` to prevent it from being upgraded:

```
IgnorePkg = gamescope
```

## Fix 4: Audio fix

Wine audio is distorted/excessively loud without increasing the PulseAudio buffer. `PULSE_LATENCY_MSEC=60` fixes it.

## Fix 5: HardwareTL

Set `HardwareTL=1` in the game INI to prevent black screen issues.

## Final Configuration

**Steam Launch Options:**
```
PULSE_LATENCY_MSEC=60 PROTON_USE_WINED3D=1 gamescope -W 1920 -H 1080 -w 1920 -h 1080 -f -- %command%
```

**Battle_Realms.ini:**
```ini
[VideoState]
Width=1920
Height=1080
Depth=32
Fullscreen=0
HardwareTL=1
D3D9On12=0
```

**Wine registry** (`compatdata/1025600/pfx/user.reg`):
```
[Software\\Wine\\DirectInput]
"MouseWarpOverride"="disable"
```

**Gamescope version:** 3.14.24 (required for NVIDIA cursor fix)

## What didn't work

- Running without gamescope (black screen in both fullscreen and windowed)
- DXVK/Vulkan rendering (animation stutter)
- Gamescope 3.15.x and 3.16.x (cursor jitter on NVIDIA)
- `--force-grab-cursor` flag (didn't fix cursor jitter)
- `WLR_NO_HARDWARE_CURSORS=1` (didn't fix cursor jitter)
- `--backend sdl` (caused crashes)
- Wine virtual desktop (black screen)

## Notes

- This cursor jitter issue is NVIDIA-specific. AMD GPUs support hardware cursors in gamescope and don't have this problem.
- gamescope 3.14.24 is confirmed stable for cursor behavior across multiple GitHub issues.
- `PROTON_USE_WINED3D=1` is documented to conflict with gamescope in some cases, but works fine here with gamescope 3.14.24.

## References

- [Gamescope Issue #1467 - Mouse not functioning correctly (3.14.29)](https://github.com/ValveSoftware/gamescope/issues/1467)
- [Gamescope Issue #1851 - force-grab-cursor lag spikes for 3.16.x](https://github.com/ValveSoftware/gamescope/issues/1851)
- [Niri Application Issues](https://yalter.github.io/niri/Application-Issues.html)
- [ProtonDB: Battle Realms Zen Edition](https://www.protondb.com/app/1025600)
- [PCGamingWiki: Battle Realms](https://www.pcgamingwiki.com/wiki/Battle_Realms)
