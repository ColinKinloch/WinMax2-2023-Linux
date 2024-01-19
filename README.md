### Sleep fix

The PCI SD card reader blocks ACPI sleep.

Solution: Disable wake for SD PCI syspath.

In `/etc/udev/rules.d/20-pci-sd-card-nowake.rules`: 
```
SUBSYSTEM=="pci", KERNEL=="0000:00:02.3", ATTR{device}=="0x14ee", ATTR{power/wakeup}="disabled"
```

### Real Sleep fix

from: https://gitlab.freedesktop.org/drm/amd/-/issues/3073

In `/etc/udev/rules.d/20-i2c-amd-nowake.rules`:
```
SUBSYSTEM=="i2c", KERNEL=="i2c-PNP0C50:00", ATTR{power/wakeup}="disabled"
SUBSYSTEM=="i2c", KERNEL=="i2c-GXTP7385:00", ATTR{power/wakeup}="disabled"
```


### Block Paddles + gamepad mouse/keyboard mapping

This also disables the face buttons in keyboard/mouse mode.

Identify "keyboard"

```
$ lsusb
Bus 001 Device 003: ID 2f24:0135   Mouse for Windows
```

Solution: Block USB device.

```
SUBSYSTEM=="usb", ATTRS{idVendor}=="2f24", ATTRS{idProduct}=="0135", ATTR{authorized}="0"
```

### Block or remap paddles

Run evtest as root and look for ` Mouse for Windows`

```bash
$ sudo evtest
...
/dev/input/event4:   Mouse for Windows
...
```

Find out what it's currently mapped to by pressing the button while `evtest` runs, in my case it's `70046` which is print screen.

```bash
$ sudo evtest /dev/input/event4
...
Event: time 1698507272.813553, type 4 (EV_MSC), code 4 (MSC_SCAN), value 70046
...
```

Create a file at `/etc/udev/hwdb.d/90-winmax2-paddles.hwdb`

`reserved` maps the button to nothing:

```hwdb
# /etc/udev/hwdb.d/90-winmax2-paddles.hwdb

# WinMax2 Paddles + keyboard/mouse gamepad
evdev:input:b0003v2F24p0135*
# Left Paddle PRINT_SCREEN
  KEYBOARD_KEY_70046=reserved
# KEYBOARD_KEY_70046=leftctrl
# Right Paddle PAUSE
 KEYBOARD_KEY_70048=reserved
# KEYBOARD_KEY_70048=leftalt
```

Then update `/etc/udev/hwdb.bin` with `systemd-hwdb`

```bash
sudo systemd-hwdb update
```

Then reboot

```
systemctl reboot
```

