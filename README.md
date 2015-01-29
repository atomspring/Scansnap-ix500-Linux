# Scansnap-ix500-Linux
Support the ix500 on Linux (Ubuntu 14.04 LTS in this case, but universal)

When you have an ix500 on Ubuntu 14.04, the OS is running libsane 1.0.23. At the time of writing (January 29th, 2015) the lastest SANE backend is 1.0.25.

Simply apply the following to support the ix500 in any SANE-capable application:

<code> sudo -i

<code> apt-get install build-essential checkinstall libusb-dev git-core; apt-get remove libsane libsane-common

So, you've got the needed stuff to build SANE, and you've removed the stale stuff from 1.0.23. Now, let's get the latest files:

<code> cd /usr/src

<code>git clone git://git.debian.org/sane/sane-backends.git

<code>cd sane-backends

We're going to configure the build, supporting only the fujitsu scanner backend to speed compile times. Leave out the BACKENDS part to support all scanners:

<code>./configure --prefix=/usr --libdir=/usr/lib/x86_64-linux-gnu --sysconfdir=/etc --localstatedir=/var BACKENDS=fujitsu

<code>make clean; make -j `grep -c CPU /proc/cpuinfo`

Let's move the dcoumentation to the correct area:
<code>mkdir -vp doc-pak && cp -v AUTHORS COPYING INSTALL NEWS README THANKS doc-pak

Install as a debian package under the name "libsane" to fix dependency issues:

<code>checkinstall --pkgname libsane --pkgversion "1.0.25git"

If this step fails because the package is trying to overwrite a file in another package, you'll need to remove that package and try to install the package you just built again:

<code>dpkg -i libsane_1.0.25-1_amd64.deb

Once successful, update libraries:

<code>ldconfig

<b>Disable SANEd if you're not scanning to the network</b> by editing /etc/default/saned on Ubuntu so that RUN=yes is RUN=no.

Open up the scanner so it turns on, then find the scanner and add it to Ubuntu's visible scanner list:

<code>sane-find-scanner

Once you have the Vendor and Device ID (Should be vendor=0x04c5 product=0x132b) push this into the list:

<code>echo -e "\n\n#ScanSnap ix500\nusb 0x04c5 0x132b" >> /etc/sane.d/fujitsu.conf

Disconnect your ix500, reconnect and run a scanning tool as root, eg simple-scan (if simple-scan got deleted, apt-get install simple-scan -y).

If this works, try running simple-scan as a regular user. If you can't see the scanner, but it worked as root, you'll need to update udev's permissions:

<code>echo -e "\n#ScanSnap ix500\nATTRS{idVendor}=="04c5", ATTRS{idProduct}=="132b", ENV{libsane_matched}="yes"" >> /etc/udev/rules.d/40-fujitsu.rules

<code>service udev reload

Finally, unplug and reconnect the scanner and you should see it in the client applications.
