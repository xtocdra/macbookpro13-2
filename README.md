# State of Linux on the MacBook Pro late 2016 with Touch Bar (model 13,2) T1 Chip

Version 1.00 as of 15 Dec 2025
The following document provides an overview about Linux support for Apple's MacBook Pro late 2016 with Touch Bar model and T1 chip.

The information below is with followings conditions:
- Arch Linux Rolling-model Installation
- Linux Kernel 6.17.9

Current Status:
Audio Input & Output: Working (Additional patch required)
FaceTimeHD Camera: Working (Additional patch required)
Touch Bar: Working (Additional patch required)
Suspend: Working (Additional setup required)
WiFi: Working (Additional setup required)
Battery: Working
Bluetooth: Working
Graphic Card: Working
Keyboard & Touchpad: Working
NVME SSD: Working
Screen: Working
System Management Controller: Working
Thunderbolt: Working
USB: Working

Not Working:
Sleep & Hibernation
Touch ID

Some Works need to be applied especially for:
Audio Input & Output
The in build kernel module from Arch Linux is not working, there is no output on sound.
The issue is lying on the Cirrus 8409 kernel driver and it needs to be patched.
Install the patch for Cirrus 8409 driver from https://github.com/davidjo/snd_hda_macbookpro
Follow the instruction in there.

Facetime HD Camera and Touch Bar
Macbook Pro late 2016 is using T1 Chip and recognized by the autodetect kernel as Apple-IBridge. But there are no kernel driver for IBridge to hook up with FacetimeHD Camera and Touch Bar. To do that a custom kernel driver needs to be installed, compiled and hooked.
Install the kernel driver from https://github.com/parport0/mbp-t1-touchbar-driver/
Follow the instruction there and make it a systemd service for apple-ibridge.service.

FacetimeHD is using uvcvideo kernel driver, which hooks to apple-ibridge.
As long as apple-ibridge driver is installed and loaded, uvcvideo will hook to IBridge automatically.

WiFi
This Model is using brcmfmac as the kernel driver and it load automatically.
The issue is in the Registration Domain and the incompatibility with wpa_supplicant apps.
Try to use iwd instead of wpa_supplicant.
If you are using NetworkManager, try to hook to iwd as the backline.
Install the package wireless-db and set the Region to your country.
The Wifi is only at 2.4G working, but it is sufficient.
Check Arch Wiki https://wiki.archlinux.org/title/Network_configuration/Wireless and read throughly.

