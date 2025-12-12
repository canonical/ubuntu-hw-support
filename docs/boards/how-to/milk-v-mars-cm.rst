================================================
Install Ubuntu on the Milk-V Mars CM and CM Lite
================================================

The `Milk-V Mars CM`_ is a RISC-V based :term:`SBC`.

.. warning::

    The Milk-V Mars CM is not yet supported by official Ubuntu images.

NVMe support
============

For attaching an NVMe drive the Waveshare CM4-IO-BASE-A board can be used.

.. warning::

    On the Raspberry Pi Compute Module 5 IO Board NVMe drives do not work.

    The Milk-V Mars CM does not drive the line PCIE_PWR_EN to enable
    power to the m.2 connector.

Boot source selection
=====================

The Milk-V Mars CM board can boot firmware from SPI flash or UART.
The boot source is selected via the nRPIBOOT line. When connected to ground
booting form UART is selected.

On the Waveshare CM4-IO-BASE-A board a switch labeled "BOOT" is connected to
the nRPIBOOT line. Switch it to to "OFF" to boot from SPI flash and to "ON" to
boot from UART.

.. image:: /images/milk-v-mars-cm-boot-select.jpg
    :width: 20em
    :alt: Boot select

UART console
============

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
===========

* The on-board GPU is not supported

* PCIe support is incomplete: an NVMe drive can be used, but Wi-Fi cards and
  external GPUs don't work

* While the 3 USB 3.0 ports are working, the USB 2.0 port is not supported by
  the 6.8 kernel


.. _Milk-V Mars CM: https://milkv.io/mars-cm
