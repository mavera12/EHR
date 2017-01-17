The Generic Module Framework supports the concept of "submodules". A submodule is a reusable series of states that can be used at any time by any other generic module. In general, submodules are used to define a series of states that are either:

1.  Used repeatedly in the same module, or
2.  Used repeatedly by several different modules

For example, in Synthea submodule are used to:

* Perform several Observations and Procedures that constitute a pregnancy workup. These states are reused at every pregnant patient's visit to the doctor, more than 10 times within the `pregnancy` module.

* Prescribe a common medication, such as an over-the-counter pain reliever, or antihistamine. The `medications/otc_pain_reliever` submodule is used by several main modules in Synthea where a pain reliever is needed.

Submodules may themselves call submodules, although this is uncommon.

## Main Modules vs. Submodules

All modules and submodules are kept in the `lib/generic/modules/` directory. For example, consider the following hierarchy:

```
modules/
  asthma.json
  diabetes.json
  medications/
    otc_pain_reliever.json
    otc_antihistamine.json
  pregnancy.json
```

All modules at the **top-level** of the `modules/` directory are considered "main modules". When generating a new patient record, Synthea starts with all main modules _only_. Depending on the progression of the main modules, submodules may then be called. In the example above, `asthma`, `diabetes`, and `pregnancy` are main modules.

Any modules **in folders** within the `modules/` directory are considered "submodules". Synthea will not use these modules to generate a new patient record unless they are explicity called by a main module using a `CallSubmodule` state. In the example above, `medications/otc_pain_reliever` and `medications/otc_antihistamine` are submodules.

## The Global `MODULES` Lookup

At runtime, all modules and submodules are loaded into the global `Synthea::MODULES` lookup. Each module is assigned a unique `key` that refers to that module within the lookup. In the example above, the `MODULES` lookup would contain the following:

#### Main Modules:

* `MODULES['asthma']` would contain the "Asthma" module
* `MODULES['diabetes']` would contain the "Diabetes" module
* `MODULES['pregnancy']` would contain the "Pregnancy" module

#### Submodules:

* `MODULES['medications/otc_pain_reliever']` would contain the "Over-the-Counter Pain Reliever" module
* `MODULES['medications/otc_antihistamine']` would contain the "Over-the-Counter Anthistamine" module

The `key` that refers to a module is derrived from that module's filename. Keys to submodules always start with the name of the directory they're in, followed by the name of the submodule.

## How the `CallSubmodule` State Works

Each main module runs in a "context" that keeps track of states available to the module, a history of states that have been processed, and the `current_state` that is being processed in a given time step. When a `CallSubmodule` state is processed Synthea does the following:

1. Saves the `key` that refers to the current module being processed so it can be retrieved and resumed later.
2. Saves the `current_state` of the current module being processed. In practice this is always the `CallSubmodule` state so it's transition may be used when the submodule eventually returns.
3. Loads the submodule from `Synthea::MODULES` and resumes processing states _in the same time step_, starting with the submodule's `Initial` state.

A submodule "returns" when it's `Terminal` state is reached. At that time, Synthea does the following:

1. Restores the calling module from `Synthea::MODULES` using the saved `key`.
2. Restores the context's `current_state` using the saved `current_state`.
3. Resumes processing states _in the same time step_ starting with the next state that the `CallSubmodule` state transitions to.

### Example

Consider a submodule with one `MedicationOrder` state that prescribes a medication. When processed, it's history may look like:

```
/===============================================================================
| Medication Submodule Log
|===============================================================================
| Entered                   | Exited                    | State
|---------------------------|---------------------------|-----------------------
| 2017-01-17T09:50:41-05:00 | 2017-01-17T09:50:41-05:00 | Initial
| 2017-01-17T09:50:41-05:00 | 2017-01-17T09:50:41-05:00 | Prescribe_Medication
| 2017-01-17T09:50:41-05:00 | 2017-01-17T09:50:41-05:00 | Med_Terminal
\===============================================================================
```

When that submodule is called by a main module, their collective history may look like:

```
/===============================================================================
| Main Module Log
|===============================================================================
| Entered                   | Exited                    | State
|---------------------------|---------------------------|-----------------------
| 2017-01-17T09:50:41-05:00 | 2017-01-17T09:50:41-05:00 | Initial
| 2017-01-17T09:50:41-05:00 | 2017-01-17T09:50:41-05:00 | Example_Condition
| 2017-01-17T09:50:41-05:00 | 2017-01-17T09:50:41-05:00 | Encounter
| 2017-01-17T09:50:41-05:00 | 2017-01-17T09:50:41-05:00 | CallSubmodule
| 2017-01-17T09:50:41-05:00 | 2017-01-17T09:50:41-05:00 | Initial
| 2017-01-17T09:50:41-05:00 | 2017-01-17T09:50:41-05:00 | Prescribe_Medication
| 2017-01-17T09:50:41-05:00 | 2017-01-17T09:50:41-05:00 | Med_Terminal
| 2017-01-17T09:50:41-05:00 |                           | Terminal
\===============================================================================
```

As you can see, the submodule's states are processed _inline_ with the main module's states when called. To those familiar with programming, calling a submodule may seem analogous to executing a function.