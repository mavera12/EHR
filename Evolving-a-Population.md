# Evolving a Population

Synthea has the ability to save a snapshot of a set of records, reload a snapshot, and evolve a snapshot for a specified period of time.

Three additional command line arguments control this functionality:

```
run_synthea [options] [state [city]]
Options: [-u updatedPopulationSnapshotPath]
         [-i initialPopulationSnapshotPath]
         [-t updateTimePeriodInDays]
```

Example usage is shown below:

```sh
$ mkdir output/1 output/2 output/3
$ ./run_synthea -p 5 -u output/1/snap --exporter.baseDirectory output/1
$ ./run_synthea -i output/1/snap -u output/2/snap -t 100 --exporter.baseDirectory output/2
$ ./run_synthea -i output/2/snap -u output/3/snap -t 100 --exporter.baseDirectory output/3
```

The above sequence of commands:

1. Creates a set of output directories
2. Creates an initial set of 5 records and saves a snapshot to `output/1/snap`
3. Loads the snapshot from `output/1/snap` continues the simulation for 100 days and saves an updated snapshot to `output/2/snap`
4. Loads the snapshot from `output/2/snap` continues the simulation for another 100 days and saves an updated snapshot to `output/3/snap`

Steps 2, 3 and 4 use the `--export.baseDirectory` to specify a different output directory for each step. If this is not done then the export will fail since the records will already exist.

The above sequence of commands results in the following set of output files:

```sh
output/1/snap
output/1/fhir/
    Dominique369_Kirlin939_b7c98c3d-d948-46f1-85eb-c26568aa15bf.json 
    Verdell500_Renner328_fc4a3b26-e0c6-4215-a9aa-1f15edfe4208.json
    Harley673_Heller342_dd96e3c8-fb6c-4371-849c-3f9c31b19644.json
    Kum811_Wintheiser220_32b8d335-473a-4df9-86ae-b75daed3f3ea.json
    Silva841_Hamill307_efe4f638-b8df-48e4-a05d-fc2f2c9e1857.json
output/2/snap
output/2/fhir/
    Dominique369_Kirlin939_b7c98c3d-d948-46f1-85eb-c26568aa15bf.json 
    Verdell500_Renner328_fc4a3b26-e0c6-4215-a9aa-1f15edfe4208.json
    Harley673_Heller342_dd96e3c8-fb6c-4371-849c-3f9c31b19644.json
    Kum811_Wintheiser220_32b8d335-473a-4df9-86ae-b75daed3f3ea.json
    Silva841_Hamill307_efe4f638-b8df-48e4-a05d-fc2f2c9e1857.json
output/3/snap
output/3/fhir/
    Dominique369_Kirlin939_b7c98c3d-d948-46f1-85eb-c26568aa15bf.json 
    Verdell500_Renner328_fc4a3b26-e0c6-4215-a9aa-1f15edfe4208.json
    Harley673_Heller342_dd96e3c8-fb6c-4371-849c-3f9c31b19644.json
    Kum811_Wintheiser220_32b8d335-473a-4df9-86ae-b75daed3f3ea.json
    Silva841_Hamill307_efe4f638-b8df-48e4-a05d-fc2f2c9e1857.json
```

## Limitations

Currently the entire set of records has to be stored in memory prior to snapshotting. This means that the size of the population is limited by hardware resources.