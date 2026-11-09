# Firmware binaries

Store only tested, versioned firmware binaries here.

Suggested layout:

- `global/<version>/firmware.bin` — release available to all compatible devices.
- `devices/<BOARD_ID>/<version>/firmware.bin` — release targeted to one configured board ID.

Never replace a published binary in place. Publish a new version and update `manifest.json` only after verifying the binary and its SHA-256 digest.
