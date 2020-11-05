Synthea generates HL7 FHIR records using the [HAPI FHIR](http://hapifhir.io/) library to generate a FHIR `Bundle` for each `Patient`. Currently, only JSON is supported.

Currently supported FHIR Resources:
- `AllergyIntolerance`
- `Bundle`
- `CarePlan`
- `Claim`
- `Condition`
- `Coverage` (R4 and STU3 only)
- `DiagnosticReport` (for labs)
- `Encounter`
- `ExplanationOfBenefit` (R4 and STU3 only, Blue Button 2.0 Implementation Guide in STU3)
- `Goal`
- `ImagingStudy`
- `Immunization`
- `Location` 
- `MedicationRequest`
- `Observation`
- `Organization`
- `Patient`
- `Practitioner`
- `Procedure`

Resources only supported with R4 and US Core:
- `CareTeam`
- `Device`
- `DiagnosticReport` (for clinical notes)
- `DocumentReference` (for clinical notes)
- `Location`
- `Medication`
- `MedicationStatement`
- `PractitionerRole`
- `Provenance`

### Configuring FHIR

The exporting of FHIR can be configured using the `src/main/resources/synthea.properties`:

```properties
# default FHIR R4 configuration.
exporter.fhir.export = true
# transaction bundle 'true' produces transaction Bundles
# else if 'false' produces collection Bundles.
exporter.fhir.transaction_bundle = true
# if bulk_data 'true' ndjson bulk format is exported
# else if 'false' (default) normal FHIR bundles are exported
exporter.fhir.bulk_data = false
# Use the US Core R4 Implementation Guide
exporter.fhir.use_us_core_ig = false
# Use Standard Health Record (SHR) extensions for STU3?
exporter.fhir.use_shr_extensions = false
# Exporting FHIR DSTU2
exporter.fhir_dstu2.export = false
# Exporting FHIR STU3
exporter.fhir_stu3.export = false
# Exporting Hospital Provider data in separate file.
exporter.hospital.fhir.export = true
exporter.hospital.fhir_stu3.export = false
exporter.hospital.fhir_dstu2.export = false
# Exporting Practitioner data in separate file.
exporter.practitioner.fhir.export = true
exporter.practitioner.fhir_stu3.export = false
exporter.practitioner.fhir_dstu2.export = false
```

### Producing and Using FHIR

In the `synthea.properties` file, make sure that `exporter.fhir.export`, `exporter.fhir.transaction_bundle`, `exporter.practitioner.fhir.export` and `exporter.hospital.fhir.export` are all set to `true`.

Next, produce some synthetic data:
```
./run_synthea -p 100
```

By default, FHIR JSON data should now exist in the `./output/fhir` sub-directory. If you have a FHIR server and client, you can `POST` those bundles to the FHIR server. See [[FHIR Transaction Bundles]] for additional details and the recommended order that bundles are POSTed.

For example:

```
curl http://hapi.fhir.org/baseR4 
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
      "url": "http://hapi.fhir.org/baseR4"
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

* The MITRE Corporation has used Synthea data to create [SyntheticMass](https://syntheticmass.mitre.org), a 1/7th scale simulated model of the Commonwealth of Massachusetts, including a FHIR server (FHIR STU3).

* An MSDN blog post illustrates [Loading Synthea FHIR Data with Logic Apps and Functions in Azure Government](https://blogs.msdn.microsoft.com/mihansen/2018/05/10/loading-synthea-fhir-data-with-logic-apps-and-functions-in-azure-government/).