# Adaptive Handheld PC Controller

A custom handheld controller and second screen for Windows, whose buttons, sticks, triggers and touch surface remap themselves depending on which application is in the foreground.

The PC does all computation. The handheld provides physical controls, on-device profiles, and a ~5-inch touchscreen that Windows treats as a real second monitor.

**Status: Stage 2 of 15 — controller input + USB HID prototype.**

---

## Concept

An Xbox/PS4-style gamepad control set built around a ~5-inch touchscreen in the centre of the device, similar in overall concept to a Wii U GamePad, with its own design and identity. A single profile switch changes what every control does — a gamepad in a game, keyboard shortcuts in CAD software, media controls on the desktop.

## V1 scope

Wired, USB-C, bus-powered. Custom controller PCB, 3D-printed enclosure, on-device profiles, a Windows companion application, and a ~5-inch panel driven as a second monitor over HDMI.

Not in V1: battery, wireless, onboard Linux SBC, video streaming, haptics, gyro.

Full specification: [`docs/V1-SPEC.md`](docs/V1-SPEC.md)

## Controls

15 digital inputs, 6 analog channels: 2 analog sticks with L3/R3 click, a 4-way D-pad, ABXY, LB/RB, analog LT/RT, Start, Select, and a dedicated Profile button. No Home/Guide button.

Full table and HID mapping: [`docs/CONTROL-INVENTORY.md`](docs/CONTROL-INVENTORY.md)

## Architecture

Two independent connections to the PC — the controller path over USB, the display path over HDMI. See [`docs/BLOCK-DIAGRAM.md`](docs/BLOCK-DIAGRAM.md).

```
controls -> MCU -> USB-C -> Windows HID stack + companion app
Windows GPU -> HDMI -> driver board -> ~5in panel; touch -> USB
```

## Development approach

Stage-gated. Each stage closes on a definition of done backed by a logged test result, not on a date. Subsystems are proven independently before integration.

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

## Repository layout

```
docs/         specification, decisions, block diagram, control inventory, test log, failures
hardware/     schematic/  pcb/  datasheets/
firmware/     MCU firmware
software/     Windows companion application
mechanical/   cad/  prints/
tests/        test procedures and scripts
assets/       photos, renders, video
reference/    external material and notes
```

## Key documents

- [`CLAUDE.md`](CLAUDE.md) — persistent project context and working rules
- [`docs/V1-SPEC.md`](docs/V1-SPEC.md) — what V1 is and how we know it is done
- [`docs/DECISIONS.md`](docs/DECISIONS.md) — numbered engineering decision log
- [`docs/TEST-LOG.md`](docs/TEST-LOG.md) — every test run and its result
- [`docs/FAILURES.md`](docs/FAILURES.md) — what broke, why, and what changed

## Team

Two second-year Electrical Engineering students, Western University. Both work across hardware and software; roles rotate each stage and significant work is cross-reviewed.

## Non-goals

This project does not implement cheating functionality — no recoil scripts, rapid-fire automation, aim assistance, or automated gameplay. Demonstrations do not depend on copyrighted game content.
