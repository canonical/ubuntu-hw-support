.. SPDX-License-Identifier: CC-BY-SA-4.0

.. _risc-v_resources:

Resources
=========

Gadget
------

The RISC-V gadget repository is: https://github.com/canonical/risc-v-gadget.git

Image definition
----------------

You can find an image definition file (``image-definition.yaml``) into the
gadget repository.

Testing
-------

You can test RISC-V image on QEMU.

.. prompt:: text $ auto

    $ sudo apt-get install opensbi qemu-system-riscv64 u-boot-qemu

.. prompt:: text $ auto

    $ qemu-system-riscv64 \
      -machine virt -nographic -m 2048 -smp 4 \
      -bios /usr/lib/riscv64-linux-gnu/opensbi/generic/fw_jump.bin \
      -kernel /usr/lib/u-boot/qemu-riscv64_smode/uboot.elf \
      -device virtio-net-device,netdev=eth0 -netdev user,id=eth0 \
      -device virtio-rng-pci \
      -drive file=<image>,format=raw,if=virtio
