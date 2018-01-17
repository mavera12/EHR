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


### Editing the Demographics File

# WIP
