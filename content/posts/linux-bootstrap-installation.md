+++
title       = 'Linux Bootstrap Installation'
aliases     = ["/posts/bootstrap-install-any-linux-distro/"]
lastmod     = '2026-02-19'
date        = '2025-10-19'
tags        = []
showTOC     = true
showSummary = true
weight      = 1000
+++

Distro is trival, just learn the basics and build your own.

<!--more-->

## Series Index

1. Linux Bootstrap Installation
2. [Linux A/B System Updates via BTRFS Snapshot](/posts/linux-ab-system-updates-via-btrfs-snapshot/)
3. [Linux Post Installation](/posts/linux-post-installtion/)
4. [Linux Desktop: Sway, Labwc, GUI Apps](/posts/linux-desktop-sway-labwc-gui-apps/)

## Preface

Want to stop distro hopping? Sure, just go read through the
[ArchWiki](https://wiki.archlinux.org/title/Main_page). Don't get me wrong,
I'm not selling Arch Linux to you, just the wiki. The reason you always be 
distracted by shining fancy components from some new releases
or emerging distros, is that you don't have a big picture about linux and
its ecosystem, and you haven't figure out what do you really need.
This is the most thing I learned after reading through the wiki.
When I finished reading, I learned what choices are there, and found my needs,
then built my own configurations.

In fact, you can install nearly all the linux distros manually in a similar way,
aka "the arch way", since the installers they offered are doing the same job
under the hood, but with less flexibility and more bloat. So if you want to
settle down, go read the wiki, learn the basics, then you can pick up whatever
distros you like and tweaking them to the shape you want, the only differences
are just package management system, release model and community support.

This guide is basically distro independent, since I don't like to be bound to
any specific platform in any form, always avoid using any distro specific tools,
try my best to maintain the portability. It's tested on Arch, Fedora and
Debian, the differences are minor.

## Live ISO

You need a live iso image to boot into live system for doing installation. 

[Arch](https://archlinux.org/download/),
[Fedora](https://www.fedoraproject.org/),
[Debian](https://www.debian.org/)

To create bootable USB stick, use [Ventoy](https://www.ventoy.net/en/index.html)
or [Rufus](https://rufus.ie/en/).

## Partition Disk

In my experience, the best partition practice is to create a separate
[EFI system partition](https://wiki.archlinux.org/title/EFI_system_partition)
and a root partition with BTRFS filesystem, we will discuss BTRFS later.

We will follow the
[Discoverable Partitions Specification](https://uapi-group.org/specifications/specs/discoverable_partitions_specification/)
to let systemd automatically discover, mount and enable the root partition
based on GPT (GUID Partition Tables), by specifying dedicated UUIDs to partitions.

Using [Parted](https://wiki.archlinux.org/title/Parted) to do the job:

```
(root)# parted /dev/nvme0n1
(parted) mklabel gpt
(parted) mkpart EFIPART fat32 1MiB 2049MiB
(parted) set 1 esp on
(parted) mkpart ROOTPART btrfs 2049MiB 100%
(parted) type 2 4f68bce3-e8cd-4db1-96e7-fbcaf984b709
(parted) quit
```

Partitions need to be aligned to specific size for LUKS working correctly,
a typical practice is 1MiB. In our example, we assigned 2GB to the EFI partition,
the rest to the root partition, and aligned them to 1MiB.

Note that we specified the discoverable UUID for the root partition, but not for
the EFI partition, because it is done by the `esp on` commands.

## LUKS

Next we set [Device Encryption](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption).
Even you don't want to bother typing another passphrase beside system
login password every time when booting up,
I still strongly recommend to configure it with a preset key file
(we will discuss this in later section), which do not
need you to enter passphrase, it will do decrypting automatically using the key file.
Although it is missing the point of disk encryption in this way, it will still
be beneficial, if your laptop is lost, this encryption
setting can prevent your data being read from random people, since
there's low possibility the person who picked your device is computer expert
with knowledges about Linux and LUKS.

You still need to set a passphrase when configuring LUKS,
save it carefully, may be use a password manager to store it,
[KeePass](https://wiki.archlinux.org/title/KeePass) is a good one.

After partitioning, you can locate partitions via their labels:

```
(root)# cryptsetup luksFormat /dev/disk/by-partlabel/ROOTPART
(root)# cryptsetup open /dev/disk/by-partlabel/ROOTPART root
(root)# ls /dev/mapper/root
```

Now the `ROOTPART` is encrypted, and must be decrypted via `cryptsetup open`
command to let it work, `/dev/mapper/root` is the decrypted root partition,
we will create filesystem on top of it.

## BTRFS

Ext4 may be the most solid filesystem, but
[BTRFS](https://wiki.archlinux.org/title/Btrfs)
is a better choice for personal use because of its modern features.

Recall the early days when I was learning and tinkering with linux, It was always
a hard decision on how much storage to allocate for the separate /home partition,
today with BTRFS, it's not a problem anymore. You could just create BTRFS subvolumes
for whatever directories you want to separate, they will share the whole storage
of the root partition, just like normal folders do.

```
(root)# mkfs.btrfs /dev/mapper/root
(root)# mount /dev/mapper/root /mnt
(root)# btrfs subvolume create /mnt/@
(root)# btrfs subvolume create /mnt/@home
(root)# umount /mnt

(root)# mount -o subvol=@ /dev/mapper/root /mnt
(root)# mount --mkdir -o subvol=@home /dev/mapper/root /mnt/home
(root)# mkfs.fat -F32 /dev/disk/by-partlabel/EFIPART
(root)# mount --mkdir /dev/disk/by-partlabel/EFIPART /mnt/efi
```

Another great BTRFS feature is it's easy to create snapshots by its
Copy on Write (CoW) nature, useful for creating backup against system crash.

## Repo Mirror

Check the online mirrorlist, pick proper one, then edit local mirrorlist config.

Arch: [mirrorlist](https://archlinux.org/mirrorlist/),
local: `/etc/pacman.d/mirrorlist`\
Fedora: [mirrorlist](https://mirrormanager.fedoraproject.org),
local: `/etc/yum.repos.d/fedora.repo`\
Debian: [mirrorlist](https://www.debian.org/mirror/list),
local: `/etc/apt/sources.list`

## Mount VFS

Mount virtual filesystems to `/mnt` then chroot into it :

```
(root)# for dir in dev proc run sys; do \
    mount --mkdir --rbind --make-rslave /$dir /mnt/$dir; done
```

## Base System

Now we are ready to install the base system. We will use a dedicated tool
to install base system packages into /mnt.

For Arch it's
[Pacstrap](https://wiki.archlinux.org/title/Installation_guide#Install_essential_packages):

```
(root)# pacstrap -K /mnt \
    base linux linux-firmware btrfs-progs dracut zram-generator vim \
    amd-ucode intel-ucode
```

For Fedora it's DNF:

```
(root)# dnf --use-host-config --releasever=43 --installroot=/mnt group install core
(root)# dnf --use-host-config --releasever=43 --installroot=/mnt install \
    kernel linux-firmware dracut zram-generator systemd-boot cryptsetup \
    glibc-langpack-en btrfs-progs vim amd-ucode-firmware
```

For Debian it's [Debootstrap](https://wiki.debian.org/Debootstrap):

```
(root)# debootstrap --include=\
    linux-image-amd64,non-free-firmware,dracut,systemd-zram-generator,\
    systemd-boot,cryptsetup,btrfs-progs,vim,amd64-microcode,intel-microcode \
    stable /mnt http://deb.debian.org/debian/
```

## Fstab

Let's generate the [fstab](https://wiki.archlinux.org/title/Fstab)
before entering chroot system, since we need to get partition UUIDs from
live system. First we write partition UUIDs
to temporary text files using `blkid` command for later use, because typing UUID
manually is annoying and error prone, note the UUID for the root partition must
be the decrypted one, which is `/dev/mapper/root`, not the `ROOTPART`.

```
(root)# blkid -s UUID -o value /dev/mapper/root > /tmp/rootuuid.txt
(root)# blkid -s UUID -o value /dev/disk/by-partlabel/EFIPART > /tmp/efiuuid.txt
```

Then we edit `/mnt/etc/fstab`. If you use `nano` text editor, you can press
`Ctrl + r` to read UUID from temporary file, or if you use `vim`, you can run
`:r /tpm/rootuuid.txt` command to read UUID.

```
UUID=xxxxxxxx-...-xxxxxxxxxxxx /     btrfs compress=zstd,subvol=/@     0 0
UUID=xxxxxxxx-...-xxxxxxxxxxxx /home btrfs compress=zstd,subvol=/@home 0 0
UUID=XXXX-XXXX /efi vfat defaults 0 0
```

## Chroot

```
(root)# chroot /mnt /bin/bash
```

## Timezone

```
(root)# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## Localization

For Arch and Debian:

```
(root)# echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
(root)# echo "zh_CN.UTF-8 UTF-8" >> /etc/locale.gen
(root)# locale-gen
```

For Fedora, they are provided by packages `glibc-langpack-en`, `glibc-langpack-zh`

Set LANG variable:

```
(root)# echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

## Hostname

```
(root)# echo "archlinux" > /etc/hostname
```

## Root Password

```
(root)# passwd
```

## Zram

We have large memory storage nowadays, so just put the swap into RAM using
[zram](https://wiki.archlinux.org/title/Zram).

Create `/etc/systemd/zram-generator.conf`, the size is in MiB.

```
[zram0]
zram-size = min(ram, 8192)
compression-algorithm = zstd
```

## Systemd-Networkd

In my experience,
[systemd-networkd](https://wiki.archlinux.org/title/Systemd-networkd)
is better than NetworkManager especially for maintaining bridged network
interfaces for virtual machines.

Run `ip link show` command to get your network interface names,
for example: enp0s1, wlan0.

For wired network interface, create `/etc/systemd/network/23-lan.network`.

```
[Match]
Name=en*
[Link]
RequiredForOnline=routable
[Network]
DHCP=yes
[DHCPv4]
RouteMetric=100
[IPv6AcceptRA]
RouteMetric=100
```

For wireless network interface, create `/etc/systemd/network/25-wlan.network`.

```
[Match]
Name=wl*
[Link]
RequiredForOnline=routable
[Network]
DHCP=yes
IgnoreCarrierLoss=3s
[DHCPv4]
RouteMetric=600
[IPv6AcceptRA]
RouteMetric=600
```

Setting unique `RouteMetric` for different network interfaces is necessary,
or they will enter into "race condition", which will cause extreamly slow network
connections.

`RequiredForOnline=routable` is necessary to prevent
`systemd-networkd-wait-online.service` hanging the systemd boot process.

By default systemd would wait for all the network interfaces online,
if your computer have multiple network interfaces, it would be a problem, 
systemd services with `Requires=network-online.target` would not start properly.
To fix this, alter the `systemd-networkd-wait-online.service` by creating
`/etc/systemd/system/systemd-networkd-wait-online.service.d/override.conf` manually
or run `systemctl edit systemd-networkd-wait-online.service`:

```
[Service]
ExecStart=
ExecStart=/usr/lib/systemd/systemd-networkd-wait-online --any
```

You may need to disable `ManageForeignRoutingPolicyRules` option in
`/etc/systemd/networkd.conf`, since it will flush all your custom
rules that are not configured in `.network` units, such as the rules added
by `ip rule` command.

```
[Network]
ManageForeignRoutingPolicyRules=no
```

Enable the services.

```
(root)# systemctl enable systemd-networkd.service
```

Debian need to move out `/etc/network/interfaces` according to
[SystemdNetworkd - Debian Wiki](https://wiki.debian.org/SystemdNetworkd)

```
(root)# mv /etc/network/interfaces /etc/network/interfaces.old
```

## Systemd-Boot

[Install UEFI boot manager](https://wiki.archlinux.org/title/Systemd-boot#Installing_the_UEFI_boot_manager).

```
(root)# bootctl install
(root)# systemctl enable systemd-boot-update.service
```

Create bootloader entry `/efi/loader/entries/linux.conf`.

```
title Arch Linux
linux /linux/vmlinuz
initrd /linux/initrd
options rootflags=subvol=@ quiet splash
```

`/linux/vmlinuz` is actually pointed to `/efi/linux/vmlinuz`, since the path
is relative to the root of your
[EFI system partition](https://wiki.archlinux.org/title/EFI_system_partition)
, which is `/efi` in this guide. We will put kernel image and initramfs image
into `/efi/linux/` via dracut configuation in the next section.

To use
[BTRFS subvolume as root](https://wiki.archlinux.org/title/Btrfs#Mounting_subvolume_as_root)
mountpoint, use kernel parameter `rootflags=subvol=@`,
or you would get an error "Failed to start Switch Root" when booting up.

Set default boot entry in `/efi/loader/loader.conf`.

```
default linux.conf
timeout 0
editor no
```

`timeout 0` means the boot menu will not be displayed by default,
and the system will immediately boot into the default entry.
To reveal the boot menu in this scenario, a key needs to be pressed and
held down during the boot process, before systemd-boot initializes.
The recommended key for this action is the space bar.
Other keys may also work, but space bar is widely suggested.

Note: If disk partitions were not following the
[Discoverable Partitions Specification](https://uapi-group.org/specifications/specs/discoverable_partitions_specification/)
, which means root partition would not be discovered and auto mounted,
booting system would stuck at
`a start job is running for /dev/gpt-auto-root` and timeout.
To fix this,
[name root partition in kernel parameters](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#Configuring_the_boot_loader)
using [rd.luks.name](https://wiki.archlinux.org/title/Dm-crypt/System_configuration#rd.luks.name).

```
options rd.luks.name=<UUID>=root root=/dev/mapper/root rootflags=subvol=@
```

For Fedora and Debian there's one more step, by default they will trigger a
systemd [kernel-install(8)](https://man.archlinux.org/man/kernel-install.8)
plugin to generate systemd-boot loader entry automatically, we just disable it
since we've already created manually:

```
(root)# ln -s /dev/null /etc/kernel/install.d/90-loaderentry.install
```

## Dracut

Remeber we discussed about setting LUKS with a preset key file? This is the right time.
We use [dracut](https://wiki.archlinux.org/title/Dracut) to generate
[initramfs](https://wiki.archlinux.org/title/Arch_boot_process#initramfs) image,
and pack the
[key file](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Types_of_keyfiles)
into it. The reason I choose dracut instead of mkinitcpio is that mkinitcpio is
Arch-specific, since I always try to avoid using tools limited to specific distros.

[Add key file](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Adding_LUKS_keys)
to encrypted partition.

```
cryptsetup luksAddKey /dev/disk/by-partlabel/ROOTPART /etc/cryptsetup-keys.d/root.key
```

Create `/etc/dracut.conf.d/dracut.conf`.

```
hostonly="yes"
enhanced_cpio="yes"
compress="cat"
do_strip="no"
install_optional_items+=" /etc/cryptsetup-keys.d/root.key "
```

By default, dracut will be triggered automatically after kernel update, for Arch
it's done by [pacman hooks](https://wiki.archlinux.org/title/Pacman#Hooks)
, for Fedora and Debian it's done by systemd
[kernel-install(8)](https://man.archlinux.org/man/kernel-install.8), they conform
different name conventions and install images into different locations. To uniform
their behaviors, we disable their default actions and replace with our own.

Since we want to install boot images into `/efi/linux/`, we first create our own
dracut install script, put it into say `/usr/local/bin/dracut-install.sh`:

```
#!/bin/bash

kver="${1}"
dest="${2}"
kimg="/usr/lib/modules/${kver}/vmlinuz"
[[ -f "${kimg}" ]] || exit 1

dracut-install() {
    local stubdir="${1}"
    local vmlinuz=${stubdir}/vmlinuz
    local initrd=${stubdir}/initrd
    install -Dm0644 "${kimg}" "${vmlinuz}"
    dracut --force --hostonly --no-hostonly-cmdline --kver "${kver}" "${initrd}"
}

[[ -d "${dest}" ]] && dracut-install "${dest}" && exit 0
dracut-install /efi/linux
```

Then we run it once manually to initialize boot images.

Next, we modify the pacman hook for Arch and the kernel-install plugin for Fedora
and Debian to let them trigger our script.

For Arch pacman hook, create `/etc/pacman.d/hooks/90-dracut-install.hook`:

```
[Trigger]
Type = Path
Operation = Install
Operation = Upgrade
Operation = Remove
Target = usr/lib/dracut/*
Target = usr/lib/firmware/*
Target = usr/src/*/dkms.conf
Target = usr/lib/systemd/systemd
Target = usr/bin/cryptsetup
Target = usr/bin/lvm

[Trigger]
Type = Path
Operation = Install
Operation = Upgrade
Target = usr/lib/modules/*/vmlinuz
Target = usr/lib/modules/*/pkgbase

[Trigger]
Type = Package
Operation = Install
Operation = Upgrade
Target = dracut

[Action]
Description = Updating initramfs with dracut
When = PostTransaction
Exec = /etc/pacman.d/scripts/dracut-install-alpm.sh
NeedsTargets
```

Then create `/etc/pacman.d/scripts/dracut-install-alpm.sh`:

```
#!/bin/bash

# If KERNEL_INSTALL_INITRD_GENERATOR is set, disable this hook
if [ -n "$KERNEL_INSTALL_INITRD_GENERATOR" ]; then
    exit 0
fi

while read -r line; do
    if [[ "$line" == 'usr/lib/modules/'+([^/])'/pkgbase' ]]; then
        read -r pkgbase < "/${line}"
        kver="${line#'usr/lib/modules/'}"
        kver="${kver%'/pkgbase'}"

        echo "--> Building initramfs for ${pkgbase} (${kver})"
        /usr/local/bin/dracut-install.sh "${kver}"
    fi
done
```

For Fedora and Debian kernel-install plugin, create
`/etc/kernel/install.d/50-dracut.install`:

```
#!/bin/bash

[[ ${#} == 4 ]] || exit 0

cmd="${1}"
kver="${2}"
dest="${3}"
kimg="${4}"

[[ "${cmd}" == "add" ]] || exit 0
[[ -f "${kimg}" ]] || exit 1

/usr/local/bin/dracut-install.sh "${kver}"
```

## Reboot

All done. Now reboot into the new system.

