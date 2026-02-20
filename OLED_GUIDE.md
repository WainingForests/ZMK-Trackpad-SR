# ZMK OLED Display Guide for nRF52840 (Seeeduino XIAO BLE)

Tested and verified configuration for SSD1306 OLED displays on ZMK v0.3 with the Seeeduino XIAO BLE (nRF52840). This guide covers the critical pitfalls and working configurations discovered through hardware testing.

## nRF52840 Peripheral Instance Constraints

The nRF52840 has 4 serial peripheral instances. **SPI and I2C share hardware instances** — you cannot use both on the same instance simultaneously.

| Instance | SPI | I2C | Notes |
|----------|-----|-----|-------|
| 0 | SPI0/SPIM0 | TWI0/TWIM0 | **Mutually exclusive.** Pick SPI or I2C, not both. |
| 1 | SPI1/SPIM1 | TWI1/TWIM1 | **Mutually exclusive.** Pick SPI or I2C, not both. |
| 2 | SPI2/SPIM2 | None | SPI only. No I2C capability. |
| 3 | SPI3/SPIM3 | None | SPI only. High-speed capable. |

**Only instances 0 and 1 support I2C.** If you need two I2C buses (e.g., OLED + trackpad), they must be on instances 0 and 1, and neither instance can also run SPI.

If you also need SPI devices (shift registers, RGB LEDs), those must go on instances 2 and 3.

### Example Allocation (Skreepad)

| Instance | Peripheral | Device | Pins |
|----------|-----------|--------|------|
| 0 | I2C0 | SSD1306 OLED | P0.04 (SDA), P0.05 (SCL) |
| 1 | I2C1 | Azoteq IQS5xx Trackpad | P0.19 (SDA), P0.15 (SCL) |
| 2 | SPI2 | 74HC595 Shift Register | P1.03 (SCK), P1.05 (MOSI) |
| 3 | SPI3 | WS2812 RGB LEDs | P1.14 (MOSI) |

### Common Mistake: Instance Conflict

If you enable both `&spi0` and `&i2c0` (or both `&spi1` and `&i2c1`), the build will fail with:

```
error: "Only one of the following peripherals can be enabled: SPI0, SPIM0, SPIS0, TWI0, TWIM0, TWIS0."
```

**Fix:** Move one of the conflicting devices to a different instance. SPI devices can go on instances 2 or 3. I2C devices can only go on instances 0 or 1.

## SSD1306 OLED Configuration

### Supported Displays

| Size | Resolution | `multiplex-ratio` | `com-sequential` |
|------|-----------|-------------------|-----------------|
| 0.91" | 128x32 | `<31>` | **Required** |
| 0.96" | 128x64 | `<63>` | Do NOT include |

### Critical: I2C Driver Selection (TWI vs TWIM)

**Do NOT override `compatible` to `"nordic,nrf-twim"` on the I2C bus used for the OLED.**

The Seeeduino XIAO BLE board DTS sets I2C0 to `compatible = "nordic,nrf-twi"` by default. The TWI driver (non-DMA, polling) works correctly with the SSD1306. The TWIM driver (DMA-based) causes **garbled display output**.

```dts
/* CORRECT - let the board default "nordic,nrf-twi" apply */
&i2c0 {
    status = "okay";
    pinctrl-0 = <&i2c0_default>;
    pinctrl-1 = <&i2c0_sleep>;
    pinctrl-names = "default", "sleep";

    oled: ssd1306@3c { ... };
};

/* WRONG - overriding to TWIM causes garbled display */
&i2c0 {
    status = "okay";
    compatible = "nordic,nrf-twim";  /* DO NOT DO THIS */
    ...
};
```

Note: Other I2C devices (like the Azoteq trackpad) may work fine with TWIM. This issue is specific to the SSD1306 display driver.

### The `xiao_i2c` Alias

On the Seeeduino XIAO BLE, `&xiao_i2c` is just an alias for `&i2c0`. You can use either. The board DTS also defines `&xiao_spi` as an alias for `&spi2`.

### Working 128x32 Device Tree Configuration

```dts
oled: ssd1306@3c {
    compatible = "solomon,ssd1306fb";
    reg = <0x3c>;
    width = <128>;
    height = <32>;
    segment-offset = <0>;
    page-offset = <0>;
    display-offset = <0>;
    multiplex-ratio = <31>;
    segment-remap;
    com-invdir;
    com-sequential;
    prechargep = <0x22>;
    inversion-on;
};
```

#### Property Reference

| Property | 128x32 Value | Purpose |
|----------|-------------|---------|
| `compatible` | `"solomon,ssd1306fb"` | SSD1306 framebuffer driver |
| `reg` | `<0x3c>` | I2C address (some modules use `<0x3d>`) |
| `width` | `<128>` | Horizontal pixels |
| `height` | `<32>` | Vertical pixels |
| `segment-offset` | `<0>` | Column start offset in RAM |
| `page-offset` | `<0>` | Page start offset |
| `display-offset` | `<0>` | COM output scan start |
| `multiplex-ratio` | `<31>` | height - 1 |
| `segment-remap` | (boolean) | Flip horizontal orientation |
| `com-invdir` | (boolean) | Flip vertical orientation |
| `com-sequential` | (boolean) | **Required for 128x32.** Sets sequential COM pin layout. Without it, display shows garbled half-screen. |
| `prechargep` | `<0x22>` | Pre-charge period timing |
| `inversion-on` | (boolean) | White pixels on black background (standard OLED look) |

### Working 128x64 Device Tree Configuration

```dts
oled: ssd1306@3c {
    compatible = "solomon,ssd1306fb";
    reg = <0x3c>;
    width = <128>;
    height = <64>;
    segment-offset = <0>;
    page-offset = <0>;
    display-offset = <0>;
    multiplex-ratio = <63>;
    segment-remap;
    com-invdir;
    prechargep = <0x22>;
    inversion-on;
};
```

Note: 128x64 does **not** use `com-sequential` — the interleaved (alternative) COM layout is correct for 64-row panels.

## Pinctrl Configuration

Define I2C pin mappings in the board-specific overlay (e.g., `boards/seeeduino_xiao_ble.overlay`):

```dts
&pinctrl {
    i2c0_default: i2c0_default {
        group1 {
            psels = <NRF_PSEL(TWIM_SDA, 0, 4)>,   /* P0.04 = SDA */
                    <NRF_PSEL(TWIM_SCL, 0, 5)>;    /* P0.05 = SCL */
        };
    };

    i2c0_sleep: i2c0_sleep {
        group1 {
            psels = <NRF_PSEL(TWIM_SDA, 0, 4)>,
                    <NRF_PSEL(TWIM_SCL, 0, 5)>;
            low-power-enable;
        };
    };
};
```

Replace the pin numbers (`0, 4` and `0, 5`) with your actual SDA/SCL pins. Format is `NRF_PSEL(function, port, pin)`.

Both `default` and `sleep` pinctrl states are required. The `sleep` state adds `low-power-enable` for power management.

## Kconfig Configuration

### Shield `.conf` File (on the side with the display)

```ini
# Display
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_BUILT_IN=y
CONFIG_ZMK_WIDGET_LAYER_STATUS=y
CONFIG_ZMK_WIDGET_BATTERY_STATUS=y
CONFIG_ZMK_WIDGET_OUTPUT_STATUS=y
CONFIG_SSD1306=y
CONFIG_I2C=y
CONFIG_LV_Z_VDB_SIZE=64
CONFIG_LV_DPI_DEF=148
```

### Shield `Kconfig.defconfig`

```kconfig
config ZMK_DISPLAY
    select LV_USE_LABEL
```

**Do NOT use `select LV_USE_CONT`** — this widget was removed in LVGL 8.x (which ZMK v0.3 uses).

### Overlay (on the side with the display)

Add the display chosen node:

```dts
/ {
    chosen {
        zephyr,display = &oled;
    };
};
```

## Split Keyboard Considerations

### Display Widget Availability by Role

| Widget | Central | Peripheral |
|--------|---------|------------|
| `ZMK_WIDGET_LAYER_STATUS` | Yes | **No** |
| `ZMK_WIDGET_BATTERY_STATUS` | Yes | Yes |
| `ZMK_WIDGET_OUTPUT_STATUS` | Yes | **No** |
| `ZMK_WIDGET_WPM_STATUS` | Yes | **No** |
| `ZMK_WIDGET_PERIPHERAL_STATUS` | **No** | Yes |

**Recommendation:** Put the display on the **central** side so all widgets work. If the display must be on the peripheral side, only use `BATTERY_STATUS` and `PERIPHERAL_STATUS`.

### Central Side Placement for Best Results

If the central side has high-bandwidth devices (trackpad, display), those devices process data locally without crossing BLE. Only lightweight key matrix data from the peripheral side crosses the BLE split link. This prevents:
- Trackpad lag from BLE round-trips
- Display data competing with BLE radio time
- RGB underglow SPI traffic interfering with BLE throughput

### Critical Warning

**Never enable `CONFIG_ZMK_DISPLAY=y` on a side that has no physical display.** This causes the MCU to freeze/become unresponsive after the display blanking timeout.

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Build error: "Only one of SPI0, TWI0..." | SPI and I2C on same instance | Move one device to a different instance |
| Display garbled (full screen) | Using `nrf-twim` instead of `nrf-twi` | Remove `compatible = "nordic,nrf-twim"` from I2C node |
| Display garbled (half screen only) | Missing `com-sequential` on 128x32 | Add `com-sequential;` to SSD1306 node |
| Display blank/dark | Wrong I2C address | Try `reg = <0x3d>` instead of `<0x3c>` |
| Display shows inverted colors | `inversion-on` present/missing | Toggle the `inversion-on;` property |
| Display upside down | Wrong orientation flags | Toggle `segment-remap;` and/or `com-invdir;` |
| Widgets don't show on peripheral | Widget requires central role | Move display to central side, or use only `BATTERY_STATUS` / `PERIPHERAL_STATUS` |
| Peripheral side freezes | `CONFIG_ZMK_DISPLAY=y` without display hardware | Only enable display config on the side with a physical screen |
| Trackpad laggy with display + RGB | Too much BLE traffic | Make the side with trackpad + display the central side |

## Complete Working Example

### Board overlay (`boards/seeeduino_xiao_ble.overlay`)

```dts
&pinctrl {
    i2c0_default: i2c0_default {
        group1 {
            psels = <NRF_PSEL(TWIM_SDA, 0, 4)>,
                    <NRF_PSEL(TWIM_SCL, 0, 5)>;
        };
    };

    i2c0_sleep: i2c0_sleep {
        group1 {
            psels = <NRF_PSEL(TWIM_SDA, 0, 4)>,
                    <NRF_PSEL(TWIM_SCL, 0, 5)>;
            low-power-enable;
        };
    };
};

&i2c0 {
    status = "okay";
    pinctrl-0 = <&i2c0_default>;
    pinctrl-1 = <&i2c0_sleep>;
    pinctrl-names = "default", "sleep";

    oled: ssd1306@3c {
        compatible = "solomon,ssd1306fb";
        reg = <0x3c>;
        width = <128>;
        height = <32>;
        segment-offset = <0>;
        page-offset = <0>;
        display-offset = <0>;
        multiplex-ratio = <31>;
        segment-remap;
        com-invdir;
        com-sequential;
        prechargep = <0x22>;
        inversion-on;
    };
};
```

### Shield overlay (central side)

```dts
/ {
    chosen {
        zephyr,display = &oled;
    };
};
```

### Shield `.conf` (central side)

```ini
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_BUILT_IN=y
CONFIG_ZMK_WIDGET_LAYER_STATUS=y
CONFIG_ZMK_WIDGET_BATTERY_STATUS=y
CONFIG_ZMK_WIDGET_OUTPUT_STATUS=y
CONFIG_SSD1306=y
CONFIG_I2C=y
CONFIG_LV_Z_VDB_SIZE=64
CONFIG_LV_DPI_DEF=148
```

### Shield `Kconfig.defconfig`

```kconfig
config ZMK_DISPLAY
    select LV_USE_LABEL
```
