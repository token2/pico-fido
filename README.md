# Pico FIDO - Token2 PIN Complexity Fork

This is a fork of [Pico FIDO](https://github.com/polhenarejos/pico-fido) with **firmware-enforced PIN complexity** designed and tested for Token2 open hardware devices.

<img width="450" height="450" alt="image" src="https://github.com/user-attachments/assets/bc8bd3b4-ba55-4fa1-a836-1ef62ac26a5c" />


PIN complexity validation is enforced directly on the microcontroller during PIN setup, preventing weak credentials from being set regardless of client software.

## Changes from Original

- **Hardware-enforced PIN complexity** - PIN complexity validation enforced at the firmware level, preventing weak credentials from being set regardless of client software
- **LED behavior** - Modified LED status indicators from the original implementation
- **VID/PID** - Uses Token2's official USB identifiers instead of the generic dummy VID/PID
- **AAGUID** - Modified to match a real authenticator identifier (self-attestation only, no external attestation at this time)

This project is not only open-source software, but also fully open hardware. The repository includes all the resources needed to understand, reproduce, and modify the [hardware design](Hardware-PCB-BOM-SCH.zip). Specifically, it contains the schematics (SCH), printed circuit board (PCB) layouts, and a complete bill of materials (BOM) listing all components used. The PCB files are provided in standard Gerber format, which can be used directly for manufacturing.

## For Complete Documentation

See the [original Pico FIDO repository](https://github.com/polhenarejos/pico-fido) for full features and general documentation.
