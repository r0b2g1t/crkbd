# README for ZMK Shield Repository

This ZMK shield repository provides support for the **PandaKB Corne V3 Choc custom PCB** with specialized configurations for the nice view display adapter and LED underglow functionality.

## Overview

This repository contains a ZMK shield definition for the Corne (CRKBD) keyboard specifically tailored for the PandaKB Corne V3 Choc PCB kit. The shield includes optimized configurations for:

- **Custom nice view adapter** for display functionality
- **LED underglow pin configurations** for RGB lighting
- **Low-profile Choc switch support** for the V3 PCB variant


## Hardware Compatibility

### Supported PCB

- **PandaKB Corne V3 Choc PCB Kit**: [Available here](https://pandakb.com/products/pcb-kit/corne-v3-choc-pcb-kit/)


### Key Features of the PandaKB Kit:

- Supports 42 keys with Kailh Choc V1 and V2 low-profile switches
- Hot-swappable switch sockets
- Supports 2 OLED displays or nice view displays
- Per-key RGB LEDs (SK6812MINI-E)
- Underglow RGB LEDs (WS2812B-5050)
- Power switch support for wireless controllers
- Battery socket compatibility


## Shield Configuration

### Pin Assignments

The shield is configured with the following pin mappings optimized for the PandaKB V3 PCB:

**Matrix Pins:**

- Rows: GPIO 4, 5, 6, 7
- Left columns: GPIO 21, 20, 19, 18, 15, 14
- Right columns: GPIO 14, 15, 18, 19, 20, 21 (reversed)

**Display Configuration (nice!view SPI adapter):**

- Protocol: SPI0
- SCK: GPIO 20 (P0.20)
- MOSI: GPIO 17 (P0.17)
- MISO: GPIO 25 (P0.25)
- CS: GPIO 8 (P0.08)
- I2C for displays is disabled in this adapter
- Supports only SPI nice!view displays via this overlay

**RGB Underglow:**

- Data pin: GPIO 6 (P0.06) configured for SPI3
- Chain length: 21 LEDs total (6 underglow + 15 per-key)
- Protocol: WS2812 via SPI


### Custom nice view Adapter

This shield includes a specialized nice view adapter configuration that:

- Enables compatibility with the nice!view display using hardware SPI
- Is optimized for low power consumption
- Sets the SPI pin mapping as follows:
  - **SCK:** P0.20
  - **MOSI:** P0.17
  - **MISO:** P0.25 (often unused by display but defined on the bus)
  - **CS:** P0.08
- Automatically disables I2C for display usage to prevent bus conflicts
- Is intended exclusively for SPI-connected nice!view displays (not I2C OLEDs)


## Repository Structure

```
crkbd/
├── boards/shields/
│   ├── crkbd/                        # Main shield definition
│   │   ├── Kconfig.defconfig         # Default configurations
│   │   ├── Kconfig.shield            # Shield identification
│   │   ├── crkbd-layouts.dtsi        # Key matrix layout definition
│   │   ├── crkbd.conf                # Configuration overrides
│   │   ├── crkbd.dtsi                # Main device tree source
│   │   ├── crkbd_left.conf           # Left half advanced config
│   │   ├── crkbd_left.overlay        # Left side pin mappings
│   │   ├── crkbd_right.conf          # Right half advanced config
│   │   ├── crkbd_right.overlay       # Right side pin mappings
│   │   └── crkbd.zmk.yml             # Shield metadata
│   ├── nice_view_adapter/            # Custom adapter configuration
│   │   ├── boards/
│   │   │   └── nice_nano_v2.overlay  # nice!view v2 pin mappings
│   │   ├── Kconfig.defconfig         # Default configurations
│   │   ├── Kconfig.shield            # Shield identification
│   │   ├── nice_view_adapter.conf    # Configuration overrides
│   │   ├── nice_view_adapter.overlay # nice!view adapter pin mappings
│   │   └── nice_view_adapter.zmk.yml # Shield metadata
├── zephyr/
│   └── module.yml                    # Zephyr module definition
├── west.yml                          # West manifest
└── LICENSE                           # MIT License
```


## Using This Shield in Your ZMK Config

### Method 1: As a ZMK Module

Add this repository as a module to your `zmk-config/config/west.yml`:

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: r0b2g1t
      url-base: https://github.com/r0b2g1t
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
    - name: crkbd-shield
      remote: r0b2g1t
      repo-path: crkbd
      revision: main
  self:
    path: config
```


### Method 2: Copy Shield Definition

1. Create a `boards/shields/` directory in your zmk-config
2. Copy the `crkbd` folder from this repository to your `config/boards/shields/` directory
3. The shield will be available for building

### Build Configuration

Update your `build.yaml` to use the shield:

**For OLED displays:**

```yaml
include:
  - board: nice_nano_v2
    shield: crkbd_left
  - board: nice_nano_v2
    shield: crkbd_right
```

**For nice view displays:**

```yaml
include:
  - board: nice_nano_v2
    shield: crkbd_left nice_view_adapter nice_view
  - board: nice_nano_v2
    shield: crkbd_right nice_view_adapter nice_view
```


### Configuration Options

Create `crkbd.conf` in your config directory to enable features:

```
# Enable RGB underglow
CONFIG_ZMK_RGB_UNDERGLOW=y
CONFIG_WS2812_STRIP=y

# Enable display
CONFIG_ZMK_DISPLAY=y

# For nice view (if using)
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y

# Battery optimizations for wireless
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000
```


## Features

### Display Support

- **Dual display compatibility**: Works with both traditional OLED and nice view displays
- **Custom nice view adapter**: Optimized power consumption and pin mapping
- **Status information**: Battery level, layer indication, and connectivity status


### RGB Lighting

- **Per-key RGB**: Individual key illumination with SK6812MINI-E LEDs
- **Underglow RGB**: 6 WS2812B underglow LEDs for ambient lighting
- **Multiple effects**: Support for various RGB patterns and animations
- **Battery-aware**: Automatic brightness reduction when battery is low


### Wireless Optimization

- **Power management**: Optimized for battery life with wireless controllers
- **Deep sleep support**: Automatic sleep modes to conserve power
- **Battery monitoring**: Real-time battery level display and alerts


## Building and Flashing

### GitHub Actions (Recommended)

1. Fork this repository or add it as a module to your zmk-config
2. GitHub Actions will automatically build firmware on commits
3. Download the `.uf2` files from the Actions artifacts
4. Flash to your nice nano controllers

### Local Building

Follow the standard ZMK local build process:

```bash
west build -b nice_nano_v2 -- -DSHIELD="crkbd_left"
west build -b nice_nano_v2 -- -DSHIELD="crkbd_right"
```


## Troubleshooting

### Display Issues

- Ensure proper pin connections match the shield configuration
- For nice view, verify the adapter shield is included in build
- Check I2C pull-up resistors are properly configured


### RGB Problems

- Verify underglow pin (P0.06) is correctly wired
- Ensure LED chain connections are solid and in correct order
- Check power supply can handle LED current draw


### Matrix Issues

- Use the pin mapping reference to verify row/column connections
- Test continuity with multimeter if keys don't register
- Consider pin remapping if hardware damage occurs


## Contributing

This shield definition is based on the standard Corne configuration but optimized for the PandaKB V3 PCB variant. Contributions for improvements or additional features are welcome.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Original Corne design by [foostan](https://github.com/foostan/crkbd)
- PandaKB for the V3 PCB improvements and kit
- ZMK firmware community for the excellent documentation and support
- nice view display innovation by nice keyboards

For additional support and community discussion, visit the [ZMK Discord](https://zmk.dev/community/discord) or [r/ErgoMechKeyboards](https://reddit.com/r/ErgoMechKeyboards).
