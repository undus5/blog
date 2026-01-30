+++
title       = 'Linux A/B System Updates via BTRFS Snapshot'
lastmod     = '2026-01-30'
date        = '2025-11-05'
tags        = []
showSummary = true
showTOC     = true
weight      = 1000
+++

Inspired from Android A/B system updates mechanism.

<!--more-->

## Series Index

1. [Linux Bootstrap Installation](/posts/linux-bootstrap-installation/)
2. Linux A/B System Updates via BTRFS Snapshot
3. [Linux Post Installation: Desktop Preparation](/posts/linux-post-installtion-desktop-preparation/)
4. [Linux Desktop: Sway, Labwc, GUI Apps](/posts/linux-desktop-sway-labwc-gui-apps/)

## Preface

This guide is distro independent, could work on any distros.

## Background

Things started when I was intrigued by some discussions about booting from
[BTRFS snapshots](https://wiki.archlinux.org/title/Btrfs#Snapshots)
directly. And before this A/B solution coming to my mind,
I was just using following commands to create and remove snapshots manually.

```
(root)# btrfs subvolume snapshot / <destination>
(root)# btrfs subvolume delete <destination>
```

## Principle

The core idea of this A/B solutions is there are two subvolumes for root partition,
`@a` and `@b`, they are mutually to be the snapshot of each other.
And we maintain two bootloader entries for them respectively.
The tricky part is you need to alter the `fstab` to make
these two subvolumes point to the right ones after generating the new snapshot
everytime.

## Subvolumes

Let's begin with subvolume overview. You need entrering a live system environment
to create subvolumes.

```
(root)# mount /dev/disk/by-partlabel/ROOTPART /mnt
(root)# btrfs subvolume create /mnt/@a
(root)# btrfs subvolume create /mnt/@a/@
(root)# btrfs subvolume create /mnt/@b
(root)# btrfs subvolume create /mnt/@b/@
(root)# btrfs subvolume create /mnt/@home
```

If you feel strange about this `ROOTPART` label, read the previous guide for
basic system installation first. For simplicity, the root partition in this guide
is not encrypted, which means LUKS is not involved.

The nested subvolumes `@` are the real root partition, which will be mounted as
`/`, and they also are the snapshots for each other
(subvolume and snapshot are basically same in BTRFS). The reason of subvolume
stucture be like this is when we creating BTRFS snapshot, the destination path
must not exist, which means we need to delete old `@` subvolume first to create
new one.

For example, if we remove this nested layer, just mount `@a` as `/`,
`@b` as `/b`, we could not remove `/b` or `@b` since it's a mount point in use.
If we add this nested layer, mount `@a/@` as `/` and still `@b` as `/b`,
we could delete and recreate `/b/@` from `@a` to `@b` and vice versa.

## Fstab

Fstab for `@a/@`:

```
# <file system> <dir> <type> <options> <dump> <pass>
UUID=40379083-xxxx-xxxx-xxxx-848931c8d87c /     btrfs compress=zstd,subvol=/@a/@ 0 0
UUID=40379083-xxxx-xxxx-xxxx-848931c8d87c /b    btrfs compress=zstd,subvol=/@b   0 0
UUID=40379083-xxxx-xxxx-xxxx-848931c8d87c /home btrfs compress=zstd,subvol=/@home 0 0
UUID=3A19-XXXX /efi vfat defaults 0 2
```

Fstab for `@b/@`:

```
# <file system> <dir> <type> <options> <dump> <pass>
UUID=40379083-xxxx-xxxx-xxxx-848931c8d87c /     btrfs compress=zstd,subvol=/@b/@ 0 0
UUID=40379083-xxxx-xxxx-xxxx-848931c8d87c /a    btrfs compress=zstd,subvol=/@a   0 0
UUID=40379083-xxxx-xxxx-xxxx-848931c8d87c /home btrfs compress=zstd,subvol=/@home 0 0
UUID=3A19-XXXX /efi vfat defaults 0 2
```

Be careful with the minor differences.

## Bootloader Entries

This A/B solution can work with any bootloader, I use systemd-boot for now.

Systemd-boot entry for `@a` in `/efi/loader/entries/a.conf`:

```
title Boot A
linux /a/vmlinuz
initrd /a/initrd
options rootflags=subvol=@a/@ quiet
sort-key A
```

Systemd-boot entry for `@b` in `/efi/loader/entries/b.conf`:

```
title Boot B
linux /b/vmlinuz
initrd /b/initrd
options rootflags=subvol=@b/@ quiet
sort-key B
```

## Bash Script

All the processes can be automated into a bash script:

```
#!/usr/bin/bash
set -e

errf() { printf "${@}" >&2 && exit 1; }

[[ ${EUID} == 0 ]] || errf "need root priviledge\n"

case ${1} in
    ab)
        _srcname=a
        _dstname=b
        ;;
    ba)
        _srcname=b
        _dstname=a
        ;;
    *)
        errf "Usage: $(basename ${0}) <ab|ba>\n"
        ;;
esac

_dstvol_alert="abort: you are running under '@${_dstname}' subvolume now\n"
findmnt /${_srcname} &>/dev/null && errf "${_dstvol_alert}"
findmnt /${_dstname} &>/dev/null || errf "${_dstvol_alert}"

printf "==> Copy vmlinuz and initrd from '@${_srcname}' to '@${_dstname}'\n"
_stubsrc=/efi/${_srcname}
_stubdst=/efi/${_dstname}
_stubtmp=/efi/t
[[ -d ${_stubdst} ]] && mv ${_stubdst} ${_stubtmp}
[[ -d ${_stubsrc} ]] && cp -r ${_stubsrc} ${_stubdst}
[[ -d ${_stubtmp} ]] && rm -rf ${_stubtmp}

_dstvol=/${_dstname}/@

# remove the read-only protection just in case
[[ -d ${_dstvol} ]] && btrfs prop set -f -ts ${_dstvol} ro false

printf "==> "
[[ -d ${_dstvol} ]] && btrfs subvolume delete ${_dstvol}

printf "==> "
btrfs subvolume snapshot / ${_dstvol}

printf "==> Modify fstab in '/${_dstname}/@'\n"
sed -i -r \
    -e "s#/${_dstname}#/${_srcname}#" \
    -e "s#@${_dstname}\s+0#@${_srcname}   0#" \
    -e "s#@${_srcname}/@#@${_dstname}/@#" \
    ${_dstvol}/etc/fstab

_time=$(date +%Y%m%d.%H%M%S)
_timetxt=/${_dstname}/timestamp.${_time}.txt
rm /${_dstname}/*.txt
printf "${_time}\n" > ${_timetxt}
printf "==> Create ${_timetxt}\n"
```

## Cache Clean

You may want to clean cache files before creating snapshot to save storage.

[Limit systemd journal size](https://wiki.archlinux.org/title/Systemd/Journal#Journal_size_limit)
in `/etc/systemd/journald.conf`:

```
[Journal]
SystemMaxUse=120M
```

Clean package cache:

[pacman](https://wiki.archlinux.org/title/Pacman#Cleaning_the_package_cache):
`paccache -r && paccache -ruk0`

[dnf-clean(8)](https://man.archlinux.org/man/dnf5-clean.8.en):
`dnf clean packages`

[apt(8)](https://man.archlinux.org/man/apt.8.en):
`apt autoremove && apt autoclean`

