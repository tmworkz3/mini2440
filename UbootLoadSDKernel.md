Often I'm asked how to load the kernel directly from the SD card, for example the OpenEmbedded on in /boot/uImage.

It turns out it's really trivial, assuming your SD card has 2 partitions, do:
```
MINI2440 # mmcinit
trying to detect SD Card...
Manufacturer:       0x1b, OEM "SM"
Product name:       "00000", revision 1.0
Serial number:      2976007484
Manufacturing date: 1/2009
CRC:                0x45, b0 = 1
READ_BL_LEN=15, C_SIZE_MULT=7, C_SIZE=3453
size = 2329935872
MINI2440 #
```

```
MINI2440 # ext2load mmc 1:2 0x32000000 /boot/uImage

2025396 bytes read
```

The '1:2' means "first card, second partition.