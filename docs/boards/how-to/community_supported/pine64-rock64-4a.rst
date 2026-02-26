.. _install-ubuntu-on-the-pine64-rock64-4a:

Install Ubuntu on the Pine64 Rock64 4A
=======================================

For the instructions below a Pine64 Rock64 4A board with an NVMe drive
is presumed.
A HDMI monitor, a USB keyboard, and a USB mouse have to be connected.

.. warning::

     On the Pine64 Rock64 4A U-Boot can either be located on SPI flash
     or on the SD-card.
     Here it is assumed that U-Boot has been erased from the SPI flash
     and the device boots via U-Boot from the SD card.

Provide installer image
-----------------------

Download the ARM64 installer image from the webpage. In the following the
image is called `resolute-desktop-arm64.iso`. Replace that filename with
your release.

To copy the content of to the SD-card we need to make its partitions accessible
as loop devices.

.. code-block:: text

     sudo apt-get update
     sudo apt-get install kpartx
     sudo kpartx -a -v resolute-desktop-arm64.iso

You will see an output like

.. code-block:: text

    add map loop15p1 (252:1): 0 7889312 linear 7:15 64
    add map loop15p2 (252:2): 0 15424 linear 7:15 7889376

The number of the loop device may be different on you computer.
Adjust the commands below accordingly.

Prepare the SD-card
-------------------

Insert the SD-card in to an SD-card reader.

.. warning::

    The device name of SD-card will depend on your system. In the instructions
    below you will have to replace /dev/sdX by the actual device.

.. code-block:: text

    sudo apt-get update
    sudo apt-get install gdisk rsync ubuntu-dev-tools
    sudo sgdisk /dev/sdX \
    --zap-all \
    --set-alignment=64 \
    --new=13:64:32767 \
    --change-name=13:uboot \
    --new=15:32767:131071 \
    --typecode=15:ef00 \
    --change-name=15:ESP \
    --new=1:131072:
    pull-lp-debs -a arm64 u-boot resolute
    # Replace `resolute` by the current release
    dpkg -x u-boot-rockchip_2025.10-0ubuntu2_arm64.deb u-boot-rockchip
    sudo dd if=u-boot-rockchip/usr/lib/u-boot/rock-pi-4-rk3399/u-boot-rockchip.bin \
      of=/dev/sdX13 bs=1M conv=fsync
    sudo dd if=resolute-desktop-arm64.iso of=/dev/sdX1 bs=1M conv=fsync
    sudo dd if=/dev/mapper/loop15p2 of=/dev/sdX15 bs=1M conv=fsync
    # Replace `loop15p2` by the loop device with the ESP partition of the
    # installer.
    pull-lp-debs -a arm64 linux
    dpkg -x linux-modules-?.*-generic_*_arm64.deb linux-modules
    sudo mount /dev/sdX15 /mnt
    sudo rsync -avP linux-modules/lib/firmware/*-generic/device-tree/* /mnt/dtb/
    sync
    sudo umount /mnt

Run installer
-------------

Insert the SD-card into the Pine64 Rock64 4A and power on the device.

The baudrate of the serial console is 1500000. As of U-Boot 2025.10 output is
only shown on the video console and not on the serial console.

On reboot hit the enter key repeatedly until you reach the U-Boot prompt
`=>`.

Fix the boot order
------------------

Power off the device. On you computer remove partition 1 and 15 from the SD-card.

.. code-block::

    sudo sgdisk /dev/sdX \
    --zap-all \
    --set-alignment=64 \
    --new=13:64:32767 \
    --change-name=13:uboot

Reinsert the SD-card and power on the device.
