# MicroPython for YD-RP2040 (16MB Flash)

Custom MicroPython firmware for the **YD-RP2040** development board by VCC-GND Studio, featuring 16MB of flash storage.

---

## 📋 About This Board

The YD-RP2040 is a Raspberry Pi Pico-compatible board with enhanced features. It uses the same RP2040 microcontroller but comes with significantly more storage and a few extra goodies.

### Hardware Comparison

| Feature | Raspberry Pi Pico | YD-RP2040 |
|---------|-------------------|-----------|
| Microcontroller | RP2040 | RP2040 |
| Flash Storage | 2MB | **16MB** |
| Filesystem Space | ~1MB | **~15MB** |
| USB Connector | Micro USB | **USB-C** |
| Onboard LED | GPIO25 | GPIO25 |
| NeoPixel RGB LED | ❌ | **GPIO23** |
| User Button | ❌ | **GPIO24** |
| Reset Button | ❌ | **✅ Hardware** |
| Castellated Pads | ✅ | ✅ |
| Form Factor | Breadboard-friendly | Breadboard-friendly |

---

## ⚡ Features

- **15MB filesystem** — Store large projects, data logs, libraries, and assets directly on the board
- **WS2812 NeoPixel LED** — Built-in RGB LED for status indication or just for fun
- **User button** — Dedicated button on GPIO24 for user input (works out of the box, no soldering required)
- **Hardware reset** — Physical reset button (no more unplugging!)
- **USB-C** — Modern, reversible connector
- **Full Pico compatibility** — All GPIOs, peripherals, and libraries work identically

---

## ⚠️ NeoPixel Solder Bridge

**Important:** Some YD-RP2040 boards ship with the NeoPixel disconnected from GPIO23. If your NeoPixel doesn't work, you need to close a solder bridge.

Look for two small pads labeled **"RGB"** near the NeoPixel LED (may also look like "R7" or "R58"). Bridge these pads with a small amount of solder to connect GPIO23 to the NeoPixel.

Once soldered, the NeoPixel will work with:

```python
from machine import Pin
from neopixel import NeoPixel

np = NeoPixel(Pin(23), 1)
np[0] = (255, 0, 0)  # Red
np.write()
```

---

## 🔌 Pinout

```
                    ┌───────────────┐
            GPIO0  ─┤1            40├─ VBUS
            GPIO1  ─┤2            39├─ VSYS
              GND  ─┤3            38├─ GND
            GPIO2  ─┤4            37├─ 3V3_EN
            GPIO3  ─┤5            36├─ 3V3
            GPIO4  ─┤6            35├─ ADC_VREF
            GPIO5  ─┤7            34├─ GPIO28 (ADC2)
              GND  ─┤8            33├─ GND
            GPIO6  ─┤9            32├─ GPIO27 (ADC1)
            GPIO7  ─┤10           31├─ GPIO26 (ADC0)
            GPIO8  ─┤11           30├─ RUN
            GPIO9  ─┤12           29├─ GPIO22
              GND  ─┤13           28├─ GND
           GPIO10  ─┤14           27├─ GPIO21
           GPIO11  ─┤15           26├─ GPIO20
           GPIO12  ─┤16           25├─ GPIO19
           GPIO13  ─┤17           24├─ GPIO18
              GND  ─┤18           23├─ GND
           GPIO14  ─┤19           22├─ GPIO17
           GPIO15  ─┤20           21├─ GPIO16
                    └───────────────┘

    Special Pins (on-board components):
    ────────────────────────────────────
    GPIO23 → NeoPixel RGB LED (requires solder bridge)
    GPIO24 → USR Button (works out of the box)
    GPIO25 → Blue LED
```

---

## 📥 Installation

### Step 1: Download the Firmware

Download `firmware.uf2` from the [Releases](../../releases) page or build it yourself using the included GitHub Actions workflow.

### Step 2: Enter Bootloader Mode

1. **Hold** the `BOOT` button on the board
2. **Connect** the USB cable to your computer while holding `BOOT`
3. **Release** the `BOOT` button

A new drive called **RPI-RP2** will appear on your computer.

### Step 3: Flash the Firmware

Drag and drop `firmware.uf2` onto the **RPI-RP2** drive.

The board will automatically reboot into MicroPython. The drive will disappear — this is normal.

### Step 4: Connect

Use any serial terminal or IDE to connect:

- **Thonny** (recommended for beginners) — Select "MicroPython (Raspberry Pi Pico)"
- **PuTTY / Screen** — Connect to the COM port at 115200 baud
- **VS Code** — Use the Pico-Go or Pymakr extension

---

## 🏗️ Building From Source

This repository includes a GitHub Actions workflow that builds the firmware automatically.

### To Build:

1. Go to the **Actions** tab
2. Click **Build MicroPython for YD-RP2040-16MB**
3. Click **Run workflow**
4. Wait ~4 minutes for the build
5. Download the artifact containing `firmware.uf2`

### Manual Build (Linux/WSL):

```bash
# Install dependencies
sudo apt install cmake gcc-arm-none-eabi libnewlib-arm-none-eabi build-essential

# Clone MicroPython
git clone https://github.com/micropython/micropython.git
cd micropython

# Initialize
make -C ports/rp2 submodules
make -C mpy-cross

# Create board definition
mkdir -p ports/rp2/boards/YD_RP2040_16MB

cat > ports/rp2/boards/YD_RP2040_16MB/mpconfigboard.h << 'EOF'
#define MICROPY_HW_BOARD_NAME           "YD-RP2040-16MB"
#define MICROPY_HW_FLASH_STORAGE_BYTES  (15 * 1024 * 1024)
EOF

cat > ports/rp2/boards/YD_RP2040_16MB/mpconfigboard.cmake << 'EOF'
set(PICO_BOARD "pico")
set(PICO_PLATFORM "rp2040")
set(PICO_FLASH_SIZE_BYTES 16777216)
EOF

# Build
cd ports/rp2
make BOARD=YD_RP2040_16MB
```

The firmware will be at `build-YD_RP2040_16MB/firmware.uf2`

---

## ❓ Troubleshooting

### NeoPixel not working

**Most likely cause:** The solder bridge is not closed. Look for pads labeled "RGB" near the NeoPixel and bridge them with solder. See the [NeoPixel Solder Bridge](#️-neopixel-solder-bridge) section above.

Other checks:
- Make sure you're using GPIO23: `NeoPixel(Pin(23), 1)`
- Check that you called `np.write()` after setting the color

### Board not recognized after flashing

- Try a different USB cable (some are charge-only)
- Try a different USB port
- Hold BOOT and press RESET to re-enter bootloader mode

### Can't enter bootloader mode

1. Unplug the board
2. Hold the BOOT button
3. Plug in USB while holding BOOT
4. Release BOOT after 1 second

### REPL not responding

- Make sure no other program is using the COM port
- Try pressing Ctrl+C to interrupt any running program
- Press the hardware RESET button

---

## 📚 Resources

- [MicroPython Documentation](https://docs.micropython.org/)
- [RP2040 Datasheet](https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf)
- [MicroPython RP2 Quick Reference](https://docs.micropython.org/en/latest/rp2/quickref.html)
- [YD-RP2040 Schematic](https://github.com/initdc/YD-RP2040/blob/master/YD-2040-2022-V1.1-SCH.pdf)

---

## 📄 License

This project uses the same license as MicroPython (MIT License).
