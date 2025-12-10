=====================================
Install Ubuntu on Raspberry Pi via PC
=====================================

This tutorial will guide you through installing Ubuntu on your Raspberry Pi,
using another machine (typically a PC) to flash the boot media. You will need
the following:

* A computer running Ubuntu, Windows, or macOS

* A Raspberry Pi (any model :doc:`supported by Ubuntu
  </how-to/raspberry-pi>`)

* Storage media; one of:

  - microSD card sized 16GB or larger, plus a microSD card reader/writer for
    your other computer

  - SSD drive with an appropriate USB3 adapter

  - NVMe drive with an appropriate HAT (Pi 5 only), plus an NVMe to USB3
    interface for your other computer

* A monitor for the Pi (optional for Ubuntu server)

* A keyboard for the Pi (optional for Ubuntu server)

* A mouse for the Pi (optional for Ubuntu server)

* An internet connection (wifi or ethernet)


Flashing the boot media
=======================

.. figure:: /images/rpi-other-software.png
    :alt: The rpi-imager application, displaying a selection of Pi 5, and
          Ubuntu 24.04.1 desktop

    The rpi-imager application, displaying a selection of Pi 5, and Ubuntu
    24.04.1 desktop

#. On your computer, install rpi-imager:

   * On Ubuntu, we recommend using the `snap of rpi-imager`_ as this will work
     regardless of your machine's architecture:

     .. code-block:: text

         sudo snap install rpi-imager

   * On Windows or macOS, visit the `Raspberry Pi software page`_ and download
     a copy of rpi-imager

#. Insert your blank media:

   * For microSD cards, insert the card into the card writer

   * For SSD or NVMe storage, connect the adapter to your media, then the
     adapter to a spare USB3 port on your computer

#. Start rpi-imager

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
   written successfully, then prompt you to remove your media

.. _snap of rpi-imager: https://snapstore.io/rpi-imager
.. _Raspberry Pi software page: https://www.raspberrypi.com/software/


Booting the Pi
==============

#. Ensure your Pi is not connected to a power source

#. Install your freshly written media in your Pi

   * A microSD card can simply go in the slot underneath the board

   * If your media is USB-connected, ensure you connect it to the one of the
     blue USB3 sockets (between the ethernet and the black USB sockets)

   * If your media is NVMe, ensure you have installed your M.2 HAT correctly,
     particularly that the PCIe flat cable, then install your blank NVMe drive
     in the HAT

#. Connect the monitor, keyboard, mouse, and ethernet to the Pi

#. Switch on the monitor

#. Switch on the Pi and wait for the boot screen to appear

#. Proceed with OS setup as normal:

   * On Ubuntu desktop images, the first-time setup wizard will guide you
     through locale selection and user creation

   * On Ubuntu server images, `cloud-init`_ will handle initial user creation
     and setup. The default username and password can be customized by
     rpi-imager, but if this is skipped the default username and password will
     be *ubuntu*
