---
description: MOSFET sizing and parameter summary for 65nm custom CMOS process — RVT and LVT analog core devices, I/O 500nm devices, and digital switching parameters, characterized on Cadence Virtuoso and Spectre.
---

# MOSFET Sizes & Parameter Summary

This page serves as the central device reference for all designs documented on this site.
Unless mentioned explicitily, all circuits reference back to this page for transistor sizing and small-signal parameters.

!!! info "How to read these tables"
    - **Drawn dimensions** are in units of minimum feature size. For core 65nm devices, multiply by 65 nm to get actual dimensions. For example, W/L = 43.1 / 2 means W = 43.1 × 65 nm = 2.8 µm, L = 2 × 65 nm = 130 nm.
    - **All identifiers** follow standard analog IC design notation — subscripts are used throughout.
    - **NMOS and PMOS** parameters are shown side by side for direct comparison.

---

## Analog Models

### Core Device — 65 nm

**Process:** Custom 65nm CMOS &nbsp;·&nbsp; **Supply:** V~DD~ = 1.2 V &nbsp;·&nbsp; **L~min~** = 65 nm

#### Regular Threshold Voltage (RVT)

| Parameter | NMOS | PMOS | Comments |
|-----------|------|------|----------|
| I~D~ | 10 µA | 10 µA | Approximate, depends on operating point |
| W/L (Drawn) | 43.1 / 2 | 121.5 / 2 | Selected based on I~D~ and V~ov~ |
| W/L (Actual) | 2.8 µm / 130 nm | 7.9 µm / 130 nm | Absolute dimensions |
| V~DSsat~ | 60 mV | 60 mV | From R~out~ vs V~DS~ curve |
| V~ov~ | 70 mV | 70 mV | 5% of VDD is recommended for general design. Chosen slightly greater than that. |
| V~TH~ | 430 mV | 400 mV | From g~m~ curve of a 50/2 device (V~DS~ = 100 mV) |
| V~GS~ | 500 mV | 470 mV | With no body effect |
| V~sat~ | 126.5 * 10^3^ m/s | 124.4 * 10^3^ m/s | Carrier saturation velocity, From BSIM4 model |
| t~ox~ | 26 Å | 27.5 Å | Oxide thickness, From BSIM4 model |
| C'~ox~ | 13.3 fF/µm² | 12.6 fF/µm² | ε~ox~ = ε~r_SiO2~ * ε~0~ (ε~r_SiO2~ = 3.9 & ε~0~ = 8.85 * 10^-12^) |
| C~ox~ | 4.84 fF | 12.94 fF | C~ox~ = C'~ox~ * W * L (W,L - Actual dimensions) |
| C~gs~ | 3.23 fF | 8.63 fF | C~gs~ = 2/3 * C~ox~ |
| C~gd~ | 0.168 fF | 0.206 fF | C~gd~ = CGDO * W (CGDO~n~ = 59.9 pF/m & CGDO~p~ = 26.1 pF/m), CGDO from BSIM4 |
| g~m~ | 115.83 µA/V | 130.51 µA/V | At VDS = 0.1 V |
| r~out~ | 214.89 kΩ | 221.02 kΩ | At I~D~ = 10 uA |
| g~ds~ | 4.65 µA/V | 4.52 µA/V | 1/r~out~ |
| A~v-open~ | 24.89 V/V | 28.84 V/V | g~m~ * r~out~, Open Circuit Gain |
| λ | 0.46 V⁻¹ | 0.45 V⁻¹ | L = 2 |
| f~T~ | 5.5 GHz | 2.75 GHz | Approx at L = 2 |

---

#### Low Threshold Voltage (LVT)

| Parameter | NMOS | PMOS | Comments |
|-----------|------|------|----------|
| I~D~ | 10 µA | 10 µA | Approximate, depends on operating point |
| W/L (Drawn) | 38.5 / 2 | 93.8 / 2 | Selected based on I~D~ and V~ov~ |
| W/L (Actual) | 2.5 µm / 130 nm | 6.1 µm / 130 nm | Absolute dimensions |
| V~DSsat~ | 50 mV | 60 mV | From R~out~ vs V~DS~ curve |
| V~ov~ | 70 mV | 70 mV | 5% of VDD is recommended for general design. Chosen slightly greater than that. |
| V~TH~ | 350 mV | 350 mV | From g~m~ curve of a 50/2 device (V~DS~ = 100 mV) |
| V~GS~ | 420 mV | 420 mV | With no body effect |
| V~sat~ | 133 * 10^3^ m/s | 75.5 * 10^3^ m/s | Carrier saturation velocity, From BSIM4 model |
| t~ox~ | 26 Å  | 27.5 Å  | Oxide thickness, From BSIM4 model |
| C'~ox~ | 13.3 fF/µm² | 12.6 fF/µm² | ε~ox~ = ε~r_SiO2~ * ε~0~ (ε~r_SiO2~ = 3.9 & ε~0~ = 8.85 * 10^-12^) |
| C~ox~ | 4.32 fF | 9.99 fF | C~ox~ = C'~ox~ * W * L (W,L - Actual dimensions) |
| C~gs~ | 2.88 fF | 6.65 fF | C~gs~ = 2/3 * C~ox~ |
| C~gd~ | 0.204 fF | 0.104 fF | C~gd~ = CGDO * W (CGDO~n~ = 81.5 pF/m & CGDO~p~ = 17.0 pF/m), CGDO from BSIM4 |
| g~m~ | 121.41 µA/V | 116.21 µA/V | At VDS = 0.1 V |
| r~out~ | 130.25 kΩ | 118.14 kΩ | At I~D~ = 10 uA |
| g~ds~ | 7.68 µA/V | 8.46 µA/V | 1/r~out~ |
| A~v-open~ | 15.81 V/V | 13.73 V/V | g~m~ * r~out~, Open Circuit Gain |
| λ | 0.77 V⁻¹ | 0.85 V⁻¹ | L = 2 |
| f~T~ | 6.3 GHz | 3.0 GHz | Approx at L = 2 |

---

### I/O Device — 500 nm

**Process:** Custom 65nm CMOS &nbsp;·&nbsp; **Supply:** V~DD~ = 3.3 V &nbsp;·&nbsp; **L~min~** = 500 nm

!!! note
    I/O devices are used exclusively in large-signal designs (Bandgap References).
    Only large-signal parameters are characterized here — small-signal parameters are not tabulated.

| Parameter | NMOS | PMOS | Comments |
|-----------|------|------|----------|
| I~D~ | 10 µA | 10 µA | Approximate, depends on operating point |
| W/L (Drawn) | 5.8 / 1 | 13.8 / 1 | Selected based on I~D~ and V~ov~ |
| W/L (Actual) | 2.9 µm / 500 nm | 6.9 µm / 500 nm | Absolute dimensions |
| V~DSsat~ | 350 mV | 200 mV | From R~out~ vs V~DS~ curve |
| V~TH~ | 560 mV | 510 mV | From g~m~ curve of a 20/1 device (V~DS~ = 180 mV) |
| V~GS~ | 740 mV | 690 mV | With no body effect |

---

## Digital Models

### Core Device — 65 nm

**Process:** Custom 65nm CMOS &nbsp;·&nbsp; **Supply:** V~DD~ = 1.2 V &nbsp;·&nbsp; **L~min~** = 65 nm

#### Switching Parameters (General)

Switching resistance per unit width and oxide capacitance per unit area. For sizing.

| R~n~ (W Drawn) | R~p~ (W Drawn) | Scale Factor | C~oxn~ (W, L Actual) | C~oxp~ (W, L Actual) |
|----------------|----------------|--------------|----------------------|----------------------|
| 33.846 kΩ/W | 67.692 kΩ/W | 65 nm | 13.3 fF/µm² × W × L | 12.6 fF/µm² × W × L |

!!! info "How to use these values"
    W in the resistance columns is the **drawn** width in units of L~min~. W and L in the capacitance columns are **actual** dimensions in µm. For example, for an NMOS with W/L (Drawn) = 10/1: R~n~ = 33.846k / 10 = 3.4 kΩ, and C~oxn~ = 13.3 fF/µm² × 0.65 µm × 0.065 µm = 561 aF.

---

#### Switching Parameters (Chosen Size)

| Flavour | W/L (Drawn) | W/L (Actual) | R~n,p~ | C~oxn,p~ |
|---------|-------------|--------------|--------|----------|
| NMOS | 10/1 | 650 nm / 65 nm | 3.4 kΩ | 561 aF |
| PMOS | 20/1 | 1.3 µm / 65 nm | 3.4 kΩ | 1.06 fF |

---

## MOSFET as Capacitor

!!! note
    The control voltage applied at the gate of MOSFET (both NMOS and PMOS) is always referenced to GND as shown in Figure-01

![MOSFET Capacitor Configuration](./parameters-assets/01_MOS_Capacitor_light.svg#only-light)
![MOSFET Capacitor Configuration](./parameters-assets/01_MOS_Capacitor_dark.svg#only-dark)
/// caption
**Figure-01:** MOSFET as a capacitor to VDD and GND
///

### Core Device — 65 nm

**Process:** Custom 65nm CMOS &nbsp;·&nbsp; **Supply:** V~DD~ = 1.2 V &nbsp;·&nbsp; **L~min~** = 65 nm

| Flavour | Drawn Size | Actual Size | V~G~ | Nominal Capacitance |
|---------|------------|-------------|------|---------------------|
| NMOS-RVT | 10 / 10 | 650 nm / 650 nm | > 0.6 V | 5.2 fF |
| NMOS-RVT | 20 / 20 | 1.3 µm / 1.3 µm | > 0.6 V | 20.9 fF |
| PMOS-RVT | 10 / 10 | 650 nm / 650 nm | < 0.6 V | 5.2 fF |
| PMOS-RVT | 20 / 20 | 1.3 µm / 1.3 µm | < 0.6 V | 20.4 fF |