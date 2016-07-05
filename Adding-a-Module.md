###Guide to adding a new module

Modules implement the core functionality of Synthea. Each module models a specific aspect of a patient's medical history. Some examples of modules currently implemented include:
* Lifecycle - birth, death, height, and weight
* Encounters - medical encounters are performed, diagnoses are made, and the results are recorded
* Metabolic Syndrome - prediabetes, diabetes, nephropathy, retinopathy, and other symptoms of diabetes
* Cardiovascular Disease - coronary heart disease, myocardial infarction, cardiac arrest, stroke, atrial fibrillation
* Immunizations - vaccines and immunization schedules
* Other modules...

To start writing a module, create a new file in the `lib/modules` folder and namespace it as follows:

	Module Synthea
		Module Modules
			Class YourModuleName < Synthea::Rules

####Rules for 'rules'
Modules are populated with Rules, which are function calls that turn a block argument into a method called on every time step. In addition, the other arguments provide information to Graphviz that show the relationships between various attributes, conditions, and rules. Rules should be called as follows:

	rule :rule_name, [:input1, ...], [:output1, ...] do |time, entity|
		*Your code here*
	end

* The first argument is a symbol with the name of the rule.
* The second argument is an array of symbols representing the 'inputs' which are the attributes that affect the rule
* The third argument is another array of symbols representing the 'outputs' which are attributes that are calculated/determined from the rule
* The fourth argument is a block with the block variables |time, entity|. The block code is the body of the method that will be called on every time step
* The first three arguments actually have no effect on the rule method itself, they are solely used to show relationships between attributes and rules in Graphviz

####Record functions

Most modules will produce attributes/conditions that will be recorded in a patient's electronic health record. To record conditions, functions need to be defined in the module that will be called upon when a medical encounter is performed. Follow these guidelines when defining these functions:

* Place these functions at the end of a module and separate them from the rest of the code with a comment
* These functions only need `entity` and `time` parameters
* The function should handle any logic and ensure that an attribute is only recorded under the correct circumstances - assume the function can be called at any point during patient generation
* The function itself should call an instance method for `entity.record_synthea`, which actually saves information. The list of instance methods to be called are located in `lib/records/record.rb`

Below is an example of a record function from the cardiovascular disease module:

	def self.perform_encounter(entity, time)
	  [:coronary_heart_disease, :atrial_fibrillation].each do |diagnosis|
	    if entity[diagnosis] && !entity.record_synthea.present[diagnosis]
	      entity.record_synthea.condition(diagnosis, time, :condition, :condition)
	    end
	  end
	end

To ensure that the record function is called during an encounter, go to `lib/modules/encounters.rb` and add a call to it in the `rule :encounter`

For more information on records, check out the Records wiki page.

####Final Note
Additional methods, hashes, and other data structures that will be used by the rules and record functions can also be added. It is strongly recommended to adhere to these guidelines, but slight deviations are acceptable as well.

