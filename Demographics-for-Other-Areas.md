By default, the project contains demographics for the entire United States in `resources/demographics.csv` which has been post-processed from publicly available US Census Bureau files.

### Generating a State or City
For example, you can generate synthetic patients for the state of New York:
```
bundle exec rake synthea:generate_state['New York']
```

Alternatively, you can generate a single city:
```
bundle exec rake synthea:generate_city['Bedford','Massachusetts']
```

### Scaling the Population
If you crack open the `config/synthea.yml` file you can generate the entire population of each city, or a percentage of the population.  Set `synthea.sequential.population_scaling` to `true` to generate scaled portion of the population, or `false` to generate everyone in the city. You can set the scale factor in `synthea.sequential.population_scaling_rate`.


### Demographics File Format

The demographics file is a CSV file, with a single row for each city.

Column Header | Description
--------------|------------
` ` | The first column has no header. It is the number of the city. The program ignores this column.
`COUNTY` | County number. Ignored.
`NAME` | City Name
`STNAME` | State Name
`POPESTIMATE2015` | City Population
`CTYNAME` | County Name
`TOT_POP` | County Population
`TOT_MALE` | Percentage of the population that is Male. `TOT_MALE` and `TOT_FEMALE` should sum to `1.0`. Legal values: `0.0 - 1.0`
`TOT_FEMALE` | Percentage of the population that is Female. `TOT_MALE` and `TOT_FEMALE` should sum to `1.0`. Legal values: `0.0 - 1.0`
`WHITE` | Percentage of the population that is White. `0.0 - 1.0`
`HISPANIC` | Percentage of the population that is ethnically Hispanic. `0.0 - 1.0`
`BLACK` | Percentage of the population that is Black. `0.0 - 1.0`
`ASIAN` | Percentage of the population that is Asian. `0.0 - 1.0`
`NATIVE` | Percentage of the population that is Native or Indigenous peoples. `0.0 - 1.0`
`OTHER` | Percentage of the population that does not fit into the other racial categories. `0.0 - 1.0`
`1` | Age Group `0..4`. Number of people within the COUNTY in this Age Group.
`2` | Age Group `5..9`. Number of people within the COUNTY in this Age Group.
`3` | Age Group `10..14`. Number of people within the COUNTY in this Age Group.
`4` | Age Group `15..19`. Number of people within the COUNTY in this Age Group.
`5` | Age Group `20..24`. Number of people within the COUNTY in this Age Group.
`6` | Age Group `25..29`. Number of people within the COUNTY in this Age Group.
`7` | Age Group `30..34`. Number of people within the COUNTY in this Age Group.
`8` | Age Group `35..39`. Number of people within the COUNTY in this Age Group.
`9` | Age Group `40..44`. Number of people within the COUNTY in this Age Group.
`10` | Age Group `45..49`. Number of people within the COUNTY in this Age Group.
`11` | Age Group `50..54`. Number of people within the COUNTY in this Age Group.
`12` | Age Group `55..59`. Number of people within the COUNTY in this Age Group.
`13` | Age Group `60..64`. Number of people within the COUNTY in this Age Group.
`14` | Age Group `65..69`. Number of people within the COUNTY in this Age Group.
`15` | Age Group `70..74`. Number of people within the COUNTY in this Age Group.
`16` | Age Group `75..79`. Number of people within the COUNTY in this Age Group.
`17` | Age Group `80..84`. Number of people within the COUNTY in this Age Group.
`18` | Age Group `85..110`. Number of people within the COUNTY in this Age Group.
`00..10` | Annual Income Group $0 USD - $10K USD. Percentage of population. `0.0 - 1.0`
`10..15` | Annual Income Group $10K USD - $15K USD. Percentage of population. `0.0 - 1.0`
`15..25` | Annual Income Group $15K 1USD - $25K USD. Percentage of population. `0.0 - 1.0`
`25..35` | Annual Income Group $25K USD - $35K USD. Percentage of population. `0.0 - 1.0`
`35..50` | Annual Income Group $35K USD - $50K USD. Percentage of population. `0.0 - 1.0`
`50..75` | Annual Income Group $50K USD - $75K USD. Percentage of population. `0.0 - 1.0`
`75..100` | Annual Income Group $75K USD - $100K USD. Percentage of population. `0.0 - 1.0`
`100..150` | Annual Income Group $100K USD - $150K USD. Percentage of population. `0.0 - 1.0`
`150..200` | Annual Income Group $150K USD - $200K USD. Percentage of population. `0.0 - 1.0`
`200..999` | Annual Income Group $200K USD - $999K USD. Percentage of population. `0.0 - 1.0`
`LESS_THAN_HS` | Education Group with Less than High School education. Percentage `0.0 - 1.0`
`HS_DEGREE` | Education Group with a High School equivalent education. Percentage `0.0 - 1.0`
`SOME_COLLEGE` | Education Group with Some College education. Percentage `0.0 - 1.0`
`BS_DEGREE` | Education Group with a Bachelors Degree or Higher education (includes PhD, JD, MD). Percentage `0.0 - 1.0`

The following columns in each row should sum to `1.0`
* Gender: `TOT_MALE`,`TOT_FEMALE`
* Race: `WHITE`,`HISPANIC`,`BLACK`,`ASIAN`,`NATIVE`,`OTHER`
* Age: `1`,`2`,`3`,`4`,`5`,`6`,`7`,`8`,`9`,`10`,`11`,`12`,`13`,`14`,`15`,`16`,`17`,`18`
* Income: `00..10`,`10..15`,`15..25`,`25..35`,`35..50`,`50..75`,`75..100`,`100..150`,`150..200`,`200..999`
* Education: `LESS_THAN_HS`,`HS_DEGREE`,`SOME_COLLEGE`,`BS_DEGREE`

Gender, Race, Age, Income, and Education all play a role in health access, outcomes, and costs.

Synthea uses Gender, Race, and Age in some disease modules because these are factors in disease prevalence and incidence rates.
Synthea uses Income and Education to determine a socioeconomic status. The socioeconomic calculation is here: https://github.com/synthetichealth/synthea/blob/00bd0928d353e6d713d9d4f6d1a3163a1d5103b6/lib/modules/lifecycle.rb#L460

## Examples for Other Areas

These are incomplete partial examples for illustration purposes.

### United Kingdom
Districting and addresses in the United Kingdom differ from the United States. Here, the County is entered in the `STNAME` column and the District is entered in the `CTYNAME`.

```csv
,COUNTY,NAME,STNAME,POPESTIMATE2015,CTYNAME,TOT_POP,TOT_MALE,TOT_FEMALE,WHITE,HISPANIC,BLACK,ASIAN,NATIVE,OTHER,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,00..10,10..15,15..25,25..35,35..50,50..75,75..100,100..150,150..200,200..999,LESS_THAN_HS,HS_DEGREE,SOME_COLLEGE,BS_DEGREE
1,1,Shrewsbury,Shropshire,2620,West Midlands,486300,0.489,0.511,0.985,0,0.001,0.004,0,0,0.063,0.063,0.063,0.052,0.052,0.052,0.052,0.052,0.076,0.076,0.076,0.076,0.076,0.035,0.035,0.035,0.035,0.035,0.133333333,0.133333333,0.133333333,0.147225,0.147225,0.147225,0.147225,0.1,0.01,0.001,0.18,0.387,22.35,22.35
```

### Netherlands