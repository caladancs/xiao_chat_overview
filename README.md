# xiao_chat

**Bluetooth Mesh firmware for networked interactive devices, built on Zephyr / nRF Connect SDK for the Seeed XIAO nRF52840 Sense.**

Each mesh node is a self-describing, scriptable device controller. Nodes drive LEDs, e-paper, audio, motors, RFID and sensors, run an on-device scripting language, and can be fully administered over the mesh from a single USB-connected controller.

![Zephyr](https://img.shields.io/badge/RTOS-Zephyr-blueviolet)
![nRF Connect SDK](https://img.shields.io/badge/nRF%20Connect%20SDK-v3.1.1-00A9CE)
![MCU](https://img.shields.io/badge/MCU-nRF52840-orange)
![Language](https://img.shields.io/badge/language-C-blue)

---

## Key features

- **Bluetooth Mesh networking**: a custom vendor model carries messages, presence, and a structured command protocol. Relay, Friend and GATT Proxy are supported.
- **Zero-touch provisioning**: a controller node discovers and provisions new nodes automatically, and each node configures itself.
- **Remote shell over mesh**: run any shell command on any node from one USB connection and get its output back over the mesh.
- **Embedded script engine**: an event-driven language (`TRIGGER`, `IF`/`ELSE`, `SET`, `DO`, `EMIT`) defines device behavior at runtime without reflashing.
- **Self-describing hardware**: each carrier board stores its peripheral configuration in EEPROM. One firmware image runs on every node.
- **Audio playback**: WAV files stream from flash to an I²S amplifier.
- **On-device storage**: LittleFS on internal flash and on 8 MB of external NOR flash.
- **Diagnostics**: battery monitoring, built-in self test, and persistent device identity.

---

## Architecture

```
                 ┌──────────────────────────────────────────┐
  Host PC  ──USB─┤ Controller node (Dongle / DK / XIAO)     │
                 │  • Provisioner                           │
                 │  • Remote shell                          │
                 │  • Network-wide control                  │
                 └───────────────────┬──────────────────────┘
                                     │  Bluetooth Mesh
          ┌──────────────────────────┼──────────────────────────┐
          ▼                          ▼                          ▼
 ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
 │  Device node    │◄─────►│  Device node    │◄─────►│  Device node    │
 │  Script engine  │ events│  Script engine  │ events│  Script engine  │
 │  + peripherals  │       │  + peripherals  │       │  + peripherals  │
 └─────────────────┘       └─────────────────┘       └─────────────────┘
```

| Layer | Responsibilities |
|---|---|
| **Mesh / transport** | Vendor model, provisioning, remote shell |
| **Application** | Script engine, event handling |
| **Drivers** | LED strip, e-paper, audio, RFID, motor, encoder, proximity, IMU |
| **Board abstraction** | EEPROM peripheral map, I/O expander, multi-slot SPI bus |
| **Platform** | LittleFS, settings/NVS, diagnostics |

---

## Tech stack

| | |
|---|---|
| **RTOS / SDK** | Zephyr RTOS, nRF Connect SDK v3.1.1 |
| **MCU** | Nordic nRF52840 (Arm Cortex-M4F, BLE 5) |
| **Language** | C (firmware), Python (host tools) |
| **Build** | CMake, west, Kconfig, Devicetree |
| **Interfaces** | SPI, I²C, I²S, PDM, ADC, USB CDC-ACM |

---

## Supported boards

| Board | Role |
|---|---|
| Seeed XIAO nRF52840 Sense / Sense Plus | Device node |
| nRF52840 Dongle | USB mesh controller |
| nRF52840 DK | Development / controller |

Board-specific configuration lives in `boards/`.

---

## Building

Requires [nRF Connect SDK](https://www.nordicsemi.com/Products/Development-software/nRF-Connect-SDK) v3.1.1.

```sh
west build -b xiao_ble/nrf52840/sense -p -- \
  -DDTC_OVERLAY_FILE=boards/xiao_ble_nrf52840_sense_plus.overlay \
  -DEXTRA_CONF_FILE=boards/xiao_ble_nrf52840_sense_plus.conf
```

The device appears as a USB serial port with an interactive shell. Build instructions for the other boards and a full shell command reference are in the project's `docs/` folder.

---

## License

Built on the [Bluetooth Mesh Chat sample](https://docs.nordicsemi.com/bundle/ncs-latest/page/nrf/samples/bluetooth/mesh/chat/README.html) from the nRF Connect SDK. Files derived from that sample keep their `LicenseRef-Nordic-5-Clause` headers. New modules are marked Apache-2.0 in their headers.
