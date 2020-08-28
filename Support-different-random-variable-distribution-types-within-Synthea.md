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

## Potential Solutions

### Move units out of ranges/durations
On the java side of things, if the random variable has units on the same level, it gets placed into a `RangeWithUnits` class. Otherwise, it gets loaded into a `Range` class. When we support different distribution types, since they will have different parameters, they are likely to have their own java classes. If the units are kept with the properties for the distributions, this will likely create the need for two classes per distribution. So, something like Gaussian and GaussianWithUnits. The downside to the move is that it breaks existing GMF JSON, but it would be straightforward to create a script to translate existing modules.

* Pros
  * Cleaner implementation of new distribution types
  * More uniform state definitions
* Cons
  * Breaks existing GMF JSON format
  * Would require translation of existing modules

### How to detect distribution type
#### Add a type property
Have an extra property in the description of the distribution called `type`. All existing values would have to be updated to include a `"type": "normal"`.

* Pros
  * Easy for the Gson related code to figure out what is going on
  * Different distribution types could have the same parameter names
* Cons
  * Breaks existing GMF JSON format
  * Would require translation of existing modules

#### Auto-detect distribution type based on property names
Look at the properties specified for the distribution and select one based on property names. So if a distribution description in JSON has a `low` and `high` property, use a normal distribution.

* Pros
  * Backwards compatible
* Cons
  * Requires distributions to specify their properties in a way that does not conflict
  * Potential for user confusion if a distribution is picked that they didn't expect

