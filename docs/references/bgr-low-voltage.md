---
description: Low Voltage Bandgap Reference design procedure — parallel CTAT and PTAT current summation topology, 0.61V output at VDD/2, 28.265 ppm/°C temperature coefficient, on core 65nm MOSFET with 1.2V supply, simulated on Cadence Virtuoso and Spectre.
---

--8<-- "includes/disclaimer.md"

# Parallel Realized Band Gap Reference (BGR)

## Prerequisites

This design shares substantial methodology with the [Series Bandgap Reference](../references/bgr-standard.md). The core distinction is the **quantity** being temperature-stabilized — the Series BGR stabilizes **voltage** (series CTAT+PTAT voltage summation), while this design stabilizes **current** (parallel CTAT+PTAT current summation).

The following sections from the Series BGR apply directly and are not repeated:

- [Fixing PTAT Current Generator resistor, R~1~](../references/bgr-standard.md#fixing-ptat-current-generator-resistor-r1) — identical procedure, same operating point (5 µA), same parasitic PNP diodes. Resistor value of 10.9 kΩ carries over directly.
- [Added error amplifier](../references/bgr-standard.md#added-error-amplifier) — same operating point (5 µA) and  same V~BE~ of 0.777 V. However, 0.777 V sits close to V~DD~ here (1.2 V core supply) unlike the Standard BGR (3.3 V I/O supply), giving rise to slight but important differences as discussed in the main body of this page.
- [Startup Circuit Design](../references/bmr.md#startup-circuit-design) — design considerations and procedure are identical to the Series BGR and BMR.

Read the [Series BGR](../references/bgr-standard.md) documentation before proceeding.