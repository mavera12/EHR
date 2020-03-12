As of [PR #566](https://github.com/synthetichealth/synthea/pull/566), Synthea includes the ability for to use "overrides" that change settings in the modules on a per-run basis. These overrides may tweak the prevalence of conditions, affect how patients flow through a module, or impact the simulation in ways we haven't even thought of yet.

For now we envision two primary use cases for overriding modules:
1) "Internationalization", for example, updating a module so that it generates a population with different condition prevalence rates than as designed, such as for a different country or subpopulation.
2) Enabling other utilities to tweak aspects of the modules without having to understand the module format directly.

**IMPORTANT**: One major limitation of module overrides as of today is that they, along with the modules themselves, are not versioned. This means that there's no way to associate a set of overrides with a specific version of the modules, so as the modules change over time, it's necessary to manually review any overrides to ensure they are still working as intended.

## Usage

### Using an Overrides File

```
run_synthea --module_override=(path_to_properties_file)
```

### Generating Overrides
The way the override generation process is essentially a depth-first search of the modules as trees, where the leaf nodes are JSON primitive fields (strings, numbers, booleans, nulls). Fields that contain a numeric value and whose name matches one from a list of names are written out to the properties file. By default the only field included is "distribution", which is the probability of taking a specific transition within a distributed or complex transition, but other fields can be chosen as well.

Command to run:
```
gradlew overrides [-PincludeFields=...] [-PexcludeFields=...] [-PincludeModules=...] [-PexcludeModules=...]
```
With parameters: (all optional)
- `includeFields`
  - numeric fields in the modules that match one of the given names will be written to the properties file (defaults to "distribution" if not provided ) 
- `excludeFields`
    - if provided, all numeric fields **except** those that match the given names
       will be written to the properties file
    - note: if both includeFields and excludeFields are given, includeFields will be ignored
- `includeModules`   
    - if provided, only modules that match the given file names will be processed
    - wildcards are allowed, eg: `"metabolic*"`
- `excludeModules`  
    - if provided, all modules except those that match the given file names will be processed
    - wildcards are allowed, eg: `"metabolic*"`



### Format
The overrides file is a standard Java properties file, with basic structure:
```
(module file path)::(jsonpath to value) = (new value)
```

For instance, a snippet of a file might look like this:
```
congestive_heart_failure.json\:\:$['states']['Age_70-79-Chance\ of\ CHF']['distributed_transition'][0]['distribution'] = 0.024
congestive_heart_failure.json\:\:$['states']['Age_80-Chance\ of\ CHF']['distributed_transition'][0]['distribution'] = 0.045
atopy.json\:\:$['states']['Initial']['complex_transition'][0]['distributions'][0]['distribution'] = 0.917
atopy.json\:\:$['states']['Initial']['complex_transition'][1]['distributions'][0]['distribution'] = 0.033
```


## Full Example
Let's say we want to update the Appendicitis module such that a reduced percent of the population gets appendicitis. For instance, South Africa and other African countries have a much lower rate than the US. As currently defined, 8.6% of males and 6.7% of females get appendicitis. Let's cut these numbers in half and set it so that 4.3% of males and 3.4% of females get appendicitis. Rather than update the module directly, let's create and use a module override file.

### Step 1. Create an override file for the module
To create the baseline override file for this module, we run the following command
```
gradlew overrides -PincludeModules=appendicitis*
```

This produces a file at `./output/overrides.properties` with the following content:
```
appendicitis.json\:\:$['states']['Male']['distributed_transition'][0]['distribution'] = 0.086
appendicitis.json\:\:$['states']['Male']['distributed_transition'][1]['distribution'] = 0.914
appendicitis.json\:\:$['states']['Female']['distributed_transition'][0]['distribution'] = 0.067
appendicitis.json\:\:$['states']['Female']['distributed_transition'][1]['distribution'] = 0.933
appendicitis.json\:\:$['states']['Pre_appendicitis']['distributed_transition'][0]['distribution'] = 0.263
appendicitis.json\:\:$['states']['Pre_appendicitis']['distributed_transition'][1]['distribution'] = 0.423
appendicitis.json\:\:$['states']['Pre_appendicitis']['distributed_transition'][2]['distribution'] = 0.221
appendicitis.json\:\:$['states']['Pre_appendicitis']['distributed_transition'][3]['distribution'] = 0.093
appendicitis.json\:\:$['states']['Appendicitis']['distributed_transition'][0]['distribution'] = 0.7
appendicitis.json\:\:$['states']['Appendicitis']['distributed_transition'][1]['distribution'] = 0.3
```

There's a lot here, and not all of it is necessarily relevant to what we care about. Let's take a look at the module:

The outcome of interest that we want here is the prevalence of 