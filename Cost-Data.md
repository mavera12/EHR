Costs in Synthea represent a very simplified version of real-world costs. While real-world costs can vary wildly due to differences providers, cities, states, payers, as of May 2018 Synthea only models cost at a basic level, where prices for a service are consistent across all instances of that service. Synthea stores all these prices in lookup tables as listed below:

## Encounters
Encounter costs are stored in `src/main/resources/costs/encounters.csv`.

Format:

| Code | Cost | Comments |
|:-----|:-----|:---------|
| The SNOMED code for the encounter | The cost, in USD. Do not include the $ | Any comments on how the cost was determined. This is only intended to be read by maintainers and not parsed by the system |

## Procedures
Procedure costs are stored in `src/main/resources/costs/procedures.csv`.

| Code | Cost | Comments |
|:-----|:-----|:---------|
| The SNOMED code for the procedure | The cost, in USD. Do not include the $ | Any comments on how the cost was determined. This is only intended to be read by maintainers and not parsed by the system |

Where not otherwise mentioned, procedure costs are likely based on publicly available Medicare data:
https://hcupnet.ahrq.gov/#query/eyJBTkFMWVNJU19UWVBFIjpbIkFUX1EiXSwiWUVBUlMiOlsiWVJfMjAxNCJdLCJTVEFURVMiOlsiU1RfTUEiXSwiQ0FURUdPUklaQVRJT05fVFlQRSI6WyJDVF9DQ1NQIl0sIlFVSUNLVEFCTEVfVFlQRSI6WyJRVFRfREVUIl0sIkRBVEFTRVRfU09VUkNFIjpbIkRTX1NJRCJdfQ==

## Medications
Medication costs are stored in `src/main/resources/costs/medications.csv`.

| Code | Cost | Comments |
|:-----|:-----|:---------|
| The RxNorm code for the medication | The cost, in USD. Do not include the $ | Any comments on how the cost was determined. This is only intended to be read by maintainers and not parsed by the system |

Where not specifically mentioned in the comments, cost data for medications is likely to have been sourced from publicly available Medicare data: https://data.cms.gov/Medicare-Part-D/Medicare-Provider-Utilization-and-Payment-Data-201/yvpj-pmj2
For example, this data source provides medication costs grouped by drug name and provider NPI. By grouping by drug name, dividing the `total_drug_cost` column by the `total_claim_count` column, and then averaging across providers, we obtain a rough estimate of the unit cost of that drug.

## Immunizations
Immunization costs are stored in `src/main/resources/costs/immunizations.csv`.

| Code | Cost | Comments |
|:-----|:-----|:---------|
| The CVX code for the immunization | The cost, in USD. Do not include the $ | Any comments on how the cost was determined. This is only intended to be read by maintainers and not parsed by the system |