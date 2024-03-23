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
The chosen ultra-low comparator topology is based on the double tail comparator publish in [1]. The main difference is that the double tail comparator has two stage: the pre-amplifier stage has small current for week inversion and obtain long integration interval and better Gm/I ratio; latch stage has a large tail current for fast regeneration, as in [2].
!["Double tail comparator schematic"](https://github.com/vietduc1210/EFAB_ULP_COM/assets/41568734/88bc9270-9728-4b2c-bb1b-54e92a500dd1)

Two stages are controlled by a pair of counter-phase clock signals, operating in two alternating phases.


When CLK = 0, the comparator operates in reset mode, M0 and M5 are off so the circuit does not consume power in this phase. M3 and M4 are turned on, pulling the output voltage of stage 1 VO1P and VO1N to a high VDD level creating initial conditions for the next phase. Similarly, M6 and M7 are turned on pulling the outputs of stage 2 VO2P and VO2N to ground. When CLK = VDD, the circuit enters the comparison phase. M0 is turned on while M3 and M4 are turned off, the voltages VO1P and VO1N on the parasitic capacitor at the output of stage 1 are discharged to ground through M1 and M2. The difference between the two input signals will create a difference in the current flowing through the two branches of stage 1, creating a different voltage discharge rate between the two output voltages. The output of pre-amplifer stage is fed into the next stage through M6 and M7. M5 is on, latch stage is active, the pair M8, M10 and M9, M11 crossed (cross coupled) are tasked with pushing one of the two output points of stage 2 to VDD level while the other output goes to ground.


## Specification challenges:
In analyzing the performance of our dynamic comparator circuit, it's evident that the majority of the specifications outlined have been satisfactorily met, signaling a promising achievement in its design and functionality. 

However, a notable limitation persists, primarily centered around the input common mode range. As stipulated by the requirements, it's imperative for the input common mode range to span from 5% to 95% relative to VDD. Regrettably, our current circuit configuration falls short in this aspect, as the ratio of the input common mode range to VDD remains significantly lower than desired due to the inherently limited range of the input common mode. 

This shortfall poses a considerable challenge, necessitating further exploration and refinement to ensure compliance with the specified operational parameters and to enhance the circuit's overall performance and versatility.

## Testbenches required for verifying circuit performance:
The simulation testbenches cover mostly DC and AC performance characterization (input and output range, power consumption, output current, bandwidth, PSRR, CMRR). 
### Power consumption

## Connections required for standalone (breakout) implementation:
The circuit uses a VDD pin for stable voltage supply, connecting stages 1 and 2 using two dedicated pins, out1n and out1p. These pins act as both outputs and inputs, facilitating seamless signal transfer and processing. The outputs are channeled out via two output pins, out2n and out2p, to the gate terminals of M8 and M10, and out2p to the gate terminals of M9 and M11. One output pin measures the VDD voltage, while the other gauges a voltage of 0V at the auxiliary battery for performance monitoring. The CLK battery is connected to MOSFET gate terminals to regulate timing and ensure synchronous operation. The CLK signal is configured as a square pulse waveform, with a low voltage of 0V and a high voltage corresponding to VDD at various frequencies such as 10kHz, 20kHz, 30kHz…Additionally, the rise and fall times of the CLK signal are precisely controlled to maintain temporal integrity, with both rising and falling edges exhibiting a consistent transition time of 100ns. This comprehensive configuration ensures optimal performance, precise timing control, and robust signal processing capabilities within the circuit.
## Test plan for standalone (breakout) implementation:
The proposed circuit, an operational amplifier, can be tested in university labs, with conventional oscilloscopes. The proposed circuit will be tested with input offset, power consumption and frequency bandwidth.
## References
[1] S. Aakash, A. Anisha, G. Jaswanth Das, T. Abhiram and J. P. Anita. "Design of a low power, high speed double tail comparator" 2017 International Conference on circuits Power and Computing Technologie. https://ieeexplore.ieee.org/document/8074370

[2] Shivam Singh Baghel and D. K. Mishra. "Design and Analysis of Double-Tail Dynamic Comparator for Flash ADCs" 2018 International Conference on Circuits and Systems in Digital Enterprise Technology. https://ieeexplore.ieee.org/document/8821129


