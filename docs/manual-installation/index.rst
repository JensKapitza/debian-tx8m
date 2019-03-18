Manual installation
===================

Before you start
----------------

Insert the TX8M module carefully into your development kit.
Unpack your development kit, connect the 12V DC power and the micro USB connector to the OTG port.
Clone our github repository at branch `u-boot` to get the files neccessary: `https://github.com/karo-electronics/debian-tx8m <https://github.com/karo-electronics/debian-tx8m>`_.

Getting uuu
-----------

The **u**\ niversal **u**\ pdate **u**\ tility is used to program your board.

1. Get ``uuu``, ``uuu.exe`` and ``libusb-1.0.dll`` from NXP's official repository `here <https://github.com/NXPmicro/mfgtools/releases/tag/uuu_1.2.0>`_.
2. Put these three files inside the root of our repository you cloned.

Running uuu to flash u-boot
---------------------------

1. Plug the micro USB, that is connected into the OTG port of your board, into your computer.

2. Open a command line inside the repository root folder on your computer.

3. Press the reset button on your board.

4. Run the following:

.. code-block:: shell

  .\uuu.exe -v

This will start *uuu* using the *uuu.auto* script and gives you information about the setup process.

When *uuu* has run with everything okay, you can just remove the bootmode jumper and unplug the micro USB cable.

Installing debian from net with U-Boot
--------------------------------------

**Ramdisk, kernel, devicetree**

There are three files inside the repository you will need to start the net-boot:

* The kernel, located in ``kernel/Image-imx8mm-tx8m``
* The dtb, located in ``dtb/imx8mm-tx8m-1610-devkit.dtb``
* The initram file, located in ``initram/uInitrd``

Setup a NFS share and put these three files inside it, to access them via TFTP in u-boot.

**NOTE:** The initram file is for the debian-stretch net installer. If you'd like to try other versions you have to convert the initram *.img* files to fit u-boot.
This is explained `here <../faq/general/ramdisk-uboot.html>`_. It's not guaranteed that other versions or systems will work with our module.

**Preparing your device**

1. Connect the 12V DC power and a network cable to your board.
2. Connect the USB to UART cable and connect to your board with a terminal to boot into U-Boot. Detailed description of how to connect to a terminal is `here <../faq/general/terminal.html>`_.

**Configuring your U-Boot**

You should now be inside the u-boot terminal. You need to run some commands to setup the netboot.

**NOTE:** When you copy multiple lines of the commands, this might work, but u-boot can also "swollow" some characters. To avoid this, just copy every single line and run it. You might use an editor to prepare the commands to your needs.

1. Setting addresses for loading netboot:

.. code-block:: shell

    setenv kernel_addr_r 0x40480000
    setenv fdt_addr_r 0x43000000
    setenv ramdisk_addr_r 0x50000000

2. Loading kernel, ramdisk and dtb:

.. code-block:: shell

  # get an ip-address
	dhcp
  # connect to your nfs-share
	setenv serverip {nfs-ip}
  # load files with tftpboot
	tftpboot ${kernel_addr_r} Image-imx8mm-tx8m
	tftpboot ${fdt_addr_r} imx8mm-tx8m-1610-devkit.dtb
	tftpboot ${ramdisk_addr_r} uInitrd

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

After installation you will be warned, that there's no bootloader. This is okay for us, so just keep in mind at which device the installation is and hit enter. The system should boot you back into u-boot.

**Booting your installation for the first time**

1. Set bootargs to boot from eMMC partition:

.. code-block:: shell

	setenv bootargs 'console=ttymxc0,115200 earlycon=ec_imx6q,0x30860000,115200 root=/dev/mmcblk0p2 rootwait rw ip=dhcp'

2. Load kernel and dtb again:

.. code-block:: shell

	dhcp
	setenv serverip {nfs-ip}
	tftpboot ${loadaddr} Image-imx8mm-tx8m
	tftpboot ${fdt_addr} imx8mm-tx8m-1610-devkit.dtb

3. Boot:

.. code-block:: shell

	booti ${loadaddr} - ${fdt_addr}

Your installation should boot, just login.

**Kernel and devicetree into eMMC**

1. Install NFS client and mount your nfs-share:

.. code-block:: shell

	apt install nfs-common
	mount {nfs-ip}:/tftpboot /mnt

2. Copy kernel and dtb into /boot

.. code-block:: shell

	cp /mnt/Image-imx8mm-tx8m /boot/
	cp /mnt/imx8mm-tx8m-1610-devkit.dtb /boot/

3. Reboot your system, now it just should startup itself.

**Enabling display-support**

.. raw:: html

  <p style="color: red;"><b>Display setup documentation is in progress at the moment, actually it won't work...</b></p>


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
