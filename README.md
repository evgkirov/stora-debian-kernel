# Debian Wheezy Kernel for Netgear Stora

It is based on Linux 3.10 (longterm maintenance release) and uses Debian's default config for kirkwood. Unlike [johnou's kernel](https://github.com/johnou/debian_stora_kernel/) it doesn't include some nice patches but it supports ext4fs and other stuff.


## Credits

* https://github.com/johnou/debian_stora_kernel/
* http://forum.doozan.com/read.php?2,13355,13367#msg-13367


## How to install

    wget https://github.com/kirov/stora-debian-kernel/releases/download/v3.10.26/linux-image-3.10.26-stora_1_armel.deb
    dpkg -i linux-image-3.10.26-stora_1_armel.deb
    mkimage -A arm -O linux -T kernel  -C none -a 0x00008000 -e 0x00008000 -n Linux-3.10.26-stora -d /boot/vmlinuz-3.10.26-stora /boot/uImage
    mkimage -A arm -O linux -T ramdisk -C gzip -a 0x00000000 -e 0x00000000 -n initramfs-3.10.26-stora -d /boot/initrd.img-3.10.26-stora /boot/uInitrd

If you need headers:

    wget https://github.com/kirov/stora-debian-kernel/releases/download/v3.10.26/linux-headers-3.10.26-stora_1_armel.deb
    dpkg -i linux-headers-3.10.26-stora_1_armel.deb


## How to build your own kernel for Stora

You will need Debian/Ubuntu machine for this.

First we install packages

    apt-get install emdebian-archive-keyring
    echo "deb http://ftp.uk.debian.org/emdebian/toolchains wheezy main" >> /etc/apt/sources.list
    apt-get update
    apt-get install build-essential fakeroot libncurses5-dev g++-arm-linux-gnueabi

We need to remove architecture validation from `/usr/bin/dpkg-gencontrol`.Backup this file first!

    if (field_get_dep_type($_)) {
        # Delay the parsing until later
    } elsif (m/^Architecture$/) {
        my $host_arch = get_host_arch();
        $fields->{$_} = $v;
        # validation removed.. :)
    } else {
        field_transfer_single($pkg, $fields);
    }

Download and prepare Linux kernel sources

    wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.10.26.tar.xz
    tar Jxf linux-3.10.26.tar.xz
    cd linux-3.10.26
    patch -p1 < ../0001-Support-for-Netgear-Stora.patch
    cp ../config-3.10.26-stora .config

Configure (optional step)

    make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabi- menuconfig

And finally build the kernel

    make -j`getconf _NPROCESSORS_ONLN` ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- EXTRAVERSION=-stora KDEB_PKGVERSION=1 KBUILD_DEBARCH=armel deb-pkg
