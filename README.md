# State of Linux on the MacBook Pro late 2016 with Touch Bar (model 13,2) T1 Chip

- Version 1.00 as of 15 Dec 2025

The following document provides an overview about Linux support for Apple's MacBook Pro late 2016 with Touch Bar and T1 chip (model 13,2).

The information below is with followings conditions:
- Arch Linux Rolling-model Installation
- Linux Kernel 6.17.9

## Current Status:
- Battery: Working
- Bluetooth: Working
- Graphic Card: Working
- Keyboard & Touchpad: Working
- NVME SSD: Working
- Screen: Working
- System Management Controller: Working
- Thunderbolt: Working
- USB: Working
- Audio Input & Output: Working (Additional patch required)
- FaceTimeHD Camera: Working (Additional patch required)
- Touch Bar: Working (Additional patch required)
- Suspend: Working (Additional setup required)
- WiFi: Working (Additional setup required)

### Not Working:
- Sleep & Hibernation
- Touch ID

## Some Works need to be applied especially for:
### Audio Input & Output
The in build kernel module from Arch Linux is not working, there is no output on sound.
The issue is lying on the Cirrus 8409 kernel driver and it needs to be patched.
Install the patch for Cirrus 8409 driver from https://github.com/davidjo/snd_hda_macbookpro

Follow the instruction in there.

### Facetime HD Camera and Touch Bar
Macbook Pro late 2016 is using T1 Chip and recognized by the autodetect kernel as Apple-IBridge. But there is non kernel driver for IBridge to hook up with FacetimeHD Camera and Touch Bar. To do that a custom kernel driver needs to be installed, compiled and hooked.

Install the kernel driver from https://github.com/parport0/mbp-t1-touchbar-driver/<br>
There will be 3 drivers needed apple-ibridge (virtual usb to manage facetimehd and touch bar), apple_ib_tb for touch bar and apple_ib_als for the ambient light sensor.

Follow the instruction there and make it a systemd service for apple-ibridge.service.
Here is an example:

```
[Unit]
Description=Activate Apple iBridge Driver (Touch Bar & Camera)
After=multi-user.target

[Service]
Type=oneshot
##### 1. Load the driver modules
ExecStart=/usr/bin/modprobe apple-ibridge
ExecStart=/usr/bin/modprobe apple-ib-tb idle_timeout=-1 dim_timeout=-1 fnmode=2
#ExecStart=/usr/bin/modprobe apple-ib-tb

##### 2. Unbind from generic usb driver and reprobe to trigger apple-ibridge binding
ExecStart=/usr/bin/bash -c 'echo -n "1-3" > /sys/bus/usb/drivers/usb/unbind'
ExecStart=/usr/bin/bash -c 'echo 1-3 > /sys/bus/usb/drivers_probe'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Enable and Start the service

Create a blacklist conf file in /etc/modprobe.d to ignore the default usbhid driver to take over apple-ibridge and run mkinitcpio.
```
options usbhid ignore_special_drivers=1 quirks=0x05ac:0x8600:0x4
```
reboot or turn 0ff/0n, avoid any usb flash drive plugged.

FacetimeHD is using uvcvideo kernel driver and it will hook to IBridge automatically.
run lsusb -t and the result is as follows:
```
/:  Bus 001.Port 001: Dev 001, Class=root_hub, Driver=xhci_hcd/12p, 480M
    |__ Port 003: Dev 002, If 0, Class=Video, Driver=uvcvideo, 480M
    |__ Port 003: Dev 002, If 1, Class=Video, Driver=uvcvideo, 480M
    |__ Port 003: Dev 002, If 2, Class=Human Interface Device, Driver=usbhid, 480M
    |__ Port 003: Dev 002, If 3, Class=Human Interface Device, Driver=usbhid, 480M
/:  Bus 002.Port 001: Dev 001, Class=root_hub, Driver=xhci_hcd/6p, 5000M
/:  Bus 003.Port 001: Dev 001, Class=root_hub, Driver=xhci_hcd/2p, 480M
/:  Bus 004.Port 001: Dev 001, Class=root_hub, Driver=xhci_hcd/2p, 10000M
/:  Bus 005.Port 001: Dev 001, Class=root_hub, Driver=xhci_hcd/2p, 480M
/:  Bus 006.Port 001: Dev 001, Class=root_hub, Driver=xhci_hcd/2p, 10000M

```

As long as apple-ibridge driver is installed and loaded, uvcvideo will hook to IBridge automatically.

### WiFi
This Model is using brcmfmac as the kernel driver and it load automatically.
The issue is in the Registration Domain and the incompatibility with wpa_supplicant apps.<br>
Try to use iwd instead of wpa_supplicant.<br>
If you are using NetworkManager, use iwd as the backline or install networkmanager-iwd package.<br>
Install the package wireless-db and set the Region to your country.<br>
The Wifi is working only at 2.4G, but it is sufficient.<br>
Check Arch Wiki https://wiki.archlinux.org/title/Network_configuration/Wireless and read throughly.

### Suspend
Follow this instruction to make the suspend mode working.
Take a look at Arch Wiki to enable the suspend mode https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate.
Follow this tips for macbook pro https://takachin.github.io/mbp2017-linux-note/en/suspend-resume.html
