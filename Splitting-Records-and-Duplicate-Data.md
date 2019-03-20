In Synthea each patient has a single longitudinal record by default, regardless of how many providers they see.

However, you have the ability to split patient records by providing organization, by altering `synthea.properties`:

```properties
# split records allows patients to have one record per provider organization
exporter.split_records = false
exporter.split_records.duplicate_data = false
```

Setting `exporter.split_records = true` will create more than one file for each patient for formats like FHIR, CCDA, and Text. CDW will split the records (i.e. the patient data won't be merged under a single patient identifier). **By design, the CSV export format is unaffected by this setting**.

Setting `exporter.split_records.duplicate_data = true` will duplicate data across the split records for FHIR, CCDA, and Text. For software using this data, you can consider this duplicated data or actually repeated tests/procedures/medications, as you prefer. 
