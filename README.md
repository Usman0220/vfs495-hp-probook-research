# Validity VFS495 (`138a:003f`) Hardware Verification & Research on HP ProBook 430 G3

This repository documents hardware verification, reverse-engineering driver validation, and standalone testing of the **Validity Sensors, Inc. VFS495 Fingerprint Reader (`138a:003f`)** on an **HP ProBook 430 G3** running **Kali Linux**.

This research builds upon and confirms the work by [Ahmad-b1995/vfs495-driver](https://github.com/Ahmad-b1995/vfs495-driver) and [saifulmd0/vfs495-linux](https://github.com/saifulmd0/vfs495-linux).

---

## Hardware & System Profile

| Component | Specification |
| :--- | :--- |
| **Laptop Model** | HP ProBook 430 G3 |
| **Vendor** | HP (Hewlett-Packard) |
| **Fingerprint Sensor** | Validity Sensors, Inc. VFS495 Fingerprint Reader |
| **USB Vendor / Product ID** | `138a:003f` |
| **Sensor Revision** | `bcdDevice 1.04` (Falcon ROM) |
| **Operating System** | Kali Linux (x86_64) |
| **Kernel Version** | `7.2.15+kali-amd64` |

---

## Key Research Findings

1. **Cross-Model Compatibility:**
   The upstream project was previously verified primarily on an **HP ZBook 17 G3**. This test proves that the same setup commands, cryptographic handshake, and sensor patches extracted from HP SoftPaq **sp84530** work cleanly on the **HP ProBook 430 G3** (`bcdDevice 1.04`).

2. **Standalone Compilation on Debian/Kali:**
   While `libfprint-tod` is packaged primarily on Ubuntu, the sensor's core user-space communication stack (`vfs495-tool`) compiles and runs directly on Debian/Kali using standard distro packages (`libgusb-dev`, `libssl-dev`, `libglib2.0-dev`).

3. **Sensor Performance & Image Capture:**
   * **Initialization / Reset:** Port reset and session setup completed in **0.46 seconds**.
   * **Sensor Arming:** Arming capacitive swipe detection completed in **0.04 seconds**.
   * **Capture Quality:**
     * **Confidence:** `97%`
     * **Coverage:** `85%`
     * **Resulting Image:** `200 x 286` Netpbm greymap (assembled from 2,137 raw scan lines in 0.17 s).

---

## Verification Test Log

```text
session ready in 0.46 s
[1/1] armed in 0.04 s: swipe now
   first data 1872 ms after arming; pauses: longest 2 ms; over 25 ms at frame:ms none
   region 823 lines, trusted 781, confident 97%, fast 0%, median lag 23, still 2%, coverage 85% -> ok
   581536 bytes, 2137 lines -> 286 rows (print lines 0-814) in 0.17 s: /tmp/test_swipe.1.pgm
```

---

## How to Reproduce / Test on Debian & Kali Linux

### 1. Install Build Dependencies
```bash
sudo apt update
sudo apt install -y git build-essential pkg-config curl python3 \
    libgusb-dev libssl-dev libglib2.0-dev
```

### 2. Clone the Driver Repository
```bash
git clone https://github.com/Ahmad-b1995/vfs495-driver.git
cd vfs495-driver
```

### 3. Fetch HP Firmware & Build Commands
The script downloads HP's Linux package (`sp84530.tar`, ~2.8 MB) and generates the sensor's initialization tables (`commands.bin`):
```bash
mkdir -p build/hp
curl -fSL -o build/hp/sp84530.tar "https://ftp.hp.com/pub/softpaq/sp84501-85000/sp84530.tar"
python3 driver/recipe/build-commands.py build/hp/sp84530.tar build/hp/commands.bin
```

### 4. Compile the Standalone Tester (`vfs495-tool`)
```bash
cd driver
gcc -O2 -Wall \
  src/vfs495-tool.c src/vfs495-proto.c src/vfs495-assemble.c src/vfs495-commands.c src/vfs495-protocol.c src/vfs495-image.c \
  -Isrc \
  $(pkg-config --cflags --libs glib-2.0 gio-2.0 gusb libcrypto) -lm \
  -o ../build/vfs495-tool
cd ..
```

### 5. Run Live Fingerprint Swipe Test
```bash
sudo ./build/vfs495-tool capture build/hp/commands.bin /tmp/test_finger 1 15
```
Swipe your finger slowly and smoothly over the sensor strip when prompted. The captured image will be saved to `/tmp/test_finger.1.pgm`.

---

## Credits & Upstream
* Driver & Reverse Engineering: [Ahmad-b1995/vfs495-driver](https://github.com/Ahmad-b1995/vfs495-driver)
* Protocol Research: [saifulmd0/vfs495-linux](https://github.com/saifulmd0/vfs495-linux)
* License: LGPL-2.1 / MIT
