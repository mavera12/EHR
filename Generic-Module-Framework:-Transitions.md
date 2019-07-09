
The generic module framework currently supports the following transitions:

* [Direct](#direct)
* [Distributed](#distributed)
* [Conditional](#conditional)
* [Complex](#complex)
* [Table](#table)

## Direct

`Direct` transitions are the simplest of transitions.  They transition directly to the indicated state.  The value of a `direct_transition` is simply the name of the state to transition to.

### Example

_Please note that, for simplicity, transition examples in this document will always start at an `Initial` state.  In real modules, transitions can be applied to any state (except a `Terminal` state) and can transition to any state._

The following example demonstrates a state that should transition directly to the `"Delay_For_Encounter"` state:

```json
"Initial": {
  "type": "Initial",
  "direct_transition": "Delay_For_Encounter"
}
```

## Distributed

`Distributed` transitions will transition to one of several possible states based on the configured distribution.  Distribution values are from `0.0` to `1.0`, such that a value of `0.55` would indicate a 55% chance of transitioning to the corresponding state.  A `distributed_transition` consists of an array of `distribution`/`transition` pairs for which the **`distribution` values should sum up to `1.0`**.

If the `distribution` values do not sum up to 1.0, the remaining distribution will transition to the last defined `transition` state. For example, given distributions of `0.3` and `0.6`, the _effective_ distribution of the last transition will actually be `0.7`.

If the `distribution` values sum up to more than `1.0`, the remaining distributions are ignored (for example, if distribution values are `0.75`, `0.5`, and `0.3`, then the second transition will have an _effective_ distribution of `0.25`, and the last transition will have an effective distribution of `0.0`).

### Example

The following example demonstrates a state that should transition to the `"Medication_1"` state 15% of the time, the `"Medication_2"` state 55% of the time, and the `"Medication_3"` state 30% of the time:

```json
"Initial": {
  "type": "Initial",
  "distributed_transition": [
    {
      "distribution": 0.15,
      "transition": "Medication_1"
    },
    {
      "distribution": 0.55,
      "transition": "Medication_2"
    },
    {
      "distribution": 0.30,
      "transition": "Medication_3"
    }
  ]
}
```

### Named Probabilities
For cases where transition probabilities are likely to change based on many different factors, it may be useful to use a "named" transition probability, where the probability of taking any transition is based on the value in an attribute rather than fixed. In this case, instead of a number, the `distribution` on the transition option is an object containing the name of the attribute to look up the transition probability, and a default value for the case where the attribute is not present on the patient.


### Example
```json
"distributed_transition": [
    {
        "distribution": { "attribute" : "probability1", "default" : 0.15 },
        "transition": "Terminal1"
    },
    {
        "distribution": { "attribute" : "probability2", "default" : 0.55 },
        "transition": "Terminal2"
    },
    {
        "distribution": { "attribute" : "probability3", "default" : 0.30 },
        "transition": "Terminal3"
    }
]
```

## Conditional

`Conditional` transitions will transition to one of several possible states based on conditional logic.  A `conditional_transition` consists of an array of `condition`/`transition` pairs which are tested in the order they are defined.  The first condition that evaluates to `true` will result in a transition to its corresponding `transition` state.  The last element in the `condition_transition` array may contain only a `transition` (with no `condition`) to indicate a "fallback transition" when all other conditions are `false`.

If none of the `conditions` evaluated to `true`, and no fallback transition was specified, the module will transition to a default `Terminal` state.

Please see the [[Logic|Generic Module Framework: Logic]] section for more information about creating logical conditions.

### Example

The following example demonstrates a state that should transition to the `"Male_Patient"` state for male patients, the `"Female_Patient` state for female patients, and the `"Unknown_Gender"` state for patients with an unrecognized gender value.

```json
"Initial": {
  "type": "Initial",
  "conditional_transition": [
    {
      "condition": {
        "condition_type": "Gender",
        "gender": "M" 
      },
      "transition": "Male_Patient"
    },
    {
      "condition": {
        "condition_type": "Gender",
        "gender": "F" 
      },
      "transition": "Female_Patient"
    },
    {
      "transition": "Unknown_Gender"
    }
  ]
}
```


## Complex

`Complex` transitions are a combination of direct, distributed, and conditional transitions.  A `complex_transition` consists of an array of condition/transition pairs which are tested in the order they are defined.  The first condition that evaluates to `true` will result in a transition based on its corresponding `transition` or `distributions`.  If the module defines a `transition`, it will transition directly to that named state. If the module defines `distributions`, it will then transition to one of these according to the same rules as the `distributed_transition`. See [Distributed](#distributed) for more detail. The last element in the `complex_transition` array may omit the `condition` to indicate a fallback transition when all other conditions are `false`.

If none of the `conditions` evaluated to `true`, and no fallback transition was specified, the module will transition to the last defined transition.

Please see the [[Logic|Generic Module Framework: Logic]] section for more information about creating logical conditions.

**Example**

The following example demonstrates a state that for male patients should transition to the `"Male_Medication_1"` state with 15% probability and the `"Male_Medication_2"` state with 85% probability, and for female patients should transition directly to the `"Female_Medication"` state.

``` json
"Initial": {
  "type": "Initial",
  "complex_transition": [
    {
      "condition": {
        "condition_type": "Gender",
        "gender": "M" 
      },
      "distributions": [
        {
          "distribution": 0.15,
          "transition": "Male_Medication_1"
        },
        {
          "distribution": 0.85,
          "transition": "Male_Medication_2"
        }
      ]
    },
    {
    "condition": {
      "condition_type": "Gender",
      "gender": "F" 
    },
    "transition": "Female_Medication"
    }
  ]
}
```

## Table

Table-based transitions are used for probabilities that vary widely between different segments or cohorts of the population. When the probability of an event occurring is based on any combination of patient [attributes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#attributes) (e.g. `race`, `ethnicity`, `gender`, `age`, `smoker` status, or any other attribute in the system) you can use a `lookup_table_transition`.

A `lookup_table_transition` is defined as a JSON array that contains one or more transitions, each with a `transition`, a `default_probability`, and the `lookup_table_name` to use to lookup transition probabilities.

The `transition` is the name of the next State to transition into, and the `default_probability` is a number (`0.0 - 1.0`) that represents the default probability of using this transition in the case that the lookup table contains no match.

The lookup table itself should have at least `N + M` columns, where `M` is the number of transitions. The last `M` columns must have headers that match the transition names (i.e. declared in the JSON) and the values in those columns must be numeric probabilities between `0.0 - 1.0`. The first `N` columns can be any subset of patient attributes.

### Example

This example consists of both JSON which defines the table-based transition in the module, and a comma-separated value (CSV) file that defines the lookup table.

```json
"Determine_Condition": {
  "type": "Simple",
  "name": "Determine_Condition",
  "lookup_table_transition": [
    {
      "transition": "Lookuptablitis",
      "default_probability": "0",
      "lookup_table_name": "lookuptablitis.csv"
    },
    {
      "transition": "No_Lookuptablitis",
      "default_probability": "1",
      "lookup_table_name": "lookuptablitis.csv"
    }
  ]
}
```

The contents of `lookuptablitis.csv` are below:

```csv
age,gender,state,Lookuptablitis,No_Lookuptablitis
0-17,M,Massachusetts,0,1
18-44,M,Massachusetts,0.25,0.75
45-64,M,Massachusetts,0.5,0.5
64-140,M,Massachusetts,0.75,0.25
0-17,F,Massachusetts,0,1
18-44,F,Massachusetts,0.3,0.7
45-64,F,Massachusetts,0.8,0.2
64-140,F,Massachusetts,0,1
...
```