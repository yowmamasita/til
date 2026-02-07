# Running Civilization III on Linux with Niri (Wayland)

**Date:** 2026-02-08
**Tags:** `linux`, `gaming`, `wayland`, `niri`, `proton`, `steam`, `gamescope`

## Problem

Sid Meier's Civilization III Complete (Steam app 3910) has two major issues when running on Niri/Wayland via Proton:

1. **Black screen on fullscreen** - The game tries to do exclusive fullscreen mode-switching via DirectDraw, which XWayland on Niri doesn't support.
2. **Map turns black when scrolling** - A long-standing Wine bug ([#41930](https://bugs.winehq.org/show_bug.cgi?id=41930)) where Wine's OSMesa integration zeroes out the game's pixel buffer, destroying rendered terrain tiles.

## Setup

- Niri 25.11 (Wayland compositor)
- NVIDIA GTX 1660
- Steam with Proton 10.0
- Display: 3840x2160 @ 1.25x scale

## Fix 1: Fullscreen via Gamescope

Gamescope acts as a nested compositor that handles the fullscreen/resolution mismatch. Set Steam launch options:

```
gamescope -W 1920 -H 1080 -w 1920 -h 1080 -f -- %command%
```

- `-W`/`-H`: output (display) resolution
- `-w`/`-h`: inner resolution the game sees
- `-f`: fullscreen

Without `-f`, the game runs in a 1080p window instead.

## Fix 2: Widescreen Resolution

Civ 3 is natively 4:3 only (1024x768). The Conquests expansion supports a `KeepRes=1` INI setting that makes the game adopt the desktop resolution, enabling true widescreen.

Edit `Conquests/conquests.ini` and add under `[Conquests]`:

```ini
KeepRes=1
```

With gamescope, the game sees the inner resolution (`-w`/`-h`) as its desktop, so set that to your desired widescreen resolution.

**Note:** City screens and main menu backgrounds are hardcoded to 1024x768 and will appear letterboxed. This is unfixable.

## Fix 3: Black Map via C3X Mod

The [C3X mod](https://github.com/maxpetul/C3X) is a community patch that fixes the Wine GDI rendering bug (among many other improvements). It has a built-in setting `draw_lines_using_gdi_plus = wine` that auto-detects Wine and uses GDI+ instead of GDI for line drawing, avoiding the OSMesa buffer corruption.

### Installing C3X on Linux

1. Download the latest release from [GitHub](https://github.com/maxpetul/C3X/releases)
2. Extract into the `Conquests/` directory
3. Run the TCC compiler and installer through Proton:

```bash
cd "/path/to/Sid Meier's Civilization III Complete/Conquests"

# Compile the installer
STEAM_COMPAT_CLIENT_INSTALL_PATH="$HOME/.local/share/Steam" \
STEAM_COMPAT_DATA_PATH="$HOME/.local/share/Steam/steamapps/compatdata/3910" \
"$HOME/.local/share/Steam/steamapps/common/Proton 10.0/proton" run \
"Z:\\path\\to\\Conquests\\tcc\\tcc.exe" \
-m32 -Wl,-nostdlib -g -lmsvcrt -luser32 -lkernel32 -DC3X_INSTALL ep.c -o temp.exe

# Run the installer to patch Civ3Conquests.exe
STEAM_COMPAT_CLIENT_INSTALL_PATH="$HOME/.local/share/Steam" \
STEAM_COMPAT_DATA_PATH="$HOME/.local/share/Steam/steamapps/compatdata/3910" \
"$HOME/.local/share/Steam/steamapps/common/Proton 10.0/proton" run \
"Z:\\path\\to\\Conquests\\temp.exe"

# Clean up
rm temp.exe
```

The installer backs up the original exe as `Civ3Conquests-Unmodded.exe` and patches `Civ3Conquests.exe` with C3X.

## Working Modes

Both modes require C3X installed and `KeepRes=1` in `conquests.ini`.

### 4K Windowed (no gamescope)

No Steam launch options needed. With `KeepRes=1`, the game runs at native 4K (3840x2160) in a window through XWayland. This works but the UI becomes very small and hard to read.

### 1080p Fullscreen (with gamescope) -- Recommended

**Steam Launch Options:**
```
gamescope -W 1920 -H 1080 -w 1920 -h 1080 -f -- %command%
```

Gamescope provides a nested compositor so the game gets proper fullscreen at 1080p. The UI is readable and the game fills the screen. Without gamescope, fullscreen is a black screen due to DirectDraw/XWayland limitations on Niri.

### conquests.ini

```ini
[Conquests]
KeepRes=1
```

### C3X

Installed via INSTALL.bat/TCC (permanent exe patch)

## References

- [Wine Bug #41930](https://bugs.winehq.org/show_bug.cgi?id=41930) - Black terrain root cause
- [Proton Issue #315](https://github.com/ValveSoftware/Proton/issues/315) - Civ III Proton tracking issue
- [C3X Mod (GitHub)](https://github.com/maxpetul/C3X)
- [PCGamingWiki: Civ III](https://www.pcgamingwiki.com/wiki/Sid_Meier%27s_Civilization_III)
- [ProtonDB: App 3910](https://www.protondb.com/app/3910)
