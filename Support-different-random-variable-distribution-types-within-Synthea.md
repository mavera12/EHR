# Synthea Random Variable Distribution Types

Synthea [Generic Module Framework](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework) states allow users to generate clinical records by using random variables to select values for various aspects of the simulation. Currently, the random variables in GMF states are uniformly distributed. Observations of many natural processes will follow different distributions, such as Gaussian or Poisson distributions. This page discusses approaches for supporting random variables with different distribution types in GMF states.

## Considerations

A few things to keep in mind when evaluating approaches for supporting different distribution types:

* Existing GMF JSON - Synthea already has a substantial amount of modules written. Any breaking changes introduced will require fixing the existing modules. Depending on the types of changes introduced, it may be possible to automate this process.