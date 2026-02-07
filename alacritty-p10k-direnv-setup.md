# Alacritty + Powerlevel10k + Direnv Configuration

**Date:** 2026-02-07
**Tags:** `alacritty`, `zsh`, `powerlevel10k`, `direnv`, `terminal`

## Problem

When using Alacritty with Powerlevel10k's instant prompt feature and direnv, you get warnings:

```
[WARNING]: Console output during zsh initialization detected.
direnv: loading ~/project/.envrc
direnv: export +TOKEN
```

This happens because direnv produces console output during shell initialization, which conflicts with P10k's instant prompt optimization.

## Solution

Disable P10k instant prompt entirely in `~/.zshrc` (before the instant prompt block):

```zsh
typeset -g POWERLEVEL9K_INSTANT_PROMPT=off
```

The `quiet` mode (which suppresses warnings) wasn't sufficient -- direnv and other tools like `mise` that produce output during init still caused issues. Disabling instant prompt entirely was the most reliable fix.

## Why This Works

- P10k's instant prompt caches the prompt and shows it immediately (~40ms startup), but conflicts with **any** console output during shell init
- `quiet` mode only suppresses the warnings -- it doesn't fix underlying timing issues with tools like direnv and mise
- Setting `off` disables instant prompt entirely, avoiding all conflicts at the cost of slightly slower prompt rendering (negligible in practice)

## Bonus: Alacritty QoL Improvements

### Config location: `~/.config/alacritty/alacritty.toml`

**Clickable URLs and file paths:**
```toml
[hints]
enabled = [
  { regex = "(ipfs:|ipns:|magnet:|mailto:|gemini:|gopher:|https:|http:|news:|file:|git:|ssh:|ftp:)[^\u0000-\u001f\u007f-\u009f<>\"\\s{-}\\^⟨⟩`]+", command = "xdg-open", post_processing = true, mouse = { enabled = true, mods = "Control" } },
  { regex = "(/[a-zA-Z0-9._/-]+)", command = "xdg-open", post_processing = true, mouse = { enabled = true, mods = "Control" } },
]
```

**Disable audio bell:**
```toml
[bell]
duration = 0
```

**Better padding:**
```toml
[window.padding]
x = 8
y = 8
```

**Spawn new window keybinding:**
```toml
[[keyboard.bindings]]
key = "N"
mods = "Control|Shift"
action = "SpawnNewInstance"
```

**Performance optimizations:**
```toml
[scrolling]
history = 50000  # More scrollback on modern systems

[window]
opacity = 0.8  # Or 1.0 for max performance
```

## Key Takeaways

- Alacritty is already GPU-accelerated by default (OpenGL)
- P10k instant prompt conflicts with any console output during init
- If you use direnv, mise, or similar tools that produce output during init, just disable instant prompt (`off`) -- `quiet` mode isn't enough
- Ctrl+Click URLs/paths is a game-changer for terminal workflow

## References

- [Powerlevel10k Instant Prompt Docs](https://github.com/romkatv/powerlevel10k#instant-prompt)
- [Alacritty Configuration](https://alacritty.org/config-alacritty.html)
