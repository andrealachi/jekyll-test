# GpkgToGpkg QGIS Model Documentation

> Selective synchronization of data between GeoPackages

## 1. Introduction

The **GpkgToGpkg** model is a QGIS Model Designer workflow developed to compare a source GeoPackage with a target GeoPackage, identify records that are missing from the target, and append them to the target database. The model does not perform a full overwrite. Instead, it processes each table individually and uses existence checks to prevent records that are already present from being copied again.

The model has a modular structure. Some core blocks are used regardless of the selected thematic options, whereas the blocks associated with Soil Sites, observed or derived Soil Profiles, Soil Bodies, and Soil Derived Objects are enabled through Boolean parameters and Conditional Branch algorithms.

> **Key principle**  
> Synchronization is incremental: the SOURCE is read, the TARGET is queried to determine whether the relevant keys already exist, and only missing records are passed to the GDAL append operations.

## 2. General workflow

1. **Input selection.** The user specifies the SOURCE, the TARGET, an optional log folder, and the functional data groups to be copied.
2. **Condition evaluation.** Eight Conditional Branch algorithms evaluate named branches. A component that explicitly depends on a branch is executed only when the corresponding expression evaluates to `true`.
3. **SOURCE/TARGET comparison.** The Execute SQL algorithms load the same table from both GeoPackages and apply a query based on `NOT EXISTS`. For most tables, `guid` is used as the unique identifier; junction tables use composite keys.
4. **Relational selection.** Join Attributes by Field Value links the main entities to dependent tables. Retain Fields removes fields added by the join and restores the schema required by the destination table.
5. **Append to the TARGET.** GDAL Convert format uses `-update` and `-append`, assigns the destination layer name with `-nln`, and uses `-nlt NONE` for non-spatial tables. The `-unsetFid` option prevents the internal feature ID from being transferred.
6. **Optional logging.** When `log_folder` is provided, the QGIS Processing log is written to a dated file in the selected folder.

## 3. Input parameters

| Parameter | Type / default | Description |
|---|---|---|
| **Copy Derived Soil Profile** | Boolean / `True` | Copies derived Soil Profiles, where `isderived = 1`, together with related `profileelement` records, descriptive tables, datastreams, and observations. The `isderivedfrom` table is copied only when both derived and observed Soil Profiles are enabled. |
| **Copy Observed Soil Profile** | Boolean / `True` | Copies observed Soil Profiles, where `isderived = 0`, together with related `profileelement` records, descriptive tables, datastreams, and observations. |
| **Copy Soil Body** | Boolean / `True` | Copies `soilbody` and `soilbody_geom` records. It also copies `derivedprofilepresenceinsoilbody` when derived Soil Profiles are enabled, and `isbasedonsoilbody` when Soil Derived Objects are enabled. |
| **Copy Soil Derived Object** | Boolean / `True` | Copies `soilderivedobject`, `isbasedonsoilderivedobject`, related datastreams and observations, and conditionally copies `isbasedonobservedsoilprofile` and `isbasedonsoilbody`. |
| **Copy Soil Site** | Boolean / `True` | Copies `soilsite` records and the related datastream and observation records. |
| **LOG FOLDER** | Folder | Folder in which the QGIS Processing log file will be saved. |
| **SOURCE_GPKG** | File | Source GeoPackage from which records are read, checked, and copied when required. |
| **TARGET_GPKG** | File | Target GeoPackage to which missing records are appended. It must contain the tables expected by the model and use a schema consistent with the SOURCE. |

## 4. Conditional branches

Conditional Branch algorithms produce named branches that can be referenced as dependencies by other algorithms.

| Branch | Expression | Purpose |
|---|---|---|
| **Cond Soil Profile enabled** | `@copy_derived_soil_profile = true OR @copy_observed_soil_profile = true` | Enables the common Soil Profile workflow when at least one profile category is requested. |
| **Cond Soil Site enabled** | `@copy_soil_site = true` | Enables Soil Sites and their related datastreams and observations. |
| **Cond Soil Body enabled** | `@copy_soil_body = true` | Enables Soil Bodies, geometries, and directly dependent relationships. |
| **Cond Soil Der Obj enabled** | `@copy_soil_derived_object = true` | Enables Soil Derived Objects and their related data. |
| **Cond IsDerived** | `@copy_derived_soil_profile = true AND @copy_observed_soil_profile = true` | Enables `isderivedfrom` only when both sides of the relationship are included. |
| **Cond Observed** | `@copy_observed_soil_profile = true` | Enables workflow sections specific to observed Soil Profiles. |
| **Cond Derived** | `@copy_derived_soil_profile = true` | Enables workflow sections specific to derived Soil Profiles. |
| **LOG** | `@log_folder IS NOT NULL AND trim(@log_folder) <> ''` | Enables log export only when a non-null, non-empty folder has been provided. |

> **Important semantic detail**  
> A condition controls only the components that explicitly declare the relevant branch as a dependency. It does not automatically behave like an `if` block surrounding an entire area of the graphical model.

## 5. Execution scenarios

| User selection | True branches | Main result |
|---|---|---|
| Observed profiles only | Cond Soil Profile enabled; Cond Observed | Copies profiles with `isderived = 0`, Soil Plots, and the relevant Profile Element, Datastream, and Observation chains. `isderivedfrom` is not copied. |
| Derived profiles only | Cond Soil Profile enabled; Cond Derived | Copies profiles with `isderived = 1` and their related chains. `isderivedfrom` is not copied. |
| Observed and derived profiles | Cond Soil Profile enabled; Cond Observed; Cond Derived; Cond IsDerived | Copies both profile groups and the `isderivedfrom` relationship. |
| Soil Sites only | Cond Soil Site enabled | Copies `soilsite` records and their related datastream and observation chains. |
| Soil Bodies and derived profiles | Cond Soil Body enabled; Cond Derived | Copies `soilbody` records, geometries, and `derivedprofilepresenceinsoilbody`. |
| Soil Derived Objects and observed profiles | Cond Soil Der Obj enabled; Cond Observed | Copies derived objects, `isbasedonsoilderivedobject`, `isbasedonobservedsoilprofile`, datastreams, and observations. |
| Log folder provided | LOG | Saves the log. If the field is null or empty, Save log to file is skipped. |

## 6. Functional execution flows

Internal model identifiers are intentionally omitted. The display names used in QGIS Model Designer provide a clearer operational description.

### 6.1 Shared reference data

This flow completes the shared tables used by datastreams and observations.

- Check for missing codelists → Copy Codelist.
- Check for missing Observing Procedure → Copy Observing Procedure.
- Check for missing sensor → Copy Sensor.
- Check for missing observedproperty → Copy Observedproperty.
- Check for missing Obsprocedure Sensor → Copy Obsprocedure Sensor.
- Check for missing Unit Of Measure → Copy Unit Of Measure.
- Check for missing Obsprocedure Obsdproperty → Copy Obsprocedure Obsdproperty.
- Check for missing Thing → Copy Thing.

Each check returns only records whose key does not already exist in the TARGET.

### 6.2 Soil Profile: common workflow

This flow is enabled when at least one group, observed or derived Soil Profiles, is requested.

- Check for missing soilprofile identifies profiles that are absent from the TARGET.
- Subset Soil Profile applies the user selections using the `isderived` field.
- Copy Soil Profile appends the selected profiles.
- Join SP PE → Table PE → Copy Profile Element.
- Join PE OH_PE → Table OH_PE → Copy Other Horizon Profile Element.
- Join OH_PE OHNT → Table OHNT → Copy Other Horizon Notation Type.
- JOIN PE FHNT → FHNT → Copy FAO Horizon Notation Type.
- JOIN SP WRBQG_P → Table WRBQG_P → Copy WRB Qualifier Group Profile.
- JOIN WRBQG_P WRBQGT → Table WRBQGT → Copy WRB Qualifier Group Type.
- JOIN SP OSNT → Table OSNT → Copy Other Soil Name Type.

The Join SP DS / Join SP DS OBS and Join PE DS / Join PE DS OBS chains also select the datastreams and observations associated with profiles and profile elements.

### 6.3 Observed Soil Profiles

This flow is enabled by **Copy Observed Soil Profile** and includes records where `isderived = 0`.

- Check for missing soilplot identifies missing Soil Plots.
- Copy Soil Plot appends their geometries to the TARGET.
- If Soil Derived Objects are also enabled, the Is Based On Observed Soil Profile relationship is copied.

### 6.4 Derived Soil Profiles

This flow is enabled by **Copy Derived Soil Profile** and includes records where `isderived = 1`.

- It uses the common Profile Element, classification, datastream, and observation workflow.
- If Soil Bodies are also enabled, Derived Profile Presence in Soil Body is copied.

### 6.5 Relationship between observed and derived profiles

The `isderivedfrom` relationship requires both profile categories.

- Check for missing isderivedfrom compares `guid_base` and `guid_related`.
- Copy Is Derived From appends only missing relationships.

> **Why an AND condition is required**  
> Copying `isderivedfrom` when only one profile group is included could create a relationship that references a profile not present in the TARGET.

### 6.6 Soil Sites

This flow is enabled by **Copy Soil Site**.

- Check for missing soilsite → Copy Soil Site.
- Join SS DS → Table DS_SS → Copy DS_SS.
- Join SS DS OBS → Table DS_SS_OBS → Copy DS_SS_OBS.

The joins restrict datastreams and observations to the Soil Sites included in the current synchronization.

### 6.7 Soil Bodies

This flow is enabled by **Copy Soil Body**.

- Check for missing soilbody → Copy Soil Body.
- Check for missing soilbody geom → Copy Soil Body Geom.
- Derived Profile Presence in Soil Body is appended when derived Soil Profiles are also enabled.
- Is Based On Soil Body is appended when Soil Derived Objects are also enabled.

### 6.8 Soil Derived Objects

This flow is enabled by **Copy Soil Derived Object**.

- Check for missing soil derived object → Copy Soil Derived Object.
- Check for missing is based on soil derived object → Copy Is Based On Soil Derived Object.
- Join SDO DS → Table DS_SDO → Copy DS_SDO.
- Join SDO DS OBS → Table DS_SDO_OBS → Copy DS_SDO_OBS.
- Is Based On Observed Soil Profile also requires observed Soil Profiles.
- Is Based On Soil Body also requires Soil Bodies.

### 6.9 Log export

This flow is independent of the thematic data domains.

- Log path checks whether `log_folder` has been provided.
- Save log to file writes the QGIS Processing log to the selected folder.
- If `log_folder` is null or empty, log export is skipped.

## 7. Types of components used

### 7.1 Check for missing…

These components perform the SOURCE/TARGET comparison:

- load the same table from the SOURCE and the TARGET;
- apply a `NOT EXISTS` query;
- return a temporary layer containing only missing records.

### 7.2 Subset and Join…

These components select relevant records:

- Subset Soil Profile distinguishes observed from derived profiles;
- Join components follow table relationships and retain only data linked to the selected entities;
- unmatched records are discarded.

### 7.3 Table…

These components clean and restore the output schema:

- retain fields required by the destination table;
- remove auxiliary attributes introduced by joins.

### 7.4 Copy…

These components modify the TARGET:

- open the existing GeoPackage in update mode;
- append records to the specified layer;
- do not transfer the temporary FID;
- use a non-spatial output layer for non-spatial tables.

### 7.5 Conditional… and Log path

These components control execution:

- evaluate Boolean parameters and the availability of the log folder;
- produce named branches used as dependencies by other components.

## 8. Consistency, assumptions, and points to consider

- The TARGET must already exist and contain the expected tables with a compatible schema. The model appends records; it does not create the complete schema.
- Duplicate prevention depends on the key defined in each query. Most tables use `guid`, `codelist` uses `id + collection`, and several junction tables use pairs of foreign keys.
- Geometries are handled only by components that declare a geometry field. Soil Plots, Soil Derived Objects, and Soil Body geometries have specific spatial configurations.
- Conditional branches protect only the components that explicitly declare them. Some supporting Execute SQL components do not have an explicit conditional dependency in the `.model3` definition. This does not modify the TARGET when the corresponding GDAL component is skipped, but it may result in unnecessary preliminary processing.
- Referential integrity depends on the selected combination. The model explicitly handles sensitive cases, such as copying `isderivedfrom` only when both observed and derived Soil Profiles are enabled.
- The model does not update existing records with different values and does not delete records from the TARGET. It is an incremental synchronization of missing records, not bidirectional replication or full data alignment.

## 9. Recommended operating procedure

1. Confirm that the SOURCE and TARGET use the same SoilWise schema version and that the TARGET contains all required tables.
2. Create a backup of the TARGET before the first run and after any modification to the model.
3. Enable only the data groups that are required. To preserve complete relationships, enable related domains together.
4. Set `log_folder` during testing and large-scale operations to retain an execution record.
5. After execution, review the number of appended records and check for orphaned references in relationship tables.

## 10. Final summary

GpkgToGpkg implements a three-stage pipeline: identification of missing records through SQL, selection of dependent records through joins and field retention, and writing to the TARGET through GDAL append operations. Conditional branches make the model configurable without duplicating complete workflows, but their effect must be interpreted through the dependencies declared by individual components. Parameter selection therefore determines not only which main entities are copied, but also which relationships, datastreams, and observations are retained in the target GeoPackage.
