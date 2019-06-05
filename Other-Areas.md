By default, Synthea contains demographics, zip codes, providers, names, and costs for the entire United States, post-processed from publicly available files.

You can modify these files or have Synthea use alternative files by altering the `src/main/resources/synthea.properties` file:

```properties
# Abridged synthea.properties file
# Default demographics is every city in the US
generate.demographics.default_file = geography/demographics.csv
generate.geography.zipcodes.default_file = geography/zipcodes.csv
generate.geography.country_code = US

# Default provider files (1 of 10 files)
generate.providers.hospitals.default_file = providers/hospitals.csv

# Default costs, to be used for pricing something that we don't have a specific price for
generate.costs.default_procedure_cost = 500.00
generate.costs.default_medication_cost = 255.00
generate.costs.default_encounter_cost = 125.00
generate.costs.default_immunization_cost = 136.00
```

You can find detailed instructions on each of these file formats on the Synthea Wiki:

* **Demographics** file contains city-level distributions of Gender, Race, Age, Income, and Education which all play a role in health access, outcomes, and costs. https://github.com/synthetichealth/synthea/wiki/Demographics-for-Other-Areas
* **Zip or postal code** file contains postal codes and geographic locations (latitude and longitude) for each location. https://github.com/synthetichealth/synthea/wiki/Zip-or-Postal-Codes
* **Provider** files contains information and locations (latitude and longitude) for each provider location. https://github.com/synthetichealth/synthea/wiki/Provider-Data
* **Names** file contains language-specific family names and first names by gender. It also contains names suitable for street addresses. https://github.com/synthetichealth/synthea/wiki/Name-Data
* **Cost** files contain costs for different types of Encounters, Procedures, Medications, and Immunizations. https://github.com/synthetichealth/synthea/wiki/Cost-Data

### Abridged Example for Shrewsbury, Shropshire, United Kingdom

Districting and addresses in the United Kingdom differ from the United States. Let's start with the **demographics** file:

```csv
ID,COUNTY,NAME,STNAME,POPESTIMATE2015,CTYNAME,TOT_POP,TOT_MALE,TOT_FEMALE,WHITE,HISPANIC,BLACK,ASIAN,NATIVE,OTHER,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,00..10,10..15,15..25,25..35,35..50,50..75,75..100,100..150,150..200,200..999,LESS_THAN_HS,HS_DEGREE,SOME_COLLEGE,BS_DEGREE
1,1,Shrewsbury,West Midlands,2620,Shropshire,486300,0.489,0.511,0.985,0,0.001,0.004,0,0,0.063,0.063,0.063,0.052,0.052,0.052,0.052,0.052,0.076,0.076,0.076,0.076,0.076,0.035,0.035,0.035,0.035,0.035,0.133333333,0.133333333,0.133333333,0.147225,0.147225,0.147225,0.147225,0.1,0.01,0.001,0.18,0.387,22.35,22.35
```

Next, we need to create postal codes suitable to Shrewsbury, so we edit the **zip codes** file:

```csv
,USPS,ST,NAME,ZCTA5,LAT,LON
0,West Midlands,WMS,Shrewsbury,SY1,52.7081,-2.7549
1,West Midlands,WMS,Shrewsbury,SY2,52.7083,-2.7546
2,West Midlands,WMS,Shrewsbury,SY3,52.7089,-2.7543
```

Finally, we need to provide at least one hospital for our Salopians, so we edit the **hospitals** file:

```csv
,id,name,address,city,state,zip,county,phone,type,ownership,emergency,quality,LAT,LON
0,010001,Royal Shrewsbury Hospital,Mytton Oak Rd,Shrewsbury,WMS,SY3 8XQ,Shropshire,01743261000,NHS Teaching Hospital,Government - Hospital District or Authority,Yes,5,52.7091,-2.7931
```

For now, let's leave the **names** and **cost** files as-is. We're ready to generate data, but there is one last cosmetic change we need to make. In this example, we're generating the United Kingdom (UK), instead of the default United States (US). Let's change the `country_code` in `synthea.properties` so generated postal addresses look correct:

```properties
generate.geography.country_code = UK
```

Now, let's generate some data.

```
./run_synthea -s 0 -p 1 "West Midlands" Shrewsbury

...abridged...

Loaded 70 modules.
Running with options:
Population: 1
Seed: 0
Location: Shrewsbury, West Midlands
Min Age: 0
Max Age: 140
1 -- Brigitte394 Stark857 (50 y/o F) Shrewsbury, West Midlands DECEASED
1 -- Jeanine128 Gleason633 (57 y/o F) Shrewsbury, West Midlands 
{alive=1, dead=1}
```

Check the FHIR output of `Jeanine128 Gleason633`:

```json
...abridged...
        "address": [
          {
            "extension": [
              {
                "url": "http://hl7.org/fhir/StructureDefinition/geolocation",
                "extension": [
                  {
                    "url": "latitude",
                    "valueDecimal": -2.7546
                  },
                  {
                    "url": "longitude",
                    "valueDecimal": 52.7083
                  }
                ]
              }
            ],
            "line": [
              "560 Corkery Annex Suite 36"
            ],
            "city": "Shrewsbury",
            "state": "West Midlands",
            "postalCode": "SY1",
            "country": "UK"
          }
        ],
...abridged...
```

We did it! We've generated `Jeanine128 Gleason633` from Shrewsbury in the United Kingdom.

If you look at the FHIR data carefully, you'll notice a few oddities. First, Jeanine128 has a Social Security Number (SSN) and a phone number that looks suspiciously like it is in the United States. Also, the FHIR records do not use FHIR Profiles from NIH. So, localization in Synthea is not fully solved, and in particular profile conformance is a remaining challenge.