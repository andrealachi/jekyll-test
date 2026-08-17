# GpkgToGpkg Tool Documentation

## General Tool Description

This tool compares each table in the **SOURCE GeoPackage** with the corresponding table in the **TARGET GeoPackage**.

For each table analysed, the model identifies records that are present in the source GeoPackage but missing from the corresponding table in the target GeoPackage. The missing records are then appended to the **TARGET GeoPackage**.

The comparison is mainly performed using the `guid` identifier field. If a record with the same `guid` already exists in the TARGET, it is not copied again. This prevents duplicate records from being created if the tool is run multiple times on the same datasets.

The model uses SQL queries of the following type to identify missing records:

```sql
SELECT ... WHERE NOT EXISTS (...)
```

The selected records are then copied to the TARGET using GDAL append operations.

The model is organised into functional blocks. Some tables are always checked and copied when needed, while others are processed only when the corresponding Boolean parameters are enabled. The main execution conditions are managed through **Conditional Branch** tools in the QGIS Model Designer.

---

## Input Parameters

### SOURCE GeoPackage

Select the GeoPackage containing the source data.

This file represents the database from which the model reads the records to be checked and, if necessary, copied to the TARGET.

---

### TARGET GeoPackage

Select the GeoPackage where the missing records will be appended.

The TARGET must have a structure consistent with the SOURCE, meaning that it must contain the same tables expected by the model.

---

### LOG Folder Optional

Select a folder where the log file will be saved.

If this parameter is provided, the model activates the branch dedicated to log writing and saves a text file with an automatically generated name, such as:

```text
copy_log_yyyyMMdd_HHmmss.txt
```

The log branch is executed only if `@log_folder` is not null and is not an empty string.

If this parameter is left empty, the model still performs the data copy process, but no log file is saved.

---

## Advanced Data Selection Parameters

The following parameters define which groups of data are copied from the SOURCE GeoPackage to the TARGET GeoPackage:

- **Copy Derived Soil Profile**
- **Copy Observed Soil Profile**
- **Copy Soil Body**
- **Copy Soil Derived Object**
- **Copy Soil Site**

These parameters are Boolean values. In the model, they control the execution of the different workflow branches through dedicated conditions.

---

### Copy Soil Site

This parameter controls the copy process for data related to **Soil Site**.

When **Copy Soil Site = TRUE**, the model processes the tables linked to soil sites. In particular, the following records are copied if they are missing in the TARGET:

- `soilsite`
- `datastream` records linked to `soilsite`
- `observation` records linked to the related `datastream` records

When **Copy Soil Site = FALSE**, the model does not copy Soil Site records and does not copy their related `datastream` and `observation` records.

---

### Copy Observed Soil Profile

This parameter controls the copy process for **observed Soil Profiles**, i.e. profiles where:

```text
isderived = 0
```

When **Copy Observed Soil Profile = TRUE**, the model enables the branches related to observed profiles.

If only this parameter is enabled, the model copies:

- `soilprofile` records with `isderived = 0`
- records linked to observed profiles
- `profileelement` records belonging to observed profiles
- descriptive tables linked to profiles and profile elements
- `datastream` records associated with the `profileelement` records of observed profiles
- `observation` records linked to those `datastream` records

Profile selection is performed through a subset expression on the `soilprofile` table, which distinguishes derived and observed profiles using the `isderived` field.

---

### Copy Derived Soil Profile

This parameter controls the copy process for **derived Soil Profiles**, i.e. profiles where:

```text
isderived = 1
```

When **Copy Derived Soil Profile = TRUE**, the model enables the branches related to derived profiles.

If only this parameter is enabled, the model copies:

- `soilprofile` records with `isderived = 1`
- `profileelement` records linked to derived profiles
- descriptive tables linked to profiles and profile elements
- `datastream` records associated with the `profileelement` records of derived profiles
- `observation` records linked to those `datastream` records

As for observed profiles, profile selection is performed through a subset expression on the `soilprofile` table, based on the `isderived` field.

---

### Special Case: `isderivedfrom` Table

The `isderivedfrom` table is copied only when both derived and observed profiles are enabled:

```text
Copy Derived Soil Profile = TRUE
AND
Copy Observed Soil Profile = TRUE
```

This is because `isderivedfrom` represents the relationship between derived profiles and observed profiles.

If only one of the two profile groups were copied, the relationship could point to profiles that are not present in the TARGET GeoPackage.

For this reason, the model uses the `Cond IsDerived` condition, which is active only when both parameters are set to `TRUE`.

---

### Copy Soil Body

This parameter controls the copy process for data related to **Soil Body**.

When **Copy Soil Body = TRUE**, the model copies the following records if they are missing:

- `soilbody`
- `soilbody_geom`
- `derivedprofilepresenceinsoilbody`, if derived profiles are also enabled
- `isbasedonsoilbody`, if Soil Derived Objects are also enabled

When **Copy Soil Body = FALSE**, the model does not copy Soil Body records, their geometries, or the relationships directly dependent on Soil Body.

---

### Copy Soil Derived Object

This parameter controls the copy process for data related to **Soil Derived Object**.

When **Copy Soil Derived Object = TRUE**, the model copies the following records if they are missing:

- `soilderivedobject`
- `isbasedonsoilderivedobject`
- `isbasedonobservedsoilprofile`, if observed profiles are also enabled
- `isbasedonsoilbody`, if **Copy Soil Body** is also enabled
- `datastream` records linked to `soilderivedobject`
- `observation` records linked to the related `datastream` records

When **Copy Soil Derived Object = FALSE**, the model does not copy Soil Derived Object records or the observational and relational data directly linked to them.

---

## Support Tables Always Checked or Copied

In addition to the data controlled by the main parameters, the model also includes support tables required to maintain database integrity, such as:

- `codelist`
- `observedproperty`
- `sensor`
- `thing`
- `unitofmeasure`
- `observingprocedure`
- `obsprocedure_sensor`
- `obsprocedure_obsdproperty`

These tables are used by `datastream` and `observation` records. They are compared between SOURCE and TARGET in order to copy missing records when required.

---

## Datastream and Observation Copy Logic

The model handles `datastream` records by separating them according to the linked Feature of Interest:

- `datastream` records linked to `soilsite`
- `datastream` records linked to `soilprofile`
- `datastream` records linked to `profileelement`
- `datastream` records linked to `soilderivedobject`

For each of these groups, the model can also copy the related `observation` records:

- `observation` records for `datastream` records linked to `soilsite`
- `observation` records for `datastream` records linked to `soilprofile`
- `observation` records for `datastream` records linked to `profileelement`
- `observation` records for `datastream` records linked to `soilderivedobject`

`datastream` and `observation` records are filtered through joins with the tables already selected by the model. This ensures that only measurements actually linked to the objects included in the current selection are copied.

For example:

- if only Soil Sites are copied, the model copies the `datastream` records linked to the selected `soilsite` records and their related `observation` records
- if only observed profiles are copied, the model copies the `datastream` records linked to the observed profiles and to their `profileelement` records
- if Soil Derived Objects are copied, the model copies the `datastream` records linked to `soilderivedobject` records and their related `observation` records

---

## Effects of the Main Parameter Combinations

### All Parameters Enabled

If all parameters are set to `TRUE`, the model performs the most complete synchronisation.

The following records are copied if they are missing:

- Soil Site
- Soil Plot records linked to observed profiles
- observed and derived Soil Profiles
- Profile Elements
- Soil Body records and related geometries
- Soil Derived Objects
- relationships between derived and observed profiles
- relationships between Soil Body, Soil Profile, and Soil Derived Object
- Datastream records
- Observation records
- SensorThings support tables and codelists

This is the recommended mode when the TARGET GeoPackage needs to be aligned with the SOURCE GeoPackage as completely as possible.

---

### Only Soil Site Enabled

If only the following parameter is enabled:

```text
Copy Soil Site = TRUE
```

the model copies missing Soil Site records and the observational data directly linked to Soil Sites.

Soil Profiles, Soil Body records, and Soil Derived Objects are not copied.

---

### Only Observed Profiles Enabled

If only the following parameter is enabled:

```text
Copy Observed Soil Profile = TRUE
```

the model copies observed profiles, their related profile elements, associated descriptive tables, and the observational data related to these objects.

Derived profiles are not copied, and the `isderivedfrom` table is not copied.

---

### Only Derived Profiles Enabled

If only the following parameter is enabled:

```text
Copy Derived Soil Profile = TRUE
```

the model copies derived profiles and their related data.

Observed profiles are not copied, and the `isderivedfrom` table is not copied because this relationship requires both sides of the relationship to be present.

---

### Observed and Derived Profiles Enabled

If both of the following parameters are enabled:

```text
Copy Derived Soil Profile = TRUE
Copy Observed Soil Profile = TRUE
```

the model copies all `soilprofile` records, both observed and derived.

In this case, the `isderivedfrom` table is also copied because it describes the relationships between derived profiles and observed profiles.

---

### Soil Body Enabled Without Derived Profiles

If the following parameter is enabled:

```text
Copy Soil Body = TRUE
```

but the following parameter is not enabled:

```text
Copy Derived Soil Profile = TRUE
```

the model copies Soil Body records and their related geometries, but it does not copy `derivedprofilepresenceinsoilbody` relationships because these depend on derived profiles.

---

### Soil Derived Object Enabled Without Soil Body

If the following parameter is enabled:

```text
Copy Soil Derived Object = TRUE
```

but the following parameter is not enabled:

```text
Copy Soil Body = TRUE
```

the model copies Soil Derived Object records and the related observational data, but it does not copy relationships with Soil Body records.

---

## Logical Execution Order

The model does not simply copy all tables independently. Instead, it follows a relational workflow.

First, it identifies missing records in the main tables. Then it uses joins and filters to restrict child tables and relationship tables to records that are actually linked to the selected objects.

Join operations use:

```text
DISCARD_NONMATCHING = true
```

This means that only records with a valid match against the previously selected data are retained.

This approach prevents orphan records or relationships pointing to objects that were not included in the transfer.

---

## Final Result

At the end of the execution, the TARGET GeoPackage contains the records that were missing compared with the SOURCE GeoPackage, limited to the data groups selected through the model parameters.

The tool can be run multiple times on the same TARGET GeoPackage. Since records are compared using the `guid` field, records that already exist are not appended again.
