---
description: Low Voltage Bandgap Reference design procedure — parallel CTAT and PTAT current summation topology, 0.61V output at VDD/2, 28.265 ppm/°C temperature coefficient, on core 65nm MOSFET with 1.2V supply, simulated on Cadence Virtuoso and Spectre.
---

--8<-- "includes/disclaimer.md"

# Parallel Realized Band Gap Reference (BGR)

## Prerequisites

This design shares substantial methodology with the [Series Bandgap Reference](../references/bgr-standard.md). The core distinction is the **quantity** being temperature-stabilized — the Series BGR stabilizes **voltage** (series CTAT+PTAT voltage summation), while this design stabilizes **current** (parallel CTAT+PTAT current summation).

The following sections from the Series BGR apply directly and are not repeated:

- [Fixing PTAT Current Generator resistor, R~1~](../references/bgr-standard.md#fixing-ptat-current-generator-resistor-r1) — identical procedure, same operating point (5 µA), same parasitic PNP diodes. Resistor value of 10.9 kΩ carries over directly.
- [Added error amplifier](../references/bgr-standard.md#added-error-amplifier) — same operating point (5 µA) and  same V~BE~ of 0.777 V (See [Table-01](#table-01)). However, 0.777 V sits close to V~DD~ here (1.2 V core supply) unlike the Standard BGR (3.3 V I/O supply), giving rise to slight but important differences as discussed in the main body of this page.
- [Startup Circuit Design](../references/bmr.md#startup-circuit-design) — design considerations and procedure are identical to the Series BGR and BMR.

Read the [Series BGR](../references/bgr-standard.md) documentation before proceeding.

## Introduction

I have documented only the design procedure, and anything relevant to that. For theory of operation, please refer to standard textbooks or any lectures.

Before we proceed, let's take a look at the schematic of Parallel BGR.

<a id="fig-01"></a>

![Parallel BGR Schematic](./bgr-low-voltage-assets/01_SchematicOfParallelBGR_dark.svg#only-dark)
![Parallel BGR Schematic](./bgr-low-voltage-assets/01_SchematicOfParallelBGR_light.svg#only-light)
/// caption
**Figure-01:** Parallel BGR schematic diagram \[Ref. CMOS Circuit Design, Layout and Simulation, Fig 23.30]
///

Just like the series BGR, same parasitic PNP Diode characterized in [PNP 20 X 20](../mosfet/BJT-parameters.md#pnp-20-x-20) is used here as well, and so the parameters listed there are summarized in *Table-01:*

<a id="table-01"></a>

| Parameter | Value | Comments |
|-----------|-------|----------|
| η | 1003.6 m | Forward current emission coefficient (Emitter-Base junction) |
| V~BE~ | 0.777 V | 1 Diode (At a current of 5 µA) |
| \(\delta V_{BE}/\delta T\) | -1.75 mV/°C | Change in the diode’s voltage with temperature (at current of 5 µA) for 8 Diodes in parallel |
/// caption
**Table-01:** PNP Diode summary
///

!!! example ""
    Refer to [Introductory section of series BGR](../references/bgr-standard.md#introduction) for explanation on why 8 diodes is chosen. I chose not to repeat it here to avoid redundancy.

### Target output for this topology - 0.6 V (V~DD~\/2)

Since this topology allows to establish an output voltage less than the bandgap voltage of silicon (about 1.22 V), let's aim for an output voltage of 0.6 V as it is halfway through our supply rails (V~DD~\/2).

This value is chosen arbitrarily and could be set to anything else.

!!! danger ""
    Just make sure that it is not too close to supply rails, i.e. neither to close to V~DD~ (1.2 V) nor GND (0 V) as such extremes are difficult to handle for a MOSFET which demands some minimum voltage across it and not too much at the same time!

### Design Considerations

For designing this, we need to find:

- Sizes for all MOSFETs (Current sourcing PMOS as well as for building error-amp).
- PTAT Current Generator resistor, R.
- CTAT Current Generator resistor, L\*R.
- Output voltage resistor, N\*R for a nominal output of 0.6 V (V~DD~\/2).

The MOSFET sizes are already chosen and their parameters are thoroughly listed in [Core Device — 65 nm](../mosfet/parameters.md#core-device-65-nm) table. Unless specified otherwise, these sizes will be the ones we use to construct the entire circuit.

And as such, these are tabulated here for convenience:

<a id="table-02"></a>

| Flavour | W/L (Drawn) | W/L (Actual) | I~D~ | V~GS~ |
|---------|-------------|--------------|------|-------|
| NMOS | 43.1 / 2 | 2.8 µm / 130 nm | 10 µA | 500 mV |
| PMOS | 121.5 / 2 | 7.9 µm / 130 nm  | 10 µA | 470 mV |
/// caption
**Table-02:** Sizes and Bias Current summary
///

!!! question
    The observant reader would have questioned these sizing choices considering [Table-01](#table-01) characterizes PNP Diodes for 5 µA while here it is characterized for 10 µA. This is not a problem as, this time, we're designing parallel BGR ***which generates a sum of PTAT and CTAT current***. So it will *not just be 5 µA* needed for PTAT current generator, but **also some more** for CTAT current generator as well.

    On the occasion that the sum of PTAT and CTAT currents exceed 10 µA, the error-amp steps in to adjust the V~SG~ of top PMOS in [Figure-01](#fig-01) to source the necessary current as long as it is in the vicinity of 10 µA.

    !!! warning "This puts a constraint on Error amplifier output swing limit"
        To allow PMOS with these sizes to source more current, it becomes necessary that the error-amp's output should be capable of swinging freely. This is discussed in [Added error amplifier](#added-error-amplifier) section.

### Value for PTAT Current Generator resistor, R

Given the same operating point (5 µA) and same PNP Diode ([PNP 20 X 20](../mosfet/BJT-parameters.md#pnp-20-x-20)), the value of *PTAT Current Generator Resistor, R, remains the same as the [one calculated for Series BGR](../references/bgr-standard.md#fixing-ptat-current-generator-resistor-r1).* 

And it's value is: \(R = 10.9 ~k\Omega\)  
Refer to [Fixing PTAT Current Generator resistor, R~1~, section of Series BGR](../references/bgr-standard.md#fixing-ptat-current-generator-resistor-r1) for information regarding the testbench and method to find this value.

## Fixing CTAT Current Generator resistor, L\*R

The PTAT current is generated by dropping ΔVBE across resistor R.

<a id="eqn-01"></a>

\[\tag{1} I_{PTAT} = \frac{\eta * V_T * \ln K}{R}\]

And the CTAT current is generated by dropping VBE across resistor L\*R.

<a id="eqn-02"></a>

\[\tag{2} I_{CTAT} = \frac{V_{BE}}{L*R}\]

The total current is the sum of I~PTAT~ and I~CTAT~.

\[I_{Total} = I_{PTAT} + I_{CTAT}\]

Making this sum total temperature independent means:

\[\frac{\delta I_{Total}}{\delta T} = \frac{\delta I_{PTAT}}{\delta T} + \frac{\delta I_{CTAT}}{\delta T} = 0\]

Substituting [Equation-01](#eqn-01) and [Equation-02](#eqn-02) into the above equation yields,

\[\frac{\delta I_{Total}}{\delta T} = \frac{\delta}{\delta T} (\frac{\eta * V_T * \ln K}{R}) + \frac{\delta}{\delta T} (\frac{V_{BE}}{L*R}) = 0\]

\[\frac{1}{R} [ \eta * \ln K * \frac{\delta V_T}{\delta T} + \frac{1}{L} * \frac{\delta V_{BE}}{\delta T} ] = 0 \]

\[\tag{3} L = \frac{- \delta V_{BE} / \delta T }{\eta * \ln K * \delta V_T / \delta T} \]

The values for:

- η is known from [Table-01](#table-01) as 1003.6 m.
- K is chosen to be 8 (from [Introductory section of series BGR](../references/bgr-standard.md#introduction))
- V~T~ is thermal voltage and δV~T~/δT is 0.085 mV/°C.
- δV~BE~/δT is also known from [Table-01](#table-01) as -1.75 mV/°C.

Therefore, the value of L is:

\[L = \frac{- (-1.75m)}{1003.6m * \ln 8 * 0.085m} = 9.8653\]

and the value of L\*R resistor is:

<a id="eqn-04"></a>

\[\tag{4} L * R = 9.8653 * 10.9k\Omega = 107.5 ~k\Omega\]

!!! note
    This value of L\*R is just a starting guess. That is, we will start our design with this value, and later adjust it to get better temperature performance in iterations.

## Added error amplifier

<a id="fig-02"></a>

![Two stage Op amp schematic](./bgr-low-voltage-assets/02_05_ErrorAmp_schematic_dark.png#only-dark)
![Two stage Op amp schematic](./bgr-low-voltage-assets/02_05_ErrorAmp_schematic_light.png#only-light)
/// caption
**Figure-02:** Error amplifier schematic (Fully compensated Two stage op-amp)
///

The error amplifier for the design of parallel BGR is a simple two stage op-amp depicted in *Figure-02*. Much detail regarding choosing topology for an error amplifier is discussed in [Added error amplifier section of series BGR](../references/bgr-standard.md#added-error-amplifier) and so, to avoid redundancy, it is not covered here.

Still, here's a short explanation for why we chose this topology for our error amplifier:

1. *NMOS input stage is used to accommodate common mode voltage of 0.777 V*.

    Common mode voltage which appears at the input of error amplifier is 0.777 V (See V~BE~ in [Table-01](#table-01)) as stated in [Prerequisites](#prerequisites) and also in [Input common mode constraint section of series BGR](../references/bgr-standard.md#inherent-constraint-on-input-common-mode-of-nmos-and-pmos-diff-amp).

    However, 0.777 V is close to VDD (1.2 V) in this design (which uses the core-device 65nm), unlike in the [Series BGR](../references/bgr-standard.md), where it was closer to GND — since that version uses  I/O device 500nm with a higher VDD of 3.3 V.

    Hence, this mandates an NMOS input stage as discussed in [Input common mode constraint section of series BGR](../references/bgr-standard.md#inherent-constraint-on-input-common-mode-of-nmos-and-pmos-diff-amp).

2. *Output swing should be high to adjust V~SG~ of top PMOS in [Figure-01](#fig-01) to source necessary current*.

    Since we're doing current summation (I~PTAT~ + I~CTAT~), our total current of operating point is not just 5 µA (I~PTAT~). So, our error-amp must adjust Gate voltage of sourcing PMOS to source this current while maintaining very little difference in the regulated nodes. This calls for two stages (more gain) with a common source stage at the output (more swing).

### Concerning ICMR

This error-amp has an ICMR of 0.6 V to 1.2 V as shown in *Figure-03*. But sometimes, the lower limit of ICMR may come closer to our voltage of interest (which in this case is 0.777 V). When that happens, it is necessary to try to increase it and **keep some margin for error.**

![ICMR of error amp RVT](./bgr-low-voltage-assets/02_01_ICMR_RVTNMOSInput_dark.svg#only-dark)
![ICMR of error amp RVT](./bgr-low-voltage-assets/02_01_ICMR_RVTNMOSInput_light.svg#only-light)
/// caption
**Figure-03:** ICMR of error-amp shown in [Figure-02](#fig-02)
///

For example, when ICMR~min~ is 0.7 V, there is a chance it may increase to 0.75 V due to process variations and that is not good as it is closer to our desired value of 0.777 V. It's *always better to leave some margin* for error.

**One simple way to increase ICMR is to replace just the input NMOS with LVT** (Low Threshold Voltage) variant. Just to demonstrate it, let's use the sizes listed in [Low Threshold Voltage (LVT)](../mosfet/parameters.md#low-threshold-voltage-lvt) table which is repeated here in *Table-03* for convenience.

<a id="table-03"></a>

| Flavour | W/L (Drawn) | W/L (Actual) | I~D~ |
|---------|-------------|--------------|------|
| NMOS-LVT | 38.5 / 2 | 2.5 µm / 130 nm | 10 µA |
/// caption
**Table-03:** Sizes and bias current summary for LVT NMOS
///

And the resulting ICMR is 0.5 V to 1.2 V, (a 0.1 V extra margin from our RVT variant) which can be seen in *Figure-04*.

![ICMR of error amp LVT](./bgr-low-voltage-assets/02_02_ICMR_LVTNMOSInput_dark.svg#only-dark)
![ICMR of error amp LVT](./bgr-low-voltage-assets/02_02_ICMR_LVTNMOSInput_light.svg#only-light)
/// caption
**Figure-04:** ICMR of error-amp shown in [Figure-02](#fig-02) with LVT NMOS (Sizes from [Table-03](#table-03)) as input
///

I will go with RVT as it already has 0.777 V (V~BE~) in it's ICMR.

### Concerning Stability

Both [Series BGR](../references/bgr-standard.md) and [BMR](../references/bmr.md) utilized single-stage error amplifiers, each contributing one pole to the feedback loop, while the current generator added another pole, resulting in the loop having a total of two poles, which is easy to compensate by making the output of error amp the dominant pole.

The selection of a two-stage error amplifier introduces an additional pole from the second stage into the loop, bringing the total number of poles in the loop to three, which significantly **threatens the stability** of the overall loop.

However, before we can attempt to resolve this issue, we need to construct the current generator section (and consequently, the complete loop) in order to analyze the stability of the loop.

![Feedback loop with poles](./bgr-low-voltage-assets/08_01_FeedbackLoop_Poles_dark.svg#only-dark)
![Feedback loop with poles](./bgr-low-voltage-assets/08_01_FeedbackLoop_Poles_light.svg#only-light)
/// caption
**Figure-05:** Feedback loop with nodes that contribute poles
///

!!! note "Of course it is not three poles, but four"
    I know it is four poles in total. But generally P~3~ and P~3~' seen in *Figure-05* tends to be low impedance nodes (or high frequency poles) and will be close to each other. Even though they are non-dominant, they **still contribute phase lag, eating up our phase margin.**

#### Open Loop Gain of error-amp

We will look at stability again when we completed our DC temperature characteristics, but in the meantime, let's take a look at the Open loop Response of error-amp of [Figure-02](#fig-02) with loading as shown in *Figure-06*.

<a id="fig-06"></a>

![Open loop Gain measurement](./bgr-low-voltage-assets/02_06_ErrorAmp_AC_TB_Loaded_dark.png#only-dark)
![Open loop Gain measurement](./bgr-low-voltage-assets/02_06_ErrorAmp_AC_TB_Loaded_light.png#only-light)
/// caption
**Figure-06:** Open loop gain measurement testbench for error-amp of [Figure-02](#fig-02)
///

Notice that in *Figure-06*:

- The output of the error amplifier includes ***a large NMOS capacitor***, consisting of 20 NMOS transistors each with a `20/20` size amounting to **roughly 400 fF** (Chosen arbitrarily) (Refer to [Core device RVT MOSFET as a Capacitor](../mosfet/parameters.md#core-device-65-nm_2) table), acting as a load. This setup is used to select a compensation capacitor C~c~ that provides an adequate phase margin.

!!! warning
    Adding a capacitor to the output of a two stage amplifier generally degrades it's phase margin and threatens stability. In order to avoid that, I decided to design this error amp with a capability to ***accommodate an intentional capacitor load at the output***.

    In other words, I have designed for a margin, where even IF I add a capacitor, as long as it is less than 400 fF (nominal capacitance of 20 NMOS of `20/20` size each), I know it will be stable.

    On the rare occasion that I do need to drive an even larger capacitor, I have no choice but redesign the error-amp.

- The output also has the ***current generator PMOS*** (2 of them in parallel to model the two PMOS which loads the error amp) with the pull down part which should have included PNP Diodes along with resistors substituted with a voltage source of 0.777 V to model them. The rationale behind this is explained in [Substitution theorem section of BMR](../references/bmr.md#substitution-theorem). This models the parasitics due to the PMOS which loads the error amp.

The open loop gain of [Figure-06](#fig-06) is shown in *Figure-07* with a C~c~ of 200 fF.

!!! note "As for how I got this C~c~ of 200 fF"
    The procedure to compensate an amplifier is extensively covered in [Compensating the op-amp section of Rail-to-rail opamp](../amplifiers/rtr-opamp.md#compensating-the-op-amp). Refer to that, as I don't want to repeat it here to avoid redundancy.

![Open Loop Gain of error-amp](./bgr-low-voltage-assets/02_07_ACResp_TwoStage_200fF_Loaded_dark.svg#only-dark)
![Open Loop Gain of error-amp](./bgr-low-voltage-assets/02_07_ACResp_TwoStage_200fF_Loaded_light.svg#only-light)
/// caption
**Figure-07:** Open Loop gain of error amp of [Figure-02](#fig-02) with a C~c~ of 200 fF and a load of 400 fF
///

And the parameters are summarized in *Table-04*:

| Parameter | Value |
|-----------|-------|
| C~c~ | 200 fF |
| A~DC~ | 47.9 dB |
| f~ugb~ | 165.9 MHz |
| Φ~ugb~ | -98° |
| Φ~margin~ | 82° |
/// caption
**Table-04:** AC open loop response key parameters for VCM of 0.777 V (V~BE~)
///

!!! danger ""
    This doesn't paint the full picture yet. Until we see the loop gain, we can never be confident about the stability.

## Startup circuit design

There are two distinct designs that explain the logic behind the creation of startup circuits (One in [BMR's Startup circuit design section](../references/bmr.md#startup-circuit-design) and the other in [Series BGR's Startup circuit design section](../references/bgr-standard.md#startup-circuit-design)).

Therefore, trying to explain this logic again is a waste of time and space. And so, I am just going to show you the schematic of Startup Circuit (See *Figure-08*) that accompanies the Parallel BGR and move on to other sections.

![Startup circuit schematic](./bgr-low-voltage-assets/05_01_StartupCkt_schematic_dark.png#only-dark)
![Startup circuit schematic](./bgr-low-voltage-assets/05_01_StartupCkt_schematic_light.png#only-light)
/// caption
**Figure-08:** Startup circuit schematic
///

## Fixing output voltage resistor, N\*R

Let's build the core current generator portion of [Figure-01](#fig-01) and see whether if it works. 

<a id="fig-09"></a>

![DC Annotated Current Generator](./bgr-low-voltage-assets/03_01_Itr1_DCAnnotatedCurrGenerator_dark.png#only-dark)
![DC Annotated Current Generator](./bgr-low-voltage-assets/03_01_Itr1_DCAnnotatedCurrGenerator_light.png#only-light)
/// caption
**Figure-09:** DC Annotated schematic of core Current Generator
///

The values for R and L\*R are found to be **10.9 kΩ** and **107.5 kΩ** from [Value for PTAT Current Generator resistor, R](../references/bgr-low-voltage.md#value-for-ptat-current-generator-resistor-r) and [Fixing CTAT Current Generator resistor, L*R](../references/bgr-low-voltage.md#fixing-ctat-current-generator-resistor-lr).

From *Figure-09*, our design so far seems to at least bias up properly. If this was not the case, we have to fix it, before we proceed to output.

!!! danger "Sometimes, even DC Op Point needs a startup circuit"
    If you ever see that your current generator doesn't bias up to it's operating point, try to replace your error-amp with an ideal VCVS of appropriate gain (say 10,000). Generally a VCVS biases up the current generator without the need of a startup.

    But if even that fails, **do add a startup circuit** as it at least tries to induce a voltage to startup the circuit. It could be useful to troubleshoot the error.

Now that this is done, let's focus on fixing a value for output resistor N\*R.

Noting that these MOSFETs have short channel, trying to use equations for computing N\*R resistor leads to errors. Instead, let's use the Substitution theorem to find this value. This way is explained in [Fixing Resistor value section of BMR](../references/bmr.md#fixing-resistor-value) and so refer to that for more information.

<a id="fig-10"></a>

![Substitution for output resistor](./bgr-low-voltage-assets/03_02_Itr1_OutputResistorSubstitute_dark.png#only-dark)
![Substitution for output resistor](./bgr-low-voltage-assets/03_02_Itr1_OutputResistorSubstitute_light.png#only-light)
/// caption
**Figure-10:** Output resistor N\*R substituted with a voltage source of 0.6 V (VbiasP comes from [Figure-09](#fig-09))
///

*Figure-10* gave the current flowing through the voltage source as 13.629 µA and from this, we can compute the impedance looking into the voltage source (the impedance which needs to be present to generate 0.6 V, in other words the value of N\*R) (Refer to [Resistance computation testbench](../references/bmr.md#resistance-computation-testbench)).

<a id="eqn-05"></a>

\[\tag{5} Output ~voltage ~resistor, ~N * R = \frac{0.6}{13.629 \mu} = 44 k\Omega\]

## DC Simulations

We have calculated the resistor values for [R](../references/bgr-low-voltage.md#value-for-ptat-current-generator-resistor-r), [L\*R](#eqn-04) and [N\*R](#eqn-05) as 10.9 kΩ, 107.5 kΩ and 44 kΩ. These are just a starting point (kind of like an initial guess) and it will take several iterations to get stable temperature performance at the output.

### Iteration 1 - Using calculated values

Now that we know the values for resistors, let's first simulate this and see the output voltage performance across temperature.

| Parameter | Value |
|-----------|-------|
| R | 10.9 kΩ |
| L\*R | 107.5 kΩ |
| N\*R | 44 kΩ |
/// caption
**Table-05:** Values of resistors for iteration 1
///

The schematic diagram with the values in *Table-05* is shown in *Figure-11*.

![BGR Schematic - 1](./bgr-low-voltage-assets/12_BGR_Schematic_Iteration_01_dark.png#only-dark)
![BGR Schematic - 1](./bgr-low-voltage-assets/12_BGR_Schematic_Iteration_01_light.png#only-light)
/// caption
**Figure-11:** Parallel BGR schematic for iteration 1. Resistor values from Table-05
///

And the output voltage vs temperature can be seen in *Figure-12.*

<a id="fig-12"></a>

![BGR temperature performance - 1](./bgr-low-voltage-assets/03_03_Itr1_BGRout_107k44k_dark.svg#only-dark)
![BGR temperature performance - 1](./bgr-low-voltage-assets/03_03_Itr1_BGRout_107k44k_light.svg#only-light)
/// caption
**Figure-12:** BGR output performance across temperature for values of resistors in Table-05
///

Clearly, the output is showing CTAT nature instead of the characteristic bow.

Also, the output as a voltage reference is quite inaccurate, since it's around \(590 ~mV \pm 10 ~mV\).  With a total variation of 20 mV, it’s no surprise that it has an excessively high temperature coefficient of **444.4 ppm/°C**.

Let's iterate.

### Iteration 2 - Increasing L\*R by 50 kΩ

***This is purely a trial and error process. But there is still some logic to it.***

The general procedure is available in [Iteration 2 section of series BGR](../references/bgr-standard.md#iteration-2), but it is briefly repeated here for convenience:

!!! danger "DO NOT TOUCH PTAT CURRENT GENERATOR RESISTOR, R"
    The PTAT Current generator resistor, R in [Figure-01](#fig-01) sets the bias current for PNP Diodes to 5 µA which enables us to use parameters characterized for this PNP Diode listed in [Table-01](#table-01).

    If you change this resistor, remember that you just made all the other calculations as invalid and have to restart by regenerating [Table-01](#table-01) and then recalculating L\*R and N\*R resistors. **It will take you nowhere!**

L\*R and N\*R resistors offers two degrees of freedom for:

- L\*R - Adjusting temperature performance
- N\*R - Adjusting output voltage

Once again, let's recall our expected output characteristic for a voltage reference derived from the BGR.

<a id="fig-13"></a>

![BGR Characteristic BOW](./bgr-low-voltage-assets/09_BGR_CharacteristicBow_dark.svg#only-dark)
![BGR Characteristic BOW](./bgr-low-voltage-assets/09_BGR_CharacteristicBow_light.svg#only-light)
/// caption
**Figure-13:** Characteristic bow in the output curve of a bandgap reference across temperature \[Ref. Designing Analog Chips, Hans Camenzind, Fig. 7-6]
///

With all this and from [Iteration 2 section of series BGR](../references/bgr-standard.md#iteration-2), the procedure is:

- Increase L\*R by considerable amount (say 50 kΩ or even 100 kΩ), and re-simulate. See if you get the left portion of *Figure-13*, that is whether [Figure-12](#fig-12) has changed to PTAT curve instead of CTAT curve.
    - If it did, we know that Bow characteristic lies between initial L\*R value of 107.5 kΩ and this new value, and so start to fine tune it in this range to get the desired curve.
    - If it still remained CTAT, then you adapt and do the opposite.
- Decreasing L\*R should be considered after you first try increasing it. 

!!! example "SPOILERS: THIS TIME WE WILL GET AN INVERTED CURVE OF *Figure-13*"
    I know this is surprising, but this time, I obtained an output curve that features the bow from *Figure-13*, **inverted**.

    Let's see an example output with this **inverted** curve. See *Figure-14*.

    <a id="fig-14"></a>
    
    ![Inverted BGR Bow](./bgr-low-voltage-assets/10_Inverted_BGR_Bow_dark.svg#only-dark)
    ![Inverted BGR Bow](./bgr-low-voltage-assets/10_Inverted_BGR_Bow_light.svg#only-light)
    /// caption
    **Figure-14:** Inverted Bow output of parallel realized BGR (V~out~ = 0.61 V) (QUCS-S/NGSPICE)
    ///
    
    The reason for presenting this is to demonstrate that the aforementioned procedural steps were formulated keeping [Figure-13](#fig-13) in mind—assuming that the BGR produces an output bow with the characteristics shown there. 
    
    But *Figure-14* shows that ***WE CAN GET AN INVERTED BOW*** at the output.
    
    So, these procedural steps are nothing more than guidelines. ***Always adapt to your iteration's results to quickly narrow down to a better output.***

---

With the above discussion, let's increase L\*R resistor by 50 kΩ to get a new value of 157.5 kΩ.

| Parameter | Value |
|-----------|-------|
| R | 10.9 kΩ |
| L\*R | 157.5 kΩ |
| N\*R | 44 kΩ |
/// caption
**Table-06:** Values of resistors for iteration 2
///

<a id="fig-15"></a>

![BGR temperature performance - 2](./bgr-low-voltage-assets/03_04_Itr2_BGRout_157k44k_dark.svg#only-dark)
![BGR temperature performance - 2](./bgr-low-voltage-assets/03_04_Itr2_BGRout_157k44k_light.svg#only-light)
/// caption
**Figure-15:** BGR output performance across temperature for values of resistors in Table-06
///

*Figure-15* shows great resilience against temperature than [Figure-12](#fig-12). But the obtained output is \(507.7 ~mV \pm 1.8 ~mV\), 0.1 V below our target of 0.6 V. For now, we will prioritize finding ZTC (Zero temperature coefficient) point. Later, we will adjust output resistor N\*R as shown in [Figure-10](#fig10) to get the output back to 0.6 V.

The temperature performance is not bad when comparing it to [Figure-12](#fig-12) but is still bad when compared to [Figure-14](#fig-14).

Let's iterate.

### Iteration 3 - Increasing L\*R by another 50 kΩ

Let's increase L\*R resistor by another 50 kΩ to get a new value of 207.5 kΩ and re simulate.

| Parameter | Value |
|-----------|-------|
| R | 10.9 kΩ |
| L\*R | 207.5 kΩ |
| N\*R | 44 kΩ |
/// caption
**Table-07:** Values of resistors for iteration 3
///

![BGR temperature performance - 3](./bgr-low-voltage-assets/03_05_Itr3_BGRout_207k44k_dark.svg#only-dark)
![BGR temperature performance - 3](./bgr-low-voltage-assets/03_05_Itr3_BGRout_207k44k_light.svg#only-light)
/// caption
**Figure-16:** BGR output performance across temperature for values of resistors in Table-07
///

Iteration 3 yielded an output of \(461.8 ~mV \pm 3.2 ~mV\). Some observations:

- Output has fallen to 0.46 V. About 0.14 V lower than our target of 0.6 V.
- ***Output has changed to PTAT nature!***
    - This is important, as it means we just crossed the ZTC point since we expect to see an inverted BGR. Which should be clear from [Figure-15](#fig-15) and *Figure-16* where both are trying to flatten out at the bottom implying an inverted curve.

---

!!! success ""
    From [Iteration 2](#iteration-2-increasing-lr-by-50-k) and [Iteration 3](#iteration-3-increasing-lr-by-another-50-k), it is clear that our required value of L\*R resistor lies between 157.5 kΩ and 207.5 kΩ.

    \[157.7 ~k \Omega < L * R < 207.5 ~k \Omega\]

Let's iterate within that range.

### Iteration 4 - Narrowed down to 175 kΩ

In order to avoid getting bogged down by numerous iterations, I'm recording the value that produced the desired output curve. I determined this value through several iterations of the L\*R resistor, ranging from 157.5 kΩ to 207.5 kΩ as discussed at the end of [Iteration 3](#iteration-3-increasing-lr-by-another-50-k).

<a id="table-08"></a>

| Parameter | Value |
|-----------|-------|
| R | 10.9 kΩ |
| L\*R | 175 kΩ |
| N\*R | 44 kΩ |
/// caption
**Table-08:** Values of resistors for iteration 4
///

![BGR temperature performance - 4](./bgr-low-voltage-assets/03_06_Itr4_BGRout_175k44k_dark.svg#only-dark)
![BGR temperature performance - 4](./bgr-low-voltage-assets/03_06_Itr4_BGRout_175k44k_light.svg#only-light)
/// caption
**Figure-17:** BGR output performance across temperature for values of resistors in Table-08
///

We got the expected bow, but our output has fallen to 0.488 V. 

---

Let's adjust N\*R resistor now.

Once again, the testbench is similar to [Figure-10](#fig-10) and is shown in *Figure-18*.

<a id="fig-18"></a>

![Substitution for output resistor - 2](./bgr-low-voltage-assets/03_07_Itr4_OutputResistorSubstitute_dark.png#only-dark)
![Substitution for output resistor - 2](./bgr-low-voltage-assets/03_07_Itr4_OutputResistorSubstitute_light.png#only-light)
/// caption
**Figure-18:** Output resistor N\*R substituted with a voltage source of 0.6 V (VbiasP comes from [Figure-09](#fig-09) with resistor values from [Table-08](#table-08))
///

Noting that the current drawn is 10.56 µA for a voltage of 0.6 V, the impedance it presents is:

\[\tag{6} Output ~voltage ~resistor, ~N * R = \frac{0.6}{10.56 \mu} = 56.8 k\Omega\]

So the new values of resistors are:

| Parameter | Value |
|-----------|-------|
| R | 10.9 kΩ |
| L\*R | 175 kΩ |
| N\*R | 56.8 kΩ |
/// caption
**Table-09:** Values of resistors for iteration 4 - adjusting the output voltage to 0.6 V
///

![BGR temperature performance - 4 - 1](./bgr-low-voltage-assets/03_07_Itr4_BGRout_175k56_8k_dark.svg#only-dark)
![BGR temperature performance - 4 - 1](./bgr-low-voltage-assets/03_07_Itr4_BGRout_175k56_8k_light.svg#only-light)
/// caption
**Figure-19:** BGR output performance across temperature for values of resistors in Table-09
///

Great! Our output is closer to 0.6 V but the characteristic bow seems to be shifted.

Let's iterate

### Iteration 5 - Final - 170 kΩ

*Figure-19* seems to be slightly leaning towards PTAT. Following the procedure from [Iteration 2](#iteration-2-increasing-lr-by-50-k), ***I need to lower the resistor value to shift toward CTAT, and in doing so, bring the bow back to the center***.

I found 170 kΩ by fine tuning L\*R value.

| Parameter | Value |
|-----------|-------|
| R | 10.9 kΩ |
| L\*R | 170 kΩ |
| N\*R | 56.8 kΩ |
/// caption
**Table-10:** Values of resistors for iteration 5
///

![BGR temperature performance - 5](./bgr-low-voltage-assets/03_08_Itr5_BGRout_170k56_8k_dark.svg#only-dark)
![BGR temperature performance - 5](./bgr-low-voltage-assets/03_08_Itr5_BGRout_170k56_8k_light.svg#only-light)
/// caption
**Figure-20:** BGR output performance across temperature for values of resistors in Table-10
///

---

And following [Figure-10](#fig-10) and [Figure-18](#fig-18), I adjusted the value of N\*R resistor to 56 kΩ to bring the output closer to 0.6 V.

| Parameter | Value |
|-----------|-------|
| R | 10.9 kΩ |
| L\*R | 170 kΩ |
| N\*R | 56 kΩ |
/// caption
**Table-11:** Values of resistors for iteration 5 - adjusting the output voltage to 0.6 V
///

![BGR temperature performance - 5 - 1](./bgr-low-voltage-assets/03_09_Itr5_BGRout_170k56k_dark.svg#only-dark)
![BGR temperature performance - 5 - 1](./bgr-low-voltage-assets/03_09_Itr5_BGRout_170k56k_light.svg#only-light)
/// caption
**Figure-21:** BGR output performance across temperature for values of resistors in Table-11
///

The final result seems satisfactory. The obtained output voltage is \(598.8 ~mV \pm 525 ~ \mu V\) with a temperature co-efficient of **41.789 ppm/°C**.

!!! warning
    I could iterate to get better performance, but it would be useless considering the fact that these specifications are obtained in a simulation and when we fabricate this, the resistors will need trimming.

    ***Do not attempt for precise value of 0.6 V or even better temperature characteristics*** by fine tuning the value of resistors. It is a futile thing to do considering process variations and worse tolerance of IC resistors. They cannot be fabricated with precise values and will need trimming. 
    
    When we say the output is 0.6 V, it is nominally 0.6 V and it could vary slightly. Run a monte carlo analysis to test this statement. ***This precise design methodology is just an illusion spread among the community and do not fall for it!***

Now that we got the characteristic bow, let's also see the supply dependance.

### Supply Dependance

Sweeping the supply from GND to VDD (1.2 V), yields *Figure-23*.

![BGR output supply Performance](./bgr-low-voltage-assets/04_SupplyDependance_dark.svg#only-dark)
![BGR output supply Performance](./bgr-low-voltage-assets/04_SupplyDependance_light.svg#only-light)
/// caption
**Figure-23:** BGR output vs Supply
///

Unlike the [Supply dependance of series BGR](../references/bgr-standard.md#supply-dependance), our parallel BGR shows only **0.15 V of margin** for supply variations (from 1.05 V to 1.2 V). Any lower than that results in huge errors from our expected output.

The sudden rise in output around 0.7 V is also easily explained remembering that our PNP Diodes generate a V~BE~ of 0.777 V [See [Table-01](#table-01)] and our supply is slowly becoming high enough to accommodate this voltage. Once it is sufficiently away from that value (around 0.9 V), our output starts to settle towards it's expected value.

Gaining anymore margin than the one we got (about 0.15 V) is extremely hard considering the fact that a huge portion of supply is eaten up by the V~BE~ generated (0.777 V, about 64.75% of V~DD~). This is one of the motivations behind characterizing PNP Diode for a low current of 5 µA, which generated this voltage.

If I used a larger current, it would have generated an even larger voltage and so, would have eaten up our supply.

Some ways of increasing margin for supply are:

- Characterize PNP Diode for even lower currents (lower than 5 µA) to generate smaller V~BE~.
- Increase the number of PNP Diodes in parallel in both left and right by the same multiplicity. For example, keep 5 diodes in the left and 40 diodes (5\*8) in the right.

The MOSFETs cannot be adjusted for more margin considering that their sizes are chosen for a minimum V~SD~ of just 60 mV (See [Core Device — 65 nm](../mosfet/parameters.md#core-device-65-nm) table).

## AC Simulations

Now that our current generator portion design is complete, let's look at the long dreaded loop gain to see whether the loop will remain stable.

### Breaking the loop

To that end, we need to break the loop at a suitable point. *Figure-24* shows the breaking of the loop.

![Breaking Loop](./bgr-low-voltage-assets/14_Breaking_Loop_dark.svg#only-dark)
![Breaking Loop](./bgr-low-voltage-assets/14_Breaking_Loop_light.svg#only-light)
/// caption
**Figure-24:** Breaking the feedback loop to measure loop gain
///

Even though it appears that there are three points where we can break the loop, only the point shown in *Figure-24* breaks the loop while the other two still has some pathway for the signal to return, that is, it doesn't break the loop.

To achieve this in a simulation, we add a really large inductor (say, 1000 H) at the point where I want to break the loop. So, the inductor doesn't disturb DC biasing, while it completely blocks pathway for AC signals.

This is applied in our testbench to measure loop gain as seen in *Figure-25*.

<a id="fig-25"></a>

![Loop Gain Testbench](./bgr-low-voltage-assets/13_01_LoopGain_schematic_NoCap_dark.png#only-dark)
![Loop Gain Testbench](./bgr-low-voltage-assets/13_01_LoopGain_schematic_NoCap_light.png#only-light)
/// caption
**Figure-25:** Loop gain measurement testbench
///

!!! warning
    Notice how the signal is coupled using a really large capacitor instead of connecting it directly. This is really important, as connecting an ideal voltage source **dictates** the voltage of that node.

    We don't want that. Instead, we want the DC to be set by the loop, and that DC is blocked with this capacitor.

!!! note
    This testbench doesn't include the output branch as it is not a part of the loop. I omitted them as I really don't want to include useless portions in a simulation.

    But, the startup is included, as ***it is part of the loop, even though it doesn't disturb the circuit once it turns on***, but still included to see IF it shows any problems in our loop.

### Loop Gain Measurements and Stability

The simulation results of [Figure-25](#fig-25) is seen in *Figure-26*.

<a id="fig-26"></a>

![Loop Gain Response](./bgr-low-voltage-assets/13_02_LoopGain_NoCap_dark.svg#only-dark)
![Loop Gain Response](./bgr-low-voltage-assets/13_02_LoopGain_NoCap_light.svg#only-light)
/// caption
**Figure-26:** Loop Gain Response
///

The Loop Gain shows a ***good phase margin of 83.6°*** (Noting that the Phase starts at 180° due to three stages). From this, we can rest assured that our loop is stable.

!!! danger "DO NOT ADD A CAPACITOR TO THE OUTPUT OF ERROR AMP"
    In both [Series BGR](../references/bgr-standard.md#transient-simulation-startup-action) and [BMR](../references/bmr.md#transient-simulation-startup-action), an intentional capacitor was added to the output of error amp to stabilize the loop.

    But that is not necessary here and in fact it is dangerous to add one since the error amp used here is a two stage amplifier as opposed to those two designs which sported a single stage amplifier. Adding a capacitor to the output of a two stage amplifier will pull the output node back to lower frequencies ***eating up phase margin***.

    And in fact, I will show some simulation results where a capacitor is connected to the output of the error amplifier, using both NMOS ([a capacitor to GND](../references/bmr.md#what-happens-with-a-capacitor-to-gnd-or-nmos-capacitor)) and PMOS ([a capacitor to VDD](../references/bmr.md#what-happens-with-a-capacitor-to-vdd-or-pmos-capacitor)) as capacitors.

    ![Loop Gain Response - NMOS Cap](./bgr-low-voltage-assets/13_03_LoopGain_NMOSCap_dark.svg#only-dark)
    ![Loop Gain Response - NMOS Cap](./bgr-low-voltage-assets/13_03_LoopGain_NMOSCap_light.svg#only-light)

    ![Loop Gain Response - PMOS Cap](./bgr-low-voltage-assets/13_04_LoopGain_PMOSCap_dark.svg#only-dark)
    ![Loop Gain Response - PMOS Cap](./bgr-low-voltage-assets/13_04_LoopGain_PMOSCap_light.svg#only-light)
    /// caption
    **Figure-27:** Loop Gain Response with (a) an NMOS as a capacitor and (b) a PMOS as a capacitor loading the error-amp
    ///

    Just as we discussed, it eats up phase margin. It is treacherous to add a capacitor to this loop for stability purposes.

## TRANSIENT Simulations

Since we already discussed the stability of the loop, we will directly jump to startup simulation. *Figure-28* shows the complete schematic of Parallel BGR.

<a id="fig-28"></a>

![BGR Schematic Full]()
/// caption
**Figure-28:** Complete schematic of Parallel BGR
///

Notice the NMOS capacitor at the output. It is really important to add one to keep it stable at a specific voltage (We will talk about this shortly).

***To simulate the circuit turning ON case, supply starts to turn ON at 30 ns with a rise time of 20 ns to reach VDD at 50 ns***

And *Figure-29* shows the transient simulation results.

<a id="fig-29"></a>

![BGR Startup action](./bgr-low-voltage-assets/05_03_StartupAction_NOCAP_dark.svg#only-dark)
![BGR Startup action](./bgr-low-voltage-assets/05_03_StartupAction_NOCAP_light.svg#only-light)
/// caption
**Figure-29:** Startup transient of [Figure-28](#fig-28). Shows both output of error-amp (VBIASP) and BGR (VBGR)
///

The noise in output of error-amp (VBIASP) around 50 ns mark is really unaccounted for, since the loop is stable as seen from [Figure-26](#fig-26). This is slightly coupled to even BGR output (VBGR) as well despite the presence of an intentional NMOS capacitor.

So what is causing this?

### Investigating the noise in startup transients

It turns out, this is ***the supply noise coupling onto the output***. And the culprit is our error-amp's compensation scheme.

Looking back at [Figure-02](#fig-02), we see that the compensation capacitor is connected in a way closer to the positive supply rail, which literally allows the supply noise to couple onto the output of error-amp through this (See *Figure-30*).

![Supply Noise Path](./bgr-low-voltage-assets/15_Supply_Noise_Coupling_dark.svg#only-dark)
![Supply Noise Path](./bgr-low-voltage-assets/15_Supply_Noise_Coupling_light.svg#only-light)
/// caption
**Figure-30:** Supply noise coupling path to output of error amp
///

When the supply turn ON in [Figure-29](#fig-29), the sudden ramp induces noise on the output error-amp through C~c~, and once the supply becomes stable, we don't see anymore ripples in any of the outputs. In fact, the ripples die down and the voltages become stable as the feedback loop is stable as seen in [Figure-26](#fig-26).

Obviously we don't want this. But, what's even more strange is the fact *there is a slight ripple on the final output of BGR (VBGR)* as seen [Figure-29](#fig-29) around the 50 ns mark. There is an NMOS loading that node, which is supposed to act as a capacitor. And yet, we do see the ripple. **This doesn't make any sense, even in *Figure-30.***

### Why the small ripple on output of BGR even with capacitor load?

NMOS acts as a capacitor, only when it is strongly inverted. Looking at [Core device MOSFET as a capacitor](../mosfet/parameters.md#core-device-65-nm_2) table, we see that for an NMOS to behave as a capacitor, it ***needs a gate voltage larger than 0.6 V***.

That table is repeated here in *Table-12* for convenience.

| Flavour | Drawn Size | Actual Size | V~G~ | Nominal Capacitance |
|---------|------------|-------------|------|---------------------|
| NMOS-RVT | 20 / 20 | 1.3 µm / 1.3 µm | > 0.6 V | 20.9 fF |
/// caption
**Table-12:** MOSFET capacitance for specific size and Voltage conditions
///

And clearly from [Figure-29](#fig-29), the instant where we see a ripple, around the 50 ns mark, ***the output is near 150 mV, which is clearly less than the required voltage (0.6 V) for proper operation***.

In order to solidify this statement, see the CV curve for an NMOS of `20/20` size shown in *Figure-31*.

![NMOS CV Curve - 20/20](./bgr-low-voltage-assets/16_CV_Curve_NMOS_20by20_dark.svg#only-dark)
![NMOS CV Curve - 20/20](./bgr-low-voltage-assets/16_CV_Curve_NMOS_20by20_light.svg#only-light)
/// caption
**Figure-31:** NMOS CV Curve for 20/20 size. (Curve title is wrong as I generated this for both 10/10 and 20/20 at the same time.)
///

Clearly an operating voltage of 0.15 V is nowhere enough to make the NMOS behave as a capacitor with a nominal value of 20.9 fF. Instead it has just about 5 fF (From *Figure-31*), almost four times lesser than it's nominal value.

And due to this, the ripple easily passes through to the output of BGR.

But once it has settled to it's steady state value (about 0.6 V), then it no longer can affect the output.

### PSRR+ measurements

Since we already know that the culprit is the supply noise, let's quantify the amount by which it gets rejected (or the amount that makes it through) using PSRR+ (Power Supply Rejection Ratio for positive supply rail).

![PSRR of BGR](./bgr-low-voltage-assets/07_03_BGR_PSRR_NOCAP_dark.svg#only-dark)
![PSRR of BGR](./bgr-low-voltage-assets/07_03_BGR_PSRR_NOCAP_light.svg#only-light)
/// caption
**Figure-32:** PSRR+ response for both output of error-amp (PSRR_VBIASP) and BGR (PSRR_BGR)
///

Just as we speculated, the output error-amp (PSRR_VBIASP) trace shows a poor PSRR+ of just 308.85 mdB (or roughly 1) (Not even attenuated!). It's literally passing the supply noise directly onto the output of error-amp. Not good.

Even though the BGR output (PSRR_BGR) shows a decent PSRR+ of -25.94 dB (or 0.056) (Good attenuation), it is acquired **only when NMOS Capacitor has strongly inverted, that is, it starts to behave as a capacitor of nominal value of 20.9 fF**. We saw this issue in [Figure-29](#fig-29).

### Modifications to mitigate supply noise

Our only hope against this is to modify the compensation scheme of [Figure-02](#fig-02) or choose an entirely different topology for error amp. Let's explore both.

!!! warning "Unverified Recommendations"
    The following topology modifications are proposed based on design intuition and reasoning — they have not been simulated or verified. A rigorous noise analysis is beyond the scope of this documentation.

    If you plan to implement these, conduct independent verification before use.

#### Modifying compensation scheme

In order to save power in [Figure-02](#fig-02), the compensation capacitor, C~c~, was directly connected to the PMOS load of diff-amp. But if we consider the original method, it is buffered through a CG stage as shown in *Figure-33*.

![Compensation using CG buffer](./bgr-low-voltage-assets/17_OriginalCompensationScheme_CGStageBuffer_dark.svg#only-dark)
![Compensation using CG buffer](./bgr-low-voltage-assets/17_OriginalCompensationScheme_CGStageBuffer_light.svg#only-light)
/// caption
**Figure-33:** Buffering the compensation capacitor through a CG stage \[Ref. CMOS Circuit Design, Layout and Simulation, Fig 24.17]
///

This is a good topology as it keeps the capacitor away from both supply rails and reduce the noise that gets coupled from there. The trade-off is of course, you burn additional power in the added leg.

#### Choosing a different topology for error-amp

Looking back at [Added error amplifier section](../references/bgr-low-voltage.md#added-error-amplifier), we saw that our output of error-amp must swing freely because we're doing current summation, that is, the sourcing PMOS has to source more than 5 µA (I~PTAT~) to accommodate the CTAT current resistors L\*R as well.

And also, this is just a DC Circuit, as in, it just generates a reference voltage and the error-amp doesn't need to be faster.

With this in mind, consider the topology for error-amp shown in *Figure-34*.

![Alternative Topology Error-amp](./bgr-low-voltage-assets/18_AlternativeTopologyForErrorAmp_dark.svg#only-dark)
![Alternative Topology Error-amp](./bgr-low-voltage-assets/18_AlternativeTopologyForErrorAmp_light.svg#only-light)
/// caption
**Figure-34:** Alternative topology for error-amp \[Ref. CMOS Circuit Design, Layout and Simulation, Fig 24.33]
///

This topology has an output stage which can swing freely, making it ideal for our BGR. Also, compensating the loop can be done just like other designs with a single stage amplifier, by keeping a MOSFET capacitor at the output of error-amp.

Notice that there are no easy path for supply noise to couple onto output of error-amp with this topology.