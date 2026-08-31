# Right-half BLE debugging

The debug build is an additional artifact. It does not replace or modify the
normal right-half firmware.

## Firmware

Download `eyelash_sofle_peripheral_right_debug` from the GitHub Actions build
and flash it only to the right half. Keep the normal firmware on the dongle and
left half.

The debug image enables ZMK debug output over USB CDC. It deliberately leaves
Zephyr's full Bluetooth-stack debug logging disabled because that volume of log
traffic can disrupt split key delivery. Running the right half from USB can mask
battery or regulator problems, so compare it with a separate USB-powered run
using the normal right-half firmware.

## Capture setup

1. Connect the dongle normally so DYA Studio can use its serial port.
2. Connect the right half with a second USB data cable.
3. On macOS, find the new serial device after connecting the right half:

   ```sh
   ls /dev/cu.usbmodem*
   ```

4. Capture the right-half log before opening DYA Studio:

   ```sh
   screen /dev/cu.usbmodemXXXX 115200
   ```

5. Type a short cross-half test repeatedly, click **Connect** in DYA Studio,
   and keep capturing through any lag or disconnect.
6. Record whether lag began at Studio connection, the time of the first delayed
   key, and the time the display changed to the disconnected icon.

Exit `screen` with `Ctrl-A`, then `K`.

## Relevant messages

Preserve the complete log, including at least 30 seconds before the first lag.
The most useful messages contain:

```text
Disconnected
reason 0x08
Error notifying
Not connected
Failed to queue
message queue full
relay event
bt_att
stack overflow
USAGE FAULT
```

`reason 0x08` is a BLE connection timeout. Queue or notification failures before
that line can distinguish a firmware scheduling problem from a link that simply
stopped exchanging packets.
