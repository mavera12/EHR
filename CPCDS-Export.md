This feature is to allow data from Synthea to be exported as CPCDS files.  CPCDS is a data standard for exporting Common Payer Clinical Data Sets in a flat CSV file with repeating line numbers for multiple entries.  Its primary role is to assist in transferring insurance data into FHIR Explanation of Benefits. 

Use with Synthea is very simple.  Adding the tag `--exporter.cpcds.export true` to your command-line arguments will export the data in the CPCDS standard.  The output of the data will be in three files in the `output/cpcds` folder.  The CPCDS Patients.csv file will show patient data, coverage shows data related to health plans, and claims shows collective encounters with providers and any procedures, medications, and diagnoses that were given. 

# Configurations

`exporter.cpcds.export` 
Set to true to export as CPCDS. Default is false.

`exporter.cpcds.append_mode`
Set to true to add subsequent runs to a single file.  Keep as false to overwrite with new data each time it is run.

`exporter.cpcds.folder_per_run`
Set to true to output each run's data in its own folder.  Keep as false to keep all data in one folder.

# References

CPCDS Implementation Guide:
https://trifolia-fhir.lantanagroup.com/igs/lantana_hapi_r4/carin-bb/toc.html