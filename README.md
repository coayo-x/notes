# Raspberry Pi 5 RGB Matrix (64×32) – Full Setup Guide

## Overview

This guide explains how to run a 64×32 HUB75 RGB LED panel on a Raspberry Pi 5 and Adafruit’s Pi 5 PIO-based Piomatter library.

---

# 1. Hardware Requirements

### Required Components

* Raspberry Pi 5
* MicroSD card (16GB+ recommended)
* HUB75 64×32 RGB LED panel
* Adafruit RGB Matrix HAT (or compatible)
* Ribbon cable (HUB75)
* 5V power supply for Raspberry Pi
* 5V 4–5A power supply for LED panel (minimum recommended)

---

# 2. Power Requirements 

A 64×32 RGB panel can draw significant current.

### Recommended Power Setup

* Raspberry Pi powered separately via USB-C.
* LED panel powered directly using a dedicated 5V 4–5A supply.
* HAT receives proper 5V routing via correct jumper placement.

### Not Recommended

* Using only a 5V 2A supply.
* Powering the panel through the Pi.
* Incorrect jumper placement between HAT and panel.

Low current will result in:

* Very dim display
* Flickering
* Random shutdowns
* No display at all

---

# 3. Install Operating System

1. Use Raspberry Pi Imager.
2. Flash to microSD.
3. Boot Raspberry Pi 5.

---

# 4. Update the System

```bash
sudo apt update
sudo apt upgrade -y
```

---

# 5. Install Required Packages

```bash
sudo apt install python3-venv python3-pip -y
```

---

# 6. Create Virtual Environment

```bash
cd ~
python3 -m venv pi5env
source pi5env/bin/activate
```

After activation, terminal should show:

```
(pi5env)
```

---

# 7. Install Required Python Libraries

```bash
pip install adafruit-blinka-raspberry-pi5-piomatter
pip install adafruit-blinka
pip install numpy
```

If NumPy errors appear related to OpenBLAS:

```bash
sudo apt install libopenblas-dev -y
```

---

# 8. Enable Interfaces

```bash
sudo raspi-config
```

Enable:

* SPI
* I2C

Reboot:

```bash
sudo reboot
```

After reboot:

```bash
source pi5env/bin/activate
```

---

# 9. Hardware Wiring

1. Attach HAT to Raspberry Pi 5.
2. Connect HUB75 ribbon cable correctly.
3. Ensure ribbon cable orientation is correct.
4. Verify jumper position between HAT and LED panel.
5. Connect 5V 4–5A supply directly to LED panel.
6. Power Raspberry Pi separately.

Incorrect jumper placement will prevent initialization.

---

# 10. Create Display Script

```bash
nano test_matrix.py
```

Paste your display code.

for example:
```bash
import time
import numpy as np
import adafruit_blinka_raspberry_pi5_piomatter as piomatter

WIDTH = 64
HEIGHT = 32

geometry = piomatter.Geometry(
    width=WIDTH,
    height=HEIGHT,
    n_addr_lines=4
)

pinout = piomatter.Pinout.AdafruitMatrixHatBGR
colorspace = piomatter.Colorspace.RGB888

framebuffer = np.zeros((HEIGHT, WIDTH, 4), dtype=np.uint8)

matrix = piomatter.PioMatter(
    colorspace,
    pinout,
    framebuffer,
    geometry
)

font = {
    "6": [
        "011110",
        "100000",
        "111110",
        "100001",
        "100001",
        "011110",
    ],
    "7": [
        "111111",
        "000010",
        "000100",
        "001000",
        "010000",
        "010000",
    ],
}

def draw_char(char, x_offset, y_offset, color, scale=4):
    pattern = font[char]
    for y, row in enumerate(pattern):
        for x, pixel in enumerate(row):
            if pixel == "1":
                for dy in range(scale):
                    for dx in range(scale):
                        framebuffer[
                            y_offset + y*scale + dy,
                            x_offset + x*scale + dx
                        ] = (*color, 255)

framebuffer[:, :] = (0, 0, 0, 255)

red_dim = (70, 0, 0)

draw_char("6", 0, 0, red_dim, scale=4)
draw_char("7", 32, 0, red_dim, scale=4)

framebuffer[:] = np.flipud(np.fliplr(framebuffer))

matrix.show()

while True:
    time.sleep(1)
```
Save:

```
CTRL + X
Y
Enter
```

---

# 11. Run the Code

Activate environment:

```bash
source pi5env/bin/activate
```

Run:

```bash
python test_matrix.py
```

If everything is configured properly, the LED panel should display the configured output.

---

# 12. Common Problems and Causes

### ModuleNotFoundError

Virtual environment not activated.

Solution:

```bash
source pi5env/bin/activate
```

---

### Panel Completely Off

* Jumper incorrectly placed
* HAT not receiving 5V
* Ribbon cable reversed
* Insufficient power supply

---

### Wrong Colors (Red shows as Blue)

Use:

```
Pinout.AdafruitMatrixHatBGR
```

instead of RGB.

---

### Low Brightness

Power supply too weak (2A is insufficient).

---

# 13. Final Notes

* Raspberry Pi 5 does not support the old rpi-rgb-led-matrix memory-mapped method reliably.
* Piomatter is required for proper Pi 5 operation.
* Stable power delivery is critical.
* Most failures are hardware-related, not software-related.
