.. SPDX-License-Identifier: CC-BY-SA-4.0

.. _dragonwing_resources:

Resources
=========

Gadget
------

The Qualcomm Dragonwing gadget repository is:
https://github.com/canonical/qualcomm-dragonwing-gadget.git

Image definition
----------------

You can find image definition files into the gadget repository:
 * a server image: ``server-image.yaml``;
 * a desktop image: ``desktop-image.yaml``.

Building
--------

When calling ``ubuntu-image``, if you are targeting the UFS storage (default),
you need to add ``--sector-size=4096``. For eMMC or NVMe, no need to add
anything.
