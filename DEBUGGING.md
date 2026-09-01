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

## No-relay experiment

The no-relay artifacts test whether DYA's custom settings and split event relay
block the peripheral notification queue. They keep Studio enabled on the dongle
but disable `CONFIG_ZMK_SETTINGS_RPC` and `CONFIG_ZMK_SPLIT_RELAY_EVENT` on all
three devices. The settings RPC module does not compile without its relay
feature, so they must be disabled together.

This changes the split GATT service. Do not mix normal and no-relay images:

1. Flash `settings_reset` to the dongle and both halves.
2. Flash the three matching `*_no_relay` images.
3. Power on the dongle, then the left and right halves.
4. Verify typing before opening DYA Studio.
5. Connect DYA, browse the same settings that previously caused the right-half
   position queue to fill, and continue typing cross-half words.

DYA keymap editing remains enabled. Runtime settings that depend on event relay,
including synchronizing idle and sleep settings across the halves, are not
expected to work in this experiment.

## Dongle-free left-central experiment

`eyelash_sofle_central_left_studio` restores the conventional split topology:
the left half is the central and the right half is its BLE peripheral. The left
half provides USB/BLE host output and DYA Studio. The image enables split relay
and the BLE management, runtime input processor, settings RPC, and runtime
sensor rotate DYA modules.

Changing the split central requires clearing the old split bonds:

1. Turn off the dongle and both halves.
2. Flash `settings_reset` to the left and right halves.
3. Flash `eyelash_sofle_central_left_studio` to the left half.
4. Flash either `eyelash_sofle_peripheral_right` or
   `eyelash_sofle_peripheral_right_debug` to the right half.
5. Keep the dongle off. Turn on the left half first, then the right half.
6. Connect the left half to the computer by USB for direct HID and DYA Studio.
   The right half may also remain connected by USB when using its debug image,
   but its key events still travel to the left half over BLE.

To return to dongle mode, reset the split settings again and flash the normal
dongle, left-peripheral, and right-peripheral artifacts as a matched set.
