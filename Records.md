## Record Basics
The overall objective of Synthea is to produce synthetic health records from generated patients. Synthea accomplishes this by keeping track of a barebones record for each patient throughout its generation. All record-related code can be found in the class `org.mitre.synthea.world.concepts.HealthRecord` with the exception of patient information such as birthdate, race, ethnicity, sex, etc. These are instead stored in an attributes map for each person object.

### Synthea Record
The class for the Synthea patient record can be found in `src/main/java/org/mitre/synthea/world/concepts/HealthRecord.java`.

* `@patient_info` - an attributes hash that contains keys `:deathdate` and `:expired` to track if the patient is dead and the time of death
* `@present` - a hash that references condition or procedure resource hashes. For example, `@present[:diabetes]` yields the hash corresponding to diabetes in `@conditions`. This hash allows for easy lookup to determine if a patient has a certain condition, as well as modification of the condition. Procedures and conditions are being tracked by this hash.

The following are arrays of hashes where each hash contains information on a single instance of its resource
* `@encounters` - Each hash is an encounter
* `@observations` - Each hash is an observation or diagnostic report
* `@conditions` - Each hash is a condition
* `@procedures` - Each hash is a procedure
* `@immunizations` - Each hash is an immunization
* `@careplans` - Each hash is a care plan
* `@medications` - Each hash is a medication order (aka prescription)

The resources appended to these arrays should be appended throughout the patient generation process, such that the array contents are always sorted in chronological order.

**An important note: the record and arrays itself should NOT be manipulated directly. When writing to the record, use the record functions provided.**

### FHIR Record

The FHIR Record uses the [HAPI FHIR](http://hapifhir.io/) library to generate a FHIR Bundle which is exported to JSON format afterwards.

Supported FHIR Resources:
- `Bundle`
- `Patient`
- `Condition`
- `Encounter`
- `AllergyIntolerance`
- `Observation`
- `DiagnosticReport`
- `Procedure`
- `Immunization`
- `CarePlan`
- `MedicationRequest`


Encounters are processed to the FHIR record one at a time. When an encounter is processed, all the other unprocessed events/resource in procedures, conditions, observations, and immunizations that occur before the encounter are written to the FHIR record. This simulates the diagnoses and recording that takes place during medical encounters. 

### CCDA Record
The CCDA Record uses the [MDHT CDA Tools](http://cdatools.org/) library along with templates from the [health-data-standards](https://github.com/projectcypress/health-data-standards) gem to export patients in CCDA XML.

Supported CCDA Entries:
 - Vital Signs
 - Conditions
 - Encounters
 - Immunizations
 - Procedures
 - Care Plans
 - Medications

### Text Record
You can enable text records in `src/main/resources/synthea.properties` by setting `exporter.csv.export = true`. The Text Exporter will output a UTF-8 text file that lists a human-readable text file containing the patient record.

### CSV Record
You can enable Comma-Separated Value (CSV) records in `src/main/resources/synthea.properties` by setting `exporter.csv.export = true`. The CSV Exporter will output a collection of UTF-8 CSV files that lists patients, encounters, medications, and so forth in separate files -- where patients and encounters have unique identifiers and other resources (such as conditions and medications) are appropriately linked to the patients and encounters. The exported files are suitable for an Extract-Transform-Load (ETL) process to import them into a relational database.
