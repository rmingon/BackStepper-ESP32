# BackStepper-ESP32

BackStepper-ESP32 is a custom control board that bolts onto the rear face of a NEMA17 stepper motor and embeds an ESP32 for closed-loop or network-connected motion projects. The PCB combines power regulation, a DRV8825 microstepping driver, and a USB-C interface so a single board can power, drive, and program the motor controller without additional wiring harnesses.

> Designed with KiCad 9.0 (2025 snapshot); all project files live under `hardware/`.

## Visuals

![3D top render of the BackStepper-ESP32 PCB](<3d_top.png>)

![3D bottom render showing the NEMA17 mounting face](<3d_bottom.png>)

![Layer view of the assembled PCB layout](<pcb.png>)

![High-level schematic overview](<schematic.png>)

## Highlights

- ESP32-WROOM-32 module exposed for Wi-Fi/Bluetooth firmware as well as classic STEP/DIR motion control.
- DRV8825 stepper driver with 1/32 microstepping support and dedicated test pads on the STEP and DIR nets.
- CH340C USB-to-UART connected to a USB-C receptacle for flashing firmware and optionally powering the board.
- AMS1117-3.3 linear regulator fed from the +5 V plane to supply the ESP32 module and logic.
- JST-XH connectors for motor phases and the external 5 V feed, plus a 1×6 2.54 mm header for auxiliary serial access.
- Boot push-button on IO0 and CH340-controlled RTS/DTR FETs for hands-free flashing with `esptool.py`.
- Status LEDs on ESP32 IO12 and IO13 that can be repurposed for application feedback.
- Mounting hole pattern sized for the M3 boss spacing on a NEMA17 housing so the board can be screwed directly to the motor.

## Core Blocks

- **ESP32 module (`U2`)** – Provides dual-core MCU with Wi-Fi / BLE. STEP is on `IO4`, DIR on `IO5`, and UART0 is routed both to the CH340C and the breakout header.
- **Power regulation (`U3`)** – AMS1117-3.3 drops the shared +5 V plane down to 3.3 V for the ESP32 and CH340. Keep input below 6 V; logic and motor power are tied together on +5 V in this revision.
- **USB interface (`J2` + `U1`)** – USB-C receptacle wired for USB 2.0. CH340C handles the USB-to-UART bridge with RTS/DTR driving BSS138 FETs (`Q1`, `Q2`) to toggle EN and IO0 for automatic bootloader entry.
- **Stepper driver (`U4`)** – TI DRV8825 in HTSSOP package. Solder jumpers select the microstepping mode and sense resistor network sets coil current (see schematic for chosen values). Outputs route straight to the JST-XH motor connector.
- **User I/O** – `BTN1` pulls IO0 to ground for manual boot, status LEDs (`D1`, `D2`) sit on IO13 and IO12, and `TP1`/`TP2` break out STEP and DIR for probing or external drive injection.

## Connectors & Controls

| Ref  | Type / Pitch | Pinout (1 → n) | Notes |
|------|--------------|----------------|-------|
| `J2` | USB-C (USB 2.0) | VBUS, D-, D+, CC1/CC2, shield | Powers CH340C/ESP32 from 5 V VBUS and provides the programming link. |
| `J4` | JST-XH 1×2 vertical | 1: GND · 2: +5 V | Primary power input. In this design +5 V feeds both DRV8825 VMOT and the AMS1117 regulator. |
| `J3` | JST-XH 1×4 vertical | 1: AOUT2 · 2: AOUT1 · 3: BOUT1 · 4: BOUT2 | Connect the stepper phases A/B (watch the order to match motor datasheet). |
| `J1` | 1×6 2.54 mm header | 1: NC · 2: RX0 · 3: TX0 · 4: +5 V · 5: NC · 6: GND | Aux UART header for programming/debug or external peripherals. |
| `BTN1` | Tactile push-button | IO0 ↔ GND | Forces the ESP32 boot pin low; use with EN toggle to enter download mode. |
| `TP1` | Test pad | STEP | Logic probe / scope pad tied to DRV8825 STEP and ESP32 IO4. |
| `TP2` | Test pad | DIR | Logic probe / scope pad tied to DRV8825 DIR and ESP32 IO5. |

## Microstepping Jumpers

Three solder jumpers strap DRV8825 `MODE0..2`. Each jumper has **GND** on the silkscreened dot side (pad A) and **3.3 V** on the opposite side (pad B). Leaving a jumper open lets the DRV8825’s internal pulldown pull the mode low.

| Microstep mode | `JP1` (MODE0) | `JP2` (MODE1) | `JP3` (MODE2) |
|----------------|---------------|---------------|---------------|
| Full step      | open (low)    | open (low)    | open (low)    |
| 1/2 step       | short to 3.3 V| open          | open          |
| 1/4 step       | open          | short to 3.3 V| open          |
| 1/8 step       | short to 3.3 V| short to 3.3 V| open          |
| 1/16 step      | open          | open          | short to 3.3 V|
| 1/32 step      | short to 3.3 V| open          | short to 3.3 V|
| 1/32 step      | open          | short to 3.3 V| short to 3.3 V|
| 1/32 step      | short to 3.3 V| short to 3.3 V| short to 3.3 V|

Reflow a small solder bridge between the center pad and the desired side to change the logic level. Remove bridges with solder wick when changing configuration.

## Powering and Using the Board

1. **Supply** – Feed +5 V either through the USB-C port (VBUS) or the `J4` JST connector. Because the DRV8825 VM pins are tied to +5 V, this revision targets low-voltage steppers; do not exceed 5–6 V on the supply without redesigning the power stage.
2. **Motor hookup** – Wire the stepper to `J3`. If the motor runs rough, swap pins within a phase pair to keep coil polarity consistent.
3. **Programming** – Connect USB-C to your PC. The CH340C enumerates as a serial port. `esptool.py` can reset the ESP32 automatically via RTS/DTR. Use `BTN1` only if manual boot selection is needed.
4. **Firmware** – STEP/DIR live on IO4/IO5; UART1 (IO19/IO21) is broken out internally if you need another serial channel. LEDs on IO12/IO13 provide quick visual feedback.

Current sense resistors and VREF test points are accessible near the driver; tune current following the DRV8825 datasheet.

## Repository Layout

- `hardware/` – KiCad project (`.pro`, `.sch`, `.kicad_pcb`) plus backup caches.
- `Capture d’écran *.png` – Draft renders / screenshots of the PCB as designed in the IDE.

Generate manufacturing outputs from KiCad (`File → Fabrication Outputs`) against the `hardware.kicad_pro` project once component values are finalised.

## License

This project is released under the **CERN Open Hardware Licence v2 Strongly Reciprocal (CERN-OHL-S-2.0)**. See the `LICENSE` file for the full text and your obligations when modifying or distributing the design or fabricated hardware.

## Next Steps

- Add a dedicated EN/RESET push-button if manual resets become necessary.
- Consider isolating motor and logic rails or moving to a higher-voltage supply for more torque.
- Document tested firmware builds and pin assignments once the mechanical integration is validated.

Feel free to open issues or PRs as the design evolves. Happy hacking!
