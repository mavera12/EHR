By default, Synthea contains demographics, zip codes, providers, names, and costs for the entire United States, post-processed from publicly available files.

You can modify these files or have Synthea use an alternative files by altering the `src/main/resources/synthea.properties` file:

```properties
# Default demographics is every city in the US
generate.demographics.default_file = geography/demographics.csv
generate.geography.zipcodes.default_file = geography/zipcodes.csv

# Default provider files (1 of 10 files)
generate.providers.hospitals.default_file = providers/hospitals.csv

# Default costs, to be used for pricing something that we don't have a specific price for
generate.costs.default_procedure_cost = 500.00
generate.costs.default_medication_cost = 255.00
generate.costs.default_encounter_cost = 125.00
generate.costs.default_immunization_cost = 136.00
```

You can modify the demographics file or have Synthea use an alternative demographics file by altering the `src/main/resources/synthea.properties` file:

```properties
# default demographics is every city in the US
generate.demographics.default_file = geography/demographics.csv
```

You can find detailed instructions on each of these file formats on the Synthea Wiki:

* **Demographics** file contains city-level distributions of Gender, Race, Age, Income, and Education which all play a role in health access, outcomes, and costs. https://github.com/synthetichealth/synthea/wiki/Demographics-for-Other-Areas
* **Zip or postal code** file contains postal codes and geographic locations (latitude and longitude) for each location. https://github.com/synthetichealth/synthea/wiki/Zip-or-Postal-Codes
* **Provider** files contains information and locations (latitude and longitude) for each provider location. https://github.com/synthetichealth/synthea/wiki/Provider-Data
* **Names** file contains language-specific family names and first names by gender. It also contains names suitable for street addresses. https://github.com/synthetichealth/synthea/wiki/Name-Data
* **Cost** files contain costs for different types of Encounters, Procedures, Medications, and Immunizations. https://github.com/synthetichealth/synthea/wiki/Cost-Data

