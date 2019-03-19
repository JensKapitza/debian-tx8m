Debian Stretch headless on your development kit
===============================================

Before you start
----------------

1. Unpack your development kit and insert the TX8M module carefully into your development kit.
2. Connect the micro USB connector to the OTG port.
3. Put the bootmode jumper in place, to bring the board to boot-mode.
4. Clone our github repository at branch *debian-headless* to get the files neccessary: `https://github.com/karo-electronics/debian-tx8m <https://github.com/karo-electronics/debian-tx8m>`_.

Getting the root filesystem
---------------------------

The root filesystem is not included inside the github repository, because it is too large.
Please download it from `here <https://www.karo-electronics.de/fileadmin/download/tx8m-devkit-debian/debian-stretch-headless-rootfs/rootfs.tar.gz>`_.
Put the archive file into the repository folder `/rootfs`.

Getting uuu
-----------

The **u**\ niversal **u**\ pdate **u**\ tility is used to program your board.

1. Get ``uuu``, ``uuu.exe`` and ``libusb-1.0.dll`` from NXP's official repository `here <https://github.com/NXPmicro/mfgtools/releases/tag/uuu_1.2.0>`_.
2. Put these three files inside the root of our repository you cloned.

Running uuu to program your board
---------------------------------

1. Plug the micro USB, that is connected into the OTG port of your board, into your computer.

2. Open a command line inside the repository root folder on your computer.

3. Press the reset button on your board.

4. Run the following:

.. code-block:: shell

  .\uuu.exe -v

This will start *uuu* using the *uuu.auto* script and gives you information about the setup process.

When *uuu* has run with everything okay, you can just remove the bootmode jumper and unplug the micro USB cable.

Booting
-------

1. Connect the 12V DC power.

2. Connect with the USB to UART cable to use the system's terminal. See `this <../faq/general/terminal.html>`_.

3. The login works with ``root:root``.

4. If you wish to install a desktop environment you can have a look at `the manual installation guide <../manual-installation/index.html#installing-desktop-environment>`_.
