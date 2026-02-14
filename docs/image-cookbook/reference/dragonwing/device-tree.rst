Device tree file
================

Qualcomm Dragonwing platforms use a combined device-tree (CDT),
``combined-dtb.dtb`` file. This file concatenates several DTB files, one of
which will be chosen by the bootloader based on the CDT settings.

To customize how the ``combined-dtb.dtb`` is prepared, please modify the file:
``arch/arm64/boot/dts/qcom/Makefile``.

The compiled ``combined-dtb.dtb can`` be found in linux-modules package, at the
path ``/lib/firmware/<kernel version>/device-tree/qcom/combined-dtb.dtb``.

After retrieving the latest ``combined-dtb.dtb`` file, you can update the file
inside the ``dtb.bin`` file (provided next to Ubnutu image). The ``dtb.bin``
is a FAT filesystem image and so you can use the ``mount`` command to replace
the content inside:

.. prompt:: none $ auto

    $ mkdir mnt
    $ sudo mount dtb.bin mnt
    $ cp <path to new combined-dtb.dtb> mnt/
    $ sudo umount mnt

After that, you can flash the new DTB to your device with the same flashing
instruction.
