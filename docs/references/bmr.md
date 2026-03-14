---
description: Beta Multiplier Reference (BMR) — supply-independent bias current circuit on 65nm custom CMOS, designed and simulated on Cadence Virtuoso and Spectre.
---

--8<-- "includes/disclaimer.md"

# Beta Multiplier Reference (BMR)

## Introduction

I have documented only the design procedure, and anything relevant to that. For theory of operation, please refer to standard textbooks or any lectures.

Before we proceed, let's take a look at the schematic of BMR.

<a id="fig-01"></a>

![BMR Schematic Diagram](./bmr-assets/12_BMR_Schematic_JBK_light.svg#only-light)
![BMR Schematic Diagram](./bmr-assets/12_BMR_Schematic_JBK_dark.svg#only-dark)
/// caption
**Figure-01:** BMR Schematic Diagram [Ref. CMOS Circuit Design, Layout and Simulation, Fig 20.22]
///

For designing this, we need to chose values for three things:

- Output Current
- MOSFET sizes (For both NMOS and PMOS) for chosen current
- Resistor Value

For this documentation, the output current is chosen to be **10 µA**.

The MOSFET sizes have already been selected for this current, and their parameters are thoroughly listed in [Regular Threshold Voltage (RVT)](/mosfet/parameters/#regular-threshold-voltage-rvt) table. For the sake of convenience, sizes are listed here:

<a id="table-01"></a>

| Flavour | W/L (Drawn) | W/L (Actual) | I~D~ |
|---------|-------------|--------------|------|
| NMOS | 43.1 / 2 | 2.8 µm / 130 nm | 10 µA |
| PMOS | 121.5 / 2 | 7.9 µm / 130 nm | 10 µA |
/// caption
**Table-01:** Sizes and Bias Current summary
///

That leaves us with **chosing value of resistor**.

## Fixing Resistor value

BMR sets the current by dropping the difference in V~GS~ of both transistors across the resistor. A Square-law driven approach is useless, as the MOSFETs we use have shorter channel.

The key to designing this lies in taking advantage of two characteristics of the circuit:

1. Drains of both NMOS are at same potential because of regulating action of amplifiers.
2. We're already aware of the target current, which is 10 µA.

Point number 2 may appear to be stating the obvious, but this will be shown to be incorrect once we remember an essential network theory concept.

### Substitution theorem

*Once we know the current through and the voltage across an element in a network, we can substitute that element with an ideal current or voltage source having the same value as the original element.*

![Subsititution theorem example](./bmr-assets/13_Substitution_Theorem_dark.svg#only-dark)
![Subsititution theorem example](./bmr-assets/13_Substitution_Theorem_light.svg#only-light)
/// caption
**Figure-02:** Demonstration of substitution theorem to an arbitrary network.
///

By now, it should be obvious how this will help us. We will use this in the reverse manner. **We will substitue the resistor with an ideal current source of 10 µA**, and then measure the voltage dropped across it. That automatically gives us the difference of V~GS~ dropped across it.

Once we know that voltage, we can compute the impedance seen into the current source, and *that gives us the resistance needed for BMR to generate 10 µA*

### Resistance computation testbench

Along with substitution theorem, we will also use point number 1 to set both MOSFETs with same potential at drain.

In *Figure-03*, **notice how I am generating the drain potential** by pumping a current of 10 µA into the left MOSFET and then copying it with an ideal VCVS of gain 1 to the right MOSFET instead of using a voltage source with some fixed value.

![BMR Resistor Value testbench](./bmr-assets/01_ResistorValue_dark.png#only-dark)
![BMR Resistor Value testbench](./bmr-assets/01_ResistorValue_light.png#only-light)
/// caption
**Figure-03:** Testbench for finding value of resistance. Annotated with DC values.
///

Naturally, even this is an approximation as the drain potentials in the actual circuit will not be exactly the same because of finite gain of amplifier, but still, *this is a good approximation for design*, which will be justified when we see the result.

From the Simulation results, we can see that it drops a voltage of **61.4 mV** to generate 10 µA.

\[Resistance, R = \frac{61.4 m}{10 \mu} = 6140 \Omega \approx 6.1 k\Omega\]

Let's simulate just the BMR core without startup to see DC performance and what current we generate.

## DC Simulations

With the resistor value we just computed and the sizes tabulated for MOSFET, let's build it and run a DC.

<a id="fig-04"></a>

![BMR DC Simulation](./bmr-assets/02_BMR_DC_dark.png#only-dark)
![BMR DC Simulation](./bmr-assets/02_BMR_DC_light.png#only-light)
/// caption
**Figure-04:** DC annotated schematic BMR. (Resistance adjusted to 6.2 kΩ.)
///

From *Figure-04*, we can see that the drain currents are indeed 10 µA (observe only for right and left MOSFET, and not for the MOSFETs that form the amplifier). And our core design is complete.

!!! warning
    Do not attempt for precise value of 10 µA by fine tuning the value of resistance. It is a futile thing to do as MOSFETs themselves have process variations and an IC resistor simply cannot be set to a precise value not to mention the horrible tolerance of it's nominal value. When we say the current is 10 µA, it is nominally 10 µA and it could be 9.8 µA or even 10.5 µA. This precise design methodology is just an illusion spread among the community and do not fall for it!

### Supply Dependance

*Figure-05* shows the drain currents of left and right MOSFETs when supply voltage is swept from GND to V~DD~ (1.2 V).

![BMR Supply Dependance](./bmr-assets/03_ID_VS_Supply_dark.svg#only-dark)
![BMR Supply Dependance](./bmr-assets/03_ID_VS_Supply_light.svg#only-light)
/// caption
**Figure-05:** Drain Current vs Supply
///

We can see that the supply can go as low as 0.7 V and our reference will still function.

## Startup Circuit Design

The design procedure of Startup circuit revolves around two things:

- When the circuit has not started, V~GS~ generated for BMR NMOS will be near GND. In that case, Leaker MOSFET must be ON (or, the gate of Leaker MOSFET should be near V~DD~).
- Vice versa, when the circuit has started.

To that end, if the gate of Leaker NMOS comes to say 100 mV, it would be near GND, and should be enough to shut it off. Of-course, in order to avoid leakage current in an OFF NMOS transistor, the gate should be pulled to almost 0 V. But, even for 100 mV, it will just leak 100's of pA which gets driven into a diode connected NMOS, and that's hardly a significant current compared to 10 µA (10 * 10^6^ pA) which is just huge. Also, diode connected MOSFETs can easily source/sink more current with just mV level increase in their V~GS~ (as they have low impedance).

### Startup Pull up and Pull Down Testbench

For the pull up, a diode connected PMOS is used. Now, we need this to be weak (or **capable of sourcing low current compared to pull down**), so, let's take a size of `2/10` (or `130 nm/650 nm`, scale factor of 65 nm).

Let's see what current will this give when it's drain is at 100 mV.

![Startup Pull up and Pull Down TB](./bmr-assets/05_StartupDesign_DC_dark.png#only-dark)
![Startup Pull up and Pull Down TB](./bmr-assets/05_StartupDesign_DC_light.png#only-light)
/// caption
**Figure-06:** Testbench for choosing Pull up and Pull Down.
///

To the leftmost side of *Figure-06*, you can see a Diode Connected PMOS with a Drain voltage of 100 mV. We can see that the current it sources in this case is just 5.84 µA.

This is good, as our pull down NMOS, when used with standard size listed in [Table-01](#table-01) , it tries to sink 10 µA. ***Clearly more than what the PMOS can source, and so it is guarenteed to pull the Drain to less than 100 mV!***.

To the right side of *Figure-06*, you can see two different NMOS pull down. One having a size of `43.1/2` (standard size, see [Table-01](#table-01)) in the centre and another having `86.2/2` (Double the width of standard size).

The reason for doubling width should be obvious, if we pull 20 µA instead of 10 µA, it should pull the drain even lower than what the standard size can achieve.

!!! note
    Doubling width is achieved by keeping another MOSFET in parallel, and not by actually doubling width of the same MOSFET. This is done for matching purpose. On the occasion when you do double width of same MOSFET, it is recommended to add fingers to it. Again for matching purpose.

And clearly, each pull their drains to 75 mV and 30 mV respectively. From this, both can be used, but just to leave a margin for error ***Twice the width is made final.***

### Leaker NMOS Testbench

The leaker NMOS is chosen arbitrarily with the minimum length. Let's start with `10/1` (or `650 nm/65 nm`) and see how much current it can leak.

![Leaker MOS Design TB](./bmr-assets/06_LeakerDesign_dark.png#only-dark)
![Leaker MOS Design TB](./bmr-assets/06_LeakerDesign_light.png#only-light)
/// caption
**Figure-07:** Current leaked by a 10/1 NMOS.
///

In the left of *Figure-07*, we see the Started case. This is simulated by remembering that when the BMR has turned ON and has stabilised, it generates a V~GS~ of 500 mV (for the standard size and chosen bias current, see [Table-01](#table-01)), which is directly applied to gate of Pull down NMOS using a voltage source.  
In the full schematic, the drain of leaker NMOS is connected to gate of BMR PMOS, and the source is connected to drain of BMR NMOS (See [Figure-01](#fig-01)). And when started, each of them are at \(V_{DD} - V_{SG,P} = 1.2 - 0.47 = 0.73 V\) and \(V_{GS,N} = 0.5 V\) (Again, see [Table-01](#table-01)).  
This is modelled using voltage sources to the drain and source of leaker NMOS each with appropriate value.  
The current leaked is just 14 fA (too less to disturb our bias current, meaning **our startup circuit is out of picture.**)

In the right of *Figure-07*, we see the initial case, i.e., not started case. And this is easily simulated by setting 0 V to the gate of Pull down NMOS, \(V_{DD} - V_{SG,P} = 1.2 - 0 = 1.2 V\) to the drain of leaker NMOS, and \(V_{GS,N} = 0 V\) to the source of leaker NMOS.  
And this is now leaking 262 µA, more than enough to kickstart the circuit.

Since the leaker performance is satisfactory for `10/1`, it is fixed.

## Startup Action - Transient Simulation

Let's attach the startup circuit to the BMR schematic of [Figure-04](#fig-04), and let's apply a voltage step to supply to see what happens. But before we do that, we need to **compensate the error amplifier** to ensure stability.

Since the error amplifier is just a diff-amp, it can be easily compensated by placing a capactor at it's output. This doesn't need a passive capacitor, and *we can just use a MOSFET as a capacitor*.

To that end, we have two options:

- NMOS as a capacitor (to GND)
- PMOS as a capacitor (to VDD)

### Using MOSFET as a capacitor to compensate the feedback loop

The error amp is driving the gate of BMR PMOS which is a high impedance node. This can be made the dominant pole by adding capacitance to it.

[Mosfet as a capacitor](/mosfet/parameters/#core-device-65-nm_2) Table has capacitance values for some standard sizes, which are repeated here for convenience:

| Flavour | Drawn Size | Actual Size | V~G~ | Nominal Capacitance |
|---------|------------|-------------|------|---------------------|
| NMOS-RVT | 20 / 20 | 1.3 µm / 1.3 µm | > 0.6 V | 20.9 fF |
| PMOS-RVT | 20 / 20 | 1.3 µm / 1.3 µm | < 0.6 V | 20.4 fF |
/// caption
**Table-02:** MOSFET capacitance for specific sizes and Voltage conditions
///

From this we can see that the voltage at the gate of BMR PMOS (or outuput of error amp) dictates whether NMOS or PMOS can be used. Nominally this will be at \(V_{DD} - V_{SG,P} = 1.2 - 0.47 = 0.73 V\), and looking at Table-02, it seems we need to pick NMOS (as \(V_{G} > 0.6 V\)), but it is not that simple.

#### What happens with a capacitor to GND (or NMOS Capacitor)?

![NMOS Capacitor as load](./bmr-assets/14_NMOSCap_As_Load_dark.svg#only-dark)
![NMOS Capacitor as load](./bmr-assets/14_NMOSCap_As_Load_light.svg#only-light)
/// caption
**Figure-08:** BMR loaded with NMOS capacitor.
///

When circuit is OFF, all nodes will be at GND and the capacitor is initially discharged. As the supply steps up to it's nominal value, the source of BMR PMOS is pulled up, while the gate of BMR PMOS, initially gets pulled up (as MOSFET doesn't act as a good capacitor before it gets inverted or turned ON, see *Figure-09*), but once it crosses 0.6 V, NMOS starts to act as a capacitor, and **will try to hold that node to that same value**.

![NMOS CV Curve](./bmr-assets/08_CV_Curve_20by20_dark.svg#only-dark)
![NMOS CV Curve](./bmr-assets/08_CV_Curve_20by20_light.svg#only-light)
/// caption
**Figure-09:** NMOS CV Curve for 20/20 size. (Curve title is wrong as I generated this for both 10/10 and 20/20 at the same time.)
///

Due to this, we will see a sudden spike in the drain currents of BMR, and,

!!! danger
    The larger the capacitance at this node, the longer it takes to charge it to its operating point, and the greater the magnitude of the current spikes becomes.

*Figure-10* shows this case, where the supply starts to turn ON at 10 ns with a rise time of 20 ns to reach VDD at 30 ns. Here, we can see that there is indeed a current spike around the 20 ns mark where the supply is still in a transient state.

![Startup action with NMOS cap](./bmr-assets/10_StartupAction_NMOSCAP_20by20_5_dark.svg#only-dark)
![Startup action with NMOS cap](./bmr-assets/10_StartupAction_NMOSCAP_20by20_5_light.svg#only-light)
/// caption
**Figure-10:** Startup transient with five 20/20 NMOS in parallel (Nominal Capictance of 100 fF).
///

!!! note
    Even though the NMOS is suitable for this operating voltage, it causes these transient spikes during startup. And the larger the capacitance it offers, the greater these spikes becomes.

!!! info
    If this is the case, I think we can do away with the startup circuit as the BMR PMOS is guarenteed to push a current into BMR NMOS which in turn, starts up the circuit. But the above simulation was done with a startup included, and it was observed that the startup barely played it's role.

#### What happens with a capacitor to VDD (or PMOS Capacitor)?

<a id="fig-11"></a>

![PMOS Capacitor as load](./bmr-assets/15_PMOSCap_As_Load_dark.svg#only-dark)
![PMOS Capacitor as load](./bmr-assets/15_PMOSCap_As_Load_light.svg#only-light)
/// caption
**Figure-11:** BMR loaded with PMOS capacitor.
///

For starters, the capacitance varies heavily with voltage at that node, as backed by CV curve of PMOS in *Figure-12*.

![PMOS CV Curve](./bmr-assets/09_CV_Curve_PMOS_20by20_dark.svg#only-dark)
![PMOS CV Curve](./bmr-assets/09_CV_Curve_PMOS_20by20_light.svg#only-light)
/// caption
**Figure-12:** PMOS CV Curve for 20/20 size. (Curve title is wrong as I generated this for both 10/10 and 20/20 at the same time.)
///

Especially our operating point of 730 mV is not in strong inversion region and from *Figure-12* we can see that it has a capacitance of 16.1 fF. Not good.

But when you think about the transient response, with discharged state, the gate of BMR PMOS will move along with Supply (or source of BMR PMOS, see [Figure-11](#fig-11)) and will not stay at GND as with NMOS Capacitor case, and so, ***we will not see any spikes,*** as the \(V_{SG} \approx 0 V\), till the startup tries to pull it down. Because, even though it's capacitance is not reaching it's nominal value, it still behaves as a capacitor to some extent.

!!! danger
    Due to this, we need to place a really large PMOS to achieve the same amount of load offered by an NMOS. Also as the capacitance varies with voltage and still won't settle to a nominal value, the startup takes too long.

!!! success
    But on the bright side, we will not see any current spikes as seen from *Figure-13*

![Startup action with PMOS cap](./bmr-assets/10_StartupAction_PMOSCAP_20by20_10_dark.svg#only-dark)
![Startup action with PMOS cap](./bmr-assets/10_StartupAction_PMOSCAP_20by20_10_light.svg#only-light)
/// caption
**Figure-13:** Startup transient with ten 20/20 PMOS in parallel (Nominal Capictance of 160 fF).
///

Just as we speculated, the reference is taking a long time to turn ON as seen in *Figure-13* (around 90 ns mark). Also see the control voltage of leaker NMOS, \(V_{LEAK}\) being 0.66 V all the time where the reference has remained OFF, meaning, that our startup circuit is working as expected.

#### Which one should you use?

Honestly, I have no idea. So, I am going to standard textbooks, and use PMOS as a capacitor. Even though the capacitance variation makes the circuit take longer to turn ON, it doesn't produce any spikes, and the startup is guarenteed to safely turn ON the reference.

So, I will go with PMOS.

## Conclusion

![Full schematic of BMR](./bmr-assets/11_Full_Schematic_dark.png#only-dark)
![Full schematic of BMR](./bmr-assets/11_Full_Schematic_light.png#only-light)
/// caption
**Figure-14:** Final schematic of BMR with sizes from Table-01 and others from the rest of this documentation.
///

The design of BMR is complete.