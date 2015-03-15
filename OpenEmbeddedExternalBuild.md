# OpenEmbedded external cross-build helper #

Once you have OpenEmbedded running and built, you might want to be able to cross-compile your own code against it to run it on the board, and use the cross toolchain as well as the target libraries to build your own program environment.

This helper script is meant to replace "make" for these tasks, and set the environment properly.

Of course, this is just an example, for my own use, and if it works for you it's great, otherwise you can massage it until it fits your own needs.

# Details #

Note the special case I use when building the kernel. This allows me to keep several buitt kernels versions without poluting the source directory.

Also note the Hack for ubuntu, you need **pkg-config 0.23** for this system to work. If your distro doesn't have it, you can copy over the one that OpenEmbedded builds for you.

```
#!/bin/bash

# Setup a cross build environment. If your "makefile" uses the proper
# cross comoile macros, and uses pkg-config for all the libs, it should
# build straight away with this

# the only "exception" is pkg-config executable. ubuntu comes with .22
# that doesn't know how to handle PKG_CONFIG_SYSROOT_DIR so you night want
# to copy the OE one, rename it and use that

function oe_set_cross_compile() {
        export OE_ROOT=/opt/build/oe/build/tmp/

        export OE_ARCH=armv4t
        export OE_BASEARCH=arm
        export OE_DISTRO=angstrom-linux-gnueabi
        export OE_STAGING=$OE_ROOT/staging/$OE_ARCH-$OE_DISTRO

        export CROSS_COMPILE=$OE_BASEARCH-$OE_DISTRO
        export CROSS_PATH=$OE_ROOT/cross/$OE_ARCH/bin
        export CROSS_VERSION=

        export PKG_CONFIG=pkg-config-0.23-oe
        export PKG_CONFIG_PATH=$OE_STAGING/usr/lib/pkgconfig
        export PKG_CONFIG_SYSROOT_DIR=$OE_STAGING

        export CFLAGS="-O3 -march=armv4t -mtune=arm920t"
        export CXXFLAGS="-O3 -march=armv4t -mtune=arm920t"

        echo Setting up for OE $CROSS_COMPILE
}
oe_set_cross_compile

#
# Special case for the kernel
#
extra=""
if [ -d kernel -a Documentation ]; then
        kv=$(head -4 Makefile|awk '{v[i++]=$3}END{print v[0]"-"v[1]"-"v[2]""v[3];}')
        ko="../kernel-$OE_ARCH-$kv.obj"
        if [ ! -d "$ko" ]; then
                mkdir -p $ko
        fi
        extra="O=$ko ARCH=$OE_BASEARCH CROSS_COMPILE=$CROSS_PATH/$CROSS_COMPILE-"
        echo Building kernel $extra

        if make $extra $*; then
                echo $0: All done
                src="$ko/arch/arm/boot/uImage"
                if [ "$src" -nt "/tftpboot/uImage" ]; then
                        echo $0: Refreshed /tftpboot/
                        cp -a "$src" "/tftpboot/"
                        dd if="$src" of="/tftpboot/uImage-nand512" \
                                bs=512 conv=sync
                        dd if="$src" of="/tftpboot/uImage-nand16k" \
                                bs=16k conv=sync
                fi
        fi
        exit
fi

make $extra $*
```