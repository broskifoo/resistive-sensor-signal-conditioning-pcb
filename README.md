# Resistive Sensor Signal-Conditioning PCB

A small 2-layer KiCad PCB that reads a 1 kΩ to 10 kΩ resistive sensor and outputs a voltage for a 3.3 V microcontroller ADC. It runs from an external 5 to 12 V DC supply and makes its own 3.3 V rail.

> **Status:** schematic and layout finished, ERC and DRC clean (0 violations, 0 unconnected items, schematic parity OK). **Not yet fabricated or tested on hardware.**

## What it does

1. **Power:** a TI LMR36510ADDA buck regulator turns the 5 to 12 V input into 3.3 V. The feedback divider is 100 kΩ / 43.2 kΩ with the 1 V feedback reference from the datasheet, which gives about 3.315 V.
2. **Sensor divider:** 3.3 V → R1 (10 kΩ) → sense node → sensor → GND. The sense voltage is about 0.30 V at 1 kΩ and 1.65 V at 10 kΩ.
3. **Scaling:** half of a TI TLV9002 dual op-amp (U1A) is a non-inverting amplifier with gain 2.3 and an offset. The other half (U1B) buffers a reference voltage of about 0.458 V made by a 62 kΩ / 10 kΩ divider from 3.3 V.
4. **Filter:** a 1 kΩ + 1 µF low-pass filter (about 159 Hz cutoff) sits at the output.

Ideal output: `Vout = 2.3 × Vsense − 1.3 × Vref`

| Sensor resistance | Sense voltage | Output voltage |
|---|---|---|
| 1 kΩ | 0.30 V | about 0.09 V |
| 10 kΩ | 1.65 V | about 3.20 V |

The output range is kept slightly inside 0 to 3.3 V because the TLV9002 output cannot swing all the way to the rails. Use 1% resistors. Real values will shift a little with tolerance.

## Connectors

| Connector | Pin 1 | Pin 2 | Pin 3 |
|---|---|---|---|
| J1 (sensor) | Sense node | GND | - |
| J2 (output) | Signal out (to ADC) | 3.3 V | GND |
| J3 (power in) | +5 to 12 V | GND | - |

## Main parts

| Ref | Part |
|---|---|
| U1 | TLV9002IDR dual op-amp, SOIC-8 |
| U2 | LMR36510ADDA buck regulator |
| L1 | 22 µH inductor (footprint: Bourns SRN6045TA, see to-do list) |
| C3, C4 | Input capacitors, 2.2 µF and 220 nF |
| C7, C8, C9 | 22 µF output capacitors |
| R, C (others) | 0805 SMD |

## Board notes

- 2 layers: signals and power on top, a continuous GND plane on the bottom.
- GND pads connect to the plane through vias. U2's exposed pad has thermal vias.
- The 3.3 V trace to the sensor side hops under another trace on the bottom layer.
- Trace widths: 0.5 mm for power, 0.25 mm for small signals.

## Files

| File | What it is |
|---|---|
| `Resistive_Sensor_PCB.kicad_sch` | Schematic |
| `Resistive_Sensor_PCB.kicad_pcb` | PCB layout |
| `Resistive_Sensor_PCB.kicad_pro` | KiCad project file |
| `gerbers/` | Gerber and drill files for manufacturing |

Made with KiCad 10.0.7.

## To do before ordering

- [ ] Confirm the L1 footprint matches the exact inductor that will be bought (size and current rating).
- [ ] Use capacitors rated at least 25 V for C3 and C4 (they sit on the 12 V input).
- [ ] Remove the leftover `TLV9002` project footprint library entry (it points to a file on the author's computer and is not used).
- [ ] Build one board and measure the output at 1 kΩ and 10 kΩ.
