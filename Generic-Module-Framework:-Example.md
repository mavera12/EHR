The [synthea_graphviz](https://github.com/synthetichealth/synthea_graphviz) tool will generate diagrams for all generic modules found in _lib/generic/modules/_.  The following is an example diagram representing a complete module for an artificial 'example' disease: Examplitis.

![Examplitis Diagram](https://github.com/synthetichealth/synthea/raw/master/test/fixtures/generic/example_module_diagram.png)

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