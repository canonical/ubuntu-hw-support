===========
Glossary
===========


.. glossary::

    API
        An `Application Programming Interface
        <https://en.wikipedia.org/wiki/API>`_

    DHCP
        The `Dynamic Host Configuration Protocol
        <https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol>`_
        is typically used when bringing up a network interface to obtain an
        IPv4 (and sometimes IPv6) address. It is also commonly used in the
        context of :term:`SBCs <SBC>` to provide information about any
        available network boot facilities (see :term:`TFTP`). See also:
        :rfc:`1497`, :rfc:`2132`.

    EBBR
        The `Embedded Base Boot Requirements
        <https://arm-software.github.io/ebbr/index.html>`_ specification.

    EDID
        The `Extended Display Identification Data
        <https://en.wikipedia.org/wiki/Extended_Display_Identification_Data>`_
        standard supported by the vast majority of computer monitors and TVs,
        especially those supporting HDMI inputs.

    EEPROM
        `Electrically Erasable Programmable Read-Only Memory
        <https://en.wikipedia.org/wiki/EEPROM>`_. A type of non-volatile memory
        which is commonly used in computers (both small and large) for storage
        of boot time configuration.

    eMMC
        `Embedded MultiMediaCard
        <https://en.wikipedia.org/wiki/MultiMediaCard#eMMC>`_, a storage
        standard that puts the components of a MultiMediaCard in an integrated
        circuit for use in circuit boards as non-volatile storage.

    FAT
        The `File Allocation Table
        <https://en.wikipedia.org/wiki/File_Allocation_Table>`_ file-system. A
        very basic file-system (lacking journaling, indexing, UNIX attributes,
        file ownership, and with very crude resolution of timestamps) with a
        long and convoluted history, that has become the modern de facto
        standard boot-time file-system owing to its simplicity of
        implementation.

    GPIO
        `General Purpose Input/Output
        <https://en.wikipedia.org/wiki/General-purpose_input/output>`_; in the
        context of :term:`SBCs <SBC>`, this typically refers to the prominent
        pin-header (often containing 2×20 pins) along one edge of the board.
        Some (but not all) of these pins may be controlled by software,
        operating as inputs or outputs.

    GRUB
        The GNU `GRand Unified Bootloader
        <https://www.gnu.org/software/grub/>`_, commonly used as the Linux
        bootloader on PC platforms, and a few :term:`SBCs <SBC>` too.

    HAT
        Hardware Attached on Top; the Raspberry Pi `specification
        <https://github.com/raspberrypi/hats>`_ guiding the design of boards
        intended to connect to the 40-pin :term:`GPIO` header, and sit "on top"
        of the Pi. With the advent of the Pi 5, the specification is now
        superseded by the `HAT+ specification
        <https://datasheets.raspberrypi.com/hat/hat-plus-specification.pdf>`_.

    I²C
        The `Inter-Integrated Circuit
        <https://en.wikipedia.org/wiki/I%C2%B2C>`_ interface. A low-speed
        peripheral bus capable of connecting multiple peripherals sharing the
        same two wires for communication. Typically runs at a bandwidth of
        100Kbits/s. Commonly found exposed on the :term:`GPIO` headers of many
        :term:`SBCs <SBC>`.

    JTAG
        An industry standard developed by the Joint Test Action Group (JTAG)
        for verification and testing of circuit boards. In the context of
        :term:`SBCs <SBC>`, typically refers to the physical `debug interface
        <https://en.wikipedia.org/wiki/JTAG#Electrical_characteristics>`_
        found on many boards.

    Netplan
        `Netplan <https://netplan.io/>`_ is a utility for easily configuring
        networking on a Linux system. You simply create a YAML description of
        the required network interfaces and what each should be configured to
        do.

    NVMe
        `Non-Volatile Memory express
        <https://en.wikipedia.org/wiki/NVM_Express>`_; the interface used by
        modern flash storage devices, developed to more closely match the
        capabilities and behavior of flash storage (in contrast to interfaces
        like :term:`SATA` which were developed for use with spinning media).

    OpenSBI
        OpenSBI is an implementation of RISC-V's Supervisor Binary Interface
        (:term:`SBI`).

    PPA
        Personal package archive

    QEMU
        The `Quick Emulator <https://www.qemu.org/>`_ project, a popular
        virtual machine implementation supporting multiple architectures and
        cross-architectural emulation under Linux/BSD.

    SATA
        `Serial AT Attachment <https://en.wikipedia.org/wiki/SATA>`_; an
        interface used by hard disk and flash storage drives, though now
        largely superseded by :term:`NVMe` in the case of flash storage.

    SBC
        A Single Board Computer.

    SBI
        The Supervisor Binary Interface defines an API on RISC-V for the
        communication with M-mode firmware.

    SPI
        The `Serial-Peripheral Interface
        <https://en.wikipedia.org/wiki/Serial_Peripheral_Interface>`_ is an
        (optionally) full-duplex peripheral bus capable of connecting multiple
        peripherals sharing three wires for communication, with separate "chip
        select" lines to indicate which device is currently in active
        communication. Commonly found exposed on the :term:`GPIO` headers of
        many :term:`SBCs <SBC>`.

    SSD
        `Solid State Drive <https://en.wikipedia.org/wiki/Solid-state_drive>`_;
        a type of persistent storage drive that has no moving parts, relying
        instead on flash memory.

    SoC
        A System-on-a-Chip is an integrated circuit design that combines
        elements of an electronic device onto a single chip instead of using
        separate components.

    TFTP
        The `Trivial File Transfer Protocol
        <https://en.wikipedia.org/wiki/Trivial_File_Transfer_Protocol>`_ is, as
        the name implies, a simplistic file transfer protocol which is commonly
        used as the initial protocol in network boot operations of :term:`SBCs
        <SBC>`. See also: :rfc:`1350`, :rfc:`2347`, :rfc:`1783`.

    TTL
        `Transistor-transistor logic
        <https://en.wikipedia.org/wiki/Transistor%E2%80%93transistor_logic>`_;
        a family of logic circuits. In the context of serial adapters and
        :term:`UART` consoles, however, this usually refers to the voltage
        level expected.

        The `RS-232 <https://en.wikipedia.org/wiki/RS-232>`_ standard (commonly
        used in serial interfaces) permits a wide range of voltages from 3V up
        to 15V. However, the UARTs on most :term:`SBCs <SBC>` expect no more
        than the 3.3V typically used on their :term:`GPIO` headers. Thus, "TTL
        UART" or "serial TTL" implies a UART interface (or adapter) that
        requires (or accepts) no more than 3.3V through its pins.

    UART
        `Universal Asynchronous receiver-transmitter
        <https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter>`_;
        the acronym most commonly used to reference a device's `serial port
        <https://en.wikipedia.org/wiki/Serial_port>`_ over which a Linux
        console may be accessed.

        Frequently used for debugging early boot problems, as the serial port
        is a much simpler device than video output, and thus less likely to
        fail in the event of kernel bugs.

    UEFI
        `Unified Extensible Firmware Interface
        <https://en.wikipedia.org/wiki/UEFI>`_ is a specification for the
        booting of computer hardware. Whilst nearly ubiquitous on modern PCs,
        support for it is considerably more sparse on :term:`SBCs <SBC>`.
