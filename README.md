# Kinesis Advantage 360 Pro ZMK Config

Personal ZMK firmware configuration for the Kinesis Advantage 360
Pro, built locally in a container. The keymap lives in
[config/adv360.keymap](config/adv360.keymap); the rest of this
README covers editing it, building the firmware, and flashing it.

## Modifying the keymap

Edit [config/adv360.keymap](config/adv360.keymap) directly. The
[ZMK documentation](https://zmk.dev/docs) covers both basic and
advanced behaviours and includes an OS-compatibility table for
keycodes. Note that the RGB Underglow, Backlight and Power
Management sections do not apply to the Advantage 360 Pro's custom
ZMK fork; see [the note below](#note).

Some ZMK features (combos, for example) need the exact key
positions in the matrix. These are available in both image and text
form [here](assets/key-positions.md).

## Mod layer

Holding Mod, the inner key on the right module's top row, reaches
the shortcuts below. Keys are named by what they type on the base
layer.

| Shortcut | Effect |
| --- | --- |
| Mod+K / Mod+J | Backlight brighter / dimmer |
| Mod+Tab | Backlight on/off |
| Mod+Enter | Underglow on/off |
| Mod+H | Show the battery level while held |
| Mod+1 to Mod+5 | Select a Bluetooth profile |
| Mod+left GUI | Clear the selected Bluetooth profile |
| Mod+macro1 / Mod+macro3 | Left / right module bootloader |
| Mod+Esc | Unlock ZMK Studio |
| Mod+V | Type the firmware version |

## Building the firmware

The firmware is built inside a container. `make` drives the build
and picks Podman if it is installed, otherwise Docker.

### macOS (Docker)

Install Docker and a container runtime with
[Homebrew](https://brew.sh):

```shell
brew install docker colima
colima start
```

On Apple Silicon the build image is x86_64, so start Colima with a
matching architecture:

```shell
colima start --arch x86_64
```

### Fedora (Podman)

Podman ships with Fedora; install it and `make` if they are
missing:

```shell
sudo dnf install podman make
```

No daemon or extra runtime is needed, and `make` detects Podman
automatically.

### Build

Run `make` to build both halves, or `make left` to build only the
left-hand side:

```shell
make
```

The compiled `.uf2` files land in the `firmware` directory; the
leading part of each filename is the build timestamp.

### Cleanup

| Command | Effect |
| --- | --- |
| `make clean_firmware` | Remove compiled firmware, keep the image |
| `make clean_image` | Remove the build image, keep firmware |
| `make clean` | Remove both |

Rebuilding the image takes a while, so prefer `make clean_firmware`
during normal iteration.

## Flashing firmware

Follow the programming instructions on page 8 of the [Quick Start
Guide](https://kinesis-ergo.com/wp-content/uploads/Advantage360-Professional-QSG-v8-25-22.pdf).

1. Connect the left side to USB.
2. Press Mod+macro1 to enter bootloader mode; the half attaches as
   a USB drive.
3. Copy `left.uf2` from the `firmware` directory to the drive; it
   disconnects when done.
4. Power both halves off (unplug and switch off).
5. Switch the left side on.
6. Connect the right side to USB to power it on.
7. Press Mod+macro3 to put the right side into bootloader mode.
8. Copy `right.uf2` to the mounted drive.
9. Unplug the right side and switch it back on.

> Both halves also have physical reset buttons for entering and
> exiting bootloader mode. Their location is described in section
> 2.7 (page 9) of the [User
> Manual](https://kinesis-ergo.com/wp-content/uploads/Advantage360-ZMK-KB360-PRO-Users-Manual-v3-10-23.pdf),
> and their use in section 5.9 (page 14).

> Some operating systems do not always report the drive as ejected
> after a file is flashed, or throw a spurious error. This does not
> mean the flash failed.

## Versioning

The compilation date, branch and Git commit hash are recorded in a
macro reachable with Mod+V, which types out `YYYYMMDD-XXXX-YYYYYY`,
where `XXXX` is the first four characters of the branch and
`YYYYYY` is the commit hash.

## N-key rollover

NKRO is enabled by default, but the higher key ranges are not, for
compatibility. To use F13-F24 or the INTL1-9 keys with NKRO, change
`CONFIG_ZMK_HID_KEYBOARD_EXTENDED_REPORT=n` to `=y` in
[adv360_left_defconfig](config/boards/arm/adv360/adv360_left_defconfig#L65).

## Battery reporting

The left module reports its battery level over BLE through
`CONFIG_BT_BAS=y` in
[adv360_left_defconfig](config/boards/arm/adv360/adv360_left_defconfig).

The fork can also proxy the right module's level as a second
battery service, with
`CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING` and
`CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY`. Do not: BlueZ
rejects a device with more than one battery service, and the two
extra services shift every GATT handle above them, so a host that
is already bonded reads HID at the old handles and the keyboard
connects but types nothing until it is re-paired.

Reporting can cause some computers to wake repeatedly. If that
happens, set `CONFIG_BT_BAS` back to `n`; holding Mod+H still
shows the level on the underglow LEDs.

## Modifier indicator colour

The CAPS/NUM/SCROLL LOCK indicator LED colour is set with a
hexadecimal RGB code. For example,
`CONFIG_ZMK_RGB_UNDERGLOW_MOD_COLOR=0xFF0000` gives red. Set it on
both modules by updating both
[adv360_left_defconfig](config/boards/arm/adv360/adv360_left_defconfig)
and
[adv360_right_defconfig](config/boards/arm/adv360/adv360_right_defconfig).

## Layer colours

ZMK supports up to 32 layers, with the highest currently active
layer shown on the layer LEDs of each module. All possible colours
are listed below; for the first 8 layers the same colour is shown
on both modules. After that only the right module cycles, until it
"rolls over" and the left module advances too, and so on. To avoid
confusion, the black/off colour is used only for layer 0.

| Layer # | L/R | Layer # | L/R | Layer # | L/R | Layer # | L/R |
| ---: | :---: | ---: | :---: | ---: | :---: | ---: | :---: |
| 0 | <img valign='middle' src='assets/swatches/000000.svg'/> <img valign='middle' src='assets/swatches/000000.svg'/> | 8 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> | 16 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 24 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> |
| 1 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> | 9 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> | 17 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 25 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> |
| 2 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> | 10 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 18 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> | 26 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> |
| 3 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> | 11 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 19 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> | 27 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> |
| 4 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 12 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> | 20 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> | 28 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> |
| 5 | <img valign='middle' src='assets/swatches/FF00FF.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 13 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> | 21 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> | 29 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> |
| 6 | <img valign='middle' src='assets/swatches/00FFFF.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> | 14 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> | 22 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 30 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> |
| 7 | <img valign='middle' src='assets/swatches/FFFF00.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> | 15 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> | 23 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 31 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> |

## Note

By default this config references [a customised version of
ZMK](https://github.com/ReFil/zmk/tree/adv360-z3.5) with Advantage
360 Pro specific functionality and changes over [base
ZMK](https://github.com/zmkfirmware/zmk). The Kinesis fork is
regularly updated with the latest changes from base ZMK, but is not
always completely current; some features, such as new keycodes, are
not immediately available on the 360 Pro after they land in base
ZMK.

The Advantage 360 Pro is compatible with base ZMK (the merge pull
request is [here](https://github.com/zmkfirmware/zmk/pull/1454)),
but some of the more advanced features, such as the indicator RGB
LEDs, will not work, and Kinesis cannot provide support for base
ZMK.

## Other support

Further support resources are on kinesis-ergo.com:

* https://kinesis-ergo.com/support/kb360pro/#firmware-updates
* https://kinesis-ergo.com/support/kb360pro/#manuals

A hardware issue may need a support ticket directly with Kinesis:

* https://kinesis-ergo.com/support/kb360pro/#ticket
