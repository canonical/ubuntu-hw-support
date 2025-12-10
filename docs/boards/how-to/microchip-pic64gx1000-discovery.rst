=========================================================
Install Ubuntu on the Microchip PIC64GX1000 Curiosity Kit
=========================================================

The first stage boot loader of the PIC64GX Curiosity Kit is called `Hart
Software Services`_ (HSS). Ubuntu requires version 2024.06 or later of this
software. The `HSS sources`_ are available from GitHub; build and programming
instructions can be found in the README of the linked repository.

.. _Hart Software Services: https://github.com/pic64gx/pic64gx-hart-software-services
.. _HSS sources: https://github.com/pic64gx/pic64gx-hart-software-services


Using the pre-installed server image
====================================

#. Download one of the supported images:

   .. ubuntu-images::
       :releases: noble-
       :suffixes: +pic64gx

#. Flash the pre-installed server image to a microSD card (see
   :doc:`/how-to/flash-images`)

#. Insert the microSD card into the board

#. Connect to the serial :term:`UART` console (see :doc:`/how-to/uart-console`)

#. Power on the board

#. Wait for an output line confirming that `cloud-init`_ has finished running;
   this service is responsible for generating SSH keys, and creating the
   default user:

   .. code-block:: text

       [   35.682018] cloud-init[909]: Cloud-init v. 24.1.3-0ubuntu3 finished at Tue, 23 Apr 2024 07:44:59 +0000. Datasource DataSourceNoCloud [seed=/var/lib/cloud/seed/nocloud-net][dsmode=net].  Up 35.65 seconds

#. Login with the user *ubuntu* and the default password *ubuntu*; you will be
   asked to choose a new password
