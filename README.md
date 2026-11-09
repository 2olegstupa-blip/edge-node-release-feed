# Edge Node Release Feed

Public distribution repository for ESP32 firmware update metadata and binary releases.

This repository contains **firmware binaries and release metadata only**. Firmware source code, Wi-Fi credentials, Telegram tokens, GitHub access tokens, and other secrets must never be committed here.

## Update selection

The device reads `manifest.json` and checks for an entry matching its configured 5-character board ID in `devices`. If a device-specific entry exists, it takes priority over `global`. If no device-specific release is available, the device may use the global release.

A missing entry (`null` or absent key) means that no update is published for that target.

## Release descriptor

Each non-null release entry uses this structure:

```json
{
  "version": "1.0.0",
  "firmware_url": "https://raw.githubusercontent.com/2olegstupa-blip/edge-node-release-feed/main/firmware/global/1.0.0/firmware.bin",
  "sha256": "64-character-lowercase-sha256",
  "chip": "esp32",
  "board": "ESP32-WROOM-32",
  "inverter_profiles": ["deye_6k_sg03"],
  "release_notes": "Short description of the release"
}
```

- `version`: published firmware version.
- `firmware_url`: immutable, version-specific binary URL.
- `sha256`: SHA-256 digest of the exact binary.
- `chip` and `board`: hardware compatibility constraints.
- `inverter_profiles`: supported inverter profiles. Use the firmware configuration IDs: `deye_6k_sg03`, `deye_30k_sg02hp3`, and `deye_50k_sg01hp3`. An empty array is not accepted by the current ESP32 checker; list each supported profile explicitly.
- `release_notes`: short human-readable summary.

Device-specific releases must also be compatible with the device's hardware and configured inverter profile. A board ID is a release selector, **not** an authentication secret.

## Publishing a release

1. Build and test the firmware.
2. Upload the binary under a new versioned path in `firmware/`; do not overwrite an existing version.
3. Calculate the binary's SHA-256 digest.
4. Add the release descriptor to `manifest.json` under `global` or the intended key in `devices`.
5. Verify the raw manifest and binary URLs, digest, and hardware compatibility before announcing the release.

Do not publish an update until its binary has passed validation. The ESP32 currently performs manifest discovery and compatibility checks only. It does not install automatically. Before enabling installation, add and verify a cryptographic manifest signature (for example Ed25519 with a public key pinned in firmware), then verify the binary digest and preserve NVS settings.

## Security

HTTPS and SHA-256 protect transport and detect accidental or malicious binary corruption only when the expected digest itself is trusted. For stronger authenticity, the firmware updater should verify a signed manifest using a public key embedded in firmware. Never embed a private signing key or GitHub token in the device firmware.
