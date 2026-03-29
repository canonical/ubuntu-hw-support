.. SPDX-License-Identifier: CC-BY-SA-4.0

.. _kernel-test-cases:

Kernel test cases
=================

This list comprises some test cases that are useful to identify missing
kernel configuration.

Docker
------

Launch a container

.. code-block:: text

    sudo apt-get install docker.io
    sudo docker pull ubuntu:noble
    sudo docker run -ti ubuntu:noble
    exit
    sudo container prune -f
    sudo image rm ubuntu:noble

File systems
------------

EFI variable file system
''''''''''''''''''''''''

Check that the EFI variable file system is mounted at
`/sys/firmware/efi/efivars`.

.. code-block:: text

    mount | grep efivarfs

LVM and encrypted volumes support
''''''''''''''''''''''''''''''''''

Test creating and managing an encrypted LVM (Logical Volume Manager) setup.

.. code-block:: text

    truncate -s 1G disk
    sgdisk --new=1::0: --typecode=1:8300 --change-name=1:'root' disk
    sudo kpartx -a -v disk
    # Take note of the created loop device. Assuming loop0p1 below.
    # Replace it by the actual value.
    sudo cryptsetup luksFormat /dev/mapper/loop0p1
    sudo cryptsetup open /dev/mapper/loop0p1 encrypted-vg
    sudo pvcreate /dev/mapper/encrypted-vg
    sudo vgcreate vg_data /dev/mapper/encrypted-vg
    sudo lvcreate -n lv_data -L 512M vg_data
    sudo mkfs.ext4 /dev/vg_data/lv_data
    sudo mount /dev/vg_data/lv_data /mnt
    # Tearing down
    sudo umount /mnt
    sudo vgchange -a n vg_data
    sudo kpartx -d -v disk
    rm disk

Firewall
--------

Add Firewall rules

.. code-block:: text

    sudo ufw enable
    sudo ufw status
    sudo ufw allow from 192.168.0.0/16 proto udp port 1714:1764 comment 'KDE Connect'
    sudo ufw allow from 192.168.0.0/16 proto tcp port 1714:1764 comment 'KDE Connect'
    sudo ufw status numeric
    sudo ufw disable
    sudo ufw status

LXC, LXD
--------

Launch a container

.. code-block:: text

    sudo snap install --channel latest/stable lxd
    # If the snap is already installed
    # sudo snap refresh --channel latest/stable lxd
    lxc init --auto
    lxc launch ubuntu:24.04 noble1
    lxc shell noble1
    sudo poweroff
    lxc delete noble1

QEMU
----

Launch an EFI Shell in QEMU

.. code-block:: text

    sudo apt install qemu-system-riscv qemu-efi-riscv64
    cp /usr/share/qemu-efi-riscv64/RISCV_VIRT_VARS.fd .
    qemu-system-riscv64 -machine virt -m 4096 -nographic \
    -drive if=pflash,format=raw,unit=0,file=/usr/share/qemu-efi-riscv64/RISCV_VIRT_CODE.fd,readonly=on \
    -drive if=pflash,format=raw,unit=1,file=RISCV_VIRT_VARS.fd

Snapcraft
---------

Build a snap

.. code-block:: text

    sudo snap install --classic snapcraft
    sudo ufw disable
    lxd init
    git clone https://git.code.sf.net/p/sispmctl/git sispmctl
    cd sispmctl/
    snapcraft pack --use-lxd --verbose
    find . -name '*.snap'

Task accounting
---------------

Check that task accounting is usable:

.. code-block:: text

    sudo apt-get install iotop
    sudo iotop
