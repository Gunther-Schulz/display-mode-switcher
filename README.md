# ddc-mode-switcher

Switch dual-mode monitor display modes (e.g. 5K desktop ↔ 2K high-refresh gaming) via DDC/CI. Use as a **toggle** or as a **wrapper** for Steam/games. Works with any dual-mode monitor through a simple config file.

## Requirements

- `ddcutil` — DDC/CI communication (any compositor)
- `i2c_dev` kernel module loaded
- **Optional:** `gdctl` (Mutter 48+) — enables verified mode switching and automatic res/scale restore. Without it, the script uses blind waits and skips restore.

## Quick start

1. Find your monitor's I2C bus and DDC toggle commands:
   ```bash
   ddcutil detect          # find the bus number
   ddcutil capabilities    # find VCP codes for mode switching
   gdctl show              # find connector names, current mode, scale
   ```

2. Create a config file:
   ```bash
   mkdir -p ~/.config/ddc-mode-switcher
   cp config.example ~/.config/ddc-mode-switcher/config
   # Edit with your values
   ```

3. Test:
   ```bash
   ddc-mode-switcher       # toggle mode
   ```

## Usage

**Toggle mode:**
```bash
ddc-mode-switcher
```

**Wrapper** (switch for game, restore on exit):
```bash
ddc-mode-switcher mangohud %command%
```

**Steam launch options:**
```
ddc-mode-switcher mangohud %command%
```

## Configuration

Create `~/.config/ddc-mode-switcher/config` (see `config.example` for a fully commented template):

```bash
BUS=5                              # I2C bus number
CONN=DP-2                          # Monitor connector name
NATIVE_RES=5120x2880               # Desktop resolution to restore to
TOGGLE_STEPS=("0x03 1" "0x03 20")  # DDC VCP toggle sequence
```

### Optional settings

| Variable | Default | Description |
|----------|---------|-------------|
| `SECONDARY_CONN` | *(none)* | Secondary monitor connector for multi-monitor restore |
| `DEFAULT_SCALE` | 1.0 | Fallback primary scale if gdctl can't read it |
| `DEFAULT_SECONDARY_SCALE` | 1.0 | Fallback secondary scale |
| `SWITCH_TIMEOUT` | 10 | Max seconds to poll gdctl for mode confirmation |
| `FALLBACK_WAIT` | 2 | Blind wait (seconds) when gdctl is unavailable |

### Environment overrides

| Variable | Description |
|----------|-------------|
| `DISPLAY_MODE_SWITCHER_CONFIG` | Config file path (overrides default location) |
| `DISPLAY_MODE_SWITCH_TIMEOUT` | Override `SWITCH_TIMEOUT` from config |

## How it works

1. **Toggle mode:** Sends the DDC VCP commands defined in `TOGGLE_STEPS` to switch the monitor between its two modes.

2. **Wrapper mode:**
   - Saves the current display state (mode + scale) via `gdctl` if available
   - If the current resolution doesn't match `NATIVE_RES`, assumes already in alternate mode and runs the command directly
   - Sends DDC toggle, then polls `gdctl show` until the resolution changes (confirmed switch) or times out
   - Runs the wrapped command
   - Sends DDC toggle again, polls until `NATIVE_RES` is confirmed
   - Restores the exact saved mode and scale via `gdctl`

3. **Without gdctl** (non-GNOME compositors): toggle still works, wrapper uses a blind wait (`FALLBACK_WAIT`) instead of polling, and skips the restore step.

## Example: ASUS ROG Strix XG27JCG

This monitor has a "Frame Rate Boost" feature: 5K (5120x2880 @180Hz) ↔ 2K (2560x1440 @330Hz), toggled via DDC VCP 0x03 soft controls (value 1 opens the OSD menu, value 20 confirms).

```bash
BUS=5
CONN=DP-2
NATIVE_RES=5120x2880
TOGGLE_STEPS=("0x03 1" "0x03 20")
SECONDARY_CONN=DP-3
DEFAULT_SCALE=1.66
```

## Installation

**From AUR** (package name: `ddc-mode-switcher`):
```bash
yay -S ddc-mode-switcher
```

**From source:**
```bash
git clone https://github.com/Gunther-Schulz/display-mode-switcher.git
cd display-mode-switcher
sudo install -Dm755 ddc-mode-switcher /usr/bin/ddc-mode-switcher
```

## License

MIT
