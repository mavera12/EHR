With Synthea, it is possible to export the symptoms a person has experienced for the different conditions he suffered from during his life cycle.
At a high level, the export will contain the following information for a given condition:
- the `age` (in years) at which the condition occurred (Note that this information is different from the time the condition was diagnosed)
- the `age` (in years) at which the condition was resolved, if any
- the number of symptoms expressed
- the name of the different symptoms as well as their severity level along the time period the condition was on.

It is possible to restrict the export to conditions that occurred during the last `{exporter.years_of_history}` years of the person. This is done by setting `exporter.symptoms.mode = 0` (default one). If `exporter.symptoms.mode` is set to any other value, then the export will consider all conditions that occurred during the lifetime of the person.

The export can be made in two formats:
- Text
- CSV

## Text
You can enable text records in `src/main/resources/synthea.properties` by setting `exporter.symptoms.text.export = true`. The Symptom Text Exporter will output, for each patient, a UTF-8 text file that lists a human-readable text file containing the patient related information. The generated files are saved into the following directory: `./{export.baseDirectory}/symptoms/text`.

## CSV
It is possible to enable Comma-Separated Value (CSV) exports in `src/main/resources/synthea.properties` by setting `exporter.symptoms.csv.export = true`. The generated files are saved into the following directory: `./{export.baseDirectory}/symptoms/csv`.

Furthermore, there are additional parameters that dictates how the export will behave across several runs:
- `exporter.symptoms.csv.append_mode`: if set to `true`, then each run will add new data to any existing CSVs. On the other hand, if set to `false`, each run will clear out the files and start fresh.
- `exporter.symptoms.csv.folder_per_run`: if set to `true`, then each run will have CSVs placed into a unique subfolder. On the other hand, if set to `false`, each run will only use the top-level csv folder.

The structure of a generated csv file is as follows:
| | Column Name | Data Type | Required? | Description |
|-|-------------|-----------|-----------|-------------|
| | PATIENT | UUID | `true` | key to the Patient. |
| | GENDER | String | `true` | Gender. `M` is male, `F` is female. |
| | RACE | String | `true` | Description of the patient's primary race. |
| | ETHNICITY | String | `true` | Description of the patient's primary ethnicity. |
| | AGE_BEGIN | Integer | `true` | The age at which the condition was onset. |
| | AGE_END | Integer | `false` | The age at which the condition was resolved (if any). |
| | PATHOLOGY | String | `true` | The name of the condition |
| | NUM_SYMPTOMS | Integer | `true` | The number of symptoms. |
| | SYMPTOMS | String | `false` | the symptoms (if any) and their severity level. |

