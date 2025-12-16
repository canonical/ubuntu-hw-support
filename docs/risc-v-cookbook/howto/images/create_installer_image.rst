.. SPDX-License-Identifier: CC-BY-SA-4.0

.. _creating-customized-live-installer-images:

Creating customized installer images
====================================

Ubuntu provides both pre-installed images as well as installer images.

The build process of installer images involves multiple steps:

* Ubuntu package livecd-rootfs is used to create multiple file system layers
  as squashfs file systems, e.g.

  - livecd.ubuntu-server.ubuntu-server-minimal.squashfs
  - livecd.ubuntu-server.ubuntu-server-minimal.ubuntu-server.installer.generic.squashfs
  - livecd.ubuntu-server.ubuntu-server-minimal.ubuntu-server.installer.squashfs
  - livecd.ubuntu-server.ubuntu-server-minimal.ubuntu-server.squashfs

* `ubuntu-cdimage <https://git.launchpad.net/ubuntu-cdimage>` takes these
  squashfs file systems and packages them as an ISO image.

Patching these tools to create an ISO image is tedious and error prone.

It is much easier to take an existing live-installer image and to patch it to
your needs.

Installing livefs-editor
------------------------

Livefs-editor can be installed in a Python virtual environment with:

.. code-block:: text

    git clone https://github.com/mwhudson/livefs-editor.git
    cd livefs-editor/
    python3 -m venv myvenv
    . myvenv/bin/activate
    pip install .

For a system-wide installation you can use:

.. code-block:: text

    sudo python3 -m pip install --break-system-packages .

Using livefs-editor
-------------------

The livefs-editor
`README <https://github.com/mwhudson/livefs-editor/blob/main/README.md#actions>`
gives an overview of the different actions that the tool can take.

Best practice is to use a yaml file describing all desired actions and
to invoke livefs-editor with it:

.. code-block:: text

    sudo livefs-editor old.iso new.iso --action-yaml update.yaml

Here is an example yaml file:

.. code-block:: yaml

    ---
    - name: cp
      source: /home/ubuntu/project/ppa.sources
      dest: $LAYERS[0]/etc/apt/sources.list.d/vendor-ppa.sources
    - name: install-packages
      packages:
      - vendor-package1
      - vendor-package2
    - name: rm
      path: new/iso/pool/main/e/efivar/libefiboot1_37-6ubuntu2_riscv64.deb
    - name: replace-kernel
      flavor: vendorflavor

The ``cp`` action sets up the vendor PPA to allow installing vendor specific
packages from there.

These are installed by the ``install-packages`` action.

With the ``rm`` a package is removed from the ISO.

The ``replace-kernel`` action replaces the generic Linux kernel by a vendor
kernel package linux-image-vendorflavor located in the vendor PPA with all
its dependencies.
