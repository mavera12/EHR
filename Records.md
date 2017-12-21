## Record Basics
The overall objective of Synthea is to produce synthetic health records from generated patients. Synthea accomplishes this by keeping track of a barebones record for each patient throughout its generation. All record-related code can be found in the folder `lib/records` with the exception of patient information such as birthdate, race, ethnicity, sex, etc. These are instead stored in an attributes hash for each person object. This code can be found in `lib/entity/entity.rb`.

### Synthea Record
The class for the Synthea record can be found in `lib/records/record.rb`. All information is stored in the form of simple hashes and arrays:

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

The functions in this class are called in the modules wherever it is appropriate to write to a patient's record. Refer to these functions when writing calls to them and passing in the appropriate arguments. The `fhir_method` and `ccda_method` parameters determine the method in `lib/results/fhir.rb` or `lib/results/ccda.rb` that will be called respectively when processing the resource. A symbol that is the name of the fhir/ccda method should be passed in as the arguments for these parameters.

**An important note: the record and arrays itself should NOT be manipulated directly. When writing to the record, use the record functions provided.**

### FHIR Record

The FHIR Record uses the [fhir_models](https://github.com/fhir-crucible/fhir_models) gem from Crucible to generate a FHIR Bundle in Ruby which is exported to JSON format afterwards.

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


`self.convert_to_fhir (entity, end_time = Time.now)` can be called to generate the FHIR record for a patient. This function creates a new FHIR Bundle object, creates the FHIR Patient object, and then loops through the arrays in the Synthea record. The `end_time` argument is a hard stop that determines which entities end up in the patient record. Any conditions, medication orders, or encounters that occur in the future (after the `end_time`) are not exported as part of the record.

Encounters are processed to the FHIR record one at a time. When an encounter is processed, all the other unprocessed events/resource in procedures, conditions, observations, and immunizations that occur before the encounter are written to the FHIR record. This simulates the diagnoses and recording that takes place during medical encounters. 

### CCDA Record
The CCDA Record uses the [health-data-standards](https://github.com/projectcypress/health-data-standards) gem to generate a CCDA Record object in Ruby. The record is exported into both a XML and HTML format afterwards.

`self.convert_to_ccda(entity, end_time = Time.now)` can be called to generate the CCDA record for a patient. The function works exactly the same as `self.convert_to_fhir()` described above.

Supported CCDA Entries:
 - Vital Signs
 - Conditions
 - Encounters
 - Immunizations
 - Procedures
 - Care Plans
 - Medications

### Text Record
You can enable text records in `config/synthea.yml` by setting `synthea:exporter:text:export` to `true`. The Text Exporter will output a UTF-8 text file that lists a human-readable text file containing the patient record.

### CSV Record
You can enable Comma-Separated Value (CSV) records in `config/synthea.yml` by setting `synthea:exporter:csv:export` to `true`. The CSV Exporter will output a collection of UTF-8 CSV files that lists patients, encounters, medications, and so forth in separate files -- where patients and encounters have unique identifiers and other resources (such as conditions and medications) are appropriately linked to the patients and encounters. The exported files are suitable for an Extract-Transform-Load (ETL) process to import them into a relational database.
