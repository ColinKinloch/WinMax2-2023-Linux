### Sleep fix

The PCI SD card reader blocks ACPI sleep.

Solution: Disable wake for SD PCI syspath.

```
SUBSYSTEM=="pci", KERNEL=="0000:00:02.3", ATTR{device}=="0x14ee", ATTR{power/wakeup}="disabled"
```

### Block Paddles

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

