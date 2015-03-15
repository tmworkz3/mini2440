# Introduction #

Once you are in debian proper, you need some configuration legwork to get the cross environment working. If you run Ubuntu, see EmdebianChroot for a howto install a sandbox.

Note that you might need to install extra pcakges as you go along for everything to work. If you want to make your own toolchain if the binary ine is not available, you have to use **emchain** too, see bellow.

# Details #

Install emdebian-tools, if not already done so.

<pre>
# *aptitude install emdebian-tools*<br>
</pre>

First, select the default architecture. We use **armel** for mini2440.
<pre>
# *dpkg-reconfigure dpkg-cross*<br>
</pre>

Next, let emsetup add itself to the source list, and also upgrade itself if necessary.
<pre>
# *emsetup*<br>
</pre>

Now it's time to install all the relevant toolchains and headers:
<pre>
# *aptitude install libc6-armel-cross libc6-dev-armel-cross binutils-arm-linux-gnueabi gcc-4.3-arm-linux-gnueabi g++-4.3-arm-linux-gnueabi*<br>
</pre>

# Build a toolchain #
If the toolchain is not available as a binary in embedian, you need to use **emchain** to build one.

**emchain** takes a long time, and will probably fail to work the first times due to lack of dependencies on your system, so read the errors carefully and install missing packages with **aptitude install XXXX** where XXXX is bison, autogen and all that.

Other than this it should be pretty straightforward!