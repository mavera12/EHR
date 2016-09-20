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

- [Synthea](https://github.com/synthethichealth/synthea): the synthetic health record generator.
- [Synthea Graphviz](https://github.com/synthethichealth/synthea_graphviz): tool to visualize synthea modules as graphs.