###Guide to adding a new module

Modules implement the core functionality of Synthea. Each module models a specific aspect of a patient's medical history. Some examples of modules currently implemented include:
* Lifecycle - birth, death, height, and weight
* Encounters - medical encounters are performed, diagnoses are made, and the results are recorded
* Metabolic Syndrome - prediabetes, diabetes, nephropathy, retinopathy, and other symptoms of diabetes
* Cardiovascular Disease - coronary heart disease, myocardial infarction, cardiac arrest, stroke, atrial fibrillation
* Immunizations - vaccines and immunization schedules
* Other modules...

To start writing your own module, create a new file in the `lib/modules` folder and namespace it as follows:

	Module Synthea
		Module Modules
			Class YourModuleName < Synthea::Rules

Modules are populated with Rules, which are functions that are called consistently on every time step. In addition to being functions, their method signatures provide information to graphviz and show the relationships between various attributes, conditions, and rules. Rules should be structured as follows:

