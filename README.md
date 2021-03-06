Ubuntu on Gumstix
=================

This repository provides a method to build an Ubuntu image for various
Gumstix boards such as Overo COMs, DuoVero COMs, and the Pepper SBC.

Prebuilt Images
---------------

If you want to get started with Ubuntu using one of our prebuilt images, you can download
these prebuilt console images for Ubuntu 15.04 for [Overo][8], [Duovero][9] and [Pepper][10].
Flash them on to the SD card and you are good to go!

	# First unmount the SD card before flashing the image
	$ xz -d <image.xz> # Uncompress the image
	$ sudo dd if=/path/to/the/image/file of=/dev/sdX bs=4K

Build Instructions
------------------

If you want to build everything from scratch:

1. Install the required tools on your host system:

        $ sudo apt-get install -y git live-build qemu-user-static gcc-arm-linux-gnueabihf

2. Fetch this repository.

        $ git clone git://github.com/gumstix/live-build.git
	$ cd live-build

3. Choose an image type and machine for which to build.  This repository
   supports the Gumstix *overo*, *duovero*, and *pepper* machines.

	Image Name      | Description
	----------------|-------------
	*vivid-console* | A developer-oriented console image of Ubuntu 15.04
	*vivid-lxde*    | Ubuntu 15.04 "Vivid Vervet" with a lightweight desktop environment (LXDE)
	*sid-console*	| Debian Sid console image

4. *Make* it!  You will be prompted for a super-user password then go get some
   coffee.

        $ make MACHINE=<machine> IMAGE=<image> -j4

5. Everything proceeding correctly, binaries for the u-boot bootloader along
   with a root filesystem tarball will be created.  Insert a microSD card to
   your development machine, note the drive name, and then format it.
   **Warning**: this erases anything currently on the microSD card!

		$ # substitute the path to the drive e.g. /dev/sdd or /dev/mmcblk0 (not the
		$ # path of a partition e.g. /dev/sdd1 or /dev/mmcblk0p1) in place of <drive>
		$ MACHINE=<machine> IMAGE=<image> scripts/mkcard.sh <drive name>

6.	Alternatively, you have the option of creating a dd-able image after the
	build process is complete using the mkiso.sh script present in the scripts/
	directory. Before using the script, make sure that the following tools are
	installed: qemu-img, mkfs, kpartx, sfdisk, losetup.

		$ sudo apt-get install qemu-utils dosfstools kpartx

	Then, run the script:

		$ MACHINE=<machine> IMAGE=<image> scripts/mkiso.sh

	This will create a [machine]-[image].img file of size 4GB which needs to be
	flashed on the microSD card.

		# First unmount the SD card
		$ sudo dd if=/path/to/the/image/file of=/dev/sdX bs=4K

	If you have an SD card bigger than 4GB, you can resize the image after flashing
	it on the SD card using *gparted*

		$ sudo apt-get install gparted
		$ sudo gparted /dev/sdX

	Now click on the partition you want to resize and select Partition -> Resize/Move from the menu.

7. Once the microSD card has been written, insert it into your Gumstix system, login
   as *gumstix* with password *gumstix* and start playing with your new Ubuntu/Debian
   image.

Customization
-------------
There are numerous ways to customize this root file system.

**1. Change distro, release, or image type.**

As well as Ubuntu images, the [Live Build][1] tool can build images for a
number of Debian-based distributions (e.g. Debian, [Kali][2]) each with different
release versions (e.g. *wheezy*, *utopic*).  Depending on the image *flavour*
and the installed packages, images can provide a variety of different desktop
environments or a stripped down console environement. Start customizing by
copying an existing image directory and adjusting the *config*.

    $ cp -r images/vivid-console images/super-gumstix
    $ sensible-editor images/super-gumstix/auto/config
    $ make MACHINE=<machine> IMAGE=super-gumstix -j4

There are some [great][3] [samples][4].

**2. Pre-install additional packages.**

To include additional packages in a custom image, simply add the package names
(as the would be passed in an *apt-get*) to the list of packages in the
*customization* directory. Carrying on from the previous customization:

    $ echo 'vim' >> images/super-gumstix/customiztion/package-lists/gumstix.list.chroot
    $ make MACHINE=<machine> IMAGE=super-gumstix -j4

As the contents of the *customization* directory get copied over to the
*config* generated by Live Build's configuration step, it is possible to add
locally created *deb* packages by dropping them in a
*customization/packages.chroot* directory.  It is even [possible][5] to extend
the list of package repositories.

**3. Change the Root File System.**

It is possible to run scripts during the construction of root file system as if
the image was actually running (i.e. within a *chroot* under QEMU emulation).
See the *customization/hooks* directory for some samples.

It is also possible to do some boot-time configuration using the *live-config*
[tool][6] (different than *live-build config*).

------------------------------------------------------------------------------
**Note:**

This is not a *build-everything-from-scratch* tool.  This tool
assembles a root file system from pre-compiled packages and provides a
useful framework for adjusting configuration files.  If something
additional needs to be compiled, there are three options:
 * compile it natively (and possibly, create a .deb for future inclusion)
 * cross-build the code---there are a litany of techniques ranging from
   building on emulated ARM hardware (qemu) through a full cross-building
   environment (it gets a [little complicated][6])
 * Use [Yocto][7]!  The Yocto environment excels at cross-building where Debian
   prefers native compilation.

------------------------------------------------------------------------------

[1]: http://live.debian.net/devel/live-build/
[2]: http://docs.kali.org/development/live-build-a-custom-kali-iso
[3]: https://git.linaro.org/ci/ubuntu-build-service.git
[4]: https://wiki.debian.org/InstallingDebianOn/TI/BeagleBone
[5]: https://git.linaro.org/ci/ubuntu-build-service.git/blob/HEAD:/utopic-armhf-nano/customization/archives/linaro-overlay-ppa.list.chroot
[6]: http://live.debian.net/devel/live-config/
[6]: https://wiki.linaro.org/Platform/DevPlatform/CrossCompile/CurrentPackageCrossBuildStatus
[7]: https://github.com/gumstix/yocto-manifest
[8]: https://catalina.gumstix.com/binaries/?sort=-last_updated&search=ubuntu-overo-master-iso
[9]: https://catalina.gumstix.com/binaries/?sort=-last_updated&search=ubuntu-duovero-master-iso
[10]: https://catalina.gumstix.com/binaries/?sort=-last_updated&search=ubuntu-pepper-master-iso
