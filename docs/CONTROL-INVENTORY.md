# Control Inventory

**Status:** Frozen 2026-09-10 (D2). Stick arrangement open (D2a).
Changes require a new entry in `DECISIONS.md`.

---

## 1. Electrical inputs

This is the count the pin map and PCB depend on.

| Control | Qty | Digital inputs | Analog channels | Notes |
|---|---|---|---|---|
| Analog stick X/Y | 2 | — | 4 | Potentiometer or Hall module, TBD Stage 2 |
| Stick click L3 / R3 | 2 | 2 | — | Integral to standard thumbstick modules |
| D-pad U / D / L / R | 1 pad | 4 | — | 4 discrete switches under one pivot; diagonals from two-at-once |
| ABXY | 4 | 4 | — | |
| LB / RB | 2 | 2 | — | Digital |
| LT / RT | 2 | — | 2 | Analog travel, full range required |
| Start | 1 | 1 | — | |
| Select | 1 | 1 | — | |
| Profile | 1 | 1 | — | Central special-button role; replaces Home/Guide |
| **Total** | | **15** | **6** | |

No Home/Guide button in V1.

Display and touchscreen are a separate subsystem (S3) and are not counted here.

## 2. What the host sees

Different from the count above, and easy to get wrong.

| Host-visible item | Count | Source |
|---|---|---|
| Buttons | 10 | ABXY (4), LB/RB (2), L3/R3 (2), Start, Select |
| Hat switch | 1 | The D-pad, reported as a single 4-bit hat — **not** as four buttons |
| Axes | 6 | Lx, Ly, Rx, Ry, LT, RT |

The **Profile button is consumed by firmware and never reported to the host.** It selects the active on-device profile.

So: 15 electrical inputs, but 10 buttons + 1 hat + 6 axes over HID. Both numbers must appear in the firmware review checklist.

## 3. XInput compatibility

The inventory is exactly the Xbox 360 / XInput control set minus the Guide button:

| XInput control | Ours |
|---|---|
| A, B, X, Y | ABXY |
| LB, RB | LB, RB |
| Back, Start | Select, Start |
| Left stick, Right stick (click) | L3, R3 |
| D-pad | D-pad hat |
| Left/right thumbstick axes | 4 axes |
| Left/right trigger | LT, RT analog |
| Guide | *(none — Profile is local)* |

This is deliberate. Per D7 the device presents standard HID, but if a title later refuses non-XInput controllers, the companion application can create a virtual Xbox pad through ViGEmBus with a 1:1 mapping and no invented assignments. **Do not add or remove controls in a way that breaks this mapping without an explicit decision.**

## 4. Pin budget

| Purpose | Pins |
|---|---|
| Digital inputs | 15 |
| Analog inputs | 6 (must sit on ADC1, the block usable with the radio active) |
| Native USB D+/D− | 2 (dedicated) |
| Status LED | 1 |
| Reserved for future IMU (I2C) | 2 |
| **Total** | **26** |

Comfortably within the candidate MCU's usable GPIO after strapping and flash/PSRAM pins are excluded.

**Consequence: every control is wired directly to its own pin.** No key matrix, no shift registers, no external ADC, no analog multiplexer. A matrix would only add diodes and ghosting problems to solve a pin shortage that does not exist.

**Dev-board caveat (Stage 2):** on ESP32-S3 modules with octal SPI PSRAM (the R8
and R16V variants), GPIO35, 36 and 37 are connected to the PSRAM and are not
available for other uses, per the ESP32-S3-WROOM-1 datasheet. Roughly 23-24 header
pins then remain usable against the 21 signals needed: it fits, with little margin.
This firmware requires no PSRAM, so the production module should be a variant
without octal PSRAM in order to recover those pins.

## 5. Open items

- **D2a — stick arrangement:** symmetric (PlayStation) or offset (Xbox). Deferred by D9 to the pre-PCB mechanical gate. Must be closed before any PCB outline work.
- **Stick and trigger modules:** to be selected in Stage 2 from controller-repair replacement parts, with real dimensions, before the ergonomic validation required by D9. Analog trigger assemblies are the hardest mechanical part of this device and will not be designed from scratch.
- **Physical placement** of Start, Select and Profile around the screen: set by the mockup.

## 6. Pin assignment

Empty until Stage 7. Do not populate before the input prototype works and D2a and D6 are closed.

| Control | MCU pin | Type | Notes |
|---|---|---|---|
| *(Stage 7)* | | | |
