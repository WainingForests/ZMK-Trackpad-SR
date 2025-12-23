# ZMK Trackpad SR - Custom Keyboard with Azoteq TPS65

This ZMK firmware repository is configured for a custom split keyboard with shift register matrix and Azoteq TPS65 capacitive trackpad.

## Hardware Configuration

- **Controller**: Seeeduino XIAO BLE (nRF52840)
- **Matrix**: 6x16 with shift register (74HC595)
- **Trackpad**: Azoteq TPS65 (IQS5xx chipset)
- **Encoders**: 3x EC11 rotary encoders
- **Underglow**: WS2812 RGB LEDs (84 LEDs)

## Pin Configuration

### TPS65 Trackpad (I2C1)
Based on the same pins previously used by PMW3610:
- **I2C SDA**: P0.19
- **I2C SCL**: P0.15
- **Reset GPIO**: P0.02 (Active LOW)
- **Ready GPIO**: P0.29 (Active HIGH)
- **I2C Address**: 0x74

### Shift Register (SPI0)
- **SCK**: P1.03
- **MOSI**: P1.05
- **CS**: P1.13

### RGB Underglow (SPI3)
- **MOSI**: P1.14

## ZMK Version

This repository uses **ZMK v0.3**. The zmk-driver-azoteq-iqs5xx was created August 9, 2025, targeting v0.3 (released August 1, 2025). ZMK main branch has breaking changes from Zephyr 4.1 update (December 2025) that break the driver.

## Building

The firmware is built automatically via GitHub Actions when you push to this repository. The workflow uses ZMK v0.3.

To build locally:
```bash
west build -b seeeduino_xiao_ble -- -DSHIELD=skreecustom_left
west build -b seeeduino_xiao_ble -- -DSHIELD=skreecustom_right
```

## TPS65 Trackpad Features

Both trackpads are configured with the following gesture support:

### Gesture Recognition
- **One-finger tap**: Single finger tap detected as left click
- **Two-finger tap**: Two finger tap detected as right click
- **Press-and-hold**: Sustained pressure (250ms threshold) detected as left click until released

### Scroll Support
- **Scroll**: Enabled for both vertical and horizontal scrolling
- **Natural scroll**: Content follows finger movement on both X and Y axes

### Tuning Parameters
- **Bottom-beta**: 5 (filtering intensity for smoothness vs responsiveness)
- **Stationary-threshold**: 5 pixels (movement required to exit stationary state)

### Configuration Notes
- The trackpad on the **right** side is active by default
- The trackpad on the **left** side is disabled by default (change `status = "disabled"` to `status = "okay"` to enable)
- All gesture features can be customized in the overlay files
- Additional options available: `switch-xy`, `flip-x`, `flip-y` for axis adjustments

## Files Structure

```
.
├── .github/workflows/
│   └── build.yml          # GitHub Actions workflow
├── boards/shields/skreecustom/
│   ├── boards/
│   │   └── seeeduino_xiao_ble.overlay  # Board-specific pin config
│   ├── Kconfig.defconfig  # Default configurations
│   ├── Kconfig.shield     # Shield definitions
│   ├── skreecustom.conf   # Common config
│   ├── skreecustom.dtsi   # Common device tree
│   ├── skreecustom_left.conf
│   ├── skreecustom_left.overlay
│   ├── skreecustom_right.conf
│   └── skreecustom_right.overlay
├── config/
│   ├── skreecustom.keymap # Keymap configuration
│   └── west.yml           # ZMK v0.3 + TPS65 driver
└── build.yaml             # Build matrix
```

## Credits

Based on [WainingForests/ZMK-XIAOv4-SR](https://github.com/WainingForests/ZMK-XIAOv4-SR)

Azoteq TPS65 driver: [AYM1607/zmk-driver-azoteq-iqs5xx](https://github.com/AYM1607/zmk-driver-azoteq-iqs5xx)
