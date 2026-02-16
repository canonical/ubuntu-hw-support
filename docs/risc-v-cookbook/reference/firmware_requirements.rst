.. SPDX-License-Identifier: CC-BY-SA-4.0

Firmware requirements
=====================

UEFI support
------------

Ubuntu boots via UEFI. The requirements specified in the
`Embedded Base Boot Requirements (EBBR) specification
<https://github.com/arm-software/ebbr>`_
should be fulfilled

U-Boot
''''''

In U-Boot the following configuration settings are needed:

* CONFIG_EFI_LOADER=y
* CONFIG_CMD_EFICONFIG=y
* CONFIG_CMD_BOOTEFI_BOOTMGR=y
* CONFIG_BOOTMETH_EFI_BOOTMGR=y (or deprecated CONFIG_DISTRO_DEFAULTS=y)

The following configuration settings are recommended:

* CONFIG_EFI_CMD_EFIDEBUG=y
* CONFIG_CMD_NVEDIT_EFI=y
* CONFIG_HEXDUMP=y

Network booting
---------------

During network booting the GRUB binary is discovered via PXE booting.
The GRUB configuration file (grub.cfg) will contain HTTP(s) URLs for the
Linux kernel and the initial RAM disk.

The following UEFI protocol need to be enabled:

* HTTP Protocol
* HTTP Utilities Protocol
* IPv4 Configuration II Protocol
* Simple Network Protocol

EDK II
''''''

In EDK II the following drivers should be enabled:

+--------------------------+--------------------------------------------------+
| Configuration Flag       | Driver                                           |
+==========================+==================================================+
| NETWORK_HTTP_BOOT_ENABLE | NetworkPkg/DnsDxe/DnsDxe.inf                     |
|                          | NetworkPkg/HttpDxe/HttpDxe.inf                   |
|                          | NetworkPkg/HttpUtilitiesDxe/HttpUtilitiesDxe.inf |
+--------------------------+--------------------------------------------------+
| NETWORK_IP4_ENABLE       | NetworkPkg/ArpDxe/ArpDxe.inf                     |
|                          | NetworkPkg/Dhcp4Dxe/Dhcp4Dxe.inf                 |
|                          | NetworkPkg/Ip4Dxe/Ip4Dxe.inf                     |
|                          | NetworkPkg/Udp4Dxe/Udp4Dxe.inf                   |
|                          | NetworkPkg/Mtftp4Dxe/Mtftp4Dxe.inf               |
+--------------------------+--------------------------------------------------+
| NETWORK_IP6_ENABLE       | NetworkPkg/Dhcp6Dxe/Dhcp6Dxe.inf                 |
|                          | NetworkPkg/Ip6Dxe/Ip6Dxe.inf                     |
|                          | NetworkPkg/Udp6Dxe/Udp6Dxe.inf                   |
|                          | NetworkPkg/Mtftp6Dxe/Mtftp6Dxe.inf               |
+--------------------------+--------------------------------------------------+
| NETWORK_TLS_ENABLE       | CryptoPkg/Library/OpensslLib/OpensslLib.inf      |
|                          | NetworkPkg/TlsDxe/TlsDxe.inf                     |
|                          | NetworkPkg/TlsAuthConfigDxe/TlsAuthConfigDxe.inf |
+--------------------------+--------------------------------------------------+
| NETWORK_PXE_BOOT_ENABLE  | NetworkPkg/UefiPxeBcDxe/UefiPxeBcDxe.inf         |
+--------------------------+--------------------------------------------------+
| NETWORK_SNP_ENABLE       | NetworkPkg/SnpDxe/SnpDxe.inf                     |
+--------------------------+--------------------------------------------------+

U-Boot
''''''

In U-Boot the following configuration settings are needed:

* CONFIG_NET_LWIP=y
* CONFIG_DNS=y
* CONFIG_WGET=y
* CONFIG_EFI_IP4_CONFIG2_PROTOCOL=y
* CONFIG_EFI_HTTP_PROTOCOL=y

For testing these additional settings are recommended:

* CONFIG_CMD_DNS
* CONFIG_CMD_WGET
