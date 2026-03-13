# display-mode-switcher

Switch dual-mode monitor display modes (e.g. 5K↔2K high-refresh) for desktop use vs gaming. Use as a **toggle** or as a **wrapper** for Steam/games. Binary: `ddc-mode-switcher`.

## Currently supported

- **ASUS ROG Strix XG27JCG** — 5K (5120×2880 @180Hz) ↔ 2K (2560×1440 @330Hz) "Frame Rate Boost"

Other dual-mode monitors may work with additional profiles. Contributions welcome.

## Requirements

- `ddcutil` — DDC/CI communication (any compositor)
- `i2c_dev` kernel module loaded
- **GNOME on Wayland** (for wrapper mode) — `gdctl` (Mutter 48+) restores res/scale when switching back to 5K (GNOME defaults to 4K otherwise). Toggle mode works on any compositor; wrapper's restore step is GNOME-specific.

## Usage

**Toggle mode** (high-res ↔ high-refresh):
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

## Environment

| Variable | Default | Description |
|----------|---------|-------------|
| `DISPLAY_MODE_SWITCHER_MONITOR` | xg27jcg | Monitor profile |
| `DISPLAY_MODE_SWITCH_DELAY` | 2 | Seconds to wait after switch before launching (wrapper only) |
| `XG27JCG_BUS` | 5 | I2C bus for XG27JCG |
| `XG27JCG_CONN` | DP-2 | Connector name (gdctl) |
| `XG27JCG_SECONDARY_CONN` | DP-3 | Secondary monitor connector |

## How it works (XG27JCG)

1. Uses DDC VCP 0x03 soft controls: value 1 opens menu, 20 confirms (toggles Frame Rate Boost)
2. When switching to 2K: compositor picks resolution automatically
3. When switching back to 5K: **GNOME** defaults to 4K → `gdctl` restores saved res/scale. On other compositors, restore is skipped.
4. Wrapper saves state (when gdctl available) before switch, restores after command exits

## Adding support for other monitors

The script is structured with per-monitor profiles. To add a monitor:

1. Add a `run_<profile>()` function with `do_toggle`, `do_restore` (if needed), and wrapper logic
2. Add the profile to the dispatch `case` in the main script
3. Document the profile in README and submit a PR

Profiles need: DDC bus, connector name(s), toggle command sequence, and optionally gdctl restore logic if the compositor misbehaves on mode switch.

## Installation

**From AUR** (package name: `ddc-mode-switcher`):
```bash
yay -S ddc-mode-switcher
```
Installs the `ddc-mode-switcher` binary.

**From source:**
```bash
git clone https://github.com/Gunther-Schulz/display-mode-switcher.git
cd display-mode-switcher
sudo install -Dm755 ddc-mode-switcher /usr/bin/ddc-mode-switcher
```

## License

MIT
