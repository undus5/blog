+++
title       = 'Linux Desktop: Sway, Labwc, GUI Apps'
lastmod     = '2026-02-21'
date        = '2025-11-30'
tags        = []
showSummary = true
showTOC     = true
weight      = 1000
+++

You only need a window manager to do your work.

<!--more-->

## Series Index

1. [Linux Bootstrap Installation](/posts/linux-bootstrap-installation/)
2. [Linux A/B System Updates via BTRFS Snapshot](/posts/linux-ab-system-updates-via-btrfs-snapshot/)
3. [Linux Post Installation: Desktop Preparation](/posts/linux-post-installtion-desktop-preparation/)
4. Linux Desktop: Sway, Labwc, GUI Apps

## Preface

You don't really need a versatile desktop suite, just a window manager can get
your job done, less components, less bugs, more efficient.

If you prefer keyboard navigation, then use Sway,
if you prefer using mouse to point and click, then then Labwc.

This guide is distro independent, tested on Arch and Fedora.

## Sway Labwc

Install [Sway](https://wiki.archlinux.org/title/Sway),
[Labwc](https://wiki.archlinux.org/title/Labwc) and other essential packages:

```
sway swaylock swaybg labwc wl-clipboard wmenu mako wob kanshi grim wev
xdg-desktop-portal-gtk
```

[xdg-desktop-portal](https://wiki.archlinux.org/title/XDG_Desktop_Portal):\
xdg-desktop-portal-gtk : necessary component for e.g. file chooser.\
[wl-clipboard](https://github.com/bugaevc/wl-clipboard) : necessary for ctrl-c ctrl-v function.\
[wmenu](https://codeberg.org/adnano/wmenu) : menu for launching apps and running commands.\
[mako](https://github.com/emersion/mako) : desktop notification.\
[wob](https://github.com/francma/wob) : indicator bar for volume or brightness.\
[kanshi](https://gitlab.freedesktop.org/emersion/kanshi): dynamic output configuration.\
[grim](https://gitlab.freedesktop.org/emersion/grim) screenshot tool for wayland.\
[wev](https://git.sr.ht/~sircmpwn/wev) : detect key name, for configuring keybindings.\
[sway-contrib](https://github.com/OctopusET/sway-contrib) : grim helper for partial screenshot.

We won't discuss their configurations in detail, you can refer to the official documentations.
Instead, I will show you some basic ideas about how I use them.

For Sway, since it is keyboard driven and tiling, I bind `Super + 1/2/3/4/5/6/7/8/9/0`,
`Super + q/w/e/r/t/y/u/i/o/p`, `Super + z/x/c/v/b/n/m` to corresponding
workspaces, and `Super + Shift + ...` to move windows into them. Then I use apps
in maximum mode, one window per workspace for most of time. In this way, I need
to track which workspaces are in use, so I choose built-in swaybar to do this work.

Unfortunately, swaybar is lacking system tray support. But I think it's not a big
problem, you don't really need it, if you want some apps running in the background,
just throw them into a dedicated workspace in the corner and forget, done.

For Labwc, there isn't much to say, all the actions can be invoked from the
right-click context menu. Since it's mouse driven and floating, single workspace
is enough, you just use `Alt + Tab` to switch between windows.

That's all. Just keep it simple and clean.

## Terminal

I prefer [foot](https://codeberg.org/dnkl/foot) and
[alacritty](https://alacritty.org/),
both are simple and fast terminal emulators.

## GTK Theme

For GTK 4:

```
(user)$ gsettings set org.gnome.desktop.interface color-scheme prefer-dark
(user)$ gsettings set org.gnome.desktop.interface color-scheme default
```

For GTK 3, install `gnome-themes-extra` package:

```
(user)$ ls /usr/share/themes
(user)$ gsettings set org.gnome.desktop.interface gtk-theme Adwaita-dark
(user)$ gsettings set org.gnome.desktop.interface gtk-theme Adwaita
```

Ref: [GTK#Basic theme configuration](https://wiki.archlinux.org/title/GTK#Basic_theme_configuration)
, [GTK 3 settings on Wayland](https://github.com/swaywm/sway/wiki/GTK-3-settings-on-Wayland)

## Qt Theme

IMHO, if you're not intended to use KDE desktop environment, then avoid choosing
KDE replated components, since they are tightly coupled with the KDE framework,
lots of dependencies would be installed even for a very simple package like
[breeze-icons](https://github.com/KDE/breeze-icons/), which is annoying.
LXQt is in a similar situation.

The original
[qt6ct](https://github.com/trialuser02/qt6ct)
is archived, although there is a
[successor](https://www.opencode.net/trialuser/qt6ct), I decided not dealing with
KDE apps anymore. For other independent Qt apps, they usually work well by default,
no need tools like qt5ct/qt6ct get involved.

## Icon Theme

Install the fallback
[icon theme](https://wiki.archlinux.org/title/Icons) `hicolor-icon-theme`.

If you want to add custom icons, create `~/.local/share/icons/hicolor/`, put
your icons into corresponding size directories such as `.../hicolor/128x128/apps/`.

Ref: [Icon Theme Specification](https://specifications.freedesktop.org/icon-theme/latest/#directory_layout).

If you want to use Breeze icon theme, just download the repo manually:

```
(user)$ git clone https://github.com/KDE/breeze-icons ~/Downloads
(user)$ cp -r ~/Downloads/breeze-icons/icons ~/.local/share/icons/Breeze
(user)$ cd ~/.local/share/icons/Breeze
(user)$ cat breeze.theme.in commonthemeinfo.theme.in > index.theme
```

Change GTK icon theme

```
(user)$ ls /usr/share/icons
(user)$ gsettings set org.gnome.desktop.interface icon-theme Breeze
```

## File Manager

Nautilus aka [GNOME/Files](https://wiki.archlinux.org/title/GNOME/Files) is a
good one.

Packages for Arch and Fedora: `nautilus ifuse gvfs gvfs-mtp gvfs-gphoto2 gvfs-afc`

[GVFS](https://wiki.archlinux.org/title/File_manager_functionality#Mounting)
is for auto mounting usb drives, mobile devices and trash functionality.

`ifuse gvfs-gphoto2 gvfs-afc` are for
[iOS](https://wiki.archlinux.org/title/IOS) device support. There's a
[glitch](https://sporks.space/2024/09/20/accessing-iphone-photos-and-media-from-nautilus-on-linux/)
after pluging in the iOS device, you can only see the virtual filesystem for iOS
apps, not for photos. To fix this, first open that virtual filesystem for apps,
the URL in the address bar is like `afc://<URL>:3`, change `:3` to `:1` and
press Enter, now you switch to the virtual filesystem for photos.

Set nautilus as default file manager:

```
xdg-mime default org.gnome.Nautilus.desktop inode/directory
```

When you use "Open With" to open file with some app, it will invoke app's desktop
entry, when the app is a command line app, there's a key-value `Terminal=true`
in its desktop entry file, for example, you want to open a text file with
neovim, Nautilus detected this 'Terminal=true' and would try to run it in the
"default terminal", then how does this default terminal being determined?
I found it from gsettings:

```
(user)$ gsettings get org.gnome.desktop.default-applications.terminal exec
```

It returns `xdg-terminal-exec`, this is the right executable, but this gsettings
key-value is not the one which affect "Open With" behavior.
I've done some experiments, found it seems hard coded, Nautilus always try to
invoke xdg-terminal-exec even when I changing this `exec` value to another executable.
Fortunately, xdg-terminal-exec is maintained as a seperated package,
we could choose not to install it and write our own for simplicity,
just create a script `/usr/local/bin/xdg-terminal-exec` with:

```
#!/bin/bash
foot "${@}"
# alacritty -e "${@}"
```

There's another missing feature when using Nautilus out of GNOME, which is
"open terminal here", and it can be implemented via Nautilus's built-in function
[Custom Scripts](https://wiki.archlinux.org/title/GNOME/Files#Custom_scripts),
but there're some inconvenience in this way, you need to right click on the folder,
select the script from context menu, which means you need to take this action
in the parent folder, which is counter-intuitive for "open terminal here".
Normally we want to do this by right-clicking on the blank inside the target folder,
so "Custom Scripts" is not a good choice here, instead, I recommend using "Open With"
to implement this function, here's how:

Create `/usr/local/bin/open-terminal-here.sh` with:

```
#!/bin/bash
abspath=$(realpath "${1}")
if [[ -d "${abspath}" ]]; then
    foot -D "${abspath}"
    # alacritty --working-directory "${abspath}"
else
    foot
    # alacritty
fi
```

Create `~/.local/share/applications/open-terminal-here.desktop` with:

```
[Desktop Entry]
Name=Open Terminal Here
Exec=open-terminal-here.sh %F
Icon=foot
Terminal=false
Type=Application
Categories=System;TerminalEmulator;
Keywords=shell;prompt;command;commandline;
MimeType=inode/directory;
```

Apply change:

```
(user)$ update-desktop-database ~/.local/share/applications
```

You may also want to disable "Recent Files":

```
(user)$ gsettings set org.gnome.desktop.privacy remember-recent-files false
```

## Zip/Unzip

I recommend [PeaZip](https://peazip.github.io/) as GUI archive manager.

Install dependency package `qt6pas` first.
Then download peazip tarball and extract to, say `~/apps/peazip`,
then use `~/apps/peazip/res/share/batch/freedesktop_integration/peazip.desktop`
as template to create desktop entries under `~/.local/share/applications/`:

`peazip-extract-newfolder.desktop`:

```
Name=PeaZip Extract Smart
Exec=bash -c '~/apps/peazip/peazip -ext2folder %F'
Icon=peazip_extract
```

`peazip-add-archive.desktop`:

```
Name=PeaZip Add Archive
Exec=bash -c '~/apps/peazip/peazip -add2archive %F'
Icon=peazip_add
MimeType=application/octet-stream;
```

`Exec=` needs absolute path if provided, but here we did a little trick to avoid
hardcoding user home directory, by using
[bash(1)](https://man.archlinux.org/man/bash.1) to expand `~`.

Ref: [Desktop Entry Specification](https://specifications.freedesktop.org/desktop-entry/latest/exec-variables.html)

There're more desktop entry examples in
`~/apps/peazip/res/share/batch/freedesktop_integration/additional-desktop-files`.

Apply change:

```
(user)$ update-desktop-database ~/.local/share/applications
```

Ref: [Desktop entries](https://wiki.archlinux.org/title/Desktop_entries)

Also recommend installing `7zip` package for zip/unzip in command line.

## Policykit

Tools like [Ventoy](https://www.ventoy.net/) need
[polkit](https://wiki.archlinux.org/title/Polkit)
to evaluate privilege.

Packages for Arch and Fedora: `polkit mate-polkit`

The executable needs to be added into autostart script for Sway and Labwc.

Excutable for Arch: `/usr/lib/mate-polkit/polkit-mate-authentication-agent-1`\
Excutable for Fedora: `/usr/libexec/polkit-mate-authentication-agent-1`

## Input Method

For input method, I use [Fcitx5](https://fcitx-im.org/wiki/Fcitx_5) and
[RIME](https://rime.im).
Here is my RIME configs for Wubi86:
[rime-wubi86s](https://github.com/undus5/rime-wubi86s).

Packages for Arch and Fedora:
`fcitx5 fcitx5-gtk fcitx5-qt fcitx5-configtool fcitx5-rime`

Add environment variables to `.bashrc`, then relogin user:

```
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

The launching command `fcitx5 -d -r` needs
to be added into autostart script for Sway and Labwc.

Ref: [Fcitx5 - ArchWiki](https://wiki.archlinux.org/title/Fcitx5)

## Other Apps

| Category | Arch | Fedora |
| --- | --- | --- |
| Audio Control | pavucontrol | - |
| PDF | zathura zathura-pdf-poppler | - |
| Image Viewer | swayimg | - |
| Video Player | mpv | - |
| Ebook Reader | foliate | - |
| Audiobook Player | [cozy](https://cozy.sh/) | - |
| Text to QR Code | qrencode | - |
| QR Code to Text | zbar | zbar-tools |
| Web Browser | chromium [brave](https://brave.com/linux/) | - |

Brave disable Crypto and AI related components via
[Group Policy](https://support.brave.com/hc/en-us/articles/360039248271-Group-Policy):

Create `/etc/brave/policies/managed/brave-policy.json` :

```
{
  "BraveAIChatEnabled": true,
  "BraveRewardsDisabled": true,
  "BraveWalletDisabled": true,
  "BraveNewsDisabled": true,
  "BraveTalkDisabled": true,
  "BraveVPNDisabled": 1
}
```

Visit `brave://policy` from address bar to check the effect.

For more apps, refer to: [Useful add ons for sway](https://github.com/swaywm/sway/wiki/Useful-add-ons-for-sway).

