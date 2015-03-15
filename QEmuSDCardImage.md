# Introduction #

QEmu has great tools to manipulate disk images, but sometime it's not easy to know how to make the first one, the one that you have to make from "outside" on the host.

Here's a quick howto on how to make a fake SD card using the root debian file system available in the "Download" page, and use it in qemu.

Note that this is very similar to the way one makes a real SD card.

# Making the Image File #

Thats easy as pie. Just pick a size over 120MB or so, thats probably the smallest size you want your image to me. Anything bigger will work. See **qemu-img --help** for details.

<pre>
michel@moskva:~/qemu/qemu-trunk.git$ *./qemu-img create mini2440/mini2440_sd.img 256M*<br>
<br>
Formatting 'mini2440/mini2440_sd.img', fmt=raw, size=262144 kB<br>
</pre>

Then, make a "network block device" out of it. The process will sit in the background, and avcept just one connection on the oort 1024 (default). See **qemu-nbd --help** for details.
<pre>
michel@moskva:~/qemu/qemu-trunk.git$ *./qemu-nbd mini2440/mini2440_sd.img & *<br>
</pre>

Now, we need to tell linux to "attach" itself to it, and create a virtual block device. First you need to have **nbd-client** installed on your linux, here's how to install it for debian derivative (and ubuntu

<pre>
michel@moskva:~/qemu/qemu-trunk.git$ *sudo aptitude install nbd-client*<br>
<br>
</pre>

## Load the necessary kernel modules ##
You also have to load the necessary modules from the kernl, in case they are not already loaded:
<pre>
michel@moskva:~/qemu/qemu-trunk.git$ *sudo modprobe nbd*<br>
<br>
michel@moskva:~/qemu/qemu-trunk.git$ *sudo modprobe dm-mod*<br>
<br>
</pre>

## "Connect" to the disk ##

Then, you can now connect to your disk !

<pre>
michel@moskva:~/qemu/qemu-trunk.git$ sudo nbd-client localhost 1024 /dev/nbd0<br>
Negotiation: ..size = 262144KB<br>
bs=1024, sz=262144<br>
</pre>

## Re/Partition the disk Image ##

Now, we know that the image is not partitioned and anything, but let's assume here that you already did it before, and that in fact it's partitioned and you want to 'see' them. **nbd-client** doesn't create the partition block devices for you, which is most annoying.

Anyway, lets now partition our "disk". This step is very  similar to what is explained in Emdebian wiki page so the gorry details are skipped for brievety purpose.

<pre>michel@moskva:~/qemu/qemu-trunk.git$ sudo fdisk /dev/nbd0<br>
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel<br>
Building a new DOS disklabel with disk identifier 0x6b164d18.<br>
Changes will remain in memory only, until you decide to write them.<br>
After that, of course, the previous content won't be recoverable.<br>
<br>
Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)<br>
<br>
Command (m for help): o<br>
Building a new DOS disklabel with disk identifier 0xe620d3e2.<br>
Changes will remain in memory only, until you decide to write them.<br>
After that, of course, the previous content won't be recoverable.<br>
<br>
Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)<br>
<br>
Command (m for help): n<br>
Command action<br>
e   extended<br>
p   primary partition (1-4)<br>
p<br>
Partition number (1-4): 1<br>
First cylinder (1-32, default 1):<br>
Using default value 1<br>
Last cylinder, +cylinders or +size{K,M,G} (1-32, default 32): +50MB<br>
<br>
Command (m for help): t<br>
Selected partition 1<br>
Hex code (type L to list codes): b<br>
Changed system type of partition 1 to b (W95 FAT32)<br>
<br>
Command (m for help): n<br>
Command action<br>
e   extended<br>
p   primary partition (1-4)<br>
p<br>
Partition number (1-4): 2<br>
First cylinder (8-32, default 8):<br>
Using default value 8<br>
Last cylinder, +cylinders or +size{K,M,G} (8-32, default 32):<br>
Using default value 32<br>
<br>
Command (m for help): p<br>
<br>
Disk /dev/nbd0: 268 MB, 268435456 bytes<br>
255 heads, 63 sectors/track, 32 cylinders<br>
Units = cylinders of 16065  512 = 8225280 bytes<br>
Disk identifier: 0xe620d3e2<br>
<br>
Device Boot      Start         End      Blocks   Id  System<br>
/dev/nbd0p1               1           7       56196    b  W95 FAT32<br>
/dev/nbd0p2               8          32      200812+  83  Linux<br>
<br>
Command (m for help): w<br>
The partition table has been altered!<br>
<br>
Calling ioctl() to re-read partition table.<br>
<br>
WARNING: Re-reading the partition table failed with error 22: Invalid argument.<br>
The kernel still uses the old table.<br>
The new table will be used at the next reboot.<br>
<br>
WARNING: If you have created or modified any DOS 6.x<br>
partitions, please see the fdisk manual page for additional<br>
information.<br>
Syncing disks.<br>
</pre>

Here we need to tell the system that that block device has partitions on. Being a "virtual" disk the hist system doesn't know about the partition themselves.

We use **kpartx** for that, you might have to install it with :
<pre>
michel@moskva:~/qemu/qemu-trunk.git$ *sudo aptitude install kpartx*<br>
<br>
</pre>

Now use kpartx to find where the partitions are, and be able to mount them
<pre>michel@moskva:~/qemu/qemu-trunk.git$ *sudo kpartx -a /dev/nbd0*<br>
<br>
gpt: 0 slices<br>
dos: 4 slices<br>
nbd0p1 : 0 112392 /dev/nbd0 63<br>
nbd0p2 : 0 401625 /dev/nbd0 112455<br>
</pre>

Now the blocks device you need are in **/dev/mapper/nbd0p1** and **/dev/mapper/nbd0p2**

# Install the system #

You can then follow the tutorial on the Embedian wiki page on how to install the system on it.
Here is a concise way to partition, and install the system :

<pre>

michel@moskva:~/qemu/qemu-trunk.git$ *sudo mkfs.vfat /dev/mapper/nbd0p1*<br>
<br>
michel@moskva:~/qemu/qemu-trunk.git$ *sudo mkfs.ext3 /dev/mapper/nbd0p2*<br>
<br>
michel@moskva:~/qemu/qemu-trunk.git$ *sudo mount /dev/mapper/nbd0p2 ./disk*<br>
<br>
michel@moskva:~/qemu/qemu-trunk.git$ *mkdir -p disk/boot*<br>
<br>
michel@moskva:~/qemu/qemu-trunk.git$ *sudo mount /dev/mapper/nbd0p1 ./disk/boot*<br>
<br>
michel@moskva:~/qemu/qemu-trunk.git$ *wget <INSERT GOOGLE URL>/emdebian-grip-090306-armel-lenny-installed.tar.bz2 *<br>
<br>
michel@moskva:~/qemu/qemu-trunk.git$ *(cd disk; sudo tar jxf ../emdebian-grip-090306-armel-lenny-installed.tar.bz2) *<br>
<br>
michel@moskva:~/qemu/qemu-trunk.git$ *sudo umount disk/boot disk*<br>
</pre>

# Done ! #

Once done, unmounted and all that, just do a:
<pre>michel@moskva:~/qemu/qemu-trunk.git$ *sudo nbd-client -d /deb/nbd0*<br>
</pre>
This will disconnect the drive, end **qemu-nbd** and you can then use the file as expected with **qemu**