---
description: Conventional Bandgap Reference design procedure — series CTAT and PTAT voltage summation topology, 1.22V silicon bandgap output, 14.204 ppm/°C temperature coefficient, on I/O 500nm MOSFET, simulated on Cadence Virtuoso and Spectre.
---

--8<-- "includes/disclaimer.md"

# Series Realized Band Gap Reference (BGR)

## Prerequisites

This design shares methodology with the 
[Beta Multiplier Reference (BMR)](../references/bmr.md).
The following sections are directly applicable here and are not repeated:

- [Fixing Resistor Value](../references/bmr.md#fixing-resistor-value) — procedure to find resistor value using the substitution theorem
- [Startup Circuit Design](../references/bmr.md#startup-circuit-design) — considerations and procedure for desiging startup circuit
- [Using MOSFET as a capacitor to compensate the feedback loop](../references/bmr.md#using-mosfet-as-a-capacitor-to-compensate-the-feedback-loop) — NMOS capacitor vs PMOS capacitor and it's effects on startup

Read those sections before proceeding.

## Introduction

I have documented only the design procedure, and anything relevant to that. For theory of operation, please refer to standard textbooks or any lectures.

Before we proceed, let's take a look at the schematic of BGR.

<a id="fig-01"></a>

![Series BGR Schematic Diagram](./bgr-standard-assets/01_Schemtic_Series_BGR_dark.svg#only-dark)
![Series BGR Schematic Diagram](./bgr-standard-assets/01_Schemtic_Series_BGR_light.svg#only-light)
/// caption
**Figure-01:** Series BGR Schematic Diagram
///

The PNP BJTs are vertical parasitic components that can be used to generate a V~BE~ voltage (CTAT) in a CMOS process. One such diode is characterized in [PNP 20 X 20](../mosfet/BJT-parameters.md#pnp-20-x-20). The parameters listed there are what we will use to design the BGR, and as such, some necessary parameters for our design are summarised in *Table-01:*

<a id="table-01"></a>

| Parameter | Value | Comments |
|-----------|-------|----------|
| η | 1003.6 m | Forward current emission coefficient (Emitter-Base junction) |
| I~S~ | 0.285 aA | Saturation Current |
| V~BE~ | 0.777 V | 1 Diode (At a current of 5 µA) |
| \(dV_{D}/dT\) | -1.75 mV/°C | Change in the diode’s voltage with temperature (at current of 5 µA) for 8 Diodes in parallel |
/// caption
**Table-01:** PNP Diode summary
///

!!! question "Why 8 PNP Diodes in parallel? Why the number 8?"
    This is entirely due to the ΔVBE caused by the emitter area mismatch between a single PNP diode and `K` PNP diodes connected in parallel (with both having same current), where `K` is any integer.

    \[\Delta V_{BE} = \eta V_{T} \ln K\]

    From [PNP 20 X 20](../mosfet/BJT-parameters.md#pnp-20-x-20), we see that η = 1003.6m and V~T~ = 26 mV (at room temperature). Plugging it into the above equation yields:

    <a id="eqn-01"></a>
    
    \[\tag{1} \Delta V_{BE} = 1003.6m * 26m * \ln 8 = 54 mV\]

    Just 54 mV! For 8 PNP Diodes in parallel. Some people will even go for a K of 100, to get a ΔVBE of 120 mV. I know it is painful, but see that to double what we got before with just 8 diodes, now needs 100 diodes. Logarithmic functions are scary!

    !!! quote
        Because the logarithm function compresses its argument... if the argument is increased by a factor of ten, ΔVBE increases by only \(V_{T} * \ln 10 \approx 60 mV, ~\eta \approx 1\).

        \- *Ref. Analysis and design of Analog Integrated Circuits: Gray, Meyer, Hurst, Pg 324*

    That is why, I choose 8. Keeping 10 diodes is also a good choice (for a ΔVBE of 60 mV), but I am going to stick with 8.

For designing this, we need to find:

- Sizes for PMOS.
- Value of PTAT Current generator resistor, R~1~.
- Error amplifier topology plus sizes for MOSFETs used to build it.
- Value of PTAT voltage source resistor, R~2~.

The MOSFET sizes are already choosen and their parameters are thoroughly listed in [I/O Device — 500 nm](../mosfet/parameters.md#io-device-500-nm) table. Unless specified otherwise, these sizes will be the ones we use to construct the entire circuit.

And as such, these are tabulated here for convenience:

<a id="table-02"></a>

| Flavour | W/L (Drawn) | W/L (Actual) | I~D~ | V~GS~ |
|---------|-------------|--------------|------|-------|
| NMOS | 5.8 / 1 | 2.9 µm / 500 nm | 10 µA | 740 mV |
| PMOS | 13.8 / 1 | 6.9 µm / 500 nm  | 10 µA | 690 mV |
/// caption
**Table-02:** Sizes and Bias Current summary
///

!!! question
    The observant reader would have questioned these sizing choices considering [Table-01](#table-01) characterizes PNP Diodes for 5 µA while here it is characterized for 10 µA. But still, follow along. These sizes are chosen for a general analog design. Just to tease you on how these two gets tied up, the error-amp steps in to adjust the V~SG~ of top PMOS in [Figure-01](#fig-01) to reduce it's current to 5 µA.

This leaves us with values for resistors R~1~ and R~2~ and the error amplifer.

## Fixing PTAT Current Generator resistor, R~1~

[Prerequisites](#prerequisites) section has already listed the knowledge needed to choose a suitable value for this resistor. Review the listed sections to grasp the reasoning behind the testbench employed to determine the value of resistor, R~1~.

The amplifier and the feedback loop will:

1. Force the same voltage across the single diode on the left as well as the resistor R~1~ and eight diodes on the right.
2. Both of these branches will conduct 5 µA.

Using [Substitution theorem](../references/bmr.md#substitution-theorem), the testbench is built and can be seen in *Figure-02*.

![PTAT Resistor R1 TB](./bgr-standard-assets/02_PTATResistor_TB_dark.png#only-dark)
![PTAT Resistor R1 TB](./bgr-standard-assets/02_PTATResistor_TB_light.png#only-light)
/// caption
**Figure-02:** PTAT Current Generator resistor R~1~ value Testbench
///

Notice how we got 54.3 mV, which is very close to what is predicted by [Equation-01](#eqn-01).

And so, with this voltage, to set a current of 5 µA, we need a resistor of:

\[R_{1} = \frac{\Delta V_{BE}}{I} = \frac{54.3m}{5\mu} = 10.86 k\Omega \approx 10.9 k\Omega\]

Now all we need is the error amplifier.

## Added error amplifier

We need to pick a topology for the error amplifier. Let's go for a single stage diff-amp because:

- It will have sufficient gain (Given L~min~ of I/O MOSFET is 500nm. See [Table-02](#table-02))
- It will be easy to compensate.

!!! tip
    Always start with the simplest topology possible. If it proves insufficient, you can always add another stage or modify it.

### NMOS input or PMOS input ?

#### Constraint on output common mode

At steady state, both inputs of error-amp will settle to same voltage due to negative feedback loop (or close to each other based on DC Gain). And so, ***with just a common-mode voltage, the error-amp is expected to set bias voltage for Current sourcing PMOS*** in [Figure-01](#fig-01).

And looking at [Table-02](#table-02), we find that the V~SG~ of PMOS for the chosen size is 690 mV to source 10 µA. Our design has chosen 5 µA, and so the V~SG~ needed will be **slightly lower than 690 mV** for the same size.

But nevertheless, using this value, the steady voltage at gate of PMOS needed for this V~SG~ is given by

\[\tag{Rough estimate} V_{G,P} = V_{DD} - V_{SG,P} = 3.3 V - 0.69 V = 2.61 V\]

And **2.61 V is close to V~DD~ of 3.3 V** for 10 µA. And it will be ***even closer for 5 µA as V~SG~ would have reduced for the same size.***

So, the observation from here is:

!!! info "Constraints on Error amplifier output common mode"
    *Our amplifier output common mode should be close to V~DD~.*

![NMOS vs PMOS output CM](./bgr-standard-assets/03_NMOS_PMOS_ErrorAmp_dark.svg#only-dark)
![NMOS vs PMOS output CM](./bgr-standard-assets/03_NMOS_PMOS_ErrorAmp_light.svg#only-light)
/// caption
**Figure-03:** Output Common Mode of NMOS and PMOS input diff-amp
///

From *Figure-03*, we could be inclined to think it's an NMOS input differential amplifier, but let's not jump to conclusions yet—there's still another issue to consider.

#### Inherent constraint on Input common mode of NMOS and PMOS Diff-amp

What exactly is the common mode voltage we expect to see? Looking at [Table-01](#table-01), we see that V~BE~ is 0.777 V for a current of 5 µA (single diode). And this is the value generated at left side single diode which is also copied to right side due to feedback loop (See [Figure-01](#fig-01)).

Clearly 0.777 V is close to GND (0 V) than V~DD~ (3.3 V). And so,

!!! info "Constraints on Error amplifier ICMR"
    *Our amplifier input common mode range should include 0.777 V which is close to GND.*

![NMOS vs PMOS ICMR](./bgr-standard-assets/04_ICMR_NMOS_PMOS_ErrorAmp_dark.svg#only-dark)
![NMOS vs PMOS ICMR](./bgr-standard-assets/04_ICMR_NMOS_PMOS_ErrorAmp_light.svg#only-light)
/// caption
**Figure-04:** ICMR of NMOS and PMOS input diff-amp
///

So to sense a common mode voltage near to GND, this time we could incline towards a PMOS.

But that *contradicts with our output common constraint*.

### Possible modifications to Diff-amp

An NMOS input diff-amp can properly bias up the PMOS current source. But a PMOS input diff-amp can sense a near ground Common Mode voltage.

So the simplest modification we can do is to ***add level shifters.*** That is:

- Shift the input of NMOS input diff-amp upwards towards it's ICMR.
- Shift the output of PMOS input diff-amp upwards towards V~DD~.