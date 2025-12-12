.. _a-b-boot-on-ubuntu-for-raspberry-pi:

A/B boot on Ubuntu for Raspberry Pi
===================================

From Ubuntu questing (25.10) onwards, the boot sequence for Ubuntu on the
Raspberry Pi includes a fallback mechanism (commonly known as an A/B boot
mechanism). This is implemented by a new command, ``piboot-try`` and
two "one-shot" systemd services, ``piboot-try-reboot.service`` and
``piboot-try-validate.service``. This mechanism is enabled on *all* Pi models
supported by Ubuntu.

The bootloader assets themselves are *not* protected by this mechanism [#why]_.
However, other boot assets, including the base device-trees, device-tree
overlays, the Linux kernel, and the initramfs are covered.


File layout
-----------

The layout of files on the boot partition is (as of 25.10) as follows:

Bootloader assets
    The bootloader itself is stored in EEPROM on the Pi 5, partially in EEPROM
    on the Pi 4, and in the following files on all earlier models, placed in
    the root of the boot partition, :file:`/boot/firmware`:

    * :file:`autoboot.txt` -- The early bootloader configuration file

    * :file:`bootcode.bin` -- The second stage bootloader (on models prior to
      the Pi 4)

    * :file:`start*.elf` and :file:`fixup*.dat` -- The third stage bootloader
      (on models prior to the Pi 5)

    * :file:`config.txt` -- The main bootloader configuration file

Cloud-init configuration
    These are *not* boot assets, but are used by `cloud-init`_ to configure the
    machine on the first boot only. They are also present in the root of the
    boot partition (:file:`/boot/firmware`) and are listed here for
    completeness only:

    * :file:`meta-data` -- The cloud-init meta-configuration, describing where
      to find the other configuration files (locally by default)

    * :file:`network-config` -- The network configuration for cloud-init

    * :file:`user-data` -- The main cloud-init configuration file, specifying
      users to create, packages to install, files to write, etc.

Boot assets
    These consist of the Linux kernel, the initramfs, the base device-tree,
    and any device-tree overlays:

    * :file:`vmlinuz` -- The Linux kernel

    * :file:`cmdline.txt` -- The Linux kernel command line

    * :file:`initrd.img` -- The initramfs

    * :file:`*.dtb` -- The base device-tree(s)

    * :file:`overlays/*.dtbo` -- All device-tree overlays

In older releases of Ubuntu, the boot assets were all placed in the root of the
boot partition, :file:`/boot/firmware`, alongside the bootloader assets and
cloud-init configuration. From questing (25.10) onwards, copies of the boot
assets (and *only* the boot assets) will be placed in the following folders on
the boot partition:

:file:`old/` folder
    Holds older copies of "known good" boot assets. This folder is not always
    present; it is removed to make space when new boot assets are written.

:file:`current/` folder
    This folder is *always* present and at runtime *always* holds the currently
    booted assets.

:file:`new/` folder
    Holds newly written "untested" boot assets, or boot assets that were tested
    but failed to boot properly. This is replaced when new boot assets are
    written, or is renamed to :file:`current/` after assets are tested and
    validated.


Typical operation
-----------------

In each of the folders on the boot partition, a :file:`state` file will be
present which records the state of the boot assets in that folder. This file
may contain one of the following strings:

good
    The boot assets have been booted successfully and validated. The
    :file:`current/state` and optional :file:`old/state` files should *always*
    contain "good"

unknown
    The boot assets have not yet been booted. The :file:`new/state` file will
    contain this immediately after a run of ``flash-kernel``.

trying
    The boot assets are currently being tested. The :file:`new/state` file will
    contain this only when the machine is booting in the "tryboot" mode (or
    just immediately prior to rebooting into this mode).

bad
    The boot assets have been tested, but failed. The :file:`new/state` file
    will contain this if a "tryboot" attempt fell back to the assets in
    :file:`current/`.

Normally, the system will boot using the assets from the :file:`current/`
folder. The other folders may or may not exist, depending on the state of the
system, but will never be used to boot under normal conditions.


Testing new boot assets
-----------------------

Installation of new boot assets is handled, as ever, by the
``flash-kernel`` command. This is automatically called in response to
several changes, including installation of a new kernel version, re-building of
the initramfs, or upgrades of flash-kernel itself. It may also be called
manually to write new copies of the boot assets to the boot partition.

From questing (25.10), rather than those files being written to the root of the
boot partition, the following will occur when ``flash-kernel`` is executed:

1. If present, the :file:`old/` folder will be removed [#space]_

2. If present, the :file:`new/` folder will be removed [#when-failed]_

3. The :file:`new/` folder will be created

4. New boot assets will copied into the :file:`new/` folder

5. The :file:`new/state` file will be created, containing "unknown"

.. note::

    Note that ``flash-kernel`` never touches the files under :file:`current/`.
    If ``flash-kernel`` is executed again before rebooting, the new boot assets
    will simply replace those currently under :file:`new/`.

With the new boot assets in place, we come to the role of the systemd services.

When the system is next booted (or rebooted), the boot will initially use the
assets under :file:`current/`. However, at some point during this boot, the
``piboot-try-reboot`` service will note that boot assets exist under
:file:`new/` with the state "unknown". It will immediately update that status
to "trying", and reboot the machine into the "tryboot" mode. In this mode, the
:file:`config.txt` file will direct the bootloader to read assets from the
:file:`new/` folder instead.

Note that the "tryboot" status is ephemeral [#reset-on-read]_. Any failure at
this point will result in the next boot (whether warm or cold) using the assets
from :file:`current/` instead.

However, if the "tryboot" boot succeeds, and reaches the point where the
``piboot-try-validate`` service runs [#later-than-reboot]_ then the boot state
will be validated (see `Boot asset validation`_ below). If successful, the
service will perform the following:

1. "good" is written to the :file:`new/state` file

2. The :file:`current/` folder is renamed :file:`old/`

3. The :file:`new/` folder is renamed :file:`current/`
   [#current-always-exists]_


Boot asset failure
------------------

Failure may occur in several places, and as noted previously not all assets are
covered. This section will not consider bootloader asset failure as, at least
on later models, they are only in EEPROM and not affected by ``flash-kernel``.

Pre-kernel failure
    In the event that the Linux kernel fails to start, the Pi's hardware
    watchdog (enabled by ``dtparam=watchdog=on`` in :file:`config.txt`) will
    hard-reset the Pi, and the "known good" assets under :file:`current/` will
    be used instead.

Kernel failure
    From questing (25.10), the kernel's command line (in
    :file:`current/cmdline.txt`) contains ``panic=10``. In the event of a fatal
    error in the kernel, it *should* pause for 10 seconds, then reboot the
    machine, causing a fallback to the "known good" assets under
    :file:`current/`. If the fatal error prevents even the panic mechanism from
    working, the hardware watchdog (above) will reboot the machine.

    .. note::

        Note that on the Pi 5 there is a failure mode in which a corrupted
        kernel is not recognized as "Pi 5 compatible". In this case,
        unfortunately, the bootloader simply freezes. Still, a power-cycle will
        reliably fallback in this scenario.

Initramfs failure
    In the event that the initramfs fails, the default behavior in prior
    releases is to drop to a busybox shell for remediation. In questing (25.10)
    onwards, the aforementioned ``panic=10`` setting prevents this and causes a
    reset instead, falling back to the "known good" assets [#enable-busybox]_.

During fallback, the ``piboot-try-reboot`` service will note that the machine
is no longer in the "tryboot" state, and yet the :file:`new/state` file
contains "trying". As this indicates boot failure, the service will write "bad"
to :file:`new/state`.


Boot asset validation
---------------------

The ``piboot-try-validate`` service relies on the
:file:`/etc/flash-kernel/piboot-validate` script to determine whether boot
assets have booted "successfully". By default, this script simply runs "true"
(which returns 0, meaning "success"). This may seem overly simplistic, but
there is little else that can be done *by default* here, and even this actually
represents quite a bit of functionality. For the ``piboot-try-validate``
service to be running:

* The kernel must have started successfully

* The initramfs must have run successfully, mounting the "real" rootfs

* Systemd must have reached the ``multi-user.target`` where the service is
  installed

If you have requirements beyond this, you are encouraged to add additional
tests to this script. Some examples are provided (as comments) within it.
However, exercise caution when adding such tests. In particular, be aware that
a failing test will cause reversion to older boot assets, potentially
containing unpatched issues.

It is reasonable, especially for remote headless machines, to add a test
ensuring that expected network interfaces are present (for example, that an
"eth0" interface exists). For example:

.. code-block:: sh

    #!/bin/sh

    set -e

    # Test whether the eth0 device is present
    ip link show eth0 >/dev/null

It is tempting to add a further test ensuring such an interface is operating
successfully; continuing the example, that "eth0" is up, and has an IPv4
address:

.. code-block:: sh

    #!/bin/sh

    set -e

    # Test whether the eth0 device is present
    ip link show eth0 >/dev/null
    # Test if eth0 has an IPv4 address
    ip --json addr show eth0 | \
        jq -e '[.[0].addr_info[].family == "inet"] | any' >/dev/null

However, be aware that, as the test now depends on something external to the
machine (e.g. a DHCP server), anything which could prevent the machine
receiving an IP address can also cause it to fail validation of a kernel update
(potentially containing security fixes). These concerns do not apply
universally, though. Such a test is safer on an interface with a statically
configured address, for instance.

If you have a headless, remote machine, you should consider tailoring the
validation script to suit your needs.


Advanced operation
------------------

If required, the double-boot can be avoided by using the
``piboot-try`` command. This has a number of options that can be used
to check the status of boot assets, to force a reboot into the "tryboot"
mode immediately, to restore prior state, and reset failed assets.


Avoiding the double-boot
~~~~~~~~~~~~~~~~~~~~~~~~

The ``--test`` option returns zero (indicating success) if new, untested boot
assets are present, and non-zero otherwise. In other words, this can be used be
determine the next boot will be a double boot.

When untested boot assets are present, the ``--reboot`` option will immediately
force a reboot into the tryboot mode, testing the new assets and avoiding the
double boot (if untested boot assets are not present, it will do nothing).

If you have a system that is ready to reboot, you could combine these options
in a script to ensure the reboot will avoid the double boot like so:

.. code-block:: sh

    #!/bin/sh

    if piboot-try --test; then
        piboot-try --reboot
    else
        reboot
    fi


Testing new boot configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The boot configuration is largely controlled through two files:
:file:`config.txt` in the root of the boot partition, and :file:`cmdline.txt`
(which exists under the "old", "current", and "new" folders).

If you wish to test a new boot configuration in :file:`config.txt`:

1. Run ``sudo flash-kernel`` to set up new boot assets

2. Place the new (or changed) configuration lines under a ``[tryboot]``
   conditional section at the end of :file:`config.txt`

3. Reboot (or use ``piboot-try --reboot`` to force the "tryboot" mode
   immediately)

4. If the boot completes successfully, move the new (or changed) configuration
   lines out of the ``[tryboot]`` section into an ``[all]`` section (or other
   appropriate section)

Alternately, if you need to test a new configuration in :file:`cmdline.txt`:

1. Run ``sudo flash-kernel`` to set up new boot assets (note that
   :file:`current/cmdline.txt` is copied to create :file:`new/cmdline.txt`)

2. Edit :file:`new/cmdline.txt`

3. Reboot (or use ``piboot-try --reboot`` to force the "tryboot" mode
   immediately)

4. If the boot completes successfully, then :file:`new/cmdline.txt` becomes
   :file:`current/cmdline.txt`


Restore old assets
~~~~~~~~~~~~~~~~~~

Sometimes bad configuration is not immediately obvious. If the "old" folder
still exists on the boot partition (if ``flash-kernel`` has not been run
since the latest boot), it can be restored to being the "current" folder with
``piboot-try --restore-old``. This moves "current" to "new" and "old" to
"current" on the boot partition.

Note that in this situation the :file:`new/state` file indicates a "good" state
because the "new" assets are already known good. In this state, the next boot
will use the assets from "current" (because the "new" assets are not untested).

In order to switch back to "new" assets you can use ``piboot-try --reset-new``.
This action re-writes :file:`new/state` to "unknown" causing the next boot to
double-boot to (re-)test the assets.


----

.. [#why] On later models (e.g. the Pi 5), the bootloader assets exist solely
   in the EEPROM which does not (currently) have an A/B mechanism

.. [#space] This is to ensure we have space on the boot partition; older
   releases of Ubuntu only allocated 256MB to the boot partition, and people
   may have upgraded their systems to questing, thus retaining this minimal
   size.

.. [#when-failed] Recall that the :file:`new/` folder will only exist if
   untested or tested-but-failed boot assets were already present.

.. [#reset-on-read] The register that stores the "tryboot" mode status is a
   clear-on-read register, ensuring the fallback.

.. [#later-than-reboot] The ``piboot-try-validate`` service runs later than
   ``piboot-try-reboot``, in the ``multi-user.target`` portion of the boot.

.. [#current-always-exists] It may appear from this sequence that there is a
   portion of time when :file:`current/` does not exist, but this is not the
   case. The sequence presented here is simplified for the sake of clarity; the
   actual sequence used is more complex but guarantees that "known good"
   assets are in place at all points during the operation.

.. [#enable-busybox] Note that if you want to use the busybox shell for
   debugging the initramfs, this means you should remove ``panic=10`` from the
   kernel command in :file:`cmdline.txt`.


.. _cloud-init: https://cloudinit.readthedocs.io/
