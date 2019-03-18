Converting ramdisk *.img* file for u-boot
=========================================

Usually you get an *.img* file from official debian repositories. U-boot can't handle this to use it as a ramdisk file.
To convert an *.img* file to an *uInitrd* for uboot, just run the following command inside a unix terminal:

.. code-block:: shell

	mkimage -A arm -T ramdisk -C none -n uInitrd -d /path/to/initrd.img /path/to/uInitrd
