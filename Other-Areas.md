> Configuration files for non-US locations can be found in the [Synthea-International](https://github.com/synthetichealth/synthea-international) repository. If you create configuration files for other locations, please consider submitting those to the Synthea-International repository.

By default, Synthea contains demographics, zip codes, providers, names, and costs for the entire United States, post-processed from publicly available files.

You can modify these files or have Synthea use alternative files by altering the `src/main/resources/synthea.properties` file:

```properties
# Abridged synthea.properties file
# Default demographics is every city in the US
generate.demographics.default_file = geography/demographics.csv
generate.geography.zipcodes.default_file = geography/zipcodes.csv
generate.geography.country_code = US
generate.geography.timezones.default_file = geography/timezones.csv

# Default provider files (1 of 10 files)
generate.providers.hospitals.default_file = providers/hospitals.csv
generate.providers.primarycare.default_file = providers/primary_care_facilities.csv

# Payers
generate.payers.insurance_companies.default_file = payers/insurance_companies.csv
generate.payers.insurance_companies.medicare = Medicare
generate.payers.insurance_companies.medicaid = Medicaid
generate.payers.insurance_companies.dual_eligible = Dual Eligible

# Default costs, to be used for pricing something that we don't have a specific price for
generate.costs.default_procedure_cost = 500.00
generate.costs.default_medication_cost = 255.00
generate.costs.default_encounter_cost = 125.00
generate.costs.default_immunization_cost = 136.00
```

You can find detailed instructions on each of these file formats on the Synthea Wiki:

* **Demographics** file contains city-level distributions of Gender, Race, Age, Income, and Education which all play a role in health access, outcomes, and costs. https://github.com/synthetichealth/synthea/wiki/Demographics-for-Other-Areas
* **Zip or postal code** file contains postal codes and geographic locations (latitude and longitude) for each location. https://github.com/synthetichealth/synthea/wiki/Zip-or-Postal-Codes
* **Time Zones** file contains timezone information for each state (or equivalent).
* **Provider** files contains information and locations (latitude and longitude) for each provider location. https://github.com/synthetichealth/synthea/wiki/Provider-Data
* **Names** file contains language-specific family names and first names by gender. It also contains names suitable for street addresses. https://github.com/synthetichealth/synthea/wiki/Name-Data
* **Cost** files contain costs for different types of Encounters, Procedures, Medications, and Immunizations. https://github.com/synthetichealth/synthea/wiki/Cost-Data

# Abridged Example for Shrewsbury, Shropshire, United Kingdom

Configuration files for this example can be found in the [Synthea-International](https://github.com/synthetichealth/synthea-international) repository, in the `example` folder. If you create configuration files for other locations, please consider submitting those to the Synthea-International repository.

## Demographic Data
Districting and addresses in the United Kingdom differ from the United States. Let's start with the **demographics** file:

### Demographics File
```csv
ID,COUNTY,NAME,STNAME,POPESTIMATE2015,CTYNAME,TOT_POP,TOT_MALE,TOT_FEMALE,WHITE,HISPANIC,BLACK,ASIAN,NATIVE,OTHER,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,00..10,10..15,15..25,25..35,35..50,50..75,75..100,100..150,150..200,200..999,LESS_THAN_HS,HS_DEGREE,SOME_COLLEGE,BS_DEGREE
1,1,Shrewsbury,West Midlands,2620,Shropshire,486300,0.489,0.511,0.985,0,0.001,0.004,0,0,0.063,0.063,0.063,0.052,0.052,0.052,0.052,0.052,0.076,0.076,0.076,0.076,0.076,0.035,0.035,0.035,0.035,0.035,0.133333333,0.133333333,0.133333333,0.147225,0.147225,0.147225,0.147225,0.1,0.01,0.001,0.18,0.387,22.35,22.35
```

## Geographic Data
Now that we have provided new demographic data, we need to provide geographic data for the area we are modeling. So, we need to create postal codes suitable to Shrewsbury, so we edit the **zip codes** file:

### Zipcodes File
```csv
,USPS,ST,NAME,ZCTA5,LAT,LON
0,West Midlands,WMS,Shrewsbury,SY1,52.7081,-2.7549
1,West Midlands,WMS,Shrewsbury,SY2,52.7083,-2.7546
2,West Midlands,WMS,Shrewsbury,SY3,52.7089,-2.7543
```

And then we also need to set the timezone for each "state" (or equivalent):

### Timezones File
```csv
STATE,ST,TIMEZONE,TZ
West Midlands,WMS,Greenwich Mean Time,GMT
```

## Provider Data
Next, we need to provide at least one hospital for our Salopians, so we edit the **hospitals** file and the **primarycare** file:

### Hospitals File
```csv
,id,name,address,city,state,zip,county,phone,type,ownership,emergency,quality,LAT,LON
0,010001,Royal Shrewsbury Hospital,Mytton Oak Rd,Shrewsbury,WMS,SY3 8XQ,Shropshire,01743261000,NHS Teaching Hospital,Government - Hospital District or Authority,Yes,5,52.7091,-2.7931
```

### Primary Care File
```csv
name,address,city,state,zip,phone,LAT,LON,hasSpecialties,ADDICTION MEDICINE,ADVANCED HEART FAILURE AND TRANSPLANT CARDIOLOGY,ALLERGY/IMMUNOLOGY,ANESTHESIOLOGY,ANESTHESIOLOGY ASSISTANT,AUDIOLOGIST,CARDIAC ELECTROPHYSIOLOGY,CARDIAC SURGERY,CARDIOVASCULAR DISEASE (CARDIOLOGY),CERTIFIED NURSE MIDWIFE,CERTIFIED REGISTERED NURSE ANESTHETIST,CHIROPRACTIC,CLINICAL NURSE SPECIALIST,CLINICAL PSYCHOLOGIST,CLINICAL SOCIAL WORKER,COLORECTAL SURGERY (PROCTOLOGY),CRITICAL CARE (INTENSIVISTS),DENTIST,DERMATOLOGY,DIAGNOSTIC RADIOLOGY,EMERGENCY MEDICINE,ENDOCRINOLOGY,FAMILY PRACTICE,GASTROENTEROLOGY,GENERAL PRACTICE,GENERAL SURGERY,GERIATRIC MEDICINE,GERIATRIC PSYCHIATRY,GYNECOLOGICAL ONCOLOGY,HAND SURGERY,HEMATOLOGY,HEMATOLOGY/ONCOLOGY,HEMATOPOIETIC CELL TRANSPLANTATION AND CELLULAR TH,HOSPICE/PALLIATIVE CARE,HOSPITALIST,INFECTIOUS DISEASE,INTERNAL MEDICINE,INTERVENTIONAL CARDIOLOGY,INTERVENTIONAL PAIN MANAGEMENT,INTERVENTIONAL RADIOLOGY,MAXILLOFACIAL SURGERY,MEDICAL ONCOLOGY,NEPHROLOGY,NEUROLOGY,NEUROPSYCHIATRY,NEUROSURGERY,NUCLEAR MEDICINE,NURSE PRACTITIONER,OBSTETRICS/GYNECOLOGY,OCCUPATIONAL THERAPY,OPHTHALMOLOGY,OPTOMETRY,ORAL SURGERY,ORTHOPEDIC SURGERY,OSTEOPATHIC MANIPULATIVE MEDICINE,OTOLARYNGOLOGY,PAIN MANAGEMENT,PATHOLOGY,PEDIATRIC MEDICINE,PERIPHERAL VASCULAR DISEASE,PHYSICAL MEDICINE AND REHABILITATION,PHYSICAL THERAPY,PHYSICIAN ASSISTANT,PLASTIC AND RECONSTRUCTIVE SURGERY,PODIATRY,PREVENTATIVE MEDICINE,PSYCHIATRY,PULMONARY DISEASE,RADIATION ONCOLOGY,REGISTERED DIETITIAN OR NUTRITION PROFESSIONAL,RHEUMATOLOGY,SLEEP MEDICINE,SPEECH LANGUAGE PATHOLOGIST,SPORTS MEDICINE,SURGICAL ONCOLOGY,THORACIC SURGERY,UNDEFINED PHYSICIAN TYPE (SPECIFY),UROLOGY,VASCULAR SURGERY,id
Royal Shrewsbury Hospital,Mytton Oak Rd,Shrewsbury,WMS,SY3 8XQ,01743261000,52.7091,-2.7931,False,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,PCP1
```

## Payer Data
We are almost done. We want to make sure our Salopians get to take advantage of the National Health Service. So we need to edit a few more parameters.

```properties
# Abridged synthea.properties file
# Payers
generate.payers.insurance_companies.default_file = payers/insurance_companies.csv
generate.payers.insurance_companies.medicare = National Health Service
generate.payers.insurance_companies.medicaid = National Health Service
generate.payers.insurance_companies.dual_eligible = National Health Service
```

And the **insurance companies** file:
```csv
,id,name,address,city,state_headquartered,zip,phone,states_covered,services_covered,deductible,default_coinsurance,default_copay,monthly_premium,ownership
0,10001,National Health Service,Richmond House 79 Whitehall,Westminster,Greater London,SW1A 2NS,111,*,*,0,0.0,0,0,Government
```

## Wrap Up

For now, let's leave the **names** and **cost** files as-is. We're ready to generate data, but there is one last cosmetic change we need to make. In this example, we're generating the United Kingdom (UK), instead of the default United States (US). Let's change the `country_code` in `synthea.properties` so generated postal addresses look correct:

```properties
generate.geography.country_code = UK
```

Now, let's generate some data.

```
./run_synthea -p 1 "West Midlands" Shrewsbury

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

Check the FHIR output of `Jeanine128 Gleason633` (you might have a different patient):

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