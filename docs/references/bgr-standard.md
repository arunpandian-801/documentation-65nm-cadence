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
**Figure-01:** Series BGR Schematic Diagram (Startup not shown)
///

The PNP BJTs are vertical parasitic components that can be used to generate a V~BE~ voltage (CTAT) in a CMOS process. One such diode is characterized in [PNP 20 X 20](../mosfet/BJT-parameters.md#pnp-20-x-20). The parameters listed there are what we will use to design the BGR, and as such, some necessary parameters for our design are summarised in *Table-01:*

<a id="table-01"></a>

| Parameter | Value | Comments |
|-----------|-------|----------|
| η | 1003.6 m | Forward current emission coefficient (Emitter-Base junction) |
| I~S~ | 0.285 aA | Saturation Current |
| V~BE~ | 0.777 V | 1 Diode (At a current of 5 µA) |
| \(\delta V_{D}/\delta T\) | -1.75 mV/°C | Change in the diode’s voltage with temperature (at current of 5 µA) for 8 Diodes in parallel |
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

- a Size for current sourcing PMOS.
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

<a id="fig-03"></a>

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

But that *contradicts with our output common mode constraint*.

### Possible modifications to Diff-amp

An NMOS input diff-amp can properly bias up the PMOS current source. But a PMOS input diff-amp can sense a near ground Common Mode voltage.

So the simplest modification we can do is to ***add level shifters.*** That is:

- Shift the input of NMOS input diff-amp upwards towards it's ICMR.
- Shift the output of PMOS input diff-amp upwards towards V~DD~.

![Diff amps with level shifters](./bgr-standard-assets/05_LevelShifterPossbilities_dark.svg#only-dark)
![Diff amps with level shifters](./bgr-standard-assets/05_LevelShifterPossbilities_light.svg#only-light)
/// caption
**Figure-05:** Diff-amps with appropriate level shifters
///

*Figure-05* shows both possibilities. Both can be used as the feedback action will adjust the node voltages to settle to a steady state.

But, I am going to ***prefer NMOS variant*** for the following reasons:

1. NMOS Diff-amp produces \(V_{DD} - V_{SG}\) voltage by default with just common mode input. Very useful to directly bias the current source PMOS. 

    ??? question "What happens with a PMOS input Diff-amp?"
        Had we used PMOS Diff-amp, there is a chance for some off-set to develop between the regulated nodes to push the output up to attain steady state. Think about it, Diff-amp operates using \(V_{out} = A_{D} * (V_{+} - V_{-})\). So to shift output upwards, you need to develop a non-zero difference between PLUS and MINUS terminals.

        Afterall, we can't set the output of this variant very close to a value unlike the NMOS input case.

2. Load of NMOS Diff-amp itself being a PMOS helps to improve matching with the current source PMOS.
3. Output is a high impedance, and can be easily compensated, by making it the dominant pole.

    ??? question "What to do with PMOS input Diff-amp?"
        You compensate by making the output of Diff-amp the dominant pole, and not the level shifted output. Level shifter presents low impedance at the output and would require large capacitors to compensate. Meanwhile, NMOS output itself is high-impedance node, and it is attached to gate terminals of Current source PMOS which is also high impedance and themselves presents some load, making it easy to compensate.

### Bias-leg Design

Looking at [Table-02](#table-02), the V~GS~ and V~SG~ of NMOS and PMOS are 740 mV and 690 mV. And the supply is 3.3 V. *So, we need to achieve a total voltage drop of 3.3 V using multiple V~GS~ or V~SG~ diode-connected drops (either 740 mV or 690 mV), depending on the MOSFET used and how many times it is utilized*.

Let's do the math. We can see that if I used 2 NMOS and 2 PMOS in diode connected configuration as my bias leg, that adds up to:

<a id="eqn-02"></a>

\[\tag{2} V_{SUP,min} = 2 * V_{GS,n} + 2 * V_{SG,p} = 2 * (740m + 690m) = 2.86 V\]

That's almost amounts to supply of 3.3 V and clearly cannot accomodate anymore V~GS~ drops in it. The DC annotated schematic of such a bias leg is shown in *Figure-06*.

<a id="fig-06"></a>

![four Diode MOS Bias leg](./bgr-standard-assets/06_01_BiasLeg_4DiodeMOS_dark.png#only-dark)
![four Diode MOS Bias leg](./bgr-standard-assets/06_01_BiasLeg_4DiodeMOS_light.png#only-light)
/// caption
**Figure-06:** Two NMOS and two PMOS diode connected bias leg
///

Clearly, the increase in reference current is due to the excess supply voltage left from what we computed in [Equation-02](#eqn-02). We computed a minimum supply of 2.86 V and that leaves \(V_{excess} = V_{DD} - V_{DD,min} = 3.3 - 2.86 = 0.44 V\) which is distributed to all MOSFETs which results in increase of current.

#### Decreasing Minimum supply

We don't need to constrain ourselves to 2.86 V as that is the minimum supply needed for 10 µA. **This is an error-amp, and we don't care about it's small signal parameter variation with bias current.**

The supply can go even lower than 2.86 V by generating a current lesser than 10 µA. But, there is an absolute limit dictated by the threshold voltage, beneath which you will reach the subthreshold region. That's a no go. **Biasing in the subthreshold will introduce horrible threshold mismatch, which will manifest as off-set at the input. We don't want that either.**

With this in mind, let's compute the minimum supply which is barely enough to keep all MOSFETs ON. Looking at [I/O Device — 500 nm](../mosfet/parameters.md#io-device-500-nm) table, we see that the threshold voltage of NMOS and PMOS are 560 mV and 510 mV.

<a id="eqn-03"></a>

\[\tag{3} V_{DD,min} = 2 * V_{TH,N} + 2 * V_{TH,P} = 2 * 0.56 + 2 * 0.51 = 2.14 V\]

!!! warning
    This is an overestimate. Looking at [Figure-06](#fig-06), both middle MOSFETs are suffering from body effect and their increase in threshold voltage is not accounted in *Equation-03*. It means, that the minimum limit will be slightly higher than 2.14 V.

We will see later that the actual limit is 2.63 V (See *Figure-07* for DC annotated schematic at 2.63 V supply and [Figure-09](#fig-09) for finding this limit) from simulations. But that is just 0.7 V below supply. Not even half the supply.

![four diode MOS minimum Supply bias](./bgr-standard-assets/06_03_BiasLeg_4DiodeMOS_VSUPmin_2600m_dark.png#only-dark)
![four diode MOS minimum Supply bias](./bgr-standard-assets/06_03_BiasLeg_4DiodeMOS_VSUPmin_2600m_light.png#only-light)
/// caption
**Figure-07:** Four diode connected MOSFET at minimum possible supply
///

Notice that in *Figure-07*, both the middle MOSFETs are in sub-threshold (***Region=3***).

Now, we have two options:

- Accept 2.63 V as the minimum supply for error-amp and thus the reference.
- Decrease it even further by removing a V~GS~ drop.

Let's explore the second option, because I like to aim for a minimum supply close to V~DD~\/2.

#### Removing 1 V~GS~ drop

If we removed a V~GS~ (or 1 diode connected MOSFET) drop from [Equation-02](#eqn-02), we can go even lower than 2.63 V limit dictated by threshold voltage.

We can remove middle PMOS or middle NMOS, but, I am going to remove middle PMOS as the middle NMOS can be used as a cascode bias reference for current sink used in Diff-amp (Follow along, you will see this shortly).

\[\tag{4} V_{DD,min} = 2 * V_{TH,N} + V_{TH,P} = 2 * 0.56 + 0.51 = 1.63 V\]

Atleast now we can theoretically expect a minimum supply close to V~DD~/2.

<a id="fig-08"></a>

![Three diode MOS bias leg](./bgr-standard-assets/06_02_BiasLeg_3DiodeMOS_dark.png#only-dark)
![Three diode MOS bias leg](./bgr-standard-assets/06_02_BiasLeg_3DiodeMOS_light.png#only-light)
/// caption
**Figure-08:** Two NMOS and one PMOS diode connected bias leg
///

The construction of such a bias leg is shown is *Figure-08*. The bias current generated is 31.5 µA for supply of 3.3 V.

!!! note
    Notice that the length of bottom transistor is increased to four times by keeping four NMOS in series, and middle one is increased to two times similarly.

    This is made to reduce the bias current generated. Had I used just the minimum length, the bias current would have increased, considering now it needs to split 3.3 V between just 3 MOSFETs.

??? note
    I did this roundabout way, because I got lazy in picking another size with longer length. Keeping MOSFETs in series to increase length is perfectly valid. [*Ref. CMOS Circuit Design, Layout and Simulation, Fig 20.35*]

#### Bias Leg Performance across supply

Sweeping the supply for [Figure-06](#fig-06) and [Figure-08](#fig-08) bias leg configurations over 2.5 V to 3.3 V yields *Figure-09*.

<a id="fig-09"></a>

![Supply dependance of Bias Leg](./bgr-standard-assets/07_ID_Vs_VSUP_BiasLeg_dark.svg#only-dark)
![Supply dependance of Bias Leg](./bgr-standard-assets/07_ID_Vs_VSUP_BiasLeg_light.svg#only-light)
/// caption
**Figure-09:** Performance of Bias Leg of [Figure-06](#fig-06) and [Figure-08](#fig-08) over supply
///

Clearly, [four diode connected bias leg](#fig-06) touches nA range (1 µA is the border to nA range) at 2.63 V while the [three diode connected bias leg](#fig-08) is still in µA range, i.e., all MOSFETs are still ON.

#### Current Sink from final bias leg (three diode connected)

![Cascode Current Sink TB](./bgr-standard-assets/08_01_Simple_vs_Cascode_TB_dark.png#only-dark)
![Cascode Current Sink TB](./bgr-standard-assets/08_01_Simple_vs_Cascode_TB_light.png#only-light)
/// caption
**Figure-10:** Cascode Current Sink biased by bias Leg of [Figure-08](#fig-08) Test bench
///

The performance of current sink (both simple and cascoded) seen *Figure-11* shows that ***both cascode and simple current sink needs the same minimum voltage of 0.6 V***.

![Current Sink Performance](./bgr-standard-assets/08_02_Simple_vs_Cascode_Results_dark.svg#only-dark)
![Current Sink Performance](./bgr-standard-assets/08_02_Simple_vs_Cascode_Results_light.svg#only-light)
/// caption
**Figure-11:** Simple and Cascoded Current sink performance
///

!!! success ""
    With that being the case, there is absolutely no reason to avoid cascoded current sink.

### Error Amp ICMR

Now we just need to see whether 0.777 V is in ICMR of our level shifted NMOS Diff-amp. *Figure-12* shows the full schematic of error amp in ICMR measurement configuration and *Figure-13* shows the simplified schematic of ICMR testbench.

<a id="fig-12"></a>

![Error Amp Schematic](./bgr-standard-assets/10_NMOS_LevelShifter_DiffAmp_dark.png#only-dark)
![Error Amp Schematic](./bgr-standard-assets/10_NMOS_LevelShifter_DiffAmp_light.png#only-light)
/// caption
**Figure-12:** Schematic diagram of Error Amplifier
///

![ICMR TB simplified](./bgr-standard-assets/11_ICMR_TB_dark.svg#only-dark)
![ICMR TB simplified](./bgr-standard-assets/11_ICMR_TB_light.svg#only-light)
/// caption
**Figure-13:** Simplified schematic of ICMR Test bench
///

In order to justify the need of a level shifter, I have included the results of ICMR measurements for a simple NMOS diff-amp of [Figure-03](#fig-03) (only the NMOS Input case) in *Figure-14(a)*.

![ICMR Results - 1](./bgr-standard-assets/12_01_ICMR_Results_withWithout_LevelShifter_dark.svg#only-dark)
![ICMR Results - 1](./bgr-standard-assets/12_01_ICMR_Results_withWithout_LevelShifter_light.svg#only-light)

![ICMR Results - 2](./bgr-standard-assets/12_02_ICMR_Vsup_2500mV_dark.svg#only-dark)
![ICMR Results - 2](./bgr-standard-assets/12_02_ICMR_Vsup_2500mV_light.svg#only-light)
/// caption
**Figure-14:** ICMR measurements for: (a) NMOS Diff-amp without level shifted inputs, (b) With level shifted inputs and \(c) level shifted input variant at a supply of 2.5 V
///

Some observations on *Figure-14*:

1. Simple NMOS Diff-amp without level shifter has an ICMR of 1.6 V to 3.3 V (*Figure-14(a)*). **Clearly doesn't include 0.777 V in it.**
2. The level shifted variant has an ICMR of 0 V to 2.72 V. The upper limit is due to level shifter as it needs the gate voltage sufficiently lower to keep the level shifting PMOS ON (*Figure-14(b)*).
3. *Figure-14\(c)* shows ICMR at a supply of 2.5 V (< 3.3 V). The ICMR is 0 V to 2.0 V roughly. Again limited by level shifter.

!!! success
    The level shifted variant includes 0.777 V in it's ICMR.

To summarize:

| V~DD~ | ICMR |
|-------|------|
| 3.3 V | 0 V to 2.7 V |
| 2.5 V | 0 V to 2.0 V |
/// caption
**Table-03:** ICMR summary for two cases of supply for Error-amp of [Figure-12](#fig-12)
///

## Startup circuit design

Since the reasoning behind creating the startup circuit is thoroughly explained in the [Startup Circuit Design](../references/bmr.md#startup-circuit-design) section of [BMR](../references/bmr.md), I won't waste time and space to repeat it here. 

![Startup circuit schematic](./bgr-standard-assets/13_StartupCircuit_dark.png#only-dark)
![Startup circuit schematic](./bgr-standard-assets/13_StartupCircuit_light.png#only-light)
/// caption
**Figure-15:** Startup circuit schematic
///

However, I will provide commentary on *Figure-15* to explain some choices in brief:

1. V~bias~ node is the Gate of Current sourcing PMOS (also the output node of error-amp). When the BGR has started up, the PMOS is biased to give 5 µA (See *Figure- ---*). V~bias~ receives appropriate voltage to set this current due to feedback loop.
2. When BGR has started up, you want to shut your leaker PMOS, and so, you need to pull it's Gate (or output of startup circuit) towards V~DD~. This is achieved by keeping three `1/3` sized diode connected NMOS, which cannot sink 5 µA and hence push the output to supply.

    ??? note
        Keeping three diode connected NMOS in pull down shouldn't surprise you. Recall the discussion in [Bias-leg Design](../references/bgr-standard.md#bias-leg-design), where we saw that many diode connected V~GS~ drops are needed to completely drop the supply of 3.3 V. That value for supply is huge, compared to what we get in core devices (L~min~ = 65 nm, V~DD~ = 1.2 V).

        So, ***it is very common to keep multiple diode connected MOSFETs in series especially in Long Channel design (or higher supply voltage designs).***

        I would even say it is the recommended way to do this, than keeping 1 ridiculously long MOSFET to completely drop the supply to Ground.

3. The leaker PMOS is taken `1/1` because just by this ratio, it gave about 70 µA (See *Figure-16, Right half*), more than enough to pull the voltage of regulated node up to start the circuit.

![Leaker MOS Design TB](./bgr-standard-assets/19_DCAnnotated_LeakerPMOS_dark.png#only-dark)
![Leaker MOS Design TB](./bgr-standard-assets/19_DCAnnotated_LeakerPMOS_light.png#only-light)
/// caption
**Figure-16:** Current leaked by a 1/1 PMOS
///

See the discussion on [Leaker NMOS Testbench](../references/bmr.md#leaker-nmos-testbench) for more information regarding the testbench seen in *Figure-16*.

## Fixing PTAT Voltage Source resistor, R~2~

The resistor value for R~2~ is fixed by aiming for ZTC (Zero Temperature Co-efficient) operating point for output. 

The output is made up of:

- CTAT Voltage source by bottom PNP Diode (8 in parallel).
- PTAT Voltage source by resistor R~2~.

Looking at [Figure-01](#fig-01), we can write,

\[V_{OUT} = V_{Q_3} + V_{R_2}\]

\[V_{OUT} = V_{Q_{3}} + L * \eta * V_T * \ln K\]

where, \(L = R_2/R_1\) and \(V_T\) is the thermal voltage.

Differentiating with respect to temperature and equating to zero (to find ZTC point) yields,

\[\frac{\delta V_{OUT}}{\delta T} = \frac{\delta V_{Q_3}}{\delta T} + L * \eta * \ln K * \frac{\delta V_{T}}{\delta T} = 0\]

Looking at [Table-01](#table-01), we get \(\delta V_{Q_3}/\delta T = -1.75~mV/C\), \(~\eta = 1003.6m~\) and \(~K = 8\). We also know \(\delta V_{T}/\delta T = 0.085~mV/C\). Plugging these values in above equation yields,

\[-1.75m + L * 1003.6m * \ln 8 * 0.085m = 0\]

\[L = 9.865\]

From L and R~1~, we get,

\[R_2 = L * R_1 = 9.865 * 10.9k \approx 107.5k\]