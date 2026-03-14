---
description: General Bias Circuit design procedure — generates cascode current source and sink bias voltages plus Class AB output stage bias, derived from Beta Multiplier Reference (BMR) on 65nm custom CMOS, simulated on Cadence Virtuoso and Spectre.
---

--8<-- "includes/disclaimer.md"

# General Bias Circuit

## Introduction

I have documented only the design procedure, and anything relevant to that. For theory of operation, please refer to standard textbooks or any lectures.

Before we proceed, let's take a look at the schematic of General-bias circuit.

![Bias Circuit Schematic diagram](./general-bias-assets/13_GeneralBiasCircuit_dark.svg#only-dark)
![Bias Circuit Schematic diagram](./general-bias-assets/13_GeneralBiasCircuit_light.svg#only-light)
/// caption
**Figure-01:** General bias circuit schematic diagram [Ref. CMOS Circuit Design, Layout and Simulation, Fig 20.47]
///

The upper half is the [Beta Multiplier Reference](/references/bmr/) and it's design is already covered seperately in it's own page. The lower half uses BMR to generate wide swing cascode current sink/sources and bias voltages for class-AB output stage.

For designing this, we need to choose values for two things:

- MOSFET Sizes for Cascode bias Voltage generation
- MOSFET Sizes for class-AB bias voltage generation

Since we have already designed the [BMR](/references/bmr/) for 10 µA, the general bias circuit will also generate bias voltages for 10 µA. For different current, **we have to redesign the BMR** as simply using a different size for a specific current will result in mismatch.

!!! tip
    It is strongly recommended that analog designers use only a small set of “unit-sized” transistors, forming all transistors from parallel combinations of these elementary devices.

    \- *Ref. Analog Integrated Circuit design: Tony Chan Caruson, David A. Johns and Kenneth W. Martin, Pg 51*

