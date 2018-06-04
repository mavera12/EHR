# Setup Synthea<sup>TM</sup>
## Prerequisites
 - Git
 - Java 1.8 (select JDK, not JRE install)

To copy the repository locally and install the necessary dependencies, open a terminal window and run the following commands:

```
git clone https://github.com/synthetichealth/synthea.git
cd synthea
./gradlew build check test
```
**Note: if running on Windows, use `.\gradlew.bat` instead of `./gradle` -- this guide uses `./gradle` for brevity**

# Running Synthea<sup>TM</sup>

The primary entry point of Synthea is the `run` gradle task, which can be run from the terminal in the `synthea` folder by running the following command.

```
./gradlew run
```

Alternatively, you may run the provided `run_synthea` script which makes it easier to passing command-line arguments to the simulation:

```
./run_synthea
```
**Note: if running on Windows, use `.\run_synthea.bat` instead of `./run_synthea` -- this guide uses `./run_synthea` for brevity**

When you run this command, you should see output similar to the following:

```
example@hostname ~/synthea $ ./run_synthea

> Task :run
Loading C:\Users\example\synthea\build\resources\main\modules\allergic_rhinitis.json
Loading C:\Users\example\synthea\build\resources\main\modules\allergies\allergy_incidence.json
[... many more lines of Loading ...]
Loading C:\Users\example\synthea\build\resources\main\modules\wellness_encounters.json
Loaded 68 modules.
Running with options:
Population: 1
Seed: 1519063214833
Location: Massachusetts

1 -- Jerilyn993 Parker433 (10 y/o) Lawrence, Massachusetts
```

This command takes additional parameters to specify different regions or common run options. Any options not specified are left at the default value.

```
run_synthea [-s seed] 
            [-p populationSize]
            [-g gender]
            [-a minAge-maxAge]
            [state [city]]
```

Some examples:

 -   `run_synthea Massachusetts` -- to generate a population in all cities and towns in Massachusetts
 -   `run_synthea Alaska Juneau` -- to generate a population in only Juneau, Alaska
 -   `run_synthea -s 12345` -- to generate a population using seed 12345. Populations generated with the same seed and the same version of Synthea should be identical
 -   `run_synthea -p 1000` -- to generate a population of 1000 patients
 -   `run_synthea -a 30-40` -- to generate a population of 30 to 40 year olds
 -   `run_synthea -g F` -- to generate only female patients
 -   `run_synthea -s 987 Washington Seattle` -- to generate a population in only Seattle, Washington, using seed 987
 -   `run_synthea -s 21 -p 100 Utah "Salt Lake City"` -- to generate a population of 100 patients in Salt Lake City, Utah, using seed 21

# Configuring Synthea<sup>TM</sup>

Many features of Synthea are configurable using the settings file at `./src/main/resources/synthea.properties`.

## Summary of Commonly-Used Configuration Options

### Export Formats

| Setting Name | Valid Values | Default | Description |
|:-------------|:-------------|:--------|:------------|
| `exporter.ccda.export` | `true`/`false` | `false` | Change this setting to `true` to enableexporting patients in CCDA format. |
| `exporter.fhir.export` | `true`/`false` | `true` | Change this setting to `false` to disable exporting patients in FHIR STU3 format. |
| `exporter.fhir_dstu2.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patients in FHIR DSTU2 format. |
| `exporter.hospital.fhir.export` | `true`/`false` | `true` | Change this setting to `false` to disable exporting hospital information in FHIR STU3 format. |
| `exporter.hospital.fhir_dstu2.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting hospital information in FHIR DSTU2 format. |
| `exporter.text.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patients in a simple text-based format. |
| `exporter.csv.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patient data in a comma-separated value format. See the [CSV File Data Dictionary](https://github.com/synthetichealth/synthea/wiki/CSV-File-Data-Dictionary). |


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

Synthea<sup>TM</sup> contains a number of configurable options relating to the world, human vital signs, risk levels, etc. These have been pre-seeded with values based on US and Massachusetts data, with citations for these values where possible. Users generally will not want to modify these unless trying to simulate a specific result or trying to adapt Synthea<sup>TM</sup> to another geographic or geopolitical location (in which case they should see the series of wiki pages under `Other Areas` starting with [Demographics for Other Areas](https://github.com/synthetichealth/synthea/wiki/Demographics-for-Other-Areas).
