# Introduction #

This document explaind how to bootstrap yourself a nice armel emdebian "grip" install for your ARM board.

# Details #

## debootstrap phase 1 ##

Create a debootstrap with emdebian base minimum system
<pre>
[michel@yap /opt/debian-armel]% *sudo debootstrap --arch=armel --foreign lenny grip/ http://www.emdebian.org/grip/ *<br>
</pre>

The resulting directory contains just whats needed to boot and populate a snall debian.
<pre>
[michel@yap /opt/debian-armel]% *ll grip*<br>
<br>
total 64<br>
drwxr-xr-x  2 root root 4096 Apr 29  2008 bin<br>
drwxr-xr-x  2 root root 4096 Dec  4 09:23 boot<br>
drwxr-xr-x  2 root root 4096 Mar  6 13:12 debootstrap<br>
drwxr-xr-x  2 root root 4096 Nov  8 07:43 dev<br>
drwxr-xr-x 30 root root 4096 Mar  6 13:12 etc<br>
drwxr-xr-x  2 root root 4096 Dec  4 09:23 home<br>
drwxr-xr-x 10 root root 4096 Apr 29  2008 lib<br>
drwxr-xr-x  2 root root 4096 Dec  4 09:23 mnt<br>
drwxr-xr-x  2 root root 4096 Dec  4 09:23 proc<br>
drwxr-xr-x  2 root root 4096 Dec  4 09:23 root<br>
drwxr-xr-x  2 root root 4096 Apr 29  2008 sbin<br>
drwxr-xr-x  2 root root 4096 Sep 16 08:48 selinux<br>
drwxr-xr-x  2 root root 4096 Aug 12  2008 sys<br>
drwxrwxrwt  2 root root 4096 Dec  4 09:23 tmp<br>
drwxr-xr-x  9 root root 4096 Apr  6  2008 usr<br>
drwxr-xr-x 11 root root 4096 Aug 12  2008 var<br>
</pre>

Create a compressed file with the content of this directory, so you can use it again.
<pre>
[michel@yap /opt/debian-armel]% *(cd grip ;  tar jcf ../emdebian-grip-090306-armel-debootstrap-lenny.tar.bz2 .)*<br>
<br>
[michel@yap /opt/debian-armel]% *ll -h*<br>
<br>
total 35M<br>
drwxr-xr-x 18 root   root   4.0K Apr  6  2008 grip<br>
-rw-r--r--  1 michel michel  34M Mar  6 13:13 emdebian-grip-090306-armel-debootstrap-lenny.tar.bz2<br>
</pre>

## SD Card Preparation ##

Insert your SD card in the host. You can detect what device name it is mapped on by doing :
<pre>
[michel@yap /opt/debian-armel]% *dmesg|tail -20*<br>
<br>
...<br>
[ 5876.556251] sd 6:0:0:0: [sdd] 1939456 512-byte hardware sectors: (993 MB/947 MiB)<br>
[ 5876.557248] sd 6:0:0:0: [sdd] Write Protect is off<br>
[ 5876.557250] sd 6:0:0:0: [sdd] Mode Sense: 03 00 00 00<br>
[ 5876.557251] sd 6:0:0:0: [sdd] Assuming drive cache: write through<br>
[ 5876.557253]  *sdd*: sdd1 sdd2 sdd3<br>
</pre>

So we know it's **sdd** here, so lets partition the card again the way we want it.

## Partition SD card ##

First partition of 50MB will be **/boot**, vfat,  and the rest is **/**, ext3. Here is the complete session in fdisk:

<pre>
[michel@yap /opt/debian-armel]% *sudo fdisk /dev/sdd*<br>
<br>
Command (m for help): *o*<br>
<br>
Building a new DOS disklabel with disk identifier 0x3e53813f.<br>
Changes will remain in memory only, until you decide to write them.<br>
After that, of course, the previous content won't be recoverable.<br>
<br>
Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)<br>
<br>
Command (m for help): *p*<br>
<br>
<br>
Disk /dev/sdd: 993 MB, 993001472 bytes<br>
31 heads, 62 sectors/track, 1009 cylinders<br>
Units = cylinders of 1922 ** 512 = 984064 bytes<br>
Disk identifier: 0x3e53813f<br>
<br>
Device Boot      Start         End      Blocks   Id  System<br>
<br>
Command (m for help): *n*<br>
<br>
Command action<br>
e   extended<br>
p   primary partition (1-4)<br>
*p*<br>
<br>
Partition number (1-4): *1*<br>
<br>
First cylinder (1-1009, default 1):<br>
Using default value 1<br>
Last cylinder, +cylinders or +size{K,M,G} (1-1009, default 1009): *+50MB*<br>
<br>
<br>
Command (m for help): *n*<br>
<br>
Command action<br>
e   extended<br>
p   primary partition (1-4)<br>
*p*<br>
<br>
Partition number (1-4): *2*<br>
<br>
First cylinder (53-1009, default 53):<br>
Using default value 53<br>
Last cylinder, +cylinders or +size{K,M,G} (53-1009, default 1009):<br>
Using default value 1009<br>
<br>
Command (m for help): *p*<br>
<br>
<br>
Disk /dev/sdd: 993 MB, 993001472 bytes<br>
31 heads, 62 sectors/track, 1009 cylinders<br>
Units = cylinders of 1922 ** 512 = 984064 bytes<br>
Disk identifier: 0x3e53813f<br>
<br>
Device Boot      Start         End      Blocks   Id  System<br>
/dev/sdd1               1          52       49941   83  Linux<br>
/dev/sdd2              53        1009      919677   83  Linux<br>
<br>
Command (m for help): *t*<br>
<br>
Partition number (1-4): *1*<br>
<br>
Hex code (type L to list codes): *L*<br>
<br>
<br>
0  Empty           1e  Hidden W95 FAT1 80  Old Minix       bf  Solaris<br>
1  FAT12           24  NEC DOS         81  Minix / old Lin c1  DRDOS/sec (FAT-<br>
2  XENIX root      39  Plan 9          82  Linux swap / So c4  DRDOS/sec (FAT-<br>
3  XENIX usr       3c  PartitionMagic  83  Linux           c6  DRDOS/sec (FAT-<br>
4  FAT16 <32M      40  Venix 80286     84  OS/2 hidden C:  c7  Syrinx<br>
5  Extended        41  PPC PReP Boot   85  Linux extended  da  Non-FS data<br>
6  FAT16           42  SFS             86  NTFS volume set db  CP/M / CTOS / .<br>
7  HPFS/NTFS       4d  QNX4.x          87  NTFS volume set de  Dell Utility<br>
8  AIX             4e  QNX4.x 2nd part 88  Linux plaintext df  'BootIt'<br>
9  AIX bootable    4f  QNX4.x 3rd part 8e  Linux LVM       e1  DOS access<br>
a  OS/2 Boot Manag 50  OnTrack DM      93  Amoeba          e3  DOS R/O<br>
* b  W95 FAT32*       51  OnTrack DM6 Aux 94  Amoeba BBT      e4  'SpeedStor'<br>
c  W95 FAT32 (LBA) 52  CP/M            9f  BSD/OS          eb  BeOS fs<br>
e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a0  IBM Thinkpad hi ee  GPT<br>
f  W95 Ext'd (LBA) 54  OnTrackDM6      a5  FreeBSD         ef  EFI (FAT-12/16/<br>
10  OPUS            55  EZ-Drive        a6  OpenBSD         f0  Linux/PA-RISC b<br>
11  Hidden FAT12    56  Golden Bow      a7  NeXTSTEP        f1  'SpeedStor'<br>
12  Compaq diagnost 5c  Priam Edisk     a8  Darwin UFS      f4  'SpeedStor'<br>
14  Hidden FAT16 <3 61  SpeedStor       a9  NetBSD          f2  DOS secondary<br>
16  Hidden FAT16    63  GNU HURD or Sys ab  Darwin boot     fb  VMware VMFS<br>
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware VMKCORE<br>
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fd  Linux raid auto<br>
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fe  LANstep<br>
1c  Hidden W95 FAT3 75  PC/IX           be  Solaris boot    ff  BBT<br>
Hex code (type L to list codes): *b*<br>
<br>
Changed system type of partition 1 to b (W95 FAT32)<br>
<br>
Command (m for help): *p*<br>
<br>
Disk /dev/sdd: 993 MB, 993001472 bytes<br>
31 heads, 62 sectors/track, 1009 cylinders<br>
Units = cylinders of 1922 ** 512 = 984064 bytes<br>
Disk identifier: 0x3e53813f<br>
<br>
Device Boot      Start         End      Blocks   Id  System<br>
/dev/sdd1               1          52       49941    b  W95 FAT32<br>
/dev/sdd2              53        1009      919677   83  Linux<br>
<br>
Command (m for help): *w*<br>
<br>
The partition table has been altered!<br>
<br>
Calling ioctl() to re-read partition table.<br>
<br>
WARNING: If you have created or modified any DOS 6.x<br>
partitions, please see the fdisk manual page for additional<br>
information.<br>
Syncing disks.<br>
</pre>

## Create new filesystems ##

Now, lets create file systems on these 2 new partitions, one vfat, one ext3

<pre>
[michel@yap /opt/debian-armel]% *sudo mkfs.vfat /dev/sdd1*<br>
<br>
mkfs.vfat 2.11 (12 Mar 2005)<br>
[michel@yap /opt/debian-armel]% *sudo mkfs.ext3 /dev/sdd2*<br>
<br>
mke2fs 1.41.3 (12-Oct-2008)<br>
warning: 543 blocks unused.<br>
<br>
Filesystem label=<br>
OS type: Linux<br>
Block size=4096 (log=2)<br>
Fragment size=4096 (log=2)<br>
57568 inodes, 229376 blocks<br>
11495 blocks (5.01%) reserved for the super user<br>
First data block=0<br>
Maximum filesystem blocks=234881024<br>
7 block groups<br>
32768 blocks per group, 32768 fragments per group<br>
8224 inodes per group<br>
Superblock backups stored on blocks:<br>
32768, 98304, 163840<br>
<br>
Writing inode tables: done<br>
Creating journal (4096 blocks): done<br>
Writing superblocks and filesystem accounting information: done<br>
<br>
This filesystem will be automatically checked every 29 mounts or<br>
180 days, whichever comes first.  Use tune2fs -c or -i to override.<br>
</pre>

## Mount new partitions ##

<pre>
[michel@yap /opt/debian-armel]% sudo mount /dev/sdd2 /mnt/arm<br>
[michel@yap /opt/debian-armel]% sudo mkdir -p /mnt/arm/boot<br>
[michel@yap /opt/debian-armel]% sudo mount /dev/sdd1 /mnt/arm/boot<br>
</pre>

## Install debootstrap ##

This will uncompress the archive into the new ext3 partition on the SD card
<pre>
[michel@yap /opt/debian-armel]% (cd /mnt/arm;sudo tar jxf /opt/debian-armel/emdebian-grip-090306-armel-debootstrap-lenny.tar.bz2 ; sync )<br>
</pre>
Some space has been taken by the bootstrap. The real amount is irrelevan, as most of it is unninstalled package files.
<pre>
[michel@yap /opt/debian-armel]% df -h<br>
Filesystem            Size  Used Avail Use% Mounted on<br>
...<br>
/dev/sdd2             882M  98M  724M  18% /mnt/arm<br>
/dev/sdd1              49M     0   49M   0% /mnt/arm/boot<br>
</pre>

# Kernel and Modules install #

If you have a source directory with your compiled kernel, you can already install your modules in their proper space with :
<pre>
[michel@yap /usr/src/linus-2.6]% *sudo make INSTALL_MOD_PATH=/mnt/arm modules_install*<br>
<br>
...<br>
INSTALL sound/usb/caiaq/snd-usb-caiaq.ko<br>
INSTALL sound/usb/snd-usb-audio.ko<br>
INSTALL sound/usb/snd-usb-lib.ko<br>
DEPMOD  2.6.29-rc7<br>
[michel@yap /usr/src/linus-2.6]% *ll /mnt/arm/lib/modules*<br>
<br>
total 4<br>
drwxr-xr-x 3 root root 4096 Mar  6 12:20 2.6.29-rc7<br>
2.6.29-rc7<br>
</pre>

# Finishing the preparation #

The file system needs to be tweaked a bit to boot properly, so we do that with a root shell into the target directory.
Be VERY careful not to use any leafing **/** in your paths!!

<pre>
[michel@yap /usr/src/linus-2.6]% *su -*<br>
<br>
root@yap:~# *cd /mnt/arm*<br>
<br>
root@yap:/mnt/arm# *cp /tftpboot/uImage boot/*<br>
<br>
root@yap:/mnt/arm# *echo "proc /proc proc none 0 0" >>etc/fstab*<br>
<br>
root@yap:/mnt/arm# *echo "mini2440" >etc/hostname*<br>
<br>
root@yap:/mnt/arm# *mknod dev/console c 5 1*<br>
<br>
root@yap:/mnt/arm# *mknod dev/ttySAC0 c 204 64*<br>
<br>
root@yap:/mnt/arm# *echo 'deb http://www.emdebian.org/grip/ lenny main' >>etc/apt/sources.list*<br>
<br>
</pre>

## Unmount the file SD card ##

<pre>
root@yap:/mnt/arm# cd ..<br>
root@yap:/mnt/arm# umount arm/boot arm<br>
</pre>

# U-boot time! #

You can either do these steps in qemu, or using your own board. The process is the same, you need to configure the system to boot from your SD card:
Set environment variables properly for booting on the memory card:
<pre>
MINI2440 # *setenv bootargs console=ttySAC0,115200 noinitrd root=/dev/mtcblk0p2 rootwait=4 rw ip=dhcp init=/bin/sh*<br>
<br>
MINI2440 # *tftp;bootm*<br>
</pre>
Eventualy, you should arrive at a naked shell prompt.

# Second Stage Install #
Do this to finish the debian install. It will take a rather long time, so be patient !
<pre>
sh-3.2# *mount /proc /proc -t proc*<br>
<br>
sh-3.2# *export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin*<br>
<br>
sh-3.2# */debootstrap/debootstrap --second-stage*<br>
<br>
...<br>
</pre>

Wait (a long time) -- udev might give you some warnings due to bad blocks on the NAND, in which case you might have to force-reinstall it when it's done :
<pre>
dpkg -i /var/cache/apt/archives/udev_0.125-7em1_armel.deb<br>
</pre>

# Finishing touches! #

Once that is done, add a bit of niceties to the install:
<pre>
root@yap:/mnt/arm# *echo ttySAC0 >>etc/securetty*<br>
<br>
root@yap:/mnt/arm# *printf "T0:123:respawn:/sbin/getty 115200 ttySAC0\n" >>etc/inittab*<br>
<br>
root@yap:/mnt/arm# *printf "auto eth0\niface eth0 inet dhcp\n" >>etc/network/interfaces*<br>
</pre>

Reboot, you're done !