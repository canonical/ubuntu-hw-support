=======================================
Install Ubuntu on Raspberry Pi directly
=======================================

This tutorial will guide you through installing Ubuntu on your Raspberry Pi
without the need for an additional machine. You will need the following:

* A Raspberry Pi Model 4B or later

* Storage media; one of:

  - microSD card sized 16GB or larger

  - SSD drive with an appropriate USB3 adapter

  - NVMe drive with an appropriate HAT (Pi 5 only)

* A monitor

* A keyboard

* A mouse

* An ethernet based internet connection

.. note::

    This procedure will *not* operate over wifi as the bootloader cannot
    configure wifi at this time.


Hardware setup
==============

#. Connect your blank storage media

   * A microSD card can simply go in the slot underneath the board

   * If your media is USB-connected, ensure you connect it to the one of the
     blue USB3 sockets (between the ethernet and the black USB sockets)

   * If your media is NVMe, ensure you have installed your M.2 HAT correctly,
     particularly that the PCIe flat cable, then install your blank NVMe drive
     in the HAT

#. Connect the monitor, keyboard, mouse, and ethernet to the Pi

#. Switch on the monitor

#. Switch on the Pi and wait for the boot screen to appear


Software installation
=====================

.. figure:: /images/rpi-solo-software.png
    :alt: The Raspberry Pi boot-loader screen, displaying the prompt to
          hold SHIFT to start the installer

    The Raspberry Pi boot-loader screen, display the prompt to hold SHIFT to
    start the installer

#. When prompted, hold :kbd:`Shift` until the boot screen indicates that the
   installer (an embedded version of `rpi-imager`_) is being downloaded

#. Once the installer is downloaded, it will launch automatically. When it
   appears, wait until a valid IP address is displayed in the lower half of the
   screen

#. Click the first button, ``CHOOSE DEVICE``, to select your model of Pi. This
   is optional, but limits the OS image selection to those images compatible
   with your board

#. Click the second button, ``CHOOSE OS`` to select an OS image. For the
   purposes of this tutorial, we will select:

   * Other general-purpose OS

   * Ubuntu

   * Ubuntu Desktop 24.04.1 LTS (64-bit)

#. Click the third button, ``CHOOSE STORAGE`` to select the destination media.
   In our case there's only a single choice, our SD card

#. Click ``NEXT`` to write the card, and choose ``YES`` to indicate you wish to
   overwrite everything on the target media

#. Writing the image, especially a desktop image, takes some considerable time
   depending on the speed of your media, and your Internet connection. However,
   be prepared to wait at least 10 minutes

#. Once writing is complete, it will re-read the media to verify the image was
   written successfully, then immediately reboot into your selected OS

#. Proceed with OS setup as normal:

   * On Ubuntu desktop images, the first-time setup wizard will guide you
     through locale selection and user creation

   * On Ubuntu server images, `cloud-init`_ will handle initial user creation
     and setup. The default username and password can be customized by
     rpi-imager, but if this is skipped the default username and password will
     be *ubuntu*

.. _rpi-imager: https://www.raspberrypi.com/software/
