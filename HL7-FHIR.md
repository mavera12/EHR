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