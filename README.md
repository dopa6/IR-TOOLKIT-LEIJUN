# 📦 UNIVERSAL-IR-TOOLKIT | ESP32-C3 Infrared Multi-Tool (LEIJUN OS v17.0)

This is a complete hardware and software blueprint to build a pocket-sized infrared (IR) master device. Powered by the ESP32-C3 RISC-V chip, it works as a raw signal copier, database player, wave generator, and multi-device emulator wrapped inside a clean user interface.

---

## 🛠️ Bill of Materials (BOM)

* **Microcontroller:** ESP32-C3 SuperMini (or standard ESP32-C3 Dev Module)
* **Display:** 1.44" or 1.8" TFT SPI screen (ST7735 driver)
* **IR LEDs:** 4x Transparent Infrared diodes (940nm)
* **Transistor:** 1x 2N2222 NPN transistor (acting as the power faucet)
* **Resistors:** 1x 1 kΩ (GPIO protection) | 1x 47 Ω + 1x 100 Ω (Wired in parallel for a custom ~32Ω power tuning)
* **Capacitor:** 1x 100 µF electrolytic capacitor (Mandatory to wipe out Wi-Fi/RF noise on the power line)
* **Buttons:** 2x Tactile momentary push buttons

---

## 📍 Hardware Wiring & Pinout

### 1. Core Interface (Display, Buttons & System Power)

| Component Connection | Pin Name on Component | ESP32-C3 GPIO / Power Pin | Note |
| :--- | :--- | :--- | :--- |
| **TFT ST7735** | VCC | **3.3V** | Main display power |
| **TFT ST7735** | GND | **GND** | System Ground |
| **TFT ST7735** | SCL / SCK | **GPIO 4** | SPI Clock |
| **TFT ST7735** | SDA / MOSI | **GPIO 6** | SPI Data |
| **TFT ST7735** | A0 / DC / RS | **GPIO 3** | Data / Command Select |
| **TFT ST7735** | CS | **GPIO 7** | Chip Select |
| **TFT ST7735** | RST / RES | **GPIO 10** | Reset |
| **TFT ST7735** | LED / BLK | **3.3V** | Screen Backlight (Always ON) |
| **Button SCROLL** | Pin 1 | **GPIO 2** | Menu Navigation (Internal Pullup) |
| **Button SCROLL** | Pin 2 | **GND** | Ground reference |
| **Button SELECT** | Pin 1 | **GPIO 1** | Action Validation (Internal Pullup) |
| **Button SELECT** | Pin 2 | **GND** | Ground reference |

### 2. Infrared Sub-Circuits (TX / RX)

| Component / Sub-circuit | Connection Node | Target Destination on ESP32 or Component | Purpose |
| :--- | :--- | :--- | :--- |
| **IR Receiver** | Left Pin (GND) | **GND** | Ground reference |
| **IR Receiver** | Middle Pin (VCC) | **3.3V** | Power supply |
| **IR Receiver** | Right Pin (OUT) | **GPIO 8** | IR Demodulated signal input |
| **100µF Capacitor** | Negative Pole (-) | IR Receiver **GND** Pin | Filters RF power ripples |
| **100µF Capacitor** | Positive Pole (+) | IR Receiver **VCC** Pin | Filters RF power ripples |
| **2N2222 Transistor** | Left Pin (Emitter) | **GND** | Closes the loop to ground |
| **2N2222 Transistor** | Middle Pin (Base) | **1 kΩ Resistor** ➔ **GPIO 5** | Protects pin from overcurrent |
| **2N2222 Transistor** | Right Pin (Collector) | **All 4 IR LED Cathodes (-)** | Electronic switch / Power faucet |
| **Parallel Resistor Block** | Side A | **3.3V** | Combined 47Ω + 100Ω yields ~32Ω |
| **Parallel Resistor Block** | Side B | **All 4 IR LED Anodes (+)** | Supplies safe max current to the LEDs |

---

## 🚀 Built-in Modules (v17.0)

* **IR CAPTURE [RX]:** Sniffs, decodes (NEC, Samsung, Sony, RC5, RC6) and commits hardware signals to the chip's onboard flash storage using `LittleFS` (`codes.json`).
* **IR TRANSMIT [TX]:** Replays saved codes individually or fires a heavy "Burst Mode" signal flood.
* **PWM DUTY TUNE [PWR]:** Live duty cycle adjustments (0% to 100%) through the transistor driver to swap between Stealth (low power) and Overdrive mode (max range).
* **DEVICE EMULATOR [EMU]:** Includes **12 standalone profiles** targeting common appliances (AC Units, Projectors, Smart Lights, Media Players, Soundbars, DSLR Cameras, Shutter Screens, etc.) running on optimal 38kHz and 40kHz carriers.
* **BROADCAST ALL [ALL]:** Sequentially triggers every single code pattern stored in the emulation matrix to blast signals globally across the environment.

---

## 🔧 Installation & Flashing

### Option A: Flash the Pre-compiled Binary (Fastest)
1. Download the `LEIJUN_OS_v17.0.bin` file from the root of this repository.
2. Connect your ESP32-C3 to your computer.
3. Flash it using an online tool like **ESP Web Flasher** or via command line with `esptool.py`.

### Option B: Compile from Source Code
1. Open the **Arduino IDE**.
2. Download these required dependencies via the Library Manager:
   * `Adafruit ST7735 and ST7789 Library`
   * `Adafruit GFX Library`
   * `ArduinoJson`
   * `IRremote` *(by Armin Joachimsmeyer)*
3. Set your target board to: **ESP32C3 Dev Module**.
4. **Important:** Change your flash partition scheme layout settings to allow **LittleFS** space to manage local database operations.
5. Flash the codebase onto your device.

---

## ⚖️ License & Disclaimer

This project is licensed under the **MIT License**. 

```text
THE SOFTWARE AND THE BINARY FILE ARE PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, 
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, 
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS 
OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN 
AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH 
THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
