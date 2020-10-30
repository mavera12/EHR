# Introduction
## Welcome to Synthea<sup>TM</sup>!
Thank you for your interest in Synthea<sup>TM</sup>. 

Synthea<sup>TM</sup> is a synthetic patient generator that models the medical history of synthetic patients. Our mission is to output high-quality synthetic, realistic but not real, patient data and associated health records covering every aspect of healthcare. The resulting data is free from cost, privacy, and security restrictions. It can be used without restriction for a variety of secondary uses in academia, research, industry, and government (although a citation would be appreciated).

## How does Synthea<sup>TM</sup> work?
Synthea<sup>TM</sup> generates synthetic patient records using an agent-based approach. Each synthetic patient is generated independently, as they progress from birth to death through modular representations of various diseases and conditions. Each patient runs through every module in the system. Once a patient dies or the simulation reaches the current day, that patient record can be exported in a number of different formats.

![](https://raw.githubusercontent.com/synthetichealth/synthea/gh-pages/images/architecture.png)

### Disease Modules
The Synthea<sup>TM</sup> [Generic Module Framework](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework) allows for the creation of state machines representing the progression and standards of care for common diseases, using a set of predefined states, transitions, and conditional logic. Modules are built based on publicly available health data, including disease incidence and prevalence statistics, and clinical practice guidelines. The disease modules represent a Monte Carlo simulation, where events occur depending with varying frequency based on repeated weighted random sampling.

For a walk-through of a complete example, see the [Complete Example](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Complete-Example) page. To view, edit, or create new modules, try our online [Module Builder](https://synthetichealth.github.io/module-builder/).

### Patient Record Formats

#### FHIR

Synthea<sup>TM</sup> currently supports exporting patients as Fast Healthcare Interoperability Resources (FHIR), versions 4.0.1 (R4), 3.0.1 (STU3) and 1.0.2 (DSTU2). FHIR is a standard created by HL7 for exchanging healthcare information electronically. The philosophy behind FHIR is to build a base set of resources that, either by themselves or when combined, satisfy the majority of common use cases. For more information on FHIR, see [http://hl7.org/fhir/index.html](http://hl7.org/fhir/index.html) While FHIR supports both XML and JSON, Synthea exports FHIR as JSON only. Synthea uses the [HAPI FHIR](http://hapifhir.io/) library to export FHIR.

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
Researchers have requested a simple table-based format that could easily be imported into any database for analysis. Unlike other formats which export a single record per patient, this format generates 9 total files, and adds lines to each based on the clinical events for each patient. These files are intended to be analogous to database tables, with the patient UUID being a foreign key. Files include: `patients.csv`, `encounters.csv`, `allergies.csv`, `medications.csv`, `conditions.csv`, `careplans.csv`, `observations.csv`, `procedures.csv`, and `immunizations.csv`. See the [CSV File Data Dictionary](https://github.com/synthetichealth/synthea/wiki/CSV-File-Data-Dictionary) for definitions of the CSV files.

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

**[Next: Basic Setup and Running](https://github.com/synthetichealth/synthea/wiki/Basic-Setup-and-Running)**