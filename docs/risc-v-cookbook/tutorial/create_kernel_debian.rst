.. SPDX-License-Identifier: CC-BY-SA-4.0

Debian packaging
================

This tutorial guides you through packaging your own custom Linux kernel using
Debian packaging.

For this tutorial, we use the Qualcomm optimized kernel to simulate a custom
kernel.

Install dependencies
--------------------

Add ``deb-src`` to the ``Types:`` line in the
``/etc/apt/sources.list.d/ubuntu.sources`` file. This will allow to get access
to source packages, and fetch the dependency list for building Linux.

Then ensure the required packages are installed:

.. prompt:: none $ auto

    $ sudo apt-get update
    $ sudo build-dep linux
    $ sudo apt install -y llvm libncurses-dev dwarves libtraceevent-dev libracefs-dev

If you are cross-compiling:

.. prompt:: none $ auto

    $ sudo apt install gcc-aarch64-linux-gnu
    $ export ARCH=arm64
    $ export $(dpkg-architecture -aarm64)
    $ export CROSS_COMPILE=aarch64-linux-gnu-

Clone the kernel repository
---------------------------

.. prompt:: none $ auto

    $ git clone -b master-next --depth 1 \
      https://git.launchpad.net/~canonical-kernel/ubuntu/+source/linux-qcom/+git/noble

Build the kernel package
------------------------

.. prompt:: none $ auto

    $ cd <kernel source working directory>
    $ fakeroot debian/rules clean
    $ fakeroot debian/rules binary

If the build is successful, several .deb binary package files will be
produced in the parent directory of the working directory.
