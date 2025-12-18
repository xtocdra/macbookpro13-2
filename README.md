# State of Linux on MacBook Pro late 2016 with Touch Bar (model 13,2) T1 Chips.

- Version 1.00 as of 15 Dec 2025

The following document provides an overview about Linux support for Apple's MacBook Pro late 2016 with Touch Bar and T1 Chips (model 13,2).

The information below is with followings conditions:
- Dual boot MacOS and Linux
- OCLP (Open Core Legacy Platform) is installed.
- Arch Linux rolling-model Installation
- Linux Kernel 6.17.9
- Systemd boot

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

#### Not Working:
- Sleep & Hibernation
- Touch ID

### Some Works need to be applied especially for:

#### Audio Input & Output
The in build kernel module from Arch Linux is not working properly, <i>snd_hda_intel</i> is loaded and hooked but there is no sound.
The issue is lying on Cirrus 8409 <i>snd_hda_codec_cs8409</i> in build kernel driver and it needs to be patched and replaced.
Install the patch for Cirrus 8409 driver from https://github.com/davidjo/snd_hda_macbookpro

Follow the instruction in there.

#### Facetime HD Camera and Touch Bar
Macbook Pro late 2016 is using T1 Chip and recognized by the autodetect kernel as Apple-IBridge. But there is non kernel driver for IBridge to hook up with FacetimeHD Camera and Touch Bar. To do that a custom kernel driver needs to be installed, compiled and hooked.

Install the kernel driver from https://github.com/parport0/mbp-t1-touchbar-driver/<br>
There will be 3 drivers needed <i>apple-ibridge</i> (virtual usb to manage facetimehd and touch bar), <i>apple_ib_tb</i> for touch bar and <i>apple_ib_als</i> for the ambient light sensor.

Follow the instruction and make a systemd service for <i>apple-ibridge.service</i>
Here is an example:

```
[Unit]
Description=Activate Apple iBridge Driver (Touch Bar & Camera)
After=multi-user.target

[Service]
Type=oneshot
##### 1. Load the driver modules
ExecStart=/usr/bin/modprobe apple-ib-tb idle_timeout=-1 dim_timeout=-1 fnmode=2

##### 2. Unbind from generic usb driver and reprobe to trigger apple-ibridge binding
ExecStart=/usr/bin/bash -c 'echo -n "1-3" > /sys/bus/usb/drivers/usb/unbind'
ExecStart=/usr/bin/bash -c 'echo 1-3 > /sys/bus/usb/drivers_probe'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Enable and Start the service

Create a config file in <i>/etc/modprobe.d</i> to ignore the default <i>usbhid</i> driver to take over <i>apple-ibridge</i> and run <i>mkinitcpio -P</i> afterwards.
```
options usbhid ignore_special_drivers=1 quirks=0x05ac:0x8600:0x4
```
reboot or turn 0ff/0n, avoid any usb flash drive plugged in booting time.

If Touch Bar is still not working, check some parameters:
```
# check all the options of paramaters for the touchbar
modinfo apple-ib-tb
# check the recorded parameters
cat /sys/module/apple_ib_tb/parameters/idle_timeout
# return should be -1
cat /sys/module/apple_ib_tb/parameters/dim/timeout
# return should be -1
# or set the parameters in the systemd service to suite your preference.
```

If the Touch Bar is still not working, try to insert driver in module section on <i>mkinitcpio.conf</i>,
```
# nano /etc.mkinitcpio.conf
MODULES=(apple-ib-tb)
# save the config file and create a config file in /etc/modprobe.d
# nano ibridge.conf
options apple_ib_tb idle_timeout=-1 dim_timeout=-1
# save and run
sudo mkinitcpio -P

# edit the systemd service (apple-ibridge.service) by deleting or uncoment this parameters:
ExecStart=/usr/bin/modprobe apple-ib-tb idle_timeout=-1 dim_timeout=-1 fnmode=2
# Then update systemd
sudo systemctl daemon-reload

reboot
```

FacetimeHD is using <i>uvcvideo</i> kernel driver and it will hook to IBridge automatically.
run <i>lsusb -t</i> and the result is as follows:
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

#### WiFi
This Model is using brcmfmac as the kernel driver and it load automatically.
The problems are in the Registration Domain and the incompatibility with wpa_supplicant apps.

Install the package <i>wireless-regdb</i> and set the Region to your country:
```
iw set reg <your country>
```
edit <i>/etc/conf/wireless-regdom</i> and uncomment your region domain.

Use <i>iwd</i> to connect to wifi access point as iwd has more compatibility to Broadcom 43602 and avoid to use wpa_supplicant (incompability issue with keyphrase password).

If you are using <i>NetworkManager</i>, use <i>iwd</i> as the <i>backline</i> or install <i>networkmanager-iwd</i> package.

The Wifi is working only at 2.4G, but it is sufficient.<br>
Check Arch Wiki https://wiki.archlinux.org/title/Network_configuration/Wireless and read throughly.

#### Suspend
Follow this instruction to make the suspend mode working.
Take a look at Arch Wiki to enable the suspend mode https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate.
Follow this tips for macbook pro https://takachin.github.io/mbp2017-linux-note/en/suspend-resume.html
