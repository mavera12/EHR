Synthea allows for the creation of custom reports each time a population is generated, using the `CustomSqlReport` class. This class uses the internal H2 data store to run arbitrary SQL queries against. These queries are defined in a configurable SQL file, and the results are exported in CSV format to `./output/reports/`. Currently only SELECT statements are allowed in the SQL.

The Synthea DataStore uses an H2 database under the hood, so any queries must be valid H2-style syntax.
See [H2 Database Functions](http://www.h2database.com/html/functions.html) for more information on what functions are available.

## Configuration Settings
 - `generate.database_type` : REQUIRED: Set to `in-memory` or `file` to enable the database against which the queries will be run
 - `exporter.custom_report` : Set to `true` to enable the custom reports. Defaults to `false`.
 - `exporter.custom_report_queries_file` : Set this to the location of an SQL file (relative to `src/main/resources`) containing queries to run at each population generation.  Defaults to `custom_queries.sql`  (aka `src/main/resources/custom_queries.sql`)



## Available Database Tables
(see also `src/main/java/org/mitre/synthea/datastore/DataStore.java` for more information on how these tables are populated)



#### PERSON
```sql
CREATE TABLE IF NOT EXISTS PERSON 
 (id varchar, name varchar, date_of_birth bigint, date_of_death bigint, 
 race varchar, gender varchar, socioeconomic_status varchar)
```

#### ATTRIBUTE 
```sql
CREATE TABLE IF NOT EXISTS ATTRIBUTE 
 (person_id varchar, name varchar, value varchar)
```

#### PROVIDER
```sql
CREATE TABLE IF NOT EXISTS PROVIDER (id varchar, name varchar)
```

#### PROVIDER_ATTRIBUTE 
```sql
CREATE TABLE IF NOT EXISTS PROVIDER_ATTRIBUTE 
 (provider_id varchar, name varchar, value varchar)
```

#### ENCOUNTER 
```sql
CREATE TABLE IF NOT EXISTS ENCOUNTER 
 (id varchar, person_id varchar, provider_id varchar, name varchar, type varchar, 
 start bigint, stop bigint, code varchar, display varchar, system varchar)
```

#### CONDITION
```sql
CREATE TABLE IF NOT EXISTS CONDITION 
 (person_id varchar, name varchar, type varchar, start bigint, stop bigint, 
 code varchar, display varchar, system varchar)
```

#### MEDICATION 
```sql
CREATE TABLE IF NOT EXISTS MEDICATION 
 (id varchar, person_id varchar, provider_id varchar, name varchar, type varchar, 
 start bigint, stop bigint, code varchar, display varchar, system varchar)
```

#### PROCEDURE 
```sql
CREATE TABLE IF NOT EXISTS PROCEDURE 
 (person_id varchar, encounter_id varchar, name varchar, type varchar, 
 start bigint, stop bigint, code varchar, display varchar, system varchar)
```

#### REPORT 
```sql
CREATE TABLE IF NOT EXISTS REPORT 
 (id varchar, person_id varchar, encounter_id varchar, name varchar, type varchar, 
 start bigint, code varchar, display varchar, system varchar)
```

#### OBSERVATION 
```sql
CREATE TABLE IF NOT EXISTS OBSERVATION 
 (person_id varchar, encounter_id varchar, report_id varchar, name varchar, 
 type varchar, start bigint, value varchar, unit varchar, 
 code varchar, display varchar, system varchar)
```

#### IMMUNIZATION 
```sql
CREATE TABLE IF NOT EXISTS IMMUNIZATION 
 (person_id varchar, encounter_id varchar, name varchar, type varchar, 
 start bigint, code varchar, display varchar, system varchar)
```

#### CAREPLAN 
```sql
CREATE TABLE IF NOT EXISTS CAREPLAN 
 (id varchar, person_id varchar, provider_id varchar, name varchar, type varchar, 
 start bigint, stop bigint, code varchar, display varchar, system varchar)
```

#### IMAGING_STUDY 
```sql
CREATE TABLE IF NOT EXISTS IMAGING_STUDY 
 (id varchar, uid varchar, person_id varchar, encounter_id varchar, start bigint, 
 modality_code varchar, modality_display varchar, modality_system varchar, 
 bodysite_code varchar, bodysite_display varchar, bodysite_system varchar, 
 sop_class varchar)
```

#### CLAIM 
```sql
CREATE TABLE IF NOT EXISTS CLAIM 
 (id varchar, person_id varchar, encounter_id varchar, medication_id varchar, 
 time bigint, cost decimal)
```

#### COVERAGE
```sql
CREATE TABLE IF NOT EXISTS COVERAGE (person_id varchar, year int, category varchar)
```

#### QUALITY_OF_LIFE 
```sql
CREATE TABLE IF NOT EXISTS QUALITY_OF_LIFE 
 (person_id varchar, year int, qol double, qaly double, daly double)
```
#### UTILIZATION 
```sql
CREATE TABLE IF NOT EXISTS UTILIZATION 
 (provider_id varchar, year int, encounters int, procedures int, 
 labs int, prescriptions int)
```

#### UTILIZATION_DETAIL 
```sql
CREATE TABLE IF NOT EXISTS UTILIZATION_DETAIL 
 (provider_id varchar, year int, category varchar, value int)
```
## Sample Queries
(Note: these queries have been formatted for readability. Currently the report generator requires that all queries be contained on a single line.)

#### Sample Query 1: select everything from the "person" table.
```sql
select * from person;
```

#### Sample Query 2: select the number of living people.
```sql
select count(*) from person where person.DATE_OF_DEATH is null;
```

#### Sample Query 3: select the people that have an active diagnosis of diabetes, along with the age of diagnosis

```sql
SELECT p.name, p.gender, 
  DATEADD('MILLISECOND', p.DATE_OF_BIRTH, DATE '1970-01-01') DOB, 
  DATEADD('MILLISECOND', c.start , DATE '1970-01-01') onset_date, 
  DATEDIFF('YEAR', 
      DATEADD('MILLISECOND', p.DATE_OF_BIRTH, DATE '1970-01-01'), 
      DATEADD('MILLISECOND', c.start, DATE '1970-01-01')) age_at_diagnosis
FROM PERSON p, CONDITION c 
WHERE p.ID = c.PERSON_ID 
AND c.code = '44054006';
```

