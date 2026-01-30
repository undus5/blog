+++
title       = 'Linux Post Installation: Desktop Preparation'
subtitle    = ''
lastmod     = '2025-11-30'
date        = '2025-11-26'
tags        = []
showSummary = true
showTOC     = true
weight      = 1000
+++

Essential preparation before desktop components getting involved.

<!--more-->

## Series Index

1. [Linux Bootstrap Installation](/posts/linux-bootstrap-installation/)
2. [Linux A/B System Updates via BTRFS Snapshot](/posts/linux-ab-system-updates-via-btrfs-snapshot/)
3. Linux Post Installation: Desktop Preparation
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
(root)# echo "FONT=ter-122b" >> /etc/vconsole.conf
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

![Linus Torvalds Fuck You Nvidia](/images/linus-torvalds-fuck-you-nvidia.webp)

I only use
[AMD GPU](https://wiki.archlinux.org/title/AMDGPU)
and
[Intel GPU](https://wiki.archlinux.org/title/Intel_graphics)
on Linux for the well known reasons.

Install `mesa` and `vulkan` related packages:

Arch AMD: `mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon`\
Arch Intel: `mesa lib32-mesa vulkan-intel lib32-vulkan-intel intel-media-driver`

For Fedora, it seems these drivers and firmwares are bundled with core package
group.

## GUI Fonts

Install Noto fonts related packages:

Arch: `noto-fonts noto-fonts-cjk noto-fonts-emoji`

Fedora:
```
google-noto-fonts-all google-noto-color-emoji-fonts
google-noto-sans-cjk-fonts google-noto-serif-cjk-fonts
```

The default lookup order for CJK fonts would pick wrong characters in some cases,
such as “复” in chinese word “复制”.
To fix this, adjust fallback font order by creating `/etc/fonts/local.conf` with:

```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
<alias>
    <family>sans-serif</family>
    <prefer>
        <family>Noto Sans</family>
        <family>Noto Sans CJK SC</family>
        <family>Noto Sans CJK TC</family>
        <family>Noto Sans CJK HK</family>
        <family>Noto Sans CJK JP</family>
        <family>Noto Sans CJK KR</family>
    </prefer>
</alias>
<alias>
    <family>serif</family>
    <prefer>
        <family>Noto Serif</family>
        <family>Noto Serif CJK SC</family>
        <family>Noto Serif CJK TC</family>
        <family>Noto Serif CJK HK</family>
        <family>Noto Serif CJK JP</family>
        <family>Noto Serif CJK KR</family>
    </prefer>
</alias>
<alias>
    <family>monospace</family>
    <prefer>
        <family>Noto Sans Mono</family>
        <family>Noto Sans Mono CJK SC</family>
        <family>Noto Sans Mono CJK TC</family>
        <family>Noto Sans Mono CJK HK</family>
        <family>Noto Sans Mono CJK JP</family>
        <family>Noto Sans Mono CJK KR</family>
    </prefer>
</alias>
</fontconfig>
```

Later you could create `~/.config/fontconfig/fonts.conf` with same format under
your user home directory to overwrite this configuration,
replace with custom fonts under `~/.local/share/fonts`.

Ref: [Font configuration#Fontconfig configuration](https://wiki.archlinux.org/title/Font_configuration#Fontconfig_configuration)
, [Font configuration#Alias](https://wiki.archlinux.org/title/Font_configuration#Alias)

## Regular User

Install [xdg-user-dirs](https://wiki.archlinux.org/title/XDG_user_directories)
package, it's for managing well known user directories
e.g. Desktop, Documents, Downloads etc.

Create regular user:

```
(root)# useradd -G wheel user1
(root)# passwd user1
```

`wheel` is the superuser group for sudo in Arch and Fedora, for Debian,
it's named `sudo`.

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

