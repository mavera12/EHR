Synthea contains a framework for defining modules using JSON.  A JSON module configuration describes a progression of states and the transitions between them.  On each Synthea generation "cycle", the generic framework processes states one at a time to trigger conditions, encounter, medications, and other clinical events.

# States

The generic module framework currently supports the following states:

* Initial
* Terminal
* Guard
* Delay
* Encounter
* ConditionOnset
* MedicationOrder
* Procedure
* Death

The following states are also planned for future implementation:

* Lab
* ConditionEnd
* MedicationEnd

## Initial

The `Initial` state type is the first state that is processed in a generic module.  It does not provide any specific function except to indicate the starting point, so it has no properties except its `type`.  The Initial state is the only state that requires a specific name: "Initial".  In addition, it is the only state for which there can only be _one_ in the whole module.

**Supported Properties**

* **type**: must be "Initial"

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

* **type**: must be "Terminal"

**Example**

```json
{
  "type": "Terminal"
}
```

## Guard

The `Guard` state type indicates a point in the module through which a patient can only pass if they meet certain logical conditions.  For example, a Guard may block a workflow until the patient reaches a certain age, after which the Guard allows the module to continue to progress.  Depending on the condition, a patient may be blocked by a Guard until they die -- in which case they never reach a `Terminal` state.  The Guard state's `allow` property provides the logical condition which must be met to allow the module to continue to the next state.  Please see the _Logic_ section for more information about creating logical conditions.

Guard states are similar to conditional transitions in some ways, but also have an important difference.  A conditional transition tests conditions once and uses the result to immediately choose the next state.  A Guard state will test the same condition on every time cycle until the condition passes, at which point it progresses to the next state.

**Supported Properties**

* **type**: must be "Guard"
* **allow**: the condition under which the Guard allows the module to progress to the next state; otherwise the module remains at the Guard state.

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

Synthea generation occurs in time cycles; currently 7-day cycles.  This means that if a module is processed on a given date, the next time it is processed will be exactly 7 days later.  If a delay expiration falls between cycles (e.g., day 3 of a 7-day cycle), then the first cycle _after_ the delay expiration will effectively _rewind_ the clock to the delay expiration time and process states using that time.  Once it reaches a state that it can't pass through, it will process it once more using the original (7-day cycle) time.

**Supported Properties**

* **type**: must be "Delay"
* **exact**: an exact amount of time to delay
  * **quantity**: the number of _units_ to delay (e.g., 4)
  * **unit**: the unit of time pertaining to the _quantity_ (e.g., "days").  Valid _unit_ values are: `years`, `months`, `weeks`, `days`, `hours`, `minutes`, and `seconds`.
* **range**: a range indicating the allowable amounts of delay.  The actual delay time will be chosen randomly from the range.
  * **low**: the lowest number (inclusive) of _units_ to delay (e.g., 5)
  * **high**: the highest number (inclusive) of _units_ to delay (e.g., 7)
  * **unit**: the unit of time pertaining to the _range_ (e.g., "days").  Valid _unit_ values are: `years`, `months`, `weeks`, `days`, `hours`, `minutes`, and `seconds`.

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

## ConditionOnset

## MedicationOrder

## Procedure

## Death

# Transitions

The generic module framework currently supports the following transitions:

* Direct
* Distributed
* Conditional

# Logic

The Guard state and Conditional transition use conditional (boolean) logic.  The following condition types are currently supported:

* Gender
* Age
* And
* Or
* Not
* True
* False
