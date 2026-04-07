---
author: ["Potato Energy Team", "ponfertato"]
categories: ["linux", "gaming", "steam", "guide"]
date: "2026-04-07T10:20:00+03:00"
description: "steamscope.sh: unified launcher for Steam games on Linux with engine, GPU, and system optimizations."
draft: false
series: ["Linux Gaming"]
slug: "steamscope"
tags: ["steam", "proton", "gamescope", "gamemode", "linux-gaming", "bash"]
title: "steamscope.sh: Universal Steam Launcher for Linux"
---

## 🎮 Why this matters

Running games via Steam on Linux often feels like "dancing with a tambourine": different engines need different flags, AMD and NVIDIA require different environment variables, and tools like Gamescope and MangoHud must be manually inserted into launch commands.

**Solution**: `steamscope.sh` - a wrapper script that:

- ✅ Auto-detects GPU (AMD/NVIDIA) and applies appropriate optimizations
- ✅ Supports engine-specific flags: Source, Unreal, Unity
- ✅ Integrates Gamescope, Gamemode, MangoHud, FSR in one command
- ✅ Works as `%command%` in Steam launch options
- ✅ Doesn't break standard launch - everything is optional

> 💡 The script doesn't replace Proton or Steam - it makes their work predictable and configurable.

---

## 📦 Installation

### Step 1: Download the script

```bash
# Create directory (if missing)
mkdir -p ~/.steam/steam/

# Download script
curl -o ~/.steam/steam/steamscope.sh \
  https://gist.githubusercontent.com/ponfertato/85fc964664423eae1ac0a7ee91d53db6/raw/steamscope.sh

# Make executable
chmod +x ~/.steam/steam/steamscope.sh
```

### Step 2: Configure game in Steam

1. Right-click game → **Properties**
2. In **Launch Options**, add:
   ```bash
   ~/.steam/steam/steamscope.sh %command%
   ```
3. (Optional) Add flags:
   ```bash
   ~/.steam/steam/steamscope.sh --gamemode --mangohud --engine=source %command%
   ```

**Done**. When launching the game, the script will automatically apply settings.

---

## ⚙️ Launch parameters (by category)

### 🚀 Performance

| Flag         | Description                                           | For whom                             |
| ------------ | ----------------------------------------------------- | ------------------------------------ |
| `--gamemode` | Enables `gamemoderun` for process priority            | Everyone, especially on weak systems |
| `--fsr`      | Enables FSR upscaling via VKD3D (AMD) / DXVK (NVIDIA) | AMD GPU, low native FPS              |
| `--dxvk`     | Forces DXVK instead of D9VK                           | Games with DirectX 9/10/11           |

### 👁 Monitoring

| Flag         | Description                                 | Example output                   |
| ------------ | ------------------------------------------- | -------------------------------- |
| `--mangohud` | Shows overlay with FPS, CPU/GPU load, temps | `FPS: 87 │ CPU: 45% │ GPU: 72°C` |

### 🖥 Runtime mode

| Flag                    | Description                                         | Why                                |
| ----------------------- | --------------------------------------------------- | ---------------------------------- |
| `--gamescope`           | Runs game in isolated Wayland session via Gamescope | Stable FPS, adaptive sync, scaling |
| `--resolution=1280x720` | Sets resolution for Gamescope (default: native)     | Testing, reducing load             |

### 🎯 Engines

| Flag              | Supported engines   | Example games                                    |
| ----------------- | ------------------- | ------------------------------------------------ |
| `--engine=source` | Source 1/2          | CS2, TF2, Garry's Mod, Left 4 Dead 2             |
| `--engine=unreal` | Unreal Engine 3/4/5 | Risk of Rain 2, Deep Rock Galactic, Satisfactory |
| `--engine=unity`  | Unity               | 7 Days to Die, Valheim, Among Us                 |

**What each engine adds**:

- **Source**: `-high -threads N +mat_vsync 0 +fps_max 0 +exec autoexec.cfg`
- **Unreal**: `-dx12 -nomansky -notexturestreaming -nomovie`
- **Unity**: `-screen-width W -screen-height H -nolog -batchmode`

---

## 🔧 How it works (under the hood)

### Auto-detect hardware

```bash
# Script checks GPU via lspci
GPU_INFO=$(lspci -k 2>/dev/null | awk '/VGA|3D/,/Kernel driver/ {if (/Kernel driver in use/) print $NF}' | tail -1)

# For AMD:
[[ "$GPU_INFO" == *"amdgpu"* ]] && USE_AMD=true
# → Adds: RADV_PERFTEST=aco, AMD_DEBUG=nodcc

# For NVIDIA:
[[ "$GPU_INFO" == *"nvidia"* ]] && USE_NVIDIA=true
# → Adds: PROTON_ENABLE_NVAPI=1, __NV_PRIME_RENDER_OFFLOAD=1
```

### Building the final command

The script builds the command in "layers":

```
[gamemoderun] → [mangohud] → [gamescope -- ...] → [GAME + engine flags]
```

Each layer is added only if the corresponding flag is passed.

### Handling `%command%`

Steam passes the launch command as `%command%`. The script:

1. Finds the position after the executable (but not inside `proton`, `steam-runtime`, etc.)
2. Inserts engine flags **after** the executable but **before** game arguments
3. This ensures flags reach the game, not the launcher

---

## 🎯 Examples for popular games

### Counter-Strike 2 (Source Engine, AMD)

```bash
~/.steam/steam/steamscope.sh \
  --engine=source \
  --gamemode \
  --mangohud \
  --fsr \
  --gamescope \
  %command%
```

**What the game gets**:

- Process priority via `gamemoderun`
- MangoHud overlay with metrics
- FSR upscaling via VKD3D (for higher FPS)
- Launch in Gamescope with adaptive sync
- Optimized Source flags: `-high -threads 8 +fps_max 0`

### Deep Rock Galactic (Unreal Engine, NVIDIA)

```bash
~/.steam/steam/steamscope.sh \
  --engine=unreal \
  --gamemode \
  --dxvk \
  --mangohud \
  %command%
```

**Highlights**:

- `--dxvk` enables DXVK for better Vulkan compatibility on NVIDIA
- Unreal flags disable heavy effects: `-nomansky -notexturestreaming`
- `UE5_ALLOW_LINUX_DEBUGGING=1` for Linux stability

### 7 Days to Die (Unity, cross-platform)

```bash
~/.steam/steam/steamscope.sh \
  --engine=unity \
  --gamemode \
  --fsr \
  --resolution=1280x720 \
  %command%
```

**Why**:

- `--resolution` reduces load for smoother gameplay
- `--fsr` compensates detail loss with upscaling
- Unity flags disable logging: `-nolog -batchmode`

---

## 🛠 Configuration tips

### Start minimal

```bash
# Game only (basic launch)
~/.steam/steam/steamscope.sh %command%

# + monitoring
~/.steam/steam/steamscope.sh --mangohud %command%

# + optimization
~/.steam/steam/steamscope.sh --gamemode --mangohud %command%
```

Add parameters one by one - easier to find what breaks.

### If the game crashes

1. **Disable `--gamescope`** - most common source of conflicts
2. **Disable `--gamemode`** - may conflict with system settings
3. **Check logs**:
   ```bash
   journalctl -f | grep -i steam
   ```

### Resolution and scaling

- By default, script gets native resolution via `xrandr`
- To force change: `--resolution=1280x720`
- Combined with `--gamescope` and `--fsr` gives flexible quality/performance control

### Localization

The script sets:

```bash
export LANG="ru_RU.UTF-8"
export LC_ALL="ru_RU.UTF-8"
```

If game requires English locale - override in launch options:

```bash
LANG=en_US.UTF-8 ~/.steam/steam/steamscope.sh %command%
```

---

## ⚠️ Common issues

```bash
# "gamescope: command not found"
→ Install:
  # Arch: sudo pacman -S gamescope
  # Debian/Ubuntu: sudo apt install gamescope
  # Fedora: sudo dnf install gamescope

# "gamemoderun: command not found"
→ Install:
  # All distros: sudo apt install gamemode or equivalent

# Game ignores engine flags
→ Ensure %command% is at the end of the launch string
→ Verify the game actually uses the expected engine (not all "names" match reality)

# No MangoHud overlay
→ Ensure mangohud is installed and in PATH
→ Some games require: export MANGOHUD_DLSYM=1 (script does this automatically)

# Low FPS after enabling --fsr
→ FSR is upscaling, not magic: it boosts FPS at the cost of quality
→ Try lowering --resolution together with --fsr

# Script doesn't detect GPU
→ Ensure pciutils is installed: sudo apt install pciutils
→ Check output: lspci -k | grep -A2 -i vga
```

---

## 🔐 Security and privacy

The script **does not collect data**, send anything over the network, or work outside the local system.

**What it does**:

- Reads `lspci` to detect GPU (for optimizations only)
- Sets environment variables for the game process
- Does not modify game or system files

**Recommendations**:

- Download script only from [official gist](https://gist.github.com/ponfertato/85fc964664423eae1ac0a7ee91d53db6)
- Verify hash when updating (optional)
- Don't pass sensitive data in script arguments

---

## 🗂 Pre-use checklist

- [ ] Installed dependencies: `gamemode`, `mangohud`, `gamescope` (optional)
- [ ] Downloaded script to `~/.steam/steam/steamscope.sh` and made executable
- [ ] Tested launch of one game with minimal flags
- [ ] Verified `%command%` is at the end of launch options string
- [ ] Reviewed [local regulations](#) on overlay usage in games (if relevant)

---

## Links

- 🎮 [Official script Gist](https://gist.github.com/ponfertato/85fc964664423eae1ac0a7ee91d53db6)
- 🐧 [ProtonDB - game compatibility](https://www.protondb.com)
- 🖥 [Gamescope on GitHub](https://github.com/ValveSoftware/gamescope)
- ⚡ [Gamemode on GitHub](https://github.com/FeralInteractive/gamemode)
- 👁 [MangoHud on GitHub](https://github.com/flightlessmango/MangoHud)
- 🎯 [FSR in VKD3D-Proton](https://github.com/HansKristian-Work/vkd3d-proton)
