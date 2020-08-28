# Synthea Random Variable Distribution Types

Synthea [Generic Module Framework](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework) states allow users to generate clinical records by using random variables to select values for various aspects of the simulation. Currently, the random variables in GMF states are uniformly distributed. Observations of many natural processes will follow different distributions, such as Gaussian or Poisson distributions. This page discusses approaches for supporting random variables with different distribution types in GMF states.

## Considerations

A few things to keep in mind when evaluating approaches for supporting different distribution types:

* Existing GMF JSON - Synthea already has a substantial amount of modules written. Any breaking changes introduced will require fixing the existing modules. Depending on the types of changes introduced, it may be possible to automate this process.
* Naming conventions - Most, but not all, random variables in Synthea have their characteristics defined in a property called `range`. This makes sense for a uniform distribution, but does not really make sense for other distribution types
* Existing simulation code -  Each subclass of `State` that generates a random observation, handles it differently
* Units - Some places co-locate the unit with the variable parameters, others have it in a separate place
* Existing module builder code - Any changes made in GMF will need to be reflected in the module builder web application

## Existing GMF States with Random Variables

| State | Random Variable Property | Units | Notes |
|-------|--------------------------|-------|-------|
| Delay | range | yes | also allows for exact values |
| Observation | range | no | also allows for exact values |
| Procedure | duration | yes |  |
| SetAttribute | range | no |  |
| Symptom | range | no | also allows for exact values |
| VitalSign | range | no | also allows for exact values |


