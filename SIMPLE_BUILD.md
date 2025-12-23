# Simplified Skreepad Build

This is a **minimal, single-board test configuration** with just the TPS65 trackpad.

## What This Tests

- ✅ TPS65 trackpad on I2C1
- ✅ All gesture features (one-finger tap, two-finger tap, press-and-hold, scroll)
- ✅ Single key matrix (minimal kscan for testing)
- ❌ NO split keyboard functionality
- ❌ NO shift register
- ❌ NO encoders

## Hardware Requirements

### Connections Required
| Signal | XIAO Pin | TPS65 Pin |
|--------|----------|-----------|
| I2C SDA | P0.19 (D4) | SDA |
| I2C SCL | P0.15 (D5) | SCL |
| Reset | P0.02 (D0) | RST |
| Ready | P0.29 (D3) | RDY |
| 3.3V | 3V3 | VCC |
| GND | GND | GND |

### Optional Test Key
- Row: P1.15 (D9)
- Col: P1.13 (D8)

## Files

```
boards/shields/skreepad/
├── Kconfig.shield          # Shield definition
├── Kconfig.defconfig       # Auto-enables I2C, INPUT, IQS5XX
├── skreepad.overlay        # Hardware configuration
└── skreepad.conf           # Shield config (empty)

config/
└── skreepad.keymap         # Minimal 1-key keymap
```

## Building

This shield is configured in `build.yaml`:

```yaml
include:
  - board: seeeduino_xiao_ble
    shield: skreepad
```

Push to GitHub and the firmware will build automatically.

## Next Steps

Once this minimal configuration works:

1. Test trackpad gestures:
   - One finger tap = left click
   - Two finger tap = right click
   - Press and hold = click + drag
   - Scroll gestures

2. Add shift register matrix

3. Add encoders

4. Add split functionality

## Troubleshooting

If the build still fails with "shield not found":
1. Check all files are committed to git
2. Verify file names match exactly (case-sensitive)
3. Check Kconfig.shield has proper syntax

If trackpad doesn't work:
1. Verify I2C pull-up resistors (4.7kΩ to 3.3V)
2. Check Ready pin is HIGH when idle
3. Verify I2C address is 0x74
4. Confirm TPS65 has stable 3.3V power
