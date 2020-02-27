## Introduction

Synthea has the concept of a [HealthRecordEditor](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/engine/HealthRecordEditor.java). This is a tool that is able to manipulate an individual's [HealthRecord](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/world/concepts/HealthRecord.java) throughout the simulation. Editors differ from [Synthea modules](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework) in that they are intended to only simulate actions that take place on an individual's electronic health record. They are not intended to be used to simulate physiological phenomena or any actions that the individual themselves would undertake.

An example of an editor is the [GrowthDataErrorsEditor](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/editors/GrowthDataErrorsEditor.java). This editor simulates errors that typically take place when manually entering observed heights and weights into an electronic health record system.

## Execution in a Synthea Simulation

All `HealthRecordEditor`s should register with the [HealthRecordEditors](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/editors/GrowthDataErrorsEditor.java) class using the `registerEditor` method. `HealthRecordEditors` is a [singleton class](https://en.wikipedia.org/wiki/Singleton_pattern) used to manage all `HealthRecordEditor`s in a simulation.

At the end of each time step in a simulation `HealthRecordEditors` will iterate through its list of registered modules, calling the `shouldRun` method. This provides each `HealthRecordEditor` a chance to decide whether or not it should take action during the time step. As an example, the `GrowthDataErrorsEditor` only operates on individuals under 20 years of age. It will respond with `false` any time `shouldRun` is invoked and the individual is over 20 in the simulation.

If the `shouldRun` method returns `true`, `HealthRecordEditors` will invoke the `process` method to allow editing of the `HealthRecord`

## Creating a Health Record Editor

Developers who want to create a new editor must implement the `org.mitre.synthea.engine.HealthRecordEditor` interface. All editors must be placed in the `org.mitre.synthea.editors` package.

As mentioned previously, new editors must register with the `HealthRecordEditors` class. It is good practice to allow new editors to be enabled via configuration. This means making the registration call in a code block that is controlled by a configuration property.