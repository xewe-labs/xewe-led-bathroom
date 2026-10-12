> Archived. Part of the XeWe LED family; see https://github.com/xewe-labs/xewe-led-os.

# XeWe LED - Bathroom — Alexa and HomeKit lights around a bathroom mirror

Personal project (XeWe smart-home lights) · 2024-03 · Solo: Max Dokukin · Status: Completed

![Bathroom mirror framed by the LED strip, lit red](static/media/thumbnails/full_size.webp)

## Overview

Smarthome Lights ESP for the bathroom: ESP8266 firmware that connects an addressable LED strip around a bathroom mirror to the
Apple HomeKit and Amazon Alexa smart-home systems. The LEDs have two modes: solid colour, and Perlin fade (slow noise-driven waves
of neighbouring hues). The sketch was adapted on 2024-03-01 from the two-strip desk and ceiling version of the same firmware
(Smarthome-Lights-ESP, now [xewe-labs/led-desk](https://github.com/xewe-labs/led-desk)) to drive a single 181-LED strip.
Colour, brightness, mode and on/off state are stored in EEPROM, so the lights come back as they were after a power cut.

## Highlights

- One firmware, two voice ecosystems: the strip appears in Alexa as 2 devices and in the Home app as a 3-accessory bridge, and after any change both are re-synced from one shared state (`Alexa.h`, `HomeKit.h`, `my_accessory.c`)
- Perlin fade mode: FastLED `inoise8` noise sampled per LED and mapped onto a 10000-wide slice of the 65535 NeoPixel hue wheel, redrawn every 10 ms (`PerlinFade.h`)
- Smooth changes everywhere: 900 ms cross-fades for colour, brightness and the Perlin base hue (`LedController.h`, `PerlinFade.h`)
- State survives reboots: 6 bytes (R, G, B, brightness, mode, on/off) in EEPROM at offset 1500 (`MemoryController.h`)
- About 900 lines of C/C++ in 9 files; single `LedController` class shared with the desk/ceiling version

## How it works

```
Alexa app / Echo ──(Espalexa, local WiFi)──┐
                                           ├→ LedController (state, colour, brightness, EEPROM) → frame() every 10 ms → Adafruit NeoPixel → 181-LED strip
Home app / Siri ──(HomeKit accessory server)┘        ↑                                                        (solid | colour transition | Perlin fade)
                                   valuesUpdated → syncValuesHomekit() + syncValuesAlexa()
```

- **`Smarthome-Lights-ESP-Bathroom.ino`** — setup (serial 115200, EEPROM 4096 B, WiFi, controller, HomeKit, Alexa) and the main loop: HomeKit loop → Alexa loop → one LED frame → sync both assistants if anything changed → 10 ms delay.
- **`LedController.h`** — the light: on/off, RGB and HSV colour, brightness 0–100 mapped to 0–255, mode switching, 900 ms colour and brightness transitions, persistence of every change.
- **`PerlinFade.h`** — the fade animation: per-LED noise value → hue offset, saturation (245–255) and brightness (100–255); the hue range narrows to 20% near red so the waves do not wrap into other colours.
- **`Color.h`** — RGB ↔ HSV conversion, colour mixing and channel helpers.
- **`MemoryController.h`** — byte-level EEPROM read/write with an optional debug trace.
- **`Alexa.h`** — Espalexa devices "Bathroom Lights" (colour) and "Bathroom Lights Fade" (on/off → Perlin/solid).
- **`HomeKit.h`, `my_accessory.c`** — HomeKit bridge "XeWe Lights Bath" with a lightbulb (on, brightness, hue, saturation) and a "Bathroom Lights Fade" switch.
- **`enums.h`** — modes (`SOLID_COLOR`, `PERLIN`, `SOLID_COLOR_TRANSITION`) and states (`OFF`, `ON`).

## Results

| Metric | Value | Baseline / note |
|---|---|---|
| LEDs driven | 181 on pin D6 (GRB, 800 kHz) | desk + ceiling version: 61 + 151 on two pins |
| Voice / app integrations | Alexa (2 devices), Apple HomeKit (3 accessories) | both kept in sync from one state |
| Light modes | solid colour, Perlin fade | plus an internal colour-transition state |
| Transition time | 900 ms | colour, brightness, fade hue |
| Frame period | 10 ms | `LedController.h`, `PerlinFade.h` |
| Persisted state | 6 bytes in EEPROM | restored at boot |

The firmware is a home installation rather than an experiment, so the results are its build facts; no power or latency
measurements were recorded.

## Getting started

Arduino IDE with the ESP8266 board package and these libraries:

- Adafruit NeoPixel
- FastLED (only `inoise8` is used)
- Espalexa
- Arduino-HomeKit-ESP8266 (`arduino_homekit_server.h`)

```text
1. Provide wifi_info.h next to the sketch with a wifi_connect() function that joins your network
   (keep your own credentials out of version control).
2. Adjust LED_PIN and LED_NUM in Smarthome-Lights-ESP-Bathroom.ino for your strip.
3. Open Smarthome-Lights-ESP-Bathroom.ino, select your ESP8266 board and upload.
4. Alexa: "discover devices" finds "Bathroom Lights" and "Bathroom Lights Fade".
5. Home app: add accessory → "XeWe Lights Bath", using the setup code in my_accessory.c.
```

Serial output at 115200 baud logs every setter call (`LED_CONRTROLLER_DEBUG` in `LedController.h`). Open items are in
[todo.md](todo.md) (a memory leak in the HomeKit client write path).

## Documents

- Photos: [whole mirror](static/media/thumbnails/full_size.webp), [strip along the mirror edge](static/media/resources/IMG_5858.webp), [strip and power supply](static/media/resources/IMG_5526.webp)
- [todo.md](todo.md)
- Image from the original Smarthome-Lights-ESP README: [IMG_1931](https://github.com/xeweva/Smarthome-Lights-ESP/assets/54597813/85e34b70-14e2-4198-ad93-fff34ed6da69)
- Predecessor (desk and ceiling lights): [xewe-labs/led-desk](https://github.com/xewe-labs/led-desk)
- Related: XeWe LED OS, modular ESP32 firmware for addressable LEDs with HomeKit, Alexa and persistent settings — [project page](https://maxdokukin.com/projects/xewe-led-os) · [github.com/xewe-labs/xewe-led-os](https://github.com/xewe-labs/xewe-led-os)
- Project page: [maxdokukin.com/projects/xewe-led-bathroom](https://maxdokukin.com/projects/xewe-led-bathroom)
