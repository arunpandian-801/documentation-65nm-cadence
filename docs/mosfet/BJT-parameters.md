---
description: Vertical PNP parasitic BJT characterization summary for 65nm custom CMOS process — diode parameters for Cadence Virtuoso and Spectre simulations. For use in Bandgap Reference designs.
---

# Parasitic Vertical BJT Parameter Summary

## PNP Diodes

**Process:** Custom 65nm CMOS &nbsp;·&nbsp; **Supply:** V~DD~ = 1.2 V &nbsp;·&nbsp; **L~min~** = 65 nm

### PNP 20 X 20

| Parameter | Value | Comments |
|-----------|-------|----------|
| Emitter Area | 2x2 µm^2^ | Geometry info |
| η | 1003.6 m | Forward current emission coefficient for Emitter-Base junction (SPICE Model parameter, **NF**) |
| I~S~ | 0.285 aA | Saturation Current |
| V~BE~ | <ul> <li>0.777 V</li> <li>0.723 V</li> </ul> | <ul> <li>1 Diode</li> <li>8 Diodes in parallel</li> </ul> (At a current of 5 µA) |
| \(dV_{D}/dT\) | <ul> <li>-1.55 mV/°C</li> <li>-1.75 mV/°C</li> </ul> | Change in the diode’s voltage with temperature for: <ul> <li>1 Diode</li> <li>8 Diodes in parallel</li> </ul> (At current of 5 µA) |
