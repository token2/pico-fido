# Pico FIDO - Token2 PIN Complexity Fork

This is a fork of [Pico FIDO](https://github.com/polhenarejos/pico-fido) with **firmware-enforced PIN complexity** designed and tested for Token2 open hardware devices.

PIN complexity validation is enforced directly on the microcontroller during PIN setup, preventing weak credentials from being set regardless of client software.

## Changes from Original

- **Hardware-enforced PIN complexity** - PIN complexity validation enforced at the firmware level, preventing weak credentials from being set regardless of client software
- **LED behavior** - Modified LED status indicators from the original implementation
- **VID/PID** - Uses Token2's official USB identifiers instead of the generic dummy VID/PID
- **AAGUID** - Modified to match a real authenticator identifier (self-attestation only, no external attestation at this time)

## For Complete Documentation

See the [original Pico FIDO repository](https://github.com/polhenarejos/pico-fido) for full features and general documentation.
