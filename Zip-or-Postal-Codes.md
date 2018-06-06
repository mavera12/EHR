By default, the project contains zip codes for the entire United States in `src/main/resources/geography/zipcodes.csv` which has been post-processed from publicly available files.

You can modify the zip codes file or have Synthea use an alternative zip code file by altering the `src/main/resources/synthea.properties` file:

```properties
# default zip codes for every city in the US
generate.geography.zipcodes.default_file = geography/zipcodes.csv
```

If you modify or replace the [demographics](https://github.com/synthetichealth/synthea/wiki/Demographics-for-Other-Areas) file **and add new cities**, then you will also need to modify or replace the zip codes file **with those new cities**.

If you add new cities, you'll probably have to add the appropriate [Provider](https://github.com/synthetichealth/synthea/wiki/Provider-Data) locations near the appropriate latitudes and longitudes.

You may also want to change patient names to be more appropriate for another area (e.g. not so North American). If so, you'll also need to modify or replace the [Names](https://github.com/synthetichealth/synthea/wiki/Name-Data) file.

## Zip File Format

The zip code file is a CSV file, with **one row for each postal code** -- therefore, a city can potentially have multiple rows.

Column Header | Description
--------------|------------
` ` | The first column has no header. It is the number of the city. The program ignores this column.
`USPS` | State Name (or possibly a County or Province in a non-US area)
`ST` | State Abbreviation
`NAME` | City Name
`ZCTA5` | Zip Code or Postal Code
`LAT` | Latitude
`LON` | Longitude

Synthea uses `LAT` and `LON` to determine nearby [Provider](https://github.com/synthetichealth/synthea/wiki/Provider-Data) facilities where patients can receive care.

## Examples for Other Areas

These are incomplete partial examples for illustration purposes.

### Abridged Example for United Kingdom
Districting and addresses in the United Kingdom differ from the United States. In this example, we assume a region such as the West Midlands is equivalent to a State.

```csv
,USPS,ST,NAME,ZCTA5,LAT,LON
0,West Midlands,WMS,Shrewsbury,SY1,52.7081,-2.7549
1,West Midlands,WMS,Shrewsbury,SY2,52.7083,-2.7546
2,West Midlands,WMS,Shrewsbury,SY3,52.7089,-2.7543
```