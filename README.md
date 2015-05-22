Current state: Experimental / Beta! See To do below.

= CloneLove =

### A network disk cloner
based on PXE, dd and gzip, assembled with Linux, udev, busybox and dropbear

CloneLove is Linux 3.18, running in RAM and started from the network. The kernel includes most disk-, controller- and network drivers, all detected and loaded, with the precious help of udev and friends.

A tiny script is taking care for automating the fetch of the image file, and the raw write to disk. SSH is included (usable in both directions) to provide a way to upload and create new disk images, or to do maintenance. CloneLove is designed to work completely silent / unattended, cloning disks over the network to multiple hosts, fast, in parallel.

When CloneLove is loaded and/or cloning, you can connect to CloneLove with ssh and login as root without password. This enables remote automated post-processing, or even doing stuff while cloning is taking place.
