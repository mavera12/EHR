# Setup Synthea<sup>TM</sup>
## Prerequisites
 - Git
 - Java 1.8 (select JDK, not JRE install)

To copy the repository locally and install the necessary dependencies, open a terminal window and run the following commands:

```
git clone https://github.com/synthetichealth/synthea.git
cd synthea
./gradlew build check test
```
**Note: if running on Windows, use `.\gradlew.bat` instead of `./gradlew` -- this guide uses `./gradlew` for brevity**

# Running Synthea<sup>TM</sup>

The primary entry point of Synthea is the `run` gradle task, which can be run from the terminal in the `synthea` folder by running the following command.

```
./gradlew run
```

Alternatively, you may run the provided `run_synthea` script which makes it easier to pass command-line arguments to the simulation:

```
./run_synthea
```
**Note: if running on Windows, use `.\run_synthea.bat` instead of `./run_synthea` -- this guide uses `./run_synthea` for brevity**

When you run this command, you should see output similar to the following:

```
example@hostname ~/synthea $ ./run_synthea

> Task :run
Loading C:\Users\example\synthea\build\resources\main\modules\allergic_rhinitis.json
Loading C:\Users\example\synthea\build\resources\main\modules\allergies\allergy_incidence.json
[... many more lines of Loading ...]
Loading C:\Users\example\synthea\build\resources\main\modules\wellness_encounters.json
Loaded 68 modules.
Running with options:
Population: 1
Seed: 1519063214833
Location: Massachusetts

1 -- Jerilyn993 Parker433 (10 y/o) Lawrence, Massachusetts
```

This command takes additional parameters to specify different regions or common run options. Any options not specified are left at the default value.

```
run_synthea [-h]
            [-s seed] 
            [-p populationSize]
            [-g gender]
            [-a minAge-maxAge]
            [-m moduleFilter]
            [state [city]]
```

Some examples:

 -   `run_synthea -h` -- output help messages and command line options and quit without further processing
 -   `run_synthea Massachusetts` -- to generate a population in all cities and towns in Massachusetts
 -   `run_synthea Alaska Juneau` -- to generate a population in only Juneau, Alaska
 -   `run_synthea -s 12345` -- to generate a population using seed 12345. Populations generated with the same seed and the same version of Synthea should be identical
 -   `run_synthea -p 1000` -- to generate a population of 1000 patients
 -   `run_synthea -a 30-40` -- to generate a population of 30 to 40 year olds
 -   `run_synthea -g F` -- to generate only female patients
 -   `run_synthea -s 987 Washington Seattle` -- to generate a population in only Seattle, Washington, using seed 987
 -   `run_synthea -s 21 -p 100 Utah "Salt Lake City"` -- to generate a population of 100 patients in Salt Lake City, Utah, using seed 21
 -   `run_synthea -m metabolic*` -- generate a population using only modules matching the `metabolic*` name filter. All core modules (e.g. lifecycle) and shared submodules are automatically included.
 -   `run_synthea --exporter.fhir.use_us_core_ig true` -- generate a population that exports FHIR according to the US Core R4 Implementation Guide profiles.
**[Next: Common Configuration](https://github.com/synthetichealth/synthea/wiki/Common-Configuration)**