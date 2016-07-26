Synthea contains a framework for defining modules using JSON.  A JSON module configuration describes a progression of states and the transitions between them.  On each Synthea generation "cycle", the generic framework processes states one at a time to trigger conditions, encounter, medications, and other clinical events.

# States

The generic module framework currently supports the following states:

* Initial
* Terminal
* Guard
* Delay
* Encounter
* ConditionOnset
* MedicationOrder
* Procedure
* Death

The following states are also planned for future implementation:

* Lab
* ConditionEnd
* MedicationEnd

# Transitions

The generic module framework currently supports the following transitions:

* Direct
* Distributed

The following transitions are also planned for future implementation:

* Conditional

# Logic

The Guard state and Conditional transition use conditional (boolean) logic.  The following condition types are currently supported:

* Gender
* Age
* And
* Or
* Not
* True
* False
