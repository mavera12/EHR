
The generic module framework currently supports the following transitions: [Direct](#direct), [Distributed](#distributed), [Conditional](#conditional), and [Complex](#complex).

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

Please see the [[Logic|Generic Module Framework: Logic]] section for more information about creating logical conditions.

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


## Complex

`Complex` transitions are a combination of direct, distributed, and conditional transitions.  A `complex_transition` consists of an array of condition/transition pairs which are tested in the order they are defined.  The first condition that evaluates to `true` will result in a transition based on its corresponding `transition` or `distributions`.  If the module defines a `transition`, it will transition directly to that named state. If the module defines `distributions`, it will then transition to one of these according to the same rules as the `distributed_transition`. See [Distributed](#distributed) for more detail. The last element in the `complex_transition` array may omit the `condition` to indicate a fallback transition when all other conditions are `false`.

If none of the `conditions` evaluated to `true`, and no fallback transition was specified, the module will transition to a default `Terminal` state.

Please see the [[Logic|Generic Module Framework: Logic]] section for more information about creating logical conditions.

**Example**

The following example demonstrates a state that for male patients should transition to the "Foo" state with 15% probability and the "Bar" state with 85% probability, and for female patients should transition directly to the "Baz" state.

``` json
{
    "type": "Initial",
    "complex_transition": [
        {
            "condition": {
                "condition_type": "Gender",
                "gender": "M" 
            },
            "distributions" : [
                {
                    "distribution": 0.15,
                    "transition": "Foo"
                },
                {
                    "distribution": 0.85,
                    "transition": "Bar"
                }
            ]
        },
        {
            "condition": {
                "condition_type": "Gender",
                "gender": "F" 
            },
            "transition" : "Baz"
        }
    ]
}
```
