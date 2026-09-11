# CLAUDE.md — Adaptive Handheld PC Controller

Persistent project context for Claude Code. Read this before touching anything in this repository.

---

## 1. What we are building

A tethered handheld PC peripheral for Windows: a custom Xbox/PS4-style gamepad control set wrapped around a built-in ~5-inch touchscreen, conceptually similar to a Wii U GamePad. Its controls, behaviour and screen contents change depending on which PC application is in the foreground.

The PC performs all computation. The handheld is an input device plus a display surface. It is **not** a computer and contains no application processor in V1.

Team: two second-year Electrical Engineering students. This is their first serious hardware build. Repository: `ashrafraf/adaptive-handheld-controller`.

## 2. Current V1 scope

V1 is a **wired, USB-C, bus-powered handheld** with a full gamepad control set, a custom PCB, a 3D-printed enclosure, on-device profiles, a Windows companion application, and a ~5-inch touchscreen that Windows sees as a genuine second monitor over HDMI.

V1 explicitly excludes: battery, wireless operation, onboard Linux SBC, network video streaming, haptics, and IMU/gyro.

## 3. Current high-level architecture

Two independent host connections:

- **Control path:** controls -> MCU (ESP32-S3, provisional) -> USB-C -> Windows, as a composite USB HID device.
- **Display path:** Windows GPU -> HDMI -> off-the-shelf display driver board -> ~5-inch panel. Capacitive touch returns to the PC over USB.

There is no SBC and no video streaming in V1. The two paths are deliberately decoupled: the MCU never carries video, and the display subsystem does not depend on the firmware.

## 4. Accepted decisions D1-D8

Full entries with reasoning live in `docs/DECISIONS.md`. Summary:

| ID | Decision | Status |
|---|---|---|
| D1 | V1 feature tiers as recorded in `docs/V1-SPEC.md` | Agreed |
| D2 | Control inventory: 15 digital inputs + 6 analog channels | Frozen (sub-decision D2a open) |
| D2a | Stick arrangement: symmetric (PS) vs offset (Xbox) | **OPEN** — deferred by D9 to the pre-PCB mechanical gate |
| D3 | Display: HDMI driver board + off-the-shelf ~5in panel; no Linux SBC in V1 | Agreed |
| D4 | Connection: wired USB-C only in V1; wireless is OPTIONAL, post-core | Agreed |
| D5 | Power: USB bus-powered, no internal battery in V1 | Agreed, pending Stage 4 current measurement |
| D6 | MCU family: ESP32-S3 | **PROVISIONAL** — see below |
| D7 | Host identity: composite USB HID (gamepad + keyboard + mouse + config interface) | Agreed |
| D8 | Repository and documentation structure | Agreed |
| D9 | Ergonomic validation deferred to a pre-PCB mechanical gate | Agreed |
| D10 | Stage plan revision: merge old Stages 2-3, renumber to 15 stages | Proposed |

**D6 is provisional.** ESP32-S3 is confirmed only when a Stage 3 prototype demonstrates a stable composite USB HID device on Windows — a working gamepad interface, a working keyboard/mouse interface, and a working vendor configuration interface, simultaneously, surviving reconnect. Until that test passes, do not treat ESP32-S3 as final and do not create artifacts that would be expensive to port (custom PCB layout, production BOM). Documented fallback: RP2040 with an external ADC or analog multiplexer.

## 5. Frozen control inventory

15 digital inputs + 6 analog channels. Full table and HID mapping in `docs/CONTROL-INVENTORY.md`.

- Analog: 2 stick X/Y pairs (4), analog LT/RT (2)
- Digital: L3/R3 (2), D-pad U/D/L/R (4), ABXY (4), LB/RB (2), Start (1), Select (1), Profile (1)

No Home/Guide button — the Profile button occupies that central special-button role.

Two counts must never be confused:

- **15 electrical inputs** — what the pin map and PCB need.
- **10 buttons + 1 hat + 6 axes** — what Windows sees. The D-pad is reported as a single HID hat switch, not four buttons, and the Profile button is consumed by firmware and never reported to the host.

The inventory is deliberately XInput-shaped minus the Guide button, so a 1:1 mapping to a virtual Xbox 360 pad remains possible later without compromises. Do not add or remove controls in a way that breaks that mapping without an explicit decision.

Display: ~5-inch capacitive touchscreen, treated as a separate subsystem. Exact panel and final size are selected in Stage 4 against manufacturer datasheets.

## 6. Feature tiers

Authoritative list in `docs/V1-SPEC.md`. Enforce these boundaries.

- **MUST HAVE:** wired USB-C bus-powered operation; the full control inventory; custom controller PCB; enumerates as a working gamepad with no extra software; keyboard/mouse output mode; >=3 on-device profiles surviving power cycle, switched by the Profile button; companion app with device detection, live input viewer, remapping, and write-to-device; ~5in panel as a Windows second monitor; comfortable 3D-printed enclosure.
- **SHOULD HAVE:** touch input reported to the PC; deadzone, sensitivity curves and stick calibration stored on device; diagnostics view; automatic profile switching on foreground application; profile status LED.
- **OPTIONAL:** haptics; IMU/gyro; RGB lighting; BLE HID input with the display still tethered.
- **FUTURE REVISION:** internal battery and power management; Linux SBC + network video streaming; custom dual-screen demo application; PCB revision 2; advanced haptics.

## 7. Stage-gate development approach

Work proceeds one stage at a time. A stage closes only when its definition of done is met and evidence is recorded in `docs/TEST-LOG.md`. Stages are gated on completion, not on dates.

1. Project definition (closed 2026-09-10)
2. Controller input + USB HID prototype  <- ACTIVE
3. Display subsystem prototype
4. Companion application prototype + configuration protocol
5. Practice PCB
6. Mechanical/ergonomic validation + main PCB requirements and pin map
7. Main PCB design
8. PCB assembly and bring-up
9. Firmware V1
10. Companion application V1
11. Enclosure
12. System integration
13. Optional features
14. Final validation
15. Documentation and demo

**Current stage: 2 — controller input + USB HID prototype.**

## 8. Subsystems and dependencies

| ID | Subsystem | Boundary | Proven by |
|---|---|---|---|
| S1 | Input | Controls -> MCU pins -> debounced/scaled values | Stage 2 |
| S2 | Host interface | MCU -> USB HID -> Windows sees working gamepad/keyboard/mouse | Stage 3 |
| S3 | Display | PC HDMI -> driver board -> panel; touch -> PC | Stage 4 |
| S4 | Configuration | Companion app <-> MCU protocol; profiles persisted in flash | Stage 5 |
| S5 | Mechanical | Control layout, enclosure, mounting, cable exit | Stages 2 and 12 |
| S6 | Power | USB-C 5V -> 3V3 rail -> MCU, panel, driver board, inside the USB budget | Stage 7 |

Dependency rules, enforced:

- No main PCB layout until S1 and S2 are proven on development hardware.
- No pin map until the control inventory is frozen (it is) and the MCU is confirmed (D6).
- No main PCB outline, control mounting position, or enclosure geometry frozen, and no PCB ordered, until the physical ergonomic validation required by D9 has been performed and logged in docs/TEST-LOG.md.
- No enclosure CAD until panel, driver board and PCB outline dimensions are physically measured.
- No power component selection until measured current draw exists for MCU, panel and driver board.
- No integration of display and controller until each works independently.

## 9. Prototype before irreversible decisions

Anything expensive or slow to reverse — PCB fabrication, enclosure tooling, bulk part purchases, production BOM — requires a working prototype and a logged test result first. Simulation, datasheet reading and reasoning are inputs to a decision, never a substitute for a measured result.

## 10. Optional features must not destabilise the core

No OPTIONAL or FUTURE feature may be started while any MUST HAVE item is broken, unproven, or regressing. Optional work must not modify core input, HID, or profile code paths in ways that cannot be reverted cleanly.

## 11. Do not silently change major engineering decisions

Major decisions are: MCU and MCU family, host interface architecture, display architecture, power architecture, connection method, control inventory, PCB architecture and partitioning, profile storage format and configuration protocol, and locked mechanical dimensions.

Claude Code must not change any of these implicitly through code, schematic, layout or documentation edits. If an edit would alter one, stop and flag it.

## 12. How to propose a change to a major decision

Before implementing, state clearly:

1. **Proposed change** — from what, to what.
2. **Why** — the evidence forcing it, with the measurement, error, or datasheet reference.
3. **Affected systems and files** — which subsystems, documents, schematics, and source files change.
4. **What must be revalidated** — which previously passed tests are invalidated and must be re-run.

Wait for a human decision, then record it as a new numbered entry in `docs/DECISIONS.md`. Never edit a past decision entry to say something different; supersede it with a new one.

## 13. What Claude Code may do freely

Normal repository maintenance without asking: creating and updating documentation, keeping directory structure and internal links consistent, code edits and refactors within an agreed architecture, test scaffolding, build scripts, formatting, and appending entries to `docs/TEST-LOG.md` and `docs/FAILURES.md` from results provided by the team.

## 14. Project posture

Keep this project practical, testable, debuggable, and appropriately scoped for a first serious hardware build. Prefer off-the-shelf modules where custom design adds complexity without meaningful engineering value; the original work belongs in the PCB, firmware, companion software, integration, testing, and mechanical design. Resist feature creep. When two paths are viable, choose the one that fails earlier and more visibly.
