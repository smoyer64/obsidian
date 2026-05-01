---
tags:
  - hardware
  - computer
  - configuration
  - dell
---
## Logitech K480 Bluetooth Keyboard (and more

  

Devices with legacy Bluetooth pairing may not be visible from the `blueman-applet` that's shown in the XFCE toolbar. In this case, it becomes necessary to pair the device from the CLI. Follow the instructions at https://www.linuxquestions.org/questions/linux-hardware-18/pairing-bluetooth-keyboard-logitech-k480-4175613889/

^^ Replace these with updated versions as you no longer use the `-a` option.

## For DisplayLink on Debian 12

Follow these instructions:

https://code.berrydejager.com/Fix-DisplayLink_drivers-linux-kernel-6/

But use this command to download the evdi driver source:

```

curl -L https://github.com/DisplayLink/evdi/archive/refs/tags/v1.14.1.tar.gz -o evdi.tar.gz
```

## Installing the v4l2loopback

Generate a key and put it in

Make the module load at boot using these instructions:

https://askubuntu.com/questions/1245212/how-do-i-automatically-run-modprobe-v4l2loopback-on-boot