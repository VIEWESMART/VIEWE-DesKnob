<h1 align="center">VIEWE 2.1" 360×360 ESP32-S31 Smart Knob Display Quick Guide</h1>

* **[中文版](./README_CN.md)**

<p align="center">
    <img src="images/Product_pic0.png" alt="SFG-VIEWE-DesKnob product appearance">
</p>

> [!NOTE]
> Product photos and diagram assets (`images/`) are not in this repository yet. Paths below match the planned layout; add files under `images/` when available.

---

## 1. Introduction

SFG-VIEWE-DesKnob (GitHub: **VIEWE-DesKnob**) is a high-performance AI smart knob display module in the VIEWE housed finished-goods series (**SFG** prefix). It is based on the Espressif **ESP32-S31NRV16** SoC (QFN80, 16 MB in-package 1.8 V Octal PSRAM; on-board external **W25Q128** Flash, 16 MB) and a **2.1-inch** round capacitive touch panel (resolution **360×360**, display driver IC **GC9B72NA**, touch IC **CST826**, host-side **QSPI** interface). Hardware uses a **dual-PCB** architecture (**Knob Module Core Board S31v1** + **Knob Module Dock Board S31v1**) linked by board-to-board **FPC2**. Core board and dock board together form a complete module.

The product integrates rotation detection (Alps **SSCM** two-way detector switches, net names **EC1** / **EC2**), a **12-LED** WS2812-compatible ring, **ES8311** audio codec + **NS4150B** amplifier + analog microphone, **Type-C** power/download (native USB, **no CH340**), **TP4055** Li-ion charging, and **MicroSD** expansion. The schematic includes **DRV2605L** haptics; it is **currently not fitted / not used**. Customers can fabricate base plates to extend the product. Typical uses include smart-home central knobs, voice-assistant front ends, and lightweight HMI.

> [!NOTE]
> **ESP32-S31** is currently a **preview** device. ESP-IDF builds must use the **`--preview`** option (for example: `idf.py --preview set-target esp32s31`). Module electrical and RF ratings follow the Espressif ESP32-S31 series module datasheet (pre-release). Numbered stable ESP-IDF releases (such as 5.4 or 5.5) do not include this chip; install ESP-IDF **master**. New designs that need Wi-Fi 6, native USB, and on-board codec audio should use this S31 knob solution.

### 1.1 Product Features

**CPU / Memory / Wireless / Security:**

- **Processor (ESP32-S31NRV16)**
  - SoC architecture: high-performance HP system + low-power LP system.
  - HP: 32-bit dual-core RISC-V, up to 320 MHz; single-precision FPU on each core; SIMD on one core; dual-core CoreMark® @320 MHz: 2195.20 (6.86/MHz).
  - LP: 32-bit single-core RISC-V, up to 40 MHz, 2-stage pipeline; remains alive in Deep-sleep.
  - Package: QFN80 (8×8 mm); chip ambient −40 ~ 85 °C (N = normal temperature grade).
  - Board antenna: schematic includes on-board antenna (CA-C03) and optional IPEX external antenna connector.
- **Memory**
  - On-chip: 320 KB ROM, 512 KB HP SRAM, 32 KB LP SRAM.
  - In-package PSRAM: **16 MB**, 1.8 V Octal SPI (encapsulated; pins not brought out). This product uses **NRV16** — do not confuse with ESP32-S31NRV32 (32 MB).
  - On-board external Flash: **W25Q128JVPIQ** (128 Mbit = **16 MB**). Flash/PSRAM supplied in the **1.8 V** domain (schematic: R4 10K to GND → 1.8 V flash; `VDD_PSRAM_1P8_*` present). W25Q128 is external Flash, not PSRAM.
  - Parallel access to flash and PSRAM supported; external flash SPI/Dual/Quad/QPI (chip capability up to 256 MB).
- **Wireless**
  - 2.4 GHz Wi-Fi 6 (IEEE 802.11ax, 20 MHz-only non-AP; also b/g/n), 1T1R.
  - Bluetooth® 5.4 (LE) + Bluetooth® Classic.
  - Zigbee 3.0 and Thread 1.4 (IEEE 802.15.4).
- **Security (chip-level summary)**
  - Secure Boot; off-chip memory encryption/decryption (XTS-AES for Flash/PSRAM).
  - Crypto accelerators: AES / SHA / RSA / ECC / HMAC, digital-signature peripherals, TRNG, key manager.
  - APM and TEE, among others (see official Espressif datasheet).
- **Peripheral interfaces ([Dock] + [Core])**
  - [Core] MicroSD (TF-018), **4-bit SDMMC**
  - [Core] QSPI display (FPC1)
  - [Core] Rotation detection: 2 bidirectional detector switches (not EC11 incremental encoders); **EC1** on GPIO2/3; **EC2** via FPC2
  - [Core + Dock] Board-to-board **FPC2** (customer expansion when using core alone)
  - [Dock] USB-C: 5 V power, firmware download and debug; USB 2.0 High-Speed OTG + USB Serial/JTAG, **no CH340**
  - [Dock] TP4055 single-cell Li-ion charging, 2.2 kΩ PROG resistor, charge current **≈455 mA** (typ. **≈470 mA** in §2.9 table)
  - [Dock] Side slide power switch + BOOT download key
  - [Dock] 12× RGB LED ring (WS2812-compatible)
  - [Dock] ES8311 + NS4150B + ZTS6216 analog MIC + speaker connector **CN2**

**Display:**

- Size: 2.1 Inch (round)
- Resolution: 360 × 360
- Display module P/N: **ALL-UE021HV-RU39-A021A**
- Interface: **QSPI** (as wired on the core board; not parallel RGB)
- Driver IC: GC9B72NA
- Touch IC: CST826 (multi-touch capacitive, G+F CTP)
- Brightness: **300 (min) / 350 (typ) cd/m²**
- Logic supply: 3.3 V

**Other:**

- Operation temperature: −20 ~ 70 °C (limited by the LCD)
- Storage temperature: −30 ~ 80 °C
- MCU module rating: −40 ~ 85 °C

### 1.2 Applications

With wireless connectivity, a round touch display, rotary interaction, and local audio capability, VIEWE-DesKnob is suitable for:

- Smart-home central control knobs / scene panels
- Controllers for speakers, lighting, HVAC, and other appliances
- Conference-table / desktop office controllers
- Industrial lightweight HMI / parameter knobs
- AI voice-assistant front ends (local MIC + speaker)
- Maker, education, and prototyping platforms

### 1.3 Product Naming

Official product name: **SFG-VIEWE-DesKnob**

| Field | Code | Meaning |
| --- | --- | --- |
| Form | SFG | Semi-Finished Goods |
| Brand | VIEWE | VIEWE brand |
| Structural features | DesKnob | Desk + Knob, desktop knob device |

---

## 2. Product information

### 2.1 Interface Description

![Interface layout](images/Interface_Layout_en.png)

This product is a cylindrical smart knob: a 2.1" round touch glass on top, a translucent LED ring in the middle, and Type-C, a slide power switch, microphone opening, and speaker opening on the side. Internal stack (rear to front): Rear housing → Li-ion battery & speaker → Dock board → light guide ring → support bracket → rotating bearing → metal housing → core board → 2.1" round touch screen (see §4).

1. **Main control:** ESP32-S31NRV16 (HP dual-core RISC-V up to 320 MHz + LP single-core), 16 MB in-package PSRAM + on-board W25Q128 (16 MB) Flash, on-board / IPEX antenna.
2. **Display & touch (FPC1):** 40-pin **FPC-05F-40PH20**, QSPI LCD + I2C touch (`TP_SCL` / `TP_SDA` / `TP_RST` / `TP_INT`); backlight **LCD_BLK** via AO3400A (Q2). See display and touch tables below.
3. **SD card slot:** External storage on the core board; **4-bit SDMMC** (see §2.8).
4. **Rotation EC1:** Core-board two-way detector switch, **GPIO2 / GPIO3**, 10 kΩ pull-ups (see §2.7).
5. **Rotation EC2:** Second detector path via **FPC2** (see §2.5).
6. **External antenna:** On-board antenna; optional IPEX.
7. **Positioning post** (mechanical).
8. **FPC2:** Core ↔ dock interconnection and customer expansion interface (see §2.5).
9–11. **[Dock] Audio:** ES8311 + NS4150B + MIC (ZTS6216) + speaker **CN2** (1.25-2P). See §2.4.
12. **[Dock] Battery CN1:** 1.25-2P, single-cell Li-ion, **3.7 V**, **750 mAh**. See §2.8.
13. **[Dock] Power slide switch SW3:** MST-12D18G4, controls **VCC_OUT** (see §2.7).
14. **[Dock] USB Type-C:** 5 V power and native USB download/debug (see §2.6).
15. **[Dock] BOOT key:** Dock **SW1** (GT-TC018B-H035-L1), pulls **BOOTMODE (GPIO61)** low for download mode (see §2.7). *Not* the core-board SSCM switch also labeled SW1.
16. **[Dock] RGB LED ring:** 12× XL-1010RGBC-2812B, data net **RGB_DATA (GPIO61)** (see §2.7).

#### Dual-board architecture

| Board | Silkscreen / schematic name | Main functions |
| --- | --- | --- |
| Core board | Knob Module Core Board S31v1 | MCU, display/touch FPC, rotation EC1 (and EC2 breakout), MicroSD, 3V3 buck, FPC2 |
| Dock board | Knob Module Dock Board S31v1 | Type-C, charging, power switch, audio, LED ring, BOOT, FPC2 |

#### Display interface (FPC1, core-board QSPI wiring)

Connector: **FPC-05F-40PH20** (40-pin). Signals brought out on the core-board schematic (do not copy parallel DB pin definitions from the panel mechanical drawing).

| Pin No. | Symbol | I/O | Description |
| --- | --- | --- | --- |
| 2 | TP_RST | I/O | Touch reset → GPIO8 |
| 3 | TP_SCL | I/O | Touch I2C SCL → GPIO9 (shared with HAPTIC nets on schematic) |
| 4 | TP_SDA | I/O | Touch I2C SDA → GPIO10 (shared with HAPTIC nets on schematic) |
| 5 | TP_INT | I | Touch interrupt → GPIO11 |
| 16 | LCD_RST | O | LCD reset → GPIO12 |
| 26 | LCD_QSPI_D3 | I/O | QSPI data 3 → GPIO13 |
| 27 | LCD_QSPI_D2 | I/O | QSPI data 2 → GPIO14 |
| 28 | LCD_QSPI_D1 | I/O | QSPI data 1 → GPIO15 |
| 29 | LCD_QSPI_D0 | I/O | QSPI data 0 → GPIO16 |
| 32 | LCD_QSPI_CS | O | QSPI chip select → GPIO17 |
| 33 | LCD_QSPI_SCL | O | QSPI clock → GPIO18 |
| 35 | LCD_TE | I | Tearing effect → GPIO19 |
| 38 | LEDK | P | Backlight cathode (via R30 = 10 Ω + Q2 AO3400A to GND) |
| 39 | LEDA | P | Backlight anode (to 3V3) |
| Other | GND/3V3/NC | — | Per display module and core-board copper |

*I: input; O: output; P: power.* Backlight enable **LCD_BLK = GPIO7** (active high; gate has 10 kΩ pull-down).

#### Touch interface summary

| Signal | GPIO | Description |
| --- | --- | --- |
| TP_RST | GPIO8 | Touch reset |
| TP_SCL | GPIO9 | I2C SCL, 2.2 kΩ pull-up to 3V3 |
| TP_SDA | GPIO10 | I2C SDA, 2.2 kΩ pull-up to 3V3 |
| TP_INT | GPIO11 | Touch interrupt |
| CST826 | — | I2C address **0x15** (per product specification) |

### 2.2 GPIO Definition

![GPIO definition](images/GPIO_Definition.png)

| GPIO | Current usage | Function description |
| --- | --- | --- |
| GPIO0–1 | Reserved | Not assigned |
| GPIO2 | EC1_B | Rotation detection channel B (core board) |
| GPIO3 | EC1_A | Rotation detection channel A (core board) |
| GPIO4–6 | Reserved | Not assigned |
| GPIO7 | LCD_BLK | Backlight enable (AO3400A gate), active high |
| GPIO8 | TP_RST | Touch reset |
| GPIO9 | TP_SCL | I2C clock (touch) |
| GPIO10 | TP_SDA | I2C data (touch) |
| GPIO11 | TP_INT | Touch interrupt |
| GPIO12 | LCD_RST | LCD reset |
| GPIO13 | LCD_QSPI_D3 | QSPI data 3 |
| GPIO14 | LCD_QSPI_D2 | QSPI data 2 |
| GPIO15 | LCD_QSPI_D1 | QSPI data 1 |
| GPIO16 | LCD_QSPI_D0 | QSPI data 0 |
| GPIO17 | LCD_QSPI_CS | QSPI chip select |
| GPIO18 | LCD_QSPI_SCL | QSPI clock |
| GPIO19 | LCD_TE | LCD TE |
| GPIO20–23 | SD-D0–D3 | SDMMC 4-bit data |
| GPIO24 | SD-CLK | SDMMC clock |
| GPIO25 | SD-CMD | SDMMC command |
| USB_DP / USB_DM | USB 2.0 HS PHY | Module native USB differential, via FPC2 to Type-C |
| GPIO33 | USB_DN | Via FPC2 to Type-C; coexist with ESP_USB_* |
| GPIO34 | USB_DP | Via FPC2 to Type-C; coexist with ESP_USB_* |
| GPIO35 | AUDIO_I2C_SCL | ES8311 I2C SCL (dock) |
| GPIO37 | FPC2 breakout | Candidate peripheral |
| GPIO38 | FPC2 breakout | Candidate peripheral |
| GPIO39 | FPC2 breakout | Candidate peripheral |
| GPIO40 | AUDIO_I2C_SDA | ES8311 I2C SDA |
| GPIO42 | I2S_MCLK | I2S master clock |
| GPIO43 | I2S_BCK | I2S bit clock |
| GPIO44 | I2S_DI | I2S data in (ADC → MCU) |
| GPIO45 | I2S_WS | I2S word select / LRCK |
| GPIO46 | I2S_DO | I2S data out (MCU → DAC) |
| GPIO47 | PA_CTRL | NS4150B amplifier enable |
| GPIO48–50 | FPC2 breakout | Candidate peripherals |
| GPIO51 | BAT_ADC | Battery/system voltage ADC (divider sample) |
| GPIO52–59 | Reserved | Not assigned |
| GPIO60 | SD-CTRL | SD 3.3 V power switch, active low |
| GPIO61 | **BOOT / RGB_DATA** | BOOT button pulls **BOOTMODE** low; same net drives **RGB_DATA** for the 12-LED ring |

> [!NOTE]
> **GPIO61** is shared between the dock **BOOT** key (**BOOTMODE**) and the WS2812-compatible **RGB_DATA** line. Entering download mode requires pulling GPIO61 low while connecting Type-C; firmware driving the LED ring must account for this shared net (see FAQ).

### 2.3 Display information

| Item | Specification | Unit | Remark |
| --- | --- | --- | --- |
| Display type | Round TFT, transmissive, normally black | — | Transmissive, Normally Black |
| Screen size | 2.1 | Inch | Diagonal |
| Resolution | 360 × 360 | Dots | — |
| Driver IC | GC9B72NA | — | — |
| Touch IC | CST826 | — | Multi-touch, G+F CTP |
| Host interface | QSPI | — | As wired on core-board FPC1 |
| Module OD (TP OD) | 62.80 ± 0.1 | mm | — |
| Viewing area VA | 53.88 ± 0.2 | mm | — |
| Active area AA | 52.92 | mm | — |
| Thickness | 3.71 ± 0.2 | mm | — |
| Brightness | 300 (min) / 350 (typ) | cd/m² | — |
| Viewing angle | 80/80/80/80 typ. | ° | — |
| Logic supply VDD | 3.3 | V | — |
| Backlight | 4× white LED | — | If ≈80 mA, Vf ≈2.8 V typ. |
| Operating temperature | −20 ~ 70 | °C | — |

### 2.4 Audio interface

**[Dock-dependent]** ES8311 codec + NS4150B amplifier + ZTS6216 analog microphone on the dock. Core-only designs must implement equivalent signals via FPC2 (I2S / AUDIO_I2C / PA_CTRL, etc.). Codec supply **CODEC_3V3** from 5 V via RT9013-33GB. Speaker on **CN2** (1.25-2P WT).

| Signal | GPIO / device | Description |
| --- | --- | --- |
| I2C SDA / SCL | GPIO40 / GPIO35 | ES8311 I2C |
| I2S_MCLK | GPIO42 | I2S master clock |
| I2S_BCK | GPIO43 | Bit clock |
| I2S_DI | GPIO44 | ADC → MCU |
| I2S_WS | GPIO45 | LRCK / WS |
| I2S_DO | GPIO46 | MCU → DAC |
| PA_EN | GPIO47 | High = amplifier on |
| MIC | ZTS6216 | Differential ADC_MIC1P/N |
| Speaker | NS4150B | Differential output, typ. 4 Ω / 3 W class |

### 2.5 Board-to-board interconnect FPC2 (customer expansion)

Use a **0.3×31P×100 mm** connecting cable. The core board expands idle IOs for a carrier board (power, USB, audio, LED data, BOOT, BAT_ADC, EC2, etc.). Pinout matches the schematic labels on core and dock.

| Pin | Symbol | Description |
| --- | --- | --- |
| 1 | GND | Ground |
| 2 | 5V | Board-to-board 5 V supply |
| 3 | BOOTMODE | Download mode (→ GPIO61) |
| 4 | GND | Ground |
| 5 | GPIO49 | To dock / candidate peripheral |
| 6 | GND | Ground |
| 7 | GPIO48 | Candidate peripheral |
| 8 | GND | Ground |
| 9 | GPIO47 | PA_CTRL |
| 10 | GPIO51 | BAT_ADC |
| 11 | GPIO46 | I2S_DO |
| 12 | GPIO50 | Candidate peripheral |
| 13 | GPIO45 | I2S_WS |
| 14 | GPIO39 | Candidate peripheral |
| 15 | GPIO44 | I2S_DI |
| 16 | GPIO38 | Candidate peripheral |
| 17 | GPIO43 | I2S_BCK |
| 18 | GPIO37 | Candidate peripheral |
| 19 | GPIO42 | I2S_MCLK |
| 20 | EC2_B | Second rotation-detection channel B (SSCM, core SW1) |
| 21 | GPIO40 | AUDIO_I2C_SDA |
| 22 | EC2_A | Second rotation-detection channel A (SSCM, core SW1) |
| 23 | GPIO35 | AUDIO_I2C_SCL |
| 24 | GND | Ground |
| 25 | USB_DN | USB differential − |
| 26 | USB_DP | USB differential + |
| 27 | GND | Ground |
| 28 | ESP_USB_P | Module USB + |
| 29 | ESP_USB_N | Module USB − |
| 30 | 5V | Board-to-board 5 V supply |
| 31–33 | GND | Ground |

### 2.6 USB & download interface

**[Dock-dependent]** Type-C, ESD, CC resistors, and BOOT key are on the dock. The full kit has **no CH340** or other USB-UART bridge; download and enumeration rely on **ESP32-S31 native USB**.

| Port | Bridge / PHY | Role | Notes |
| --- | --- | --- | --- |
| Type-C | USB_DP/DN, USB 2.0 HS PHY, ESP_USB_P/N | Download / log / 5 V in | CC1/CC2 = 5.1 kΩ to GND (UFP); H5VL10B ESD on differential lines; hold **BOOT (GPIO61)**, then connect Type-C |

The schematic brings out both **ESP_USB_P/N** and **USB_DP/DN** on Type-C (plug A/B sides). Treat them as module native USB in software. Recommended: hold BOOT → connect Type-C → `idf.py --preview flash monitor`.

### 2.7 Rotation detection / keys / LED ring

Rotary interaction is **not** a classic EC11 quadrature encoder. The mechanism drives two Alps Alpine **SSCM110100** compact two-way detector switches. Net names **EC1_A/B** and **EC2_A/B** can be misread as encoder channels; this product uses **detector switches** instead.

| RefDes (core) | P/N | Nets | Role |
| --- | --- | --- | --- |
| SW2 | SSCM110100 | EC1_A / EC1_B | Primary path to MCU GPIO2/3 |
| SW1 | SSCM110100 | EC2_A / EC2_B | Second path via FPC2 to dock |

**RefDes note:** Core-board **SW1** = SSCM detector; dock-board **SW1** = BOOT tactile key **GT-TC018B-H035-L1**. Same designator, different parts.

**SSCM110100 (summary):** 1 pole / 2 positions; 4.0×5.0×1.5 mm; 1.57 mm travel; 0.35 N max force; rating 1 mA / 5 V DC max, 50 µA / 3 V DC min; 50 000 cycles; −10 ~ +60 °C. Software must handle contact bounce.

**Keys & power (dock):**

| Name | Location | Function |
| --- | --- | --- |
| BOOT (dock SW1) | Dock | GT-TC018B-H035-L1, pulls BOOTMODE (GPIO61) low |
| Power slide SW3 | Dock side | MST-12D18G4, controls VCC_OUT |

**LED ring (dock):** 12× **1010RGBC-2812B**, daisy-chained; **RGB_DATA = GPIO61** via R54 = 100 Ω; supply **VDD_RGB / VCC_OUT**.

### 2.8 Storage & battery charging

**Storage**

| Item | Description |
| --- | --- |
| On-board Flash | 16 MB, 1.8 V domain |
| PSRAM | In-package 16 MB 1.8 V Octal SPI (ESP32-S31NRV16) |
| MicroSD | SDMMC 4-bit: CLK = GPIO24, CMD = GPIO25, D0–D3 = GPIO20–23; **SD-CTRL = GPIO60** (3.3 V switch, active low) |

**Battery & charging (dock)**

| Item | Description |
| --- | --- |
| Charge IC | TP4055 (U1) |
| Battery connector | CN1, 1.25-2P WT (BAT+) |
| Power path | Slide switch SW3 selects/connects VCC_OUT |
| Voltage sense | BAT_ADC = **0.4 × VCC_OUT** → GPIO51 |
| Nominal voltage | 3.7 V |
| Battery capacity | 750 mAh |

### 2.9 Voltage & current

| Item | Conditions | Min | Typ | Max | Unit |
| --- | --- | --- | --- | --- | --- |
| Input voltage (USB) | Type-C VBUS | — | 5.0 | — | V |
| Core logic | TLV62569 output | — | 3.3 | — | V |
| Charge current | TP4055, Rprog = 2.2 kΩ | — | ≈470 | — | mA |
| System operating current | Display on / Wi-Fi / audio, 5 V | — | 600 | — | mA |
| System standby current | Backlight off, RF idle | — | 500 | — | mA |
| Backlight LED current | Module spec 4 LED | — | ≈80 | — | mA |

Recommended supply: Type-C **5 V / ≥1 A** adapter (leave margin when charging and running together).

### 2.10 Reliability test (summary)

| Item | Conditions | Min | Typ | Max | Unit |
| --- | --- | --- | --- | --- | --- |
| Working temperature | 60% RH at 5 V | −20 | 25 | 70 | °C |
| Storage temperature | — | −30 | 25 | 85 | °C |
| Working humidity | 25 °C | 10% | 60% | 90% | RH |
| ESD | — | — | — | Contact ±4 kV / Air ±8 kV | kV |

---

## 3. Functional block diagram

![Functional block diagram](images/Functional_Block_Diagram.png)

> **Note:** ESP32-S31 includes a 2.4 GHz radio that supports Wi-Fi 6, Bluetooth 5.4 (LE), Bluetooth Classic, and 802.15.4. Because they share the same RF front-end, Wi-Fi and Bluetooth cannot transmit or receive simultaneously; the radio switches between protocols as needed. USB 2.0 High-Speed is an independent PHY and does not share that front-end.

---

## 4. Mechanical drawing & exploded structure

![Front / side / back views](images/Dimension_drawing.jpg)

![Exploded structure](images/Exploded_structure.png)

Stack (rear to front): Rear housing → Li-ion battery & speaker → Dock board → light guide ring → support bracket → rotating bearing → metal housing → Core board → 2.1" round touch screen.

---

## 5. Software

This repository does **not** yet include an `examples/` tree or a published board support package name for DesKnob. **ESP-IDF** example projects and documentation are **planned (TBD)**. Open-source materials called out in the product specification are still being refined.

When examples are published, they are expected to follow the same ESP32-S31 constraints as other VIEWE S31 products:

- Install ESP-IDF **master** (for example with [EIM](https://dl.espressif.com/dl/eim/index.html)). Numbered stable releases do not include `esp32s31`.
- Always pass **`--preview`** for this target.

### 5.1 Getting started (ESP-IDF, preview target)

1. **Install ESP-IDF master**
   - Confirm the toolchain:

```powershell
idf.py --version
idf.py --preview --list-targets
```

   - The version path should contain `master`. The target list must include `esp32s31`.
2. **Set target and flash (when a project is available)**

```powershell
idf.py --preview set-target esp32s31
idf.py --preview -p PORT flash monitor
```

   - Replace `PORT` with the OS USB device for **native USB** (not a CH340 serial port).
   - Recommended download sequence: slide power on if needed → **hold BOOT** → connect Type-C → release BOOT after enumeration → flash.

If `esp32s31` is missing from the target list, the install is not master or `--preview` was omitted.

---

## 6. Related documents

- [Product specification V1.0 (PDF)](datasheet/SFG-VIEWE-DesKnob_V1.0_SPEC.pdf)
- Espressif **ESP32-S31** preliminary / series documentation (see Espressif website; pre-release for preview silicon)

---

## 7. FAQ

* **Q.** Why is there no **CH340** COM port when I plug in Type-C?
* **A.** The full kit uses **ESP32-S31 native USB** (USB 2.0 High-Speed PHY + USB Serial/JTAG). There is no USB-UART bridge IC. Select the USB device your OS assigns to the module, not a separate serial adapter.

* **Q.** How do I enter firmware download mode?
* **A.** On the dock, hold **BOOT** (pulls **GPIO61 / BOOTMODE** low), then connect **Type-C** (5 V). Release BOOT after the port enumerates. Run `idf.py --preview flash` (or `flash monitor`). The specification also mentions tapping **RESET** after holding BOOT when applicable.

* **Q.** Why is `esp32s31` missing from `idf.py --list-targets`?
* **A.** ESP32-S31 is a **preview** chip. Install ESP-IDF **master** and use **`--preview`** on commands (for example `idf.py --preview --list-targets` and `idf.py --preview set-target esp32s31`). Numbered stable releases such as 5.4 or 5.5 will not list this target.

* **Q.** GPIO61 is both **BOOT** and **RGB_DATA** — is that correct?
* **A.** Yes, per the product specification and GPIO table: **GPIO61** serves **BOOTMODE** for the download key and **RGB_DATA** for the 12-LED WS2812-compatible ring. Plan firmware and hardware test accordingly (do not drive the LED chain in a way that blocks download when you need GPIO61 low).

* **Q.** Is the knob an **EC11 encoder**?
* **A.** No. Rotation uses Alps **SSCM110100** two-way **detector switches** (EC1 on GPIO2/3, EC2 via FPC2), not quadrature encoder pulses. Handle debouncing in software.

* **Q.** Core-board **SW1** and dock **SW1** — same part?
* **A.** No. Core **SW1** is the **SSCM** rotation switch (EC2). Dock **SW1** is the **BOOT** key. The schematic reuses designator SW1 on both boards.

* **Q.** Where are code examples and schematics on GitHub?
* **A.** This repository is being populated. The specification notes that drawings and open materials are still being refined; watch this repo and the PDF under `datasheet/` for updates.
