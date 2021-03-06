The Generic Module Framework currently supports a variety of state types (see below). During generation, Synthea processes each module state-by-state. Synthea will always continue processing the module and [[transition|Generic Module Framework: Transitions]] to the next state, unless the current state is:

1. [Terminal](#terminal), in which case the module ends.
2. [Delay](#delay), in which case the module will only continue after the specified amount of time has passed.
3. [Guard](#guard), in which case the module will only continue if the specified `allow` [[logic|Generic Module Framework: Logic]] resolves to `true`.
4. `wellness` [Encounter](#encounter) **and** the patient is not actively in a `wellness` encounter (in other words, it will wait until the next scheduled wellness visit).
5. [CallSubmodule](#callsubmodule), in which case whether or not processing continues is a result of the submodule.

The Generic Module Framework currently supports the following states:

**Control States**
* [Initial](#initial)
* [Terminal](#terminal)
* [Simple](#simple)
* [Guard](#guard)
* [Delay](#delay)
* [SetAttribute](#setattribute)
* [Counter](#counter)
* [CallSubmodule](#callsubmodule)

**Clinical States**
* [Encounter](#encounter), [EncounterEnd](#encounterend) 
* [ConditionOnset](#conditiononset), [ConditionEnd](#conditionend) 
* [AllergyOnset](#allergyonset), [AllergyEnd](#allergyend) 
* [MedicationOrder](#medicationorder), [MedicationEnd](#medicationend) 
* [CarePlanStart](#careplanstart), [CarePlanEnd](#careplanend) 
* [Procedure](#procedure)
* [ImagingStudy](#imagingstudy)
* [Device](#device), [DeviceEnd](#device)
* [SupplyList](#supplylist)
* :warning: [VitalSign](#vitalsign) 
* [Observation](#observation), [MultiObservation](#multiobservation), [DiagnosticReport](#diagnosticreport) 
* [Symptom](#symptom) 
* [Death](#death)

**Other States**
* [Physiology](#physiology)

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
| `unit` | `string` | The unit of time, e.g. `"days"`. Must be a valid [unit of time](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#units).|

##### `range`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `low` | `numeric` | The lowest (inclusive) number of `unit`. |
| `high` | `numeric` | The greatest (inclusive) number of `unit`. |
| `unit` | `string` | The unit of time, e.g. `"days"`. Must be a valid [unit of time](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#units).|

The exact delay will be chosen randomly from the specified range as a real number, using a uniform distribution. For example, if the unit of time selected is `years`, Synthea will _not_ produce delays in year increments; instead, it will select a random date/time within the specified range, regardless of the units in which the range is specified.

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
"Delay_Until_Symptoms_Improve": {
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

An Encounter state _without_ the `wellness` property set will be processed and recorded in the patient record immediately.  Since this creates an encounter, the `encounter_class` and one or more `codes` must be specified in the state configuration.  This is how generic modules can introduce encounters that are not already scheduled by other modules.

### Encounters and Related Events

Encounters are typically the mechanism through which a patient's record will be updated. This makes sense since most recorded events (diagnoses, prescriptions, and procedures) should happen in the context of an encounter. When an Encounter state is successfully processed, Synthea will look through the previously processed states for un-recorded ConditionOnset or AllergyOnset instances that indicate that Encounter (by name) as the `target_encounter`. If Synthea finds any, they will be recorded in the patient's record at the time of the encounter. This is the mechanism for onsetting a disease _before_ it is discovered and diagnosed.

### Current Encounter

During a single time step, if an Encounter state is processed it becomes the `current_encounter`. This encounter will be used as the encounter for all subsequent clinical states, for example a Procedure or MedicationOrder. When an [EncounterEnd](#encounterend) state is reached, the `current_encounter` ends and remains `nil` until another Encounter is processed. If Synthea cannot identify a `current_encounter` for a clinical state an error is raised.


### Encounter Classes

Common encounter classes are:

* `"wellness"`: A regularly scheduled appointment with a PCP.
* `"ambulatory"`: A visit in a variety of outpatient settings, e.g. a visit to a PCP or a specialist
* `"emergency"`: A visit to an emergency department
* `"urgentcare"`: A visit to an urgent care facility
* `"inpatient"`: A non-emergency visit to a hospital, e.g. for routine surgery
* `"outpatient"`: A visit in a variety of outpatient settings, e.g. a visit to a PCP or a specialist. See `ambulatory`.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Encounter"`. |
| `wellness` | `boolean` | If `true`, Synthea blocks 1-3 years until the next wellness encounter.<br/>**(optional)** if `false`. |
| `encounter_class` | `string` | One of [`"wellness"`, `"ambulatory"`, `"emergency"`, `"urgentcare"`, `"inpatient"`, `"outpatient"`] |
| `reason` | `string` | **(optional)** Either an `"attribute"` or a `"State_Name"` referencing a<br/>_previous_ `ConditionOnset` state. |
| `codes` | `[]` | One or more codes that describe the Encounter. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). |

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

## EncounterEnd

The `EncounterEnd` state type indicates the end of the encounter the patient is currently in, for example when the patient leaves a clinician's office, or is discharged from a hospital. (See also [Encounter#Current Encounter](#currentencounter) for more information on "current encounters".) The time the encounter ended is recorded on the patient's record.

### Note on Wellness Encounters

Because wellness encounters are scheduled and initiated outside the generic modules, and a single wellness encounter may contain observations or medications from multiple modules, an EncounterEnd state will not record the end time for a wellness encounter. Hence it is not strictly necessary to use an EncounterEnd state to end the wellness encounter. Still, it is recommended to use an EncounterEnd state to mark a clear end to the encounter.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"EncounterEnd"`. |
| `discharge_disposition` | `code` | **(optional)** A single code that describes the discharge disposition. Should be a code from the [Discharge Disposition set](https://phinvads.cdc.gov/vads/ViewValueSet.action?id=9C24960D-3B7D-4B6A-B86B-8F101867BD4F)

### Example

The following is an example of an `EncounterEnd` state that ends the current encounter, where the patient is discharged to their home.

```
"End_Encounter" : {
   "type" : "EncounterEnd",
   "discharge_disposition" : {
     "system" : "NUBC",
     "code" : "01",
     "display" : "Discharged to home care or self care (routine discharge)"
   }
 }
```

## ConditionOnset

The `ConditionOnset` state type indicates a point in the module where the patient acquires a condition.  This is _not_ necessarily the same as when the condition is diagnosed and recorded in the patient's record.  In fact, it is possible for a condition to onset but never be discovered.

If the ConditionOnset state's `target_encounter` is set to the name of a future encounter, then the condition will only be diagnosed when that future encounter occurs.

### Future Implementation Considerations

Although the generic module framework supports a distinction between a condition's onset date and diagnosis date, currently only the diagnosis date is recorded.  In the future, the `Synthea::Output::Record::condition` method should be updated to support an onset date.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"ConditionOnset"`. |
| `target_encounter` | `string` | A `"State_Name"` referencing a<br/>future or concurrent `Encounter` state. |
| `assign_to_attribute` | `string` | **(optional)** The name of the `"attribute"` to assign this state to. |
| `codes` | `[]` | One or more codes that describe the Condition. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). |

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
| `codes` | `[]` | One or more codes that described the Condition. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). |

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


## AllergyOnset

The `AllergyOnset` state type indicates a point in the module where the patient acquires an allergy.  This is _not_ necessarily the same as when the allergy is diagnosed and recorded in the patient's record.  In fact, it is possible for an allergy to onset but never be discovered.

If the AllergyOnset state's `target_encounter` is set to the name of a future encounter, then the allergy will only be diagnosed when that future encounter occurs. 
### Future Implementation Considerations

Although the generic module framework supports a distinction between an allergy's onset date and diagnosis date, currently only the diagnosis date is recorded.  In the future, the `Synthea::Output::Record::condition` method should be updated to support an onset date.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"AllergyOnset"`. |
| `target_encounter` | `string` | A `"State_Name"` referencing a<br/>future or concurrent `Encounter` state. |
| `assign_to_attribute` | `string` | **(optional)** The name of the `"attribute"` to assign this state to. |
| `codes` | `[]` | One or more codes that describe the Allergy. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). |

### Example

The following is an example of an `AllergyOnset` that should be diagnosed at the `"ED_Visit"` Encounter.

```json
"Peanut_allergy": {
  "type": "AllergyOnset",
  "target_encounter": "ED_Visit",
  "codes": [
    {
    "system": "SNOMED-CT",
    "code": "91935009",
    "display": "Allergy to peanuts"
    }
  ]
}
```


## AllergyEnd

The `AllergyEnd` state type indicates a point in the module where a currently active allergy should be ended, for example if the patient's allergy subsides with time. The `AllergyEnd` state supports three ways of specifying the allergy to end:

1. By `codes[]`, specifying the system, code, and display name of the allergy to end
2. By `allergy_onset`, specifying the name of the `AllergyOnset` state in which the allergy was onset
3. By `referenced_by_attribute`, specifying the name of the `attribute` to which a previous `AllergyOnset` state assigned a condition

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"AllergyEnd"`. |

And one of: 

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `allergy_onset` | `string` | The name of a _previous_ `AllergyOnset` state.
| `referenced_by_attribute` | `string` | The name of the `"attribute"` the allergy was assigned to. |
| `codes` | `[]` | One or more codes that described the Allergy. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). |

### Examples

The following is an example of an `AllergyEnd` state that ends a peanut allergy by code:

```json
"Allergy_Resolves": {
  "type": "AllergyEnd",
  "codes": [
    {
      "system": "SNOMED-CT",
      "code": "91935009",
      "display": "Allergy to peanuts"
    }
  ]
}
```

The following is an example of an `AllergyEnd` state that ends an allergy by attribute:

```json
"Allergy_Resolved": {
  "type": "AllergyEnd",
  "referenced_by_attribute": "allergy_variant"
},
```

The following is an example of an `AllergyEnd` state that ends an allergy by the name of a `AllergyOnset` state:

```json
"End_Allergy": {
  "type": "AllergyEnd",
  "allergy_onset": "Shellfish_allergy"
},
```


## MedicationOrder

The `MedicationOrder` state type indicates a point in the module where a medication is prescribed.  `MedicationOrder` states may only be processed during an Encounter, and so must occur after the target Encounter state and before the EncounterEnd. See the [Encounter](#encounter) section above for more details.

The `MedicationOrder` state supports identifying a previous `ConditionOnset` or the name of an `attribute` as the `reason` for the prescription.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"MedicationOrder"`. |
| `assign_to_attribute` | `string` | **(optional)** The name of the `"attribute"` to assign this state to. |
| `reason` | `string` | **(optional)** Either an `"attribute"` or a `"State_Name"`<br/>referencing a _previous_ `ConditionOnset` state. |
| `codes` | `[]` | One or more codes that describe the Medication.<br/>Must be valid [RxNorm codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#rxnorm-codes). |
|`prescription` | `{}` | **(optional)** Detailed information about the prescription, including dosage information (see below). |
| `administration` | `boolean` | **(optional)** If `true` this represents the administration of a medication rather than a prescription. |
| `chronic` | `boolean` | **(optional)** If `true`, a medication is considered a chronic medication for a chronic condition. This will cause Synthea to reissue the same medication as a new prescription AND discontinue the old prescription at each wellness encounter. |

##### `prescription`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `refills` | `numeric` | **(optional)** The number of refills to allow. If not specified, defaults to `0`. |
| `as_needed` | `boolean` | If `true`, the medication may be taken as needed instead of on a specific schedule. **(optional)** if `false`. |
| `dosage` | `{}` | The amount and frequency that the medication should be taken. |
| `duration` | `{}` | The total length of time that the medication should be taken for. |
| `instructions` | `[]` | **(optional)** Specific instructions for taking the medication. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes) from the [Instruction Valueset](http://hl7.org/fhir/2017Jan/valueset-additional-instructions-codes.html).  |

##### `dosage`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `amount` | `numeric` | The number of doses to take each time. |
| `frequency` | `numeric` | The number of times the medication should be taken per period. |
| `period` | `numeric` | The number of `units` that represents a period. |
| `unit` | `string` | The unit of time describing the period. |

##### `duration`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `quantity` | `numeric` | The number of `units` that the medication should be taken for. |
| `unit` | `string` | The unit of time describing the duration. |


### Examples

The following is an example of a `MedicationOrder` that should be prescribed at the `"Annual_Checkup"` Encounter and cite the `"Diabetes"` condition as the reason:

```json
"Prescribe_Metformin": {
  "type": "MedicationOrder",
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

The following is an example of a `MedicationOrder` that also specifies dosage information. In this example, the patient should take "1 dose, twice per day for 2 weeks, before food":

```json
"Examplitol": {
  "type": "MedicationOrder",
  "reason": "Examplitis",
  "codes": [
    {
      "system": "RxNorm",
      "code": "123456",
      "display": "Examplitol 100mg [Examplitol]"
    }
  ],
  "prescription": {
    "refills": 2,
    "dosage": {
      "amount": 1,
      "frequency": 2,
      "period": 1,
      "unit": "days"
    },
    "duration": {
      "quantity": 2,
      "unit": "weeks"
    },
    "instructions": [
      {
        "system": "SNOMED-CT",
        "code": "311501008",
        "display": "Half to one hour before food"
      }
    ]
  }
}
```

If `as_needed` is true then dosage information does not need to be specified:

```json
"Examplitol": {
  "type": "MedicationOrder",
  "reason": "Examplitis",
  "codes": [
    {
      "system": "RxNorm",
      "code": "123456",
      "display": "Examplitol 100mg [Examplitol]"
    }
  ],
  "prescription": {
    "as_needed": true
  }
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
| `codes` | `[]` | One or more codes that described the MedicationOrder.<br/>Must be valid [RxNorm codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#rxnorm-codes). |

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

The `CarePlanStart` state type indicates a point in the module where a care plan should be prescribed.  `CarePlanStart` states may only be processed during an Encounter, and so must occur after the target Encounter state and before the EncounterEnd. See the [Encounter](#encounter) section above for more details. One or more `codes` describes the care plan and a list of `activities` describes what the care plan entails.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"CarePlanStart"`. |
| `assign_to_attribute` | `string` | **(optional)** The name of the `"attribute"` to assign this state to. |
| `reason` | `string` | **(optional)** Either an `"attribute"` or a `"State_Name"`<br/>referencing a _previous_ `ConditionOnset` state. |
| `codes` | `[]` | One or more codes that describe the CarePlan.<br/>Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). |
| `activities` | `[]` | **(optional)** One or more codes that describe the CarePlan.<br/>Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). |
| `goals` | `[]` | **(optional)** One or more goals associated with the CarePlan. |

##### `goals`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `observation` | `{}` | **(optional)** Logical test that should be true when the goal has been achieved. For instance, BMI < 30. |
| `text` | `string` | **(optional)** Goal description. |
| `addresses` | `[]` | The name of an attribute assigned via `ConditionOnset`. |

##### `observation`: ( See also the [[Observation|Generic Module Framework: Logic#observation]] logical condition type.)

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `codes` | `[]` | List of codes. |
| `operator` | `string` | One of [`"=="`, `"!="`, `"<"`, `">"`, `"<="`, `">="`, `"is nil"`, `"is not nil"`]. |
| `value` | `numeric` | The value to test the observation against. |

### Example

The following is an example of a `CarePlanStart` that should be prescribed at the `"Annual_Checkup"` Encounter and cite the `"Diabetes"` ConditionOnset as the reason:

```json
"Diabetes_CarePlan": {
  "type": "CarePlanStart",
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
| `codes` | `[]` | One or more codes that described the care plan.<br/>Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). |

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

The `Procedure` state type indicates a point in the module where a procedure should be performed. `Procedure` states may only be processed during an Encounter, and so must occur after the target Encounter state and before the EncounterEnd. See the [Encounter](#encounter) section above for more details. Optionally, you may define a duration of time that the procedure takes.

The `Procedure` also supports identifying a previous `ConditionOnset` or an attribute as the `reason` for the procedure.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Procedure"`. |
| `reason` | `string` | **(optional)** Either an `"attribute"` or a `"State_Name"` referencing a<br/>_previous_ `ConditionOnset` state. |
| `codes` | `[]` | One or more codes that describe the Procedure. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). |
| `duration` | `{}` | **(optional)** The length of the procedure, as a range with unit. |

##### `duration`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `low` | `numeric` | The minimum length of time the procedure takes. |
| `high` | `numeric` | The maximum length of time the procedure takes. |
| `unit` | `string` | The unit of time, e.g. `"minutes"`. Must be a valid [unit of time](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#units). |


### Example

The following is an example of a Procedure that should be performed at the `"Inpatient_Encounter"` Encounter and cite the `"Appendicitis"` condition as the reason. From start to finish, an appendectomy takes 2-3 hours.

```json
"Appendectomy": {
  "type": "Procedure",
  "reason": "Appendicitis",
  "codes": [
    {
      "system": "SNOMED-CT",
      "code": "6025007",
      "display": "Laparoscopic appendectomy"
    }
  ],
  "duration": {
    "low": 2,
    "high": 3,
    "unit": "hours"
  }
}
```

## ImagingStudy

The `ImagingStudy` state type indicates a point in the module where an imaging study was performed on the patient. An ImagingStudy consists of one or more `series` or images.

Each series is a set of images taken of a specific `body_site` (for example, "Right Knee") using a specific `modality` (for example "Magnetic Resonance").

Images taken during each series are represented as `instances`, where each instance captures a `title` and `sop_class` describing the DICOM Subject-Object Pair (SOP) that constitutes the image.

An ImagingStudy **must** occur during an Encounter.

### Usage Notes

When an ImagingStudy state is processed, both an ImagingStudy and a Procedure are added to the patient's record. Because the [FHIR U.S. Core Implementation Guide](https://www.hl7.org/fhir/us/core/) does not profile ImagingStudy, large vendors in the U.S. will support Procedure via the Argonauts program, but not necessarily ImagingStudy. If modules produce both, then the fact that a patient had an X-Ray or MRI or Ultrasound will be found by clients no matter the system importing/exposing the data.

An ImagingStudy state typically precedes a series of Observation states and their encompassing DiagnosticReport state. Observations and DiagnosticReports should be used to capture the physician's interpretation of the ImagingStudy's results.

### Supported Properties

| Attribute | Type | Description |
|:----------|:----:|:------------|
| `type` | `string` | Must be `"ImagingStudy"`. |
| `procedure_code` | `{}` | A [SNOMED](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes) code describing the ImagingStudy as a Procedure. |
| `series` | `[]` | One or more series of images. |

##### `series`:

| Attribute | Type | Description |
|:----------|:----:|:------------|
| `body_site` | `{}` | A [SNOMED](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes) Body Structures code describing what part of the body the images in the series were taken of. |
| `modality` | `{}` | A [DICOM-DCM](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#dicom-dcm-codes) code describing the method used to take the images. |
| `instances` | `[]` | One or more image instances taken in the series. |

##### `instances`:

| Attribute | Type | Description |
|:----------|:----:|:------------|
| `title` | `string` | A title for the image. |
| `sop_class` | `{}` | A [DICOM-SOP](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#dicom-sop-codes) code describing the type of image taken. |

### Example

The following is an example of an ImagingStudy state capturing a single X-Ray image taken of the patient's ankle:

```
"Ankle_X_Ray": {
  "type": "ImagingStudy",
  "procedure_code": {
    "system": "SNOMED-CT",
    "code": "19490002",
    "display": "Ankle X-ray"
  },
  "series": [
    {
      "body_site": {
        "system": "SNOMED-CT",
        "code": "344001",
        "display": "Ankle"
      },
      "modality": {
        "system": "DICOM-DCM",
        "code": "DX",
        "display": "Digital Radiography"
      },
      "instances": [
        {
          "title": "Image of ankle",
          "sop_class": {
            "system": "DICOM-SOP",
            "code": "1.2.840.10008.5.1.4.1.1.1.1",
            "display": "Digital X-Ray Image Storage"
          }
        }
      ]
    }
  ]
}
```

## Device

The `Device` state type indicates a point in the module where a permanent or semi-permanent device or piece of medical equipment is associated to a patient. Examples of devices include a prosthetic leg, or a pacemaker.

Note that this represents the association of the device to the patient, not the actual attachment or implantation, if necessary. If the device is to be implanted in the patient, or otherwise physically attached, that should be represented with a separate [Procedure](#procedure) state.

A Device state **must** occur during an Encounter.

### Supported Properties

| Attribute | Type | Description |
|:----------|:----:|:------------|
| `type` | `string` | Must be `"Device"`. |
| `code` | `code` | A single SNOMED code that describes the device. Preferred to be a code from the [Device type value set](https://www.hl7.org/fhir/valueset-device-type.html) |
| `manufacturer` | `string` | **(optional)** The manufacturer of the device. Should be used in situations where there is one primary manufacturer for the type of device being referenced |
| `model` | `string` | **(optional)** The device model name. Should be used along with `manufacturer` in situations where there is one primary manufacturer and model for the type of device being used |
| `assign_to_attribute` | `string` | **(optional)** The name of the `"attribute"` to assign this state to. |


### Example

The following is an example of an Device state for an implantable artificial heart:

```
"Artificial_Heart": {
  "type": "Device",
  "code": {
    "system": "SNOMED-CT",
    "code": "13459008",
    "display": "Temporary artificial heart prosthesis, device (physical object)"
  },
  "manufacturer": "SynCardia",
  "model": "Total Artificial Heart"
}
```

## DeviceEnd
The `DeviceEnd` state type indicates the point that a permanent or semi-permanent device (for example, a prosthetic, or pacemaker) is removed from a person. The actual procedure in which the device is removed is not automatically generated
and should be added separately. 
The `DeviceEnd` state supports three ways of specifying the condition to end:

1. By `codes[]`, specifying the system, code, and display name of the device to end
2. By `device`, specifying the name of the `Device` state in which the device was added
3. By `referenced_by_attribute`, specifying the name of the `attribute` to which a previous `Device` state assigned a device

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"DeviceEnd"`. |

And one of: 

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `device` | `string` | The `"State_Name"` of a _previous_ `Device` state.
| `referenced_by_attribute` | `string` | The name of the `"attribute"` the device was assigned to. |
| `codes` | `[]` | One or more codes that described the Device. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). |

### Examples

The following is an example of a `DeviceEnd` state that removes a ventilator device by code:

```json
"Remove_Ventilator": {
  "type": "DeviceEnd",
  "codes": [
    {
      "system": "SNOMED-CT",
      "code": "706172005",
      "display": "Ventilator (physical object)"
    }
  ]
}
```

## SupplyList

The `SupplyList` state type indicates that certain supplies are required for the current encounter. This may be as simple as examination gloves or audiovisual equipment for a telehealth encounter. 

A SupplyList state **must** occur during an Encounter. Multiple SupplyList states may occur during a single encounter.

### Supported Properties

| Attribute | Type | Description |
|:----------|:----:|:------------|
| `type` | `string` | Must be `"Device"`. |
| `supplies` | `[]` | One or more supply items required during this encounter. |


##### `supplies`:

| Attribute | Type | Description |
|:----------|:----:|:------------|
| `code` | `{}` | A [SNOMED](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes) code describing the required item |
| `quantity` | `number` | How many of this item are required |



### Example

The following is an example of a SupplyList state indicating a need for 2 gloves and 1 gown:

```
"Necessary_Supplies": {
  "type": "SupplyList",
  "supplies": [
    {
      "quantity": 2,
      "code": {
        "system": "SNOMED-CT",
        "code": "52291003",
        "display": "Glove, device (physical object)"
      }
    },
    {
      "quantity": 1,
      "code": {
        "system": "SNOMED-CT",
        "code": "788177008",
        "display": "Examination gown, single-use (physical object)"
      }
    }
  ]
}
```

## VitalSign
The `VitalSign` state type indicates a point in the module where a patient's vital sign is set. Vital Signs represent the physical state of the patient, in contrast to Observations which are the recording of that physical state.

> :warning: **_WARNING:_**  This state only sets a value. It is NOT recorded in the patient record. The `vital_sign` attribute MUST be a legal value from the [Vital Sign Enumeration](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/world/concepts/VitalSign.java).

### Usage Notes
In general, the Vital Sign should be used if the value directly affects the patient's physical condition. For example, high blood pressure directly increases the risk of heart attack so any conditional logic that would trigger a heart attack should reference a Vital Sign instead of an Observation. On the other hand, if the value only affects the patient's care, using just an Observation would be more appropriate. For example, it is the observation of MMSE that can lead to a diagnosis of Alzheimer's; MMSE is an observed value and not a physical metric, so it should not be stored in a VitalSign.

### Supported Properties
| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"VitalSign"`. |
| `vital_sign`| `string` | The name of the Vital Sign, which may be referenced later. :warning: Currently supported vital signs can be found in the [Vital Sign Enumeration](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/world/concepts/VitalSign.java).
| `unit` | `string` | The unit of measure in which the vital sign is measured (e.g. `"cm"`). |

And one of: 

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `exact` | `{}` | The exact value to be recorded for this Observation |
| `range` | `{}` | A range of values from which one value will be recorded |


##### `exact`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `quantity` | `numeric` | The exact value of `unit` observed. |

##### `range`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `low` | `numeric` | The lowest (inclusive) numeric value of `unit` observed. |
| `high` | `numeric` | The greatest (inclusive) numeric value of `unit` observed. |

### Example
The following is an example of a VitalSign state that sets the patient's Systolic Blood Pressure to 120 mmHg:

```
"SetBP" : {
   "type" : "VitalSign",
   "vital_sign" : "Systolic Blood Pressure",
   "exact" : { "quantity" : "120" },
   "unit" : "mmHg"
},
```

## Observation

The `Observation` state type indicates a point in the module where an observation is recorded. Observations include clinical findings, vital signs, lab tests, etc.  `Observation` states may only be processed during an Encounter, and so must occur after the target Encounter state and before the EncounterEnd. See the [Encounter](#encounter) section above for more details.

### Observation Categories
Common observation categories include:

- `"vital-signs"` :  Clinical observations measure the body's basic functions such as such as blood pressure, heart rate, respiratory rate, height, weight, body mass index, head circumference, pulse oximetry, temperature, and body surface area.
- `"procedure"` : Observations generated by other procedures. This category includes observations resulting from interventional and non-interventional procedures excluding lab and imaging (e.g. cardiology catheterization, endoscopy, electrodiagnostics, etc.). Procedure results are typically generated by a clinician to provide more granular information about component observations made during a procedure, such as where a gastroenterologist reports the size of a polyp observed during a colonoscopy.
- `"laboratory"` : The results of observations generated by laboratories. Laboratory results are typically generated by laboratories providing analytic services in areas such as chemistry, hematology, serology, histology, cytology, anatomic pathology, microbiology, and/or virology. These observations are based on analysis of specimens obtained from the patient and submitted to the laboratory.
- `"exam"` : Observations generated by physical exam findings including direct observations made by a clinician and use of simple instruments and the result of simple maneuvers performed directly on the patient's body.
- `"social-history"` : The Social History Observations define the patient's occupational, personal (e.g. lifestyle), social, and environmental history and health risk factors, as well as administrative data such as marital status, race, ethnicity and religious affiliation.


### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Observation"`. |
| `category` | `string` | The type of Observation being made, for example . Must be a valid [HL7 Observation Category](http://hl7.org/fhir/2017Jan/valueset-observation-category.html). |
| `unit` | `string` | The unit of measure in which the observation is recorded (e.g. `"cm"`). |
| `codes` | `[]` | One or more codes that describe the Observation. Must be valid [LOINC codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#loinc-codes). |

And one of: 

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `exact` | `{}` | The exact value to be recorded for this Observation |
| `range` | `{}` | A range of values from which one value will be recorded |
| `attribute` | `string` | The name of an attribute that contains the value of some Observation |
| `vital_sign` | `string` | The name of a Vital Sign that contains a value |
| `value_code` | `{}` | The code that describes the value of the observation. Must be a valid [SNOMED code](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). | 

##### `exact`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `quantity` | `numeric` | The exact value of `unit` observed. |

##### `range`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `low` | `numeric` | The lowest (inclusive) numeric value of `unit` observed. |
| `high` | `numeric` | The greatest (inclusive) numeric value of `unit` observed. |

The actual value for the observation will be chosen randomly from this range.

### Example

The following is an example of an Observation that should be taken at the `"Checkup"` Encounter, that will result in an observation of body height between 40 and 60 cm:

```json
"Height_Measurement": {
  "type": "Observation",
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


## MultiObservation
The `MultiObservation` state indicates that some number of Observation values should be embedded directly within a single observation. For example consider Blood Pressure, which is really 2 values, Systolic and Diastolic Blood Pressure. `MultiObservation` states may only be processed during an Encounter, and so must occur after the target Encounter state and before the EncounterEnd. See the [Encounter](#encounter) section above for more details.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"MultiObservation"`. |
| `category` | `string` | The type of Observation being made, for example . Must be a valid [HL7 Observation Category](http://hl7.org/fhir/2017Jan/valueset-observation-category.html). |
| `codes` | `[]` | One or more codes that describe the Observation. Must be valid [LOINC codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#loinc-codes). |
| `observations` | `Observation[]` | The list of observations that should be included as sub-observations within this observation. Each observation should be a valid [Observation](#observation) state, excluding the type and transition.

### Example
The following example shows a `MultiObservation` for Blood Pressure:

```
    "Record_BP": {
      "type": "MultiObservation",
      "category": "vital-signs",
      "codes": [
        {
          "system": "LOINC",
          "code": "55284-4",
          "display": "Blood Pressure"
        }
      ],
      "observations": [
        {
          "category": "vital-signs",
          "vital_sign": "Systolic Blood Pressure",
          "codes": [
            {
              "system": "LOINC",
              "code": "8480-6",
              "display": "Systolic Blood Pressure"
            }
          ],
          "unit": "mmHg"
        },
        {
          "category": "vital-signs",
          "vital_sign": "Diastolic Blood Pressure",
          "codes": [
            {
              "system": "LOINC",
              "code": "8462-4",
              "display": "Diastolic Blood Pressure"
            }
          ],
          "unit": "mmHg"
        }
      ]
    }
```

## DiagnosticReport
The `DiagnosticReport` state indicates that some number of Observations should be grouped together within a single Diagnostic Report. This can be used when multiple observations are part of a single panel.  `DiagnosticReport` states may only be processed during an Encounter, and so must occur after the target Encounter state and before the EncounterEnd. See the [Encounter](#encounter) section above for more details.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"DiagnosticReport"`. |
| `codes` | `[]` | One or more codes that describe the DiagnosticReport. Must be valid [LOINC codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#loinc-codes). |
| `observations` | `Observation[]` | The list of observations that should be included as sub-observations within this observation. Each observation should be a valid [Observation](#observation) state, excluding the type and transition.

### Example
The following example shows a `DiagnosticReport` which groups the 4 observations into a Lipid Panel Diagnostic Report:

```
    "Record_LipidPanel": {
      "type": "DiagnosticReport",
      "number_of_observations": 4,
      "codes": [
        {
          "system": "LOINC",
          "code": "57698-3",
          "display": "Lipid Panel"
        }
      ],
      "observations": [
        {
          "category": "laboratory",
          "vital_sign": "Total Cholesterol",
          "codes": [
            {
              "system": "LOINC",
              "code": "2093-3",
              "display": "Total Cholesterol"
            }
          ],
          "unit": "mg/dL"
        },
        {
          "category": "laboratory",
          "vital_sign": "Triglycerides",
          "codes": [
            {
              "system": "LOINC",
              "code": "2571-8",
              "display": "Triglycerides"
            }
          ],
          "unit": "mg/dL"
        },
        {
          "category": "laboratory",
          "vital_sign": "LDL",
          "codes": [
            {
              "system": "LOINC",
              "code": "18262-6",
              "display": "Low Density Lipoprotein Cholesterol"
            }
          ],
          "unit": "mg/dL"
        },
        {
          "category": "laboratory",
          "vital_sign": "HDL",
          "codes": [
            {
              "system": "LOINC",
              "code": "2085-9",
              "display": "High Density Lipoprotein Cholesterol"
            }
          ],
          "unit": "mg/dL"
        }
      ],
      "direct_transition": "Lab_ACR"
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
| `probability` | `numeric` | **(optional)** Probability (0-100%) the patient will have the symptom. |
| `exact` or `range` | `{}` | **(choice)**  Must be either an `exact` value or a `range` of values for the symptom. |

##### `probability`:
Probability is expressed from `0.0` to `1.0` (0 to 100%) representing the probability the patient will have the given symptom. If the probability is not met, then the Symptom is never created, as if the Symptom state did not exist. This value does not affect the value generated for the `range` attribute. If no value is given, the default is `1.0` representing a 100% chance the patient will have the Symptom.

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

The following is an example of a Symptom state that will occur in a patient 20% of the time:

```json
"Headache": {
  "type": "Symptom",
  "symptom": "Headache",
  "cause": "Tension",
  "probability": 0.2,
  "exact" : 10
}
```

## SetAttribute

The `SetAttribute` state type sets a specified attribute on the patient entity. In addition to the `assign_to_attribute` property on `MedicationOrder`/`ConditionOnset`/etc states, this state allows for arbitrary text or values to be set on an `attribute`, or for the `attribute` to be reset.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"SetAttribute"`. |
| `attribute` | `string` | The name of the [attribute](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#attributes) being set. |
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

The `Counter` state type increments or decrements a specified numeric `attribute` on the patient entity, by a given `amount`. In essence, this state can be used to count the number of times it is processed. Note: The `attribute` is initialized with a default value of `0` if not previously set.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Counter"`. |
| `attribute` | `string` | The name of the [attribute](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#attributes) being counted. |
| `action` | `string` | **(choice)** One of [`"increment"`, `"decrement"`]. |
| `amount` | `numeric` | The amount to increment or decrement the counter by when the state is hit. _(optional, defaults to `1` if not provided)_

### Examples

The following is an example of a `Counter` that increments the number of `"bronchitis_occurrences"` by `1` every time it is processed:

```json
"Count_A_Bronchitis_Occurence": {
  "type": "Counter",
  "attribute": "bronchitis_occurrences",
  "action": "increment"
}
```

The following is an example of a `Counter` that decrements the `"loop_index"` by `2` every time it is processed:

```json
"Decrement_Loop_Index": {
  "type": "Counter",
  "attribute": "loop_index",
  "action": "decrement",
  "amount": 2
}
```

## Death

The `Death` state type indicates a point in the module at which the patient dies or the patient is given a terminal diagnosis (e.g. "you have 3 months to live").  When the Death state is processed, the patient's death is immediately recorded (the `alive?` method will return `false`) unless `range` or `exact` attributes are specified, in which case the patient will die sometime in the future.  In either case the module will continue to progress to the next state(s) for the current time-step.  Typically, the Death state should transition to a `Terminal` state.

The Cause of Death listed on a Death Certificate can be specified in three ways:

1. By `codes[]`, specifying the system, code, and display name of the condition causing death.
2. By `condition_onset`, specifying the name of the `ConditionOnset` state in which the condition causing death was onset.
3. By `referenced_by_attribute`, specifying the name of the `attribute` to which a previous `ConditionOnset` state assigned a condition that caused death.

### Implementation Warning

If a `Death` state is processed after a `Delay`, it may cause inconsistencies in the record.  This is because the `Delay` implementation must _rewind_ time to correctly honor the requested delay duration.  If it rewinds time, and then the patient dies at the rewinded time, then any modules that were processed before the generic module may have created events and records with a timestamp _after_ the patient's death.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Death"`. |
| `exact` or `range` | `{}` | **(optional, choice)** An exact amount or range of time that the patient has left to live. |
| `codes` | `[]` | **(optional)** One or more codes that describe the Cause of Death. The first code is used on the Death Certificate. Must be valid [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#snomed-codes). |
| `condition_onset` | `string` | **(optional)** The `"State_Name"` of a _previous_ `ConditionOnset` state that is the Cause of Death to be listed on the Death Certificate. |
| `referenced_by_attribute` | `string` | **(optional)** The name of the `"attribute"` the condition was assigned to, which should be used as the Cause of Death on the Death Certificate. |

##### `exact`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `quantity` | `numeric` | The number of `unit` left to live. |
| `unit` | `string` | The unit of time, e.g. `"days"`. Must be a valid [unit of time](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#units).|

##### `range`:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `low` | `numeric` | The lowest (inclusive) number of `unit` left to live. |
| `high` | `numeric` | The greatest (inclusive) number of `unit` left to live. |
| `unit` | `string` | The unit of time, e.g. `"days"`. Must be a valid [unit of time](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#units).|

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

## CallSubmodule

The `CallSubmodule` state immediately processes a reusable series of states contained in a [submodule](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Submodules). These states are processes in the _same_ time step, starting with the submodule's `Initial` state. Once the submodule's `Terminal` state is reached, execution of the calling module resumes.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"CallSubmodule"`. |
| `submodule` | `string` | The name of a [submodule](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Submodules) to call. If no submodule with that name is found, an error is raised. |

### Example

In this example, the `medications/otc_pain_reliever` submodule is called to prescribe an over-the-counter pain reliever in the current time step:

```json
"Medication_Submodule": {
  "type": "CallSubmodule",
  "submodule": "medications/otc_pain_reliever"
}
```

## Physiology

The `Physiology` state type indicates a point in the module where the patient's physiology should be simulated, typically to support a particular `Observation` during a medical procedure.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Must be `"Physiology"`. |
| `model` | `string` | Path to the model file relative to `src/main/resources/physiology/models` |
| `solver` | `string` | Which differential equation solver to use for the simulation. One of ["adams_bashforth", "adams_moulton", "dormand_prince_54", "dormand_prince_853", "euler", "gragg_bulirsch_stoer", "higham_hall_54", "rosenbrock", "runge_kutta" |
| `step_size` | `numeric` | Simulation step size in seconds. |
| `sim_duration` | `numeric` | Simulation duration in seconds. |
| `lead_time` | `numeric` | **(optional)** Simulation lead time in seconds before output data is captured. Defaults to 0.0. |
| `alt_direct_transition` | `string` | Next state to transition to in the event Physiology States are turned off in synthea.properties. |
| `inputs` | `[]` | List of input definitions for the model. See Inputs below. |
| `outputs` | `[]` | List of output definitions for the model. See Outputs below. |

### Model files

Synthea currently supports physiology models in the Systems Biology Markup Language (SBML) format. Thousands of existing models are available in the [BioModels database](https://www.ebi.ac.uk/biomodels/). Model files must be placed in `src/main/resources/physiology/models`, and may be placed in a subfolder for organizational purposes.

### Inputs
Physiology model inputs are derived from Patient VitalSigns and/or attributes. Each entry in the inputs array supports the following fields:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Type of the Patient parameter. Required if using the ???from??? field. One of ["ATTRIBUTE", "VITAL_SIGN"]. |
| `from` | `string` | Name of the Patient attribute or VitalSign where the source value is stored. |
| `from_exp` | `string` | A mathematical expression which will evaluate to provide the input value. |
| `to` | `string` | Name of the model parameter to place the input value in. |

Either `from` or `fromExp` must be defined for each input definition. The `type` field is only required when using `from`. The ???fromExp??? allows for both VitalSigns and Attributes to be used in the equation. See Expression Processing.

### Outputs
Model outputs are derived from model parameters. They support many of the same fields as inputs, with a few key differences:

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `type` | `string` | Type of the target Patient parameter. One of ["ATTRIBUTE", "VITAL_SIGN"]. |
| `from` | `string` | Name of the model parameter to pull the latest value from as the output. |
| `from_list` | `string` | Name of the model parameter to pull the list of values from. `to` field *must* target an Attribute. |
| `from_exp` | `string` | A mathematical expression of model parameters which will evaluate to provide the output value. |
| `to` | `string` | Name of the Patient attribute / VitalSign to place the output value in. |
| `variance` | `numeric` | Amount of random variance to add to the resulting value. This uses a uniform random distribution across +/- variance range. |

Unlike all other instances where expressions may be used, the fromExp field in this case requires model parameters instead of Patient attributes and VitalSigns. This means the developer *must* be familiar with the variables defined by the model used and what their value indicates.

### Example

The following is an example of a `Physiology` state to simulate cardiac hemodynamics during an `Encounter`.

```json
"Simulate_CVS": {
      "type": "Physiology",
      "model": "circulation/Smith2004_CVS_human.xml",
      "solver": "runge_kutta",
      "step_size": 0.01,
      "sim_duration": 4.0,
      "lead_time": 2.0,
      "inputs": [
        {"from": "Pulmonary Resistance", "to": "R_pul"},
        {"from_exp": "#{BMI} * #{BMI Multiplier}", "to": "R_sys"}
      ],
      "outputs": [
        {
          "from": "V_ao",
          "to": "Final Aortal Volume",
          "type": "Attribute"
        },
        {
          "from_list": "P_ao",
          "to": "Arterial Pressure Values",
          "type": "Attribute"
        },
        {
          "from_exp": "((Max(#{V_lv}) - Min(#{V_lv})) / Max(#{V_lv}))*100",
          "to": "LVEF",
          "type": "Attribute"
        },
        {
          "from_exp": "Max(#{P_ao})",
          "to": "SBP",
          "type": "Attribute"
        },
        {
          "from_exp": "Min(#{P_ao})",
          "to": "DBP",
          "type": "Attribute"
        }
      ],
      "direct_transition": "ChartObservation",
      "alt_direct_transition": "ContrivedLVEF"
    }
```
