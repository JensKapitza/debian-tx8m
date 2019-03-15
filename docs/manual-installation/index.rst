Manual installation
===================

.. raw:: html

  <p style="color: red;"><b>This page is under construction...</b></p>


Before you start
----------------

Insert the TX8M module carefully into your development kit.
Unpack your development kit, connect the 12V DC power and the micro USB connector to the OTG port.
Clone our github repository to get the files neccessary: `https://github.com/karo-electronics/debian-tx8m <https://github.com/karo-electronics/debian-tx8m>`_.

Flashing U-Boot to your module
------------------------------

- TODO: uuu for this, documentation for this.

Installing debian from net with U-Boot
--------------------------------------

**Preparing your device**

1. Connect a network cable.
2. Connect to your starter kit via PUTTY or another terminal and boot into U-Boot.

**Ramdisk, kernel, devicetree**

Download the ramdisk file `here <http://ftp.halifax.rwth-aachen.de/debian/dists/stretch/main/installer-arm64/current/images/netboot/debian-installer/arm64/initrd.gz>`_.
Extract it, to get the *initrd* file. On a unix machine use *mkimage* to convert the image for u-boot.

.. code-block:: shell

	mkimage -A arm -T ramdisk -C none -n uInitrd -d /path/to/initrd /path/to/uInitrd

Put the generated *uInitrd*, our kernel(4.9 lothar) and devicetree to a directory of your choice inside a NFS-share your u-boot can access.


**Configuring your U-Boot**

The following commands work for Ka-Ro diretcly.

1. Setting addresses for loading netboot:

.. code-block:: shell

    setenv kernel_addr_r 0x40480000
    setenv fdt_addr_r 0x43000000
    setenv ramdisk_addr_r 0x50000000

2. Loading kernel, ramdisk and dtb:

.. code-block:: shell

	dhcp
	setenv serverip '192.168.1.9'
	tftpboot ${kernel_addr_r} /tx8m/Image_mx8mmevk
	tftpboot ${fdt_addr_r} /tx8m/imx8mm-tx8m-1610-mb7.dtb
	tftpboot ${ramdisk_addr_r} /tx8m/debian-installer/arm64/uInitrd

**Booting the installer**

Boot the net installer by typing:

.. code-block:: shell

	booti ${kernel_addr_r} ${ramdisk_addr_r}:${filesize} ${fdt_addr_r}

You will be guided through the installation. Make sure to:

* Install only headless debian with default settings
* Partition manual with following settings:

  .. code-block:: shell

	|p1,256mb,ext4,mount=/boot
	|p2,all-space,ext4,mount=/
	|p3,512mb,swap
* TODO insert screenshots of installation

After installation you will be warned, that there's no bootloader. This is okay for us, so just keep in mind at which device the installation is and hit enter. The system should boot you back into u-boot.

**Booting your installation for the first time**

1. Set bootargs to boot from eMMC partition:

.. code-block:: shell

	setenv bootargs 'console=ttymxc0,115200 earlycon=ec_imx6q,0x30860000,115200 root=/dev/mmcblk0p2 rootwait rw ip=dhcp'

2. Load kernel and dtb again:

.. code-block:: shell

	dhcp
	setenv serverip 192.168.1.9
	tftpboot ${loadaddr} /tx8m/Image_mx8mmevk
	tftpboot ${fdt_addr} /tx8m/imx8mm-tx8m-1610-mb7.dtb

3. Save the environment:

.. code-block:: shell

	saveenv

4. Boot:

.. code-block:: shell

	booti ${loadaddr} - ${fdt_addr}

Your installation should boot, just login.

**Kernel and devicetree into eMMC**

1. Install NFS client and mount TFTP-boot:

.. code-block:: shell

	apt install nfs-common
	mount 192.168.1.9:/volume1/trans/tftpboot /mnt

2. Copy kernel and dtb into /boot

.. code-block:: shell

	cp /mnt/tx8m/Image_mx8mmevk /boot/
	cp /mnt/tx8m/imx8mm-tx8m-1610-mb7.dtb /boot/

Just reboot your system, you should be in u-boot again.

**Booting from eMMC**

1. Change load-commands:

.. code-block:: shell

	setenv loadkernel 'ext4load mmc 0 ${loadaddr} Image_mx8mmevk'
	setenv loadfdt 'ext4load mmc 0 ${fdt_addr} imx8mm-tx8m-1610-mb7.dtb'
	saveenv

2. Now your module should boot debian itself, try:

.. code-block:: shell

	boot

**Enabling display-support**

You have to copy two folders from Lothar's root-filesystem to get the display working.

1. Mount it:

.. code-block:: shell

	mount 192.168.1.225:/tftpboot/KARO /mnt

2. Copy folders needed:

.. code-block:: shell

	cp -r /mnt/tx8/root /
	cp -r /usr/local/arm /usr/local

3. Test init script if display will turn on:

.. code-block:: shell

	/root/bin/dsi83-init

The display should turn on and display a blinking dash.

4. Editing init-script to our needs:

Open the */root/bin/dsi83-init* file with an editor and at the end of the file insert this:

.. code-block:: shell

	gpio_mode pe20 GPOUT 0

This will set our GPIO mode to 8bit.
**TODO/FIXME:** This only works as long, as the screen stays active, we need to find a way to enable this on every display sleep/wakeup.

5. Remove entry from *.autoexec* file.

At the end of */root/.autoexec* theres the script call for *dsi83-init*. This will run our script when you login. We need this to be executed before we login, so remove this line from the autoexec script.

**Installing and setting up the desktop**

The following lines will show you how to install the LXDE desktop environment for our system.

1. Install the package:

.. code-block:: shell

	apt install --no-install-recommends task-lxde-desktop

2. Set Xorg config file to use the framebuffer device:

Edit the file */etc/X11/xorg.conf*. If it doesn't exist, create it.
Add the following lines to enable framebuffer:

.. code-block:: shell

	Section "Device"
	    Identifier "FBDEV"
	    Driver "fbdev"
	    Option "fbdev" "/dev/fb0"
	#   Option "ShadowFB" "false"
	EndSection

3. Set the display init script to be executed at startup:

Edit */etc/lightdm/lightdm.conf*, uncomment the line with *display-setup-script* and set it to this:

.. code-block:: shell

	display-setup-script=/root/bin/dsi83-init


**Installing and setting up an on-screen keyboard**

1. Install the onboard keyboard package:

.. code-block:: shell

	apt install onboard

2. Edit */etc/lightdm/lightdm-gtk-greeter.conf*, and under *[greeter]* setction uncomment the *keyboard* line and set it to this:

.. code-block:: shell

	keyboard=onboard

3. If you want to have the keyboard enabled at startup also add this line:

.. code-block:: shell

	a11y-states=+keyboard
