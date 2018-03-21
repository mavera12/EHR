Synthea can be configured to export data as CSV into `./output/csv`.

To export CSV, edit `./src/main/resources/synthea.properties` and change this setting:

```
exporter.csv.export = true
```

After running Synthea, the CSV exporter will create nine files:

| File | Description |
|------|-------------|
| [`allergies.csv`](#Allergies) | Patient allergy data. |
| `careplans.csv` | Patient care plan data, including goals. |
| `conditions.csv` | Patient conditions or diagnoses. |
| `encounters.csv` | Patient encounter data. |
| `immunizations.csv` | Patient immunization data. |
| `medications.csv` | Patient medication data. |
| `observations.csv` | Patient observations including vital signs and lab reports. |
| `patient.csv` | Patient demographic data. |
| `procedures.csv` | Patient procedure data including surgeries. |

Data Dictionary information for each CSV table follows below.

# Allergies

# CarePlans

# Conditions

# Encounters

# Immunizations

# Medications

# Observations

# Patients

# Procedures

