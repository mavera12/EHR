##Record Basics
The overall objective of Synthea is to produce synthetic health records from generated patients. Synthea accomplishes this by keeping track of a barebones record for each patient throughout its generation. All record-related code can be found in the folder `lib/records` with the exception of patient information such as birthdate, race, ethnicity, sex, etc. These are instead stored in an attributes hash for each person object. This code can be found in `lib/entity/entity.rb`.

###Synthea Record
The class for the Synthea record can be found in `lib/records/record.rb`. All information is stored in the form of simple hashes and arrays:

* `@patient_info` - an attributes hash that contains keys `:deathdate` and `:expired` to track if the patient is dead and the time of death
* `@present` - a hash that references a resource hash in one of the below arrays. For example, `@present[:diabetes]` yields the hash corresponding to diabetes in `@conditions`. This hash allows for easy lookup to determine if a patient has a certain condition, as well as modification of the condition. Procedures and conditions are being tracked by this hash.

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

###FHIR Record

The FHIR Record uses the [fhir_models](https://github.com/fhir-crucible/fhir_models) gem from Crucible to generate a FHIR Bundle in Ruby which is exported to JSON format afterwards.

`self.convert_to_fhir (entity)` can be called to generate the FHIR record for a patient. This function creates a new FHIR Bundle object, creates the FHIR Patient object, and then loops through the arrays in the Synthea record. Encounters are processed to the FHIR record one at a time. When an encounter is processed, all the other unprocessed events/resource in procedures, conditions, observations, and immunizations that occur before the encounter are written to the FHIR record. This simulates the diagnoses and recording that takes place during medical encounters. 

The `'fhir'` key for a resource should return a symbol that is the name of the appropriate method to process the resource.

###CCDA Record
The CCDA Record uses the [health-data-standards](https://github.com/projectcypress/health-data-standards) gem to generate a CCDA Record object in Ruby. The record is exported into both a XML and HTML format afterwards.

`self.convert_to_ccda(entity)` can be called to generate the CCDA record for a patient. The function works exactly the same as `self.convert_to_fhir(entity)` described above.

The `'ccda'` key for a resource should return a symbol that is the name of the appropriate method to process the resource.