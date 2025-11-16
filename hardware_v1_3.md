# Hardware Build Spec

**Version:** 1.3
**Date:** October 30, 2025
**Depends on:** PRD v1.3, Interaction & State Spec v1.3, Test Plan v0.1+

---

## 1) Overview

Handheld, lanyardable device that:

* Logs events via **7 front buttons** (6 behaviors + **INTERVENE**)
* Adds one-shot **target** via **3 side buttons** (ME/SIB/OTHER)
* **Auto-records audio** for incidents, with **REC on OLED**
* Stores `events.csv` + `incident_*.wav` on **microSD**
* Gives **haptic** + **OLED** confirmation on every press
* Runs from a **LiPo** with onboard charging

Primary v1 data path: **microSD removal** (simple/reliable). USB-C readout is a stretch.

---

## 2) Industrial / Envelope Targets (musts)

* **Size:** H ~110–130 mm; W ~45–55 mm; T ~18–22 mm
* **Mass:** ≤ ~80 g
* **Ergonomics:** one-handed use; **side thumb** access to targets
* **Lanyard:** anchor to **enclosure**, not PCB; center mass so it hangs flat
* **Non-escalating look:** neutral, low-profile buttons; no flashing LEDs; REC stays on OLED

---

## 3) Subsystems

* **MCU:** ESP32-class dev board **with LiPo charger & USB-C** (e.g., Feather-class ESP32-S3).
* **Storage:** microSD (SPI) → **FAT32**; accessible slot in enclosure.
* **Mic:** I2S digital MEMS mic (SPH0645/ICS-43434 class), mono 8–16 kHz.
* **Display:** 0.96" I2C OLED (SSD1306/SH1107).
* **Haptic:** 3 V coin motor + NPN/MOSFET + flyback diode (or haptic driver IC if preferred).
* **Buttons:** 10x momentary NO tactiles (low-profile) or a thin membrane set for the front grid, plus 3 side tactiles.
* **Battery:** 3.7 V LiPo 1000–1500 mAh (JST-PH 2-pin), with strain relief.
* **Power control:** use board **EN** pin or a small slide switch; no raw LiPo inline switch.

---

## 4) Bill of Materials (reference)

* 1× ESP32 dev board w/ LiPo charging & USB-C (Feather-class ESP32-S3 recommended)
* 1× microSD SPI breakout (if not onboard)
* 1× I2S MEMS mic breakout (SPH0645/ICS-43434 family)
* 1× 0.96" 128×64 I2C OLED (SSD1306/SH1107)
* 1× Coin vibration motor (3 V) + 1× small NPN (e.g., S8050/2N2222) or small MOSFET, 1× flyback diode
* 10× low-profile tact switches (front): VERBAL, PHYSICAL, PROPERTY, REFUSAL, SELF_HARM, REGULATED, **INTERVENE** + (optionally spare)
* 3× side tact switches (ME, SIB, OTHER)
* 1× 3.7 V LiPo 1000–1500 mAh (JST-PH)
* 1× ABS handheld enclosure ≈ 120×50×20 mm (Hammond-1553B-class or similar)
* Panel hardware, wire (28–26 AWG), heat-shrink, headers, standoffs, strain reliefs

*(You can prototype on perfboard; v2 can move to a custom PCB.)*

---

## 5) Electrical Design

### 5.1 Block Diagram (text)

LiPo → ESP32 board (charger/reg)
ESP32 ↔ I2C (OLED)
ESP32 ↔ I2S (MIC)
ESP32 ↔ SPI (microSD)
ESP32 GPIO (10 front buttons + 3 side buttons) w/ **internal pull-ups** → active-low
ESP32 GPIO → haptic driver transistor → coin motor (+ diode to 3 V)

### 5.2 Pin Hygiene (critical)

* **Avoid boot/strap pins** for buttons or lines that could be held low at boot.
  On classic ESP32: **GPIO0/2/12/15** are risky. On **ESP32-S3**, check board docs for boot pins and avoid them for buttons.
* Prefer pins with **internal pull-ups** available for inputs.
* Map **MISO** to an **input-only** pin if on classic ESP32 (e.g., GPIO34–39).

### 5.3 Example Pin Map (DEV REFERENCE — finalize per board)

*(Classic ESP32-WROOM-32 style; safe, typical choices. Adjust for your exact board.)*

| Function           | Pin | Notes                          |
| ------------------ | --- | ------------------------------ |
| VERBAL             | 14  | tap/hold → threat              |
| PHYSICAL           | 27  | tap/hold → severe              |
| PROPERTY           | 26  | tap (hold optional → severe)   |
| REFUSAL            | 33  | tap                            |
| SELF_HARM          | 32  | tap/hold → danger              |
| REGULATED          | 25  | tap                            |
| **INTERVENE**      | 17  | tap → SUPPORT, hold → BOUNDARY |
| Target **ME**      | 4   | one-shot target arm            |
| Target **SIB**     | 5   | one-shot target arm            |
| Target **OTHER**   | 18  | one-shot target arm            |
| OLED SDA (I2C)     | 21  | I2C                            |
| OLED SCL (I2C)     | 22  | I2C                            |
| HAPTIC driver gate | 19  | GPIO → transistor/MOSFET       |
| SD **CS**          | 13  | SPI CS                         |
| SD **SCK**         | 16  | SPI CLK (adjust per board)     |
| SD **MOSI**        | 23  | SPI MOSI                       |
| SD **MISO**        | 34  | **input-only OK**              |
| I2S **BCLK**       | 12* | *Mind strap; pick per board*   |
| I2S **LRCLK/WS**   | 15* | *Mind strap; pick per board*   |
| I2S **DATA**       | 36  | input-only OK                  |
| LiPo               | JST | to board battery port          |

> **Action:** When you select the actual dev board, regenerate this table with that board’s recommended SD/I2S pins and non-strap GPIOs, then **print and tape to bench**.

### 5.4 Button Wiring

* One side of each button → **GND**
* Other side → assigned **GPIO**
* Enable **internal pull-up** on each GPIO
* **Active-low** press = stable and simple
* Debounce in firmware; don’t add RC unless needed

### 5.5 Haptic Driver

* ESP32 GPIO → base/gate of small transistor (with base resistor ~1–4.7 kΩ if BJT)
* Motor between **+3 V** and transistor; **flyback diode** across motor
* Coin motor current is small but still not for direct GPIO drive
* Keep motor wiring short; route away from mic

### 5.6 I2S Mic & EMI

* Place mic **away from** the motor and SD lines; keep a short run
* Put a **small unobstructed port** in the enclosure; don’t point into clothing or lanyard path
* Sample rate target: **16 kHz mono, 16-bit PCM** (8 kHz is acceptable if storage tight)

### 5.7 SD Storage

* microSD SPI; **FAT32**
* Name files:

  * CSV: `events.csv` (append)
  * Audio: `incident_YYYY-MM-DDThh-mm-ss.wav`
* Flush CSV **per row** (best effort) to survive power loss

---

## 6) Mechanical Design

### 6.1 Front Layout (2×3 + 1)

```
[ OLED 0.96" ]
VERBAL   PHYSICAL
PROPERTY REFUSAL
SELFHARM REGULATED
   [  INTERVENE  ]  ← distinct color/shape/texture
```

* Low-profile caps or a thin membrane for grid; keep spacing tactilely obvious
* INTERVENE visually distinct (your action ≠ child behavior)

### 6.2 Side Layout (targets)

```
   [ ME ]
   [ SIB ]
   [ OTHER ]
```

* Vertical stack on thumb side
* Slight recess/bump to feel by touch
* No haptic buzz on target (default), to stay discreet

### 6.3 Lanyard & Mass

* **Anchor molded into shell**, not PCB
* Balance battery+board centrally so it hangs flat
* Keep mic port clear of strap path

### 6.4 Ports & Access

* **microSD** accessible from a side/bottom slot (preferred for v1)
* **USB-C** access for charging/programming (and future export)
* Optional **EN** slide switch hole if you want hard “sleep”

---

## 7) Power & Runtime

* **Battery:** 1000–1500 mAh LiPo
* **Charging:** onboard (USB-C)
* **Runtime targets:**

  * Many hours idle (REC off)
  * Multiple short incidents + at least one long (10–20 min) incident
* **Safety:** isolate LiPo from screws/edges; strain-relief JST; don’t charge while worn during escalations

---

## 8) Firmware-Relevant Hardware Settings (tie-ins)

* Inputs: internal **pull-ups**, active-low
* **Tap vs hold:** firmware threshold = **1000 ms** (tunable), designed to **ignore hard fast taps as holds**
* Cooldown: **5 min** (tunable)
* OLED event confirm: ~**2 s**, then revert to REC/idle
* WAV header finalize on stop; flush CSV per event

---

## 9) Survivability Requirements (hard requirements)

* **No missed presses** under shaky/fast use (debounce + ISR where appropriate)
* Survives:

  * **Drop** from chest height to carpet/couch/floor
  * **Lanyard yank** without internal damage
* Continues functioning after the above without reflash
* No bright/external recording indicators (OLED only)

---

## 10) Assembly Notes

* Label harnesses; consider small JST-XH/MHX for button groups (front 7; side 3) to simplify enclosure service
* Hot-glue or zip-tie **strain relief** on LiPo leads and lanyard eyelet
* Insulate under mic and SD boards; avoid shorts on enclosure standoffs
* Dry-fit panel holes with a paper drill guide before committing

---

## 11) Test Hooks (map to Test Plan)

* **Firmware test build**: GPIO test page (press→OLED echo) before sealing case
* **Audio sanity**: quick record/playback with finger snaps and short speech
* **REC indicator**: verify state on OLED during ACTIVE/COOLDOWN/IDLE
* **Drop/yank**: perform on soft surface, then re-run quick tests

---

## 12) Print Pack (bring to bench)

1. Final **Pin Map** (for your exact board)
2. **Front/Side drill guide** with labels
3. **Wiring diagram** (buttons → GND, GPIO list, I2C/I2S/SPI nets)
4. **Power & Safety** one-pager (LiPo handling + strain relief)
5. **Test checklist** excerpt (Section 11 + Test Plan)

---

## 13) Open Decisions (choose once, then freeze)

* **Board:** Feather-class ESP32-S3 (rec.) vs audio-dev board with built-in SD/mic (bigger but simpler wiring)
* **Front inputs:** individual tact switches vs thin **membrane pad** (membrane = flatter, more durable; tacts = easier now)
* **SD exposure:** external slot (rec. v1) vs internal only + USB export (slimmer but less flexible)
* **RTC:** skip v1 (aligns with PRD stretch)

> Document your choices on this page and update the **Pin Map** accordingly.

---

### Done Criteria (hardware side)

* All subsystems wired and mounted per this spec
* Pin map matches firmware constants
* Buttons feel distinct; INTERVENE distinct
* Side targets reachable eyes-off
* Lanyard anchor & strain relief present
* SD accessible; USB-C charge works
* Passes survivability checks; no missed presses in shake test
