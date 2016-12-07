## Generic Modules

## States

States are the core of a Generic Module Framework. Modules typically have many states. Common states includes `Encounter`, `ConditionOnset`, `Procedure`, and `MedicationOrder`. While `states` can have any string as a name, the best practice is to use `Upper_Snake_Case` for state names. **State names must be unique within the same module** but may conflict across modules.

All states must minimally have a `type` identifying the type of state.

#### Example

```json
"Hello_World": {
  "type": "Simple"
}
```

## Attributes

Attributes allow modules to store, reference, and share medications, conditions, procedures, etc. as well as as arbitrary strings. Attributes are generic containers that you can assign states, strings, and numeric values to for easy access and reference. Attributes can be assigned using `assign_to_attribute` or using a `SetAttribute` state. A special kind of Attribute is a `Counter`, defined using the `Counter` state.

While the name of an `attribute` can be any string, the best practice is to use `lower_snake_case` for attribute names. **Attribute names must be unique across all modules**. 

#### Examples

Good:

```json
{
  "assign_to_attribute": "opioid_prescription"
}
```

Bad:

```json
{
  "assign_to_attribute": "Type of Alzheimer's"
}
```

## Units

### Units of Time

Valid units of time are `"years"`, `"months"`, `"weeks"`, `"days"`, `"hours"`, `"minutes"`, and `"seconds"`.

#### Example

```json
{
  "quantity": 4,
  "unit": "hours"
}
```

### Units of Age

Valid units of age are `"years"`, `"months"`, `"weeks"`, `"days"`, `"hours"`, `"minutes"`, and `"seconds"`.

#### Example

```json
{
  "quantity": 40,
  "unit": "years"
}
```

### Units of Measurement

Any string is a valid unit of measurement. Metric units are typically used.

#### Example

```json
{
  "unit": "cm"
}
```

## SNOMED Codes

[SNOMED Clinical Terms](https://en.wikipedia.org/wiki/Systematized_Nomenclature_of_Medicine) (SNOMED-CT) describe clinical findings, symptoms, diagnoses, procedures, body structures, organisms and other etiologies, substances, pharmaceuticals, devices and specimens. In the Generic Module Framework these codes are used to describe `Encounter`, `Procedure`, `ConditionOnset`, and `CarePlanStart` states.

For a searchable directory of valid SNOMED codes see the U.K. National Pathology Exhange [SNOMED CT Browser](http://www.snomedbrowser.com/).

#### Example

```json
{
  "system": "SNOMED-CT",
  "code": "44054006",
  "display": "Diabetes"
}
```

## RxNorm Codes

[RxNorm](https://en.wikipedia.org/wiki/RxNorm) codes describe prescriptions of medication. They include dosage information and common Brands. In the Generic Module Framework these codes are used only for `MedicationOrder` states.

For a searchable directory of RxNorm codes see the U.S. National Library of Medicine [RxNorm Browser](https://mor.nlm.nih.gov/RxNav/).

#### Example

```json
{
  "system": "RxNorm",
  "code": "575971",
  "display": "oxaliplatin 5 MG/ML [Eloxatin]"
}
```

## LOINC Codes

[LOINC]() codes describe tests, measurements, and observations. In the Generic Module Framework these codes are used only for `Observation` states.

For a searchable directory of LOINC codes see the [LOINC Browser](https://search.loinc.org/).

#### Example

```json
{
  "system": "LOINC",
  "code": "33756-8",
  "display": "Polyp size greatest dimension by CAP cancer protocols"
}
```

## A Basic Module

A new module must minimally have a `name`, and a set of `states` that comprise the module. Every module must have an `"Initial"` state. Optionally, `remarks` may be used to add comments throughout the module.

```json
{
  "name": "The Name of the Module",
  "remarks": [
    "Additional comments about the module, ",
    "perhaps on multiple lines. Links to sources ",
    "also go here."
  ],
  "states": {
    
    "Initial": {
      "type": Initial,
      "remarks": [
        "Every module needs an initial state. Most ",
        "modules should also have a Terminal state."
      ]
    },
    
    "Terminal": {
      "type": "Terminal"
    }
  }
}
```

## Remarks

Remarks are comments added GMF module files. All `remarks` sections are ignored by Synthea and are used for references, calculations, and details about part of the module. Remarks should always be a `list[]` of `strings`, for example:

```json
{
  "remarks": [
    "Remarks describe their parent object or detail important ",
    "aspects of the module. Links to sources may be included here: ",
    "http://www.example.com/source/ ",
    
    "You may write use extra newlines to separate out remarks."
  ]
}
```



