# Engineering Decision Log

Append-only. Never rewrite a past entry to say something different — supersede it with a new numbered entry and mark the old one Superseded.

Status values: **Agreed** · **Provisional** (accepted but awaiting a specific validation) · **Open** (not yet decided) · **Superseded**.

---

## D1 — V1 feature tiers
**Date:** 2026-09-10 · **Status:** Agreed

**Decision:** Adopt the MUST HAVE / SHOULD HAVE / OPTIONAL / FUTURE REVISION tiers recorded in `V1-SPEC.md` section 3.

**Reason:** Fixes the boundary of V1 so scope arguments are settled once rather than repeatedly. Protects against feature creep on a first hardware build.

**Affects:** Everything. All stage definitions of done derive from the MUST HAVE list.

**Revisit when:** A MUST HAVE item proves technically infeasible, or an OPTIONAL item becomes necessary for a MUST HAVE item to work.

---

## D2 — Control inventory
**Date:** 2026-09-10 · **Status:** Agreed (frozen); sub-decision D2a open

**Decision:** 15 digital inputs and 6 analog channels — 2 analog sticks (4 axes), L3/R3 stick clicks, 4-way D-pad, ABXY, LB/RB, analog LT/RT (2 axes), Start, Select, Profile. No Home/Guide button; the Profile button occupies that central special-button role. Full table in `CONTROL-INVENTORY.md`.

**Reason:** Covers every V1 use case with a control set players already understand, and matches the Xbox 360 / XInput control set minus the Guide button, which keeps a 1:1 mapping to a virtual Xbox pad available later without compromises. Also fits directly on the candidate MCU with no key matrix, shift registers, external ADC or analog multiplexer.

**Alternatives considered:** Adding a Home/Guide button (rejected — the Profile button covers it); adding rear paddles or a second shoulder row (rejected — feature creep, and the touchscreen absorbs auxiliary functions).

**Correction recorded:** An earlier Stage 1 draft stated 18 digital inputs. That was an arithmetic error and included a Home/Guide button. 15 is correct.

**Affects:** Pin map, PCB architecture, firmware HID descriptor, enclosure layout, companion app remapping UI.

**Revisit when:** Mockup testing shows a control is unreachable or missing, or Stage 3 reveals an HID constraint.

---

## D2a — Stick arrangement
**Date:** — · **Status:** **Open**

**Question:** Symmetric sticks (both low, PlayStation style) or offset sticks (left stick high-outer with the D-pad below, Xbox style)?

**Why it is not cosmetic:** It sets the PCB outline, the grip angle, and how far the ~5-inch screen pushes the controls outward.

**Closes when:** Both team members have held the full-scale physical mockup in both arrangements and chosen one. Decision recorded here before any PCB outline work begins.

---

## D3 — Display architecture
**Date:** 2026-09-10 · **Status:** Agreed

**Decision:** Off-the-shelf ~5-inch panel driven by an off-the-shelf HDMI display driver board, presented to Windows as a second monitor. Capacitive touch returns to the PC over USB. **No Linux SBC in V1.**

**Alternatives considered:** Linux SBC inside the handheld running a Moonlight/Sunshine stream over the network; MCU-driven SPI display showing telemetry only.

**Reason:** The SBC exists to move video without a video cable. V1 is tethered, so it solves a problem V1 does not have, while adding SBC selection, a Linux image, streaming software, network latency tuning, SBC power and thermal design, and an SBC-to-MCU link. The HDMI path delivers the same user-visible feature — a genuine second screen with touch and true dual-screen mode — with no streaming software and no network latency.

**Accepted cost:** The driver board is bulky and forces a second cable out of the enclosure.

**Affects:** S3, S6 power budget, enclosure internal volume, cable exit design, companion app (no streaming client needed).

**Revisit when:** The device goes wireless. The SBC is the correct answer then, and V1 mechanical design should leave the internal volume plausible for it.

---

## D4 — Connection method for V1
**Date:** 2026-09-10 · **Status:** Agreed

**Decision:** Wired USB-C only. Wireless is OPTIONAL and is only attempted after all MUST HAVE functionality is stable.

**Reason:** Removes pairing, latency, power and reliability variables from the critical path of a first build. The chosen MCU family keeps BLE available later as a firmware change rather than a redesign.

**Affects:** D5 (bus power is only possible because the device is tethered), D6, enclosure cable exit, firmware architecture.

---

## D5 — Power architecture for V1
**Date:** 2026-09-10 · **Status:** Agreed, pending Stage 4 measurement

**Decision:** USB bus-powered from the PC. No internal battery, no charger, no protection circuitry in V1.

**Open verification:** Total draw of MCU, panel and driver board must be measured in Stage 4 and shown to fit the available USB budget. If it does not, the resolution is an externally powered display path, **not** a battery.

**Affects:** S6, BOM, enclosure, thermal, D3.

**Revisit when:** Stage 4 measurements exceed the USB budget, or the device goes wireless.

---

## D6 — MCU family
**Date:** 2026-09-10 · **Status:** **Provisional**

**Decision:** ESP32-S3 for prototyping, with the `ESP32-S3-WROOM-1` module as the provisional production part.

**Provisional until:** A Stage 3 prototype demonstrates a stable composite USB HID device on Windows — gamepad, keyboard/mouse, and a vendor configuration interface working simultaneously and surviving disconnect/reconnect. Until that test passes and is logged in `TEST-LOG.md`, do not produce artifacts that are expensive to port: no custom PCB layout, no production BOM.

**Requirements it must satisfy:** >=15 digital inputs, >=6 ADC channels, native USB device capable of composite HID, dev board around $15, a credible path to BLE later, and a package two students can hand-solder onto a first custom PCB.

**Alternatives considered:**
- *RP2040 / RP2350* — best-in-class USB HID support and documentation, but only 3 exposed ADC channels, so 6 analog channels needs an external ADC or a 74HC405x multiplexer; no BLE without the CYW43 variant; bare-chip design needs external flash and a crystal.
- *STM32F4/G4* — best ADC of the three, but steepest learning curve, priciest dev boards, most external components, and no wireless path.

**Reason for the recommendation:** 10 usable ADC1 channels covers all 6 analog inputs on the ADC block that stays usable with the radio active; BLE is already on-die so wireless later is firmware work rather than a redesign; and the WROOM-1 module drops onto a custom PCB as a single castellated part with no crystal, no USB PHY and no external flash to route — a substantial derisking of a first board.

**Known weakness:** ESP32-S3 ADC linearity and noise are worse than STM32 or RP2040. Calibration is required regardless, so this is manageable, but it must be verified in Stage 2.

**Documented fallback:** RP2040 with an external ADC or analog multiplexer. Switching costs only the input prototype if it happens before Stage 8.

**Affects:** Pin map, PCB architecture and BOM, firmware toolchain, wireless feasibility.

---

## D7 — How Windows sees the device
**Date:** 2026-09-10 · **Status:** Agreed

**Decision:** A composite USB HID device exposing a gamepad interface, a keyboard interface, a mouse interface, and a separate vendor interface for companion-application configuration traffic. The MCU does **not** attempt to present itself as a native Xbox/XInput device.

**Reason:** Standard HID needs no driver, no signed device identity and no reverse engineering, and it supports the keyboard/mouse and configuration paths that XInput cannot carry. The control inventory is deliberately XInput-shaped, so if a game later refuses non-XInput controllers, the companion application can create a virtual Xbox pad on the PC side via ViGEmBus with a 1:1 mapping — a PC-software addition, not a firmware or hardware change.

**Known risk:** Some titles accept only XInput controllers. Accepted for V1; the ViGEmBus route is the planned mitigation and is classified OPTIONAL.

**Affects:** Firmware HID descriptor, companion app architecture, `CONTROL-INVENTORY.md` host mapping.

**Revisit when:** Testing shows a game we care about rejects the HID gamepad.

---

## D8 — Repository and documentation structure
**Date:** 2026-09-10 · **Status:** Agreed

**Decision:** Adopt the directory structure and document set described in `README.md` and created in this repository, with `CLAUDE.md` at the root as persistent project context, and this file as the append-only decision log.

**Reason:** Design decisions and test evidence are the parts of a hardware project most easily lost. Recording them from the start also produces the final documentation almost for free.

**Affects:** All future work and all Claude Code interaction with this repository.
