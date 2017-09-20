# design-space-example
This OpenMETA project demonstrates the variability structures and constraints available as part of the Design Space feature.

## Variability Model
Our sample project is based on component selection for the drivetrain for a heavy vehicle. We want to consider a number of diesel engines as transmissions, as well as options for a hybrid drivetrain.

[![image of system variability architecture](architecture.svg)](https://docs.google.com/drawings/d/1vpe10HUfPfzYQmR3JW2pXkALklzd0Or0NvBXizgAnLQ/edit?usp=sharing)

In this architecture, we always have a **Diesel Engine** and a **Transmission**, and we have several _**alternatives**_ for each. They are joined by a driveshaft. We will always have a **Drivetrain Control Unit** that controls these via the CAN bus.

_**Optionally**_, we may turn this into a hybrid drivetrain by adding an **Integrated Starter-Generator (ISG)** and supporting **Hybrid System Elements**. The Integrated Starter-Generator sits in between the engine and transmission and is connected to each by a driveshaft. The Hybrid System Elements include the **Battery**, for which we have several alternatives. The other Hybrid System Elements are joined to the Drivetrain Control Unit via the CAN bus.

With this variability model, there are 160 possible instantiations of the system. _**Constraints**_ offer a method for rapidly reducing the number of instances under consideration. After applying all of the constraints, only 68 of the 160 possibilities remain.

## Constraints
With _**constraints**_, we can express the criteria that each combination must meet in order to be valid. There are two types of constraints that we will consider here: _**property**_ and _**implication**_.

Constraint Concept | Type
------------------ | ----
The combined power of the engine and ISG cannot exceed the power-handling capacity of the transmission. | property
If the ISG is selected for inclusion, then the other Hybrid System Elements must also be selected, and vice-versa. | implication
The total mass of all components must not exceed 10,000 kg. | property
If an ISG is selected, it must be capable of handling the power output of the engine. | property

### The combined power of the engine and ISG cannot exceed the power-handling capacity of the transmission.
For this constraint, we add the maximum power properties of the diesel engine and ISG components, subtract the power-handling capacity of the transmission, and check that the resulting value is greater than zero.

`(Diesel Power) + (ISG Power) - (Transmission Power Capacity) > 0`

![Must Have Transmission Overhead constraint](must_have_transmission_overhead.png)

To represent this, we first take the **MaximumPower** attribute from **Engine** and the **MechanicalPowerProductionMax** attribute from **IntegratedStarterGenerator** and route them into a _**SimpleFormula**_, whose output we assign the **MaxPowerProduction**.

From **Transmission** we take the **MaxPower** attribute and connect it to a _**Custom Formula**_ while renaming the variable **TransmissionMaxPower**. We assign **MaxPowerProduction** to that same _**Custom Formula**_, and enter the following formula in its _**Expression**_ attribute:

`TransmissionMaxPower - MaxPowerProduction`

We take the output of the _**Custom Formula**_ and assign it to **TransmissionPowerOverhead**.

Finally, we create a _**Property Constraint**_ called _MustHaveTransmissionOverhead_ and attach it to **TransmissionPowerOverhead**. We set the constraint's _**TargetValue**_ to `0` and its _**TargetType**_ to `Must Exceed`.

## Component Details

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
