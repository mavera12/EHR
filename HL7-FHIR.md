Synthea generates HL7 FHIR records using the [HAPI FHIR](http://hapifhir.io/) library to generate a FHIR `Bundle` for each `Patient`. Currently, only JSON is supported.

Currently supported FHIR Resources:
- `Bundle`
- `Patient`
- `Encounter`
- `Condition`
- `AllergyIntolerance`
- `Observation`
- `DiagnosticReport`
- `Procedure`
- `ImagingStudy`
- `Immunization`
- `CarePlan`
- `MedicationRequest`

### Configuring FHIR

The exporting of FHIR can be configured using the `src/main/resources/synthea.properties`:

```properties
# default FHIR configuration.
exporter.fhir.export = true
# transaction bundle 'true' produces transaction Bundles
# while 'false' produces collection Bundles.
exporter.fhir.transaction_bundle = true
# Standard Health Record (SHR) extensions for STU3
exporter.fhir.use_shr_extensions = true
# Exporting FHIR DSTU2
exporter.fhir_dstu2.export = false
# Exporting Hospital Provider Data in STU3 or DSTU2
exporter.hospital.fhir.export = true
exporter.hospital.fhir_dstu2.export = false
```

### Producing and Using FHIR

In the `synthea.properties` file, make sure that `exporter.fhir.export`, `exporter.fhir.transaction_bundle`, and `exporter.hospital.fhir.export` are all set to `true`.

Next, produce some synthetic data:
```
./run_synthea -p 100
```

By default, FHIR JSON data should now exist in the `./output/fhir` sub-directory. If you have a FHIR server and client, you can `POST` those bundles to the FHIR server. For example:

```
curl http://hapi.fhir.org/baseDstu3 
  --data-binary "@/Users/example/synthea/output/fhir/Maryetta775_Rowe323_2cb7e4dd-9d8b-49cf-b1e4-9839be8bc754.json" 
  -H "Content-Type: application/fhir+json"
```

The FHIR server should return a `transaction-response` `Bundle`:
```json
{
  "resourceType": "Bundle",
  "id": "ca4d459f-b078-4c04-a152-c7ce76a25179",
  "type": "transaction-response",
  "link": [
    {
      "relation": "self",
      "url": "http://hapi.fhir.org/baseDstu3"
    }
  ],
  "entry": [
    {
      "response": {
        "status": "201 Created",
        "location": "Patient/4147259/_history/1",
        "etag": "1",
        "lastModified": "2018-06-06T18:26:06.038+00:00"
      }
    },
    ...abridged...
  ]
}
```

### Other Examples of Using Synthea FHIR Data

* The [Healthcare Services Platform Consortium (HSPC)](https://www.hspconsortium.org) has a [developer sandbox environment](https://sandbox.hspconsortium.org/#/login) that allows developers to spin-up FHIR servers and load them with Synthea data. They also host a FHIR server preloaded with Synthea data called `HSPC Synthea STU3 (3.0.1)` (authentication required).

* The [SMART Health IT](https://smarthealthit.org) team has [pregenerated datasets](http://docs.smarthealthit.org/data/stu3-sandbox-data.html) available for download including Synthea data, which are also available through https://dev.smarthealthit.org. They also provide a Docker version of the HAPI FHIR Server preloaded with Synthea data here: https://github.com/smart-on-fhir/hapi, and have used Synthea data with their [Bulk Data Server](https://github.com/smart-on-fhir/bulk-data-server).

* Algorex Health has used Synthea data to explore [Open Clinical Analysis](https://blog.algorexhealth.com/2017/04/open-clinical-analysis-with-mitre-part-2/).

* The MITRE Corporation has used Synthea data to create [SyntheticMass](https://syntheticmass.mitre.org), a 1/7th scale simulated model of the Commonwealth of Massachusetts, including a FHIR server (FHIR v1.8).