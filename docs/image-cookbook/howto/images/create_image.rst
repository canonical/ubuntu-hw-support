.. SPDX-License-Identifier: CC-BY-SA-4.0

.. _create-customized-image-with-ubuntu-image:

Create customized image with ubuntu-image
=========================================

For creating an image with ubuntu-image we need

* a git repository with the gadget definition
* a image definition file image-definition.yaml which may be located in the same
  git repository

Please, execute the :ref:`your-first-ubuntu-image` tutorial.
This provides a sample project.

Install dependencies
--------------------

.. prompt:: text $ auto

    $ sudo apt-get update
    $ sudo apt-get install git snapd qemu-user-static ubuntu-dev-tools
    $ sudo systemctl restart systemd-binfmt

Install ubuntu-image
--------------------

ubuntu-image is provided as a snap and can be installed with

.. prompt:: bash $ auto

    $ sudo snap install --classic ubuntu-image

Create a gadget definition
--------------------------

At a minimum the Gadget definition consists of

* the definition file :file:`gadget.yaml`
* a :file:`Makefile` where the default target
  * copies :file:`gadget.yaml` to directory ``meta/``
  * copies all installation files to directory ``install/``

The YAML file format is described in https://yaml.org/spec/.

An overview of the available fields in :file:`gadget.yaml` is provided in
:ref:`gadget-yaml-fields`.

The gadget definition is provided as a Yaml file :file:`meta/gadget.yaml`.
:file:`gadget.yaml` describes

* partitions of the image
* files installed on partitions other than the root partition

