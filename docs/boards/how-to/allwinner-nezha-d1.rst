.. _install-ubuntu-on-the-allwinner-nezha-d1:

Install Ubuntu on the Allwinner Nezha D1
========================================

The `Nezha D1`_ is a RISC-V based :term:`SBC`.


Using the pre-installed server image
------------------------------------

#. Download one of the supported images:

   .. ubuntu-images::
       :releases: jammy-
       :suffix: +nezha

#. Flash the pre-installed server image to an SD card (see
   :ref:`flash-images-to-a-microsd-card`)

#. Insert the SD card into the board

#. Optionally connect a USB UART adapter to the :term:`UART` on the
   :term:`GPIO` header (see `UART console`_ and :ref:`connect-to-a-uart-console`)

#. Power on the board

#. Be patient for HDMI output; early boot is only available on the UART console
   and the board's performance means that HDMI output may not appear for a few
   minutes

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

The :term:`UART` is located on its own three-pin header labeled "DEBUG",
adjacent to the main :term:`GPIO` header. The GND, RX, and TX pins are all
separately labeled, and are from the point of view of the board. Assuming the
typical coloring of USB to :term:`TTL` serial adapters the following
connections have to be made:

===== ==========
Board Adapter
===== ==========
GND   GND, black
RX    TX,  green
TX    RX,  white
===== ==========

Do not connect the red 3.3 V wire.

Connect with the following settings (see :ref:`connect-to-a-uart-console`):

* 115200 baud
* 8 data bits
* no parity
* 1 stop bit
* no flow control


Limitations
-----------

* Wifi/Bluetooth is not working: the wifi driver only available in the vendor
  kernel.

* The Ethernet MAC address changes on every boot; this "feature" is described
  `here <https://linux-sunxi.org/Ethernet>`_ along with the solutions to fix
  this in the section "Setting the MAC address".

* Shutdown fails.

* The following kernel modules are used for reading from the SD card:

  * mmc-block

  * sunxi-mmc

  They must either be built into the kernel or must be included in the initial
  RAM disk via /etc/initramfs-tools/modules or via a file in
  /usr/share/initramfs-tools/modules.d/.


.. _Nezha D1: https://d1.docs.aw-ol.com/en/d1_dev/
.. _cloud-init: https://cloudinit.readthedocs.io/
