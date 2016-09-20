##Welcome to the Synthea Wiki!

Synthea is a synthetic patient generator that models the medical history of synthetic patients. Some attributes that are currently modeled include cardiovascular diseases, diabetes and associated symptoms, food allergies, immunizations, and others. The objective is to create a collection of fake, but realistic, electronic health records outputted in a variety of formats. Currently supported formats are C-CDA and FHIR STU3.

For a quick start guide, see the [README](https://github.com/synthetichealth/synthea/blob/master/README.md).

## We are actively looking for project contributors and maintainers! 

If you are new to open source, please check out [github's guide to contributing](https://guides.github.com/activities/contributing-to-open-source/), otherwise please start contributing by:
- Creating Issues
- Forking our repos and issuing pull requests
- Editing our Wiki
- Contributing disease modules. In particular we are targeting the [Global Burden of Disease (GBD) Top 10 Leading Causes of Years of Life Lost (YLL) for the United States](http://www.healthdata.org/united-states) (listed below), but are interested in any diseases. We prefer that new contributions come in the form of [generic modules](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework).
1. Ischemic Heart Disease ([done](https://github.com/synthetichealth/synthea/blob/master/lib/modules/cardiovascular_disease.rb))
2. Lung Cancer (in progress)
3. Alzheimer Disease
4. COPD
5. Cerebrovascular Disease
6. Road Injuries (in progress)
7. Self-harm
8. Diabetes ([done](https://github.com/synthetichealth/synthea/blob/master/lib/modules/metabolic_syndrome.rb))
9. Colorectal Cancer
10. Drug Use Disorders ([opioid model](https://github.com/synthetichealth/synthea/blob/master/lib/generic/modules/opioid_addiction.json))

# Guide to Synthea Projects

Crucible is a Ruby on Rails application that is built to run on top of Mongo DB. The project has five main repositories:

- [Crucible](https://github.com/fhir-crucible/crucible): the Ruby on Rails application
- [Plan Executor](https://github.com/fhir-crucible/plan_executor): the library that contains and runs all the tests executed by Crucible. This can also be used as a separate command-line tool and integrated with Continuous Integration tools.
- [FHIR Client](https://github.com/fhir-crucible/fhir_client): this library implements the FHIR RESTful API including the extended operations (e.g. $everything).
- [FHIR Models](https://github.com/fhir-crucible/fhir_models): this library implements the FHIR Resources, FluentPath, and validation. Most of the code is auto-generated from the [StructureDefinitions and ValueSets produced by the official FHIR build](http://hl7-fhir.github.io/downloads.html).
- [FHIR Scorecard](https://github.com/fhir-crucible/fhir_scorecard): this library implements the rubrics used to score a Bundle representing a patient record.