# V1 Specification

**Status:** Stage 1, open. Feature tiers agreed 2026-09-10 (D1). Control inventory frozen (D2).
**Owner:** both team members. Changes require a new entry in `DECISIONS.md`.

---

## 1. Product

A tethered handheld PC peripheral for Windows: an Xbox/PS4-style gamepad control set built around a ~5-inch touchscreen in the centre of the device, conceptually similar to a Wii U GamePad but with its own design and identity.

The defining feature is adaptivity — the mapping of every physical control, and eventually the contents of the screen, changes with the PC application in use.

The PC performs all computation. The handheld is an input device and a display surface, not a computer.

## 2. V1 definition

A **wired, USB-C, bus-powered handheld** with:

- the full control inventory in `CONTROL-INVENTORY.md`
- a custom controller PCB carrying the MCU and all control interfaces
- a ~5-inch touchscreen driven as a genuine Windows second monitor over HDMI
- on-device profiles
- a Windows companion application
- a 3D-printed enclosure

V1 explicitly excludes: internal battery, wireless operation, onboard Linux SBC, network video streaming, haptics, IMU/gyro.

## 3. Feature tiers

### MUST HAVE — V1 is not complete without every item

| # | Requirement | Acceptance criterion |
|---|---|---|
| M1 | Wired USB-C connection, bus-powered | Device operates from a single PC USB-C port with no external supply |
| M2 | Full control inventory | All 15 digital inputs and 6 analog channels read correctly and independently |
| M3 | Custom controller PCB | The MCU and all control interfaces are on a board we designed, ordered and assembled |
| M4 | Works as a gamepad with no extra software | A clean Windows install recognises the device and it plays a real game |
| M5 | Keyboard/mouse output mode | Physical controls emit keystrokes and mouse motion |
| M6 | On-device profiles | >=3 profiles stored on device, switched by the Profile button, surviving power cycle |
| M7 | Companion application | Detects the device, shows live inputs, remaps any control, writes the profile to the device |
| M8 | Second display | Windows extends the desktop onto the ~5in panel at its native resolution |
| M9 | Enclosure | 3D-printed shell holds all hardware and is comfortable to hold for 30 continuous minutes |

### SHOULD HAVE

- Touchscreen input reported to the PC (absolute mouse or Windows touch)
- Per-axis deadzone, sensitivity curve and stick calibration stored on device
- Diagnostics view in the companion app: raw ADC values, firmware version, error state
- Automatic profile switching based on the foreground Windows application
- Status LED indicating the active profile

### OPTIONAL — only after every MUST HAVE item is stable

- Haptics (one or two actuators)
- 6-axis IMU / gyro aiming
- RGB lighting
- Wireless (BLE HID) input, with the display still tethered

### FUTURE REVISION

- Internal battery and power management
- Linux SBC and network video streaming for a fully wireless device
- Custom dual-screen demonstration application
- PCB revision 2, advanced haptics, improved enclosure

## 4. Interfaces to the PC

Two independent connections in V1:

| Path | Medium | Carries |
|---|---|---|
| Control | USB-C | Composite USB HID: gamepad, keyboard, mouse, and a vendor configuration interface |
| Display | HDMI + USB | Video to the panel; capacitive touch back to the PC |

Reducing this to a single cable is out of scope for V1.

## 5. Constraints

- Total V1 hardware budget: approximately $150-350
- No hard deadline; stages close on definition of done, not dates
- Available equipment: soldering iron and hand tools, multimeter, 3D printer, oscilloscope/logic analyzer
- Two team members, both working across hardware and software

## 6. Non-goals

No cheating functionality of any kind — no recoil scripts, rapid-fire automation, aim assistance, or automated gameplay. Public demonstrations must not depend on copyrighted game content.

## 7. Definition of done for V1

Every MUST HAVE acceptance criterion in section 3 passes and is recorded in `TEST-LOG.md`, with the device fully assembled in its enclosure and operated for an extended session without failure.
