# Changelog

## v2.0 - TPS65 Trackpad Configuration Update

### Changes from Original WainingForests Repository

#### Hardware Changes
- **Replaced**: PMW3610 optical trackball sensor (SPI)
- **With**: Azoteq TPS65 capacitive trackpad (I2C)
- **Pins**: Reused same physical pins (P0.19, P0.15, P0.02, P0.29)

#### Configuration Changes

##### Removed Features
- All PMW3610-specific configuration
- All trackball input processors:
  - `zip_xy_transform` (coordinate inversion)
  - `zip_xy_to_scroll_mapper` (scroll conversion)
  - `zip_scroll_transform` (scroll axis manipulation)
  - `zip_scroll_scaler` (scroll speed adjustment)
  - `zip_temp_layer` (temporary layer activation)
- Input processor includes (`<input/processors.dtsi>`)
- Input transform includes (`<dt-bindings/zmk/input_transform.h>`)

##### Added Features
- **TPS65 Gesture Recognition**:
  - One-finger tap → Left click
  - Two-finger tap → Right click
  - Press-and-hold (250ms threshold) → Left click until released

- **Native Scroll Support**:
  - Bidirectional scrolling (vertical and horizontal)
  - Natural scroll on both axes (content follows finger)

- **Tuning Parameters**:
  - bottom-beta: 5 (filtering intensity)
  - stationary-threshold: 5 pixels

- **I2C Configuration**:
  - I2C bus enabled on I2C1
  - TPS65 at address 0x74
  - Reset and ready GPIOs configured

##### Kconfig Updates
- Removed: `PMW3610`, `PMW3610_SWAP_XY`, `PMW3610_REPORT_INTERVAL_MIN`
- Added: `I2C`, `IQS5XX`

##### ZMK Version
- Pinned to **v0.3** (required for TPS65 driver compatibility)

#### Driver Integration
- Added `zmk-driver-azoteq-iqs5xx` module via `config/west.yml`
- Module source: https://github.com/AYM1607/zmk-driver-azoteq-iqs5xx

#### Files Modified
- `.github/workflows/build.yml` - Updated to use ZMK v0.3
- `boards/shields/skreecustom/boards/seeeduino_xiao_ble.overlay` - Changed SPI1 to I2C1
- `boards/shields/skreecustom/Kconfig.defconfig` - Updated driver configs
- `boards/shields/skreecustom/skreecustom.dtsi` - Removed input processors
- `boards/shields/skreecustom/skreecustom_left.overlay` - Added TPS65 with gestures
- `boards/shields/skreecustom/skreecustom_right.overlay` - Added TPS65 with gestures
- `config/west.yml` - Created with ZMK v0.3 and TPS65 driver
- `build.yaml` - Copied from source

#### Documentation Added
- `README.md` - Full configuration and feature documentation
- `PIN_MAPPING.md` - Detailed pin conversion and comparison
- `CHANGELOG.md` - This file
- `.gitignore` - Build artifacts and editor files

## v1.0 - Original Configuration

Initial configuration from WainingForests/ZMK-XIAOv4-SR with PMW3610 trackball sensor.
