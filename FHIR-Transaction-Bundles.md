Synthea can be configured to export FHIR transaction bundles using the following configuration option:

```
exporter.fhir.transaction_bundle = true
```

When this option is used, the patient transaction bundles will not include `Practitioner`, `Organization` or `Location` resources to avoid creation of duplicates when importing more than one patient bundle into a FHIR server.

Resources within the patient bundles will refer to these omitted resources using [query references](https://www.hl7.org/fhir/http.html#trules) of the form:

```
Practitioner?identifier=http://hl7.org/fhir/sid/us-npi|999999647
```

A FHIR server will attempt to resolve these query references while processing the transaction bundle but, for this to be successful, the server will need to already hold matching resources.

To this end, Synthea can be additionally configured to export separate bundles for:

1. `Practitioner` resources: set `exporter.practitioner.fhir.export = true` (FHIR R4), `exporter.practitioner.fhir_dstu2.export = true` (DSTU2), or `exporter.practitioner.fhir_stu3.export = true` (STU3)

2. `Organization` and `Location` resources: set `exporter.hospital.fhir.export = true` (FHIR R4), `exporter.hospital.fhir_dstu2.export = true` (DSTU2), or `exporter.hospital.fhir_stu3.export = true` (STU3)

These bundles use [`ifNoneExist` preconditions](http://hl7.org/fhir/DSTU2/http.html#ccreate) to avoid creating duplicate resources when loaded into a FHIR server so it should be safe (provided the FHIR server supports preconditions) to load these bundles as needed. Note that these bundles will only include the resources referenced by the patient bundles exported at the same time, so, if a second run of Synthea is performed to create addition patient bundles, the corresponding `Practitioner` and `Organization` bundles will need to be preloaded prior to loading the patient bundles.

Here is the recommended sequence for creating and loading FHIR transaction bundles:

1. Configure Synthea with the following properties:
  - `exporter.fhir.transaction_bundle = true`
  - `exporter.practitioner.fhir.export = true`
  - `exporter.hospital.fhir.export = true`
2. `run_sythea ...`
3. Upload the `output/fhir/hospitalinformation.....json` bundle
4. Upload the `output/fhir/practitionerinformation.....json` bundle
5. Upload each patient bundle

Synthea will emit a warning if transaction bundles are enabled without also enabling practitioner and organization bundles.