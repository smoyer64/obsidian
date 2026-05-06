---
tags:
  - computer
  - configuration
  - dell
  - hardware
  - anna
---

  ## Installing Xubuntu

- Shrink the Windows OS partition and switch the BIOS from RAID to AHCI following the instructions at https://medium.com/@tylergwlum/my-journey-installing-ubuntu-18-04-on-the-dell-xps-15-7590-2019-756f738a6447.
- Boot into Xubuntu live CD and perform the installation.
- Remove the snap package manager using the following commands:

`apt autoremove --purge snapd gnome-software-plugin-snap` followed by 'rm -rf ~/snap' to remove applications which might have already been installed. See https://cialu.net/a-better-ubuntu-linux-without-the-crappy-snap/ for rationale.

## Security, GPG, SSH and passwords

- Encrypt your home directory following the instructions at https://www.howtogeek.com/116032/how-to-encrypt-your-home-folder-after-installing-ubuntu/.
- Add an SSH key using `ssh-keygen -t ed25519 -C "smoyer1@selesy.com"`

## Installing Git, Connect to Github and run selesy/workstation

- Install Git using `sudo apt install git`
- Add your SSH key to Github using the instructions at https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

---

**This computer was given to Anna**