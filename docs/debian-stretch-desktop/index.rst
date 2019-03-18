Debian Stretch desktop on your development kit
==============================================

Before you start
----------------

1. Unpack your development kit and insert the TX8M module carefully into your development kit.
2. Connect the micro USB connector to the OTG port.
3. Put the bootmode jumper in place, to bring the board to boot-mode.
4. Clone our github repository at branch *master* to get the files neccessary: `https://github.com/karo-electronics/debian-tx8m <https://github.com/karo-electronics/debian-tx8m>`_.

Getting the root filesystem
---------------------------

**NOTE:** The root filesystem works, but is very large. It will be optimized in the following weeks.

The root filesystem is not included inside the github repository, because it is too large.
Please download it from `here <https://www.karo-electronics.de/fileadmin/download/tx8m-devkit-debian/debian-stretch-desktop-rootfs/rootfs.tar.gz>`_.
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

1. Connect the 12V DC power, a network cable, and the touchscreen into the USB host port of the board. (At the moment network is neccessary to start Debian, we're working on it.)

2. Wait for your Debian desktop to appear. The login works with ``root:root``.

Terminal access from host computer
----------------------------------

See `this <../faq/general/terminal.html>`_.
