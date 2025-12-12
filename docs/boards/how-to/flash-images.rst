.. _flash-images-to-a-microsd-card:

Flash images to a microSD card
==============================

Several popular tools exist for flashing images to microSD cards, but this
guide uses only basic command line tools (specifically :manpage:`dd(1)`),
ensuring it can be followed even on console-only systems.

The most dangerous aspect of writing images in this manner is ensuring you are
overwriting the correct device. With root authority, ``dd`` will happily
overwrite any device you point it at, including the device holding your root
file-system! To avoid this, we will use :lp-pkg:`inotify-tools` to detect the
device node representing our microSD card.


Procedure
=========

#. Start with your target microSD card disconnected from your machine

#. Install the pre-requisite packages we're going to be using:

   .. code-block:: text

       sudo apt install inotify-tools atool wget

#. Download the image you wish to write. Here we'll be using the Ubuntu 24.04
   Server for Raspberry Pi image, but you should substitute your
   ``cdimage.ubuntu.com`` URL here:

   .. code-block:: text

       wget http://cdimage.ubuntu.com/releases/24.04/release/ubuntu-24.04-preinstalled-server-arm64+raspi.img.xz

#. If you wish to verify the download, also get the :file:`SHA256SUMS` file
   from the same path, and use it to verify the downloaded file:

   .. code-block:: text

       wget http://cdimage.ubuntu.com/releases/24.04/release/SHA256SUMS
       sha256sum --ignore-missing --check SHA256SUMS

#. Start :manpage:`inotifywait(1)`, watching for new device nodes under
   :file:`/dev`:

   .. code-block:: text

       inotifywait --event create --format "%w%f" /dev

#. Plug your microSD card into your machine. At this point, ``inotifywait``
   should exit, displaying the name of a new node under :file:`/dev`. For
   example:

   .. terminal::
       :user: ubuntu
       :host: ubuntu

       :input: inotifywait --event create --format "%w%f" /dev
       Setting up watches.
       Watches established.
       /dev/sda1

   It is likely that the node printed will be a device representing a
   *partition* of the microSD card, rather than the whole microSD card itself.
   In the example above, we see :file:`/dev/sda1`. This is the first partition
   of :file:`/dev/sda`, which is the device representing the entire microSD
   card.

#. We're now ready to write the image to the microSD card. If the image you
   downloaded is compressed (indicated by a :file:`.gz`, :file:`.bz2`,
   :file:`.xz`, or :file:`.zst` suffix on the filename) we cannot write it
   directly. We'll use the :manpage:`aunpack(1)` command to extract the image,
   then write the result to the microSD device with :manpage:`dd(1)`:

   .. code-block:: text

       aunpack ubuntu-24.04-preinstalled-server-arm64+raspi.img.xz
       sudo dd \
           if=ubuntu-24.04-preinstalled-server-arm64+raspi.img \
           of=/dev/sdX \
           bs=16M status=progress

   .. warning::

       Take care to specify the correct output device after the ``of=``
       parameter. Bear in mind this should be the device covering the entire
       microSD card, not just a partition.

   .. note::

       We are using :manpage:`sudo(8)` because ``dd`` requires root authority
       to write to the microSD card device. You will be prompted for your
       user's password before the write begins (assuming a typical sudo
       configuration).

#. Once :command:`dd` has completed, run :manpage:`sync(1)` to be
   reasonably certain that everything is flushed:

   .. code-block:: text

       sudo sync

#. You can remove the microSD card and insert it into your target board


Alternate device names
======================

In some cases, depending on the microSD interface in use, you may see output
like the following from :command:`inotifywait`:

.. terminal::
    :user: ubuntu
    :host: ubuntu

    :input: inotifywait --event create --format "%w%f" /dev
    Setting up watches.
    Watches established.
    /dev/mmcblk0p1

In this case, we are also seeing a device representing the first partition of
the microSD card, :file:`/dev/mmcblk0p1`. However, here we need to remove the
``p1`` suffix; the device representing the entire microSD card is
:file:`/dev/mmcblk0`.

This is often the case where the microSD interface is built into your machine
(e.g. the microSD card slot on a Raspberry Pi), or where the interface is
connected by something other than USB.


Avoiding decompression
======================

If you have limited disk space and do not wish to unpack the OS image, you can
decompress the image on the fly and pipe the result to :command:`dd`. For this,
use the :manpage:`acat(1)` command. For example:

.. code-block:: text

    acat ubuntu-24.04-preinstalled-server-arm64+raspi.img.xz | \
        sudo dd of=/dev/sdX bs=16M status=progress
