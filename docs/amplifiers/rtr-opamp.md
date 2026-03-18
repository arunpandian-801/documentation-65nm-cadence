---
description: Rail-to-rail input Class AB output op-amp design procedure — folded cascode topology with parallel NMOS and PMOS input stages on 65nm custom CMOS, characterised across all three CMR regions, simulated on Cadence Virtuoso and Spectre.
---

--8<-- "includes/disclaimer.md"

# Rail to rail input, Class AB output op-amp

## Introduction

I have documented only the design procedure, and anything relevant to that. For theory of operation, please refer to standard textbooks or any lectures.

Before we proceed, let's take a look at the schematic of this op-amp.

<a id="fig-01"></a>

![Rail to rail ip opamp schematic](./rtr-opamp-assets/08_Rail_to_rail_op_amp_Schematic_dark.svg#only-dark)
![Rail to rail ip opamp schematic](./rtr-opamp-assets/08_Rail_to_rail_op_amp_Schematic_light.svg#only-light)
/// caption
**Figure-01:** Rail-to-rail input, class AB output opamp schematic [Ref. CMOS Circuit Design, Layout and Simulation, Fig 24.48]
///

### Design Considerations

The design of this circuit is pretty straightforward as the only challenge in this is frequency compensation. Biasing is done by [General Bias Circuit](/references/general-bias/#general-bias-circuit) and the sizes are from [Regular Threshold Voltage (RVT)](/mosfet/parameters/#regular-threshold-voltage-rvt) table and **half the sizes** noted in [Low Threshold Voltage (LVT)](/mosfet/parameters/#low-threshold-voltage-lvt) table respectively.

For the sake of convenience, those sizes are listed here:

<a id="table-01"></a>

| Flavour | W/L (Drawn) | W/L (Actual) | COMMENTS |
|---------|-------------|--------------|----------|
| NMOS-RVT | 43.1 / 2 | 2.8 µm / 130 nm | Same as standard size |
| PMOS-RVT | 121.5 / 2 | 7.9 µm / 130 nm | Same as standard size |
| NMOS-LVT | 20 / 2 | 1.3 µm / 130 nm | Roughly **half** the standard size |
| PMOS-LVT | 47.7 / 2 | 3.1 µm / 130 nm | Roughly **half** the standard size |
/// caption
**Table-01:** Sizes and bias current summary.
///

!!! note ""
    It is strongly advised to review the documentation of the [General Bias Circuit](/references/general-bias/#general-bias-circuit) before moving forward with this one. Particularly pay attention to the discussion on:

    - [Sizing Class AB Bias Voltage Generation MOSFET](/references/general-bias/#sizing-class-ab-bias-voltage-generation-mosfet)
    - [Biasing a Class AB output stage](/references/general-bias/#biasing-a-class-ab-output-stage)

### Load Considerations

Thanks to class AB output stage, this op-amp is capable of driving low impedance (or heavy) loads. So, the load taken for this documentation is a resistor of 10 kΩ in parallel with a capacitor of 10 pF (10 kΩ || 10 pF). And as such, the output stage MOSFETs are 10 times (just a general number. A good starting point for heavy loads. If this proves insufficient, we will increase it.) the sizes listed in [Regular Threshold Voltage (RVT)](/mosfet/parameters/#regular-threshold-voltage-rvt) table.

!!! info
    Since this schematic diagram is quite large, I don't want to repeat it for every single simulation. Therefore, the schematic shown in [Figure-06](#fig-06) should be our starting point, and it will remain as the one we use until compensation is completed. All testbenches will include a symbol representing an op-amp that abstracts this schematic for simplicity until told otherwise.

Before we compensate this op-amp, let's see the ICMR.

## DC Simulations

### Input Common Mode Range

We will tie both inputs together and sweep it from negative rail to positive rail as shown in *Figure-02*.

![ICMR TB](./rtr-opamp-assets/09_ICMR_TB_dark.svg#only-dark)
![ICMR TB](./rtr-opamp-assets/09_ICMR_TB_light.svg#only-light)
/// caption
**Figure-02:** Testbench to find ICMR. The Output nodes are shown in Figure-03
///

The nodes *OUT_P_BIAS* and *OUT_N_BIAS* are the outputs of the 1rst stage, i.e. the outputs of the Cascode stack. These nodes can be seen in the partial schematic of *Figure-03*.

![Output Nodes in actual schematic](./rtr-opamp-assets/10_ICMR_Outputs_dark.svg#only-dark)
![Output Nodes in actual schematic](./rtr-opamp-assets/10_ICMR_Outputs_light.svg#only-light)
/// caption
**Figure-03:** The Nodes OUT_P_BIAS and OUT_N_BIAS are shown in actual schematic for cross-reference with Figure-02.
///

<a id="fig-04"></a>

![Output curve over ICMR](./rtr-opamp-assets/11_CommonModeSweep_dark.svg#only-dark)
![Output curve over ICMR](./rtr-opamp-assets/11_CommonModeSweep_light.svg#only-light)
/// caption
**Figure-04:** Outputs of folded cascode stack vs V~CM~. Used to find ICMR. Notice that **ICMR extends past supply rails**.
///

Clearly from *Figure-04*, we can see that the ICMR for this op-amp is -0.2 V to 0.4 V (PMOS ON), then 0.85 V to 1.4 V (NMOS ON). In between these two ranges, both diff-amps simultaneously turn OFF and ON as the ICMR approaches their operating range. This range, 0.4 V to 0.85 V is called the **cross-over region** for a rail-to-rail input stage, and the term is self-explanatory as the input stage crosses over from PMOS to NMOS.

Even though there are common-mode variations in the **cross-over region** we will still include this in our overall ICMR by keeping in mind that such non-linearities are seldom avoidable.

!!! danger ""
    The **cross-over region** presents a ***HUGE HURDLE*** in frequency compensation.

## AC Simulations

### Cross-over region dictates frequency compensation

In the cross-over region, both input diff-amps are ON. And so both will contribute a change in current of \((g_{mn} + g_{mp}) * v_{in}\) into the cascode stack. And when we're outside this region, only one of the input diff-amps will contribute a change based on our input common mode voltage V~CM~.

To realize a fully compensated op-amp, we have to account for this worst case (*cross-over region*, both N and P Diff-amps ON) which makes the **op-amp slower** when only one input stage (Either N or P Diff-amp) is ON.

<a id="note-collapse-1"></a>

??? note
    There are ways to mitigate this problem by implementing constant g~m~ for all three cases (Only N, only P and both N and P Diff-amps ON), i.e. making the total transconductance available at all time the same by modifying the input stage to get \(g_{mn} + g_{mp} \approx g_{mn,only} \approx g_{mp,only}\) \[Ref. *Operational Amplifiers, Johan Huijsing, section 4.4*], but such modifications are beyond the scope of this documentation.

### Frequency compensation strategy

Because we have two paths for changing current in cascode stack, it makes sense that we add two compensation capacitors as shown in *Figure-05*

<a id="fig-05"></a>

![Compensation capacitors connection](./rtr-opamp-assets/13_Compensation_Strategy_dark.svg#only-dark)
![Compensation capacitors connection](./rtr-opamp-assets/13_Compensation_Strategy_light.svg#only-light)
/// caption
**Figure-05:** Connection of compensation capacitors.
///

Comparing this with [Figure-01](#fig-01), we can see that we're ***overcompensating*** the op-amp for stability across all regions of ICMR.

### Compensating the op-amp

!!! quote
    In amplifiers using one of the latter two methods (error feed forward or compensation)‚ the error at the output is not measured and corrected‚ but during design time the expected error of the active components is estimated and a correction circuit to remove this error is added.

    \- *Ref. Structured Electronic Design, Negative-feedback Amplifiers: C.J.M Verhoeven, et al., Pg 29*

The above highlighted quote is the accurate representation of design style based on compensation. And as such, let's first see the AC response of uncompensated op-amp to see what we can possibly get and to choose f~ugb~.

Naturally we will simulate with the worst case of \(V_{CM} = V_{DD}/2 = 0.6 V\), middle of cross-over region. Remember, we're trying to overcompensate the op-amp. So naturally, we will start from the worst case.

To that end, the testbench for measuring Open loop response is shown in *Figure-06*. The simplified schematic of Testbench is shown in *Figure-07* and the results in *Figure-08*.

<a id="fig-06"></a>

![Open Loop Gain TB](./rtr-opamp-assets/01_Uncompensated_AC_TB_dark.png#only-dark)
![Open Loop Gain TB](./rtr-opamp-assets/01_Uncompensated_AC_TB_light.png#only-light)
/// caption
**Figure-06:** Opamp schematic with open loop gain measurement configuration. (V~CM~ = 0.6 V)
///

<a id="fig-07"></a>

![Open Loop Gain TB Simplified](./rtr-opamp-assets/14_Uncompensated_AC_TB_Simplified_dark.svg#only-dark)
![Open Loop Gain TB Simplified](./rtr-opamp-assets/14_Uncompensated_AC_TB_Simplified_light.svg#only-light)
/// caption
**Figure-07:** Simplified Schematic of Figure-06
///

![Open Loop Gain of Uncompensated op-amp](./rtr-opamp-assets/02_Uncompensated_AC_Results_dark.svg#only-dark)
![Open Loop Gain of Uncompensated op-amp](./rtr-opamp-assets/02_Uncompensated_AC_Results_light.svg#only-light)
/// caption
**Figure-08:** Open Loop Gain of uncompensated op-amp in Figure-06. (V~CM~ = 0.6 V)
///

We can see that:

\[DC~Gain,~A_{DC} = 74.9~dB\]

\[Unity~Gain~frequency,~f_{ug} = 136.918~MHz\]

Improving DC gain is possible by implementing drain regulation in the cascode stack using auxiliary amplifiers. And the unity gain is 136.918 MHz, so we can set it to anything below that by adding compensation circuitry (just capacitors as shown in [Figure-05](#fig-05)).

I am not touching DC Gain here as our objective is just frequency compensation. In that case, let's make ***100 MHz*** as our unity gain bandwidth using compensation (as I want my op-amp as fast as possible after all).

Remember that each of these capacitors affect their respective current change paths (Actually they do affect other ones too, but for the sake of design, this compromise is good enough for a first order approximation). And we also know the transconductance of input MOSFETs from [Regular Threshold Voltage (RVT)](/mosfet/parameters/#regular-threshold-voltage-rvt) table which is tabulated below for convenience:

<a id="table-02"></a>

| Flavour | g~m~ | COMMENTS |
|---------|------|----------|
| NMOS-RVT | 115.83 µA/V | At VDS = 0.1 V |
| PMOS-RVT | 130.51 µA/V | At VDS = 0.1 V |
/// caption
**Table-02:** Transconductance of input MOSFETs summary
///

And the equation for unity gain frequency is:

\[\tag{1} f_{ugb} = \frac{g_{m}}{2* \pi * C_{c}}\]

We will just use NMOS transconductance, g~mn~, and 100 MHz as our f~ugb~ to find the value for capacitance.

!!! warning
    As for why we're not using PMOS transconductance as well is because *Table-02* has approximate values for g~m~ (Depends on operating point) and trying to do precision design with that is stupidity. So taking just one of them is fine as we're doing just an approximate estimate with these values.

    Finding a capacitance from *Equation-01* merely gives us a starting point (kind of like an initial guess) to our design, and then we will iterate from there in simulations.

    Again, do not attempt for precision design by hand.

Plugging values into *Equation-01* gives,

\[C_{c} = \frac{g_{m}}{2 * \pi * f_{ugb}} = \frac{115.83 µ}{2 * \pi * 100 M} = 184 fF \approx 190 fF\]

### Iteration 1

Let's see the open loop response for three different values for C~c~: 190 fF, 250 fF, and 300 fF. As for how I got those other two values, do note that 190 fF is close to 200 fF. And from 200 fF I took two other values with a step (difference) of 50 fF each. If this proves to be not enough, we can use a step of 100 fF or even 200 fF between consecutive values in each of our simulations.

Also, I won't replace 190 fF with 200 fF, since it would be useful to see what response our hand-calculated value yields.

The testbench remains same as [Figure-07](#fig-07), with the modifications on [Figure-05](#fig-05) applied to schematic of [Figure-06](#fig-06) with capacitor values of 190 fF, 250 fF and 300 fF in each step.

The simulation results are seen in *Figure-09* and the key performance parameters are tabulated in [Table-03](#table-03).

![AC results iteration 1 - 1](./rtr-opamp-assets/15_Iteration1_01_results_dark.svg#only-dark)
![AC results iteration 1 - 1](./rtr-opamp-assets/15_Iteration1_01_results_light.svg#only-light)

<a id="fig-09-c"></a>

![AC results iteration 1 - 2](./rtr-opamp-assets/15_Iteration1_02_results_dark.svg#only-dark)
![AC results iteration 1 - 2](./rtr-opamp-assets/15_Iteration1_02_results_light.svg#only-light)
/// caption
**Figure-09:** AC Simulation result of open loop gain for: (a) 190 fF, (b) 250 fF and \(c) 300 fF C~c~ values. (V~CM~ = 0.6 V)
///

<a id="table-03"></a>

| C~c~ | f~ugb~ | Φ~ugb~ | Φ~margin~ |
|------|--------|--------|-----------|
| 190 fF | 112.6 MHz | -144.2° | 35.8° |
| 250 fF | 100 MHz | -132.2° | 47.8° |
| 300 fF | 86.5 MHz | -121.9° | 58.1° |
/// caption
**Table-03:** Iteration 1 AC open loop response key parameters for V~CM~ of 0.6 V (Cross-over region).
///

From *Table-03*, one can clearly see that the hand-calculated value of 190 fF gave us an f~ugb~ which is in the ball-park of our target value of 100 MHz. That's reassuring to see, but the phase margin is too low for a stable operation across all feedback factors.

250 fF gives our target f~ugb~, but it would have been our final choice **if** the phase margin was slightly higher (above 50°).  
300 fF gives a really good phase margin, but the f~ugb~ is below our target value.

And so we have two options:

- Accept 300 fF as our final choice with a slightly lower f~ugb~.
- Improve phase margin of 250 fF case as it gives an f~ugb~ that is close to our target value.

!!! question
    You might think that aiming for 50° is still risky. And you're right. But think about our compensation strategy. ***We're overcompensating*** the op-amp. So, if I design for a phase margin of say, 75° or even 80° (just like any standard *sane* design) for just the cross-over region, that will definitely result in even larger phase margin for ICMR outside this region. ***Too large a phase margin will result in slower op-amps*** (As you will see in transient simulation section). It's a trade-off: Stability vs Speed of closed loop system.

Let's set aside the 300 fF as our fall-back option, and let's iterate once more.

### Iteration 2

If you look closely at [Figure-09-\(c)](#fig-09-c), you can see that the output pole is close to f~ugb~ and is the culprit which ends up eating our phase margin. This is inevitable as we have a heavy load at the output (10 kΩ || 10 pF).

But still, if we can move this pole, *even slightly away*, that should free up some phase margin for our **250 fF** case.

To that end our only option is to increase current in the output stage.

#### Increasing Current in output stage

!!! quote
    ... the quiescent current that flows in these devices (output NMOS and PMOS) ... To reduce this current, we can increase the lengths of the floating current sources (class AB Biasing MOSFETs - LVT) used in the folded-cascode section. This increases their gate-source voltage drops and moves the gates of the push-pull MOSFETs (output NMOS and PMOS) towards the power supply rails (shutting them off).

    \- *Ref. CMOS Circuit Design, Layout and Simulation, Pg 835.*

Even though the above quote gives us ways to reduce current, we will do the reverse (increase width) to achieve more current.

So we have two cranks for increasing current in output stage:

- Increase width of the class AB Biasing LVT MOSFETs (aka Floating Current sources) to reduce V~GS~ dropped from biasing voltage. Which in turn increases V~GS~ of output MOSFETs.
- Increase width of output MOSFETs.

Let's do both.

##### Adjust Class AB Biasing LVT MOSFETs (aka Floating Current sources)

It sounds simple enough, but ***by how much should I increase this size?***

The answer to that question remains in our [Regular Threshold Voltage (RVT)](/mosfet/parameters/#regular-threshold-voltage-rvt) table. Think about it, you want to build an op-amp with predictable performance. And to achieve that, we have choosen sizes and tabulated performance characteristics for those sizes in order to conduct design of complex circuits.

So ***it makes sense to always aim for our (chosen) desired operating point of each and every MOSFET in any circuit.***

We intend to adjust the sizes of the Floating current sources (Class AB Biasing LVT MOSFETs. Known by this term from now on for ease of typing) to ***set the biasing voltages of our output RVT MOSFETs to V~GS~ generated by our bias circuit.*** This was not the case before, because, we generated \(2 * V_{GS}\) for biasing our output MOSFETs (See the discussion on [Sizing Class AB Bias Voltage Generation MOSFET](/references/general-bias/#sizing-class-ab-bias-voltage-generation-mosfet) and [Biasing a Class AB output stage](/references/general-bias/#biasing-a-class-ab-output-stage) to understand what I mean by this) and that gets dropped more on our Floating Current sources. This is exactly why we even opted for increasing width of Floating Current sources.

I will put that intent in bullets for you to understand what I mean by this:

- Size Floating Current source, so that gate voltages of output MOSFETs match the voltage generated from bias circuit (*V~biasn~ and V~biasp~*).
- While doing so, aim for a current of 5 µA in each of those MOSFETs.

!!! warning
    In doing this, we have intentionally introduced a mismatch between the bias voltage generation MOSFET for floating current source and the MOSFET biased with that voltage. Such things must be well characterized across process corner variations before approving for production. The only reason why we even attempted to do this is because sufficiently wider and shorter MOSFETs tends to have little variations in their threshold voltage. There *IS* a mismatch, but it would be tolerable.

!!! warning
    Think of the above proposed steps as a **guideline** and nothing more. There is no way to set a current of 5 µA precisely in each of the floating current source nor to attain a specific V~GS~ drop that leaves a voltage that is close to what the bias circuit has generated in a physically realized circuit. The point of aiming for a specific current and a voltage is to define a goal which allows us to size these MOSFETs to attain an improvement in overall performance. This is done in order to avoid from mindlessly choosing sizes.

    Again, do not attempt for precision design.

*Figure-10* shows the DC annotated testbench to size Floating Current Sources. Notice how the necessary voltage conditions are copied using a VCVS to replicate the desired operating point. Also notice that both MOSFETs are roughly conducting a current of 5 µA.

![Floating Current MOSFET Sizing](./rtr-opamp-assets/12_FloatingCurrSrc_Sizing_TB_dark.png#only-dark)
![Floating Current MOSFET Sizing](./rtr-opamp-assets/12_FloatingCurrSrc_Sizing_TB_light.png#only-light)
/// caption
**Figure-10:** DC Annotated Floating Current Source sizing test bench.
///

The sizes chosen in *Figure-10* has become the **new** sizes for our Floating Current Source MOSFETs (LVT).

They are tabulated in *Table-04:*

| Flavour | W/L (Drawn) | W/L (Actual) |
|---------|-------------|--------------|
| NMOS-LVT | 26.2 / 2 | 1.7 µm / 130 nm |
| PMOS-LVT | 66.2 / 2 | 4.3 µm / 130 nm |
/// caption
**Table-04:** New sizes for Floating Current Source MOSFETs (or Class AB Biasing MOSFETs)
///

##### Increase widths of output MOSFETs

Our output MOSFETs are already 10 times wider than standard RVT Sizes. Let's double it, and make it 20 times wider and see what happens.

#### Results of these two changes

Again the same three compensation capacitors: 190 fF, 250 fF, and 300 fF. New sizes for Floating current sources from *Table-04*, and output MOSFETs are now ***20 times*** the standard sizes listed in [Table-01](#table-01).

![AC results iteration 2 - 1](./rtr-opamp-assets/16_Iteration2_01_results_dark.svg#only-dark)
![AC results iteration 2 - 1](./rtr-opamp-assets/16_Iteration2_01_results_light.svg#only-light)
/// caption
**Figure-11:** AC Simulation result of open loop gain for: (a) 190 fF, (b) 250 fF with the changes made. (Skipped 300 fF case as 250 fF has satisfied our f~ugb~ and phase margin conditions) (V~CM~ = 0.6 V)
///

| C~c~ | f~ugb~ | Φ~ugb~ | Φ~margin~ |
|------|--------|--------|-----------|
| 190 fF | 112.6 MHz | -139.2° | 40.8° |
| 250 fF | 96.9 MHz | -126.4° | 53.6° |
/// caption
**Table-05:** Iteration 2 AC open loop response key parameters for a V~CM~ of 0.6 V (Cross-over region).
///

Clearly from *Table-05* and *Figure-11* we can see that with our new changes, 250 fF itself has achieved a phase margin of 53.6° with f~ugb~ closer to target of 100 MHz.

300 fF case is skipped, as 250 fF itself has gone sligthly lower than target of 100 MHz. Anymore increase in C~c~ will definitely lower f~ugb~ even lower than 250 fF case. Also, why bother with a waste run when we've already met our requirements?

Now comes the moment of truth. ***What do we get outside cross-over region?***

#### Open loop response with V~CM~ outside cross-over region

Let's see the open loop response with common mode voltage outside cross-over region. The V~CM~ values taken to see only PMOS and only NMOS input diff amp are 0.2 V and 1.0 V (A value within their ICMRs, see [Figure-04](#fig-04) and the associated discussion beneath it) respectively.

![AC Results outside crossover](./rtr-opamp-assets/16_Iteration2_02_VariousVCM_results_dark.svg#only-dark)
![AC Results outside crossover](./rtr-opamp-assets/16_Iteration2_02_VariousVCM_results_light.svg#only-light)
/// caption
**Figure-12:** AC Simulation result of open loop gain for: (a) V~CM~ of 0.2 V and (b) V~CM~ of 1.0 V.
///

| V~CM~ | f~ugb~ | Φ~ugb~ | Φ~margin~ | A~DC~ |
|-------|--------|--------|-----------|-------|
| 0.2 V | 52.4 MHz | -105.1° | 74.9° | 67.1 dB |
| 1.0 V | 47.5 MHz | -110.4° | 69.6° | 69.5 dB |
/// caption
**Table-06:** Iteration 2 AC open loop response key parameters outside cross-over region.
///

Some observations:

- The phase margins are good in both cases.
- Reduction in f~ugb~ is trivial considering the fact that we've overcompensated the op-amp for the cross-over region (both input Diff-amps ON case).
- **Reduction in DC gain** is also due to one of the diff-amps turning OFF. Gain of cascode stage with both ON is \((g_{mn} + g_{mp}) * r_{out}\) and only \(g_{mn} * r_{out}\) or \(g_{mp} * r_{out}\) based on which input diff-amp is ON.

!!! note ""
    From [Table-02](#table-02), the transconductance of NMOS and PMOS is roughly same (\(g_{mn} \approx g_{mp}\)) and ***so we roughly see the gain getting reduced to half of what it was in cross-over region***.

    Reduced by half means 6 dB lower. So we can expect to see a gain of \(74.9~dB - 6~dB = 68.9~dB\) and our simulated values are also closer to that.

Now we know that the op-amp is fully compensated and let's put it in follower configuration and see what happens when we apply a large step from 0.1 V to 1.1 V.

## TRANSIENT Simulations

The final schematic of op-amp is shown in *Figure-13*.

![Final op-amp schematic](./rtr-opamp-assets/07_Rail_to_rail_Schematic_full_dark.png#only-dark)
![Final op-amp schematic](./rtr-opamp-assets/07_Rail_to_rail_Schematic_full_light.png#only-light)
/// caption
**Figure-13:** Final Schematic of Rail-to-rail input, class AB output op-amp. (Bias circuit not shown)
///

This final schematic should be compared with [Figure-01](#fig-01). Clearly, we have overcompensated the op-amp, and should expect to see a really slow response to this large step. This is expected as we're stepping from 0.1 V to 1.1 V, both of these are outside cross-over region and as such only one input diff-amp will be able to react to it at a time.

![Transient Step TB](./rtr-opamp-assets/17_Transient_FollowerConfig_TB_simplified_dark.svg#only-dark)
![Transient Step TB](./rtr-opamp-assets/17_Transient_FollowerConfig_TB_simplified_light.svg#only-light)
/// caption
**Figure-14:** Op-amp of Figure-13 in follower config with a large step input.
///

![Step Response](./rtr-opamp-assets/06_FollowerConfig_Tran_dark.svg#only-dark)
![Step Response](./rtr-opamp-assets/06_FollowerConfig_Tran_light.svg#only-light)
/// caption
**Figure-15:** Step response of Opamp of Figure-13.
///

The op-amp is showing a slow but stable response as discussed before.

But this is not painting the full picture. In *Figure-15*, we are stepping from 0.1 V to 1.1 V and both of these are outside cross-over region.

***What happens when we step within cross-over region??***

### Stepping within cross-over region - worst case

From [Figure-04](#fig-04), we can see that the cross-over region is from 0.4 V to 0.85 V. Let's step within this region.

And for a worst case scenario, let's step around 0.6 V as both input diff-amps are guarenteed to be ON, and that shows how stable our op-amp is.

<a id="fig-16"></a>

![Stepping within cross-over](./rtr-opamp-assets/06_FollowerConfig_Tran_02_550mV_to_650mV_dark.svg#only-dark)
![Stepping within cross-over](./rtr-opamp-assets/06_FollowerConfig_Tran_02_550mV_to_650mV_light.svg#only-light)
/// caption
**Figure-16:** Stepping within the cross-over region. (0.55 V to 0.65 V input step).
///

!!! success ""
    Don't be scared by seeing this response as the overshoot is just a mere 20 mV. The voltage scale on this curve is just 10 mV/div.

    Rather what should scare you is the settling time, as it takes 20 ns for ringing to die down even the step up and down itself happens in less that 10 ns.

This result is not surprising considering the fact that there's just a phase margin of 50° (53.6° to be precise). But this is inevitable considering the fact that we haven't implemented any constant-g~m~ control at the input stage (See the [Collapsable note](#note-collapse-1) for more info).

!!! quote
    To make sure a feedback circuit does not oscillate observe the pulse response. If there is ringing with ***fewer than 4 peaks***, the circuit is **stable**.

    \- *Ref. Desigining Analog Chips, Hans Camenzind, Pg. 6-14*

But this ringing is considered stable as per the above quote and we will stop the worst case discussion here.

### Stepping from and to cross-over region - extra cases

For the sake of completeness, let's also see what happens when I step from cross-over region and to cross-over region.

![Step from and to cross-over](./rtr-opamp-assets/06_FollowerConfig_Tran_05_04and03_Combined_dark.svg#only-dark)
![Step from and to cross-over](./rtr-opamp-assets/06_FollowerConfig_Tran_05_04and03_Combined_light.svg#only-light)
/// caption
**Figure-17:** Stepping (a) from cross-over region (0.6 V to 1.1 V) and (b) to cross-over region (0.1 V to 0.6 V).
///

Notice how settling is shorter when you reach cross-over region. In case (a) the output step down takes just 10 ns while step up takes 20 ns. And in case (b) the output step up takes 10 ns and step down takes 20 ns. This happens because both input differential amplifiers are active in the crossover region, causing changes in the cascode stack to occur more rapidly compared to when only one differential amplifier is on.

In both cases, you can see ringing when op-amp tries to settle in cross-over region, but it is not as bad as [Figure-16](#fig-16).

## Conclusion

The design of rail-to-rail input class AB output op-amp is complete.

The achieved specs are summarised below:

| V~CM~ | A~DC~ | f~ugb~ | Φ~margin~ |
|-------|-------|--------|-----------|
| 0.2 V | 67.1 dB | 52.4 MHz | 74.9° |
| 0.6 V | 74.9 dB | 96.9 MHz | 53.6° |
| 1.0 V | 69.5 dB | 47.5 MHz | 69.6° |
/// caption
**Table-07:** Spec summary
///

## QUCS-S / NGSPICE simulations

This circuit is also built and tested in [QUCS-S](https://ra3xdh.github.io/) / [NGSPICE](https://ngspice.sourceforge.io/) and the simulation results are available in [this document](https://drive.google.com/file/d/1FoIW39QAYbu-UY1ACLr4fn-CPb8MJKg4/view?usp=drive_link).