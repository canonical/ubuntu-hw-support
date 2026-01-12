.. SPDX-License-Identifier: CC-BY-SA-4.0

.. _your-first-kernel-package:

Your first kernel package
=========================

To build your first kernel package, you need a git repository containing your
custom kernel tree. You can create that custom kernel from the upstream Linux
kernel, or you can start from a device optimized Ubuntu Linux kernel.
Examples of device optimized kernel:

* Generic: `linux <https://git.launchpad.net/~ubuntu-kernel/ubuntu/+source/linux/+git/noble/>`__
* Qualcomm Dragonwing: `linux-qcom <https://git.launchpad.net/~canonical-kernel/ubuntu/+source/linux-qcom/+git/noble/>`__

There is multiple ways to generate a kernel package from your custom kernel,
using debian packaging (if you start from an Ubuntu kernel, or if you setup
the debian packaging yourself) or without, using ``ukpack``.

.. toctree::
    :maxdepth: 2

    create_kernel_ukpack
    create_kernel_debian


Next steps
----------

You can new install or upload the package for further testing.
