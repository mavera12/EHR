Synthea can be configured to export data as CSV into `./output/csv`.

To export CSV, edit `./src/main/resources/synthea.properties` and change this setting:

```
exporter.csv.export = true
```

After running Synthea, the CSV exporter will create nine files:

| File | Description |
|------|-------------|
| [`allergies.csv`](#allergies) | Patient allergy data. |
| [`careplans.csv`](#careplans) | Patient care plan data, including goals. |
| [`conditions.csv`](#conditions) | Patient conditions or diagnoses. |
| [`encounters.csv`](#encounters) | Patient encounter data. |
| [`immunizations.csv`](#immunizations) | Patient immunization data. |
| [`medications.csv`](#medications) | Patient medication data. |
| [`observations.csv`](#observations) | Patient observations including vital signs and lab reports. |
| [`patients.csv`](#patients) | Patient demographic data. |
| [`procedures.csv`](#procedures) | Patient procedure data including surgeries. |

Data Dictionary information for each CSV table follows below.

# Allergies
| | Column Name | Data Type | Required? | Description |
|-|-------------|-----------|-----------|-------------|
| | Start | Date (`YYYY-MM-DD`) | `true` | The date the allergy was diagnosed. |
| | Stop | Date (`YYYY-MM-DD`) | `false` | The date the allergy ended, if applicable. |
| :old_key: | Patient | UUID | `true` | Foreign key to the Patient. |
| :old_key: | Encounter | UUID | `true` | Foreign key to the Encounter when the allergy was diagnosed. |
| | Code | String | `true` | Allergy code from SNOMED-CT |
| | Description | String | `true` | Description of the Allergy |

# CarePlans
| | Column Name | Data Type | Required? | Description |
|-|-------------|-----------|-----------|-------------|
| :key: | ID | UUID | `true` | Primary Key. Unique Identifier of the care plan. |
| | Start | Date (`YYYY-MM-DD`) | `true` | The date the care plan was initiated. |
| | Stop | Date (`YYYY-MM-DD`) | `false` | The date the care plan ended, if applicable. |
| :old_key: | Patient | UUID | `true` | Foreign key to the Patient. |
| :old_key: | Encounter | UUID | `true` | Foreign key to the Encounter when the care plan was initiated. |
| | Code | String | `true` | Code from SNOMED-CT |
| | Description | String | `true` | Description of the care plan. |
| | ReasonCode | String | `true` | Diagnosis code from SNOMED-CT that this care plan addresses. |
| | ReasonDescription | String | `true` | Description of the reason code. |

# Conditions
| | Column Name | Data Type | Required? | Description |
|-|-------------|-----------|-----------|-------------|
| | Start | Date (`YYYY-MM-DD`) | `true` | The date the condition was diagnosed. |
| | Stop | Date (`YYYY-MM-DD`) | `false` | The date the condition resolved, if applicable. |
| :old_key: | Patient | UUID | `true` | Foreign key to the Patient. |
| :old_key: | Encounter | UUID | `true` | Foreign key to the Encounter when the condition was diagnosed. |
| | Code | String | `true` | Diagnosis code from SNOMED-CT |
| | Description | String | `true` | Description of the condition. |

# Encounters
| | Column Name | Data Type | Required? | Description |
|-|-------------|-----------|-----------|-------------|
| :key: | ID | UUID | `true` | Primary Key. Unique Identifier of the encounter. |
| | Date | Date (`YYYY-MM-DD`) | `true` | The date the encounter took place. |
| :old_key: | Patient | UUID | `true` | Foreign key to the Patient. |
| | Code | String | `true` | Encounter code from SNOMED-CT |
| | Description | String | `true` | Description of the type of encounter. |
| | Cost | Numeric | `true` | The base cost of the encounter, **not** including any costs related to medications, immunizations, procedures, or other services. |
| | ReasonCode | String | `false` | Diagnosis code from SNOMED-CT, **if** this encounter targeted a specific condition. |
| | ReasonDescription | String | `false` | Description of the reason code. |

# Immunizations
| | Column Name | Data Type | Required? | Description |
|-|-------------|-----------|-----------|-------------|
| | Date | Date (`YYYY-MM-DD`) | `true` | The date the immunization was administered. |
| :old_key: | Patient | UUID | `true` | Foreign key to the Patient. |
| :old_key: | Encounter | UUID | `true` | Foreign key to the Encounter where the immunization was administered. 
| | Code | String | `true` | Immunization code from CVX. |
| | Description | String | `true` | Description of the immunization. |
| | Cost | Numeric | `true` | The line item cost of the immunization. |

# Medications

# Observations

# Patients

# Procedures