# MMDVM for Pi-Star

[![Firmware build](https://github.com/MW0MWZ/MMDVM/actions/workflows/firmware.yml/badge.svg?branch=master)](https://github.com/MW0MWZ/MMDVM/actions/workflows/firmware.yml)
[![Latest release](https://img.shields.io/github/v/release/MW0MWZ/MMDVM?label=release&sort=semver)](https://github.com/MW0MWZ/MMDVM/releases/latest)
[![Latest pre-release](https://img.shields.io/github/v/release/MW0MWZ/MMDVM?include_prereleases&label=pre-release&sort=semver)](https://github.com/MW0MWZ/MMDVM/releases)
[![Release date](https://img.shields.io/github/release-date-pre/MW0MWZ/MMDVM?label=released)](https://github.com/MW0MWZ/MMDVM/releases)
[![Licence: GPL v2](https://img.shields.io/badge/licence-GPL%20v2-blue)](LICENCE)

This is Pi-Star's fork of [G4KLX's MMDVM](https://github.com/g4klx/MMDVM), the firmware for full MMDVM modems and repeater boards. The firmware itself is upstream's. This fork adds a build of every supported STM32 board on every push, and publishes each release with a manifest and checksums for Pi-Star to upgrade a modem from. Fixes and features made here are offered back upstream.

## What this fork adds

| Path | What it is |
|---|---|
| [`boards.json`](boards.json) | The board map: each board by the name Pi-Star knows it by (the ids from Pi-Star 4's `pistar-modemupgrade`, plus aliases), the Makefile target it is built with, the MCU crystal where it is not the target's default, any `Config.h` options set or cleared for it, and the ways it can be flashed. |
| [`scripts/release_build.sh`](scripts/release_build.sh) | Builds every board in `boards.json` from the committed tree, then writes `<id>.bin` per board, `firmware.json` and `SHA256SUMS`. |
| [`.github/workflows/firmware.yml`](.github/workflows/firmware.yml) | Runs the build on every push and pull request, and publishes a release when a version is tagged. |

The build fails when a Makefile board target is not used by any board in `boards.json`, so a board added upstream is never silently left out. It also fails when a board names a target that does not exist, or a name is used twice.

## Boards

| Pi-Star id | Board | Target | Flashed with |
|---|---|---|---|
| `rpt_mmdvm_pi` | MMDVM-Pi (STM32F446) | `pi` | stm32flash (GPIO 20/21) |
| `rpt_zum-0.9` | ZUM Radio-MMDVM for Pi v0.9 | `pi` | stm32flash (GPIO 20/21) |
| `rpt_mmdvm_pi-f722` | MMDVM-Pi (STM32F722) | `pi-f722` | stm32flash (GPIO 20/21) |
| `rpt_zum-1.0` | ZUM Radio-MMDVM for Pi v1.0 | `pi-f722` | stm32flash (GPIO 20/21) |
| `rpt_mmdvm_f4m` | MMDVM-F4M Pi-Hat (F0DEI) | `f4m` | stm32flash (GPIO 20/21) |
| `rpt_mmdvm_f7m` | MMDVM-F7M Pi-Hat (F0DEI) | `f7m` | stm32flash (GPIO 20/21) |
| `rpt_mmdvm_hat` (also `rpt_mmdvm_hat-0.2`, `rpt_mmdvm_hat-0.3`) | MMDVM_RPT_Hat v0.2 / v0.3 (F0DEI, DB9MAT, DF2ET) | `rpt_hat` | stm32flash (GPIO 20/21) |
| `rpt_mmdvm_hat_tgo` | MMDVM_RPT_Hat (BG4TGO, STM32F405) | `rpt_hat_tgo` | by hand |
| `rpt_builder_v3` (also `rpt_builder_v4`) | Repeater Builder STM32-DVM v3 / v4 | `dvm` | stm32flash (GPIO 20/21) |
| `rpt_builder_v5` | Repeater Builder STM32-DVM v5 | `dvm722` | stm32flash (GPIO 20/21) |
| `rpt_drcc_nqf` | DRCC_DVM (BG7NQF) | `drcc_nqf` | by hand |
| `rpt_drcc_hhp446` | DRCC_DVM (BG5HHP, STM32F446) | `hhp446` | by hand |
| `rpt_eda405` | STM32-DVM-MTR2K / MASTR3 (STM32F405) | `eda405` | by hand |
| `rpt_eda446` | STM32-DVM-MTR2K / MASTR3 (STM32F446) | `eda446` | by hand |
| `rpt_nucleo-64` | Nucleo-64 STM32F446RE (Morpho header) | `nucleo` | st-flash, stm32flash (GPIO 20/21) |
| `rpt_nucleo-64-arduino` | Nucleo-64 STM32F446RE (Arduino header) | `nucleo` | st-flash |
| `rpt_nucleo-144` | Nucleo-144 STM32F767ZI | `f767` | st-flash, stm32flash (GPIO 20/21) |
| `rpt_discovery` | STM32F4-Discovery | `dis` | st-flash |

"stm32flash (GPIO 20/21)" is the Pi's serial port, with the STM32's BOOT0 and RESET on Raspberry Pi GPIO 20 and 21 (BCM). "st-flash" is the ST-LINK on the board's own USB. "By hand" is the serial port with BOOT0 set and reset pressed by hand; any board on a serial port can be flashed that way.

Boards that share a target, crystal and options share an image. The ZUM Radio boards have a 12 MHz MCU crystal and are built without `EXTERNAL_OSC`; the DRCC boards have no mode LEDs and are built without `MODE_LEDS`.

MMDVM firmware does not name the board in its version string (only the DRCC and Repeater Builder builds do), so Pi-Star asks which board a modem is, once. What the modem reports narrows the choice.

## Releases

Each [release](https://github.com/MW0MWZ/MMDVM/releases) carries:

- `<id>.bin` for every board in `boards.json` (for example `rpt_mmdvm_pi.bin`, `rpt_zum-1.0.bin`, `rpt_mmdvm_hat.bin`).
- `firmware.json`, the manifest (`"format": 1`, `"family": "MMDVM"`). For every board it lists the id, name and aliases, the Makefile target, MCU and crystal, the image file, size and SHA-256, and the exact version string the image reports to MMDVM-Host (which ends `GitID #<commit>`). It also gives the hardware type and TCXO the image reports, and each way it can be flashed (from `0x08000000`).
- `SHA256SUMS`, covering the images and the manifest.

To check a release by hand: `sha256sum -c SHA256SUMS`.

Releases are tagged with the firmware's version from [`Version.h`](Version.h), a date, with a `v`:

- `v<VERSION>` (for example `v20261123`) is a release, and becomes the latest release.
- `v<VERSION>-<suffix>` is a pre-release: `-test` while a change is being tried, `-beta` once it works.

A tag that does not match `Version.h` fails the build. Builds that are not tagged are named `v<VERSION>-g<commit>`.

## Compatibility with MMDVM-Host

The firmware and [MMDVM-Host](https://github.com/g4klx/MMDVM-Host) are upgraded separately, so every change here must work with any mix of versions:

- the firmware never sends a message an older MMDVM-Host does not know;
- new information is only sent in answer to a new command, which older firmware refuses (NAK), so the host knows not to ask again;
- existing messages, the version string and the configuration layout are never changed.

### Display port speed (`0x8F`)

The display port (the Nextion serial port, `SERIAL_REPEATER`) runs at `SERIAL_REPEATER_BAUD_RATE` (9600). A host can change its speed for a while, for example to load a screen design faster:

| | |
|---|---|
| Command | `E0 07 8F` followed by the speed as 4 bytes, least significant first (115200 is `00 C2 01 00`) |
| Speeds | 9600, 19200, 38400, 57600 or 115200 |
| Reply | ACK once the port runs at the new speed; NAK 4 for any other speed or length |

Anything already queued for the display is sent at the old speed first. The speed goes back to `SERIAL_REPEATER_BAUD_RATE` on the next `GET_VERSION` (which every host sends when it opens the modem) and on a reset. Nothing changes unless the command is sent. Firmware without this command, and builds without `SERIAL_REPEATER`, answer NAK 1.

### LEDs off (`0x8E`)

The LEDs work as always unless a host switches them off:

| | |
|---|---|
| Command | `E0 04 8E 01` switches the COS and mode LEDs off; `E0 04 8E 00` puts them back to normal |
| Reply | ACK; NAK 4 for any other value or length |

Switched off, the LEDs stay dark until they are switched back on or the modem is reset. `GET_VERSION` does not switch them back on, so the setting survives a host opening the modem. Switched back on, they show the modem's state again at once.

Only LEDs change:

- the service LED goes dark too where it is constant (`CONSTANT_SRV_LED`); where it blinks it keeps blinking, as it may be what keeps an external hardware watchdog from resetting the modem;
- the PTT line and the mode pins (`MODE_PINS`, which switch external channels or filters) are not LEDs and always follow the modem.

Firmware without this command answers NAK 1.

## Building

The build needs the ARM GCC toolchain, `make`, `git`, `jq` and the [STM32F4XX_Lib](https://github.com/juribeparada/STM32F4XX_Lib) and [STM32F7XX_Lib](https://github.com/juribeparada/STM32F7XX_Lib) submodules. To build every board as CI does, in Docker:

    git clone --recurse-submodules https://github.com/MW0MWZ/MMDVM
    cd MMDVM
    docker run --rm -v "$PWD":/src -w /src debian:bookworm-slim bash -c '
      apt-get update && apt-get install -y --no-install-recommends gcc-arm-none-eabi \
        libnewlib-arm-none-eabi libstdc++-arm-none-eabi-newlib make git jq binutils bsdextrautils
      git config --global --add safe.directory /src
      scripts/release_build.sh dist'

The images, `firmware.json` and `SHA256SUMS` are written to `dist/`. Only committed changes are built. To build one board by hand, run `make <target>` (see the Makefile and [BUILD.txt](BUILD.txt)).

# Introduction

This is the source code of the MMDVM firmware that supports D-Star, DMR, System Fusion, P25, NXDN, POCSAG, and FM modes.

It runs on the Arduino Due, the ST-Micro STM32F4xxx and STM32F7xxx processors, as well as the Teensy 3.5/3.6. What these platforms have in common is the use of an ARM Cortex-M3, M4, or M7 processors with a minimum clock speed greater of 80 MHz, and access to at least one analogue to digital converter, one digital to analogue converter, as well as a number of GPIO pins.

In order to build this software for the Arduino Due, you will need to edit a file within the Arduino GUI and that is detailed in the BUILD.txt file. The STM32 support is supplied via the ARM GCC compiler. The Teensy support uses Teensyduino.

This software is licenced under the GPL v2 and is primarily intended for amateur and educational use.

Portions of the ARM support code include the following copyright:

   Copyright (c) 2011 - 2013 ARM LIMITED

   All rights reserved.
   Redistribution and use in source and binary forms, with or without
   modification, are permitted provided that the following conditions are met:
   - Redistributions of source code must retain the above copyright
     notice, this list of conditions and the following disclaimer.
   - Redistributions in binary form must reproduce the above copyright
     notice, this list of conditions and the following disclaimer in the
     documentation and/or other materials provided with the distribution.
   - Neither the name of ARM nor the names of its contributors may be used
     to endorse or promote products derived from this software without
     specific prior written permission.

   THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
   AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
   IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
   ARE DISCLAIMED. IN NO EVENT SHALL COPYRIGHT HOLDERS AND CONTRIBUTORS BE
   LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
   CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
   SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
   INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
   CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
   ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
   POSSIBILITY OF SUCH DAMAGE.
