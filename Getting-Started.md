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

Synthea<sup>TM</sup> currently supports exporting patients as Fast Healthcare Interoperability Resources (FHIR), version 1.8.0. FHIR is a standard created by HL7 for exchanging healthcare information electronically. The philosophy behind FHIR is to build a base set of resources that, either by themselves or when combined, satisfy the majority of common use cases. For more information on FHIR, see [http://hl7.org/fhir/2017Jan/index.html](http://hl7.org/fhir/2017Jan/index.html) While FHIR supports both XML and JSON, Synthea exports FHIR as JSON only. Synthea uses the [fhir_models](https://github.com/fhir-crucible/fhir_models) library to export FHIR.

#### C-CDA

Synthea<sup>TM</sup> uses the [health-data-standards](https://github.com/projectcypress/health-data-standards) library to export patients as Consolidated Clinical Document Architecture (C-CDA) format. C-CDA is an XML-based standard defined by HL7, that uses templates from a standard library to represent clinical concepts. For more information on C-CDA,  see [http://www.hl7.org/implement/standards/product_brief.cfm?product_id=258](http://www.hl7.org/implement/standards/product_brief.cfm?product_id=258). 

#### HTML
Synthea<sup>TM</sup> uses the [health-data-standards](https://github.com/projectcypress/health-data-standards) library to export patients in a human-readable HTML format.

**Sample:**  
![html_sample](https://cloud.githubusercontent.com/assets/13512036/24475242/430c4ecc-149d-11e7-96bc-9280fee0989a.JPG)

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
 - Ruby 2.1.0 or higher
 - [Graphviz](http://www.graphviz.org/) (required for generating images from modules, patients can be generated without this)
 - [MongoDB](https://www.mongodb.com/) (required to output records in C-CDA or HTML format, patients can be generated in FHIR or other formats without this)


To copy the Synthea<sup>TM</sup> repository locally and install the necessary gems, open a terminal window and run the following commands:

```
git clone https://github.com/synthetichealth/synthea.git
cd synthea
gem install bundler
bundle install
```

## Platform-Specific Notes
### Mac OS X
Ruby 2.0.0 comes shipped with Mac OS. If you want to use another or multiple versions of Ruby, we recommend using RVM. See [https://rvm.io/](https://rvm.io/) for more information.

You may already have Git installed if you use Xcode. For more information on Git, see [https://www.atlassian.com/git/tutorials/install-git#mac-os-x](https://www.atlassian.com/git/tutorials/install-git#mac-os-x) .

Graphviz can also be installed from [Homebrew](https://brew.sh/) (with `brew install graphviz`) or [MacPorts](https://www.macports.org/) (with port "graphviz")

For more info on MongoDB on Mac, see [https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)


### Windows
For Windows-specific installation instructions of Git and Synthea<sup>TM</sup>, please see [https://github.com/synthetichealth/synthea/wiki/Windows-installation](https://github.com/synthetichealth/synthea/wiki/Windows-installation)

For more on MongoDB on Windows, see [https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)


# Configuring Synthea<sup>TM</sup>
Many features of Synthea<sup>TM</sup> are configurable using the settings file at `./config/synthea.yml`. This settings file is hierarchical, so to refer to individual settings we use the complete name. For example, to describe the following settings:


```
synthea:
  exporter:
    ccda:
      export: true
      upload: null
    fhir:
      export: true
      upload: null
    html:
      export: false
    text:
      export: false
    csv:
      export: false
      export_headers: true
```

We describe the configuration as:

 - `synthea.exporter.ccda.export` is set to `true`
 - `synthea.exporter.ccda.upload` is set to `null`
 - `synthea.exporter.fhir.export` is set to `true`
 - `synthea.exporter.fhir.upload` is set to `null`
 - `synthea.exporter.html.export` is set to `false`
 - `synthea.exporter.text.export` is set to `false`
 - `synthea.exporter.csv.export` is set to `false`
 - `synthea.exporter.csv.export_headers` is set to `true`


## Summary of Configuration Options

### Export Formats

| Setting Name | Valid Values | Default | Description |
|:-------------|:-------------|:--------|:------------|
| `synthea.exporter.ccda.export` | `true`/`false` | `true` | Change this setting to `false` to disable exporting patients in CCDA format. Note that CCDA format requires MongoDB, so if MongoDB is not installed, this setting must be changed to `false` |
| `synthea.exporter.fhir.export` | `true`/`false` | `true` | Change this setting to `false` to disable exporting patients in FHIR format. |
| `synthea.exporter.fhir.upload` | `null`/URL of FHIR Server | `null` | Put the URL of a FHIR server here to enable automatic uploading of FHIR records to that server upon generation. Does nothing if `synthea.exporter.ccda.export` is `false`. NOTE: this process may significantly slow generation, so it may be preferable to explore alternative options. |
| `synthea.exporter.html.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patients in HTML format. For human-readable records, we recommend either HTML or Text export be enabled. Note that HTML format requires MongoDB, so if MongoDB is not installed, this setting must be left at `false` |
| `synthea.exporter.text.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patients in a simple text-based format. |
| `synthea.exporter.csv.export` | `true`/`false` | `false` | Change this setting to `true` to enable exporting patient data in a comma-separated value format. |
| `synthea.exporter.csv.export_headers` | `true`/`false` | `true` | This setting enables or disables exporting a header line on CSV exported records. It may be desired in certain cases to disable the header line by changing this setting to `false`, for example if concatenating multiple results together. |

### Other Export Settings

| Setting Name | Valid Values | Default | Description |
|:-------------|:-------------|:--------|:------------|
| `synthea.exporter.years_of_history` | Whole number | `5` | The number of years of patient history to include in patient records. For example, if set to `5`, then all patient history older than 5 years old (2012 or earlier) will not be included in the exported records. Note that conditions and medications that are currently active will still be exported, regardless of this setting. Set this to `0` to keep all history in the patient record. |
| `synthea.exporter.location` | Folder paths | `./output/` | This is the base folder in which all patient records will be exported. Files will be exported to subfolders based on their type. (For example FHIR records will be stored in the `/fhir/` subfolder under this folder. |
| `synthea.exporter.folder_per_city` | `true`/`false` | `false` | If true, patient records will be grouped into subfolders based on the patient's city. |
| `synthea.exporter.subfolders_by_id_substring` | `true`/`false` | `true` | If true, patient records will be grouped into subfolders based on their UUID, which is a randomly generated unique identifer. This reduces the number of files in any single folder, and ensures a roughly even distribution of files among folders. However there is no correlation between files in any given folder as the IDs are random. |
| `synthea.exporter.use_uuid_filenames` | `true`/`false` | `true` | If true, patient records will have filenames based on their UUID. If false, patient records will have filenames based on the patient's name and age. This is mostly intended as a debugging feature - if watching the terminal as patients are generating, this makes it easier to find the record once it is generated.   |


### Generation Settings

| Setting Name | Valid Values | Default | Description |
|:-------------|:-------------|:--------|:------------|
| `synthea.sequential.population` | Whole numbers | `./output/` | This is the number of living patients (population size) that Synthea will generate. Synthea will generate patients until the number of living patients reaches this value. |
| `synthea.sequential.multithreading` | `true`/`false` | `false` | If true, (and supported by your version of Ruby) Synthea will generate patients using multiple threads. This results in significantly improved throughput and shortened time to generate large number of patients |
| `synthea.sequential.clean_output_each_run` | `true`/`false` | `true` | If true, Synthea will clear out the output directory each time it starts generation. This means that any generated patients will be lost if not backed up or transferred elsewhere. This defaults to true in order to conserve space. |


### Other Settings

Synthea<sup>TM</sup> contains a number of configurable options relating to the world, human vital signs, risk levels, etc. These have been pre-seeded with values based on US and Massachusetts data, with citations for these values where possible. Users generally will not want to modify these unless trying to simulate a specific result or trying to adapt Synthea<sup>TM</sup> to another geographic or geopolitical location.

# Running Synthea<sup>TM</sup>

The primary entry point of Synthea<sup>TM</sup> is the "generate" task, which can be run from the terminal in the `synthea` folder by running the following command.

```
bundle exec rake synthea:generate
```

When you run this command, you should see output similar to the following:

```
user@hostname ~/synthea $ bundle exec rake synthea:generate
Clearing output folders...
Loaded "Pregnancy" module from /home/user/synthea/lib/generic/modules/pregnancy.json
Loaded "Dermatitis" module from /home/user/synthea/lib/generic/modules/dermatitis.json
Loaded "Fibromyalgia" module from /home/user/synthea/lib/generic/modules/fibromyalgia.json
[... many more modules loaded ...]
Loaded "Injuries" module from /home/user/synthea/lib/generic/modules/injuries.json
Generating 100 patients...
[2017-03-29 14:11:43] 1: Raynor416, Gino157. White French canadian. 26 y/o M 161 lbs. --
[2017-03-29 14:11:44] 2(d: myocardial_infarction): Jones760, Zackary250. White American. 57 y/o M 206 lbs. -- atopic_dermatitis, allergy_to_fish, perennial_allergic_rhinitis, coronary_heart_disease, myocardial_infarction, history_of_myocardial_infarction
[2017-03-29 14:11:45] 3: Lehner650, Delia192. White Italian. 4 y/o F 33 lbs. --
...

```


This command takes an optional parameter, which is the name of a file containing census statistics, for example:

```
bundle exec rake synthea:generate['./config/Suffolk_County.json']
```
This command uses the census statistics to choose "targets" for generated patients based on city, including a target age, gender, race, ethnicity, and socioeconomic status. 
When you run this command, you should see similar output to the above, but additionally some indication that there are city-specific populations being generated:

```
user@hostname ~/synthea $ bundle exec rake synthea:generate['./config/Middlesex_County.json']
Clearing output folders...
Loaded "Pregnancy" module from /home/user/synthea/lib/generic/modules/pregnancy.json
[...]
Loaded "Injuries" module from /home/user/synthea/lib/generic/modules/injuries.json
Generating 100 patients...
Generating 1 patients within Acton
[2017-03-29 16:07:47] Acton 1/1: Goyette117, Jannie717. White Portuguese. 62 y/o M 243 lbs. -- drug_overdose, chronic_sinusitis_(disorder)
Generating 1 patients within Arlington
[2017-03-29 16:07:49] Arlington 1/1(d: chronic_obstructive_bronchitis_(disorder)): Boyle939, Kade518. White Italian. 61 y/o M 251 lbs. -- prediabetes, chronic_sinusitis_(disorder), chronic_obstructive_bronchitis_(disorder), coronary_heart_disease, osteoporosis_(disorder)
[2017-03-29 16:07:51] Arlington 1/1(d: myocardial_infarction): Kilback378, Jace581. White Italian. 27 y/o M 181 lbs. -- coronary_heart_disease, chronic_sinusitis_(disorder), myocardial_infarction, history_of_myocardial_infarction
[2017-03-29 16:07:54] Arlington 1/1: Tromp265, Josianne933. White Italian. 64 y/o M 241 lbs. --
Generating 1 patients within Ashby
[2017-03-29 16:07:55] Ashby 1/1: Keeling746, Napoleon147. White Polish. 27 y/o M 146 lbs. --
Generating 1 patients within Ashland
[2017-03-29 16:07:57] Ashland 1/1: Trantow54, Colt161. White Polish. 45 y/o F 219 lbs. -- female_infertility, prediabetes, appendicitis, rupture_of_appendix, history_of_appendectomy
Generating 1 patients within Ayer
```

Note in this case that Arlington patient 1/1 was generated 3 times, because the first 2 times the patient died before reaching the target age range.



In either case, when Synthea<sup>TM</sup> is finished generating patients, you will see a large volume of metrics, ending with output similar to the following:

```
  "population_count": 161,
  "living": 100,
  "living_adults": 78,
  "age_sum": 7191,
  "adults": 134,
  "dead": 61,
  "living_hypertension_and_diabetes": 5
}
Outputting Metabolic Syndrome Results to output/prevalences.csv
Completed in 5 minute(s) 4 second(s).
Finished.
user@hostname ~/synthea $ 

```



## Examples of Common Tasks

### Generate a Specific Number of Patients

1. Change the value of setting `synthea.sequential.population` to the desired number of (living) patients. Synthea will generate patients until the target number of living patients have been generated.
2. From a terminal, run `bundle exec rake synthea:generate`

### Generate Patients only in FHIR Format

1. Change the value of setting `synthea.exporter.fhir.export` to `true` and change the value of the other `synthea.exporter.fhir.____` settings to `false`.
2. From a terminal, run `bundle exec rake synthea:generate`

### Generate Patients Based on a Single County

1. Identify the configuration file containing census information for the desired county.
2. From a terminal, run `bundle exec rake synthea:generate[<the path to the selected configuration file>]`


### Generate Visual Representation of Disease Modules

1. Ensure that Graphviz is installed.
2. From a terminal, run `bundle exec rake synthea:graphviz`
3. The log will output the location of the generated files.