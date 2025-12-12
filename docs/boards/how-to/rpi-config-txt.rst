========================================
Edit the Raspberry Pi boot configuration
========================================

The Raspberry Pi's boot configuration consists of a series of simple text
files. Depending on the model of Pi, the boot configuration either exists
entirely on the boot partition (the first :term:`FAT`-formatted partition on
your boot media), or has a portion based in a boot :term:`EEPROM` (on the Pi 4
and 5).

The format of the boot configuration is based on the classic MS-DOS
`INI-format`_, with the following variations:

* Section and property names are case *sensitive*

* Comments start with a hash-sign (``#``), not a semi-colon (``;``)

* Comments may only start at the beginning of a line, not trailing a property

* Ordering of sections and properties is relevant because…

* Duplicate names are permitted with later declarations overriding earlier ones

* Quoted values are not permitted

* Spaces are not permitted (or more precisely are not ignored) either side of
  the "=" in a ``key=value`` setting and hence, should not be used


Main configuration
==================

The main boot configuration, which applies to all models of Pi, lives in a file
called :file:`config.txt` placed on the boot partition of your boot media (the
first :term:`FAT`-formatted partition). Typically on Ubuntu, this is mounted
under the path :file:`/boot/firmware`. Files on the partition can only be
manipulated by the root user, hence to edit the boot configuration at run-time
you would use something like:

.. code-block:: text

    sudo vi /boot/firmware/config.txt

Below, we'll walk through the default boot configuration on Ubuntu Server for
Raspberry Pi. This will illustrate the format, and demonstrate a typical (if
minimal) configuration example:

.. literalinclude:: config.txt
    :language: ini
    :lines: 1-5

.. note::

    The lack of an equals sign in the ``initramfs`` line is intentional, and is
    a quirk of the ``config.txt`` format. It only applies to this property.

We start off with the ``[all]`` section. As the name implies, this applies to
all Pis, whatever model, or serial number. This first block simply defines:

arm_64bit
    Informs the bootloader we want to start the ARM cores in 64-bit mode
    (:spellexception:`ARMv8`)

kernel
    Specifies the location of the Linux kernel (here, :file:`vmlinuz` on the
    boot partition)

cmdline
    The file containing the command line to pass to the Linux kernel (the
    single line in :file:`cmdline.txt` on the boot partition). Be aware that
    the command line will also include parameters determined by the bootloader
    itself. If you query :file:`/proc/cmdline` at run-time you should typically
    find the content of :file:`cmdline.txt` is present as a suffix

initramfs
    The archive containing the initial ram-disk (the content of the initial
    root file-system), and where to load it in memory. Here, we tell the
    bootloader to place it immediately after the kernel

.. literalinclude:: config.txt
    :language: ini
    :lines: 7-12

We carry on, still in the ``[all]`` section, with a series of ``dtparam``
lines. These are device-tree parameters which customize the base device-tree.
In this case these enable the ALSA, :term:`I²C`, and :term:`SPI` interfaces.

.. literalinclude:: config.txt
    :language: ini
    :lines: 23-26

Next are a series of options controlling video output. The two important
options are:

:spellexception:`dtoverlay`
    This loads the "full" KMS device-tree overlay, leaving the Linux kernel in
    control of the display hardware, rather than the boot firmware

:spellexception:`disable_fw_kms_setup`
    This disables the firmware's detection of the monitor (via :term:`EDID`),
    leaving the Linux kernel to handle detection it

.. literalinclude:: config.txt
    :language: ini
    :lines: 28-29

This section simply enables the serial :term:`UART` on :term:`GPIO` 14 and 15.
This is only included on the Ubuntu Server images because headless setups
(where the serial console is useful for debugging) are relatively common with
this image. See :ref:`connect-to-a-uart-console` for further information.

The Ubuntu Desktop images do *not* enable the serial UART by default as we
assume a keyboard and monitor will be present there.

.. literalinclude:: config.txt
    :language: ini
    :lines: 37-60

Lastly, we have a series of properties and sections that simply apply
workarounds for various issues. Mostly, these involve loading device-tree
overlays or applying parameters to them. It is noteworthy, however, that there
are several sections named after models of Raspberry Pi. This limits their
effects only to matching models.

The final line specifies an empty ``[all]`` section. If scripts or applications
blindly append further options to this file, this ensures they don't
inadvertently wind up only applying to the last filter (for the Compute Module
4, in this case).

Conditional sections
--------------------

There are several other conditional section types that can be used. Refer to
the Raspberry Pi documentation of the `config.txt format`_ for the definitive
list, but a brief set of examples follows:

[all], [none]
    Special sections that apply to all boards and no boards respectively. While
    seemingly pointless, the ``[none]`` section is handy for effectively
    "commenting out" multiple properties

[pi0], [pi2], [pi4], [cm4], etc.
    Sections that limit their effect to the respective model of Raspberry Pi.
    Please note there are some subtleties here. For instance, the ``[pi0]``
    section applies to the Pi Zero, Zero W, *and* Zero 2W. Likewise, the
    ``[pi4]`` section applies to the 4B, 400, and Compute Modules 4 and 4S.
    Consult the linked documentation above for the specific overlaps

[EDID=Foo-Bar Monitor]
    Sections that limit their effect to Raspberry Pis attached to a monitor
    with the specified :term:`EDID` name (see `the EDID filter`_ section for
    information on determining your monitor's EDID name). Useful if you have
    workarounds that apply only to a specific monitor

:spellexception:`[0x12345678]`
    Sections that apply only to a board with the specified serial number. This
    can be found at the bottom of :file:`/proc/cpuinfo`. Only use the *last*
    eight hexadecimal digits of the listed serial number

[gpio4=1]
    Sections that apply only when the specified :term:`GPIO` is found in the
    specified state (1 for high, 0 for low) at boot time.


EEPROM configuration
====================

On Raspberry Pi models 4 and later (including the Compute Module 4, the 400,
and the Pi 5), there is an additional boot configuration file stored in the
boot :term:`EEPROM`. This is parsed and acted upon *prior* to
:file:`config.txt` and in many cases determines the boot media on which to find
:file:`config.txt`.

As this configuration is stored in EEPROM, it is not something that ships on
Ubuntu images, and will be specific to your board. The default configuration
has also evolved over time, so that later models of the Pi have a different
default EEPROM configuration to earlier models.

Furthermore, as it only exists in EEPROM, it is not a file accessible via a
regular file-system. Use the ``rpi-eeprom-config`` utility from the
:lp-pkg:`rpi-eeprom` package to view the configuration:

.. code-block:: text

    sudo rpi-eeprom-config

This utility can also be used to edit the configuration:

.. code-block:: text

    sudo rpi-eeprom-config --edit

This will launch your configured text editor (:manpage:`nano(1)` or
:manpage:`vim(1)` by default) with an ephemeral copy of the EEPROM
configuration. Upon successful exit, the utility will write the (potentially
modified) copy back to the EEPROM.

.. note::

    In some cases (older versions of ``rpi-eeprom``, or where the
    :manpage:`flashrom(8)` utility is not present), a reboot is required to
    write the configuration to the EEPROM.

The format of the EEPROM configuration is the same as for :file:`config.txt`
above, but the properties are different.


Boot order
----------

By far the most important property for most people, will be `BOOT_ORDER`_. This
dictates which boot sources the Pi will attempt to read, and in what order. The
value is a ``0x`` prefixed hexadecimal number. Read **right to left**, each
digit specifies a boot source to attempt. The possible boot sources, at the
time of writing, are:

+-------+-------------+----------------------------------------------------+
| Digit | Source      | Description                                        |
+=======+=============+====================================================+
| 0     | Card Detect | Deprecated                                         |
+-------+-------------+----------------------------------------------------+
| 1     | SD Card     | Attempt to boot from the SD card slot, or the      |
|       |             | :term:`eMMC` on Compute Modules                    |
+-------+-------------+----------------------------------------------------+
| 2     | TFTP        | Attempt :term:`DHCP` over ethernet, then           |
|       |             | :term:`TFTP` to retrieve further boot assets       |
+-------+-------------+----------------------------------------------------+
| 3     | RPI BOOT    | Attempt USB host boot, where the Pi acts as a      |
|       |             | device to a USB host. Typically used for flashing  |
|       |             | :term:`eMMC` in Compute Modules                    |
+-------+-------------+----------------------------------------------------+
| 4     | USB MSD     | Attempt USB boot from an attached mass-storage     |
|       |             | device (MSD); this is the typical mode used to     |
|       |             | boot from attached SSDs or hard-drives             |
+-------+-------------+----------------------------------------------------+
| 5     | USB MSD     | Attempt USB boot from a mass-storage device        |
|       | (Type-C)    | attached to the USB-C socket; Pi 4 and CM4 only    |
+-------+-------------+----------------------------------------------------+
| 6     | NVMe        | Attempt boot from an :term:`NVMe` device connected |
|       |             | to the PCIe interface (CM4 and Pi 5 only)          |
+-------+-------------+----------------------------------------------------+
| 7     | HTTP        | Attempt :term:`DHCP` over ethernet, then HTTP to   |
|       |             | retrieve further boot assets (this mode is used to |
|       |             | launch rpi-imager from the bootloader)             |
+-------+-------------+----------------------------------------------------+
| E     | Stop        | Terminate further boot attempts and display the    |
|       |             | error pattern; power-cycle is required to continue |
+-------+-------------+----------------------------------------------------+
| F     | Loop        | Restart the sequence from the initial boot mode    |
+-------+-------------+----------------------------------------------------+

Some typical values are given below:

:spellexception:`0xF41`
    Try SD (1), followed by USB MSD (4), then repeat (F). This is the default
    boot order on modern Raspberry Pis

:spellexception:`0xF14`
    Try USB MSD (4), followed by SD (1), then repeat (F)

:spellexception:`0xF461`
    Try SD (1), followed by :term:`NVMe` (6), USB MSD (4), then repeat (F)

:spellexception:`0xF21`
    Try SD (1), followed by :term:`TFTP` network, then repeat

.. warning::

    We would strongly recommend leaving the SD card (1) *somewhere* in the boot
    order, to ensure that in the event of invalid configuration you always have
    a simple means of booting a rescue image to reconfigure things.

    For example, if you configure ``BOOT_ORDER=0xF2`` (network, then retry),
    but fail to correctly configure a TFTP network boot for your Pi, it is more
    difficult to recover (though usually still possible with the official
    recovery image).


.. _INI-format: https://en.wikipedia.org/wiki/INI_file
.. _config.txt format: https://www.raspberrypi.com/documentation/computers/config_txt.html
.. _the EDID filter: https://www.raspberrypi.com/documentation/computers/config_txt.html#the-edid-filter
.. _BOOT_ORDER: https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#BOOT_ORDER
