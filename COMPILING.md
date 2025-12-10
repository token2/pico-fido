# Compiling Pico-FIDO with Custom VID, PID, and AAGUID for RP2350

This guide walks you through compiling the pico-fido firmware from the token2 repository for RP2350 with your custom USB identifiers and AAGUID.

## Prerequisites

Before starting, ensure you have:
- ARM GCC toolchain installed (for Pico development)
- CMake 3.13 or higher
- Pico SDK properly installed
- Git
- Make

### Install on Ubuntu 22.04/24.04

```bash
# Update package lists
sudo apt-get update

# Install required packages
sudo apt-get install -y cmake gcc-arm-none-eabi libnewlib-arm-none-eabi build-essential git python3-pip

# Install additional dependencies for Pico SDK
sudo apt-get install -y libstdc++-arm-none-eabi-newlib
```

### Download and Setup Pico SDK

```bash
# Create a directory for Pico development
mkdir -p ~/pico
cd ~/pico

# Clone the Pico SDK
git clone https://github.com/raspberrypi/pico-sdk.git --branch master
cd pico-sdk
git submodule update --init

# Verify installation
cd ~
echo "Pico SDK path: $HOME/pico/pico-sdk"
```

**The PICO_SDK_PATH will be:** `$HOME/pico/pico-sdk`

## Step 1: Clone the pico-fido Repository

```bash
cd ~/pico
git clone https://github.com/token2/pico-fido
cd pico-fido
git submodule update --init --recursive
```

After this step, your working directory will be: `~/pico/pico-fido`

## Step 2: Modify AAGUID and LED Blinking details in the SDK code

The AAGUID is defined in `pico-keys-sdk/src/cbor.c`. Open the file and find the line:

```bash
nano ~/pico/pico-fido/pico-keys-sdk/src/cbor.c
```

Locate the AAGUID definition (search for `const uint8_t aaguid`):

**Find this:**
```c
const uint8_t aaguid[16] = { 0x89, 0xFB, 0x94, 0xB7, 0x06, 0xC9, 0x36, 0x73, 0x9B, 0x7E, 0x30, 0x52, 0x6D, 0x96, 0x81, 0x45 };
```

**Replace with:**
```c
const uint8_t aaguid[16] = { 0xab, 0x32, 0xf0, 0xc6, 0x22, 0x39, 0xaf, 0xbb, 0xc4, 0x70, 0xd2, 0xef, 0x4e, 0x25, 0x4d, 0xb7 };
```

Save the file (Ctrl+O, Enter, Ctrl+X in nano) or use your preferred editor.


### LED Blinking Defaults and Updated Values

The default LED blinking values defined in the SDK  
(see the original lines here: [pico-keys-sdk `led.h` L53–L62](https://github.com/polhenarejos/pico-keys-sdk/blob/main/src/led/led.h#L53-L62))  
were not always useful or easily distinguishable in practice.

To improve clarity, we updated the LED patterns as follows:

```c
enum  {
    MODE_NOT_MOUNTED = (MAX_BTNESS << LED_BTNESS_SHIFT) | (LED_COLOR_RED    << LED_COLOR_SHIFT) | (1500 << LED_ON_SHIFT) | (1500 << LED_OFF_SHIFT),
    MODE_MOUNTED     = (MAX_BTNESS << LED_BTNESS_SHIFT) | (LED_COLOR_GREEN  << LED_COLOR_SHIFT) | (3000 << LED_ON_SHIFT) | (3000 << LED_OFF_SHIFT),
    MODE_SUSPENDED   = (MAX_BTNESS << LED_BTNESS_SHIFT) | (LED_COLOR_BLUE   << LED_COLOR_SHIFT) | (1000 << LED_ON_SHIFT) | (2000 << LED_OFF_SHIFT),
    MODE_PROCESSING  = (MAX_BTNESS << LED_BTNESS_SHIFT) | (LED_COLOR_GREEN  << LED_COLOR_SHIFT) | (50   << LED_ON_SHIFT) | (50   << LED_OFF_SHIFT),
    MODE_BUTTON      = (MAX_BTNESS << LED_BTNESS_SHIFT) | (LED_COLOR_YELLOW << LED_COLOR_SHIFT) | (100  << LED_ON_SHIFT) | (100  << LED_OFF_SHIFT),

    MODE_ALWAYS_ON   = UINT32_MAX,
    MODE_ALWAYS_OFF  = 0
};





## Step 3: Create Build Directory

```bash
cd ~/pico/pico-fido
mkdir build
cd build
```

## Step 4: Configure with CMake

Run CMake with your custom VID/PID and board type:

```bash
 cmake .. -DPICO_SDK_PATH=$HOME/pico/pico-sdk -DPICO_BOARD=pico2 -DUSB_VID=0x349E -DUSB_PID=0x0099 
```

## Step 5: Compile the Firmware

```bash
make -j$(nproc)
```

The `-j$(nproc)` flag uses all available CPU cores to speed up compilation.

If compilation succeeds, you'll see:
```
[100%] Built target pico_fido
```

The compiled firmware will be located at:
```
build/pico_fido.uf2
```

## Step 6: Flash to RP2350

1. **Put your RP2350 board into bootloader mode:**
   - Hold the BOOTSEL button while plugging in USB, or
   - Hold BOOTSEL and press the RESET button

2. **Find the mount point** - A USB mass storage device will appear:
   ```bash
   lsblk
   # Look for a device like sda1 or sdb1 that appeared recently
   # Or check: ls /media/$USER/
   ```

3. **Mount it manually if needed:**
   ```bash
   mkdir -p ~/pico/rp2350-mount
   sudo mount /dev/sdX1 ~/pico/rp2350-mount
   # Replace sdX1 with the actual device from lsblk
   ```

4. **Copy the UF2 file:**
   ```bash
   cp ~/pico/pico-fido/build/pico_fido.uf2 ~/pico/rp2350-mount/
   ```
   
   Or if the device auto-mounted:
   ```bash
   cp ~/pico/pico-fido/build/pico_fido.uf2 /media/$USER/RP2350/
   ```

5. **Unmount and verify:**
   ```bash
   sync
   sudo umount ~/pico/rp2350-mount
   # The device will automatically disconnect and reboot
   ```

The firmware is now installed. The LED should blink periodically after a few seconds.

## Verification

Once flashed:
1. The LED should blink periodically (idle mode)
2. When you connect the device, it should appear as a USB HID device with your custom VID/PID (0x349E:0x0099)

Verify with:
```bash
lsusb | grep 349E
# Should show: "Bus XXX Device YYY: ID 349e:0099"
```

## Troubleshooting

**CMake configuration fails:**
- Ensure `PICO_SDK_PATH=$HOME/pico/pico-sdk` is set correctly
- Run `cmake --version` to verify CMake 3.13+
- If SDK not found, verify it exists: `ls -la $HOME/pico/pico-sdk`

**Compilation fails with cryptography errors:**
- Ensure submodules are fully initialized: `git submodule update --init --recursive`

**AAGUID not applying:**
- Check if it's defined as a constant in a header file that was included during compilation
- Some builds may require a clean rebuild: `rm -rf build && mkdir build && cd build`

**Device doesn't appear after flashing:**
- Try flashing again, ensure BOOTSEL was held during USB connection
- Check for any error messages in dmesg: `dmesg | tail -20`

## Notes

- The dummy VID/PID (0xFEFF:0xFCFD) is used by default to avoid licensing issues
- VID/PID (0x349E:0x0099) can be used only on devices provided by Token2
- This firmware is released under AGPLv3 - if you distribute modified binaries, you must provide source code
- The AAGUID identifies your authenticator type in the FIDO ecosystem; however, there is currently no certificate provided, therefore no attestation
