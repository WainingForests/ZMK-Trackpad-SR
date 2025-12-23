# Troubleshooting Guide

## Common Build Errors

### SSL Connection Timeout (tinycrypt, zcbor, etc.)

**Error:**
```
fatal: unable to access 'https://github.com/zephyrproject-rtos/tinycrypt/': SSL connection timeout
ERROR: update failed for project tinycrypt
```

**Cause:** This is a transient GitHub Actions network issue, not a configuration problem.

**Solution:**
1. Re-run the GitHub Actions workflow
2. The issue usually resolves itself after 1-2 retries
3. If persistent, check [GitHub Status](https://www.githubstatus.com/)

### Shield Not Found Error

**Error:**
```
No shield named 'skreecustom_left' found
```

**Cause:** Missing or incorrect Kconfig files.

**Solution:** Verify these files exist:
- `boards/shields/skreecustom/Kconfig.shield`
- `boards/shields/skreecustom/Kconfig.defconfig`

### Module Import Error

**Error:**
```
FATAL ERROR: can't import from project zmk-driver-azoteq-iqs5xx
Expected to import from west.yml at revision...
```

**Cause:** Trying to import from a module that doesn't have a west.yml.

**Solution:** Remove `import: true` from the module in `config/west.yml`:
```yaml
- name: zmk-driver-azoteq-iqs5xx
  remote: aym1607
  revision: main
  # NO import: true here
```

### IQS5XX Config Symbol Not Found

**Error:**
```
warning: IQS5XX (defined at...) has direct dependencies... with value n
```

**Cause:** Wrong Kconfig symbol name.

**Solution:** Use `INPUT_AZOTEQ_IQS5XX` instead of `IQS5XX`:
```kconfig
config INPUT_AZOTEQ_IQS5XX
    default y
```

## Verification Steps

### Check Your Configuration

1. **Verify west.yml**
```yaml
projects:
  - name: zmk
    remote: zmkfirmware
    revision: v0.3
    import: app/west.yml
  - name: zmk-driver-azoteq-iqs5xx
    remote: aym1607
    revision: main
    # No import here
```

2. **Verify Kconfig.defconfig has**
```kconfig
config I2C
    default y

config INPUT
    default y

config INPUT_AZOTEQ_IQS5XX
    default y
```

3. **Verify input-listener nodes exist**

Left overlay should have:
```dts
/ {
    trackpad_central_listener: trackpad_central_listener {
        compatible = "zmk,input-listener";
        device = <&trackpad_central>;
    };
};
```

Right overlay should have:
```dts
/ {
    trackpad_peripheral_listener: trackpad_peripheral_listener {
        compatible = "zmk,input-listener";
        device = <&trackpad_peripheral>;
    };
};
```

## ZMK Version Notes

### CRITICAL: Driver and ZMK Timeline

- **ZMK v0.3.0 released:** August 1, 2025
- **zmk-driver-azoteq-iqs5xx created:** August 9, 2025 (8 days later)
- **Zephyr 4.1 breaking changes:** December 9, 2025 (merged to main)

**The TPS65 driver was created FOR v0.3**, just after its release. The driver does NOT work with current main branch due to Zephyr 4.1 breaking changes (HWMv2, LVGL 9.3, bootloader changes, etc.).

### v0.3 (Current Configuration - CORRECT)
- **Pro:** Driver was built for this version
- **Pro:** Stable release
- **Pro:** No breaking changes
- **Recommended:** YES - This is the correct version

### main branch (DO NOT USE)
- **Con:** Zephyr 4.1 breaking changes (December 2025)
- **Con:** Driver not updated for new APIs
- **Con:** HWMv2, LVGL 9.3, bootloader changes break compatibility
- **Not Recommended:** Driver will not work

## Reference Configurations

### Working Examples
1. **HumanComputingInc/zmk-keyboard-iqs5xx-dev** (ZMK main)
   - Minimal IQS5XX development shield
   - Shows basic input-listener setup

2. **Aleblazer/ZMK-Zodify** (branch: XDLCD-Azoteq-Test)
   - Full keyboard with TPS43 trackpad
   - Uses ZMK main branch
   - Demonstrates split keyboard with trackpad

## Getting Help

If you continue to have issues:

1. Check the [ZMK Discord](https://zmk.dev/community/discord/invite) #help channel
2. Review [zmk-driver-azoteq-iqs5xx](https://github.com/AYM1607/zmk-driver-azoteq-iqs5xx) issues
3. Verify hardware connections match pin configuration
4. Test with a minimal configuration first

## Hardware Verification

### Pin Checklist

Ensure your hardware matches the configured pins:

| Signal | Pin | Configuration |
|--------|-----|---------------|
| I2C SDA | P0.19 | I2C data |
| I2C SCL | P0.15 | I2C clock |
| Reset | P0.02 | Active LOW |
| Ready | P0.29 | Active HIGH |

### Common Hardware Issues

1. **Wrong I2C address:** TPS65 should be at 0x74
2. **Pull-up resistors:** I2C lines need 4.7kΩ pull-ups
3. **Power:** Ensure TPS65 has stable 3.3V supply
4. **Ready pin polarity:** Must be Active HIGH (not LOW like PMW3610 IRQ)
