---
tags:
  - hardware
  - computer
  - configuration
  - debian
---
## Install the OS from USB

Copy the chosen ISO file to a USB drive using the following command:

```
sudo dd if=../../../Downloads/debian-13.4.0-amd64-DVD-1.iso of=/dev/sda bs=1M status=progress
```

Run the graphical installer:
- The answers to most questions are obvious
- Put everything on a single partition
- Create a LUKS encrypted disk
## Run `workstation

## Add passwords using `password-store`

## Add configuration using `chezmoi`


`