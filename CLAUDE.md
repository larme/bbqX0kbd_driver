# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Build the kernel module
make

# Install the kernel module and configure the system
sudo make install

# Uninstall the module
sudo make uninstall

# Clean build artifacts
make clean

# Rebuild module dependencies after manual changes
sudo depmod -a
```

## Architecture Overview

This is a Linux kernel module driver for the Beepy keyboard hardware. The codebase follows a modular architecture:

### Core Components

1. **Main Module** (`src/main.c`): Entry point that orchestrates initialization of all subsystems
   - Handles I2C device probing and removal
   - Coordinates initialization order: input → params → sysfs → ioctl

2. **Input Interface** (`src/input_iface.[ch]`): Handles keyboard input events and key mapping
   - Manages interrupt-based key event handling
   - Implements sticky modifier keys and meta mode
   - Integrates with Linux input subsystem

3. **Sysfs Interface** (`src/sysfs_iface.[ch]`): Exposes device controls via `/sys/firmware/beepy/`
   - LED control (color, brightness)
   - Keyboard backlight control
   - Battery status reporting

4. **IOCTL Interface** (`src/ioctl_iface.[ch]`): Provides direct device control interface
   - Used for advanced device configuration
   - Integrates with Sharp DRM driver for display indicators

5. **Parameters Interface** (`src/params_iface.[ch]`): Module parameter management
   - Touchpad mode configuration (meta vs keys)
   - Runtime configurable via `/sys/module/beepy_kbd/parameters/`

### Key Files

- `beepy-kbd.dts`: Device tree overlay defining hardware connections (I2C address 0x1F, interrupt pin)
- `beepy-kbd.map`: Keyboard keymap file installed to `/usr/share/kbd/keymaps/`
- `init/S01beepykbd`: Init script for loading keymap at boot (note: has path typo on line 14)
- `src/bbq*_codes.h`: Hardware-specific keycodes for different keyboard variants
- `src/registers.h`: I2C register definitions for keyboard firmware communication

### Build System

- Uses kernel module build system (Kbuild)
- Supports both manual builds and DKMS integration
- Buildroot package support via `beepy-kbd.mk`
- Device tree overlay compilation included

## Known Issues

1. **Build Error**: The current code has a kernel API compatibility issue - the probe function signature is outdated for kernel 6.12+ (see error.txt)
2. **Init Script Path**: Line 14 in `init/S01beepykbd` has a missing slash in the keymap path

## Development Notes

- The driver only supports interrupt mode (polling disabled)
- Currently hardcoded for BBQ20KBD_PMOD variant
- Integrates with Sharp DRM driver for visual mode indicators
- Module uses LGPL-2.0 license