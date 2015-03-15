# Introduction #
OpenEmbedded is a really powerful system that build a tailor-made distribution for you using recipes. Here is a howto to start quickly with OE.

# Quick Start #

The first step is to make sure that
  * you are running Linux
  * you have lot of free space (~10GB)
  * you have git installed

## OE ##
Open a console, make a new directory (that we will call _oeroot_ here) where you want to keep all things OE-related (make sure there is lot of space available for content in that directory).

Then retrieve files from the openembedded-mini2440 repository:
```
git clone git://repo.or.cz/openembedded/mini2440.git base
```

Since you are still in the _oeroot_ directory, make these directories:
  * build/ : where all the magic is going to happen
  * sources/ : directory where all sources will be downloaded

Then, time to configure, go to _base_ and use _mini2440\_local\_conf\_example.conf_ as a base conf file.
```
cp mini2440_local_conf_example.conf conf/local.conf
```

Edit conf/local.conf, there modify:
  * DL\_DIR: where to dl the sources ? In the {{{/oeroot/sources/}} directory
  * BBFILES: where are the recipes ? In `/oeroot/base/recipes/*/*.bb`
  * TMPDIR: where should OE build ? In `/oerrot/build`, make sure it does not end with a / !
  * ASSUME\_PROVIDED: all the packages specified here must be installed on your system. If there aren't, install them or comment the line.

## BitBake ##
Install bitbake on your system, for example, using apt-get:
```
apt-get install bitbake
```

Bitbake needs to know where is your OE root, you need to export the BBPATH var.
If you are shell is bash, put
```
export BBPATH=/oeroot/
```
in your ~/.bashrc file. To have this effect immediately, just type _bash_ (yeah that's dirty, a clean way is to do `source ~/.bashrc`).

## Build ##
This is going to be looong (hopefully, otherwise there is a problem):
```
bitbake console-image
```

_One day later_

## Install ##
Go to `/oeroot/build/deploy/glibc/images/mini2440/`.

That's the images output of OE, and there is few files.

`console-image-mini2440.*` are (links to...) various formats of the linux root image you just built.

  * tar is an archive with the root fs in it, make an empty partition on an sd card and just `tar xvfp` on it
  * jffs2 is a partition for block device like the NAND on the mini2440, you can flash it using the bootloader
  * ext3 is a partition that you can copy to a sd card for example using _dd_ with something like this:
```
dd if=console-image-mini24440.ext3 of=/dev/mmc1p0
```
**You will loose everything on your SD card, and that's a dirty way to do things IMHO.**

`uImage-mini2440.bin` is a link to you kernel, use UbootLoadSDKernel tutorial if you want to from your SD card.

`u-boot-mini2440.bin` is a link to the latest u-boot version build, if you already have u-boot installed, no need to reinstall it.

Another useful directory: `/oeroot/build/deploy/glibc/ipk/`
If you already have an image you already worked on, and you don't want to start from scratch because you need another library/software, compile it using bitbake
` bitbake socketcan `
Where _socketcan_ is the name of the recipe you want to execute.
When it is finished, go to the ipk directory, then _probably_ to the _armv7a_ directory, and grab your package there.
A recipe can build multiple packages, for example socketcan produce two package, while qt4-embedded produce about thirty (not kidding) packages.

# FAQ #

## It's big ! ##
By default, bitbake keeps everything it needed to work (.o files, etc ...).
If you want to save some space, add that property to the `conf/local.conf` file:
```
INHERIT += "rm_work"
```

## How can I customize an image ? ##
You can make yours, that would inherit `console-image` or any image you want, and add the package you want installed.

Just go to `/oeroot/base/recipes/images/`, make a new file my-image.bb and edit it.

Here is a sample recipe (qt4-image.bb) that install qt4-embedded and some part of boost automatically:
```
#Qt4 + boost image
require console-image.bb

DEPENDS += "qt4-embedded boost tslib"

IMAGE_INSTALL += "qt4-embedded \
                  tslib-calibrate tslib-tests \
                  boost-thread boost-date-time"

export IMAGE_BASENAME = "qt4-image"
```