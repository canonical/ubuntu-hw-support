.. _install-ubuntu-on-the-milk-v-mars-cm-and-cm-lite:

Install Ubuntu on the Milk-V Mars CM and CM Lite
================================================

The `Milk-V Mars CM`_ is a RISC-V based :term:`SBC`.

Using the pre-installed server image
------------------------------------

#. Download one of the supported images:

   .. ubuntu-images:
       :releases: noble
       :archs: riscv64
       :matches: (jh7110\.img.xz)

#. Flash the pre-installed server image to a USB drive.

#. Attach the USB drive.

#. Connect a USB UART adapter to the :term:`UART` on the :term:`GPIO` header
   (see `UART console`_ and :ref:`connect-to-a-uart-console`)

#. Power on the board

#. When "Hit any key to stop autoboot" is displayed, press :kbd:`Enter`

#. Reset the U-Boot environment with the following command:

   .. code-block:: text

       env erase

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

#. Download package package u-boot-starfive from
   https://launchpad.net/ubuntu/+source/u-boot/

#. Unpack the debian package with

   .. code-block:: bash

       dpkg -x u-boot-starfive*.deb u-boot-starfive

#. Copy the following files from package u-boot-starfive to a USB drive:

   * u-boot-starfive/usr/lib/u-boot/starfive_visionfive2/u-boot.bin
   * u-boot-starfive/usr/lib/u-boot/starfive_visionfive2/u-boot-spl.bin.normal.out

#. Connect a USB UART adapter to the :term:`UART` on the :term:`GPIO` header
   (see `UART console`_ and :ref:`connect-to-a-uart-console`)

#. Power on the board

#. When "Hit any key to stop autoboot" is displayed, press :kbd:`Enter`

#. Enter the following commands to flash U-Boot to SPI:

   .. code-block:: text

       sf probe
       load usb 0:1 $kernel_addr_r u-boot-spl.bin.normal.out
       sf update $kernel_addr_r 0 $filesize
       load usb 0:1 $kernel_addr_r u-boot.itb
       sf update $kernel_addr_r 0x100000 $filesize

#. Switch the board off

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

#. Flash the live server image to a USB drive.

#. Attach the USB drive.

#. Connect a USB UART adapter to the UART on the GPIO header (see
   `UART console`_ and :ref:`connect-to-a-uart-console`)

#. Power on the board

#. If no prior operating system installation exists, U-Boot will start up
   :term:`GRUB` on the USB drive. In this case you can skip the next step.

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

.. note::

    U-Boot does not allow the operating system to write :term:`UEFI` variables.
    You can do this manually using U-Boot's :command:`eficonfig` command.


NVMe support
------------

For attaching an NVMe drive the Waveshare CM4-IO-BASE-A board can be used.

.. warning::

    On the Raspberry Pi Compute Module 5 IO Board NVMe drives do not work.

    The Milk-V Mars CM does not drive the line PCIE_PWR_EN to enable
    power to the m.2 connector.

Boot source selection
---------------------

The Milk-V Mars CM board can boot firmware from SPI flash or UART.
The boot source is selected via the nRPIBOOT line. When connected to ground,
booting from UART is selected.

On the Waveshare CM4-IO-BASE-A board a switch labeled "BOOT" is connected to
the nRPIBOOT line. Switch it to to "OFF" to boot from SPI flash and to "ON" to
boot from UART.

.. image:: /images/milk-v-mars-cm-boot-select.jpg
    :width: 20em
    :alt: Boot select

UART console
------------

The :term:`UART` is available on the Waveshare CM4-IO-BASE-A board via the
:term:`GPIO` connector. Assuming the typical coloring of USB to :term:`TTL`
serial adapters the following connections have to be made:

=========== ==========
Board       Adapter
=========== ==========
GND, pin  6 GND, black
TX,  pin  8 RX,  white
RX,  pin 10 TX,  green
=========== ==========

Do not connect the red 3.3 V wire.

.. image:: /images/milk-v-mars-cm-uart.jpg
    :width: 20em
    :alt: UART

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

* While the 3 USB 3.0 ports are working, the USB 2.0 port is not supported by
  the 6.8 kernel


.. _Milk-V Mars CM: https://milkv.io/mars-cm
