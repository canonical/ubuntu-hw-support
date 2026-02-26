.. _configure-your-board-for-headless-use:

Configure your board for headless use
=====================================

All Ubuntu pre-installed images, both for cloud and :term:`SBCs <SBC>` use
`cloud-init`_ for their first boot setup, including initial user creation. It
is possible to customize the first boot to configure a board for remote usage.
This includes securely importing all necessary credentials, and making it
possible to locate easily on the local network, removing the need to ever
attach a screen or keyboard to the board.

You will need the following:

* An Ubuntu One (Launchpad) or GitHub user account

* An SSH public/private key-pair (see :manpage:`ssh-keygen(1)` for more
  information)

* Your SSH public key registered with `Launchpad`_ or `GitHub`_


Cloud-init seeds
----------------

The cloud-init "seed" provides the configuration that cloud-init will act upon
during the first boot. The location of the seed is image dependent. On the
Ubuntu for Raspberry Pi images, it is present on the boot partition (the first
:term:`FAT` partition on the images). On other images, it may be present on a
:term:`FAT` partition labeled "CIDATA". The location will be documented in the
board's corresponding chapter in these docs.

Wherever it resides, the cloud-init seed always consists of the following
files, all in `YAML`_ format:

meta-data
    While mandatory, this is largely uninteresting for our purposes here. It
    simply "identifies" the board and specifies whether the seed has a
    network-based component (this is more useful in a cloud context)

user-data
    This defines the bulk of the cloud-init configuration, specifying initial
    user characteristics, packages to install, files to manipulate, and so on

network-config
    Optional to the seed, but required in our case to ensure that cloud-init
    has a valid network connection from which to retrieve keys and packages


Network configuration
---------------------

The :file:`network-config` file contains the :term:`netplan` compatible
configuration for networking cards present on the board. The default network
configuration typically *attempts* connection on available interfaces, but does
not require it. This is fine for the default cloud-init configuration, which
does not require network access to complete.

However, if your configuration requires network access for anything (installing
packages, retrieving SSH keys from an online account, mounting remote
file-systems, and so on), you will need to change the default configuration to
require a connection. This ensures that cloud-init will wait until the
specified connection is fully online before proceeding with the rest of its
configuration.

As an example, take the default :file:`network-config` on the Raspberry Pi
images:

.. code-block:: yaml

    version: 2
    ethernets:
      eth0:
        dhcp4: true
        optional: true

If your board has an ethernet port which will be connected on first boot, you
can simply change this indicate the connection is mandatory:

.. code-block:: yaml
    :emphasize-lines: 5

    version: 2
    ethernets:
      eth0:
        dhcp4: true
        optional: false

On a board which has ethernet, but which will be using wifi for connectivity
you might add a ``wifis`` section instead with a mandatory interface:

.. code-block:: yaml
    :emphasize-lines: 6-12

    version: 2
    ethernets:
      eth0:
        dhcp4: true
        optional: true
    wifis:
      wlan0:
        dhcp4: true
        optional: false
        access-points:
          my-wifi-ssid:
            password: "my very secret wifi password"

Note that there is no specific restriction on the type of network connectivity
required. Ethernet or wifi can be used for first boot configuration. That said,
ethernet is typically preferred where available as the simpler and more
reliable medium.


Customizing the users
---------------------

On the vast majority of pre-installed Ubuntu images, the default username is
"ubuntu" with a default password of "ubuntu". Obviously this is well known
and insecure. For this reason, SSH password-based authentication is disabled by
default on all such images, and an initial password change is mandated on
login.

All these aspects may be configured with the :file:`user-data` portion of the
cloud-init seed. The ``user`` and ``users`` keys control the details of the
user(s) created on first boot.

user
    Specifies the attributes of the "default" user, including:

    name
        The name of the user to create. Defaults to "ubuntu"

    plain_text_passwd
        The password for the user. Defaults to "ubuntu"

    hashed_passwd
        You can also supply a default password as a hash (see below for
        instructions on generating the hash)

    lock_passwd
        If set to ``true`` (the default), disables password based login

    groups
        The list of groups to add the user to

    homedir
        The location of the user's home directory. Defaults to
        :file:`/home/{name}`

    shell
        Path to the user's login shell. Defaults to :manpage:`bash(1)` on
        Ubuntu

    ssh_import_id
        Import SSH keys from the specified account (see below for more
        information)

    sudo
        List of strings containing :manpage:`sudo(8)` rules for this user

users
    The list of users to be created. Initially, this is just the "default" user
    defined by the ``user`` key above. However, additional entries using the
    same sub-keys as those under ``user`` may also be included in the list. The
    "default" entry may also be excluded to prevent its creation.

The default user configuration on Ubuntu could be expressed as follows. Note,
this configuration is implicit in the cloud-init installation; it doesn't need
to be specified in your :file:`user-data`, this is simply to give context for
the changes below:

.. code-block:: yaml

    user:
      name: ubuntu
      plain_text_passwd: "ubuntu"
      groups: [adm, cdrom, dip, lxd, sudo]
      lock_passwd: true
      shell: /bin/bash
      sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    users:
      - default

If you want to rename the default user to "fred" and set a different password
that isn't locked, you can use the following in your :file:`user-data`:

.. code-block:: yaml

    user:
      name: fred
      plain_text_passwd: "flintst0ne"
      lock_passwd: false

You can also specify a hash of a password:

.. code-block:: yaml
    :emphasize-lines: 3

    user:
      name: fred
      hashed_passwd: "$6$rounds=500000$V0fxPRRWCnTWfCIz$dV9YdtDo5MOrOyXPMw6tuHVtV/dxc3EtRzIyl7AaZD.GZvL0nNvdG1VT4xYwvM0e/j70eYsbRpKKB5CxtpGUd1"
      lock_passwd: false

The hash can be generated with the :manpage:`mkpasswd(1)` utility from the
:lp-pkg:`whois` package, like so:

.. terminal::
    :user: ubuntu
    :host: ubuntu

    :input: mkpasswd --method=SHA-512 --rounds=500000
    Password: # not echoed
    $6$rounds=500000$V0fxPRRWCnTWfCIz$dV9YdtDo5MOrOyXPMw6tuHVtV/dxc3EtRzIyl7AaZD.GZvL0nNvdG1VT4xYwvM0e/j70eYsbRpKKB5CxtpGUd1

.. warning::

    Be aware that this is barely more secure than a plain text password. In
    both cases, the password or the hash will typically be world readable after
    the machine has booted. You are *strongly recommended* not to rely on this
    for remote first boot login. See `SSH authentication`_ below for a more
    secure alternative.

To define a user in addition to the default one, add it to the ``users`` key.
Include "default" to ensure the user defined under ``user`` is also created:

.. code-block:: yaml

    user:
      name: fred
      hashed_passwd: "$6$rounds=500000$V0fxPRRWCnTWfCIz$dV9YdtDo5MOrOyXPMw6tuHVtV/dxc3EtRzIyl7AaZD.GZvL0nNvdG1VT4xYwvM0e/j70eYsbRpKKB5CxtpGUd1"
      lock_passwd: false
    users:
      - default
      - name: barney
        hashed_passwd: "$6$rounds=500000$TEz/1c9AInDtCeCu$enA9jQEKTDHjypdMXdfTXMN5Khw./J3r0uIzHpktNjxZXw26k22mwcJ68el8GFDSR5i6unmmg/ePm.lVxkfbF0"
        groups: [lxd]
        lock_passwd: false

Alternatively, you can suppress creation of the default user (by not including
"default" under ``users``), and simply define all the users directly:

.. code-block:: yaml

    users:
      - name: fred
        hashed_passwd: "$6$rounds=500000$V0fxPRRWCnTWfCIz$dV9YdtDo5MOrOyXPMw6tuHVtV/dxc3EtRzIyl7AaZD.GZvL0nNvdG1VT4xYwvM0e/j70eYsbRpKKB5CxtpGUd1"
        groups: [adm, cdrom, dip, lxd, sudo]
        lock_passwd: false
        sudo: ["ALL=(ALL) NOPASSWD:ALL"]
      - name: barney
        hashed_passwd: "$6$rounds=500000$TEz/1c9AInDtCeCu$enA9jQEKTDHjypdMXdfTXMN5Khw./J3r0uIzHpktNjxZXw26k22mwcJ68el8GFDSR5i6unmmg/ePm.lVxkfbF0"
        groups: [lxd]
        lock_passwd: false

.. note::

    In this case, as neither user is the "default" user, their definitions will
    not inherit from the cloud-init defaults. Remember to include sudo or
    administration rights for at least one user in this case.


SSH authentication
------------------

By default, password-based authentication for SSH is disabled because the
default usernames and passwords are both well known and trivially guessable.

You can control whether SSH password-based authentication is enabled via
cloud-init. You can also import SSH keys for public-key authentication from
either a GitHub or Launchpad account. The following keys in :file:`user-data`
are used for this:

ssh_pwauth
    If set to "true" (it is "false" by default), password-based authentication
    will be permitted for SSH

ssh_import_id
    Defines the list of accounts to request SSH public keys from. May be
    specified at the top-level, in which case imported keys are assigned to all
    users created by cloud-init, or under individual user definitions, in
    which case the keys apply just to that user

We *strongly recommend* you leave SSH password-based authentication disabled.
Importing SSH public keys for your user(s) from GitHub or Launchpad is a much
more secure option as at no point will your machine be remotely accessible with
a username / password combination defined in a world-readable file. Naturally,
this requires an internet connection (see `Network configuration`_ above).

As noted, the ``ssh_import_id`` value is a list of account names. GitHub
accounts are prefixed with ``gh:`` and Launchpad accounts with ``lp:``. For
example:

.. code-block:: yaml

    ssh_import_id:
      - lp:launchpad_username
      - gh:github_username

For a complete example, consider the following :file:`user-data` file, which
changes the default username to "fred", the default password to "flintst0ne",
leaves SSH password-based authentication disabled (explicitly), and imports SSH
keys from the GitHub user "fred_flintstone":

.. code-block:: yaml

    user:
      name: fred
      plain_text_passwd: flintst0ne
      lock_passwd: false
    ssh_pwauth: false
    ssh_import_id:
      - gh:fred_flintstone


Finding your board
------------------

A common issue with headless :term:`SBCs <SBC>` is how to locate them on the
network once they have booted. One method is to configure each with a static IP
address, but this involves a certain amount of complexity to ensure all
machines have a unique configuration. Another is to have a router configuration
which is capable of reporting newly seen machines.

However, a popular alternative is to use `mDNS`_. If you have
:manpage:`avahi-daemon(8)` active on your system, you can locate machines by
their hostname within the ``.local`` domain. To accomplish this, you will use
the following keys within :file:`user-data`:

hostname
    Sets the machine's hostname to the specified value

package_update
    If set to "true", causes the local package index to updated. This matters
    less on Ubuntu cloud images as they are regenerated daily. However, the
    Ubuntu board images are static once released and thus extremely likely to
    have an out of date package index on first boot

package_upgrade
    If set to "true", causes the package manager to run an upgrade of
    installed packages during first boot. Again, this is more important on
    Ubuntu board images than cloud images

packages
    The list of extra packages to install, once any requested update or upgrade
    is concluded

The following example will set the machine's hostname to "mypi", ensure all
packages are up to date, and install "avahi-daemon".

.. code-block:: yaml

    hostname: mypi
    package_update: true
    package_upgrade: true
    packages:
      - avahi-daemon

Be aware this will require an internet connection on first boot (see `Network
configuration`_ above).

Once cloud-init has finished, you should be able to reach your machine (with
:manpage:`ping(1)` or :manpage:`ssh(1)`) as "mypi.local", assuming you also
have "avahi-daemon" installed on your client machine.


Optimizations
-------------

If you are booting many boards with the same release or distribution, it may be
beneficial to configure a local apt cache (see :manpage:`apt-cacher-ng(8)`). If
you have such a cache, you can use it by specifying it in your
:file:`user-data`. Change the highlighted line to the URL for your cache:

.. code-block:: yaml
    :emphasize-lines: 3

    apt:
      conf: |
        Acquire::http { Proxy "http://acng.example.com:3142"; }

.. _Launchpad: https://launchpad.net/
.. _GitHub: https://github.com/
.. _YAML: https://en.wikipedia.org/wiki/YAML
.. _mDNS: https://en.wikipedia.org/wiki/Multicast_DNS
