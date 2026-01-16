# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a ZMK firmware configuration for the Cornix keyboard - a split ergonomic keyboard with 52 keys (54-key layout). The repository uses the Zephyr RTOS build system and West manifest for dependency management.

## Architecture

### Build System

- **West Manifest**: `config/west.yml` defines all dependencies including:
  - ZMK firmware (pinned to revision `e34793e`)
  - Third-party modules: zmk-helpers, zmk-rgbled-widget, zmk-dongle-display, zmk-dongle-screen, prospector-zmk-module
- **Build Configuration**: `build.yaml` defines all firmware variants to build
- **Module Definition**: `zephyr/module.yml` marks this as a Zephyr module with board/snippet roots
- **GitHub Actions**: `.github/workflows/build.yml` automatically builds firmware on push/PR using ZMK's reusable workflow

### Board Definitions

Located in `boards/arm/cornix/`:
- `cornix_left.dts` - Left half (peripheral)
- `cornix_ph_left.dts` - Left half with placeholder dongle support
- `cornix_right.dts` - Right half (central)
- `cornix_dongle.dts` - Dongle variant
- `cornix.dtsi` - Common board definition (nRF52840-based, WS2812 LED support, key scanning)
- `cornix_sensors.dtsi` - Sensor configurations
- `nrf_e73.dtsi` - E73-2G4M08S1C module specifics
- `cornix-pinctrl.dtsi` - Pin control definitions
- `cornix-layouts.dtsi` - Physical layout definitions

### Shield Definitions

Located in `boards/shields/`:
- `cornix_indicator/` - LED indicator shield
- `cornix_dongle_eyelash/` - Dongle display variant
- `cornix_yads/` - YADS dongle with screen support
- `cornix_prospector/` - Prospector ZMK module integration

### Keymap Structure

Primary keymap: `config/cornix.keymap`
- Uses zmk-helpers for behavior definitions
- Includes modular components from `config/includes/`:
  - `cornix54.h` - Key position matrix mapping (52-key layout definition)
  - `combos.dtsi` - Key combination definitions
  - `hrm.dtsi` - Home row mod behaviors
  - `mouse.dtsi` - Mouse/pointing device behaviors
- Defines 10 layers: BASE, WIN, LOWER, RAISE, ADJUST, NAVI, NUM, DEBUG, APPS, SELECTION
- Implements "timeless homerow mods" pattern (urob/zmk-config style)
- Custom timing: HM_TAPPING_TERM=250ms, HM_TAPPING_TERM_FAST=200ms

Alternative keymaps:
- `cornix-hrm.keymap` - Home row mod variant
- `cornix.editor.keymap` - Editor-focused layout

## Common Commands

### Building Firmware

Firmware is built automatically via GitHub Actions. To trigger a manual build:
```bash
git push  # Triggers workflow on push
```

Or manually trigger:
```bash
gh workflow run build.yml
```

### Build Artifacts

The build produces these firmware files (defined in `build.yaml`):
- `cornix_left_default` - Standard left half
- `cornix_left_for_dongle` - Left half with dongle support
- `cornix_right` - Right half
- `reset` - Settings reset firmware
- `yads_dongle` - YADS dongle with ZMK Studio
- `prospector_dongle` - Prospector dongle with ZMK Studio
- `dongle_reset` - Dongle reset firmware

### Working with Dependencies

Dependencies are managed via West:
```bash
west update  # Update all dependencies from west.yml
west list    # List all projects
```

## Key Implementation Details

### Hardware Platform
- MCU: nRF52840 (E73-2G4M08S1C module)
- Layout: Split keyboard, 52 keys total
- Features: RGB underglow (WS2812), split connectivity, optional dongle displays

### Keymap Position Mapping
The `cornix54.h` file defines the critical position-to-key mapping. Key positions are named using a grid system:
- L/R: Left/Right hand
- T/M/B: Top/Middle/Bottom row
- H: Hand/thumb cluster
- P: Palm keys
- Numbers: Position within row (0-5 for main keys)

Example: `LT0` = Left Top row, position 0 (leftmost)

### Configuration Files
- `config/cornix.conf` - Main configuration options
- `boards/arm/cornix/cornix.conf` - Board-specific configs
- Shield-specific configs in respective shield directories

### ZMK Studio Support
Enabled for dongle builds via:
- `snippet: studio-rpc-usb-uart`
- `cmake-args: -DCONFIG_ZMK_STUDIO=y`

## Important Notes

- ZMK revision is **pinned** to `e34793e` for stability
- When modifying keymaps, respect the position mapping in `cornix54.h`
- Shield overlays must be compatible with base board definitions
- Settings reset firmware is available for both keyboard and dongle variants
