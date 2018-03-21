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

# CarePlans

# Conditions

# Encounters

# Immunizations

# Medications

# Observations

# Patients

# Procedures

