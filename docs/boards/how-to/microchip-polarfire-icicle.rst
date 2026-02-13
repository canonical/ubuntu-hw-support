.. _install-ubuntu-on-the-microchip-polarfire-soc-icicle-kit:

Install Ubuntu on the Microchip Polarfire SoC Icicle Kit
========================================================


FPGA Design
-----------

Ubuntu's kernel 5.19 requires the `Icicle Kit Reference Design`_ v2022.10 or
later. The archive :file:`MPFS_ICICLE_BASE_DESIGN_{yyyy}_{mm}.zip` contains a
job file and digest file which can be used. This job file also includes the
latest version of the `Hart Software Services`_ at the time of the release.
`FlashPro Express`_ is needed to program the job file to the FPGA. `FlashPro
Express`_ standalone is bundled with the `Programming and Debug Tools`_.

.. _Hart Software Services: https://github.com/polarfire-soc/hart-software-services/releases
.. _HSS sources: https://github.com/polarfire-soc/hart-software-services
.. _FlashPro Express: https://www.microchip.com/en-us/products/fpgas-and-plds/fpga-and-soc-design-tools/programming-and-debug/flashpro-express
.. _Programming and Debug Tools: https://www.microchip.com/en-us/products/fpgas-and-plds/fpga-and-soc-design-tools/programming-and-debug


Hart Software Services
----------------------

The first stage boot loader of the `PolarFire`_ :term:`SoC` FPGA Icicle Kit is
called `Hart Software Services`_ (HSS). Ubuntu's kernel 5.19 requires version
2022.10 or later of this software. Microchip provides a `FlashPro`_ Express
programming job that can be used to update the HSS on a kit to the latest
version without having to build the software, this job is provided with the
FPGA design. Otherwise the `HSS sources`_ are available from GitHub and can be
built and programmed independently of the design using the command line or the
`SoftConsole IDE`_.

.. _FlashPro: https://www.microchip.com/en-us/products/fpgas-and-plds/fpga-and-soc-design-tools/programming-and-debug/flashpro
.. _Icicle Kit Reference Design: https://github.com/polarfire-soc/icicle-kit-reference-design/releases
.. _PolarFire: https://www.microchip.com/en-us/products/fpgas-and-plds/fpgas/polarfire-fpgas
.. _SoftConsole IDE: https://www.microchip.com/en-us/products/fpgas-and-plds/fpga-and-soc-design-tools/soc-fpga/softconsole


Using the pre-installed server image
------------------------------------

#. Download one of the supported images:

   .. ubuntu-images::
       :releases: jammy-
       :suffixes: +icicle

#. Flash the pre-installed server image to a microSD card or the Icicle Kit on
   board eMMC (see :ref:`flash-images-to-a-microsd-card`). The board uses package
   ``u-boot-microchip`` as second stage boot loader. It is installed in
   the partition with label "loader" on the above pre-installed server disk
   image. Ubuntu's U-Boot has been set up to use all U54 cores for Linux. If
   you want to use one or more cores for a real time operating system in
   parallel to Ubuntu, you will have to uninstall this package and install your
   own U-Boot onto the partition loader

#. Insert the microSD card into the board

#. Connect to the :term:`UART` console (see `UART console`_ and
   :ref:`connect-to-a-uart-console`)

#. Power on the board

   .. note::

       Pressing the reset button does not reset all devices into their initial
       state. Booting Ubuntu has been failing when inserting the SD-card and
       pressing the reset button while the vendor SDK was running

#. Wait for an output line confirming that `cloud-init`_ has finished running;
   this service is responsible for generating SSH keys, and creating the
   default user:

   .. code-block:: text

        [  291.932176] cloud-init[1282]: Cloud-init v. 22.3.4-0ubuntu1 finished at Thu, 20 Oct 2022 08:25:11 +0000. Datasource DataSourceNoCloud [seed=/var/lib/cloud/seed/nocloud-net][dsmode=net].  Up 291.79 seconds

#. Login with the user *ubuntu* and the default password *ubuntu*; you will be
   asked to choose a new password


Cloud-init seed
---------------

Sample files for a cloud-init seed are present on the FAT partition labeled
"CIDATA". See :ref:`configure-your-board-for-headless-use` for more information.


UART console
------------

The board features a USB to quad :term:`UART` bridge controller (J11), hence
this is one of the boards that does *not* require a USB-UART adapter. Connect
with a regular micro-USB cable.

When connected, it appears as several devices in Linux. The first UART
(typically :file:`/dev/ttyUSB0`) has the HSS output, while the second
(typically :file:`/dev/ttyUSB1`) is used for the U-Boot and Linux console.
Connect with the following settings (see :ref:`connect-to-a-uart-console`):

* 115200 baud
* 8 data bits
* no parity
* 1 stop bit
* no flow control


USB
---

The USB OTG port (J16) can both be used a USB client or a USB host port.
Jumpers on the board switch between client and host mode (J15) and control the
provision of 5 Volt to the USB port (J17). The Ubuntu kernel supports the host
mode.


Limitations
-----------

* As of Linux kernel version 6.2, PCIe is not supported.
