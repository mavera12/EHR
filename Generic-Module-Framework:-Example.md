This example contains a [diagram](#examplitis-diagram) with complete [JSON](#examplitis-json) and a walk-through. The `graphviz` gradle task will generate diagrams for all generic modules found in `src/main/resources/modules/`. Alternatively, modules can be edited using the online [Module Builder](https://synthetichealth.github.io/module-builder/) tool. The following is an example diagram representing a complete module for an artificial example disease: Examplitis.

## Examplitis Diagram
![Example Diagram](https://raw.githubusercontent.com/synthetichealth/synthea/ruby/test/fixtures/generic/example_module_diagram.png)

## Examplitis JSON
The above diagram was generated based on the following JSON module file:

```json
{
  "name": "Examplitis",
  "remarks": [
    "Examplitis is a painful condition that affects only males. Most patients ",
    "can be cured with Examplitol or an Examplotomy but some never recover."
  ],
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
      "codes": [
        {
          "system": "SNOMED-CT",
          "code": "123",
          "display": "Examplitis"
        }
      ],
      "direct_transition": "Wellness_Encounter"
    },

    "Wellness_Encounter": {
      "type": "Encounter",
      "wellness": true,
      "direct_transition": "Examplitol"
    },

    "Examplitol": {
      "type": "MedicationOrder",
      "reason": "Examplitis",
      "codes": [
        {
          "system": "RxNorm",
          "code": "456",
          "display": "Examplitol"
        }
      ],
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
      "encounter_class": "ambulatory",
      "codes": [
        {
          "system": "SNOMED-CT",
          "code": "ABC",
          "display": "Examplotomy Encounter"
        }
      ],
      "direct_transition": "Examplotomy"
    },

    "Examplotomy": {
      "type": "Procedure",
      "duration": {
        "low": 2,
        "high": 3,
        "unit": "hours"
      },
      "codes": [
        {
          "system": "SNOMED-CT",
          "code": "789",
          "display": "Examplotomy"
        }
      ],
      "reason": "Examplitis",
      "direct_transition": "End_Examplotomy_Encounter"
    },

    "End_Examplotomy_Encounter": {
    	"type": "EncounterEnd",
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

## Examplitis Walk-Through

Every patient starts at the [Initial](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#initial) state at birth. As soon as the Initial state is processed, each patient will conditionally transition to either the `Age Guard` (if they are male) or [Terminal](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#terminal) state (if they are female) depending on their gender. Female patients who transition to the [Terminal](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#terminal) state will exit this module; other modules will continue to execute for those patients.

In the meantime, the [Guard](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#guard) state named `Age Guard` will stop the male patients from proceeding any further until the specified condition is met. In this example, the condition is `Allow if age > 40 years`. While this module waits until the patients are over 40 years old, the other modules in the system will continue to execute. Once the men reach their 40th birthday, they have a 10% chance of developing Examplitis (where 10% reflects their approximate lifetime risk). Therefore, 10% of male patients will transition immediately to the `Pre_Examplitis` state, whereas the remaining 90% of patients will transition directly to the [Terminal](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#terminal) state, at which point their progression of the Examplitis module is complete.

The patients in the `Pre_Examplitis` state are in a [Delay](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#delay) state, which indicates that the module progression will pause until a certain amount of time has passed. In this case, the state will pause for a random amount of time between 0 and 10 years. Once the amount of time has passed, progression will continue and the patient will transition to the `Examplitis` state.

The `Examplitis` state is a [ConditionOnset](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#conditiononset) state, which indicates the point at which the condition first onsets for the patient. The [ConditionOnset](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#conditiononset) includes the SNOMED code for the condition which will be added to the patient's record once the condition is diagnosed. From there, patients will transition directly to the `Wellness_Encounter` state, an [Encounter](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#encounter) where the condition is diagnosed.

An [Encounter](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#encounter) state indicates an interaction between patient and healthcare provider. As the patient progresses through subsequent states, the system keeps a reference to the current Encounter the patient is in, to associate to subsequent events. The patient then is diagnosed with the "Examplitis" condition for future reference, then transitions to the `Examplitol` state, which is a [MedicationOrder](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#medicationorder) for a fictional medication.

At this point, there are a few possibilities:

* 70% chance the patient is cured or in some kind of remission (transition to [Terminal](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#terminal) state)
* 10% chance the patient will be a candidate for surgery (transition to the `Pre_Examplotomy` [Delay](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#delay) state)
* 20% chance the patient will die (transition to the `Last_Days` [Delay](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#delay) state)

In the case of the `Last_Days` state, the patient will [Delay](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#delay) 8 to 20 years before succumbing to [Death](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#death).

Other patients who are candidates for surgery, will [Delay](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#delay) in the `Pre_Examplotomy` state for 18 to 36 months before having an `Examplotomy_Encounter`.

During the `Examplotomy_Encounter` patients will have an `Examplotomy` [Procedure](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#procedure) performed that has a duration of 2 to 3 hours. Afterwards, the encounter is explicitly completed with the `End_Examplotomy_Encounter` [EncounterEnd](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-States#encounterend) state. Sadly, the `Examplotomy` is a risky procedure, and 10% of those undertaking the surgery die under the scalpel. The remaining 90% transition to the Terminal state, and the module progression is complete.