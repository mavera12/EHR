# Let's Build a Synthetic Dataset

In a classroom setting, the instructor will walk you through the process of using, configuring, and customizing Synthea<sup>TM</sup> – an open-source synthetic patient generator – to create a patient and provider cohort and all the FHIR DSTU2, STU3, or R4 resources you need for your next software development, testing or integration event…

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
Estimate: long time... too long.

## Upload all the data to Microsoft Azure

Zip your `./output/fhir` folder into `fhir_{p}_{state}_{town}.zip`.

Get the `azcopy` utility to upload your data to Microsoft Azure:

- [Windows (zip)](https://aka.ms/downloadazcopy-v10-windows)
- [Linux (tar)](https://aka.ms/downloadazcopy-v10-linux)
- [MacOS (zip)](https://aka.ms/downloadazcopy-v10-mac)

Unpack the download to use the `azcopy` executable -- no installation should be required.

Copy your file to our Azure storage:

```
azcopy cp "output/fhir_{p}_{state}_{town}.zip" "https://syntheadevdays2019.blob.core.windows.net/synthea/?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-06-22T04:32:42Z&st=2019-06-10T20:32:42Z&spr=https&sig=7mG96EHAJ1jIVlShwxxui5g74%2F%2F6enrCXjCx%2BteM0k0%3D" --recursive=true
```

List all the data files:
```
azcopy list https://syntheadevdays2019.blob.core.windows.net/synthea
```

Download a particular data `{file}`:
```
azcopy cp https://syntheadevdays2019.blob.core.windows.net/synthea/{file} .
```