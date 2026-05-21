# Ferris Sweep ZMK Config

Personal ZMK firmware for a **Ferris Sweep** — a 34-key wireless split keyboard.
- Hardware: nice!nano v2 on both halves (nRF52840, BLE)
- Shield: `cradio` (left + right)
- ZMK branch: `v0.3-branch` (see `config/west.yml`)
- Builds via GitHub Actions → produces UF2 files for flashing
- **OS Target:** macOS (uses Command for shortcuts; see `pc-layout` tag for Windows/Linux version)

## Key files

| File | Purpose |
|------|---------|
| `config/cradio.keymap` | Full keymap in ZMK devicetree syntax |
| `config/cradio.conf` | Kconfig options (BLE, features) |
| `config/west.yml` | ZMK version / manifest |
| `build.yaml` | Shields built by CI: `cradio_left`, `cradio_right`, `settings_reset` |
| `keymap.yaml` | Parsed keymap for visualization (generated, do not edit manually) |
| `keymap-config.yaml` | Config for keymap-drawer visualization tool |
| `visualization.svg/png` | Generated keymap image used in README |

## Layer structure

| Layer | Activation | Notes |
|-------|-----------|-------|
| Default | — | QWERTY; ñ on right pinky (macro); left thumb = NAV hold, right thumb = SYM hold |
| NAV (1) | Hold left thumb | Arrows, page up/dn, clipboard (Cmd+Z/X/C/V, Cmd+Shift+Z redo), Cmd+Tab/Cmd+Shift+Tab, Cmd+W, media, sticky mods |
| SYM (2) | Hold right thumb | Symbols, brackets; dead acute Option+E (´) for Spanish accents (á, é, í, ó, ú); sticky mods |
| NUM (3) | Hold both thumbs | Numbers (odd left / even right), F-keys, sticky mods |
| Bluetooth (4) | O+P combo | BT profile select (0–3) and BT_CLR_ALL |
| Mouse (5) | Z+? combo | Mouse movement, scroll, left/right click |

## Combos

| Keys (positions) | Output |
|-----------------|--------|
| Q+W (0,1) | Escape |
| Z+X (20,21) | Enter (left hand) |
| .+? (28,29) | Enter (right hand) |
| D+F (12,13) | Backspace (left hand) |
| J+K (16,17) | Backspace (right hand) |
| O+P (8,9) | Bluetooth layer (momentary) |
| Z+? (20,29) | Mouse layer (momentary) |

## Macros

- `Luthiers` — types `Luthiers1.*`
- `Luthiersvictor` — types `Luthiers1.*victor`
- `ntilde` — types ñ (macOS: Option+N, then N)

## Visualization workflow

After editing the keymap, regenerate the image:

```bash
keymap parse -c 34 -z config/cradio.keymap > keymap.yaml
sed -i '' 's/AltGr+N/⌥+N/g' keymap.yaml
sed -i '' 's/AltGr+E/⌥+E/g' keymap.yaml
keymap -c keymap-config.yaml draw keymap.yaml > visualization.svg
rsvg-convert -o visualization.png visualization.svg
```

Requires `keymap-drawer` (`pipx install keymap-drawer`) and `rsvg-convert` (`librsvg`).

Note: `sed` commands replace PC "AltGr" terminology with macOS "⌥" (Option) symbol.

## Known issues & fixes

### Right half random disconnections (fixed in commit 8d429c1)

**Symptom:** The right half (peripheral) drops its BLE connection to the left half (central) randomly after a short time.

**Root cause:** ZMK's default BLE settings use aggressive connection interval negotiation and 2M PHY, which is less stable for the short inter-half link. Additionally, the central repeatedly polls the peripheral's battery level, which can trigger reconnect loops.

**Fix applied in `config/cradio.conf`:**
```
CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y          # enables planned-default stability improvements, disables 2M PHY
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y             # max TX power (+8 dBm)
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=n  # stops central from polling right-half battery
```

**References:** [ZMK issue #916](https://github.com/zmkfirmware/zmk/issues/916), [ZMK BLE Kconfig docs](https://zmk.dev/docs/config/bluetooth#kconfig)

**If still disconnecting:** Try `CONFIG_ZMK_BLE_EXPERIMENTAL_FEATURES=y` (enables stricter pairing security — requires re-pairing all hosts).

### macOS conversion (tag: pc-layout)

Converted all shortcuts from Control-based (PC/Linux) to Command-based (macOS). Key changes:
- Clipboard operations: Cmd+Z/X/C/V instead of Ctrl+Z/X/C/V
- Redo: Cmd+Shift+Z instead of Ctrl+Y
- Window switching: Cmd+Tab/Cmd+Shift+Tab instead of Ctrl+Tab/Ctrl+Shift+Tab
- Close window: Cmd+W instead of Ctrl+W

**Spanish character support (macOS US International):**
- ñ: Macro sends Option+N, then N (macOS dead key sequence)
- Acute accents (á, é, í, ó, ú): Changed from Option+' to Option+E (macOS dead acute key)

**To revert to PC layout:** `git checkout pc-layout` or create a branch from that tag.
