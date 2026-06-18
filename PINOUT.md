# GPIO Pinout Reference
## Hardware: Raspberry Pi 3B + InnoMaker HiFi DAC Hat (PCM5122) + ST7735 1.8" SPI TFT

All pin numbers use BCM (Broadcom) numbering, which is what the software uses.
Physical pin numbers refer to the 40-pin GPIO header (1 = top-left with USB ports facing down).

---

## GPIO Assignments (from config.py)

| BCM | Physical | Connected To           | Role              | Notes                                      |
|-----|----------|------------------------|-------------------|--------------------------------------------|
| 4   | 7        | Select button          | Button input      |                                            |
| 7   | 26       | Screen LED             | LED output        | SPI CE1 — safe, no second SPI device       |
| 9   | 21       | Play/Pause button      | Button input      | SPI MISO — safe, display is write-only (see note below) |
| 14  | 8        | Stop button            | Button input      | UART TX — safe if serial console disabled (see note below) |
| 15  | 10       | Rewind button          | Button input      | UART RX — safe if serial console disabled (see note below) |
| 26  | 37       | Fast-forward button    | Button input      |                                            |
|     |          |                        |                   |                                            |
| 5   | 29       | Year encoder CLK       | Rotary encoder    |                                            |
| 12  | 32       | Year encoder DT        | Rotary encoder    |                                            |
| 20  | 38       | Year encoder SW        | Rotary encoder    | Push-to-click; moved from GPIO6 (InnoMaker DAC mute conflict) |
|     |          |                        |                   |                                            |
| 22  | 15       | Month encoder CLK      | Rotary encoder    |                                            |
| 16  | 36       | Month encoder DT       | Rotary encoder    |                                            |
| 23  | 16       | Month encoder SW       | Rotary encoder    | Push-to-click                              |
|     |          |                        |                   |                                            |
| 17  | 11       | Day encoder CLK        | Rotary encoder    |                                            |
| 13  | 33       | Day encoder DT         | Rotary encoder    |                                            |
| 27  | 13       | Day encoder SW         | Rotary encoder    | Push-to-click                              |

---

## InnoMaker HiFi DAC Hat (PCM5122) Reserved Pins

These pins are claimed by the InnoMaker DAC Hat and **must not be used** for anything else.

| BCM | Physical | Function       | Why It's Reserved                                   |
|-----|----------|----------------|-----------------------------------------------------|
| 2   | 3        | I2C SDA        | DAC chip control (PCM5122 communicates over I2C)    |
| 3   | 5        | I2C SCL        | DAC chip control                                    |
| 6   | 31       | DAC mute       | Active mute control — driven by DAC Hat             |
| 18  | 12       | PCM CLK        | I2S audio clock                                     |
| 19  | 35       | PCM FS         | I2S audio frame sync (word select)                  |
| 21  | 40       | PCM DOUT       | I2S audio data out                                  |
| 26  | 37       | IR (unpopulated)| Reserved on HAT but not wired — no active conflict |

Note: GPIO 2/3 are the I2C bus. The InnoMaker uses them, but I2C is a shared bus —
other I2C devices at different addresses can coexist if needed in the future.

Note: GPIO20 (physical pin 38) is NOT used by the InnoMaker DAC Hat and is free for use.

---

## ST7735 1.8" SPI TFT Display Pins

The display uses the hardware SPI bus plus two additional control lines.

| BCM | Physical | Display Pin | Adafruit Constant | Notes                              |
|-----|----------|-------------|-------------------|------------------------------------|
| 10  | 19       | MOSI (SDA)  | `board.SPI()`     | SPI data out from Pi               |
| 11  | 23       | SCLK (SCK)  | `board.SPI()`     | SPI clock                          |
| 8   | 24       | CS          | `board.CE0`       | Chip select — active low           |
| 24  | 18       | DC (RS)     | `board.D24`       | Data/Command select                |
| 25  | 22       | RST         | `board.D25`       | Reset — active low                 |
| 9   | 21       | MISO        | —                 | Not connected to display; hardware pin only (see note) |

Power: Display VCC → 3.3V (pin 17), GND → any GND pin.

---

## Notes on Dual-Use Pins

### GPIO 9 — SPI MISO (play_pause_pin)
GPIO 9 is the SPI MISO line (data back from device to Pi). The ST7735 display is
write-only and does not wire MISO, so the pin is electrically free. The SPI hardware
sets MISO as an input, which is compatible with a pull-up button. This is intentional
and safe for this build — do not add any SPI device that uses MISO without reassigning
play_pause_pin first.

### GPIO 14 & 15 — UART TX/RX (stop_pin, rewind_pin)
GPIO 14 and 15 are the Pi's mini UART (ttyS0). On the Pi 3B the hardware UART
(ttyAMA0) is used for Bluetooth, so ttyS0 is idle by default. These pins are safe to
use as regular GPIO as long as the serial console is not enabled.

To verify: run `raspi-config` → Interface Options → Serial Port.
- "Would you like a login shell to be accessible over serial?" → **No**
- "Would you like the serial port hardware to be enabled?" → **Yes** (keep hardware enabled, just no login shell)

If the serial console is currently enabled, disable it before wiring these buttons.

---

## Do Not Use — Full Reserved Pin Summary

Avoid all of the following when adding any new hardware to this build.

| BCM | Physical | Reserved By              | Reason                        |
|-----|----------|--------------------------|-------------------------------|
| 2   | 3        | InnoMaker DAC Hat        | I2C SDA (DAC control)         |
| 3   | 5        | InnoMaker DAC Hat        | I2C SCL (DAC control)         |
| 6   | 31       | InnoMaker DAC Hat        | DAC mute (active)             |
| 8   | 24       | SPI display              | Chip select (CE0)             |
| 9   | 21       | SPI display / config     | MISO / play_pause_pin         |
| 10  | 19       | SPI display              | MOSI                          |
| 11  | 23       | SPI display              | SCLK                          |
| 14  | 8        | config                   | stop_pin                      |
| 15  | 10       | config                   | rewind_pin                    |
| 18  | 12       | InnoMaker DAC Hat        | I2S PCM CLK                   |
| 19  | 35       | InnoMaker DAC Hat        | I2S PCM FS                    |
| 21  | 40       | InnoMaker DAC Hat        | I2S PCM DOUT                  |
| 24  | 18       | SPI display              | DC (Data/Command)             |
| 25  | 22       | SPI display              | RST (Reset)                   |

Also avoid GPIO 0 and 1 (physical pins 27/28) — these are the HAT ID EEPROM lines.
