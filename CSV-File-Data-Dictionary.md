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
| | Cost | Numeric | `true` | The base cost of the encounter, **not** including any line item costs related to medications, immunizations, procedures, or other services. |
| | ReasonCode | String | `false` | Diagnosis code from SNOMED-CT, **only if** this encounter targeted a specific condition. |
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
| | Column Name | Data Type | Required? | Description |
|-|-------------|-----------|-----------|-------------|
| | Start | Date (`YYYY-MM-DD`) | `true` | The date the medication was prescribed. |
| | Stop | Date (`YYYY-MM-DD`) | `false` | The date the prescription ended, if applicable. |
| :old_key: | Patient | UUID | `true` | Foreign key to the Patient. |
| :old_key: | Encounter | UUID | `true` | Foreign key to the Encounter where the medication was prescribed. 
| | Code | String | `true` | Medication code from RxNorm. |
| | Description | String | `true` | Description of the medication. |
| | Cost | Numeric | `true` | The line item cost of the medication. |
| | ReasonCode | String | `false` | Diagnosis code from SNOMED-CT specifying why this medication was prescribed. |
| | ReasonDescription | String | `false` | Description of the reason code. |

# Observations
| | Column Name | Data Type | Required? | Description |
|-|-------------|-----------|-----------|-------------|
| | Date | Date (`YYYY-MM-DD`) | `true` | The date the observation was performed. |
| :old_key: | Patient | UUID | `true` | Foreign key to the Patient. |
| :old_key: | Encounter | UUID | `true` | Foreign key to the Encounter where the observation was performed. 
| | Code | String | `true` | Observation or Lab code from LOINC |
| | Description | String | `true` | Description of the observation or lab. |
| | Value | String | `true` | The recorded value of the observation. |
| | Units | String | `false` | The units of measure for the value. |
| | Type | String | `true` | The datatype of `Value`: `text` or `numeric` |

# Patients
| | Column Name | Data Type | Required? | Description |
|-|-------------|-----------|-----------|-------------|
| :key: | ID | UUID | `true` | Primary Key. Unique Identifier of the patient. |
| | BirthDate | Date (`YYYY-MM-DD`) | `true` | The date the patient was born. |
| | DeathDate | Date (`YYYY-MM-DD`) | `false` | The date the patient died. |
| | SSN | String | `true` | Patient Social Security identifier. |
| | Drivers | String | `false` | Patient Drivers License identifier. |
| | Passport | String | `false` | Patient Passport identifier. |
| | Prefix | String | `false` | Name prefix, such as `Mr.`, `Mrs.`, `Dr.`, etc. |
| | First | String | `true` | First name of the patient. |
| | Last | String | `true` | Last or surname of the patient. |
| | Suffix | String | `false` | Name suffix, such as `PhD`, `MD`, `JD`, etc. |
| | Maiden | String | `false` | Maiden name of the patient. |
| | Marital | String | `false` | Marital Status. `M` is married, `S` is single. Currently no support for divorce (`D`) or widowing (`W`) |
| | Race | String | `true` | Description of the patient's primary race. |
| | Ethnicity | String | `true` | Description of the patient's primary ethnicity. |
| | Gender | String | `true` | Gender. `M` is male, `F` is female. |
| | BirthPlace | String | `true` | Name of the town where the patient was born. |
| | Address | String | `true` | Patient's current full address without commas or newlines. |

# Procedures
| | Column Name | Data Type | Required? | Description |
|-|-------------|-----------|-----------|-------------|
| | Date | Date (`YYYY-MM-DD`) | `true` | The date the procedure was performed. |
| :old_key: | Patient | UUID | `true` | Foreign key to the Patient. |
| :old_key: | Encounter | UUID | `true` | Foreign key to the Encounter where the procedure was performed. 
| | Code | String | `true` | Procedure code from SNOMED-CT |
| | Description | String | `true` | Description of the procedure. |
| | Cost | Numeric | `true` | The line item cost of the procedure. |
| | ReasonCode | String | `false` | Diagnosis code from SNOMED-CT specifying why this procedure was performed. |
| | ReasonDescription | String | `false` | Description of the reason code. |
