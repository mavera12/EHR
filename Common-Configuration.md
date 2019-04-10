Many features of Synthea are configurable using the settings file at `./src/main/resources/synthea.properties`.

## Summary of Commonly-Used Configuration Options

### Export Formats

| Setting Name | Valid Values | Default | Description |
|:-------------|:-------------|:--------|:------------|
| `exporter.ccda.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patients in CCDA format. |
| `exporter.fhir.export` | `true`/`false` | `true` | Change this setting to `false` to disable exporting patients in FHIR R4 format. |
| `exporter.fhir.use_us_core_ig` | `true`/`false` | `false` | US Core Implementation Guide for FHIR R4 |
| `exporter.fhir_stu3.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patients in FHIR STU3 format. |
| `exporter.fhir_dstu2.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patients in FHIR DSTU2 format. |
| `exporter.hospital.fhir.export` | `true`/`false` | `true` | Change this setting to `false` to disable exporting hospital information in FHIR R4 format. |
| `exporter.hospital.fhir_stu3.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting hospital information in FHIR STU3 format. |
| `exporter.hospital.fhir_dstu2.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting hospital information in FHIR DSTU2 format. |
| `exporter.text.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patients in a simple text-based format. |
| `exporter.csv.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patient data in a comma-separated value format. See the [CSV File Data Dictionary](https://github.com/synthetichealth/synthea/wiki/CSV-File-Data-Dictionary). |
| `exporter.csv.append_mode` | `true`/`false` | `false` | Change this setting to `true` to enable append mode for the CSV exporter, where all new runs will be appended onto the existing CSV records, rather than overwriting the files.  For instance, this could be used to help produce a single set of CSVs spanning patients across multiple states, since only one state can be run at a time.|
| `exporter.csv.folder_per_run` | `true`/`false` | `false` | Change this setting to `true` to enable exporting CSVs in a separate subfolder per run of Synthea. Currently the CSVs get reset each time, so if you want to keep a run you have to specifically copy it off somewhere else. This option makes CSVs more like other export formats in that files will persist until manually deleted. Note that this setting will supersede `exporter.csv.append_mode` if both are true|


### Other Export Settings

| Setting Name | Valid Values | Default | Description |
|:-------------|:-------------|:--------|:------------|
| `exporter.years_of_history` | Whole number | `10` | The number of years of patient history to include in patient records. For example, if set to `5`, then all patient history older than 5 years old (from the time you execute the program) will not be included in the exported records. Note that conditions and medications that are currently active will still be exported, regardless of this setting. Set this to `0` to keep all history in the patient record. |
| `exporter.baseDirectory` | Folder paths | `./output/` | This is the base folder in which all patient records will be exported. Files will be exported to subfolders based on their type. (For example FHIR records will be stored in the `/fhir/` subfolder under this folder. |
| `exporter.subfolders_by_id_substring` | `true`/`false` | `false` | If true, patient records will be grouped into subfolders based on their UUID, which is a randomly generated unique identifer. This reduces the number of files in any single folder, and ensures a roughly even distribution of files among folders. However there is no correlation between files in any given folder as the IDs are random. |
| `synthea.exporter.use_uuid_filenames` | `true`/`false` | `false` | If true, patient records will have filenames based only on their UUID. If false, patient records will have filenames including the patient's name. This is mostly intended as a debugging feature - if watching the terminal as patients are generating, this makes it easier to find the record once it is generated. |


### Generation Settings

| Setting Name | Valid Values | Default | Description |
|:-------------|:-------------|:--------|:------------|
| `generate.default_population` | Whole numbers | `./output/` | This is the number of living patients (population size) that Synthea will generate. Synthea will generate patients until the number of living patients reaches this value. |
| `generate.database_type` | `file`, `in-memory`, or `none` | `in-memory` | Synthea uses a database to track various metrics, which can be used to produce various reports. If these reports are not desired, the database may be disabled to improve runtime performance. Alternatively, if it is desirable to persist this data across runs, use the `file` option to save data to a persistent database file (`./database.mv.db`, which can be accessed using the [h2 database client](https://github.com/h2database/h2database)). But beware of `file` as the size of the database can grow VERY large. |


### Other Settings

Synthea<sup>TM</sup> contains a number of configurable options relating to the world, human vital signs, risk levels, etc. These have been pre-seeded with values based on US and Massachusetts data, with citations for these values where possible. See [Default Demographic Data](https://github.com/synthetichealth/synthea/wiki/Default-Demographic-Data). Users generally will not want to modify these unless trying to simulate a specific result or trying to adapt Synthea<sup>TM</sup> to another geographic or geopolitical location (in which case they should see the series of wiki pages under `Other Areas` starting with [Demographics for Other Areas](https://github.com/synthetichealth/synthea/wiki/Demographics-for-Other-Areas).