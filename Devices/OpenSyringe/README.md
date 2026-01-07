# OpenSyringe - Raspberry Pi Infusion Pump Emulation

Emulation guide for the [MTU MOST Linear Actuator](https://github.com/mtu-most/linear-actuator) Raspberry Pi-based open-source infusion pump.

## Overview

The MTU MOST linear actuator is an open-source syringe pump originally designed to run on Raspberry Pi hardware.

## Prerequisites

Install QEMU on your Linux host:

```bash
sudo apt update
sudo apt install qemu-system-arm qemu-utils wget unxz-utils -y
```

## Setup Instructions

### Step 1: Create Working Directory

```bash
mkdir opensyringe-emulation
cd opensyringe-emulation
```

### Step 2: Download Raspberry Pi OS

```bash
# Download Raspberry Pi OS Lite
wget https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2024-03-15/2024-03-15-raspios-bookworm-armhf-lite.img.xz

# Extract
unxz 2024-03-15-raspios-bookworm-armhf-lite.img.xz

# Resize to 8GB to provide working space
qemu-img resize 2024-03-15-raspios-bookworm-armhf-lite.img 8G
```

### Step 3: Download QEMU Kernel

Pre-built QEMU-compatible kernels work better than extracting from the image:

```bash
# Download kernel
wget https://github.com/dhruvvyas90/qemu-rpi-kernel/raw/master/kernel-qemu-5.10.63-bullseye

# Download device tree blob
wget https://github.com/dhruvvyas90/qemu-rpi-kernel/raw/master/versatile-pb-bullseye-5.10.63.dtb
```

### Step 4: Launch QEMU

```bash
qemu-system-arm \
  -M versatilepb \
  -cpu arm1176 \
  -m 256 \
  -kernel kernel-qemu-5.10.63-bullseye \
  -dtb versatile-pb-bullseye-5.10.63.dtb \
  -hda 2024-03-15-raspios-bookworm-armhf-lite.img \
  -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" \
  -net nic -net user,hostfwd=tcp::8080-:8080 \
  -no-reboot
```

**Port Forwarding:**
- Port 8080 (host) → Port 8080 (Web interface)

### Step 5: First Boot

Wait for the boot sequence to complete. Login with:
- **Username:** `pi`
- **Password:** `raspberry`

Change the default password:
```bash
passwd
```

## Installing OpenSyringe Software

Inside the QEMU emulated Raspberry Pi:

### Install Dependencies

```bash
sudo apt update
sudo apt install -y git python3 python3-pip python3-tornado
```

### Clone the Repository

```bash
cd ~
git clone https://github.com/mtu-most/linear-actuator.git
cd linear-actuator
```

### Install Python Dependencies

```bash
pip3 install websocket-client --break-system-packages
```

Note: `--break-system-packages` is needed on newer Debian/Raspberry Pi OS versions.

### Clone Additional Dependencies

The pump-server needs `rpc.js` from python-websocketd:

```bash
cd ~
git clone https://github.com/mtu-most/python-websocketd.git
```

Copy the required file:

```bash
cp ~/python-websocketd/rpc.js ~/linear-actuator/pump-server/
```

## Running the Pump Server

Start the pump server:

```bash
cd ~/linear-actuator/pump-server
python3 pump-server.py
```

The server will start and listen on port 8080.

## Accessing the Emulated Pump

### Web Interface

From your host machine, open a web browser to:
```
http://localhost:8080
```

## GPIO Emulation

QEMU doesn't emulate real GPIO pins, so motor control won't function. For testing purposes, create a mock GPIO library:

Create `~/mock_gpio.py`:

```python
class GPIO:
    BCM = 'BCM'
    BOARD = 'BOARD'
    OUT = 1
    IN = 0
    HIGH = 1
    LOW = 0

    @staticmethod
    def setmode(mode):
        print(f"[GPIO Mock] setmode({mode})")

    @staticmethod
    def setwarnings(flag):
        print(f"[GPIO Mock] setwarnings({flag})")

    @staticmethod
    def setup(channel, mode, initial=LOW):
        print(f"[GPIO Mock] setup(channel={channel}, mode={mode}, initial={initial})")

    @staticmethod
    def output(channel, state):
        print(f"[GPIO Mock] output(channel={channel}, state={state})")

    @staticmethod
    def input(channel):
        print(f"[GPIO Mock] input(channel={channel})")
        return 0

    @staticmethod
    def cleanup():
        print("[GPIO Mock] cleanup()")
```

Modify the pump software to use the mock:

```python
# At the top of the pump control script
import sys
sys.path.insert(0, '/home/pi')
from mock_gpio import GPIO
```