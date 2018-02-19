# Introduction
## Welcome to Synthea<sup>TM</sup>!
Thank you for your interest in Synthea<sup>TM</sup>. 

Synthea<sup>TM</sup> is a synthetic patient generator that models the medical history of synthetic patients. Our mission is to output high-quality synthetic, realistic but not real, patient data and associated health records covering every aspect of healthcare. The resulting data is free from cost, privacy, and security restrictions. It can be used without restriction for a variety of secondary uses in academia, research, industry, and government (although a citation would be appreciated).

## How does Synthea<sup>TM</sup> work?
Synthea<sup>TM</sup> generates synthetic patient records using an agent-based approach. Each synthetic patient is generated independently, as they progress from birth to death through modular representations of various diseases and conditions. Each patient runs through every module in the system. Once a patient dies or the simulation reaches the current day, that patient record can be exported in a number of different formats.

![](https://raw.githubusercontent.com/synthetichealth/synthea/gh-pages/images/architecture.png)

### Disease Modules
The Synthea<sup>TM</sup> [Generic Module Framework](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework) allows for the creation of state machines representing the progression and standards of care for common diseases, using a set of predefined states, transitions, and conditional logic. Modules are built based on publicly available health data, including disease incidence and prevalence statistics, and clinical practice guidelines. The disease modules represent a Monte Carlo simulation, where events occur depending with varying frequency based on repeated weighted random sampling.

As an example, consider the Appendicitis module:

![](https://synthetichealth.github.io/synthea/graphviz/Appendicitis.png)

Every patient starts at the **Initial** state at birth. As soon as the Initial state is processed, each patient will conditionally transition to either the **Male** or **Female** state depending on their gender. Males have an approximate lifetime risk of appendicitis of 8.6%, and females have an approximate lifetime risk of 6.7%. This percent of patients will then transition immediately to the **Pre\_appendicitis** state, whereas the remaining 90+% of patients will transition directly to the **Terminal** state, at which point their progression of the Appendicitis module is complete.

The patients in the Pre\_appendicitis state will transition randomly to one of four states, representing the age distribution of appendicitis:

* 26.3% will transition to **Ages\_1\_17**, representing the  proportion of the population afflicted by appendicitis between ages 1 and 17
* 42.3% will transition to **Ages\_18\_44**
* 22.1% will transition to **Ages\_45\_64**
* The remaining 9.3% will transition to **Ages\_65\_Plus**

These four states are of the **Delay** type, which indicates that the module progression will pause until a certain amount of time has passed. For instance, the Ages\_18\_44 state will pause for a random amount of time between 18 and 44 years. Once the amount of time has passed, progression will continue and the patient will transition to the **Appendicitis** state.

The Appendicitis state is a **ConditionOnset**, which indicates the point at which the condition first onsets for the patient. The ConditionOnset includes the SNOMED code for the condition which will be added to the patient's record once the condition is diagnosed. From there, 30% of patients will transition to **Rupture** where they experience a ruptured appendix and then transition to the **Appendicitis\_Encounter** state, and 70% will transition directly to the Appendicitis\_Encounter.

An **Encounter** state indicates an interaction between patient and healthcare provider. As the patient progresses through subsequent states, the system keeps a reference to the current Encounter the patient is in, to associate to subsequent events. The patient then is diagnosed with a "History of appendectomy" condition for future reference, then transitions to the **Appendectomy\_Encounter** state and then the **Appendectomy** state where an appendectomy is performed. The patient then transitions to a 1-5 day recovery period, at which point they transition to the **End\_Appendectomy\_Encounter** state where the encounter ends and they are discharged. Finally, they transition to the Terminal state, and the module progression is complete.


### Patient Record Formats

#### FHIR

Synthea<sup>TM</sup> currently supports exporting patients as Fast Healthcare Interoperability Resources (FHIR), versions 3.0.1 (STU3) and 1.0.2 (DSTU2). FHIR is a standard created by HL7 for exchanging healthcare information electronically. The philosophy behind FHIR is to build a base set of resources that, either by themselves or when combined, satisfy the majority of common use cases. For more information on FHIR, see [http://hl7.org/fhir/index.html](http://hl7.org/fhir/index.html) While FHIR supports both XML and JSON, Synthea exports FHIR as JSON only. Synthea uses the [HAPI FHIR](http://hapifhir.io/) library to export FHIR.

#### C-CDA
Synthea<sup>TM</sup> uses the [MDHT CDA Tools](http://cdatools.org/) library along with templates from the [health-data-standards](https://github.com/projectcypress/health-data-standards) Ruby gem to export patients as Consolidated Clinical Document Architecture (C-CDA) format. C-CDA is an XML-based standard defined by HL7, that uses templates from a standard library to represent clinical concepts. For more information on C-CDA,  see [http://www.hl7.org/implement/standards/product_brief.cfm?product_id=258](http://www.hl7.org/implement/standards/product_brief.cfm?product_id=258). 


#### Text
For quick, human-readable patient records, Synthea<sup>TM</sup> includes a text-based exporter. This format does not adhere to any standards but is clear and easy for a person to read and understand. 

**Sample:**

```
Mekhi724 Kemmer911
==================
Race:           White
Ethnicity:      Non-Hispanic
Gender:         F
Age:            33
Birth Date:     1983-11-04
Marital Status: M
--------------------------------------------------------------------------------
ALLERGIES: N/A
--------------------------------------------------------------------------------
MEDICATIONS:
2013-08-22 [CURRENT] : Acetaminophen 160 MG for Acute bronchitis (disorder)
1996-05-12 [CURRENT] : Acetaminophen 160 MG for Acute bronchitis (disorder)
1995-04-13 [CURRENT] : Acetaminophen 160 MG for Acute bronchitis (disorder)
1984-01-14 [CURRENT] : Penicillin V Potassium 250 MG for Streptococcal sore throat (disorder)
--------------------------------------------------------------------------------
CONDITIONS:
2015-10-30 - 2015-11-07 : Fetus with chromosomal abnormality
2015-10-30 - 2015-11-07 : Miscarriage in first trimester
2015-10-30 - 2015-11-07 : Normal pregnancy
2013-08-22 - 2013-09-08 : Acute bronchitis (disorder)
1985-08-07 -            : Food Allergy: Fish
--------------------------------------------------------------------------------
CARE PLANS:
2013-08-22 [STOPPED] : Respiratory therapy
                         Reason: Acute bronchitis (disorder)
                         Activity: Recommendation to avoid exercise
                         Activity: Deep breathing and coughing exercises
--------------------------------------------------------------------------------
OBSERVATIONS:
2014-01-14 : Body Weight                              73.9 kg
2014-01-14 : Body Height                              163.7 cm
2014-01-14 : Body Mass Index                          27.6 kg/m2
2014-01-14 : Systolic Blood Pressure                  133.0 mmHg
2014-01-14 : Diastolic Blood Pressure                 76.0 mmHg
2014-01-14 : Blood Pressure                           2.0 
--------------------------------------------------------------------------------
PROCEDURES:
2015-10-30 : Standard pregnancy test for Normal pregnancy
2014-01-14 : Documentation of current medications
--------------------------------------------------------------------------------
ENCOUNTERS:
2015-11-07 : Encounter for Fetus with chromosomal abnormality
2015-10-30 : Encounter for Normal pregnancy
2014-01-14 : Outpatient Encounter
2013-08-22 : Encounter for Acute bronchitis (disorder)
--------------------------------------------------------------------------------
```

#### CSV
Researchers have requested a simple table-based format that could easily be imported into any database for analysis. Unlike other formats which export a single record per patient, this format generates 9 total files, and adds lines to each based on the clinical events for each patient. These files are intended to be analogous to database tables, with the patient UUID being a foreign key. Files include: `patients.csv`, `encounters.csv`, `allergies.csv`, `medications.csv`, `conditions.csv`, `careplans.csv`, `observations.csv`, `procedures.csv`, and `immunizations.csv` .

**Sample**:

patients.csv
```
ID,BIRTHDATE,DEATHDATE,SSN,DRIVERS,PASSPORT,PREFIX,FIRST,LAST,SUFFIX,MAIDEN,MARITAL,RACE,ETHNICITY,GENDER,BIRTHPLACE,ADDRESS
5e0d195e-1cd9-494d-8f9a-757c15da2aed,1946-12-14,2015-10-03,999-12-2377,S99962866,false,Mrs.,Miracle267,Ledner332,,Raynor597,M,white,irish,F,Millbury MA,2502 Fisher Manor Boston MA 02132
52082709-06ce-4fde-9c93-cfb4e6542ae1,1968-05-23,,999-17-1808,S99941406,X41451685X,Mrs.,Alda869,Gorczany848,,Funk527,M,white,italian,F,Gardner MA,46973 Velda Gateway Franklin Town MA 02038
8b4c62c8-b116-4b58-9259-466485b0345c,1967-06-22,1985-07-04,999-11-1173,S99955795,,Ms.,Moshe832,Zulauf396,,,,white,english,F,Boston MA,250 Reba Park Carver MA 02330
965c5539-598b-4a9b-a670-e0259667deb8,1934-11-04,2015-06-19,999-63-2195,S99931866,X71888970X,Mr.,Verla554,Roberts329,,,S,white,irish,M,Fall River MA,321 Abdullah Bridge Needham MA 02492
2b28d6c3-9e0c-48d4-99f9-292488133101,1964-08-13,,999-55-5054,S99990374,X68574707X,Ms.,Henderson277,Labadie810,,,S,black,dominican,F,North Attleborough MA,55825 Barrows Prairie Suite 144 Boston MA 02134
```

conditions.csv
```
START,STOP,PATIENT,ENCOUNTER,CODE,DESCRIPTION
1965-10-10,,5e0d195e-1cd9-494d-8f9a-757c15da2aed,918b17f4-e815-44ef-9eeb-41953bbcf7e9,38341003,Hypertension
1966-09-09,,5e0d195e-1cd9-494d-8f9a-757c15da2aed,918b17f4-e815-44ef-9eeb-41953bbcf7e9,15777000,Prediabetes
1988-09-25,,5e0d195e-1cd9-494d-8f9a-757c15da2aed,918b17f4-e815-44ef-9eeb-41953bbcf7e9,239872002,Osteoarthritis of hip
1990-09-01,,5e0d195e-1cd9-494d-8f9a-757c15da2aed,918b17f4-e815-44ef-9eeb-41953bbcf7e9,410429000,Cardiac Arrest
1990-09-01,,5e0d195e-1cd9-494d-8f9a-757c15da2aed,918b17f4-e815-44ef-9eeb-41953bbcf7e9,429007001,History of cardiac arrest (situation)
```


# Installing Synthea<sup>TM</sup>
## Prerequisites
 - Git
 - Java 1.8 or higher


To copy the Synthea<sup>TM</sup> repository locally and install the necessary gems, open a terminal window and run the following commands:

To clone the SyntheaTM repo, then build and run the test suite:

```
git clone https://github.com/synthetichealth/synthea.git
cd synthea
./gradlew build check test
```
**Note: if running on Windows, use .\gradlew.bat instead of ./gradle -- this guide uses ./gradle for brevity**

# Running Synthea<sup>TM</sup>

The primary entry point of Synthea<sup>TM</sup> is the "run" task, which can be run from the terminal in the `synthea` folder by running the following command.

```
./gradlew run
```

Alternatively, you may run the provided script "run_synthea" which allows for passing command-line arguments. 

```
./run_synthea
```
**Note: if running on Windows, use .\run_synthea.bat instead of ./run_synthea -- this guide uses ./run_synthea for brevity**

When you run this command, you should see output similar to the following:

```
user@hostname ~/synthea $ ./run_synthea

> Task :run
Loading C:\Users\dehall\synthea\build\resources\main\modules\allergic_rhinitis.json
Loading C:\Users\dehall\synthea\build\resources\main\modules\allergies\allergy_incidence.json
[... many more lines of Loading ...]
Loading C:\Users\dehall\synthea\build\resources\main\modules\wellness_encounters.json
Loaded 68 modules.
Running with options:
Population: 1
Seed: 1519063214833
Location: Massachusetts

1 -- Jerilyn993 Parker433 (10 y/o) Lawrence, Massachusetts

```


This command takes additional parameters to specify different regions or common run options. Any options not specified are left at the default value.

`run_synthea run_synthea [-s seed] [-p populationSize] [state [city]]`

Some examples:

 -   `run_synthea Massachusetts` -- to generate a population in all cities and towns in Massachusetts
 -   `run_synthea Alaska Juneau` -- to generate a population in only Juneau, Alaska
 -   `run_synthea -s 12345` -- to generate a population using seed 12345. Populations generated with the same seed and the same version of Synthea should be identical
 -   `run_synthea -p 1000` -- to generate a population of 1000 patients
 -   `run_synthea -s 987 Washington Seattle` -- to generate a population in only Seattle, Washington, using seed 987
 -   `run_synthea -s 21 -p 100 Utah "Salt Lake City"` -- to generate a population of 100 patients in Salt Lake City, Utah, using seed 21



# Configuring Synthea<sup>TM</sup>
Many features of Synthea<sup>TM</sup> are configurable using the settings file at `./src/main/resources/synthea.properties`.

## Summary of Configuration Options

### Export Formats

| Setting Name | Valid Values | Default | Description |
|:-------------|:-------------|:--------|:------------|
| `exporter.ccda.export` | `true`/`false` | `false` | Change this setting to `true` to enableexporting patients in CCDA format. |
| `exporter.fhir.export` | `true`/`false` | `true` | Change this setting to `false` to disable exporting patients in FHIR STU3 format. |
| `exporter.fhir_dstu2.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patients in FHIR DSTU2 format. |
| `exporter.hospital.fhir.export` | `true`/`false` | `true` | Change this setting to `false` to disable exporting hospital information in FHIR STU3 format. |
| `exporter.hospital.fhir_dstu2.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting hospital information in FHIR DSTU2 format. |
| `exporter.text.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patients in a simple text-based format. |
| `exporter.csv.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patient data in a comma-separated value format. |


### Other Export Settings

| Setting Name | Valid Values | Default | Description |
|:-------------|:-------------|:--------|:------------|
| `exporter.years_of_history` | Whole number | `10` | The number of years of patient history to include in patient records. For example, if set to `5`, then all patient history older than 5 years old (2012 or earlier) will not be included in the exported records. Note that conditions and medications that are currently active will still be exported, regardless of this setting. Set this to `0` to keep all history in the patient record. |
| `exporter.baseDirectory` | Folder paths | `./output/` | This is the base folder in which all patient records will be exported. Files will be exported to subfolders based on their type. (For example FHIR records will be stored in the `/fhir/` subfolder under this folder. |
| `exporter.subfolders_by_id_substring` | `true`/`false` | `false` | If true, patient records will be grouped into subfolders based on their UUID, which is a randomly generated unique identifer. This reduces the number of files in any single folder, and ensures a roughly even distribution of files among folders. However there is no correlation between files in any given folder as the IDs are random. |
| `synthea.exporter.use_uuid_filenames` | `true`/`false` | `false` | If true, patient records will have filenames based only on their UUID. If false, patient records will have filenames including the patient's name. This is mostly intended as a debugging feature - if watching the terminal as patients are generating, this makes it easier to find the record once it is generated. |


### Generation Settings

| Setting Name | Valid Values | Default | Description |
|:-------------|:-------------|:--------|:------------|
| `generate.default_population` | Whole numbers | `./output/` | This is the number of living patients (population size) that Synthea will generate. Synthea will generate patients until the number of living patients reaches this value. |
| `generate.database_type` | `file`, `in-memory`, or `none` | `in-memory` | Synthea uses a database to track various metrics, which can be used to produce various reports. If these reports are not desired, the database may be disabled to improve runtime performance. Alternatively, if it is desirable to persist this data across runs, use the `file` option to save data to a persistent database file. ( ./database.mv.db , which can be accessed using the H2 DB client) |


### Other Settings

Synthea<sup>TM</sup> contains a number of configurable options relating to the world, human vital signs, risk levels, etc. These have been pre-seeded with values based on US and Massachusetts data, with citations for these values where possible. Users generally will not want to modify these unless trying to simulate a specific result or trying to adapt Synthea<sup>TM</sup> to another geographic or geopolitical location.


## Examples of Common Tasks

### Generate a Specific Number of Patients

Option 1:
1. Change the value of setting `generate.default_population` to the desired number of (living) patients. Synthea will generate patients until the target number of living patients have been generated.
2. From a terminal, run `./run_synthea`

OR 

Option 2:
1. From a terminal, run `./run_synthea -p <number_of_patients>`

### Generate Patients only in FHIR Format
1. Change the value of setting `exporter.fhir.export` to `true` and change the value of the other `exporter.___.export` settings to `false`.
2. From a terminal, run `./run_synthea`


### Generate Patients Based on a Single State
1. From a terminal, run `./run_synthea <state_name>`


### Generate Patients Based on a Single City
1. From a terminal, run `./run_synthea <state_name> <city_name>`


### Generate Visual Representation of Disease Modules
1. From a terminal, run `./gradle graphviz`
2. The log will output the location of the generated files.