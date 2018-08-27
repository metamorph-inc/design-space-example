# design-space-example
This OpenMETA project demonstrates the variability structures and constraints available as part of the Design Space feature.

## Variability Model
Our sample project is based on component selection for the drivetrain of a heavy vehicle. We want to consider a number of diesel engines and transmissions, as well as options for a hybrid drivetrain.

[![image of system variability architecture](images/architecture.svg)](https://docs.google.com/drawings/d/1vpe10HUfPfzYQmR3JW2pXkALklzd0Or0NvBXizgAnLQ/edit?usp=sharing)

In this architecture, we always have a **Diesel Engine** and a **Transmission**, and we have several _**alternatives**_ for each. They are joined by a driveshaft. We will always have a **Drivetrain Control Unit** that controls these via the CAN bus.

_**Optionally**_, we can turn this into a hybrid drivetrain by adding an **Integrated Starter-Generator (ISG)** and supporting **Hybrid System Elements**. The Integrated Starter-Generator sits between the engine and transmission and is connected to each by a driveshaft. The Hybrid System Elements include the **Battery**, for which we have several alternatives, the **High-Voltage Battery Power Converter**, the **Hybrid Control Unit**, and the **Generator Control Unit**. The other Hybrid System Elements are joined to the Drivetrain Control Unit via the CAN bus.

This varability model provides us with 200 distinct drivetrain configurations. We can then use _**Constraints**_ to reduce the number of configuration instances under consideration. After applying all of the constraints, only 14 of the original 200 possibilities remain.

## Constraints
With _**constraints**_, we can express the criteria that each configuration must meet in order to be valid. There are two types of constraints that we will consider here: _**property**_ and _**implication**_.

Constraint Name | Concept | Type
--------------- | ------- | ----
[MustHaveTransmissionOverhead](#musthavetransmissionoverhead) | The combined power of the engine and ISG cannot exceed the power-handling capacity of the transmission. | property
[HybridSystemElementConsistency](#hybridsystemelementconsistency) | If the ISG is selected for inclusion, then the other Hybrid System Elements must also be selected, and vice-versa. | implication
[MassLimit](#masslimit) | The total mass of all components must not exceed 2,000 kg. | property
[MustBeAbleToHandleEnginePower](#mustbeabletohandleenginepower) | If an ISG is selected, it must be capable of handling the power output of the engine. | property
[MinimumPower](#minimumpower) | The drivetrain must be capable of producing at least 500 hp. | property

### MustHaveTransmissionOverhead
_The combined power of the engine and ISG cannot exceed the power-handling capacity of the transmission._

For this constraint, we add the maximum power properties of the diesel engine and ISG components, subtract the power-handling capacity of the transmission, and check that the resulting value is greater than zero.

`(Diesel Power) + (ISG Power) - (Transmission Power Capacity) > 0`

![Must Have Transmission Overhead constraint](images/must_have_transmission_overhead.png)

To represent this, we take the **MaximumPower** attribute from **Engine** and the **MechanicalPowerProductionMax** attribute from **IntegratedStarterGenerator** and add them using a _**SimpleFormula**_, set for `Addition`, whose output we assign to **MaxPowerProduction**.

From **Transmission** we take the **MaxPower** attribute and connect it to a _**Custom Formula**_ while renaming the variable **TransmissionMaxPower**. We assign **MaxPowerProduction** to that same _**Custom Formula**_, and enter the following formula in its _**Expression**_ attribute:

`TransmissionMaxPower - MaxPowerProduction`

We take the output of the _**Custom Formula**_ and assign it to **TransmissionPowerOverhead**.

Finally, we create a _**Property Constraint**_ called _MustHaveTransmissionOverhead_ and attach it to **TransmissionPowerOverhead**. We set the constraint's _**TargetValue**_ to `0` and its _**TargetType**_ to `Must Exceed`.

### HybridSystemElementConsistency
_If the ISG is selected for inclusion, then the other Hybrid System Elements must also be selected, and vice-versa._

For this constraint, we capture the idea that these two elements must either be selected together or not selected at all. To represent this, we create a _**Visual Constraint**_ called **HybridSystemElementConsistency**. It will ensure that, if we choose to have an integrated starter-generator, then the required supporting elements will be selected as well. It will also ensure that we don't select those supporting elements unless we have an integrated starter-generator that will benefit from their inclusion.

![HybridSystemElementConsistency](images/select_hybrid_elements.png)

Double-click **HybridSystemElementConsistency** to see its _implication_ rules. There are two references: one to **HybridSystemElements** and one to **IntegratedStarterGenerator**. The directed lines between them are marked with the word _**implies**_. This means that if **HybridSystemElements** is selected, it implies that **IntegratedStarterGenerator** was selected also. Because we have directed lines in both directions, this means that the reverse is also true.

Any _valid_ system configuration will need to either have both of these items or none of them.

![HybridSystemElementConsistency](images/select_hybrid_elements_inside.png)

### MassLimit
_The total mass of all components must not exceed 2,000 kg._

For this constraint, we want to sum the mass of all of the components in our system, and only consider configurations where the total mass is 2,000 kg or less.

We take the **Mass** property from each of the elements in our system and add them together using a _**SimpleFormula**_, set for `Addition`, sending the result to **TotalMass**. We then attach a _**PropertyConstraint**_ called **MassLimit** with _**TargetValue**_ of `2000` and _**TargetType**_ of `Must Exceed`.

![Mass Limit Constraint](images/mass_limit_constraint.png)

### MustBeAbleToHandleEnginePower
_If an ISG is selected, it must be capable of handling the power output of the engine._

This constraint only applies for configurations where the **IntegratedStarterGenerator** is selected. In those cases, we want to ensure that the ISG is rated to handle the maximum power that can be created by the engine.

At the system level, we route the **Engine**'s **MaximumPower** property into the **IntegratedStarterGenerator** _**Optional**_ container. This will make the **MaximumPower** property available to _**Optional**_ container's internal constraint definition.

![ISG Power Handling Value Routing](images/isg-power-handling-value-routing.png)

Inside the **IntegratedStarterGenerator** container, we calculate the **PowerHandlingMargin** of the ISG by taking its **MaxPowerHandling** value and subtracting the **EngineMaxPower** that we retrieved from the engine at the system level. We then attach the _**PropertyConstraint**_ **MustBeAbleToHandleEnginePower** to that value, with **TargetValue** of `0` and **TargetType** of `Must Exceed`.

![ISG Power Handling Constraint](images/isg-power-handling-constraint.png)

### MinimumPower
_The drivetrain must be capable of producing at least 500 hp._

For this constraint, we want to sum the power-producing capability of the diesel engine and, if selected, the integrated starter-generator. The resulting power must meet or exceed 500 hp.

We take the **MaximumPower** property from **Engine** and the **MechanicalPowerProductionMax** property from **IntegratedStarterGenerator** and sum them using a _**SimpleFormula**_, set for `Addition`, sending the result to the **MaxPowerProduction** property. We attach the _**PropertyConstraint**_ **MinimumPower** to that value, with **TargetValue** of `500` and **TargetType** of `Must Meet or Exceed`.

![Minimum Power](images/minimum-power.png)

### If the ISG is selected for inclusion, then the other Hybrid System Elements must also be selected, and vice-versa.

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
