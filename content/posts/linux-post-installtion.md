+++
title       = 'Linux Post Installation'
subtitle    = ''
lastmod     = '2026-05-10'
date        = '2025-11-26'
tags        = []
showSummary = true
showTOC     = true
weight      = 1000
+++

Essential configurations before desktop components getting involved.

<!--more-->

## Series Index

1. [Linux Bootstrap Installation](/posts/linux-bootstrap-installation/)
2. [Linux A/B System Updates via BTRFS Snapshot](/posts/linux-ab-system-updates-via-btrfs-snapshot/)
3. Linux Post Installation
4. [Linux Desktop: Sway, Labwc, GUI Apps](/posts/linux-desktop-sway-labwc-gui-apps/)

## Preface

This guide is distro independent, tested on Arch and Fedora.

## Default Editor

```
(root)# echo "export EDITOR=/usr/bin/nvim" > /etc/profile.d/default-editor.sh
```

You could replace `nvim` with whatever you like.

## Console Fonts

Install package:

Arch: terminus-fonts\
Fedora: terminus-fonts-console

```
(root)# echo "FONT=ter-120b" >> /etc/vconsole.conf
```

Full font list:

Arch: `ls /usr/share/kbd/consolefonts/`\
Fedora: `ls /usr/lib/kbd/consolefonts/`

Change font temporally: `setfont <font_name>`

Ref: [Linux_console#Fonts](https://wiki.archlinux.org/title/Linux_console#Fonts)

## PipeWire

Install [PipeWire](https://wiki.archlinux.org/title/PipeWire) related packages:

Arch: `pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber`\
Fedora: `pipewire pipewire-utils`

## Bluetooth

Install [Bluetooth](https://wiki.archlinux.org/title/Bluetooth) related packages:

Arch: `bluez bluez-utils`\
Fedora: `bluez bluez-tools`

Enable systemd service: `systemctl enable --now bluetooth.service`.

## Printer

Install [CUPS](https://wiki.archlinux.org/title/CUPS) related packages:

Arch, Fedora: `cups cups-pdf`

Enable systemd service: `systemctl enable --now cups.service`.

The CUPS server can be fully administered through the web interface,
and there’s documentation for adding printer
[http://localhost:631/help/admin.html](http://localhost:631/help/admin.html).

## GPU Drivers

To choose [GPU](https://wiki.archlinux.org/title/Graphics_processing_unit) for
Linux, AMD is still the prefered option for stability and performance,
but for daily use, it doesn't matter. I've learned that Nvidia driver is getting
solid enough these days, and Intel is catching up too.

Install `mesa` and `vulkan` related packages:

Arch AMD: `mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon`\
Arch Intel: `mesa lib32-mesa vulkan-intel lib32-vulkan-intel intel-media-driver`\
Arch Nvidia: `nvidia-open` for GPU newer than GTX 1650

For Fedora, need to enable [RPM Fusion](https://rpmfusion.org/)
then install freeworld packages:

```
(root)# dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-{free,nonfree}-release-$(rpm -E %fedora).noarch.rpm 
# for AMD
(root)# dnf install mesa-va-drivers-freeworld
# for Intel
(root)# dnf install intel-media-driver
# for Nvidia newer than GTX 1650
(root)# dnf install akmod-nvidia
```

Ref: [Multimedia on Fedora](https://rpmfusion.org/Howto/Multimedia)
, [NVIDIA on Fedora](https://rpmfusion.org/Howto/NVIDIA)

Use [mpv](https://wiki.archlinux.org/title/Mpv#Hardware_video_acceleration)
to test
[hardware acceleration](https://wiki.archlinux.org/title/Hardware_video_acceleration)
, with command `mpv --hwdec=auto <videofile>`

## Disable Watchdogs

This setting is for
[improving performance](https://wiki.archlinux.org/title/Improving_performance#Watchdogs).

Check for a hardware watchdog module:

```
(root)# lsmod | grep wdt
```

Add to
[kernel module blacklist](https://wiki.archlinux.org/title/Kernel_module#Blacklisting):

```
(root)# cat > /etc/modprobe.d/nowdt.conf << EOB
blacklist iTCO_wdt
blacklist sp5100_tco
blacklist intel_oc_wdt
EOB
```

## Console Caps Ctrl

Remap `CapsLock` to `Ctrl` for console.

```
(root)# cd /usr/share/kbd/keymaps/i386/qwerty
(root)# gzip -dc < us.map.gz > usa.map
(root)# sed -i '/^keycode[[:space:]]58/c\keycode 58 = Control' usa.map
(root)# echo "KEYMAP=usa" >> /etc/vconsole.conf
```

Ref: [Linux_console/Keyboard_configuration#Creating_a_custom_keymap](https://wiki.archlinux.org/title/Linux_console/Keyboard_configuration#Creating_a_custom_keymap)

