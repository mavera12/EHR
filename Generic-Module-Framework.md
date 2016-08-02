Synthea contains a framework for defining modules using JSON.  A JSON module configuration describes a progression of states and the transitions between them.  On each Synthea generation time-step, the generic framework processes states one at a time to trigger conditions, encounter, medications, and other clinical events.

# States

The generic module framework currently supports the following states: [Initial](#initial), [Terminal](#terminal), [Guard](#guard), [Delay](#delay), [Encounter](#encounter), [ConditionOnset](#conditiononset), [MedicationOrder](#medicationorder), [Procedure](#procedure), and [Death](#death).

The following states are also planned for future implementation: Lab, ConditionEnd, and MedicationEnd.

## Initial

The `Initial` state type is the first state that is processed in a generic module.  It does not provide any specific function except to indicate the starting point, so it has no properties except its `type`.  The Initial state is the only state that requires a specific name: "Initial".  In addition, it is the only state for which there can only be _one_ in the whole module.

**Supported Properties**

* **type**: must be "Initial" _(required)_

**Example**

_Please note that, for simplicity, state examples in this document will not include any transition properties.  See the Transitions section for information about transitions._

```json
{
  "type": "Initial"
}
```

## Terminal

The `Terminal` state type indicates the end of the module progression.  Once a Terminal state is reached, no further progress will be made.  As such, Terminal states cannot have any transition properties.  If desired, there may be multiple Terminal states with different names to indicate different ending points -- but this has no actual effect on the records that are produced.

**Supported Properties**

* **type**: must be "Terminal" _(required)_

**Example**

```json
{
  "type": "Terminal"
}
```

## Guard

The `Guard` state type indicates a point in the module through which a patient can only pass if they meet certain logical conditions.  For example, a Guard may block a workflow until the patient reaches a certain age, after which the Guard allows the module to continue to progress.  Depending on the condition, a patient may be blocked by a Guard until they die -- in which case they never reach a `Terminal` state.  The Guard state's `allow` property provides the logical condition which must be met to allow the module to continue to the next state.  Please see the [Logic](#logic) section for more information about creating logical conditions.

Guard states are similar to [conditional transitions](#conditional) in some ways, but also have an important difference.  A conditional transition tests conditions once and uses the result to immediately choose the next state.  A Guard state will test the same condition on every time-step until the condition passes, at which point it progresses to the next state.

**Supported Properties**

* **type**: must be "Guard" _(required)_
* **allow**: the condition under which the Guard allows the module to progress to the next state; otherwise the module remains at the Guard state _(required)_

**Example**

The following is an example of a Guard state that only allows the module to continue if the patient is male and at least 40 years old.

```json
{
  "type": "Guard",
  "allow": {
    "condition_type": "And",
    "conditions": [
      {
        "condition_type": "Gender",
        "gender": "M" 
      },
      {
        "condition_type": "Age",
        "operator": ">=",
        "quantity": 40,
        "unit": "years"
      }
    ]
  }
}
```

## Delay

The `Delay` state type introduces a pre-configured temporal delay in the module's timeline.  As a simple example, a Delay state may indicate a one-month gap in time between an initial encounter and a followup encounter.  The module will not pass through the Delay state until the proper amount of time has passed.  The Delay state may define an exact time to delay (e.g., 4 days) or a range of time to delay (e.g., 5 - 7 days).

**Implementation Detail**

Synthea generation occurs in time-steps; currently defaults in the configuration to 7-days.  This means that if a module is processed on a given date, the next time it is processed will be exactly 7 days later.  If a delay expiration falls between time-steps (e.g., day 3 of a 7-day time-step), then the first time-step _after_ the delay expiration will effectively _rewind_ the clock to the delay expiration time and process states using that time.  Once it reaches a state that it can't pass through, it will process it once more using the original (7-day time-step) time.

**Supported Properties**

* **type**: must be "Delay" _(required)_
* **exact**: an exact amount of time to delay _(required if `range` is not set)_
  * **quantity**: the number of _units_ to delay (e.g., 4) _(required)_
  * **unit**: the unit of time pertaining to the _quantity_ (e.g., "days").  Valid _unit_ values are: `years`, `months`, `weeks`, `days`, `hours`, `minutes`, and `seconds`. _(required)_
* **range**: a range indicating the allowable amounts of delay.  The actual delay time will be chosen randomly from the range. _(required if `exact` is not set)_
  * **low**: the lowest number (inclusive) of _units_ to delay (e.g., 5) _(required)_
  * **high**: the highest number (inclusive) of _units_ to delay (e.g., 7) _(required)_
  * **unit**: the unit of time pertaining to the _range_ (e.g., "days").  Valid _unit_ values are: `years`, `months`, `weeks`, `days`, `hours`, `minutes`, and `seconds`. _(required)_

**Examples**

The following is an example of a Delay state that delays exactly 4 days.

```json
{
  "type": "Delay",
  "exact": {
    "quantity": 4,
    "unit": "days"
  }
}
```

The following is an example of a Delay state that delays at least 5 days and at most 7 days.

```json
{
  "type": "Delay",
  "range": {
    "low": 5,
    "high": 7,
    "unit": "days"
  }
}
```

## Encounter

The `Encounter` state type indicates a point in the module where an encounter should take place.  Encounters are important in Synthea because they are generally the mechanism through which the actual patient record is updated (a disease is diagnosed, a medication is prescribed, etc).  The generic module framework supports integration with scheduled wellness encounters from Synthea's [Encounters](https://github.com/synthetichealth/synthea/blob/master/lib/modules/encounters.rb) module, as well as creation of new stand-alone encounters.

**Scheduled Wellness Encounters vs. Standalone Encounters**

An Encounter state with the `wellness` property set to `true` will block until the next scheduled wellness encounter occurs.  Scheduled wellness encounters are managed by the _Encounters_ module in Synthea and, depending on the patient's age, typically occur every 1 - 3 years.  When a scheduled wellness encounter finally does occur, Synthea will search the generic modules for currently blocked Encounter states and will immediately process them (and their subsequent states).  An example where this might be used is for a condition that onsets between encounters, but isn't found and diagnosed until the next regularly scheduled wellness encounter.

An Encounter state without the `wellness` property set will be processed and recorded in the patient record immediately.  Since this creates an encounter, the encounter class and at least one code must be specified in the state configuration.  This is how generic modules can introduce encounters that are not already scheduled by other modules.

**Encounters and Related Events**

Encounters are typically the mechanism through which a patient's record will be updated.  This makes sense since most recorded events (diagnoses, prescriptions, and procedures) should happen in the context of an encounter.  When an Encounter state is successfully processed, it will look through the previously processed states for un-recorded ConditionOnset instances that indicate it (by name) as the `target_encounter`.  If it finds any, it will record the corresponding diagnosis in the patient's record at the time of the encounter. This is the mechanism for onsetting a disease _before_ it is discovered and diagnosed.

As soon as the Encounter state is processed, Synthea will continue to progress through the module.  If any subsequent states identify the previous Encounter as their `target_encounter` _and_ they occur at the _same time_ as the target encounter, then they will be added to the patient record.  This is the preferred mechanism for simulating events that happen _at_ the encounter (e.g., MedicationOrders, Procedures, and Encounter-caused ConditionOnsets).

**Future Implementation Considerations**

Future implementations should also consider a more robust mechanism for defining the length of an encounter and the activities that happen during it.  Currently, encounter activities must start at the same exact time as the encounter start in order to be recorded.  This, however, is unrealistic for multi-day inpatient encounters.

**Supported Properties**

* **type**: must be "Encounter" _(required)_
* **wellness**: if `true`, indicates that this state should block until a regularly scheduled wellness encounter occurs _(required if `class` and `codes` are not set)_
* **class**: indicates the class of the encounter, as defined in the [EncounterClass](http://hl7.org/fhir/DSTU2/valueset-encounter-class.html) value set _(required if `wellness` is not set)_
* **codes[]**: a list of codes indicating the encounter type _(at least one required if `wellness` is not set)_
  * **system**: the code system.  Currently, only `SNOMED-CT` is allowed. _(required)_
  * **code**: the code _(required)_
  * **display**: the human-readable code description _(required)_

**Examples**

The following is an example of an Encounter state that blocks until a regularly scheduled encounter.

```json
{
  "type": "Encounter",
  "wellness": true
}
```

The following is an example of an Encounter state indicating an ED visit.

```json
{
  "type": "Encounter",
  "class": "emergency",
  "codes": [{
    "system": "SNOMED-CT",
    "code": "50849002",
    "display": "Emergency Room Admission"
  }]
}
```

## ConditionOnset

The `ConditionOnset` state type indicates a point in the module where the patient acquires a condition.  This is _not_ necessarily the same as when the condition is diagnosed and recorded in the patient's record.  In fact, it is possible for a condition to onset but never be discovered.

If the ConditionOnset state's `target_encounter` is set to the name of a future encounter, then the condition will be diagnosed when that future encounter occurs.  If the `target_encounter` is set to the name of a previous encounter, then the condition will only be diagnosed if the ConditionOnset start time is the same as the encounter's start time.  See the Encounter section above for more details.

**Future Implementation Considerations**

Although the generic module framework supports a distinction between a condition's onset date and diagnosis date, currently only the diagnosis date is recorded.  In the future, the `Synthea::Output::Record::condition` method should be updated to support an onset date.

Currently, the generic module framework does not provide a way to resolve (or abate) conditions.  There are two ways this could potentially be implemented in the future: (1) by introducing a `ConditionEnd` state (recommended), or (2) by introducing a property in `ConditionOnset` to indicate its intended duration.

**Supported Properties**

* **type**: must be "ConditionOnset" _(required)_
* **target_encounter**: the name of the Encounter state at which this condition should be diagnosed and recorded _(optional)_
* **codes[]**: a list of codes indicating the condition _(at least one required)_
  * **system**: the code system.  Currently, only `SNOMED-CT` is allowed. _(required)_
  * **code**: the code _(required)_
  * **display**: the human-readable code description _(required)_

**Example**

The following is an example of a ConditionOnset that should be diagnosed at the "ED_Visit" Encounter.

```json
{
  "type": "ConditionOnset",
  "target_encounter": "ED_Visit",
  "codes": [{
    "system": "SNOMED-CT",
    "code": "47693006",
    "display": "Rupture of appendix"
  }]
}
```

## MedicationOrder

The `MedicationOrder` state type indicates a point in the module where a medication should be prescribed.  The MedicationOrder state must come after the `target_encounter` Encounter state in the module, but must have the same start time as that Encounter; otherwise it will not be recorded in the patient's record.  See the Encounter section above for more details.

The `MedicationOrder` also supports identifying a previous `ConditionOnset` as the `reason` for the prescription.

**Future Implementation Considerations**

Currently, the generic module framework does not provide a way to end medications.  There are two ways this could potentially be implemented in the future: (1) by introducing a `MedicationEnd` state, or (2) by introducing a property in `MedicationOrder` to indicate its intended duration.

**Supported Properties**

* **type**: must be "MedicationOrder" _(required)_
* **target_encounter**: the name of the Encounter state at which this medication should be prescribed.  This Encounter must come before the MedicationOrder in the module, but must have the same start time as the MedicationOrder. _(required)_
* **codes[]**: a list of codes indicating the medication _(at least one required)_
  * **system**: the code system.  Currently, only `RxNorm` is allowed. _(required)_
  * **code**: the code _(required)_
  * **display**: the human-readable code description _(required)_
* **reason**: the name of the ConditionOnset state which represents the reason for which the medication is prescribed.  This ConditionOnset must come _before_ the MedicationOrder in the module. _(optional)_

**Example**

The following is an example of a MedicationOrder that should be prescribed at the "Annual_Checkup" Encounter and cite the "Diabetes" ConditionOnset as the reason.

```json
{
  "type": "MedicationOrder",
  "target_encounter": "Annual_Checkup",
  "codes": [{
    "system": "RxNorm",
    "code": "860975",
    "display": "24 HR Metformin hydrochloride 500 MG Extended Release Oral Tablet"
  }],
  "reason": "Diabetes"
}
```

## Procedure

The `Procedure` state type indicates a point in the module where a procedure should be performed.  The Procedure state must come after the `target_encounter` Encounter state in the module, but must have the same start time as that Encounter; otherwise it will not be recorded in the patient's record.  See the Encounter section above for more details.

The `Procedure` also supports identifying a previous `ConditionOnset` as the `reason` for the procedure.

**Future Implementation Considerations**

Currently, the generic module framework does not provide a way to indicate the duration of a procedure.  This would probably be best implemented by simply introducing a property in `Procedure` to indicate its intended duration.

**Supported Properties**

* **type**: must be "Procedure" _(required)_
* **target_encounter**: the name of the Encounter state at which this procedure should be performed.  This Encounter must come before the Procedure in the module, but must have the same start time as the Procedure. _(required)_
* **codes[]**: a list of codes indicating the procedure _(at least one required)_
  * **system**: the code system.  Currently, only `SNOMED-CT` is allowed. _(required)_
  * **code**: the code _(required)_
  * **display**: the human-readable code description _(required)_
* **reason**: the name of the ConditionOnset state which represents the reason for procedure.  This ConditionOnset must come _before_ the Procedure in the module. _(optional)_

**Example**

The following is an example of a Procedure that should be performed at the "Inpatient_Encounter" Encounter and cite the "Appendicitis" ConditionOnset as the reason.

```json
{
  "type": "Procedure",
  "target_encounter": "Inpatient_Encounter",
  "codes": [{
    "system": "SNOMED-CT",
    "code": "6025007",
    "display": "Laparoscopic appendectomy"
  }],
  "reason": "Appendicitis"
}
```

## Death

The `Death` state type indicates a point in the module at which the patient dies.  When the Death state is processed, the patient's death is immediately recorded and the patient's `:is_alive` attribute is set to `false`.  The module will continue to progress to the next state(s) for the current time-step, but will not progress any further in future time-steps.  Typically, the Death state should transition to a `Terminal` state.

**Implementation Warning**

If a `Death` state is processed after a `Delay`, it may cause inconsistencies in the record.  This is because the `Delay` implementation must _rewind_ time to correctly honor the requested delay duration.  If it rewinds time, and then the patient dies at the rewinded time, then any modules that were processed before the generic module may have created events and records with a timestamp _after_ the patient's death.

**Supported Properties**

* **type**: must be "Death" _(required)_

**Example**

```json
{
  "type": "Death"
}
```

# Transitions

The generic module framework currently supports the following transitions: [Direct](#direct), [Distributed](#distributed), and [Conditional](#conditional).

## Direct

`Direct` transitions are the simplest of transitions.  They transition directly to the indicated state.  The value of a `direct_transition` is simply the name of the state to transition to.

**Example**

The following example demonstrates a state that should transition directly to the "Foo" state.

_Please note that, for simplicity, transition examples in this document will always start at an `Initial` state.  In real modules, transitions can be applied to any state (except a `Terminal` state) and can transition to any state._

```json
{
  "type": "Initial",
  "direct_transition": "Foo"
}
```

## Distributed

`Distributed` transitions will transition to one of several possible states based on the configured distribution.  Distribution values are from 0.0 to 1.0, such that a value of 0.55 would indicate a 55% chance of transitioning to the corresponding state.  A `distributed_transition` consists of an array of `distribution`/`transition` pairs for which the `distribution` values should sum up to 1.0.

If the `distribution` values do not sum up to 1.0, the remaining distribution will transition to the last defined `transition` state. For example, given distributions of 0.3 and 0.6, the _effective_ distribution of the last transition will actually be 0.7.

If the `distribution` values sum up to more than 1.0, the remaining distributions are ignored (for example, if distribution values are 0.75, 0.5, and 0.3, then the second transition will have an _effective_ distribution of 0.25, and the last transition will have an effective distribution of 0.0).

**Example**

The following example demonstrates a state that should transition to the "Foo" state 15% of the time, the "Bar" state 55% of the time, and the "Baz" state 30% of the time.

```json
{
  "type": "Initial",
  "distributed_transition": [
    {
      "distribution": 0.15,
      "transition": "Foo"
    },
    {
      "distribution": 0.55,
      "transition": "Bar"
    },
    {
      "distribution": 0.30,
      "transition": "Baz"
    }
  ]
}
```

## Conditional

`Conditional` transitions will transition to one of several possible states based on conditional logic.  A `conditional_transition` consists of an array of `condition`/`transition` pairs which are tested in the order they are defined.  The first condition that evaluates to `true` will result in a transition to its corresponding `transition` state.  The last element in the `condition_transition` array may contain only a `transition` (with no `condition`) to indicate a fallback transition when all other conditions are `false`.

If none of the `conditions` evaluated to `true`, and no fallback transition was specified, the module will transition to a default `Terminal` state.

Please see the [Logic](#logic) section for more information about creating logical conditions.

**Example**

The following example demonstrates a state that should transition to the "Foo" state for male patients, the "Bar" state for female patients, and the "Baz" state for patients with an unrecognized gender value.

```json
{
  "type": "Initial",
  "conditional_transition": [
    {
      "condition": {
        "condition_type": "Gender",
        "gender": "M" 
      },
      "transition": "Foo"
    },
    {
      "condition": {
        "condition_type": "Gender",
        "gender": "F" 
      },
      "transition": "Bar"
    },
    {
      "transition": "Baz"
    }
  ]
}
```

# Logic

The Guard state and Conditional transition use conditional (boolean) logic.  The following condition types are currently supported: [Gender](#gender), [Age](#age), [And](#and), [Or](#or), [Not](#not), [True](#true), and [False](#false).

The following condition types should be considered for future versions:

* Attribute: check if a patient attribute is present or compare it against a value
* PriorEvent: check if a patient event occurred
* PriorState: check if a specific state is in the module's state history

## Gender

The `Gender` condition type tests the patient's gender.

**Supported Properties**

* **condition_type**: must be "Gender" _(required)_
* **gender**: the gender to test for.  Synthea currently defines the following gender values: `M`: male, and `F`: female _(required)_

**Example**

The following Gender condition will return `true` if the patient is male; `false` otherwise.

```json
{
  "condition_type": "Gender",
  "gender": "M" 
}
```

## Age

The `Age` condition type tests the patient's age.  The following units are supported: `years`, `months`, `weeks`, `days`, `hours`, `minutes`, and `seconds`.

**Supported Properties**

* **condition_type**: must be "Age" _(required)_
* **operator**: indicates how to compare the actual age against the _quantity_.  Valid _operator_ values are: `<`, `<=`, `==`, `>=`, `>`, and `!=`. _(required)_
* **quantity**: the number, corresponding to the _unit_, to indicate the age to test against _(required)_
* **unit**: the unit corresponding to the _quantity_.  Valid _unit_ values are: `years`, `months`, `weeks`, `days`, `hours`, `minutes`, and `seconds`. _(required)_

**Example**

The following Age condition will return `true` if the patient is 40 years old or more.

```json
{
  "condition_type": "Age",
  "operator": ">=",
  "quantity": 40,
  "unit": "years"
}
```

## And

The `And` condition type tests that a set of sub-conditions are _all_ true.  If _all_ sub-conditions are true, it will return `true`, but if _any_ are false, it will return `false`.

**Supported Properties**

* **condition_type**: must be "And" _(required)_
* **conditions[]**: an array of sub-conditions to test _(required)_

**Example**

The following And condition will return `true` if the patient is male and at least 40 years old; `false` otherwise.

```json
{
  "condition_type": "And",
  "conditions": [
    {
      "condition_type": "Gender",
      "gender": "M" 
    },
    {
      "condition_type": "Age",
      "operator": ">=",
      "quantity": 40,
      "unit": "years"
    }
  ]
}
```

## Or

The `Or` condition type tests that _at least one_ of its sub-conditions is true.  If _any_ sub-condition is true, it will return `true`, but if _all_ sub-conditions are false, it will return `false`.

**Supported Properties**

* **condition_type**: must be "Or" _(required)_
* **conditions[]**: an array of sub-conditions to test _(required)_

**Example**

The following Or condition will return `true` if the patient is male _or_ the patient is at least 40 years old; `false` otherwise.

```json
{
  "condition_type": "Or",
  "conditions": [
    {
      "condition_type": "Gender",
      "gender": "M" 
    },
    {
      "condition_type": "Age",
      "operator": ">=",
      "quantity": 40,
      "unit": "years"
    }
  ]
}
```

## Not

The `Not` condition type negates its sub-condition.  If the sub-condition is true, it will return `false`; if the sub-condition is false, it will return `true`.

**Supported Properties**

* **condition_type**: must be "Not" _(required)_
* **condition**: the sub-condition to test _(required)_

**Example**

The following Not condition will return `true` if the patient is _not_ male; `false` otherwise.

```json
{
  "condition_type": "Not",
  "condition": {
    "condition_type": "Gender",
    "gender": "M" 
  }
}
```

## True

The `True` condition always returns `true`.  This condition is mainly used for testing purposes and is not expected to be used in any real module.

**Supported Properties**

* **condition_type**: must be "True" _(required)_

**Example**

The following True condition always returns `true`.

```json
{
  "condition_type": "True"
}
```

## False

The `False` condition always returns `false`.  This condition is mainly used for testing purposes and is not expected to be used in any real module.

**Supported Properties**

* **condition_type**: must be "False" _(required)_

**Example**

The following False condition always returns `false`.

```json
{
  "condition_type": "False"
}
```

# Complete Module Example: Examplitis

The [synthea_graphviz](https://github.com/synthetichealth/synthea_graphviz) tool will generate diagrams for all generic modules found in _lib/generic/modules/_.  The following is an example diagram representing a complete module for a made up disease: Examplitis.

![Examplitis Diagram](https://github.com/synthetichealth/synthea/raw/generic_modules/test/fixtures/generic/example_module_diagram.png)

The above diagram was generated based on the following JSON module file:

```json
{
    "name": "Examplitis",
    "states": {
        "Initial": {
            "type": "Initial",
            "conditional_transition": [
                {
                    "condition": {
                        "condition_type": "Gender",
                        "gender": "M" 
                    },
                    "transition": "Age_Guard"
                },
                {
                    "transition": "Terminal"
                }
            ]
        },
        "Age_Guard": {
            "type": "Guard",
            "allow": {
              "condition_type": "Age",
              "operator": ">",
              "quantity": 40,
              "unit": "years"
            },
            "distributed_transition": [
                {
                    "distribution": 0.10,
                    "transition": "Pre_Examplitis"
                },
                {
                    "distribution": 0.90,
                    "transition": "Terminal"
                }
            ]
        },
        "Pre_Examplitis": {
            "type": "Delay",
            "range": {
                "low": 0,
                "high": 10,
                "unit": "years"
            },
            "direct_transition": "Examplitis" 
        },
        "Examplitis": {
            "type": "ConditionOnset",
            "target_encounter": "Wellness_Encounter",
            "codes": [{
                "system": "SNOMED-CT",
                "code": "123",
                "display": "Examplitis"
            }],
            "direct_transition": "Wellness_Encounter"
        },
        "Wellness_Encounter": {
            "type": "Encounter",
            "wellness": true,
            "direct_transition": "Examplitol"
        },
        "Examplitol": {
            "type": "MedicationOrder",
            "target_encounter": "Wellness_Encounter",
            "codes": [{
                "system": "RxNorm",
                "code": "456",
                "display": "Examplitol"
            }],
            "reason": "Examplitis",
            "distributed_transition": [
                {
                    "distribution": 0.2,
                    "transition": "Pre_Examplotomy"
                },
                {
                    "distribution": 0.1,
                    "transition": "Last_Days"
                },
                {
                    "distribution": 0.7,
                    "transition": "Terminal"
                }
            ]
        },
        "Pre_Examplotomy": {
            "type": "Delay",
            "range": {
                "low": 18,
                "high": 36,
                "unit": "months"
            },
            "direct_transition": "Examplotomy_Encounter"
        },
        "Examplotomy_Encounter": {
            "type": "Encounter",
            "class": "daytime",
            "codes": [{
                "system": "SNOMED-CT",
                "code": "ABC",
                "display": "Examplotomy Encounter"
            }],
            "direct_transition": "Examplotomy"
        },
        "Examplotomy": {
            "type": "Procedure",
            "target_encounter": "Examplotomy_Encounter",
            "codes": [{
                "system": "SNOMED-CT",
                "code": "789",
                "display": "Examplotomy"
            }],
            "reason": "Examplitis",
            "distributed_transition": [
                {
                    "distribution": 0.1,
                    "transition": "Death"
                },
                {
                    "distribution": 0.9,
                    "transition": "Terminal"
                }
            ]
        },
        "Last_Days": {
            "type": "Delay",
            "range": {
                "low": 8,
                "high": 20,
                "unit": "years"
            },
            "direct_transition": "Death" 
        },
        "Death": {
            "type": "Death"
        },
        "Terminal": {
            "type": "Terminal"
        }
    }
}
```

# Logging

The generic module framework supports simple logging to show how patients progress through modules.  When enabled, a table will be printed to standard out when each patient reaches a `Terminal` state in the workflow.  _(Note that if a patient never reaches a `Terminal` state, then that patients table will not be generated)._

Logging is disabled by default.  To enable logging, turn it on in the _config/synthea.yml_ file:
```yaml
  generic:
    log: true
```

**Examples From the Examplitis Module**

The following example shows the log of a patient who was female, so she went from "Initial" directly to "Terminal".

```
/===============================================================================
| Examplitis Log
|===============================================================================
| Entered                   | Exited                    | State
|---------------------------|---------------------------|-----------------------
| 1980-02-02T19:59:42-05:00 | 1980-02-02T19:59:42-05:00 | Initial
| 1980-02-02T19:59:42-05:00 |                           | Terminal
\===============================================================================
```

As opposed the the previous example, this example shows the log of a patient who acquired Examplitis and ultimately needed surgery.  Since he went from "Examplotomy" to "Terminal", however, we know he did not _die_ of Examplitis.

```
/===============================================================================
| Examplitis Log
|===============================================================================
| Entered                   | Exited                    | State
|---------------------------|---------------------------|-----------------------
| 1957-06-28T18:44:14-04:00 | 1957-06-28T18:44:14-04:00 | Initial
| 1957-06-28T18:44:14-04:00 | 1998-06-26T18:44:14-04:00 | Age_Guard
| 1998-06-26T18:44:14-04:00 | 2005-06-26T18:44:14-04:00 | Pre_Examplitis
| 2005-06-26T18:44:14-04:00 | 2005-06-26T18:44:14-04:00 | Examplitis
| 2005-06-26T18:44:14-04:00 | 2005-08-30T00:54:57-04:00 | Wellness_Encounter
| 2005-08-30T00:54:57-04:00 | 2005-08-30T00:54:57-04:00 | Examplitol
| 2005-08-30T00:54:57-04:00 | 2008-07-30T00:54:57-04:00 | Pre_Examplotomy
| 2008-07-30T00:54:57-04:00 | 2008-07-30T00:54:57-04:00 | Examplotomy_Encounter
| 2008-07-30T00:54:57-04:00 | 2008-07-30T00:54:57-04:00 | Examplotomy
| 2008-07-30T00:54:57-04:00 |                           | Terminal
\===============================================================================
```

# Relevant Files and Paths

Within the _synthea_ repository, the following files and paths are relevant to the generic module framework:

* **lib/generic/modules/**: the path containing the JSON module files that Synthea should process
* **lib/generic/context.rb**: the `Context` class definition; responsible for maintaining a modules state definition, state history, and current state for each patient.  The `Context` class also processes states and transitions.
* **lib/generic/logic.rb**: the `Logic` module implements the methods corresponding to the support logical condition types
* **lib/generic/states.rb**: the `States` module implements the support state types
* **modules/generic.rb**: the `Generic` module defines the Synthea `rule` that is executed on every time-step.  It loops through the modules in _lib/generic/modules/_, setting up and running the context for each module.
* **test/fixtures/generic/*.json**: the test fixtures for all of the generic module framework unit tests
* **test/generic_*_test.rb**: the unit tests for the generic module framework's context, logic, and state implementations