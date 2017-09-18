# design-space-example
This OpenMETA project demonstrates the variability structures and constraints available as part of the Design Space feature.

## Variability Model
Our sample project is based on component selection for the drivetrain for a heavy vehicle. We want to consider a number of diesel engines as transmissions, as well as options for a hybrid drivetrain.

[![image of system variability architecture](architecture.svg)](https://docs.google.com/drawings/d/1vpe10HUfPfzYQmR3JW2pXkALklzd0Or0NvBXizgAnLQ/edit?usp=sharing)

In this architecture, we always have a **Diesel Engine** and a **Transmission**, and we have several _**alternatives**_ for each. They are joined by a driveshaft. We will always have a **Drivetrain Control Unit** that controls these via the CAN bus.

_**Optionally**_, we may turn this into a hybrid drivetrain by adding an **Integrated Starter-Generator** and supporting **Hybrid System Elements**. The Integrated Starter-Generator sits in between the engine and transmission and is connected to each by a driveshaft. The Hybrid System Elements include the **Battery**, for which we have several alternatives. The other Hybrid System Elements are joined to the Drivetrain Control Unit via the CAN bus.

Engine | Mass | Power | Price
------ | ---- | ------------- | -----
C7 | 588 kg | 300 hp | $13,500
C9 | 864 kg | 375 hp | $18,500
C13 | 908.4 kg | 520 hp | $25,500
C15 | 1600 kg | 580 hp | $26,500
C18 | 1650 kg | 800 hp | $40,000

Transmission | Mass | Power | Price
------------ | ---- | ----- | -----
CX 28 | 255 kg | 400 hp | $3,250
CX31-P600 | 456 kg | 600 hp | $9,000
CX35-P800 | 650 kg | 800 hp | $14,000
TH35-E81 | 973 kg | 410 hp | $23,000

Battery | Mass | Capacity | Voltage
------- | ---- | -------- | -------
A | 175 kg | 23 kWh | 393 V
B | 53.3 kg | 1.31 kWh | 201.6 V
C | 105.1 kg | 4.4 kWh | 288 V
D | 141.2 kg | 8.8 kWh | 201.6 V
