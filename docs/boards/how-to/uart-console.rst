=========================
Connect to a UART console
=========================

Serial :term:`UART` consoles permit communication between a computer and your
board.


Hardware
========

On some boards (e.g. the HiFive Unmatched), the UART console is exposed via a
USB socket, typically a micro-USB socket, and in these cases it is common that
multiple UARTs are exposed by the one socket. However, most boards expose their
UART console via simple header pins using `RS-232`_ signaling.

UARTs consist of a minimum of three pins:

* ground (often abbreviated GND),

* receive (RX), and

* transmit (TX)

On some boards, the UART has its own dedicated header. On others, the UART is
part of a larger :term:`GPIO` header.

To connect your computer to the UART pins on your board, a USB adapter is
commonly required. Some minor care in selecting adapters is needed: you want an
adapter that signals at :term:`TTL` levels (3.3 volts) rather than the
traditional `RS-232`_ levels (between 3 and 15 volts).

Adapters that use traditional RS-232 levels, which would cause permanent damage
to most single board computers, typically have a `DE-9`_ connector. Adapters
that signal at TTL levels, by contrast, have three or four `DuPont connector`_
tails:

.. image:: /images/uart-ttl-adapter.jpg
    :alt: A close-up of a typical USB UART adapter, showing the USB end on the
          left, and the "tails" with the female DuPont connectors on the right.
          The tails have the common colors, red, black, white, and green.

The receive and transmit lines cross between the computer and the board. In
other words, the computer's RX pin connects to the board's TX pin,
and the computer's TX pin connects to the board's RX pin. The GND pins are
simply connected together.

.. image:: /images/uart-connections.*
    :alt: A diagram showing the computer's GND, TX, and RX connectors on the
          left, and the board's GND, TX, and RX pins on the right. Dashed
          lines connect GND to GND, TX to RX, and RX to TX. The red +5V pin
          is also shown, unconnected.

If your adapter includes a red 5V output line, it should remain disconnected
from the board.

.. warning::

    Many boards use 3.3 V on their GPIO pins and do not tolerate higher
    voltages. Some boards use 1.8 V. It is crucial to check the documentation
    for your specific board to determine the correct GPIO voltage, as exceeding
    the recommended voltage can damage the board.

On some UART adapters the voltage needs to be set via a jumper:

.. image:: /images/uart-ttl-adapter-ch340g.jpg
    :alt: A CH340G base UART adapter with 3.3 V selected.

Once connected to your computer, your USB UART adapter will appear as a tty
device, commonly named :file:`/dev/ttyUSB0`. However, depending on the hardware
used, other device names are possible including :file:`/dev/ttyACM0` and
:file:`/dev/ttyAMA0`. If more than one serial adapter is present, the number at
the end may be different too. If in doubt, check the output of
:manpage:`dmesg(1)`. When attached you should see something like the following
appear:

.. terminal::
    :user: ubuntu
    :host: ubuntu

    :input: sudo dmesg
    ...
    [605927.453848] usb 2-2: new full-speed USB device number 2 using xhci-hcd
    [605927.599633] usb 2-2: New USB device found, idVendor=067b, idProduct=2303, bcdDevice= 3.00
    [605927.599645] usb 2-2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    [605927.599649] usb 2-2: Product: USB-Serial Controller
    [605927.599652] usb 2-2: Manufacturer: Prolific Technology Inc.
    [605927.653506] usbcore: registered new interface driver usbserial_generic
    [605927.653537] usbserial: USB Serial support registered for generic
    [605927.656003] usbcore: registered new interface driver pl2303
    [605927.656045] usbserial: USB Serial support registered for pl2303
    [605927.656109] pl2303 2-2:1.0: pl2303 converter detected
    [605927.682845] usb 2-2: pl2303 converter now attached to ttyUSB0

Note that the last line displays the device name of the newly registered
adapter.

.. _DE-9: https://en.wikipedia.org/wiki/D-subminiature
.. _RS-232: https://en.wikipedia.org/wiki/RS-232
.. _DuPont connector: https://en.wikipedia.org/wiki/Pin_header


Software
========

The Ubuntu archive contains several serial clients suitable for communicating
with a UART console. The following is an example of installing the popular
:manpage:`picocom(1)` client, and accessing the UART device connected to
:file:`/dev/ttyUSB0` at a `baud rate`_ of 115200bps:

.. code-block:: text

    sudo apt install picocom
    picocom -b 115200 /dev/ttyUSB0

You should replace the baud rate ``115200`` with the one specified by your
board. However, 115200bps is an extremely common default, and can generally be
assumed if the specific rate is unknown. Replace the device
:file:`/dev/ttyUSB0` with the one for your adapter (determined above).

To exit picocom, type :kbd:`Ctrl+A Ctrl+X`.

Another common choice is GNU :manpage:`screen(1)`. Once again, substitute the
device and baud-rate accordingly:

.. code-block:: text

    sudo apt install screen
    screen /dev/ttyUSB0 115200

To exit screen, type :kbd:`Ctrl+A K` then answer ``y`` to the prompt.

.. _baud rate: https://en.wikipedia.org/wiki/Baud


Troubleshooting
===============

Always troubleshoot your hardware first. Ensure your connections are good, that
the TX and RX pins are connected the right way round, and that the GND pin is
connected properly. If your UART pins are part of a larger GPIO header,
double-check you are looking at the correct set of pins for your board.

The following sections deal with specific problems common to UART usage.


Permission denied
-----------------

Typically, USB UART adapters will be accessible to any regular logged in user.
However, some serial interfaces (particularly built-in ones) do not match the
"uaccess" mechanism, and are only accessible to the root user and members of
the ``dialout`` group. If you are seeing a permission denied error, check your
access to the serial device:

.. code-block:: text

    test -r /dev/ttyUSB0 || echo "No read access"
    test -w /dev/ttyUSB0 || echo "No write access"

If needed, the following command will add the current user to the ``dialout``
group, but you will need to logout and login again for this to take effect:

.. code-block:: text

    sudo adduser $USER dialout


Flow control
------------

Some UARTs have additional pins for `flow control`_ (sometimes labeled
:abbr:`RTS (Request To Send)` and :abbr:`CTS (Clear To Send)`). However, the
vast majority of USB UART adapters only have the basic three pins. In such
cases, the flow-control pins can simply be ignored.

The lack of hardware flow-control does mean that pasting large chunks of text
can fill the UART's buffer and result in lost characters. However, because lack
of flow control is so common these days, workarounds are present in many serial
clients (typically some form of enforced delay between transmitted
key-presses).

screen
    Use :kbd:`Ctrl+A :` to enter command mode, then enter ``slowpaste 1`` to
    set a 1ms delay on character transmission.

minicom
    Under terminal settings (accessed with :kbd:`Ctrl+A T`), change the
    "Character tx delay" to 1 for a 1ms delay between character transmissions.

picocom
    Unfortunately this functionality is not supported yet.

.. _flow control: https://en.wikipedia.org/wiki/Flow_control_(data)#Hardware_flow_control


Full screen applications
------------------------

Some console applications take up the full screen, e.g. :manpage:`top(1)` or
:manpage:`vim(1)`. However, a serial terminal has no direct means of knowing
how many columns or rows are available to the client. Typically, the terminal
will assume it is only 80 columns by 24 rows large. If this does not match how
many columns and rows are *actually* available, the application will either
wind up compressed to the top left corner of the client, or will wrap its lines
resulting in a "corrupted" display.

To work around this, open another terminal of the same size as the one running
your serial client (if your terminal supports tabs, you may wish to open
another tab beside the client one; this ensures both terminals are the same
size). In the new terminal, run the following command:

.. code-block:: text

    echo stty cols $COLUMNS rows $LINES

For example:

.. terminal::
    :user: ubuntu
    :host: ubuntu

    :input: echo stty cols $COLUMNS rows $LINES
    stty cols 105 rows 51

Paste the output of this command into your serial client, and set the TERM
environment variable to ``screen-256color`` to indicate that the terminal can
use modern color codes too:

.. code-block:: text

    stty cols 105 rows 51
    export TERM=screen-256color

Once this is done, provided you do not resize the terminal containing your
serial client, you should find you are able to run full screen applications
like top and vim as normal (albeit full screen updates will be rather slow at
115200bps).
