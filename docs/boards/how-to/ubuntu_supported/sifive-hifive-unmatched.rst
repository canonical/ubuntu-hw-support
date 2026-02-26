.. _install-ubuntu-on-the-sifive-hifive-unmatched:

Install Ubuntu on the SiFive HiFive Unmatched
=============================================

The `SiFive HiFive Unmatched`_ is a RISC-V based :term:`SBC`.


Using the pre-installed server image
------------------------------------

#. Download one of the supported images:

   .. ubuntu-images::
       :releases: noble-
       :suffixes: +unmatched

#. Flash the pre-installed server image to a microSD card (see
   :ref:`flash-images-to-a-microsd-card`)

#. Insert the microSD card into the board

#. Set the boot source to the microSD card (see `Boot source selection`_)

#. Connect to the :term:`UART` console (see `UART console`_ and
   :ref:`connect-to-a-uart-console`)

#. Power on the board

#. Wait for an output line confirming that `cloud-init`_ has finished running;
   this service is responsible for generating SSH keys, and creating the
   default user:

   .. code-block:: text

       [   35.682018] cloud-init[909]: Cloud-init v. 24.1.3-0ubuntu3 finished at Tue, 23 Apr 2024 07:44:59 +0000. Datasource DataSourceNoCloud [seed=/var/lib/cloud/seed/nocloud-net][dsmode=net].  Up 35.65 seconds

#. Login with the user *ubuntu* and the default password *ubuntu*; you will be
   asked to choose a new password


Using the live server image
---------------------------

The live installer image is used to install Ubuntu to an :term:`NVMe` drive
attached to the M.2 slot on the board.

#. Download one of the supported images:

   .. ubuntu-images::
       :releases: noble
       :image-types: live-server
       :archs: riscv64

#. Flash the pre-installed server image to a microSD card (see
   :ref:`flash-images-to-a-microsd-card`)

#. Insert the microSD card into the board

#. Set the boot source to the microSD card (see `Boot source selection`_)

#. Connect to the :term:`UART` console (see `UART console`_ and
   :ref:`connect-to-a-uart-console`)

#. Power on the board; if there is no operating system already on the NVMe
   drive, skip to step 9

#. When "Hit any key to stop autoboot" is displayed, press :kbd:`Enter`

#. Enter the following commands to boot the installer:

   .. code-block:: text

       pci enum
       nvme scan
       load mmc 0:1 $fdt_addr_r dtb/sifive/hifive-unmatched-a00.dtb
       load mmc 0:1 $kernel_addr_r EFI/boot/bootriscv64.efi
       bootefi $kernel_addr_r $fdt_addr_r

#. From the GRUB menu, select "Try or Install Ubuntu Server"

#. Loading the installer takes some time. Once it is loaded, follow the
   `Ubuntu Server installation tutorial
   <https://ubuntu.com/tutorials/install-ubuntu-server>`_

#. Once the installation is complete, reboot the board without removing the
   microSD card

#. Press :kbd:`Enter` when "Hit any key to stop autoboot" is displayed, and
   enter:

   .. code-block:: text

       pci enum
       nvme scan
       efidebug boot add -b 0001 'Ubuntu' nvme 0:1 /EFI/ubuntu/grubriscv64.efi
       efidebug boot order 0001
       bootefi bootmgr

#. On the next boot, U-Boot will automatically start GRUB

.. note::

    U-Boot does not allow the operating system to write :term:`UEFI` variables.
    You can do this manually using U-Boot's :command:`eficonfig` command.

Installing the Gnome desktop
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you have a graphics card installed, you may want to use a graphical desktop.
The Gnome desktop can be installed with

.. code-block:: text

    sudo apt update
    sudo apt install ubuntu-desktop
    sudo systemctl start gdm3


Boot source selection
---------------------

The SiFive HiFive Unmatched board can boot firmware from the built-in flash
memory, or from the microSD card. The boot source is selected via DIP switches.

=====  =====  =====  =====  ==============================
MSEL3  MSEL2  MSEL1  MSEL0  Boot Source
=====  =====  =====  =====  ==============================
0      1      0      1      QSPI0 Flash
0      1      1      0      QSPI0 Flash
0      1      1      1      QSPI1 Flash
1      0      0      0      QSPI1 microSD Card
1      0      0      1      QSPI2 Flash
1      0      1      0      QSPI0 Flash
1      0      1      1      QSPI2 microSD Card (*default*)
1      1      0      0      QSPI1 Flash
1      1      0      1      QSPI1 Flash
1      1      1      0      QSPI0 Flash
1      1      1      1      QSPI0 Flash
=====  =====  =====  =====  ==============================

On Ubuntu, the only supported setting is the default QSPI2 microSD card
setting of 1011 as illustrated below:

.. image:: /images/unmatched-boot-source-sd.jpg
    :width: 15em
    :alt: Boot from microSD card with the default setting of 1011

.. warning::

    On version 2 of the HiFive Unmatched boards, the silk screen showing the
    orientation of the MSEL DIP switches is incorrect and should be ignored
    (follow the settings on the image above). The suffix of the board assembly
    number label indicates the board version, e.g. the 2 in HF105-ASSY-2A0
    indicates board version 2.

Cloud-init seed
---------------

Sample files for a cloud-init seed are present on the FAT partition labeled
"CIDATA". See :ref:`configure-your-board-for-headless-use` for more information.


UART console
------------

The board makes both :term:`JTAG` and :term:`UART` available over the micro USB
connector located adjacent to the microSD card slot, hence this board does
*not* require a USB-UART adapter. Connect with a regular micro-USB cable.

When connected, it appears as two separate devices in Linux. The first UART
(typically :file:`/dev/ttyUSB0`) is JTAG, and the second (typically
:file:`/dev/ttyUSB1`) is the UART console. Connect with the following settings
(see :ref:`connect-to-a-uart-console`):

* 115200 baud
* 8 data bits
* no parity
* 1 stop bit
* no flow control

However, please note the boot ROM messages are written at 57600 baud.

.. _SiFive HiFive Unmatched: https://www.sifive.com/boards/hifive-unmatched
