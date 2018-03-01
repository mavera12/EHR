
The generic module framework currently supports the following transitions:

* [Direct](#direct)
* [Distributed](#distributed)
* [Conditional](#conditional)
* [Complex](#complex)

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

If none of the `conditions` evaluated to `true`, and no fallback transition was specified, the module will transition to a default `Terminal` state.

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
