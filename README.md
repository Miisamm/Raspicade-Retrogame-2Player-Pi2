Raspicade-Retrogame-2Player-Pi2
===============================

Raspberry Pi GPIO-to-USB utility for classic game emulators — 2-player arcade cabinet version.

Based on [Adafruit Retrogame](https://github.com/adafruit/Adafruit-Retrogame), extended for 2-player wiring.

> **This fork adds kernel 6.x compatibility.** The original code broke silently on Linux 6.x (Debian Bookworm/Trixie) due to a GPIO sysfs base offset change. See [Kernel 6.x Fix](#kernel-6x-fix) below.

---

## Compilation

```bash
git clone https://github.com/Miisamm/Raspicade-Retrogame-2Player-Pi2.git
cd Raspicade-Retrogame-2Player-Pi2
make
```

Requires `gcc` and standard build tools (`sudo apt install build-essential`).

---

## Installation

Retrogame requires the `uinput` kernel module. To enable it persistently:

```bash
sudo sh -c 'echo uinput >> /etc/modules'
sudo modprobe uinput
```

### Autostart with systemd (recommended)

Create `/etc/systemd/system/retrogame.service`:

```ini
[Unit]
Description=Retrogame GPIO joystick daemon
After=local-fs.target

[Service]
Type=simple
ExecStart=/home/pi/Raspicade-Retrogame-2Player-Pi2/retrogame
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Then enable it:

```bash
sudo systemctl enable retrogame
sudo systemctl start retrogame
```

### Legacy rc.local method

Add this line to `/etc/rc.local` before `exit 0`:

```bash
/home/pi/Raspicade-Retrogame-2Player-Pi2/retrogame &
```

---

## Pinout Mapping

```
Player 1:
  GPIO 02 -> KEY_UP          Up
  GPIO 03 -> KEY_DOWN        Down
  GPIO 04 -> KEY_LEFT        Left
  GPIO 17 -> KEY_RIGHT       Right
  GPIO 27 -> KEY_LEFTCTRL    Button 1
  GPIO 22 -> KEY_LEFTALT     Button 2
  GPIO 10 -> KEY_SPACE       Button 3
  GPIO 09 -> KEY_LEFTSHIFT   Button 4
  GPIO 11 -> KEY_Z           Button 5
  GPIO 05 -> KEY_X           Button 6
  GPIO 06 -> KEY_1           Start P1
  GPIO 13 -> KEY_5           Coins/Credits P1

Player 2:
  GPIO 18 -> KEY_R           Up
  GPIO 23 -> KEY_F           Down
  GPIO 24 -> KEY_D           Left
  GPIO 25 -> KEY_G           Right
  GPIO 08 -> KEY_A           Button 1
  GPIO 07 -> KEY_S           Button 2
  GPIO 12 -> KEY_Q           Button 3
  GPIO 16 -> KEY_W           Button 4
  GPIO 20 -> KEY_E           Button 5
  GPIO 21 -> KEY_T           Button 6
  GPIO 19 -> KEY_2           Start P2
  GPIO 26 -> KEY_6           Coins/Credits P2

System:
  GPIO 15 -> KEY_0           Halt system (triggers sudo halt)
```

Holding **Start P1 + Coins P1** for more than 1 second sends **KEY_ESC**.

All buttons connect between the GPIO pin and GND. Internal pullups are used — no resistors needed.

---

## Kernel 6.x Fix

On Linux kernel 6.x, the BCM2835/BCM2711 GPIO chip (`pinctrl-bcm2835`) no longer appears at sysfs base offset 0. It moves to a higher offset (typically 512 on Pi 3/4), causing all GPIO sysfs exports to silently target the wrong chip — buttons appear to do nothing.

Additionally, the original Pi board detection relied on `mem_size=` in `/proc/cmdline`, which was removed in kernel 6.x.

**This fork fixes both issues:**

1. `detectGpioBase()` — scans `/sys/class/gpio/gpiochip*/label` to find the `pinctrl-bcm2835` or `pinctrl-bcm2711` chip and reads its actual base offset dynamically. Works on all kernel versions.

2. `boardType()` — reads the SoC peripheral base address from `/proc/device-tree/soc/ranges` instead of parsing `/proc/cmdline`. More reliable across kernel versions.

**Tested on:** Raspberry Pi 3 Model B, Linux 6.12, Debian Trixie + RetroPie 4.8

---

## Arcade Wiring

See the original [Raspicade wiki](https://github.com/ian57/Raspicade-Retrogame-2Player-BPlus/wiki) for wiring diagrams.
