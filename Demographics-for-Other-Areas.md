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
 | The first column has no header. It is the number of the city. The program ignores this column.
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
`1` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`2` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`3` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`4` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`5` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`6` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`7` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`8` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`9` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`10` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`11` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`12` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`13` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`14` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`15` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`16` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`17` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
`18` | Age Group `0..4`. Percentage of the population in this age group. `0.0 - 1.0`
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