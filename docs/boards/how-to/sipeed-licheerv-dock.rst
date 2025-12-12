==========================================
Install Ubuntu on the Sipeed LicheeRV Dock
==========================================

The `LicheeRV board`_ can be used in different configurations. We currently
only supply an image for the LicheeRV with Dock.


Using the pre-installed server image
====================================

#. Download one of the supported images:

   .. ubuntu-images::
       :releases: noble-
       :suffix: +licheerv

#. Flash the pre-installed server image to a microSD card (see
   :ref:`flash-images-to-a-microsd-card`)

#. Insert the microSD card into the board

#. Connect to the serial :term:`UART` console (see :ref:`connect-to-a-uart-console`)

#. Power on the board

#. Wait for an output line confirming that `cloud-init`_ has finished running;
   this service is responsible for generating SSH keys, and creating the
   default user:

   .. code-block:: text

        [  291.932176] cloud-init[1282]: Cloud-init v. 22.3.4-0ubuntu1 finished at Thu, 20 Oct 2022 08:25:11 +0000. Datasource DataSourceNoCloud [seed=/var/lib/cloud/seed/nocloud-net][dsmode=net].  Up 291.79 seconds

#. Login with the user *ubuntu* and the default password *ubuntu*; you will be
   asked to choose a new password

Create a swap file
==================

The available RAM of the LicheeRV is quite small. It is advisable to create a
swap file.

.. code-block:: text

    sudo dd if=/dev/zero of=/swapfile bs=1M count=1024 conv=fsync
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile

Add the following line to /etc/fstab to make the swap memory available at boot:

.. code-block:: text

    /swapfile swap swap defaults 0 0

Wi-Fi configuration
===================

Install drivers
---------------

This step is only required for kernel versions earlier than 6.7. It is *not*
required for Ubuntu 24.04.

#. To enable Wi-Fi, the DKMS package :lp-pkg:`licheerv-rtl8723ds-dkms` is needed
   on lower kernel releases. If your LicheeRV is connected to the network via a
   USB Ethernet adapter, that is easy:

   .. code-block:: text

       sudo apt update
       sudo apt install licheerv-rtl8723ds-dkms

#. If your LicheeRV is not connected to the network, you must download the
   relevant ``licheerv-rtl8723ds-dkms_*.deb`` file from the
   :lp-pkg:`licheerv-rtl8723ds-dkms` Launchpad page and write it to the SD
   card.

#. Once you have booted the device the package can be installed with

   .. code-block:: text

       sudo dpkg -i licheerv-rtl8723ds-dkms_*.deb

   The installation process requires building a kernel module and takes at
   least an hour.

Configure Netplan
-----------------

#. Configure :term:`Netplan` to connect to your local network. Below is an
   example configuration:

   .. code-block:: yaml

       network:
           version: 2
           renderer: networkd
           wifis:
               wlan0:
                   dhcp4: yes
                   dhcp6: yes
                   access-points:
                       "YOUR_SSID":
                           password: "YOUR_PASSWORD"

   Store this configuration under a file matching the pattern
   :file:`/etc/netplan/{filename}.yaml`, for example
   :file:`/etc/netplan/wifi-config.yaml`. This file must be owned by, and only
   accessible to, the root user:

   .. code-block:: text

       sudo chown root:root /etc/netplan/wifi-config.yaml
       sudo chmod 600 /etc/netplan/wifi-config.yaml

#. Either reboot or execute the following commands to load the kernel module
   and apply the configuration:

   .. code-block:: text

       sudo modprobe 8723ds
       sudo netplan apply
       sudo systemctl restart systemd-networkd.service

   You should now be connected to your wireless network.


Cloud-init seed
===============

Sample files for a cloud-init seed are present on the FAT partition labeled
"CIDATA". See :ref:`configure-your-board-for-headless-use` for more information.


Limitations
===========

* Microphone does not work

* The following kernel modules are used for reading from the SD card:

  * mmc-block

  * sunxi-mmc

  They must either be built into the kernel or must be included in the initial
  RAM disk via /etc/initramfs-tools/modules or via a file in
  /usr/share/initramfs-tools/modules.d/.


.. _LicheeRV board: https://wiki.sipeed.com/hardware/en/lichee/RV/Dock.html
