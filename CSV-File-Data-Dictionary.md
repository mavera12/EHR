Synthea can be configured to export data as CSV into `./output/csv`.

To export CSV, open `./src/main/resources/synthea.properties` and change this setting:

```
exporter.csv.export = true
```

After running Synthea, the CSV exporter will create nine files:
