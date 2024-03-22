# Proposal for IP block development for the Efabless 2024 Chipalooza challenge

IP Block name:		Ultra low-power comparator					
Designer/Design Team:	BKIC 2 team
Date:				March 22, 2024

## Circuit description:
The proposed circuit is a ultra low-power comparator designed for SkyWater 130 nm process technology. The circuit should performance should pass the specifications shown in the following table:

| Parameter                           | Min | Typical | Max        | Unit   | Notes |
| ----------------------------------- | --- | ------- | ---------- | ------ | ----- |
| Operating Temperature               | -40 |      25 |         85 |     °C |.      |
| Input Common-Mode Range             |.    |         |            |   %Vdd |       |
| Input Offset Voltage                |     |     0.1 |        0.5 |     mV | At 1pF capcitor load |
| Propagation Delay                   |     |     0.5 |.       1   |.    us | |
| Output Voltage Swing                |  0  |         |        1.8 |     mV | At 1pF capcitor load |
| Power Consumption (Enabled)         |     |         |.      85.5 |     nA | At 1pF capcitor load |
| Power Consumption (Disabled).       |     |       1 |.        10 |     nA | |
| PSRR (Power Supply Rejection Ratio) |  70 |      80 |            |     dB | |
| CMRR (Common mode Rejection Ratio)  |  80 |.        |.           |     dB | |
| Output Current Drive                |     |.    N/A |.           |     mA | |
| Input Capacitance                   |     |.    100 |.           |     dB | |
| Frequency Bandwidth                 |    5|.     20 |.       200 |    kHz | |
| Output load                         |     |       1 |.           |     pF | |




## Circuit pinout:

| Pinout | Pin name | Use |
| --- | --- | --- |
| avdd | analog power | 0-1.8V |
| dvdd | digital power | 1.8V |
| avss | analog ground | |
| dvss | digital ground | |
| ena | enable | dvdd domain |
| ibias | bandgap-controlled current bias | |
| iptat | PTAT current bias from bandgap | |
| vinn | negative input | |
| vinp | positive input | |
| vout | output voltage | |

If the top level designer should choose to implement an analog test bus (ATB), the circuit would require extra pins to select the internal voltage nodes to be measured. A good idea should be to add an extra pin to choose between the on-chip biasing currents and an external one. Another possibility is to add an offset voltage trimming pin, but the extra circuit will add complexity and area usage.

## Circuit architecture:
The chosen ultra-low comparator topology is based on the double tail comparator publish in [1]. The main difference is that the double tail comparator has two stage: the pre-amplifier stage has small current for week inversion and obtain long integration interval and better Gm/I ratio; latch stage has a large tail current for fast regeneration, as in [2]. Besides, this circuit do not need a high input voltage or stacking of too many transistors as others dynamic comparator, and latch stage has positive feedback so that it can meet the required speed in the ADCs.
![ULP_COM](https://github.com/vietduc1210/EFAB_ULP_COM/assets/41568734/88bc9270-9728-4b2c-bb1b-54e92a500dd1)
Two stages are controlled by a pair of counter-phase clock signals, operating in two alternating phases.
When CLK = 0, the comparator operates in reset mode, M0 and M5 are off so the circuit does not consume power in this phase. M3 and M4 are turned on, pulling the output voltage of stage 1 VO1P and VO1N to a high VDD level creating initial conditions for the next phase. Similarly, M6 and M7 are turned on pulling the outputs of stage 2 VO2P and VO2N to ground. When CLK = VDD, the circuit enters the comparison phase. M0 is turned on while M3 and M4 are turned off, the voltages VO1P and VO1N on the parasitic capacitor at the output of stage 1 are discharged to ground through M1 and M2. The difference between the two input signals will create a difference in the current flowing through the two branches of stage 1, creating a different voltage discharge rate between the two output voltages. The output of pre-amplifer stage is fed into the next stage through M6 and M7. M5 is on, latch stage is active, the pair M8, M10 and M9, M11 crossed (cross coupled) are tasked with pushing one of the two output points of stage 2 to VDD level while the other output goes to ground.


## Specification challenges:
The chosen topology rail-to-rail operation is relatively easy, as there is no need for constant bandwidth for the full range common-mode input voltage. No input voltage offset was requested, so the major area constraint specification will be dictated by the transistors flicker noise.

The biggest constraint resulting from the demanded specifications is the minimum voltage supply. The amplifier should be able to use only 300 uA no load, and 1.8 mA with a 5 kOhm maximum load. The main limitation of the class AB output stage is the supply voltage needed to deliver an excess current of 1.5 mA and still function properly with a 3.0 V minimum voltage supply for SS corner at -40 °C.

## Testbenches required for verifying circuit performance:
The simulation testbenches cover mostly DC and AC performance characterization (DC gain, input and output range, power consumption, output current, bandwidth, phase margin, PSRR, CMRR, input referred noise), and transient ones (THD, slew-rate). All those simulation testbenches are available at the original article’s repository [1], based on the diagram available at [3]. 

## Connections required for standalone (breakout) implementation:
The actual circuit input pins should be located as close as possible to the pads. The parasitic input current is mostly composed of the ESD protection. The output current can be quite high, so extra care must be taken for the output node and the supply nets of the output stage.

Current consumption measurements are only possible if the circuit has a dedicated power supply pin. If the circuit is working as intended, the current consumption can be inferred by the biasing current, which certainly will be measured from the on-chip standalone current bias generator. ATB points could be inserted to measure the biasing voltages, but are not strictly necessary.

## Test plan for standalone (breakout) implementation:
The proposed circuit, an operational amplifier, can be tested in most university labs, with conventional oscilloscopes. The proposed circuit specifications do not require high speed, ultra-low-noise, and ultra-low-current measurement equipment. Detailed measurement guidelines can be found in Analog Devices tutorials [4], among many other references.

## References
[1] S. Aakash, A. Anisha, G. Jaswanth Das, T. Abhiram and J. P. Anita. "Design of a low power, high speed double tail comparator" 2017 International Conference on circuits Power and Computing Technologie. https://ieeexplore.ieee.org/document/8074370
[2] Shivam Singh Baghel and D. K. Mishra. "Design and Analysis of Double-Tail Dynamic Comparator for Flash ADCs" 2018 International Conference on Circuits and Systems in Digital Enterprise Technology. https://ieeexplore.ieee.org/document/8821129

