# Introduction #

This gets rid of the QTopia install completely. You can always reflash it with vivi later if you really care, but we dont. This page assumes you are running a reasonably standard linux distro for the first bit (xmodem).

This assume you have set the switch to **NOR BOOT** and reset the board.

This also assume you have a u-boot already compiled, in a directory called /tftpboot and that you have managed to configure your host computer as a tftp server.

# Setup picocom #

You need a serial port terminal access.  "picocom" is rather superior to "minicom" these days, due to the fact it can be uded easily frim the command line. No wierd menus and modem init needed.
You also need "lrzsz" for the x/y/z modem command line tools.

# Run picocom #

This assume you have your mini plugged on the main serial port. If you use a usb/serial adapter it's most probsbly ttyUSB0.

<pre>

[michel@yap ~]% *picocom -b 115200 /dev/ttyS0 --send-cmd "sx -vv" *<br>
<br>
picocom v1.4<br>
<br>
port is        : /dev/ttyS0<br>
flowcontrol    : none<br>
baudrate is    : 115200<br>
parity is      : none<br>
databits are   : 8<br>
escape is      : C-a<br>
noinit is      : no<br>
noreset is     : no<br>
nolock is      : no<br>
send_cmd is    : _sx -vv_<br>
receive_cmd is : rz -vv<br>
<br>
Terminal ready<br>
</pre>

Now type return... to see the "vivi" bootloader menu. We're going to immediately quit that menu too.

<pre>
##### FriendlyARM BIOS for 2440 #####<br>
[x] bon part 0 320k 2368k<br>
[v] Download vivi<br>
[k] Download linux kernel<br>
[y] Download root_yaffs image<br>
[c] Download root_cramfs image<br>
[a] Absolute User Application<br>
[n] Download Nboot<br>
[e] Download Eboot<br>
[i] Download WinCE NK.nb0<br>
[w] Download WinCE NK.bin<br>
[d] Download & Run<br>
[z] Download zImage into RAM<br>
[g] Boot linux from RAM<br>
[f] Format the nand flash<br>
[p] Partition for Linux<br>
[b] Boot the system<br>
[s] Set the boot parameters<br>
[t] Print the TOC struct of wince<br>
[u] Backup NAND Flash to HOST through USB(upload)<br>
[r] Restore NAND Flash from HOST through USB<br>
[q] Goto shell of vivi<br>
Enter your selection: *q*<br>
<br>
</pre>

## Load u-boot into RAM ##

This sets us up to the "command line" mode of vivi. We're going to attempt to load the u-boot code into RAM first, and run it from there.
We _could_ flash it directly here, but since we are going to scrub the nand, it's irrelevant.

First, check the size of the u-boot binary. The "nand512" one is padded to fit to a flash block.
<pre>

[michel@yap ~]% *ls -l /tftpboot/u-boot-nand512.bin*<br>
<br>
-rw-r--r-- 1 michel michel _232448_ Mar 13 09:28 /tftpboot/u-boot-nand512.bin<br>
</pre>

**vivi** default parameters are wrong, you need a longer timeout otherwise xmodem will not work.

<pre>

Supervivi> *param set xmodem_timeout 100000000*<br>
<br>
Change 'xmodem_timeout' value. 0x000f4240(1000000) to 0x05f5e100(100000000)<br>
</pre>

Now, tell vivi to start receiving something of 232448 bytes:

<pre>

Supervivi> *load ram 0x33f80000 232448 x*<br>
<br>
Ready for downloading using xmodem...<br>
Waiting...<br>
</pre>

Now in picocom, type "control A" then "control S", this brings a filename prompt.
<pre>
+++ file: */tftpboot/u-boot-nand512.bin*<br>
<br>
sx -vv /tftpboot/u-boot-nand512.bin<br>
Sending /tftpboot/u-boot-nand512.bin, 1816 blocks: Give your local XMODEM receive command now.<br>
Bytes Sent: 232448   BPS:7259<br>
<br>
Transfer complete<br>
<br>
+++ exit status: 0<br>
2448 bytes<br>
Supervivi><br>
</pre>

Loaded !

## Run u-boot from RAM ##
Now, lets RUN the code we have loaded:
<pre>

Supervivi> *go 0x33f80000*<br>
<br>
U-Boot 1.3.2-dirty-moko12 (Mar 12 2009 - 15:50:44)<br>
<br>
I2C:   ready<br>
DRAM:  64 MB<br>
Flash:  2 MB<br>
NAND:  Bad block table not found for chip 0<br>
Bad block table not found for chip 0<br>
64 MiB<br>
+++ Warning - bad CRC or NAND, using default environment<br>
<br>
USB:   S3C2410 USB Deviced<br>
In:    serial<br>
Out:   serial<br>
Err:   serial<br>
MAC: 08:08:11:18:12:27<br>
Hit any key to stop autoboot:  0<br>
MINI2440 #<br>
</pre>

## Check factory bad blocks ##

The nand comes with a list of "already bad" blocks, we can consult it:

<pre>
MINI2440 # *nand info*<br>
<br>
Device 0: NAND 64MiB 3,3V 8-bit, page size 512, sector size 16 KiB<br>
MINI2440 # *nand bad*<br>
<br>
Device 0 bad blocks:<br>
01df0000<br>
037d8000<br>
03ff0000<br>
03ff4000<br>
03ff8000<br>
03ffc000<br>
</pre>

# Scrub the nand #

It's better to clean the nand completely, detect real bad blocks and such. u-boot complains a bit
about it, but we want a clean nand and will create a nice, up-to-date table.

<pre>
MINI2440 # *nand scrub*<br>
<br>
NAND scrub: device 0 whole chip<br>
Warning: scrub option will erase all factory set bad blocks!<br>
There is no reliable way to recover them.<br>
Use this command only for testing purposes if you<br>
are sure of what you are doing!<br>
<br>
Really scrub this NAND flash? <y/N><br>
Erasing at 0x1d70000 --  46% complete.<br>
NAND 64MiB 3,3V 8-bit: MTD Erase failure: -5<br>
Erasing at 0x37ac000 --  87% complete.<br>
NAND 64MiB 3,3V 8-bit: MTD Erase failure: -5<br>
Erasing at 0x3ffc000 -- 100% complete.<br>
Bad block table not found for chip 0<br>
Bad block table not found for chip 0<br>
OK<br>
</pre>

So, it seems it didn't find any more bad blocks, we're clean.

<pre>
MINI2440 # *nand bad*<br>
<br>
Device 0 bad blocks:<br>
01df0000<br>
037d8000<br>
03ff0000<br>
03ff4000<br>
03ff8000<br>
03ffc000<br>
</pre>

# Create a Bad Block Table #

This writes this table on the last blocks of the flash, so the kernel can find it.
<pre>
MINI2440 # *nand createbbt*<br>
<br>
Create BBT and erase everything ? <y/N> *y*<br>
<br>
Skipping bad block at  0x01df0000<br>
Skipping bad block at  0x037d8000<br>
Skipping bad block at  0x03ff0000<br>
Skipping bad block at  0x03ff4000<br>
Skipping bad block at  0x03ff8000<br>
Skipping bad block at  0x03ffc000<br>
<br>
Creating BBT. Please wait ...Bad block table not found for chip 0<br>
Bad block table not found for chip 0<br>
Bad block table written to 0x03ffc000, version 0x01<br>
Bad block table written to 0x03ff8000, version 0x01<br>
<br>
MINI2440 #<br>
</pre>

# Default NAND Partions #

By default u-boot uses a default set of variables, because it can't find a valid
one in the nand.

First lets look where the default partitioning:

<pre>
MINI2440 # *mtdparts*<br>
<br>
device nand0 <mini2440-nand>, # parts = 4<br>
#: name                        size            offset          mask_flags<br>
0: u-boot              0x00040000      0x00000000      0<br>
*1: env                 0x00020000      0x00040000      0*<br>
<br>
2: kernel              0x00500000      0x00060000      0<br>
3: root                0x03aa0000      0x00560000      0<br>
<br>
active partition: nand0,0 - (u-boot) 0x00040000 @ 0x00000000<br>
<br>
defaults:<br>
mtdids  : nand0=mini2440-nand<br>
mtdparts: <NULL><br>
MINI2440 #<br>
</pre>

So the environment is suposed to be written at 0x00040000 in the NAND.

# Set a "custom" ethernet MAC address #

The board does not have an eeprom with a valid MAC address, it relies on the bootloader to
set it, and it can be anything that is "valid". Since you are not a constructor, you don't have
"registered" addresses, so what I ususaly do is pick addresses from a "dead" nanufacturer.

My current favourite is "Pr1me Computers", long dead, plenty of valid MAC addresses !
<pre>
MINI2440 # *set ethaddr 08:00:2f:00:00:02*<br>
</pre>

# Set the environmnet "dynamic" offset #

This u-boot has a way to have a "dynamic" environment offset written in the first NAND block
(the most resilient block on the chip!). It will allow you to skio a "bad" environment block
and relocate it somewhere further the chip.

You first need to tell u-boot to save the default environment offset.

<pre>
MINI2440 # *dynenv set 40000*<br>
<br>
device 0 offset 0x40000, size 0x3fc0000<br>
45 4e 56 30 - 00 00 04 00<br>
</pre>

# Save the environment to flash #

Here goes, save the variables to the specified location. Including your new MAC address!
<pre>
MINI2440 # *saveenv*<br>
<br>
Saving Environment to NAND...<br>
Erasing Nand...<br>
</pre>

# Loading / flashing the first u-boot #

First you need a working network connection. I'm not going into details here, you can use dhcp
or fixed IP addresses, and a remote TFTP server.

## Loading u-boot from the network ##

Load the u-boot binary, note we use the 512 aligned one, since NAND write needs sizes that
are padded to 512 bytes. We load it bang in the middle of the RAM for convenience.
<pre>
MINI2440 # *tftp 32000000 u-boot-nand512.bin*<br>
<br>
dm9000 i/o: 0x20000300, id: 0x90000a46<br>
DM9000: running in 16 bit mode<br>
MAC: 08:00:2f:00:00:02<br>
TFTP from server 10.0.0.4; our IP address is 10.0.0.111<br>
Filename 'u-boot-nand512.bin'.<br>
Load address: 0x32000000<br>
Loading: T ################<br>
done<br>
Bytes transferred = 232448 (_38c00_ hex)<br>
</pre>

## Flash u-boot ##

The ".e" variation of "nand write" in u-boot will write the error-correction bytes,
and skip any bad blocks in the way.
Note that we don't need to _erase_ the NAND block here as it's been "scrubbed".
<pre>
MINI2440 # *nand write.e 32000000 0 38c00*<br>
<br>
NAND write: device 0 offset 0x0, size 0x38c00<br>
<br>
Writing data at 0x38a00 -- 100% complete.<br>
232448 bytes written: OK<br>
MINI2440 #<br>
</pre>

# Done #

There you go. The first time you boot a real linux, you might hit various other problem, like

## Initializing the hardware clock ##
The RTC chip might come with garbage in, if you see this:

<pre>
s3c2410-rtc s3c2410-rtc: hctosys: invalid date/time<br>
...<br>
Setting the system clock.<br>
The Hardware Clock does not contain a valid time, so we cannot set the System Time from it.<br>
</pre>

Use "hwclock" to force it's initialization
<pre>
# *hwclock --set --utc --date="13 march 2009 09:54:00"*<br>
<br>
# *reboot*<br>
<br>
</pre>