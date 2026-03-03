.. _install-ubuntu-on-qemu-arm64:

Install Ubuntu on QEMU (ARM64)
===============================

In the description below we use QEMU invoked from the command line interface
to launch an ARM 64-bit virtual machine. You could instead use a GUI based tool
like :lp-pkg:`Virt-Manager <virt-manager>`.


Prerequisites
-------------

To boot an ARM64 bit virtual machine you will need the following packages

* :lp-pkg:`qemu-system-arm <qemu>` -- :term:`QEMU` is used to
  emulate the ARM machine.

* :lp-pkg:`qemu-efi-aarch64 <edk2>` -- EDK II is the reference implementation
  of the :term:`UEFI` :term:`API`.

The packages can be installed with the following commands:

.. code-block:: text

    sudo apt update
    sudo apt install qemu-system-arm qemu-efi-aarch64


Installing Ubuntu
-----------------

#. Download one of the supported images:

   **Server without desktop**

   .. ubuntu-images::
       :releases: noble-
       :image-types: live-server
       :archs: arm64
       :matches: (live-server-arm64\.iso)

   **Desktop**

   .. ubuntu-images::
       :releases: questing-
       :image-types: desktop
       :archs: arm64
       :matches: (desktop-arm64\.iso)

#. Create the disk image onto which you will install Ubuntu; 16 GiB should be
   enough:

   .. code-block:: text

       truncate -s 16G disk

#. Start the installer with:

   .. code-block:: text

       qemu-system-aarch64 \
         -M virt,gic-version=3 \
         -m 4096 \
         -smp 4 \
         -cpu max \
         -serial mon:stdio \
         -device virtio-gpu-pci \
         -device qemu-xhci \
         -device usb-kbd \
         -device usb-tablet \
         -drive if=pflash,format=raw,unit=0,file=/usr/share/AAVMF/AAVMF_CODE.fd,readonly=on \
         -drive if=pflash,format=raw,unit=1,file=AAVMF_VARS.fd \
         -drive if=none,file=disk,format=raw,id=VIRTIO1 \
         -device virtio-blk,drive=VIRTIO1,bootindex=1 \
         -drive if=none,file=ubuntu-live-server-arm64.iso,format=raw,readonly=on,id=VIRTIO2 \
         -device virtio-blk,drive=VIRTIO2,bootindex=2 \
         -net nic,model=virtio \
         -net user

   Replace the ISO file name according to your download.

   Some important options to use are:

   \-accel kvm
      Add this option to use KVM acceleration if your host is an ARM system
      that supports it.

   \-cpu
      controls the emulated CPU

   \-M
      selects the platform emulated by QEMU. Version 3 or higher of the
      ARM Generic Interrupt Controller (GIC) is needed if you want to
      emulate more than eight CPUs.

   \-m
      specifies the memory size

   \-smp
      specifies the number of CPUs

#. Follow the installation steps in
   `Ubuntu Server installation tutorial
   <https://ubuntu.com/tutorials/install-ubuntu-server>`_ or the
   `Ubuntu Desktop installation tutorial
   <https://ubuntu.com/tutorials/install-ubuntu-desktop>`_.


Running Ubuntu
~~~~~~~~~~~~~~

When running the installed image you won't need to reference the ISO anymore:

.. code-block:: text

    qemu-system-aarch64 \
      -M virt,gic-version=3 \
      -m 4096 \
      -smp 4 \
      -cpu max \
      -serial mon:stdio \
      -device virtio-gpu-pci \
      -device qemu-xhci \
      -device usb-kbd \
      -device usb-tablet \
      -drive if=pflash,format=raw,unit=0,file=/usr/share/AAVMF/AAVMF_CODE.fd,readonly=on \
      -drive if=pflash,format=raw,unit=1,file=AAVMF_VARS.fd \
      -drive if=none,file=disk,format=raw,id=VIRTIO1 \
      -device virtio-blk,drive=VIRTIO1,bootindex=1 \
      -net nic,model=virtio \
      -net user
