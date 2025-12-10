=========================================
Server installer or pre-installed images?
=========================================

The classic Ubuntu Server images for :term:`SBCs <SBC>` can be roughly divided
into two types:

* Pre-installed images which are written directly to their final target media

* Installer images which are written to some installation media, booted on the
  target machine, and which then copy themselves to the final target media

Which you should choose depends on which board you have, and your particular
needs for the eventual system. In some cases, the choice is limited. Notably,
we currently only distribute pre-installed images for the Raspberry Pi series
of boards (the general arm64 live server images do not boot on the Raspberry
Pi).


Hardware requirements
=====================

The installer images necessarily require that you have *two* machines, one to
generate the installation media (commonly a PC), and the target SBC to run the
installer (assuming that the target SBC has no existing OS). It naturally
follows that you need two drives: the installer drive (commonly a USB stick or
SD card), and the final target media (often an :term:`SSD` or :term:`NVMe`
drive).

The pre-installed images *may* require two machines. Certain boards, notably
the later models of the Raspberry Pi, are capable of :doc:`flashing
pre-installed images to local media </tutorials/raspberry-pi-solo>` from their
bootloader. Regardless, the pre-installed images typically only require one
piece of target media (commonly an SD card), and thus have simpler initial
requirements.


Storage flexibility
===================

By their nature, the pre-installed images dictate the partition layout that the
eventual system will have. Beyond selecting the type and size of your target
media, you will have no option to customize the number or layout of partitions,
and the selection of file-systems is essentially fixed.

.. note::

    One notable exception here is that `btrfs`_ does include a tool to convert
    an `ext4`_ partition in-place. If you would prefer a btrfs root
    file-system, it is usually possible to convert a pre-installed image to
    btrfs before booting it for the first time.

Installer images, via the `subiquity`_ installer, have the option to configure
their storage including partition size, type, and file-system format selection.


Automated installation
======================

The installer images provide an auto-install facility that can be used to
automate the configuration of the new server, including user creation, package
installation, and storage layout.

The pre-installed images also provide an automated setup facility via the
`cloud-init`_ service. Barring storage layout, which is essentially fixed in
pre-installed images, this provides as much flexibility in the configuration of
users (:ref:`mod_cc_users_groups` module), package installation
(:ref:`mod_cc_package_update_upgrade_install` module), network configuration
(:ref:`network_config_v2`), and more.

.. note::

    Cloud-init does provide storage layout facilities via the
    :ref:`mod_cc_disk_setup` module, but this is intended for specific cloud
    services where such things can be configured by the container or VM.

In short, both image types offer considerable flexibility for automated
installation, so this shouldn't be a limiting factor in your selection.


.. _btrfs: https://en.wikipedia.org/wiki/Btrfs
.. _ext4: https://en.wikipedia.org/wiki/Ext4
