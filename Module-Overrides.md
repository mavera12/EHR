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
# SCREENSHOT OF APPENDICITIS MODULE HERE

The outcome of interest that we want here is the prevalence of the Appendicitis condition, so let's see which factors can affect that.
# SCREENSHOT
Working backwards from the Appendicitis state, there's a 1-3 day Delay, and a number of Symptom states. None of these directly impact prevalence. Next there are four possible Delays that impact when the person gets appendicitis. Technically this does impact prevalence (consider how the lifetime prevalence of a condition might change depending on whether it's more common among children or among the elderly) however we're not going to worry about that detail for now. Let's step a little further back and see the states labeled Male and Female. These states each contain a transition distribution where the patient goes down the appendicitis path, and a transition distribution where the patient goes to Terminal and does not get appendicitis. This is what we were looking for. This corresponds to the following 4 lines in the override file, and we can ignore the rest:
```
appendicitis.json\:\:$['states']['Male']['distributed_transition'][0]['distribution'] = 0.086
appendicitis.json\:\:$['states']['Male']['distributed_transition'][1]['distribution'] = 0.914
appendicitis.json\:\:$['states']['Female']['distributed_transition'][0]['distribution'] = 0.067
appendicitis.json\:\:$['states']['Female']['distributed_transition'][1]['distribution'] = 0.933
```
Now, let's take advantage of the fact that Synthea will automatically re-distribute transition percentages to add up to 100%, so we can also ignore the last distribution in each case, resulting in just 2 lines of interest:
```
appendicitis.json\:\:$['states']['Male']['distributed_transition'][0]['distribution'] = 0.086
appendicitis.json\:\:$['states']['Female']['distributed_transition'][0]['distribution'] = 0.067
```
It should be readily apparent how these two values align with the previously mentioned statistics that 8.6% of males and 6.7% of females get appendicitis. And so to update these to cut the numbers in half, we can do that directly, as follows:

```
appendicitis.json\:\:$['states']['Male']['distributed_transition'][0]['distribution'] = 0.043
appendicitis.json\:\:$['states']['Female']['distributed_transition'][0]['distribution'] = 0.034
```

Now we can save this as a new properties file, for instance `./appendicitis_overrides.properties`, and then run Synthea with these settings with:
```
run_synthea --module_override=./appendicitis_overrides.properties
```

Now when Synthea runs, it should produce a population where roughly 4.3% of males and 3.4% of females get Appendicitis. Confirmation of this can be done using the Prevalence Report.
# Link to the prevalence report

Obviously this was a very reduced example, but the basic premise holds for anything that you might want to override:
 - Identify the specific metric of interest
 - Locate where in the module(s) that metric is written to the record
 - Work backwards from there to the Initial state to find which parameters can affect that metric of interest
   - Note that this may involve cross-module interactions if the module of interest uses conditional logic based on attributes, active conditions, active medications, etc

# Examples
Some examples of pre-defined populations are available at https://github.com/synthetichealth/populations