
The Guard state and Conditional transition use conditional (boolean) logic.  The following condition types are currently supported: [Gender](#gender), [Age](#age), [Date](#date), [Socioeconomic Status](#socioeconomic-status), [Symptom](#symptom), [PriorState](#priorstate), [Active Condition](#activecondition), [Observation](#observation), [Attribute](#attribute), [And](#and), [Or](#or), [Not](#not), [True](#true), and [False](#false).

The following condition types should be considered for future versions:

* PriorEvent: check if a patient event occurred

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


## Date

The `Date` condition type tests the current year being simulated.  For example, this may be used to drive different logic depending on the suggested medications or procedures of different time periods, or model different frequency of conditions.

**Supported Properties**

* **condition_type**: must be "Date" _(required)_
* **operator**: indicates how to compare the current year of simulation against the _year_.  Valid _operator_ values are: `<`, `<=`, `==`, `>=`, `>`, and `!=`. _(required)_
* **year**: the year to test the current year of simulation against _(required)_

**Example**

The following Date condition will return `true` if the year is 1990 or later.

```json
{
  "condition_type": "Year",
  "operator": ">=",
  "year": 1990
}
```


## Socioeconomic Status

The `Socioeconomic Status` condition type tests the patient's socioeconomic status. Socioeconomic status is based on income, education, and occupation, and is categorized in Synthea as "High", "Medium", or "Low".

**Supported Properties**

* **condition_type**: must be "Socioeconomic Status" _(required)_
* **category**: the gender to test for.  Synthea currently defines the following categories: `High`, `Middle`, or `Low`. _(required)_

**Example**

The following Socioeconomic Status condition will return `true` if the patient is "Middle" Socioeconomic Status; `false` otherwise.

```json
{
  "condition_type" : "Socioeconomic Status",
  "category" : "Middle"
}
```

## Symptom

The `Symptom` condition type tests a patient's current symptoms. Synthea tracks symptoms in order to drive a patient's encounters, on a scale of 1-100. A symptom may be tracked for multiple conditions, in these cases only the highest value is considered. See also the [[Symptom|Generic Module Framework: States#symptom]] state.

**Supported Properties**

* **condition_type**: must be "Symptom" _(required)_
* **symptom**: the name of the symptom to test against. _(required)_
* **operator**: indicates how to compare the current symptom score against the _value_.  Valid _operator_ values are: `<`, `<=`, `==`, `>=`, `>`, and `!=`. _(required)_
* **value**: the value to test the current symptom score against _(required)_

**Example**

The following Symptom condition will return `true` if the patient's symptom score for "Chest Pain" is greater than or equal to 50

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

**Implementation Warnings**

* Synthea does not support conversion between arbitrary units, so all observations of a given type are expected to be made in the same units. 
* The given observation must have been recorded prior to performing this logical check, unless the `operator` is `is nil` or `is not nil`. Otherwise, the GMF will raise an exception that the observation value cannot be compared as there has been no observation made.

**Supported Properties**

* **condition_type**: must be "Observation" _(required)_
* **codes[]**: a list of codes indicating the observation(optional, required if `referenced_by_attribute` is not set)_
  * **system**: the code system.  Currently, only `LOINC` is allowed. _(required)_
  * **code**: the code _(required)_
  * **display**: the human-readable code description _(required)_
* **referenced_by_attribute**: the name of the Attribute in which a previous `Observation` state recorded an observation _(optional, required if `codes[]` is not set)_
* **operator**: indicates how to compare the actual attribute value against the value.  Valid _operator_ values are: `<`, `<=`, `==`, `>=`, `>`, `!=`, `is nil`, and `is not nil`. _(required)_
* **value**: the value to test the most recent observation value against _(required, unless `operator` is `is nil` or `is not nil`)_

**Example**

The following Observation condition will return `true` if the most recent Observation for LOINC '72107-6' [Mini Mental State Examination] on the patient has a value greater than 22.

```json
{
    "condition_type" : "Observation",
    "codes" : [{
      "system" : "LOINC",
      "code" : "72107-6",
      "display" : "Mini Mental State Examination"
    }],
    "operator" : ">",
    "value" : 22
}
```

The following Observation condition will return `true` if the most recent Observation referenced by the attribute 'Diabetes Test Performed' on the patient has any value that is not nil. In other words, if any observation has been made and stored in attribute 'Diabetes Test Performed', this condition will return `true`.

```json
{
    "condition_type" : "Observation",
    "referenced_by_attribute" : "Diabetes Test Performed",
    "operator" : "is not nil"
}
```



## Active Condition

The `Active Condition` condition type tests whether a given condition is currently diagnosed and active on the patient. 


**Future Implementation Considerations**

Currently to check if a condition has been added but not diagnosed, it is possible to use the [PriorState](#priorstate) condition to check if the state has been processed. In the future it may be preferable to add a distinct "Present Condition" logical condition to clearly specify the intent of looking for a present but not diagnosed condition.

**Supported Properties**

* **condition_type**: must be "Active Condition" _(required)_
* **codes[]**: a list of codes indicating the condition (optional, required if `referenced_by_attribute` is not set)_
  * **system**: the code system.  Currently, only `SNOMED-CT` is allowed. _(required)_
  * **code**: the code _(required)_
  * **display**: the human-readable code description _(required)_
* **referenced_by_attribute**: the name of the Attribute in which a previous `ConditionOnset` state added a condition _(optional, required if `codes[]` is not set)_


**Example**

The following Active Condition condition will return `true` if the patient currently has an active diagnosis of SNOMED-CT 73211009 [Diabetes mellitus].

```json
{
    "condition_type" : "Active Condition",
    "codes": [{
        "system": "SNOMED-CT",
        "code": "73211009",
        "display": "Diabetes mellitus"
    }]
}
```

The following Active Conditon condition will return `true` if the condition referenced by the attribute "Alzheimer's Variant" on the patient is currently active. In other words, if any condition has onset and is stored in attribute "Alzheimer's Variant", this condition will return `true`.

```json
{
    "condition_type" : "Active Condition",
    "referenced_by_attribute" : "Alzheimer's Variant"
}
```


## PriorState

The `PriorState` condition type tests the progression of the patient through the module, and checks if a specific state has already been processed (in other words, the state is in the module's state history). 

**Supported Properties**

* **condition_type**: must be "PriorState" _(required)_
* **name**: the name of the state to check for in the module's history. _(required)_

**Example**

The following PriorState condition will return `true` if the patient has already passed through a state called 'EmergencyEncounter', false otherwise.

```json
{
  "condition_type" : "PriorState",
  "name" : "EmergencyEncounter"
}
```


## Attribute

The `Attribute` condition type tests a named attribute on the patient entity.

**Supported Properties**

* **condition_type**: must be "Attribute" _(required)_
* **attribute**: the name of the attribute to test against _(required)_
* **operator**: indicates how to compare the actual attribute value against the value.  Valid _operator_ values are: `<`, `<=`, `==`, `>=`, `>`, `!=`, `is nil`, and `is not nil`. _(required)_
* **value**: the value to test the _attribute_ value against _(required, unless `operator` is `is nil` or `is not nil`)_

**Example**

The following Attribute condition will return `true` if attribute 'Opioid Prescription' on the patient is set to 'Vicodin'.

```json
{
  "condition_type": "Attribute",
  "attribute": "Opioid Prescription",
  "operator": "==",
  "value": "Vicodin"
}
```

The following Attribute condition will return `true` if attribute 'Opioid Prescription' on the patient has any value that is not nil.

```json
{
  "condition_type": "Attribute",
  "attribute": "Opioid Prescription",
  "operator": "is not nil"
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
