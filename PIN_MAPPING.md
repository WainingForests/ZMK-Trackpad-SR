# Pin Mapping: PMW3610 to TPS65 Conversion

## Original PMW3610 Configuration (SPI)

The PMW3610 optical sensor used SPI1 with the following pins:

| Function | Pin | Notes |
|----------|-----|-------|
| SPI SCK  | P0.19 | SPI clock |
| SPI MOSI | P0.15 | SPI data out |
| SPI MISO | P0.15 | SPI data in (same as MOSI) |
| CS (Chip Select) | P0.02 | Active LOW |
| IRQ (Motion) | P0.29 | Active LOW with pull-up |

**spi-max-frequency**: 2000000 Hz
**CPI**: 600

## New TPS65 Configuration (I2C)

The Azoteq TPS65 capacitive trackpad uses I2C1 with the same pins repurposed:

| Function | Pin | Notes |
|----------|-----|-------|
| I2C SDA  | P0.19 | I2C data line |
| I2C SCL  | P0.15 | I2C clock line |
| Reset GPIO | P0.02 | Active LOW (was CS) |
| Ready GPIO | P0.29 | Active HIGH (was IRQ) |

**I2C Address**: 0x74
**Compatible**: `azoteq,iqs5xx`

## Key Differences

1. **Protocol Change**: SPI → I2C
   - P0.19: SPI_SCK → I2C_SDA
   - P0.15: SPI_MOSI/MISO → I2C_SCL

2. **Control Pins**:
   - P0.02: Chip Select → Reset (both active LOW)
   - P0.29: Motion IRQ (active LOW) → Ready signal (active HIGH)

3. **Driver Requirements**:
   - PMW3610: Built into ZMK main
   - TPS65: Requires external module (`zmk-driver-azoteq-iqs5xx`)

## Kconfig Changes

### Removed
```kconfig
config PMW3610
    default y

config PMW3610_SWAP_XY
    default y

config PMW3610_REPORT_INTERVAL_MIN
    default 8
```

### Added
```kconfig
config I2C
    default y

config IQS5XX
    default y
```

## Device Tree Node Comparison

### PMW3610 (Old)
```dts
&spi1 {
    cs-gpios = <&gpio0 2 GPIO_ACTIVE_LOW>;

    trackball_peripheral: trackball_peripheral@0 {
        compatible = "pixart,pmw3610";
        reg = <0>;
        spi-max-frequency = <2000000>;
        irq-gpios = <&gpio0 29 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
        cpi = <600>;
    };
};
```

### TPS65 (New)
```dts
&i2c1 {
    trackpad_peripheral: trackpad_peripheral@74 {
        compatible = "azoteq,iqs5xx";
        reg = <0x74>;
        reset-gpios = <&gpio0 2 GPIO_ACTIVE_LOW>;
        rdy-gpios = <&gpio0 29 GPIO_ACTIVE_HIGH>;

        // Gesture support
        one-finger-tap;
        two-finger-tap;
        press-and-hold;
        press-and-hold-time = <250>;

        // Scroll support
        scroll;
        natural-scroll-y;
        natural-scroll-x;

        // Tuning parameters
        bottom-beta = <5>;
        stationary-threshold = <5>;

        evt-type = <INPUT_EV_REL>;
        x-input-code = <INPUT_REL_X>;
        y-input-code = <INPUT_REL_Y>;
    };
};
```

## TPS65 Gesture Features

The TPS65 configuration includes advanced gesture recognition:

### Tap Gestures
- **one-finger-tap**: Detected as left mouse button click
- **two-finger-tap**: Detected as right mouse button click
- **press-and-hold**: Sustained pressure (250ms threshold) detected as left click until released

### Scroll Configuration
- **scroll**: Enables bidirectional scroll gesture recognition
- **natural-scroll-x**: Content follows finger movement horizontally
- **natural-scroll-y**: Content follows finger movement vertically

### Tuning Parameters
- **bottom-beta** (0-255): Filtering intensity
  - Lower values = more filtering = smoother but laggier
  - Higher values = less filtering = snappier tracking
  - Default: 5
- **stationary-threshold**: Pixel movement required to exit stationary state
  - Default: 5 pixels

### Additional Available Options (not enabled by default)
- **switch-xy**: Transpose x and y coordinates
- **flip-x**: Reverse x-axis direction
- **flip-y**: Reverse y-axis direction

## Input Processor Changes

### Removed (Trackball-specific)
All input processors from the original PMW3610 trackball configuration have been removed:
- `zip_xy_transform` (inversion transforms)
- `zip_xy_to_scroll_mapper` (scroll conversion)
- `zip_scroll_transform` (scroll axis manipulation)
- `zip_scroll_scaler` (scroll speed adjustment)
- `zip_temp_layer` (temporary layer activation)

The TPS65 driver handles gestures and scrolling natively, so no external input processors are needed.

## Notes

- The TPS65 trackpad provides capacitive touch tracking instead of optical tracking
- No CPI setting needed for capacitive trackpad
- The ready GPIO polarity changed from ACTIVE_LOW to ACTIVE_HIGH
- Both devices use relative input events (INPUT_EV_REL)
- Gesture recognition is built into the TPS65 driver, eliminating the need for input processors
