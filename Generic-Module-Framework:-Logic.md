
The `Guard` state, `conditional_transition`, and `complex_transition` use conditional (boolean) logic.  The following condition types are currently supported:

* [Gender](#gender), [Socioeconomic Status](#socioeconomic-status), [Race](#race)
* [Age](#age), [Date](#date)
* [Symptom](#symptom), [Observation](#observation), [Vital Sign](#vitalsign)
* [Active Condition](#active-condition), [Active Medication](#active-medication), [Active CarePlan](#active-careplan)
* [Attribute](#attribute), [PriorState](#priorstate)
* [And](#and), [Or](#or), [Not](#not)
* [At Least](#at-least), [At Most](#at-most)
* [True](#true), [False](#false)

The following condition types should be considered for future versions:

* PriorEvent: check if a patient event occurred

## Gender

The `Gender` condition type tests the patient's gender.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Gender"`. |
| `gender` | `string` | One of [`"M"`, `"F"`]. |

### Example

The following `Gender` condition will return `true` if the patient is male; `false` otherwise:

```json
{
  "condition_type": "Gender",
  "gender": "M" 
}
```

## Age

The `Age` condition type tests the patient's age.  

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Age"`. |
| `operator` | `string` | One of [`"=="`, `"!="`, `"<"`, `">"`, `"<="`, `">="`]. |
| `quantity` | `numeric` | The number of `units` to test. |
| `unit` | `string` | Must be a valid [unit of age](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#units).  |

### Example

The following `Age` condition will return `true` if the patient is 40 years old or more:

```json
{
  "condition_type": "Age",
  "operator": ">=",
  "quantity": 40,
  "unit": "years"
}
```

## Date

The `Date` condition type tests the current year, month, or date being simulated.  For example, this may be used to drive different logic depending on the suggested medications or procedures of different time periods, or model different frequency of conditions.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Date"`. |
| `operator` | `string` | One of [`"=="`, `"!="`, `"<"`, `">"`, `"<="`, `">="`]. |
| `year` | `numeric` | The year to test the current year of the simulation against. |
| `month` | `numeric` | The month to test the current month of the simulation against. |
| `date` | `string` | The date to test the current date of the simulation against. Must be formatted `"yyyy-MM-dd HH:mm:ss.SSS"` |

### Example

The following Date condition will return `true` if the year is `1990` or later:

```json
{
  "condition_type": "Date",
  "operator": ">=",
  "year": 1990
}
```

The following Date condition will return `true` if the month is `4` (April):

```json
{
  "condition_type": "Date",
  "operator": "==",
  "month": 4
}
```

The following Date condition will return `true` if the date is `"2018-07-04 01:00:00.000"` (July 4th, 2018 at 1 hour, 0 minutes, 0 seconds and 0 milliseconds) or earlier:

```json
{
  "condition_type": "Date",
  "operator": "<=",
  "date": "2018-07-04 01:00:00.000"
}
```

## Socioeconomic Status

The `Socioeconomic Status` condition type tests the patient's socioeconomic status. Socioeconomic status is based on income, education, and occupation, and is categorized in Synthea as `"High"`, `"Middle"`, or `"Low"`.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Socioeconomic Status"`. |
| `category` | `string` | One of [`"High"`, `"Middle"`, `"Low"`]. |

### Example

The following `Socioeconomic Status` condition will return `true` if the patient is "Middle" Socioeconomic Status; `false` otherwise:

```json
{
  "condition_type": "Socioeconomic Status",
  "category": "Middle"
}
```

## Race

The `Race` condition type tests a patient's race. Synthea supports the following races:

`"White"`, `"Native"` (Native American), `"Hispanic"`, `"Black"`, `"Asian"`, and `"Other"`.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Race"`. |
| `race` | `string` | One of the races listed above. |

### Example

The following `Race` condition will return `true` if the patient is "Hispanic"; `false` otherwise:

```json
{
  "condition_type": "Race",
  "race": "Hispanic"
}
```

## Symptom

The `Symptom` condition type tests a patient's current symptoms. Synthea tracks symptoms in order to drive a patient's encounters, on a scale of 1-100. A symptom may be tracked for multiple conditions, in these cases only the highest value is considered. See also the [[Symptom|Generic Module Framework: States#symptom]] state.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Symptom"`. |
| `symptom` | `string` | The name of the symptom to test against. |
| `operator` | `string` | One of [`"=="`, `"!="`, `"<"`, `">"`, `"<="`, `">="`]. |
| `value` | `numeric` | The value between `0` and `100` to test the symptom against. |

### Example

The following `Symptom` condition will return `true` if the patient's symptom score for "Chest Pain" is greater than or equal to 50:

```json
{
  "condition_type": "Symptom",
  "symptom" : "Chest Pain",
  "operator": ">=",
  "value": 50
}
```

## Observation

The `Observation` condition type tests the most recent observation of a given type against a given value. 

### Implementation Warnings

* Synthea does not support conversion between arbitrary units, so all observations of a given type are expected to be made in the same units. 
* The given observation must have been recorded prior to performing this logical check, unless the `operator` is `is nil` or `is not nil`. Otherwise, the GMF will raise an exception that the observation value cannot be compared as there has been no observation made.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Observation"`. |
| `operator` | `string` | One of [`"=="`, `"!="`, `"<"`, `">"`, `"<="`, `">="`, `"is nil"`, `"is not nil"`]. |
| `value` | `numeric` | The value to test the observation against (if numeric). |
| `value_code`| `{}` | The value to test the observation against (if code). Must be a valid LOINC code (https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#loinc-codes). |
| `codes` or `referenced_by_attribute` | `[]` or `string` | **(choice)** Must be either a list of [LOINC codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#loinc-codes) or an `attribute` referencing an observation. |

### Examples

The following `Observation` condition will return `true` if a previous `Observation` with code `LOINC [72107-6] Mini Mental State Examination` has a value greater than `22`:

```json
{
  "condition_type": "Observation",
  "codes": [
    {
      "system": "LOINC",
      "code": "72107-6",
      "display": "Mini Mental State Examination"
    }
  ],
  "operator": ">",
  "value": 22
}
```

The following `Observation` condition will return `true` if the most recent `Observation` referenced by the attribute `"diabetes_test_performed"` on the patient has any value that is not nil. In other words, if any observation has been made and stored in attribute `diabetes_test_performed`, this condition will return `true`:

```json
{
  "condition_type": "Observation",
  "referenced_by_attribute": "diabetes_test_performed",
  "operator": "is not nil"
}
```

## Vital Sign
The `Vital Sign` condition type tests a patient's current vital signs. Synthea tracks vital signs in order to drive a patient's physical condition, and are recorded in observations. See also the [[Symptom|Generic Module Framework: States#VitalSign]] state.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Vital Sign"`. |
| `vital_sign` | `string` | The name of the vital sign to test against. |
| `operator` | `string` | One of [`"=="`, `"!="`, `"<"`, `">"`, `"<="`, `">="`]. |
| `value` | `numeric` | The value to test the vital sign against. |

### Example

The following `Vital Sign` condition will return `true` if the patient's Systolic Blood Pressure vital sign is greater than 120:

```json
{
    "condition_type" : "Vital Sign",
    "vital_sign" : "Systolic Blood Pressure",
    "operator" : ">",
    "value" : 120
  },
```

## Active Condition

The `Active Condition` condition type tests whether a given condition is currently diagnosed and active on the patient. 

### Future Implementation Considerations

Currently to check if a condition has been added but not diagnosed, it is possible to use the [`PriorState`](#priorstate) condition to check if the state has been processed. In the future it may be preferable to add a distinct "Present Condition" logical condition to clearly specify the intent of looking for a present but not diagnosed condition.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Active Condition"`. |
| `codes` or `referenced_by_attribute` | `[]` or `string` | **(choice)** Must be either a list of [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#snomed-codes) or an `attribute` referencing a condition. |

### Examples

The following `Active Condition` condition will return `true` if the patient currently has an active diagnosis of `SNOMED-CT 73211009 [Diabetes mellitus]`:

```json
{
  "condition_type": "Active Condition",
  "codes": [
    {
      "system": "SNOMED-CT",
      "code": "73211009",
      "display": "Diabetes mellitus"
    }
  ]
}
```

The following `Active Condition` condition will return `true` if the condition referenced by the attribute `"alzheimers_variant"` on the patient is currently active. In other words, if any condition has onset and is stored in the attribute "Alzheimer's Variant", this condition will return `true`:

```json
{
  "condition_type": "Active Condition",
  "referenced_by_attribute": "alzheimers_variant"
}
```

## Active Medication

The `Active Medication` condition type tests wheter a given medication is currently prescribed and active for the patient.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Active Medication"`. |
| `codes` or `referenced_by_attribute` | `[]` or `string` | **(choice)** Must be either a list of [RxNorm codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#snomed-codes) or an `attribute` referencing a medication order. |

### Examples

The following `Active Medication` condition will return `true` if the patient currently has an active medication of `RxNorm 849727 Naproxen sodium 220 MG [Aleve]`:

```json
{
  "condition_type": "Active Medication",
  "codes": [
    {
      "system": "RxNorm",
      "code": "849727",
      "display": "Naproxen sodium 220 MG [Aleve]"
    }
  ]
}
```

The following `Active Medication` condition will return `true` if the medication referenced by the attribute `"pain_medication"` on the patient is currently active. In other words, if any medication order has started and is stored in the attribute `"pain_medication"`, this condition will return `true`:

```json
{
  "condition_type" : "Active Medication",
  "referenced_by_attribute" : "pain_medication"
}
```

## Active CarePlan

The `Active CarePlan` condition type tests whether a given care plan is currently prescribed and active for the patient. 

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Active CarePlan"`. |
| `codes` or `referenced_by_attribute` | `[]` or `string` | **(choice)** Must be either a list of [SNOMED codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Primitive-Types#snomed-codes) or an `attribute` referencing a care plan. |

### Examples

The following `Active CarePlan` condition will return `true` if the patient currently has an active care plan of `SNOMED-CT 698360004 [Diabetes self management plan]`:

```json
{
  "condition_type": "Active CarePlan",
  "codes": [
    {
      "system": "SNOMED-CT",
      "code": "698360004",
      "display": "Diabetes self management plan"
    }
  ]
}
```

The following Active CarePlan condition will return `true` if the care plan referenced by the attribute `"diabetes_careplan"` on the patient is currently active. In other words, if any care plan has started and is stored in attribute `"diabetes_careplan"`, this condition will return `true`:

```json
{
  "condition_type" : "Active CarePlan",
  "referenced_by_attribute" : "diabetes_careplan"
}
```

## PriorState

The `PriorState` condition type tests the progression of the patient through the module, and checks if a specific state has already been processed (in other words, the state is in the module's state history). The search for the state may be limited by time or the name of another state.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"PriorState"`. |
| `name` | `string` | The name of the state to check for in the module's history. |
| `since` | `string` | The name of a state at which this logic will stop checking for the target state.
| `within` | `{}` | An exact timespan to use to limit the search for the target state

### Example

The following `PriorState` condition will return `true` if the patient has already passed through a state called `"Emergency_Encounter"`, `false` otherwise:

```json
{
  "condition_type": "PriorState",
  "name": "Emergency_Encounter"
}
```

The following `PriorState` condition will return `true` if the patient has passed through a state called `"Emergency_Encounter"` within the last 3 years, or `false` if the patient has never passed through the state, or last passed through the state more than 3 years ago:

```json
{
  "condition_type": "PriorState",
  "name": "Emergency_Encounter",
  "within": { "quantity": 3, "unit" : "years"}
}
```

The following `PriorState` condition will return `true` if the patient has already passed through a state called `"Emergency_Encounter"` since last passing through a state called "Followup", `false` otherwise. (In other words, if the patient hit state "Emergency_Encounter" more recently than state "Followup". Note that it is not necessary for the state "Followup" to have been hit for this to return true.)

```json
{
  "condition_type": "PriorState",
  "name": "Emergency_Encounter",
  "since" : "Followup"
}
```


## Attribute

The `Attribute` condition type tests a named `attribute` on the patient entity.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Attribute"`. |
| `attribute` | `string` | The name of the `attribute` to check. |
| `operator` | `string` | One of [`"=="`, `"!="`, `"<"`, `">"`, `"<="`, `">="`, `"is nil"`, `"is not nil"`]. |
| `value` | `numeric` or `string` | The value to compare against the `attribute`. |

### Examples

The following `Attribute` condition will return `true` if attribute `"opioid_prescription"` on the patient is set to `"Vicodin"`:

```json
{
  "condition_type": "Attribute",
  "attribute": "opiod_prescription",
  "operator": "==",
  "value": "Vicodin"
}
```

The following `Attribute` condition will return `true` if attribute `"opioid_prescription"` on the patient has any value that is not `nil`:

```json
{
  "condition_type": "Attribute",
  "attribute": "opioid_prescription",
  "operator": "is not nil"
}
```

## And

The `And` condition type tests that a set of sub-conditions are _all_ true.  If _all_ sub-conditions are true, it will return `true`, but if _any_ are false, it will return `false`.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"And"`. |
| `conditions` | `[]` | An array of sub-conditions to test. |

### Example

The following `And` condition will return `true` if the patient is male and at least 40 years old; `false` otherwise:

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

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Or"`. |
| `conditions` | `[]` | An array of sub-conditions to test. |

### Example

The following `Or` condition will return `true` if the patient is male _or_ the patient is at least 40 years old; `false` otherwise:

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

## At Least

The `At Least` condition type tests that a minimum number of conditions from a set of sub-conditions are true.  If the _minimum_ number or more sub-conditions are true, it will return `true`, but if less than the minimum are true, it will return `false`. (If the _minimum_ is the same as the number of sub-conditions provided, this is equivalent to the `And` condition. If _minimum_ is 1, this is equivalent to the `Or` condition.)

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"At Least"`. |
| `minimum` | `numeric` | Must be between `1` and the number of sub-conditions (inclusive).
| `conditions` | `[]` | An array of sub-conditions to test. |

### Example

The following "At Least" condition will return `true` if 2 or more of the following 3 conditions are met: 1) the patient is male, 2) the patient is at least 40 years old, 3) the patient has an active Diabetes self management plan; `false` otherwise. For example, if the patient is Male, age 46, and does not have a Diabetes self management plan, this condition will return true. If the patient is Female, age 67, and does not have a Diabetes self management plan, this condition will return false:

``` json
{
  "condition_type": "At Least",
  "minimum": 2,
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
    },
    {
      "condition_type" : "Active CarePlan",
      "codes": [
        {
          "system": "SNOMED-CT",
          "code": "698360004",
          "display": "Diabetes self management plan"
        }
      ]
    }
  ]
}
```

## At Most

The `At Most` condition type tests that a maximum number of conditions from set of sub-conditions are true.  If the _maximum_ number or fewer sub-conditions are true, it will return `true`, but if more than the maximum are true, it will return `false`. 

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"At Most"`. |
| `maximum` | `numeric` | Must be between `1` and the number of sub-conditions (inclusive).
| `conditions` | `[]` | An array of sub-conditions to test. |

## Example

The following "At Most" condition will return `true` if 2 or fewer of the following 3 conditions are met: 1) the patient is male, 2) the patient is at least 40 years old, 3) the patient has an active Diabetes self management plan; `false` otherwise. For example, if the patient is Male, age 46, and does not have a Diabetes self management plan, this condition will return true. If the patient is Female, age 67, and does not have a Diabetes self management plan, this condition will return true. If the patient is Male, age 41, and has a Diabetes self management plan, this condition will return false:

``` json
{
  "condition_type": "At Most",
  "maximum" : 2,
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
    },
    {
      "condition_type": "Active CarePlan",
      "codes": [
        {
          "system": "SNOMED-CT",
          "code": "698360004",
          "display": "Diabetes self management plan"
        }
      ]
    }
  ]
}
```

## Not

The `Not` condition type negates its sub-condition. If the sub-condition is true, it will return `false`; if the sub-condition is false, it will return `true`.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"Not"`. |
| `condition` | `{}` | The sub-condition to negate. |

### Example

The following Not condition will return `true` if the patient is _not_ male; `false` otherwise:

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

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"True"`. |

### Example

The following True condition always returns `true`:

```json
{
  "condition_type": "True"
}
```

## False

The `False` condition always returns `false`.  This condition is mainly used for testing purposes and is not expected to be used in any real module.

### Supported Properties

| Attribute | Type | Description |
|:----------|:-----|:------------|
| `condition_type` | `string` | Must be `"False"`. |

### Example

The following False condition always returns `false`:

```json
{
  "condition_type": "False"
}
```