The generic module framework currently supports the following states: [Initial](#initial), [Terminal](#terminal), [Guard](#guard), [Simple](#simple), [Delay](#delay), [Encounter](#encounter), [ConditionOnset](#conditiononset), [MedicationOrder](#medicationorder), [MedicationEnd](#medicationend), [Procedure](#procedure), [Symptom](#symptom), [SetAttribute](#setattribute), and [Death](#death).

The following states are also planned for future implementation: Lab, ConditionEnd, Symptom.

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

## Simple

The `Simple` state type indicates a state that performs no additional actions, adds no additional information to the patient entity, and just transitions to the next state.  As an example, this state may be used to chain conditional or distributed transitions, in order to represent complex logic.

**Supported Properties**

* **type**: must be "Simple" _(required)_

**Example**

```json
{
  "type": "Simple"
}
```

## Guard

The `Guard` state type indicates a point in the module through which a patient can only pass if they meet certain logical conditions.  For example, a Guard may block a workflow until the patient reaches a certain age, after which the Guard allows the module to continue to progress.  Depending on the condition, a patient may be blocked by a Guard until they die -- in which case they never reach a `Terminal` state.  The Guard state's `allow` property provides the logical condition which must be met to allow the module to continue to the next state.  Please see the [[Logic|Generic Module Framework: Logic]] section for more information about creating logical conditions.

Guard states are similar to [[conditional transitions|Generic Module Framework: Transitions#conditional]] in some ways, but also have an important difference.  A conditional transition tests conditions once and uses the result to immediately choose the next state.  A Guard state will test the same condition on every time-step until the condition passes, at which point it progresses to the next state.

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
* **assign_to_attribute**: The name of the attribute this condition should be referred to by. Attributes allow modules to store, reference, and share medications, conditions, procedures, etc, as well as as arbitrary strings.

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

**Supported Properties**

* **type**: must be "MedicationOrder" _(required)_
* **target_encounter**: the name of the Encounter state at which this medication should be prescribed.  This Encounter must come before the MedicationOrder in the module, but must have the same start time as the MedicationOrder. _(required)_
* **codes[]**: a list of codes indicating the medication _(at least one required)_
  * **system**: the code system.  Currently, only `RxNorm` is allowed. _(required)_
  * **code**: the code _(required)_
  * **display**: the human-readable code description _(required)_
* **reason**: the name of the ConditionOnset state which represents the reason for which the medication is prescribed.  This ConditionOnset must come _before_ the MedicationOrder in the module. _(optional)_
* **assign_to_attribute**: The name of the attribute this medication should be referred to by. Attributes allow modules to store, reference, and share medications, conditions, procedures, etc, as well as as arbitrary strings.

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


## MedicationEnd

The `MedicationEnd` state type indicates a point in the module where a currently prescribed medication should be ended. The `MedicationEnd` state supports three ways of specifying the medication to end:
1. By `codes[]`, specifying the code system, code, and display name of the medication to end, or
2. By `medication_order`, specifying the name of the `MedicationOrder` state in which the medication was prescribed, or
3. By `referenced_by_attribute`, specifying the name of the Attribute in which a previous `MedicationOrder` state assigned a medication.

**Supported Properties**

* **type**: must be "MedicationEnd" _(required)_
* **codes[]**: a list of codes indicating the medication _(optional, required if neither `medication_order` nor `referenced_by_attribute` is set)_
  * **system**: the code system.  Currently, only `RxNorm` is allowed. _(required)_
  * **code**: the code _(required)_
  * **display**: the human-readable code description _(required)_
* **medication_order**: the name of the `MedicationOrder` state in which the medication was prescribed _(optional, required if neither `codes[]` nor `referenced_by_attribute` is set)_
* **referenced_by_attribute**: the name of the Attribute in which a previous `MedicationOrder` state assigned a medication _(optional, required if neither `medication_order` nor `codes[]` is set)_
* **reason**:  text reason for why the medication is ended. If not provided, defaults to "prescription expired" _(optional)_

**Example**

The following is an example of a MedicationEnd that ends a prescription for Metformin, specified by code:

```json
{
  "type": "MedicationEnd",
  "codes": [{
    "system": "RxNorm",
    "code": "860975",
    "display": "24 HR Metformin hydrochloride 500 MG Extended Release Oral Tablet"
  }]
}
```

The following is an example of a MedicationEnd that ends a prescription for a medication, specified by the name of an attribute where a previous state assigned a value:
```json
{
  "type": "MedicationEnd",
  "referenced_by_attribute" : "InsulinMed1"
},
```

The following is an example of a MedicationEnd that ends a prescription for a medication, specified by the name of the `MedicationOrder` state where the medication was prescribed:
```json
{
  "type": "MedicationEnd",
  "medication_order" : "Bromocriptine_Start"
},
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
* **assign_to_attribute**: The name of the attribute this procedure should be referred to by. Attributes allow modules to store, reference, and share medications, conditions, procedures, etc, as well as as arbitrary strings.

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

## Symptom

The `Symptom` state type adds or updates a patient's symptom. Synthea tracks symptoms in order to drive a patient's encounters, on a scale of 1-100. A symptom may be tracked for multiple conditions, in these cases only the highest value is considered. See also the [[Symptom|Generic Module Framework: Logic#symptom]] logical condition type.


**Supported Properties**

* **type**: must be "Symptom" _(required)_
* **symptom**: the name of the symptom being tracked _(required)_
* **cause**: the underlying cause of the symptom. Defaults to the name of the module if not set. _(optional)_
* **exact**: an exact value to score this symptom _(required if `range` is not set)_
  * **quantity**: the score to set (e.g., 4) _(required)_
* **range**: a range indicating the allowable symptom values.  The actual value will be chosen randomly from the range. _(required if `exact` is not set)_
  * **low**: the lowest number (inclusive) allowed (e.g., 5) _(required)_
  * **high**: the highest number (inclusive) allowed (e.g., 7) _(required)_

**Examples**

The following is an example of a Symptom state that sets the symptom value for Chest Pain to be exactly 27.

```json
{
  "type": "Symptom",
  "symptom" : "Chest Pain",
  "cause" : "Asthma",
  "exact": {
    "quantity": 27
  }
}
```

The following is an example of a Symptom state that sets the symptom value for Chest Pain to be between 90 and 100.
```json
{
  "type": "Symptom",
  "symptom" : "Chest Pain",
  "cause" : "Heart Attack",
  "range": {
    "low": 90,
    "high": 100
  }
}
```

## SetAttribute

The `SetAttribute` state type indicates a point in the module that sets a specified value on the patient entity. In addition to the `assign_to_attribute` property on `MedicationOrder`/`ConditionOnset`/etc states, this state allows for arbitrary text or values to be set on an Attribute, or for the Attribute to be reset.

**Supported Properties**

* **type**: must be "SetAttribute" _(required)_
* **attribute**: the name of the Attribute to set the _value_ for. _(required)_
* **value**: the text or other value to set for this _attribute_. If not provided, the _value_ is set to nil. _(optional)_

**Example**

The following is an example of a SetAttribute that sets the value of Attribute 'Opioid Prescription' to 'Vicodin':

```json
{
  "type": "SetAttribute",
  "attribute": "Opioid Prescription",
  "value": "Vicodin"
}
```

The following is an example of a SetAttribute that sets the value of Attribute 'Opioid Prescription' to nil, or no value:

```json
{
  "type": "SetAttribute",
  "attribute": "Opioid Prescription"
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
