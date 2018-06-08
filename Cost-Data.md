Costs in Synthea represent a very simplified version of real-world costs. While real-world costs can vary wildly due to differences in providers, cities, states, and payers -- as of May 2018 -- Synthea only models cost at a basic level, where prices for a service are selected from a range and then adjusted using a state-based adjustment factor. Synthea stores all these prices in lookup tables as listed below.

In the case where costs are not included in the lookup tables, there are default values configured in the `src/main/resources/synthea.properties` file:

```properties
# Default Costs, to be used for pricing something that we don't have a specific price for
# -- $500 for procedures is completely invented
generate.costs.default_procedure_cost = 500.00
# -- $255 for medications - also invented
generate.costs.default_medication_cost = 255.00
# -- Encounters billed using avg prices from https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3096340/
# -- Adjustments for initial or subsequent hospital visit and level/complexity/time of encounter
# -- not included. Assume initial, low complexity encounter (Tables 4 & 6)
generate.costs.default_encounter_cost = 125.00
# -- https://www.nytimes.com/2014/07/03/health/Vaccine-Costs-Soaring-Paying-Till-It-Hurts.html
# -- currently all vaccines cost $136.
generate.costs.default_immunization_cost = 136.00
```

## Encounters
Encounter costs are stored in `src/main/resources/costs/encounters.csv`.

Format:

| Code | Min | Mode | Max | Comments |
|:-----|:----|:-----|:----|:---------|
| The SNOMED code for the encounter | The minimum cost, in USD. | The most common cost, in USD. | The maximum cost, in USD. | Any comments on how the cost was determined. This is only intended to be read by maintainers and not parsed by the system |

## Procedures
Procedure costs are stored in `src/main/resources/costs/procedures.csv`.

| Code | Min | Mode | Max | Comments |
|:-----|:----|:-----|:----|:---------|
| The SNOMED code for the procedure | The minimum cost, in USD. | The most common cost, in USD. | The maximum cost, in USD. | Any comments on how the cost was determined. This is only intended to be read by maintainers and not parsed by the system |

Where not otherwise mentioned, procedure costs are likely based on publicly available Medicare data:
https://hcupnet.ahrq.gov/#query/eyJBTkFMWVNJU19UWVBFIjpbIkFUX1EiXSwiWUVBUlMiOlsiWVJfMjAxNCJdLCJTVEFURVMiOlsiU1RfTUEiXSwiQ0FURUdPUklaQVRJT05fVFlQRSI6WyJDVF9DQ1NQIl0sIlFVSUNLVEFCTEVfVFlQRSI6WyJRVFRfREVUIl0sIkRBVEFTRVRfU09VUkNFIjpbIkRTX1NJRCJdfQ==

## Medications
Medication costs are stored in `src/main/resources/costs/medications.csv`.

| Code | Min | Mode | Max | Comments |
|:-----|:----|:-----|:----|:---------|
| The RxNorm code for the medication | The minimum cost, in USD. | The most common cost, in USD. | The maximum cost, in USD. | Any comments on how the cost was determined. This is only intended to be read by maintainers and not parsed by the system |

Where not specifically mentioned in the comments, cost data for medications is likely to have been sourced from publicly available Medicare data: https://data.cms.gov/Medicare-Part-D/Medicare-Provider-Utilization-and-Payment-Data-201/yvpj-pmj2
For example, this data source provides medication costs grouped by drug name and provider NPI. By grouping by drug name, dividing the `total_drug_cost` column by the `total_claim_count` column, and then averaging across providers, we obtain a rough estimate of the unit cost of that drug.

## Immunizations
Immunization costs are stored in `src/main/resources/costs/immunizations.csv`.

| Code | Min | Mode | Max | Comments |
|:-----|:----|:-----|:----|:---------|
| The CVX code for the immunization | The minimum cost, in USD. | The most common cost, in USD. | The maximum cost, in USD. | Any comments on how the cost was determined. This is only intended to be read by maintainers and not parsed by the system |



## Regional Adjustment Factors
Adjustment factors are stored in `src/main/resources/costs/adjustmentFactors.csv`

| State | Adj_Factor |
|:------|:-----------|
| The abbreviation of the state for which the factor applies | The amount to multiply costs taking place in this state by |

The initial values provided in this table are not intended to be comprehensive or authoritative. These were calculated using CMS urban and rural GAFs for 2011, weighted by the % of the population living in urban in rural areas by that state.