# Let's Build a Synthetic Dataset

In a classroom setting, the instructor will walk you through the process of using, configuring, and customizing SyntheaTM – an open-source synthetic patient generator – to create a patient and provider cohort and all the FHIR DSTU2, STU3, or R4 resources you need for your next software development, testing or integration event…

## Prerequisites

* Git
  * https://git-scm.com/downloads 
* Java JDK 8, 11, 12
  * https://adoptopenjdk.net

## Clone and Build the Repository

```
git clone https://github.com/synthetichealth/synthea.git
cd synthea
./gradlew test
```

Estimate 5-10 minutes

## Browse the Diseases

Browse the diseases in the [Module Builder](https://synthetichealth.github.io/module-builder/)

The instructor will make a change to the CHF module. Follow along.

Download the JSON, and overwrite your current module with the changes.

```
cp ~/Downloads/congestive_heart_failure.json ./src/main/resources/modules/congestive_heart_failure.json
./run_synthea -p 10
```

When finished, undo the changes:

```
git checkout -- src
```

## Run Synthea

```
./run_synthea –s 0
```

Look at the FHIR record in `./output/fhir`

## Configuring Properties

Examine the configurable properties in the [synthea.properties](https://github.com/synthetichealth/synthea/wiki/Common-Configuration) file.

## POST our patient record to a server

```
curl http://hapi.fhir.org/baseR4 --data-binary "@/Users/Path/synthea/output/fhir/Brigitte394_Jaskolski867_f45fe01f-de94-4587-97f0-4cbd39f775c5.json" -H "Content-Type: application/fhir+json" 
```

## Generate a lot of data
```
./run_synthea -p 10000 Washington
```

Estimate 5-10 minutes and 5-10 GB storage

## POST all the data to a server
```
cd output/fhir/
for file in *; do curl --write-out '.' http://hapi.fhir.org/baseR4 --data-binary "@$file" -H "Content-Type: application/fhir+json" > /dev/null; done;
```
Estimate: long time...
