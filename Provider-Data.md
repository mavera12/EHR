By default, the project contains provider facilities for the entire United States in the `src/main/resources/providers/*.csv` files which have been post-processed from publicly available data from CMS.

**Currently the provider files in use are `hospitals`, `veterans`, `primarycare`, and `urgentcare`**

You can modify provider files or have Synthea use alternative provider files by altering the `src/main/resources/synthea.properties` file:

```properties
# Providers
generate.providers.hospitals.default_file = providers/hospitals.csv
generate.providers.longterm.default_file = providers/longterm.csv
generate.providers.nursing.default_file = providers/nursing.csv
generate.providers.rehab.default_file = providers/rehab.csv
generate.providers.hospice.default_file = providers/hospice.csv
generate.providers.dialysis.default_file = providers/dialysis.csv
generate.providers.homehealth.default_file = providers/home_health_agencies.csv
generate.providers.veterans.default_file = providers/va_facilities.csv
generate.providers.urgentcare.default_file = providers/urgent_care_facilities.csv
generate.providers.primarycare.default_file = providers/primary_care_facilities.csv
```

If you modify or replace the [Demographics](https://github.com/synthetichealth/synthea/wiki/Demographics-for-Other-Areas) file or [Zip or Postal Codes](https://github.com/synthetichealth/synthea/wiki/Zip-or-Postal-Codes) file then it is very likely you'll need to modify or replace the `hospitals.csv` provider file with hospitals relevant to the geographic locations (e.g. latitude and longitude).

## Hospitals File Format

The hospitals file is a CSV file, with a single row for each hospital.

Column Header | Description
--------------|------------
` ` | The first column has no header. It is the number of the hospital. The program ignores this column.
`id` | This the identifier assigned by CMS, but in other countries, this can just be a unique identifier.
`name` | The name of the hospital.
`address` | The street address.
`city` | The city.
`state` | The state abbreviation (or province or territory).
`zip` | The zip or postal code.
`county` | The county. The program ignores this column.
`phone` | The hospital or facility phone number.
`type` | The type of hospital. Free text string. Currently, only the string `"VA Facility"` has any meaning, which specifies a facility dedicated to military veterans.
`ownership` | Free-text. Something like `Government` or `Voluntary non-profit` or `Proprietary`. Currently unused.
`emergency` | `Yes` indicates the presence of an Emergency Room (ER) or Emergency Department (ED), and `No` does not.
`quality` | Quality score from `1 - 5`. Currently does nothing, but may eventually be used to effect that quality of care delivered.
`LAT` | Latitude
`LON` | Longitude

`LAT` and `LON` are used to select which facilities patients choose from when seeking care.

### Example for Hospital in the United Kingdom

```csv
,id,name,address,city,state,zip,county,phone,type,ownership,emergency,quality,LAT,LON
0,010001,Royal Shrewsbury Hospital,Mytton Oak Rd,Shrewsbury,WMS,SY3 8XQ,Shropshire,01743261000,NHS Teaching Hospital,Government - Hospital District or Authority,Yes,5,52.7091,-2.7931
```