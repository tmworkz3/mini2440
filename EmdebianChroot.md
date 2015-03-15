# Introduction #

Ubuntu is pretty problematic to build cross tools, and you need a clean debian for using the "emdebian" toolchains

# Details #

<pre>

$ *mkdir /opt/debian-chroot*<br>
<br>
$ *cd /opt/*<br>
<br>
$ *sudo debootstrap sid ./debian-chroot http://ftp.fr.debian.org/debian *<br>
<br>
</pre>

Next chroot into it properly. Update the debian, and install the emdebian tools.

<pre>

$ *sudo chroot /opt/debian-chroot /bin/bash*<br>
<br>
# *aptitude update && aptitude dist-upgrade && aptitude install emdebian-tools*<br>
<br>
...<br>
</pre>

After 200M of download and such, you're setup !