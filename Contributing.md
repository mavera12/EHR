Currently, our goal is to improve existing Synthea modules and to guide the development of new modules. We are seeking more information and assistance with the following:

* Disease Models
  * [Existing Modules](#existing-modules)   
  * [New Modules](#new-modules)

## Existing Modules
**We are seeking review of individual modules by medical students and medical professionals.** Full-size, graphical representations of each module are available in our [Module Gallery](https://github.com/synthetichealth/synthea/wiki/Module-Gallery).

We need **quantitative** feedback in the form of [issues](https://github.com/synthetichealth/synthea/issues). Our models are driven by statistics, incidence, and prevalence rates. "How much?", "How often?", "At what ages?", and "When, historically?" are all good questions to ask. For starters, here are some questions about existing modules that we need answered:

## New Modules

We are constantly seeking new [generic modules](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework).

***

# Contributions to Avoid

* [External Systems Integrations](#external-systems-integrations)
* [New Sources of Randomness](#new-sources-of-randomness)

## External Systems Integrations
We don't typically accept pull requests of this type -- we consider the post-processing, extract, transformation, and loading (ETL) of Synthea data into external systems to be outside the scope of the project. While loading data into your system of choice important to you, it may not be to others, who might prefer OMOP, VistA, Document DBs (e.g. Mongo DB), Elastic Search, or something else entirely.

While we aim to support multiple formats (e.g. FHIR, C-CDA, CSV), we do not intend to directly support external systems integrations. Please do not issue a pull request for external systems integrations.

## New Sources of Randomness
Except in a few purposeful locations, the only source of randomness in Synthea should be from the *private* `Random` number generator inside each `Person`. If you write code that requires random number generation, you *must* use the `Person#rand*` methods to obtain those random numbers. Any creation of new `Random` objects will be declined until the code is appropriately refactored. This criteria also applies for the use of `System.currentTimeInMillis()`.
