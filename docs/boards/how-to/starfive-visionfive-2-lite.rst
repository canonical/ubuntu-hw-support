.. _install-ubuntu-on-the-starfive-visionfive-2-lite:

Install Ubuntu on the StarFive VisionFive 2 Lite
================================================

The `StarFive VisionFive 2 Lite`_ is a RISC-V based :term:`SBC`.

.. warning::

    The StarFive VisionFive 2 Lite is not yet supported by official Ubuntu
    images.

Using the pre-installed server image
------------------------------------

#. Download one of the supported images.

#. Flash the pre-installed server image to a microSD card (see
   :ref:`flash-images-to-a-microsd-card`)

#. Insert the microSD card into the board

#. Set the boot source to the microSD card (see `Boot source selection`_)

#. Connect a USB UART adapter to the :term:`UART` on the :term:`GPIO` header
   (see `UART console`_ and :ref:`connect-to-a-uart-console`)

#. Power on the board

#. When "Hit any key to stop autoboot" is displayed, press :kbd:`Enter`

#. Reset the U-Boot environment with the following commands:

   .. code-block:: text

       env default -f -a
       env save

#. Power cycle the board

#. Wait for an output line confirming that `cloud-init`_ has finished running;
   this service is responsible for generating SSH keys, and creating the
   default user:

   .. code-block:: text

       [   35.682018] cloud-init[909]: Cloud-init v. 24.1.3-0ubuntu3 finished at Tue, 23 Apr 2024 07:44:59 +0000. Datasource DataSourceNoCloud [seed=/var/lib/cloud/seed/nocloud-net][dsmode=net].  Up 35.65 seconds

#. Login with the user *ubuntu* and the default password *ubuntu*; you will be
   asked to choose a new password


Using the live server image
---------------------------

The live installer image is used to install Ubuntu to an :term:`eMMC`, USB, or
:term:`NVMe` drive. To boot the live image, U-Boot must be installed on the SPI
flash.

.. warning::

    The vendor U-Boot is not compatible with :term:`EBBR` and cannot boot
    Ubuntu without manual changes.


Install U-Boot to the SPI flash
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Flash the pre-installed server image to a microSD card (see
   :ref:`flash-images-to-a-microsd-card`)

#. Insert the microSD card into the board

#. Set the boot source to the microSD card (see `Boot source selection`_)

#. Connect a USB UART adapter to the :term:`UART` on the :term:`GPIO` header
   (see `UART console`_ and :ref:`connect-to-a-uart-console`)

#. Power on the board

#. When "Hit any key to stop autoboot" is displayed, press :kbd:`Enter`

#. Enter the following commands to flash U-Boot to SPI:

   .. code-block:: text

       sf probe
       load mmc 1:1 $kernel_addr_r /usr/lib/u-boot/starfive_visionfive2/u-boot-spl.bin.normal.out
       sf update $kernel_addr_r 0 $filesize
       load mmc 1:1 $kernel_addr_r /usr/lib/u-boot/starfive_visionfive2/u-boot.itb
       sf update $kernel_addr_r 0x100000 $filesize

#. Switch the board off

#. Set the boot source to the SPI flash (see `Boot source selection`_)

#. Power on the board

#. When "Hit any key to stop autoboot" is displayed, press :kbd:`Enter`

#. Enter the following commands to reset the U-Boot environment:

   .. code-block:: text

       env default -f -a
       env save

#. Switch the board off


Boot the live server image
~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Download one of the supported images.

#. Flash the live server image to a microSD card (see
   :ref:`flash-images-to-a-microsd-card`)

#. Insert the microSD card into the board

#. Ensure the boot source is SPI flash (see `Boot source selection`_), *not*
   microSD card

#. Connect a USB UART adapter to the UART on the GPIO header (see
   `UART console`_ and :ref:`connect-to-a-uart-console`)

#. Power on the board

#. If no prior operating system installation exists, U-Boot will start up
   :term:`GRUB` on the microSD card. Go to step 9

#. If a prior operating system exists, press :kbd:`Enter` when "Hit any key to
   stop autoboot" is displayed, and enter:

   .. code-block:: text

       load mmc 1:1 $fdt_addr_r dtb/starfive/jh7110-starfive-visionfive-2-v1.3b.dtb
       # for the 1.2a version of the board:
       # load mmc 1:1 $fdt_addr_r dtb/starfive/jh7110-starfive-visionfive-2-v1.2a.dtb
       load mmc 1:1 $kernel_addr_r EFI/boot/bootriscv64.efi
       bootefi $kernel_addr_r $fdt_addr_r

#. From the GRUB menu, select "Try or Install Ubuntu Server"

#. Loading the installer takes some time. Once it is loaded, follow the
   `Ubuntu Server installation tutorial
   <https://ubuntu.com/tutorials/install-ubuntu-server>`_

#. Once the installation is complete, reboot the board without removing the
   microSD card

.. note::

    U-Boot does not allow the operating system to write :term:`UEFI` variables.
    You can do this manually using U-Boot's :command:`eficonfig` command.


Boot source selection
---------------------

The StarFive VisionFive 2 board can boot firmware from SPI flash or UART.
eMMC, or UART. The boot source is selected via DIP switches.

To boot from UART the button marked `Boot` must be held pressed.

Cloud-init seed
---------------

Sample files for a cloud-init seed are present on the FAT partition labeled
"CIDATA". See :ref:`configure-your-board-for-headless-use` for more information.


UART console
------------

The :term:`UART` console is accessible via the :term:`GPIO` header. Assuming
the typical coloring of USB to :term:`TTL` serial adapters the following
connections have to be made:

=========== ==========
Board       Adapter
=========== ==========
GND, pin  6 GND, black
TX,  pin  8 RX,  white
RX,  pin 10 TX,  green
=========== ==========

.. image:: /images/starfive-visionfive-2-lite-gpio.jpg
    :width: 15em
    :alt: GPIO

Connect with the following settings (see :ref:`connect-to-a-uart-console`):

* 115200 baud
* 8 data bits
* no parity
* 1 stop bit
* no flow control


Limitations
-----------

* The on-board GPU is not supported

* PCIe support is incomplete: an NVMe drive can be used, but Wi-Fi cards and
  external GPUs don't work


.. _StarFive VisionFive 2 Lite: https://www.kickstarter.com/projects/starfive/visionfive-2-lite-unlock-risc-v-sbc-at-199
