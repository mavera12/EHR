The Generic Module Framework currently supports the following states:

* [Initial](#initial), [Terminal](#terminal)
* [Simple](#simple)
* [Guard](#guard), [Delay](#delay)
* [Encounter](#encounter)
* [ConditionOnset](#conditiononset), [ConditionEnd](#conditionend)
* [MedicationOrder](#medicationorder), [MedicationEnd](#medicationend)
* [CarePlanStart](#careplanstart), [CarePlanEnd](#careplanend)
* [Procedure](#procedure)
* [Observation](#observation)
* [Symptom](#symptom)
* [SetAttribute](#setattribute), [Counter](#counter)
* [Death](#death)


## Initial

The `Initial` state type is the first state that is processed in a generic module.  It does not provide any specific function except to indicate the starting point, so it has no properties except its `type`. The Initial state requires the specific name `"Initial"`.  In addition, it is the only state for which there can only be **one** in the whole module.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Initial"`. |


### Example

_Please note that, for simplicity, state examples in this document will not include any transition properties.  See the [[Transitions|Generic Module Framework: Transitions]] section for information about transitions._

```json
"Initial": {
  "type": "Initial"
}
```

## Terminal

The `Terminal` state type indicates the end of the module progression.  Once a Terminal state is reached, no further progress will be made. As such, Terminal states cannot have any transition properties. If desired, there may be multiple Terminal states with different names to indicate different ending points; however, this has no actual effect on the records that are produced.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Terminal"`. |

### Example

```json
"Terminal": {
  "type": "Terminal"
}
```

## Simple

The `Simple` state type indicates a state that performs no additional actions, adds no additional information to the patient entity, and just transitions to the next state.  As an example, this state may be used to chain conditional or distributed transitions, in order to represent complex logic.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Simple"`. |

### Example

```json
"Simple_State": {
  "type": "Simple"
}
```

## Guard

The `Guard` state type indicates a point in the module through which a patient can only pass if they meet certain logical conditions. For example, a Guard may block a workflow until the patient reaches a certain age, after which the Guard allows the module to continue to progress. Depending on the condition(s), a patient may be blocked by a Guard until they die - in which case they never reach the module's `Terminal` state.

  The Guard state's `allow` property provides the logical condition(s) which must be met to allow the module to continue to the next state. Please see the [[Logic|Generic Module Framework: Logic]] section for more information about creating logical conditions.

Guard states are similar to [[conditional transitions|Generic Module Framework: Transitions#conditional]] in some ways, but also have an important difference.  A conditional transition tests conditions once and uses the result to immediately choose the next state. A Guard state will test the same condition on every time-step until the condition passes, at which point it progresses to the next state.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Guard"`. |
| `allow` | `{}` | **(choice)** Any valid [logical condition](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Logic) to test. |


### Example

The following is an example of a Guard state that only allows the module to continue if the patient is male and at least 40 years old.

```json
"Age_Guard": {
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

The `Delay` state type introduces a pre-configured temporal delay in the module's timeline. As a simple example, a Delay state may indicate a one-month gap in time between an initial encounter and a followup encounter. The module will not pass through the Delay state until the proper amount of time has passed. The Delay state may define an `exact` time to delay (e.g. 4 days) or a `range` of time to delay (e.g. 5 - 7 days).

### Implementation Details

Synthea generation occurs in time steps; the default time step is 7 days.  This means that if a module is processed on a given date, the next time it is processed will be exactly 7 days later.  If a delay expiration falls between time steps (e.g. day 3 of a 7-day time step), then the first time step _after_ the delay expiration will effectively _rewind_ the clock to the delay expiration time and process states using that time.  Once it reaches a state that it can't pass through, it will process it once more using the original (7-day time step) time.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Delay"`. |
| `exact` or `range` | `{}` |**(choice)**  Must be either an `exact` delay or a `range` delay. |

##### `exact`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `quantity` | `numeric` | The number of `unit`. |
| `unit` | `string` | The unit of time, e.g. `"days"`. Must be a valid [unit of time](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#units).|

##### `range`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `low` | `numeric` | The lowest (inclusive) number of `unit`. |
| `high` | `numeric` | The greatest (inclusive) number of `unit`. |
| `unit` | `string` | The unit of time, e.g. `"days"`. Must be a valid [unit of time](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#units).|

The exact value for the delay will be chosen randomly from this range.

### Examples

The following is an example of a Delay state that delays exactly 4 days:

```json
"Delay_For_Followup_Visit": {
  "type": "Delay",
  "exact": {
    "quantity": 4,
    "unit": "days"
  }
}
```

The following is an example of a Delay state that delays at least 5 days and at most 7 days:

```json
"Delay_Unitl_Symptoms_Improve": {
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

### Scheduled Wellness Encounters vs. Standalone Encounters

An Encounter state with the `wellness` property set to `true` will block until the next scheduled wellness encounter occurs.  Scheduled wellness encounters are managed by the _Encounters_ module in Synthea and, depending on the patient's age, typically occur every 1 - 3 years.  When a scheduled wellness encounter finally does occur, Synthea will search the generic modules for currently blocked Encounter states and will immediately process them (and their subsequent states).  An example where this might be used is for a condition that onsets between encounters, but isn't found and diagnosed until the next regularly scheduled wellness encounter.

An Encounter state without the `wellness` property set will be processed and recorded in the patient record immediately.  Since this creates an encounter, the `encounter_class` and on or more `codes` must be specified in the state configuration.  This is how generic modules can introduce encounters that are not already scheduled by other modules.

### Encounters and Related Events

Encounters are typically the mechanism through which a patient's record will be updated. This makes sense since most recorded events (diagnoses, prescriptions, and procedures) should happen in the context of an encounter. When an Encounter state is successfully processed, Synthea will look through the previously processed states for un-recorded ConditionOnset instances that indicate that Encounter (by name) as the `target_encounter`. If Synthea finds any, they will be recorded in the patient's record at the time of the encounter. This is the mechanism for onsetting a disease _before_ it is discovered and diagnosed.

As soon as the Encounter state is processed, Synthea will continue to progress through the module. If any subsequent states identify the previous Encounter as their `target_encounter` _and_ they occur at the _same time_ as the target encounter, then they will be added to the patient record.  This is the preferred mechanism for simulating events that happen _at_ the encounter (e.g., MedicationOrders, Procedures, and ConditionOnsets).

### Future Implementation Considerations

Future implementations should also consider a more robust mechanism for defining the length of an encounter and the activities that happen during it.  Currently, encounter activities must start at the same exact time as the encounter start in order to be recorded.  This, however, is unrealistic for multi-day inpatient encounters.

### Encounter Classes

Common encounter classes are:

* `"emergency"`: A visit to an emergency department
* `"inpatient"`: A non-emergency visit to a hospital, e.g. for routine surgery
* `"ambulatory"`: A visit in a variety of outpatient settings, e.g. a visit to a PCP or a specialist

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Encounter"`. |
| `wellness` | `boolean` | If `true`, Synthea blocks 1-3 years until the next wellness encounter.<br/>**(optional)** if `false`. |
| `encounter_class` | `string` | One of [`"emergency"`, `"inpatient"`, `"ambulatory"`] |
| `reason` | `string` | **(optional)** Either an `"attribute"` or a `"State_Name"` referencing a<br/>_previous_ `ConditionOnset` state. |
| `codes` | `[]` | One or more codes that describe the Encounter. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#snomed-codes). |

### Examples

The following is an example of an `Encounter` state that blocks until a regularly scheduled encounter:

```json
"Next_Wellness_Encounter": {
  "type": "Encounter",
  "wellness": true
}
```

The following is an example of an `Encounter` state indicating an ED visit:

```json
"ED_Visit_For_Fracture": {
  "type": "Encounter",
  "encounter_class": "emergency",
  "reason": "broken_leg",
  "codes": [
    {
      "system": "SNOMED-CT",
      "code": "50849002",
      "display": "Emergency Room Admission"
    }
  ]
}
```

## ConditionOnset

The `ConditionOnset` state type indicates a point in the module where the patient acquires a condition.  This is _not_ necessarily the same as when the condition is diagnosed and recorded in the patient's record.  In fact, it is possible for a condition to onset but never be discovered.

If the ConditionOnset state's `target_encounter` is set to the name of a future encounter, then the condition will be diagnosed when that future encounter occurs.  If the `target_encounter` is set to the name of a previous encounter, then the condition will only be diagnosed if the ConditionOnset start time is the same as the encounter's start time.  See the Encounter section above for more details.

### Future Implementation Considerations

Although the generic module framework supports a distinction between a condition's onset date and diagnosis date, currently only the diagnosis date is recorded.  In the future, the `Synthea::Output::Record::condition` method should be updated to support an onset date.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"ConditionOnset"`. |
| `target_encounter` | `string` | Either an `"attribute"` or a `"State_Name"` referencing a<br/>future or concurrent `Encounter` state. |
| `assign_to_attribute` | `string` | **(optional)** The name of the `"attribute"` to assign this state to. |
| `reason` | `string` | **(optional)** Either an `"attribute"` or a `"State_Name"`<br/>referencing a _previous_ `ConditionOnset` state. |
| `codes` | `[]` | One or more codes that describe the Condition. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#snomed-codes). |

### Example

The following is an example of a `ConditionOnset` that should be diagnosed at the `"ED_Visit"` Encounter.

```json
"Appendicitis": {
  "type": "ConditionOnset",
  "target_encounter": "ED_Visit",
  "assign_to_attribute": "appendicitis",
  "codes": [
  	{
	  "system": "SNOMED-CT",
	  "code": "47693006",
	  "display": "Rupture of appendix"
    }
  ]
}
```


## ConditionEnd

The `ConditionEnd` state type indicates a point in the module where a currently active condition should be ended, for example if the patient has been cured of a disease. The `ConditionEnd` state supports three ways of specifying the condition to end:

1. By `codes[]`, specifying the system, code, and display name of the condition to end
2. By `condition_onset`, specifying the name of the `ConditionOnset` state in which the condition was onset
3. By `referenced_by_attribute`, specifying the name of the `attribute` to which a previous `ConditionOnset` state assigned a condition

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"ConditionEnd"`. |

And one of: 

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_onset` | `string` | The `"State_Name"` of a _previous_ `ConditionOnset` state.
| `referenced_by_attribute` | `string` | The name of the `"attribute"` the condition was assigned to. |
| `codes` | `[]` | One or more codes that described the Condition. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#snomed-codes). |

### Examples

The following is an example of a `ConditionEnd` state that ends the condition Influenza by code:

```json
"End_Influenza": {
  "type": "ConditionEnd",
  "codes": [
    {
      "system": "SNOMED-CT",
      "code": "6142004",
      "display": "Influenza"
    }
  ]
}
```

The following is an example of a `ConditionEnd` state that ends a condition by attribute:

```json
"End_Cold": {
  "type": "ConditionEnd",
  "referenced_by_attribute": "common_cold"
},
```

The following is an example of a `ConditionEnd` state that ends a condition by the name of a `ConditionOnset` state:

```json
"End_Pre_Diabetes": {
  "type": "ConditionEnd",
  "condition_onset": "Pre_Diabetes"
},
```


## MedicationOrder

The `MedicationOrder` state type indicates a point in the module where a medication should be prescribed.  The MedicationOrder state must come after the `target_encounter` Encounter state in the module, but must have the same start time as that Encounter; otherwise it will not be recorded in the patient's record.  See the Encounter section above for more details.

The `MedicationOrder` also supports identifying a previous `ConditionOnset` or the name of an `attribute` as the `reason` for the prescription.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"MedicationOrder"`. |
| `target_encounter` | `string` | Either an `"attribute"` or a `"State_Name"` referencing a<br/>_previous_ but concurrent `Encounter` state. |
| `assign_to_attribute` | `string` | **(optional)** The name of the `"attribute"` to assign this state to. |
| `reason` | `string` | **(optional)** Either an `"attribute"` or a `"State_Name"`<br/>referencing a _previous_ `ConditionOnset` state. |
| `codes` | `[]` | One or more codes that describe the Medication.<br/>Must be valid [RxNorm codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#rxnorm-codes). |

### Example

The following is an example of a `MedicationOrder` that should be prescribed at the `"Annual_Checkup"` Encounter and cite the `"Diabetes"` condition as the reason:

```json
"Prescribe_Metformin": {
  "type": "MedicationOrder",
  "target_encounter": "Annual_Checkup",
  "assign_to_attribute": "diabetes_medication",
  "reason": "Diabetes",
  "codes": [
    {
      "system": "RxNorm",
      "code": "860975",
      "display": "24 HR Metformin hydrochloride 500 MG Extended Release Oral Tablet"
    }
  ]
}
```


## MedicationEnd

The `MedicationEnd` state type indicates a point in the module where a currently prescribed medication should be ended. The `MedicationEnd` state supports three ways of specifying the medication to end:

1. By `codes[]`, specifying the code system, code, and display name of the medication to end
2. By `medication_order`, specifying the name of the `MedicationOrder` state in which the medication was prescribed
3. By `referenced_by_attribute`, specifying the name of the `attribute` to which a previous `MedicationOrder` state assigned a medication

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"MedicationEnd"`. |

And one of: 

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `medication_order` | `string` | The `"State_Name"` of a _previous_ `MedicationOrder` state.
| `referenced_by_attribute` | `string` | The name of the `"attribute"` the medication was assigned to. |
| `codes` | `[]` | One or more codes that described the MedicationOrder.<br/>Must be valid [RxNorm codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#rxnorm-codes). |

### Examples

The following is an example of a `MedicationEnd` that ends a prescription for Metformin, specified by code:

```json
"End_Metformin": {
  "type": "MedicationEnd",
  "codes": [
    {
      "system": "RxNorm",
      "code": "860975",
      "display": "24 HR Metformin hydrochloride 500 MG Extended Release Oral Tablet"
    }
  ]
}
```

The following is an example of a `MedicationEnd` that ends a prescription by attribute:

```json
"End_Insulin_Medicine": {
  "type": "MedicationEnd",
  "referenced_by_attribute": "insulin_medicine"
},
```

The following is an example of a `MedicationEnd` that ends a prescription for a medication by the name of a `MedicationOrder` state:

```json
"End_Bromocriptine": {
  "type": "MedicationEnd",
  "medication_order": "Bromocriptine_Start"
},
```

## CarePlanStart

The `CarePlanStart` state type indicates a point in the module where a care plan should be prescribed. The CarePlanStart state must come after a `target_encounter` Encounter state in the module, but must have the same start time as that Encounter; otherwise it will not be recorded in the patient's record. See the Encounter section above for more details. One or more `codes` describes the care plan and a list of `activities` describes what the care plan entails.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"CarePlanStart"`. |
| `target_encounter` | `string` | Either an `"attribute"` or a `"State_Name"` referencing a<br/>_previous_ but concurrent `Encounter` state. |
| `assign_to_attribute` | `string` | **(optional)** The name of the `"attribute"` to assign this state to. |
| `reason` | `string` | **(optional)** Either an `"attribute"` or a `"State_Name"`<br/>referencing a _previous_ `ConditionOnset` state. |
| `codes` | `[]` | One or more codes that describe the CarePlan.<br/>Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#snomed-codes). |
| `activities` | `[]` | **(optional)** One or more codes that describe the CarePlan.<br/>Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#snomed-codes). |

### Example

The following is an example of a `CarePlanStart` that should be prescribed at the `"Annual_Checkup"` Encounter and cite the `"Diabetes"` ConditionOnset as the reason:

```json
"Diabetes_CarePlan": {
  "type": "CarePlanStart",
  "target_encounter": "Annual_Checkup",
  "assign_to_attribute": "diabetes_careplan",
  "reason": "Diabetes",
  "codes": [
    {
      "system": "SNOMED-CT",
      "code": "698360004",
      "display": "Diabetes self management plan"
    }
  ],
  "activities": [
    {
      "system": "SNOMED-CT",
      "code": "160670007",
      "display": "Diabetic diet"
    }
  ]
}
```

## CarePlanEnd

The `CarePlanEnd` state type indicates a point in the module where a currently prescribed care plan should be ended. The `CarePlanEnd` state supports three ways of specifying the care plan to end:

1. By `codes[]`, specifying the code system, code, and display name of the care plan to end
2. By `careplan`, specifying the name of the `CarePlanStart` state in which the care plan was prescribed
3. By `referenced_by_attribute`, specifying the name of the `attribute` to which a previous `CarePlanStart` state assigned a care plan

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"CarePlanEnd"`. |

And one of: 

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `careplan` | `string` | The `"State_Name"` of a _previous_ `CarePlanStart` state.
| `referenced_by_attribute` | `string` | The name of the `"attribute"` the medication was assigned to. |
| `codes` | `[]` | One or more codes that described the care plan.<br/>Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#snomed-codes). |

### Examples

The following is an example of a `CarePlanEnd` that ends a prescription for "diabetes self management", by code:

```json
"End_Diabetes_Careplan": {
  "type": "CarePlanEnd",
  "codes": [
    {
      "system": "SNOMED-CT",
      "code": "698360004",
      "display": "Diabetes self management plan"
    }
  ]
}
```

The following is an example of a `CarePlanEnd` that ends a prescription for a care plan by attribute:

```json
"End_Diabetes_Careplan": {
  "type": "CarePlanEnd",
  "referenced_by_attribute": "Diabetes_CarePlan"
}
```

The following is an example of a `CarePlanEnd` that ends a prescription for a care plan by the name of the `CarePlanStart` state where the care plan was prescribed:

```json
"End_Diabetes_Careplan": {
  "type": "CarePlanEnd",
  "careplan": "Diabetes_Self_Management"
}
```

## Procedure

The `Procedure` state type indicates a point in the module where a procedure should be performed.  The Procedure state must come after the `target_encounter` Encounter state in the module, but must have the same start time as that Encounter; otherwise it will not be recorded in the patient's record.  See the Encounter section above for more details.

The `Procedure` also supports identifying a previous `ConditionOnset` or an attribute as the `reason` for the procedure.

### Future Implementation Considerations

Currently, the generic module framework does not provide a way to indicate the duration of a procedure.  This would probably be best implemented by simply introducing a property in `Procedure` to indicate its intended duration.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Procedure"`. |
| `target_encounter` | `string` | Either an `"attribute"` or a `"State_Name"` referencing a<br/>_previous_  but concurrent `Encounter` state. |
| `reason` | `string` | **(optional)** Either an `"attribute"` or a `"State_Name"` referencing a<br/>_previous_ `ConditionOnset` state. |
| `codes` | `[]` | One or more codes that describe the Procedure. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#snomed-codes). |

### Example

The following is an example of a Procedure that should be performed at the `"Inpatient_Encounter"` Encounter and cite the `"Appendicitis"` condition as the reason.

```json
"Appendectomy": {
  "type": "Procedure",
  "target_encounter": "Inpatient_Encounter",
  "reason": "Appendicitis",
  "codes": [
    {
      "system": "SNOMED-CT",
      "code": "6025007",
      "display": "Laparoscopic appendectomy"
    }
  ]
}
```

## Observation

The `Observation` state type indicates a point in the module where an observation is recorded. Observations include clinical findings, vital signs, lab tests, etc.

If the Observation state's `target_encounter` is set to the name of a future encounter, then the observation will be recorded when that future encounter occurs.  If the `target_encounter` is set to the name of a previous encounter, then the observation will only be recorded if the Observation start time is the _same_ as the encounter's start time.  See the Encounter section above for more details.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Observation"`. |
| `target_encounter` | `string` | Either an `"attribute"` or a `"State_Name"` referencing a concurrent or future `Encounter` state. |
| `unit` | `string` | The unit of measure in which the observation is recorded (e.g. `"cm"`). |
| `exact` or `range` | `{}` | **(choice)**  Must be either an `exact` or a `range` observation. |
| `codes` | `[]` | One or more codes that describe the Observation. Must be valid [LOINC codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#loinc-codes). |

##### `exact`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `quantity` | `numeric` | The exact number of `unit` observed. |

##### `range`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `low` | `numeric` | The lowest (inclusive) number of `unit` observed. |
| `high` | `numeric` | The greatest (inclusive) number of `unit` observed. |

The actual value for the observation will be chosen randomly from this range.

### Example

The following is an example of an Observation that should be taken at the `"Checkup"` Encounter, that will result in an observation of body height between 40 and 60 cm:

```json
"Height_Measurement": {
  "type": "Observation",
  "target_encounter": "Checkup",
  "unit": "cm",
  "range": {
    "low": 40,
    "high": 60
  },
  "codes": [
    {
      "system": "LOINC",
      "code": "8302-2",
      "display": "Body Height"
    }
  ]
}
```


## Symptom

The `Symptom` state type adds or updates a patient's symptom. Synthea tracks symptoms in order to drive a patient's encounters, on a scale of 1-100. A symptom may be tracked for multiple conditions, in these cases only the highest value is considered. See also the [[Symptom|Generic Module Framework: Logic#symptom]] logical condition type.


### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Symptom"`. |
| `symptom` | `string` | The name of the symptom being tracked. |
| `cause` | `string` | The underlying cause of the symptom. Defaults to the name of the module if not set. |
| `exact` or `range` | `{}` | **(choice)**  Must be either an `exact` value or a `range` of values for the symptom. |

##### `exact`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `quantity` | `numeric` | The exact severity of the symptom, between `0` and `100`. |

##### `range`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `low` | `numeric` | The lowest (inclusive) severity of the symptom, between `0` and `100`. |
| `high` | `numeric` | The greatest (inclusive) severity of the symptom, between `0` and `100`. |

The actual value for the symptom will be chosen randomly from this range.

### Examples

The following is an example of a Symptom state that sets the symptom value for Chest Pain to be exactly 27:

```json
"Chest_Pain": {
  "type": "Symptom",
  "symptom": "Chest Pain",
  "cause": "Asthma",
  "exact": {
    "quantity": 27
  }
}
```

The following is an example of a Symptom state that sets the symptom value for Chest Pain to be between 90 and 100:

```json
"Chest_Pain": {
  "type": "Symptom",
  "symptom": "Chest Pain",
  "cause": "Heart Attack",
  "range": {
    "low": 90,
    "high": 100
  }
}
```

## SetAttribute

The `SetAttribute` state type sets a specified attribute on the patient entity. In addition to the `assign_to_attribute` property on `MedicationOrder`/`ConditionOnset`/etc states, this state allows for arbitrary text or values to be set on an `attribute`, or for the `attribute` to be reset.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"SetAttribute"`. |
| `attribute` | `string` | The name of the [attribute](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#attributes) being set. |
| `value` | `numeric`, `boolean`, or `string` | **(optional)** The value of the attribute. If not provided, `value` is set to `null`. |

### Examples

The following is an example of a `SetAttribute` state that sets the value of the `attribute` `"number_of_children"` to `3`:

```json
"Set_Number_Of_Children": {
  "type": "SetAttribute",
  "attribute": "number_of_children",
  "value": 3
}
```

The following is an example of a `SetAttribute` state that clears the `"number_of_children"` `attribute`:

```json
'Clear_Number_Of_Children": {
  "type": "SetAttribute",
  "attribute": "number_of_children"
}
```

## Counter

The `Counter` state type increments or decrements a specified numeric `attribute` on the patient entity. In essence, this state counts the number of times it is processed. Note: The `attribute` is initialized with a default value of `0` if not previously set.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Counter"`. |
| `attribute` | `string` | The name of the [attribute](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#attributes) being counted. |
| `action` | `string` | **(choice)** One of [`"increment"`, `"decrement"`]. |

### Examples

The following is an example of a `Counter` that decrements the `"loop_index"` by `1` every time it is processed:

```json
"Decrement_Loop_Index": {
  "type": "Counter",
  "attribute": "loop_index",
  "action": "decrement"
}
```

The following is an example of a `Counter` that increments the number of `"bronchitis_occurrences"` by `1` every time it is processed:

```json
"Count_A_Bronchitis_Occurence": {
  "type": "Counter",
  "attribute": "bronchitis_occurrences",
  "action": "increment"
}
```

## Death

The `Death` state type indicates a point in the module at which the patient dies or the patient is given a terminal diagnosis (e.g. "you have 3 months to live").  When the Death state is processed, the patient's death is immediately recorded (the `alive?` method will return `false`) unless `range` or `exact` attributes are specified, in which case the patient will die sometime in the future.  In either case the module will continue to progress to the next state(s) for the current time-step.  Typically, the Death state should transition to a `Terminal` state.

### Implementation Warning

If a `Death` state is processed after a `Delay`, it may cause inconsistencies in the record.  This is because the `Delay` implementation must _rewind_ time to correctly honor the requested delay duration.  If it rewinds time, and then the patient dies at the rewinded time, then any modules that were processed before the generic module may have created events and records with a timestamp _after_ the patient's death.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Death"`. |
| `exact` or `range` | `{}` | **(optional, choice)** An exact amount or range of time that the patient has left to live. |

##### `exact`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `quantity` | `numeric` | The number of `unit` left to live. |
| `unit` | `string` | The unit of time, e.g. `"days"`. Must be a valid [unit of time](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#units).|

##### `range`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `low` | `numeric` | The lowest (inclusive) number of `unit` left to live. |
| `high` | `numeric` | The greatest (inclusive) number of `unit` left to live. |
| `unit` | `string` | The unit of time, e.g. `"days"`. Must be a valid [unit of time](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#units).|

### Examples

The following example is an immediate death:

```json
"Death": {
  "type": "Death"
}
```

This example gives the patient 3 - 5 months to live:

```json
"Eventual_Death": {
  "type": "Death",
  "range": {
    "low": 3,
    "high": 5,
    "unit": "months"
  }
}
```