By default, the project contains Payers (i.e. public and private health insurance companies) in the `src/main/resources/payers/insurance_companies.csv` file, which is a list of some public and private insurers who provide services nationally (US). 

**The default financial numbers (e.g. premiums and copays) are not accurate.**

You can modify the payer file or have Synthea use an alternative payer file by altering the `src/main/resources/synthea.properties` file:

```properties
# Payers
generate.payers.insurance_companies.default_file = payers/insurance_companies.csv

# Payer selection behavior
# How patients select a payer:
#  best_rates - select insurance with best rates for person's existing conditions and medical needs
#  random  - select randomly.
generate.payers.selection_behavior = random
```

## Payer File Format

The payer file is a CSV file, with a single row for each payer.

Column Header | Description
--------------|------------
` ` | The first column has no header. It is the number of the payer. The program ignores this column.
`id` | This the payer identifier, this can just be a unique identifier.
`name` | The name of the payer.
`address` | The street address.
`city` | The city.
`state_headquartered` | The state abbreviation (or province or territory).
`zip` | The zip or postal code.
`phone` | The hospital or facility phone number.
`states_covered` | The states that this payer offers plans (e.g. patients in these states can purchase coverage). Use `*` to signify all states. Otherwise, used a quoted comma-separated string. e.g. "MA,CT,NY"
`services_covered` | The services covered (i.e. paid for) by this payer. Use `*` to signify all services. Otherwise, used a quoted comma-separated string. e.g. "wellness, medications,emergency"
`deductible` | The amount required out of pocket from patients before the payer will reimburse.
`default_coinsurance` | Coinsurance as a percentage. `0.0 - 1.0`. Currently unused. Maybe used in future versions.
`default_copay` | The amount required out of pocket from patients at each encounter.
`monthly_premium` | The monthly fee paid by the patient to obtain an insurance plan from this payer.
`ownership` | String. `Government` or `Private`
