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

## Initial

The `Initial` state type is the first state that is processed in a generic module.  It does not provide any specific function except to indicate the starting point, so it has no properties except its `type`.  The Initial state is the only state that requires a specific name: "Initial".  In addition, it is the only state for which there can only be _one_ in the whole module.

**Supported Properties**

* **type**: must be "Initial"

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

* **type**: must be "Terminal"

**Example**

```json
{
  "type": "Terminal"
}
```

## Guard

The `Guard` state type indicates a point in the module through which a patient can only pass if they meet certain logical conditions.  For example, a Guard may block a workflow until the patient reaches a certain age, after which the Guard allows the module to continue to progress.  Depending on the condition, a patient may be blocked by a Guard until they die -- in which case they never reach a `Terminal` state.  The Guard state's `allow` property provides the logical condition which must be met to allow the module to continue to the next state.  Please see the _Logic_ section for more information about creating logical conditions.

Guard states are similar to conditional transitions in some ways, but also have an important difference.  A conditional transition tests conditions once and uses the result to immediately choose the next state.  A Guard state will test the same condition on every time cycle until the condition passes, at which point it progresses to the next state.

**Supported Properties**

* **type**: must be "Guard"
* **allow**: the condition under which the Guard allows the module to progress to the next state; otherwise the module remains at the Guard state.

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

Synthea generation occurs in time cycles; currently 7-day cycles.  This means that if a module is processed on a given date, the next time it is processed will be exactly 7 days later.  If a delay expiration falls between cycles (e.g., day 3 of a 7-day cycle), then the first cycle _after_ the delay expiration will effectively _rewind_ the clock to the delay expiration time and process states using that time.  Once it reaches a state that it can't pass through, it will process it once more using the original (7-day cycle) time.

**Supported Properties**

* **type**: must be "Delay"
* **exact**: an exact amount of time to delay
  * **quantity**: the number of _units_ to delay (e.g., 4)
  * **unit**: the unit of time pertaining to the _quantity_ (e.g., "days").  Valid _unit_ values are: `years`, `months`, `weeks`, `days`, `hours`, `minutes`, and `seconds`.
* **range**: a range indicating the allowable amounts of delay.  The actual delay time will be chosen randomly from the range.
  * **low**: the lowest number (inclusive) of _units_ to delay (e.g., 5)
  * **high**: the highest number (inclusive) of _units_ to delay (e.g., 7)
  * **unit**: the unit of time pertaining to the _range_ (e.g., "days").  Valid _unit_ values are: `years`, `months`, `weeks`, `days`, `hours`, `minutes`, and `seconds`.

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

The `Encounter` state type indicates a point in the module where an encounter should take place.  Encounters are important in Synthea because they are generally the mechanism through which the actual patient record is updated (a disease is diagnosed, a medication is prescribed, etc).  The generic module framework supports integration with scheduled wellness encounters from Synthea's _Encounters_ module, as well as creation of new stand-alone encounters.

**Scheduled Wellness Encounters vs. Standalone Encounters**

An Encounter state with the `wellness` property set to `true` will block until the next scheduled wellness encounter occurs.  Scheduled wellness encounters are managed by the _Encounters_ module in Synthea and, depending on the patient's age, typically occur every 1 - 3 years.  When a scheduled wellness encounter finally does occur, Synthea will search the generic modules for currently blocked Encounter states and will immediately process them (and their subsequent states).  An example where this might be used is for a condition that onsets between encounters, but isn't found and diagnosed until the next regularly scheduled wellness encounter.

An Encounter state without the `wellness` property set will be processed and recorded in the patient record immediately.  Since this creates an encounter, the encounter class and at least one code must be specified in the state configuration.  This is how generic modules can introduce encounters that are not already scheduled by other modules.

**Encounters and Related Events**

Encounters are typically the mechanism through which a patient's record will be updated.  This makes sense since most recorded events (diagnoses, prescriptions, and procedures) should happen in the context of an encounter.  When an Encounter state is successfully processed, it will look through the previously processed states for un-recorded ConditionOnset, MedicationOrder, and Procedure instances that indicate it (by name) as the `target_encounter`.  It will then add the appropriate diagnosis, prescription, and/or procedure to the patient's record.  This is the mechanism through which _previously_ processed states can be added to the patient's record.

As soon as the Encounter state is processed, Synthea will continue to progress through the module.  If any subsequent states identify the previous Encounter as their `target_encounter` _and_ they occur at the _same time_ as the target encounter, then they will be added to the patient record.  This is the preferred mechanism for simulating events that happen _at_ the encounter (e.g., a procedure).

**Future Implementation Considerations**

It may not be appropriate for previously occurring MedicationOrders or Procedures to be recorded in an encounter.  This is currently implemented for consistency, but should be reconsidered.

Future implementations should also consider a more robust mechanism for defining the length of an encounter and the activities that happen during it.  Currently, encounter activities must start at the same exact time as the encounter start in order to be recorded.  This, however, is unrealistic for multi-day inpatient encounters.

**Supported Properties**

* **type**: must be "Encounter"
* **wellness**: if `true`, indicates that this state should block until a regularly scheduled wellness encounter occurs
* **class**: indicates the class of the encounter, as defined in the [EncounterClass](http://hl7.org/fhir/DSTU2/valueset-encounter-class.html) value set
* **codes[]**: a list of codes indicating the encounter type
  * **system**: the code system.  Currently, only `SNOMED-CT` is allowed.
  * **code**: the code
  * **display**: the human-readable code description

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

## MedicationOrder

## Procedure

## Death

# Transitions

The generic module framework currently supports the following transitions:

* Direct
* Distributed
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
