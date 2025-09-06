# From LiveCD

```
livecd /home/gentoo/install # fdisk /dev/nvme0n1

Welcome to fdisk (util-linux 2.41.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.


Command (m for help): p

Disk /dev/nvme0n1: 476.94 GiB, 512110190592 bytes, 1000215216 sectors
Disk model: SAMSUNG MZVLB512HBJQ-000H1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 00351B84-7E73-412C-ACA7-0F8F13907C20

Device             Start       End   Sectors   Size Type
/dev/nvme0n1p1      2048   1023999   1021952   499M EFI System
/dev/nvme0n1p2   1024000   1286143    262144   128M Microsoft reserved
/dev/nvme0n1p3   1286144 211978239 210692096 100.5G Microsoft basic data
/dev/nvme0n1p4 211980288 221976575   9996288   4.8G Windows recovery environment

Command (m for help):


```

I already have a GPT disklabel, so I just have to create the necessary partitions.

Creating the swap partition:

```
Command (m for help): n
Partition number (5-128, default 5): 5
First sector (221976576-1000215182, default 221976576):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (221976576-1000215182, default 1000214527): +16G

Created a new partition 5 of type 'Linux filesystem' and of size 16 GiB.
```

Label it as swap:

```
Command (m for help): t
Partition number (1-5, default 5): 5
Partition type or alias (type L to list all): L
Partition type or alias (type L to list all): 19

Changed type of partition 'Linux filesystem' to 'Linux swap'.

```

Print the current partitions:

```
Command (m for help): p
Disk /dev/nvme0n1: 476.94 GiB, 512110190592 bytes, 1000215216 sectors
Disk model: SAMSUNG MZVLB512HBJQ-000H1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 00351B84-7E73-412C-ACA7-0F8F13907C20

Device             Start       End   Sectors   Size Type
/dev/nvme0n1p1      2048   1023999   1021952   499M EFI System
/dev/nvme0n1p2   1024000   1286143    262144   128M Microsoft reserved
/dev/nvme0n1p3   1286144 211978239 210692096 100.5G Microsoft basic data
/dev/nvme0n1p4 211980288 221976575   9996288   4.8G Windows recovery environment
/dev/nvme0n1p5 221976576 255531007  33554432    16G Linux swap

```

Now we create the root partition:

```
Command (m for help): n
Partition number (6-128, default 6):
First sector (255531008-1000215182, default 255531008):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (255531008-1000215182, default 1000214527):

Created a new partition 6 of type 'Linux filesystem' and of size 355.1 GiB.

Command (m for help): t
Partition number (1-6, default 6):
Partition type or alias (type L to list all): L
Partition type or alias (type L to list all): 23

Changed type of partition 'Linux filesystem' to 'Linux root (x86-64)'.

Command (m for help): p
Disk /dev/nvme0n1: 476.94 GiB, 512110190592 bytes, 1000215216 sectors
Disk model: SAMSUNG MZVLB512HBJQ-000H1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 00351B84-7E73-412C-ACA7-0F8F13907C20

Device             Start        End   Sectors   Size Type
/dev/nvme0n1p1      2048    1023999   1021952   499M EFI System
/dev/nvme0n1p2   1024000    1286143    262144   128M Microsoft reserved
/dev/nvme0n1p3   1286144  211978239 210692096 100.5G Microsoft basic data
/dev/nvme0n1p4 211980288  221976575   9996288   4.8G Windows recovery environmen
/dev/nvme0n1p5 221976576  255531007  33554432    16G Linux swap
/dev/nvme0n1p6 255531008 1000214527 744683520 355.1G Linux root (x86-64)
```

Now write the changes to disk:

```
Command (m for help): w

The partition table has been altered.
Syncing disks.

livecd /home/gentoo/install #
```

Make sure to change the Type-UUIDs to conform to DPS (Discoverable Partition Specification)!

## Luks

```
livecd /home/gentoo # cryptsetup -v luksFormat /dev/nvme0n1p6
```

```
livecd /home/gentoo # mkfs.btrfs -L rootfs /dev/mapper/root
```
 
## Applying a filesystem

For the root filesystem:

```
livecd /home/gentoo # lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0   3.6G  1 loop /run/rootfsbase
sda           8:0    1  57.7G  0 disk 
└─sda1        8:1    1  57.7G  0 part /run/initramfs/live
nvme0n1     259:0    0 476.9G  0 disk 
├─nvme0n1p1 259:1    0   499M  0 part 
├─nvme0n1p2 259:2    0   128M  0 part 
├─nvme0n1p3 259:3    0 100.5G  0 part 
├─nvme0n1p4 259:4    0   4.8G  0 part 
├─nvme0n1p5 259:5    0    16G  0 part 
└─nvme0n1p6 259:6    0 355.1G  0 part 
livecd /home/gentoo # mkfs.btrfs -f /dev/nvme0n1p6
btrfs-progs v6.14
See https://btrfs.readthedocs.io for more information.

Performing full device TRIM /dev/nvme0n1p6 (355.09GiB) ...
NOTE: several default settings have changed in version 5.15, please make sure
      this does not affect your deployments:
      - DUP for metadata (-m dup)
      - enabled no-holes (-O no-holes)
      - enabled free-space-tree (-R free-space-tree)

Label:              (null)
UUID:               f66f8882-459e-40f4-af87-f01b0c7c784b
Node size:          16384
Sector size:        4096        (CPU page size: 4096)
Filesystem size:    355.09GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP               1.00GiB
  System:           DUP               8.00MiB
SSD detected:       yes
Zoned device:       no
Features:           extref, skinny-metadata, no-holes, free-space-tree
Checksum:           crc32c
Number of devices:  1
Devices:
   ID        SIZE  PATH          
    1   355.09GiB  /dev/nvme0n1p6
```
For the swap partition:

```
livecd /home/gentoo # mkswap /dev/nvme0n1p5
Setting up swapspace version 1, size = 16 GiB (17179865088 bytes)
no label, UUID=31e55f72-4af4-4afa-a565-3eb599d3e6da
livecd /home/gentoo # swapon /dev/nvme0n1p5
```

## Mounting the root partition

```
livecd /home/gentoo # mkdir --parents /mnt/gentoo/
livecd /home/gentoo # mount /dev/nvme0n1p6 /mnt/gentoo/
```

Mount the EFI partition too

```
livecd /home/gentoo # mkdir --parents /mnt/gentoo/efi
livecd /home/gentoo # mount /dev/nvme0n1p1 /mnt/gentoo/efi/
```

# Installing a stage file

```
livecd /home/gentoo # cd /mnt/gentoo/
livecd /mnt/gentoo # date
Wed Sep  3 20:27:28 UTC 2025
livecd /mnt/gentoo # chronyd -q
2025-09-03T20:27:35Z chronyd version 4.7 starting (+CMDMON +REFCLOCK +RTC +PRIVDROP +SCFILTER -SIGND +NTS +SECHASH +IPV6 -DEBUG)
2025-09-03T20:27:35Z Wrong owner of /run/chrony (UID != 0)
2025-09-03T20:27:35Z Disabled command socket /run/chrony/chronyd.sock
2025-09-03T20:27:35Z Running with root privileges
2025-09-03T20:27:40Z System clock wrong by 25196.524922 seconds (step)
2025-09-04T03:27:37Z chronyd exiting
```

I downloaded the following stage3 file:

https://distfiles.gentoo.org/releases/amd64/autobuilds/20250831T170358Z/stage3-amd64-desktop-openrc-20250831T170358Z.tar.xz

```
livecd /mnt/gentoo # tar xpvf stage3-amd64-desktop-openrc-20250831T170358Z.tar.xz --xattrs-include='*.*' --numeric-owner -C /mnt/gentoo/
```

## Configuring compile flags

```
livecd /mnt/gentoo # vim /mnt/gentoo/etc/portage/make.conf 
```

Add `-march=native` to `COMMON_FLAGS` and add a `RUST_FLAGS` option. Also add `MAKEOPTS`
```
livecd /mnt/gentoo # cat /mnt/gentoo/etc/portage/make.conf 
# These settings were set by the catalyst build script that automatically
# built this stage.
# Please consult /usr/share/portage/config/make.conf.example for a more
# detailed example.
COMMON_FLAGS="-march=native -O2 -pipe"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"

RUSTFLAGS="${RUSTFLAGS} -C target-cpu=native"

MAKEOPTS="-j8 -l9"

# NOTE: This stage was built with the bindist USE flag enabled

# This sets the language of build output to English.
# Please keep this setting intact when reporting bugs.
LC_MESSAGES=C.utf8
```

## Installing the base system

```
livecd /mnt/gentoo # cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

Chroot into the new location:

```
livecd /mnt/gentoo # arch-chroot /mnt/gentoo/
livecd / # lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0   3.6G  1 loop 
sda           8:0    1  57.7G  0 disk 
└─sda1        8:1    1  57.7G  0 part 
nvme0n1     259:0    0 476.9G  0 disk 
├─nvme0n1p1 259:1    0   499M  0 part /efi
├─nvme0n1p2 259:2    0   128M  0 part 
├─nvme0n1p3 259:3    0 100.5G  0 part 
├─nvme0n1p4 259:4    0   4.8G  0 part 
├─nvme0n1p5 259:5    0    16G  0 part [SWAP]
└─nvme0n1p6 259:6    0 355.1G  0 part /

```

Thus, we see that we are now in your new root partition.

```
livecd / # source /etc/profile
livecd / # export PS1="(chroot) ${PS1}"
(chroot) livecd / # 
```

## Configuring portage

```
(chroot) livecd / # emerge-webrsync 
```

```
(chroot) livecd / # emerge --ask --verbose --oneshot app-portage/mirrorselect
```

```
(chroot) livecd / # mirrorselect -i -o >> /etc/portage/make.conf 
```

I am already using the profile I want, so I won't switch.

Configure a binary package host:

`/etc/portage/binrepos.conf/gentoobinhost.conf`
```
# These settings were set by the catalyst build script that automatically
# built this stage.
# Please consider using a local mirror.

[gentoobinhost]
priority = 1
sync-uri = http://gentoo-mirror.flux.utah.edu/releases/amd64/binpackages/23.0/x86-64-v3/
```

I won't be configuring Gentoo to use binary packages by default.

```
(chroot) livecd / # emerge --ask --oneshot app-portage/cpuid2cpuflags
```

```
(chroot) livecd / # cat /etc/portage/package.use/00cpu-flags 
*/* CPU_FLAGS_X86: aes avx avx2 bmi1 bmi2 f16c fma3 mmx mmxext pclmul popcnt rdrand sha sse sse2 sse3 sse4_1 sse4_2 sse4a ssse3 vpclmulqdq
```

Configure `VIDEO_CARDS`

```
(chroot) livecd / # cat /etc/portage/package.use/00video_cards 
*/* VIDEO_CARDS: amdgpu radeonrsi
```

I will change `ACCEPT_LICENSE` to accept basically any license:

In `/etc/portage/make.conf` add 

```
ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE @EULA"
```

We can check that the change worked:

```
(chroot) livecd / # portageq envvar ACCEPT_LICENSE
@FREE @BINARY-REDISTRIBUTABLE @EULA
```

Now I will update the world set:

```
(chroot) livecd / # emerge --ask --verbose --update --deep --changed-use @world
```

Set the timezone

```
(chroot) livecd / # ln -sf /usr/share/zoneinfo/US/Pacific /etc/localtime 
```

Generate the locale

```
(chroot) livecd / # cat /etc/locale.gen 
# /etc/locale.gen: list all of the locales you want to have on your system.
# See the locale.gen(5) man page for more details.
#
# The format of each line:
# <locale name> <charset>
#
# Where <locale name> starts with a name as found in /usr/share/i18n/locales/.
# It must be unique in the file as it is used as the key to locale variables.
# For non-default encodings, the <charset> is typically appended.
#
# Where <charset> is a charset located in /usr/share/i18n/charmaps/ (sans any
# suffix like ".gz").
#
# All blank lines and lines starting with # are ignored.
#
# For the default list of supported combinations, see the file:
# /usr/share/i18n/SUPPORTED
#
# Whenever glibc is emerged, the locales listed here will be automatically
# rebuilt for you.  After updating this file, you can simply run `locale-gen`
# yourself instead of re-emerging glibc.

en_US ISO-8859-1
en_US.UTF-8 UTF-8
#ja_JP.EUC-JP EUC-JP
#ja_JP.UTF-8 UTF-8
#ja_JP EUC-JP
#en_HK ISO-8859-1
#en_PH ISO-8859-1
#de_DE ISO-8859-1
#de_DE@euro ISO-8859-15
#es_MX ISO-8859-1
#fa_IR UTF-8
#fr_FR ISO-8859-1
#fr_FR@euro ISO-8859-15
#it_IT ISO-8859-1
```

```
(chroot) livecd / # locale-gen 
 * Generating 3 locales (this might take a while) with 16 jobs
 *  (1/3) Generating en_US.ISO-8859-1 ...                                         [ ok ]
 *  (3/3) Generating C.UTF-8 ...                                                  [ ok ]
 *  (2/3) Generating en_US.UTF-8 ...                                              [ ok ]
 * Generation complete
 * Adding locales to archive ...                                                  [ ok ]
```

Set the locale

```
(chroot) livecd / # eselect locale list
Available targets for the LANG variable:
  [1]   C
  [2]   C.utf8
  [3]   POSIX
  [4]   en_US
  [5]   en_US.iso88591
  [6]   en_US.utf8
  [7]   C.UTF8 *
  [ ]   (free form)
```

```
(chroot) livecd / # eselect locale set 2
Setting LANG to C.utf8 ...
Run ". /etc/profile" to update the variable in your shell.
```

```
(chroot) livecd / # env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
>>> Regenerating /etc/ld.so.cache...
```

## Configuring the Kernel

```
(chroot) livecd / # emerge --ask sys-kernel/linux-firmware
```

I will be using EFI stub booting instead of GRUB

In `/etc/portage/package.accept_keywords/installkernel`
```
sys-kernel/installkernel
sys-boot/uefi-mkconfig
app-emulation/virt-firmware
```

In `/etc/portage/package.use/installkernel`
```
sys-kernel/installkernel efistub
```

```
(chroot) livecd / # emerge --ask sys-kernel/installkernel
```

Add the `dracut` USE flag to `installkernel`.

In `/etc/portage/package.use/installkernel`
```
sys-kernel/installkernel efistub dracut
```

I will not be using a UKI (Unified Kernel Image).

Install the distribution kernel

```
emerge --ask sys-kernel/gentoo-kernel
```

Add the global `USE` flag to `/etc/portage/make.conf`:

```
USE="dist-kernel"
```

Install the kernel sources:

```
emerge --ask sys-kernel/gentoo-sources
```

