Current state: Experimental / Beta! See To do below.

# CloneLove

### A network disk cloner
#### based on PXE, dd and gzip, assembled with Linux, udev, busybox and dropbear

CloneLove is Linux 3.18, running in RAM and started from the network. The kernel includes most disk-, controller- and network drivers, all detected and loaded, with the precious help of udev and friends.

A tiny script is taking care for automating the fetch of the image file, and the raw write to disk. SSH is included (usable in both directions) to provide a way to upload and create new disk images, or to do maintenance. CloneLove is designed to work completely silent / unattended, cloning disks over the network to multiple hosts, fast, in parallel.

When CloneLove is loaded and/or cloning, you can connect to CloneLove with ssh and login as root without password. This enables remote automated post-processing, or even doing stuff while cloning is taking place.

### Download

Click [here](https://w3b.net/clonelove/clone.go) to download release 20150101, md5sum d6cf8a91a543cfb2dc8f9c22754af234

### Usage

Create a Master server. This server should do:

* DHCP, serve IP addresses in a 10.11.12.0 network
* PXE, the network boot mechanism, available in most NICs
* TFTP, the protocol for the initial file transfers
* HTTP, to serve the disk images to the network
* SSH, for uploading new disk images to the Master server

In CentOS6, the following packages were used to build a Master server:

    sudo yum install openssh-server dhcp tftp-server syslinux httpd

Next,

* Unpack clonelove_tftpboot.tgz into the 'tftpboot' directory structure of your Master server
* Modify /var/lib/tftpboot/pxelinux.cfg/default to your needs
* Or... Make more specific, IP related files in pxelinux.cfg/*, to make cloning groups
* Check if you are not blocking traffic, add necessary rules or disable your firewall
* Apply the configurations you find below
* Go! Start creating disk images and cloning machines
* Place image files into the web server's document root, to make them available in the network like this: **`http://10.11.12.1/my_vda.img.gz`**

**Assumptions / Requirements,**

* The Master server is in an isolated network, on a dedicated switch
* The Master server has IP address 10.11.12.1
* The Master server has a user clonelove, with no password set (`sudo useradd clonelove` `sudo passwd -d clonelove`)
* Login without password is allowed in the SSH server configuration (see below)
* You know what you are doing and you have determined and configured the correct target disk to be cloned to, in `/var/lib/tftpboot/pxelinux.cfg/*`

### Install CloneLove

    wget https://w3b.net/clonelove/clonelove_tftpboot.tgz
    sudo tar xzf clonelove_tftpboot.tgz -C /var/lib/

### Configuration Files

##### SSH

`/etc/ssh/sshd_config`

    Protocol 2
    SyslogFacility AUTHPRIV
    PermitEmptyPasswords yes
    PasswordAuthentication yes
    ChallengeResponseAuthentication no
    GSSAPIAuthentication no
    UsePAM yes

##### TFTP / XINETD

`/etc/xinetd.d/tftp`

    service tftp
    {
    	socket_type		= dgram
    	protocol		= udp
    	wait			= yes
    	user			= root
    	server			= /usr/sbin/in.tftpd
    	server_args		= -s /var/lib/tftpboot
    	disable			= no
    	per_source		= 11
    	cps			= 100 2
    	flags			= IPv4
    } 

##### DHCP

`/etc/dhcp/dhcpd.conf`

    ddns-update-style none;
    option space PXE;
    option PXE.mtftp-ip               code 1 = ip-address;  
    option PXE.mtftp-cport            code 2 = unsigned integer 16;
    option PXE.mtftp-sport            code 3 = unsigned integer 16;
    option PXE.mtftp-tmout            code 4 = unsigned integer 8;
    option PXE.mtftp-delay            code 5 = unsigned integer 8;
    option PXE.discovery-control      code 6 = unsigned integer 8;
    option PXE.discovery-mcast-addr   code 7 = ip-address;
    subnet 10.11.12.0 netmask 255.255.255.0 {
      class "pxeclients" {
        match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
        option vendor-class-identifier "PXEClient";
        vendor-option-space PXE;
        option PXE.mtftp-ip 0.0.0.0;
        option routers 10.11.12.1;
        filename "pxelinux.0";
        next-server 10.11.12.1;
      }
      pool {
        max-lease-time 86400;
        default-lease-time 86400;
        range 10.11.12.101 10.11.12.254;
        allow unknown clients;
      }
    }

##### PXE Boot Loader

`/var/lib/tftpboot/pxelinux.cfg/default`

    DEFAULT bzImage
    
    DISPLAY boot.msg
    
    # Uncomment *one* of the 'APPEND' lines below.
    
    # Silent cloning from a compressed disk image, with reboot in 5 seconds when ready:
    #APPEND initrd=initramfs.cpio.gz rw quiet ip=dhcp disk=vda image=oblive.img.gz gzip reboot=5 root=/dev/ram0
    
    # Silent cloning from a compressed disk image, drop into shell:
    #APPEND initrd=initramfs.cpio.gz rw quiet ip=dhcp disk=vda image=my_vda.img.gz gzip root=/dev/ram0
    
    # Drop into a shell, with tools to make a new disk image:
    APPEND initrd=initramfs.cpio.gz rw quiet ip=dhcp disk=vda image=none root=/dev/ram0
    
Get the point?

If not, please do not use CloneLove! Make sure you understand above mechanism before use!

### Tips for image creation

To do.

### Tweaking

To do.
