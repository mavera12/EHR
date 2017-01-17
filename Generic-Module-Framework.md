Synthea contains a framework for defining modules using JSON.  These JSON modules describe a progression of states and the transitions between them.  On each Synthea generation time-step, the generic framework processes states one at a time to trigger conditions, encounters, medications, and other clinical events.

![ear infection](https://cloud.githubusercontent.com/assets/13512036/18751952/e054e258-80ae-11e6-9b09-2350ed77b56c.png)


This simplified example of childhood ear infections shows the flow of a generic module. In this instance, children get ear infections at different rates based on their age, are then diagnosed at an encounter, and then are prescribed either an antibiotic or a painkiller. 

Future plans for the generic module framework include a web interface to allow clinicians or other healthcare professionals to design Synthea modules with no programming experience required.


Details and examples for these concepts are available on the following pages:

### [[Basics|Generic Module Framework: Basics]]
### [[States|Generic Module Framework: States]]
### [[Transitions|Generic Module Framework: Transitions]]
### [[Logic|Generic Module Framework: Logic]]
### [[Submodules|Generic Module Framework: Submodules]]
### [[Logging|Generic Module Framework: Logging]]
### [[Complete Module Example: Examplitis|Generic Module Framework: Example]]

***


## Relevant Files and Paths

Within the _synthea_ repository, the following files and paths are relevant to the generic module framework:

* **`lib/generic/modules/`**: the path containing the JSON module files that Synthea should process
* **`lib/generic/context.rb`**: the `Context` class definition; responsible for maintaining a module's state definition, state history, and current state for each patient.  The `Context` class also processes states and transitions.
* **`lib/generic/logic.rb`**: the `Logic` module implements the methods corresponding to the supported logical condition types
* **`lib/generic/states.rb`**: the `States` module implements the supported state types
* **`modules/generic.rb`**: the `Generic` module defines the Synthea `rule` that is executed on every time-step.  It loops through the modules in `lib/generic/modules/`, setting up and running the context for each module.
* **`test/fixtures/generic/*.json`**: the test fixtures for all of the generic module framework unit tests
* **`test/generic_*_test.rb`**: the unit tests for the generic module framework's context, logic, and state implementations