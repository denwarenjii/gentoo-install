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

## Applying a filesystem


