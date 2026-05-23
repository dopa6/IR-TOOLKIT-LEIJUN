# 📡 UNIVERSAL-IR-TOOLKIT — LEIJUN OS `v17.0`
> Pocket-sized infrared multi-tool powered by ESP32-C3 · Raw copier · Database player · Wave generator · 12-profile device emulator

---

## 📋 Table of Contents

1. [Overview](#-overview)
2. [Features](#-features)
3. [Bill of Materials](#-bill-of-materials-bom)
4. [Hardware Wiring](#-hardware-wiring--pinout)
   - [TFT Display](#1-tft-st7735-display)
   - [Buttons](#2-buttons)
   - [IR Receiver](#3-ir-receiver)
   - [IR Transmitter (LED Driver)](#4-ir-transmitter-led-driver-circuit)
5. [Circuit Architecture](#-circuit-architecture)
6. [Software Modules](#-software-modules)
   - [IR CAPTURE \[RX\]](#1-ir-capture-rx)
   - [IR TRANSMIT \[TX\]](#2-ir-transmit-tx)
   - [IR JAMMER \[CW\]](#3-ir-jammer-cw--signal-generator)
   - [TV-B-GONE \[KILL\]](#4-tv-b-gone-kill)
   - [PWM DUTY TUNE \[PWR\]](#5-pwm-duty-tune-pwr)
   - [DEVICE EMULATOR \[EMU\]](#6-device-emulator-emu)
   - [QUICK CLONE \[RAW\]](#7-quick-clone-raw)
   - [BROADCAST ALL \[ALL\]](#8-broadcast-all-all)
7. [Device Emulator Profiles](#-device-emulator-profiles)
8. [Navigation & Controls](#-navigation--controls)
9. [Installation & Flashing](#-installation--flashing)
   - [Option A — Pre-compiled Binary](#option-a--flash-the-pre-compiled-binary-fastest)
   - [Option B — Compile from Source](#option-b--compile-from-source)
10. [Storage & Persistence](#-storage--persistence)
11. [UI & Visual Design](#-ui--visual-design)
12. [Supported IR Protocols](#-supported-ir-protocols)
13. [Firmware Changelog](#-firmware-changelog)
14. [License & Disclaimer](#-license--disclaimer)

---

## 🔍 Overview

**LEIJUN OS** is a self-contained infrared Swiss Army knife built around the **ESP32-C3** RISC-V microcontroller. It fits in your pocket and gives you full control over IR signals: capture unknown remotes, replay saved codes, brute-force TV power sequences, generate raw carrier waves, emulate 12 categories of smart devices, and broadcast a universal code matrix — all from a tiny cyberpunk-styled TFT display navigated with just **2 buttons**.

The device is designed for:
- **Reverse-engineering** IR remotes and appliances
- **Accessibility** (replacing lost remotes)
- **Testing** IR receiver hardware and circuits
- **Automation** research and home lab experimentation

---

## ✨ Features

| Feature | Description |
|---|---|
| 📥 **IR Capture** | Sniff and save up to 10 IR codes with protocol auto-detection |
| 📤 **IR Transmit** | Replay individual codes or flood with Burst Mode (×15) |
| 📡 **Signal Generator** | Output a clean CW carrier from 30 kHz to 56 kHz |
| 📺 **TV-B-GONE** | Brute-force power-off across EU and NA TV databases |
| 🎛️ **PWM Duty Tuner** | Adjust TX power from 10% (stealth) to 100% (overdrive) |
| 🎭 **Device Emulator** | 12 device profiles, 100+ codes across common appliances |
| ⚡ **Quick Clone** | Capture and instantly replay a single code (no save needed) |
| 🌐 **Broadcast All** | Sequentially fires 40 universal codes across 5 protocols |
| 💾 **Persistent Storage** | All captured codes and power settings survive reboots via LittleFS |
| 🎨 **Cyberpunk UI** | Scrollable menus, animated radar, live counters, progress bars |

---

## 🧰 Bill of Materials (BOM)

| # | Component | Specification | Qty | Notes |
|---|---|---|---|---|
| 1 | **Microcontroller** | ESP32-C3 SuperMini or Dev Module | 1 | RISC-V, Wi-Fi/BT capable |
| 2 | **TFT Display** | 1.44" or 1.8" SPI (ST7735 driver) | 1 | ST7735R / B tab variants |
| 3 | **IR LEDs** | 940 nm transparent infrared diodes | 4 | Wired in parallel |
| 4 | **NPN Transistor** | 2N2222 (TO-92 package) | 1 | Acts as electronic switch |
| 5 | **Resistor** | 1 kΩ ¼W | 1 | Base current limiter (GPIO protection) |
| 6 | **Resistor** | 47 Ω ¼W | 1 | Wired in parallel with 100 Ω → ~32 Ω |
| 7 | **Resistor** | 100 Ω ¼W | 1 | Wired in parallel with 47 Ω → ~32 Ω |
| 8 | **Capacitor** | 100 µF electrolytic | 1 | Decouples RF noise on IR receiver VCC |
| 9 | **Buttons** | Tactile momentary push buttons | 2 | SCROLL + SELECT |

> **💡 Parallel resistor math:** `47 Ω ∥ 100 Ω = (47×100)/(47+100) ≈ 32 Ω` — this limits LED current to a safe level while maximising range from the 3.3 V rail.

---

## 📍 Hardware Wiring & Pinout

### 1. TFT ST7735 Display

| TFT Pin | ESP32-C3 GPIO | Function |
|---|---|---|
| VCC | 3.3V | Power |
| GND | GND | Ground |
| SCL / SCK | **GPIO 4** | SPI Clock |
| SDA / MOSI | **GPIO 6** | SPI Data |
| A0 / DC / RS | **GPIO 3** | Data / Command Select |
| CS | **GPIO 7** | Chip Select |
| RST / RES | **GPIO 10** | Hardware Reset |
| LED / BLK | 3.3V | Backlight (always ON) |

### 2. Buttons

| Button | Pin 1 | Pin 2 | Mode |
|---|---|---|---|
| **SCROLL** | GPIO 2 | GND | Internal Pull-up, Active LOW |
| **SELECT** | GPIO 1 | GND | Internal Pull-up, Active LOW |

> Both buttons are read with `INPUT_PULLUP`. A press drives the pin LOW. Debounce is enforced in software at **180 ms**.

### 3. IR Receiver

| Receiver Pin | Connection | Note |
|---|---|---|
| Left (GND) | GND | Ground |
| Middle (VCC) | 3.3V | Power (with 100 µF cap in parallel) |
| Right (OUT) | **GPIO 8** | Demodulated digital signal output |

> ⚠️ The **100 µF capacitor** must be placed directly across the VCC and GND pins of the IR receiver module to filter RF noise injected by the ESP32's Wi-Fi/BT radio. Without it, spurious decodes may occur.

### 4. IR Transmitter LED Driver Circuit

```
3.3V ──[47Ω ∥ 100Ω ≈ 32Ω]──┬── Anode (+) LED 1
                              ├── Anode (+) LED 2
                              ├── Anode (+) LED 3
                              └── Anode (+) LED 4
                                         │
                              Cathode (−) of all 4 LEDs
                                         │
                            Collector (C) of 2N2222
                            
GPIO 5 ──[1kΩ]── Base (B) of 2N2222

Emitter (E) of 2N2222 ── GND
```

| 2N2222 Pin | Connection | Role |
|---|---|---|
| Emitter (E) | GND | Current return path |
| Base (B) | 1 kΩ → **GPIO 5** | PWM control signal (protected) |
| Collector (C) | All 4 LED Cathodes | Electronic switch |

> The 1 kΩ resistor on the base protects GPIO 5 from the transistor's base current. GPIO 5 outputs a PWM signal; the 2N2222 amplifies it to drive all 4 LEDs simultaneously.

---

## 🔬 Circuit Architecture

```
┌─────────────────────────────────────────────────────┐
│                    ESP32-C3                          │
│                                                      │
│  GPIO 1 ←── [SELECT Button] ──── GND               │
│  GPIO 2 ←── [SCROLL Button] ──── GND               │
│                                                      │
│  GPIO 8 ←── [IR Receiver OUT]                       │
│                                                      │
│  GPIO 5 ──→ [1kΩ] ──→ [2N2222 Base]                │
│                           │                          │
│  GPIO 4 ──→ TFT SCK       Collector ──→ LED Array   │
│  GPIO 6 ──→ TFT MOSI      Emitter  ──→ GND         │
│  GPIO 3 ──→ TFT DC                                  │
│  GPIO 7 ──→ TFT CS        [47Ω∥100Ω] ─→ LED Anodes │
│  GPIO 10 ──→ TFT RST       3.3V ──────┘             │
│                                                      │
│  3.3V ──→ TFT VCC, TFT BLK                          │
│  3.3V ──→ IR Receiver VCC + [100µF cap]             │
└─────────────────────────────────────────────────────┘
```

---

## 🚀 Software Modules

### 1. IR CAPTURE `[RX]`

Activates the IR receiver on **GPIO 8** and listens for incoming signals. Once a valid frame is detected, the code is automatically decoded and saved to `codes.json` in LittleFS flash storage.

- **Protocols decoded:** NEC, Samsung, Sony, RC5, RC6
- **Storage:** Up to **10 codes** in flash (`codes.json` via ArduinoJson)
- **Duplicate detection:** Ignores codes already present in the database
- **Animation:** Animated radar rings confirm active listening
- **Exit:** Press SELECT to return to the main menu

### 2. IR TRANSMIT `[TX]`

Browse the saved code database and transmit codes to your target device.

- **Normal send:** Press SELECT once → sends the selected code once
- **Burst Mode:** Hold SELECT for **1.8 seconds** → fires the code **15 times** in rapid succession (~150 ms interval) for long-range or stubborn targets
- **Delete a code:** Hold SCROLL + SELECT simultaneously for **500 ms** to permanently remove a code from storage
- **Navigation:** Short press SCROLL cycles through saved codes; hold SCROLL **500 ms** to go back to the menu

### 3. IR JAMMER `[CW]` — Signal Generator

Outputs a **raw continuous-wave (CW) carrier** on the IR LED array. Useful for saturating IR receivers, testing sensitivity, or generating reference signals.

| Frequency | Use Case |
|---|---|
| **30 kHz** | Industrial low-band sensors |
| **33 kHz** | Legacy heritage remotes |
| **36 kHz** | Standard alternative carrier |
| **38 kHz** | Universal IR standard (most common) |
| **40 kHz** | High sensitivity receivers |
| **56 kHz** | Bang & Olufsen proprietary |

- TX power uses the **current PWM Duty** setting from the Power Tweaker
- Press SELECT to start transmission, press any button to stop

### 4. TV-B-GONE `[KILL]`

A targeted power-off payload that cycles through a database of common TV power codes across multiple protocols.

**Step 1 — Select region:**
- `EUROPE [EU]` — 16 codes tuned for PAL/DVB market brands
- `NORTH AMERICA [NA]` — 16 codes tuned for NTSC market brands

**Step 2 — Select repeat mode:**
- `×1 PRECISE` — each code sent once (fast, clean sweep)
- `×3 CLASSIC` — each code sent 3 times per protocol (higher success rate)

**Each code is sent across 5 protocols per iteration:** NEC → Samsung → Sony → RC5 → RC6

> For each of the 16 target codes, the system fires up to 5 protocol variants × up to 3 repeats = up to **240 IR frames total** in a single payload run.

### 5. PWM DUTY TUNE `[PWR]`

Adjusts the PWM duty cycle of the 2N2222 transistor base drive, effectively controlling the **peak current and range** of the IR LED array.

| Level | Duty | Mode |
|---|---|---|
| 1–2 | 10–20% | STEALTH (short range, low visibility) |
| 3–5 | 30–50% | STANDARD (normal operation) |
| 6–9 | 60–90% | OVERDRIVE (maximum range) |

- Setting is **persisted to flash** (`tweaker.json`) via a long-press of SCROLL
- Applied live: all TX modes (Burst, Emulator, Matrix, Signal Gen) respect the current duty value
- `ledcWrite` value is displayed in real time on-screen

### 6. DEVICE EMULATOR `[EMU]`

Continuously streams IR codes from a selected device profile, cycling through all codes in the pool at **100 ms intervals**. Runs until SCROLL is pressed.

- **12 scrollable profiles** in a dedicated sub-menu
- Carrier frequency auto-selected per profile (38 kHz or 40 kHz)
- Frame counter displayed live on-screen
- Current code hex value and pool progress bar shown in real time

### 7. QUICK CLONE `[RAW]`

Capture a single IR code and replay it immediately — without saving to the database. Perfect for one-shot cloning tasks.

1. Point your remote at the receiver → code is captured and displayed
2. Press SELECT to retransmit the cloned code instantly
3. Short press SCROLL to discard and capture again
4. Long press SCROLL (500 ms) to return to the main menu

### 8. BROADCAST ALL `[ALL]` — Matrix Cycler

Fires a hardcoded matrix of **40 universal IR codes** across 5 protocols in a continuous loop at 80 ms intervals. Covers the widest possible device range in a single blast.

**Code categories in the matrix:**
- TVs (NEC + Samsung), Sony (SIRC), RC5/RC6 commands
- AC units (Samsung + NEC), Projectors, Smart Lights
- TV-B-Gone payload codes, Gate/garage receivers

> Live scrolling "flux" display shows each transmitted code with protocol color-coding as they fire.

---

## 🎭 Device Emulator Profiles

| # | Profile | Target Brands | Carrier | Codes |
|---|---|---|---|---|
| 1 | **PROJECTORS** | Epson, BenQ, Optoma | 38 kHz | 12 |
| 2 | **CLIMATE / AC** | Daikin, LG, Mitsubishi | 38 kHz | 10 |
| 3 | **SMART LIGHTS** | MagicHome, Tuya RGB | 38 kHz | 16 |
| 4 | **GATES / GARAGE** | Generic gate receivers | 40 kHz | 8 |
| 5 | **MEDIA PLAYERS** | Apple TV, Roku, FireTV | 38 kHz | 14 |
| 6 | **AUDIO / FANS** | Bose, JBL, Sony, Dyson | 38 kHz | 12 |
| 7 | **DSLR CAMERAS** | Canon, Nikon, Sony, Pentax | 40 kHz | 8 |
| 8 | **MOTOR. SCREENS** | Somfy, Elite Screens | 38 kHz | 9 |
| 9 | **BARCODE SCAN.** | Honeywell, Zebra | 38 kHz | 8 |
| 10 | **ROBOT VACUUMS** | Roomba, Roborock, Xiaomi | 38 kHz | 12 |
| 11 | **SMART PLUGS** | Tuya, Belkin Wemo | 38 kHz | 8 |
| 12 | **SPACE HEATERS** | Dyson, DeLonghi | 38 kHz | 10 |

**Total emulator codes: 127 across 12 profiles**

---

## 🕹️ Navigation & Controls

The entire device is controlled with **2 buttons only:**

| Button | Action | Effect |
|---|---|---|
| **SCROLL** | Short press (release < 500 ms) | Next item / Next option |
| **SCROLL** | Hold ≥ 500 ms | Back to previous screen / Save setting |
| **SCROLL** | Hold ≥ 600 ms (in main menu) | Scroll UP (reverse direction) |
| **SELECT** | Short press | Confirm / Send code / Start action |
| **SELECT** | Hold ≥ 1.8 s (TX mode) | Activate Burst Mode (×15 flood) |
| **SELECT + SCROLL** | Hold simultaneously ≥ 500 ms | Delete current code (TX mode) |

**Main menu navigation:**
- Short SCROLL press → move cursor **down**
- Long SCROLL press (600 ms) → move cursor **up**
- The menu shows **4 items at a time** and scrolls with `^` / `v` indicators

---

## 🔧 Installation & Flashing

### Option A — Flash the Pre-compiled Binary (Fastest)

1. Download `LEIJUN_OS.ino.merged.bin` from the root of this repository
2. Connect your ESP32-C3 via USB
3. Flash using one of these methods:

**Via browser (no install required):**
- Go to [ESP Web Flasher](https://esp.huhn.me/) or [Espressif Web Flash](https://espressif.github.io/esptool-js/)
- Select your COM port and flash the `.bin` file at address `0x0`

**Via command line:**
```bash
pip install esptool
esptool.py --chip esp32c3 --port /dev/ttyUSB0 --baud 460800 \
  write_flash -z 0x0 LEIJUN_OS.ino.merged.bin
```

### Option B — Compile from Source

**1. Install Arduino IDE** (v2.x recommended)

**2. Add ESP32 board support:**
- Go to `File → Preferences → Additional Board Manager URLs`
- Add: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
- Install `esp32 by Espressif Systems` from the Board Manager

**3. Install required libraries** via `Sketch → Include Library → Manage Libraries`:

| Library | Author | Purpose |
|---|---|---|
| `Adafruit ST7735 and ST7789 Library` | Adafruit | TFT display driver |
| `Adafruit GFX Library` | Adafruit | Graphics primitives |
| `ArduinoJson` | Bblanchon | JSON for flash storage |
| `IRremote` | Armin Joachimsmeyer | IR send/receive engine |

**4. Configure the board:**
- Board: `ESP32C3 Dev Module`
- USB CDC On Boot: `Enabled` (for Serial monitor)
- Partition Scheme: **`Minimal SPIFFS (1.9MB APP with OTA/190KB SPIFFS)`** or any scheme that allocates LittleFS space

> ⚠️ **Critical:** You **must** change the partition scheme to one that includes LittleFS/SPIFFS space. Without this, `codes.json` and `tweaker.json` cannot be written and all saves will silently fail.

**5. Flash:**
- Select your COM port and click Upload

---

## 💾 Storage & Persistence

The firmware uses **LittleFS** on the ESP32-C3's internal flash to store two files:

| File | Contents | Max Size |
|---|---|---|
| `/codes.json` | Up to 10 captured IR codes (protocol, value, bit count) | ~2 KB |
| `/tweaker.json` | Current PWM duty step index (0–8) | ~64 B |

Both files are loaded automatically on boot. Codes survive power cycles, firmware re-flashes that don't erase flash, and resets.

**JSON schema — `codes.json`:**
```json
{
  "codes": [
    { "proto": 3, "val": 551489775, "bits": 32 },
    { "proto": 2, "val": 46272,     "bits": 12 }
  ]
}
```

**JSON schema — `tweaker.json`:**
```json
{ "step": 4 }
```

> Protocol integer mapping: `NEC=3`, `SONY=2`, `RC5=4`, `RC6=5`, `SAMSUNG=6`

---

## 🎨 UI & Visual Design

LEIJUN OS uses a custom **dark palette** rendered on the ST7735 TFT:

| Color Name | Hex (RGB565) | Used For |
|---|---|---|
| `C_BG` | `#000000` | Background |
| `C_SURFACE` | Dark grey | Inactive menu items |
| `C_ACCENT` | Cyan-blue | Active highlights, borders |
| `C_ACCENT2` | Bright cyan | Text values, code display |
| `C_SUCCESS` | Green | Confirmed actions, saved |
| `C_ALERT` | Red | Burst mode, warnings |
| `C_ORANGE` | Orange | Matrix flux (Samsung proto) |
| `C_MUTED` | Mid-grey | Help text, labels |

**UI features:**
- Animated radar rings during IR receive modes
- Progress bars for burst transmissions and emulator sweeps
- Scrolling flux log in Matrix Cycler mode (real-time protocol feed)
- LED blink indicator (top-right corner) during active TX loops
- Footer slot counter (`n/10`) with color shift when full

---

## 📡 Supported IR Protocols

| Protocol | Standard | Typical Brands |
|---|---|---|
| **NEC** | 38 kHz, 32-bit | LG, Samsung (some), Philips, generic |
| **Samsung** | 38 kHz, 32-bit | Samsung TVs, monitors, appliances |
| **Sony SIRC** | 40 kHz, 12/15/20-bit | Sony all product lines |
| **RC5** | 36 kHz, 12-bit | Philips legacy, Cambridge Audio |
| **RC6** | 36 kHz, 20-bit | Philips, Windows MCE, Sky |

> Unknown protocols are rejected during capture (minimum 8-bit threshold) to avoid saving noise.

---

## 📝 Firmware Changelog

### v17.0 (current)
- **FEAT:** Device Emulator expanded from 4 to **12 profiles**
- **NEW profiles added:**
  - MEDIA PLAYERS (Apple TV, Roku, FireTV)
  - AUDIO / FANS (Bose, JBL, Sony soundbars, Dyson fan)
  - DSLR CAMERAS (Canon RC-6, Nikon ML-L3, Sony RMT-DSLR2, Pentax CS-205)
  - MOTORIZED SCREENS (Somfy RTS, Elite Screens)
  - BARCODE SCANNERS (Honeywell Xenon, Zebra DS)
  - ROBOT VACUUMS (iRobot Roomba, Roborock, Xiaomi Mi Robot)
  - SMART PLUGS (Tuya IR, Belkin Wemo IR variant)
  - SPACE HEATERS (Dyson Hot+Cool, DeLonghi)
- **KEEP:** All fixes from v16.0 / v15.0 / v14.0 / v13.0

### v16.0
- EMU sub-menu made scrollable (4 visible items, `^`/`v` indicators)
- Anti-flicker partial redraw for EMU navigation

### v15.0
- QUICK SLOT `[RAW]` module added
- Animated radar shared between RX and Quick Slot

### v14.0
- MATRIX CYCLER `[ALL]` module added (40 codes, 5 protocols)
- Live flux scrolling display

### v13.0
- TV-B-GONE two-step wizard (region + repeat mode)
- EU / NA pool separation

---

## ⚖️ License & Disclaimer

This project is licensed under the **MIT License**.

```
THE SOFTWARE AND THE BINARY FILE ARE PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS
OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN
AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH
THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

> **⚠️ Legal notice:** This tool is intended for personal use, home automation, accessibility, and hardware research on devices you own. Interfering with IR systems in public spaces, commercial venues, or on devices you do not own may violate local laws. The authors accept no responsibility for misuse.

---

*LEIJUN OS v17.0 · ESP32-C3 RISC-V · Built with ❤️ and infrared*
