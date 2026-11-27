# Exporting SoilBodies by Geometry Layer

## Purpose of the Tool

The **Export SoilBody by geometry layer** tool copies a selected spatial subset of the SoilWise data model from one GeoPackage to another.

The geometry layer is used as the only initial selection filter. For each non-empty geometry linked through the `guid_soilbody` field, the tool identifies the corresponding Soil Body and copies it to an existing target GeoPackage.

The tool automatically performs the main required operations:

- reads the selected SoilBody geometry layer from the source GeoPackage;
- identifies the Soil Bodies linked to valid geometries;
- copies the selected `soilbody` records;
- copies valid `derivedprofilepresenceinsoilbody` rows when the referenced Soil Profiles already exist in the target;
- copies valid `isbasedonsoilbody` rows when the referenced Soil Derived Objects already exist in the target;
- recreates the selected geometry layer in the target GeoPackage;
- transforms geometries to EPSG:3035 when necessary;
- reports the number of records copied.

The target GeoPackage must already contain the non-spatial SoilWise database structure. The tool does not create a complete SoilWise GeoPackage from an empty file.

## Adding the Script to the QGIS Toolbox

### Save the Script on Your Computer

Copy the following file:

`copy_soilbody_by_geometry.py`

into a permanent folder on your computer.

It is recommended to create a dedicated folder for QGIS scripts, for example:

`C:\QGIS_Scripts`

or:

`Documents\QGIS\Scripts`

Do not leave the script in the Downloads folder, a temporary folder, or as an email attachment.

The script must retain the `.py` file extension.

### Open the Processing Toolbox

Start QGIS and open the SoilWise project.

From the main menu, select:

**Processing → Toolbox**

In some QGIS configurations, the Toolbox may already be visible in the right-hand panel.

If the **Processing** menu is not available, make sure that the main Processing plugin is enabled:

1. Open **Plugins → Manage and Install Plugins**.
2. Select the **Installed** tab.
3. Search for **Processing**.
4. Make sure that it is enabled.

### Add the Script

A button for managing Python scripts is available at the top of the Processing Toolbox.

To add the script:

1. Open the **Processing Toolbox**.
2. Click the **Scripts** button in the Toolbox toolbar.
3. Select **Add Script to Toolbox**.
4. Locate the `copy_soilbody_by_geometry.py` file.
5. Select the file and confirm.

Once added, the tool should appear in the following Toolbox group:

**SoilWise Geopackage Tools**

with the name:

**Export SoilBody by geometry layer**

To find it quickly, enter one of the following terms in the Toolbox search bar:

- `SoilBody`
- `geometry layer`

## Requirements Before Running the Tool

### Source GeoPackage

The source GeoPackage must contain:

- the SoilBody geometry layer used to select the records;
- the `soilbody` table;
- any optional relationship tables that need to be copied;
- a valid `guid_soilbody` field in the selected geometry layer.

Only features with a non-empty geometry and a populated `guid_soilbody` value are considered.

### Target GeoPackage

The target GeoPackage must:

- already exist;
- be different from the source GeoPackage;
- already contain the non-spatial SoilWise schema;
- contain the destination tables required by the records being copied.

The tool copies only columns that exist in both the source and target tables. Existing records are not duplicated.

### Geometry Type

The source geometry layer must be of type:

**MultiPolygon**

The geometry type is fixed by the SoilBody data model and cannot be selected in the tool dialog.

If the source layer is not MultiPolygon, the tool stops before modifying the target GeoPackage.

### Backup Recommendation

Before running the tool, create a backup copy of the target GeoPackage.

The geometry layer with the selected name is created or overwritten in the target. A backup allows the previous version to be restored if the wrong layer name or input file is selected.

## Running the Tool

### Start the Tool

In the Processing Toolbox:

1. Expand the **SoilWise Geopackage Tools** group.
2. Double-click **Export SoilBody by geometry layer**.

The tool dialog will open.

### Select the Source GeoPackage

In **Source GeoPackage**, select the `.gpkg` file containing:

- the SoilBody geometry layer;
- the Soil Bodies to export;
- any related SoilWise records.

### Select the Target GeoPackage

In **Target GeoPackage (existing SoilWise schema)**, select the existing GeoPackage that will receive the copied records.

The source and target files must be different.

### Enter the Geometry Table Name

In **Geometry table name**, enter the exact name of the geometry layer stored in the source GeoPackage.

For example:

`soilbody_geometry`

The name must match the physical table name in the source GeoPackage, not only a different display name assigned in the QGIS Layers panel.

The geometry layer must contain the field:

`guid_soilbody`

### Run the Process

After completing all parameters:

1. Click **Run**.
2. Wait for the process to finish.
3. Read the messages displayed in the Processing dialog.
4. Check the final record counts.
5. Close the tool dialog when processing is complete.

## How Records Are Selected

The geometry layer is the starting point for the entire export.

The tool examines all its features and selects only those that:

- have a valid, non-empty geometry;
- contain a value in `guid_soilbody`.

The values contained in `guid_soilbody` identify the Soil Bodies to transfer.

Features with an empty geometry or without a linked Soil Body are ignored.

If no valid geometry is found, the tool stops without performing the export.

## Content Copied to the Target GeoPackage

### SoilBodies

The selected records are copied from the `soilbody` table first.

This first step is necessary because the geometry layer and the optional relationship tables may refer to these parent records.

After copying, the tool verifies that every Soil Body referenced by the selected geometries is available in the target GeoPackage.

If one or more required Soil Bodies cannot be copied, the tool stops and reports the missing identifiers.

### Derived Profile Presence in SoilBody

The tool can copy rows from:

`derivedprofilepresenceinsoilbody`

A row is copied only when:

- it refers to one of the selected Soil Bodies;
- its related Soil Profile already exists in the target GeoPackage.

If the related Soil Profile is missing from the target, the row is skipped without interrupting the export. The script does not copy the missing Soil Profile automatically.

### SoilDerivedObject-to-SoilBody Relationships

The tool can also copy rows from:

`isbasedonsoilbody`

A row is copied only when:

- it refers to one of the selected Soil Bodies;
- its related Soil Derived Object already exists in the target GeoPackage.

If the related Soil Derived Object is missing, the relationship is skipped without interrupting the export. The script does not copy that Soil Derived Object automatically.

### Geometry Layer

After copying the SoilBody and optional relationship records, the tool creates the selected geometry layer in the target GeoPackage.

The output layer:

- keeps the same name as the source geometry table;
- is created as MultiPolygon;
- preserves the source attribute fields and their order;
- excludes a field named `fid`;
- contains only the valid geometries used for the selection;
- is written using EPSG:3035;
- is transformed to EPSG:3035 if the source layer uses a different coordinate reference system.

If a layer with the same name already exists in the target GeoPackage, that layer is overwritten. The rest of the GeoPackage is not replaced.

## Content Not Copied Automatically

The geometry layer is used to select Soil Bodies only. The tool does not automatically copy every record related to those Soil Bodies.

In particular, it does not automatically copy:

- Soil Profiles referenced by `derivedprofilepresenceinsoilbody`;
- Soil Derived Objects referenced by `isbasedonsoilbody`;
- Datastreams;
- Observations;
- Profile Elements;
- Samples;
- Sampling records;
- other SoilWise tables not explicitly handled by the script.

Optional relationships are copied only when their required parent records already exist in the target GeoPackage.

## Processing Results

At the end of the process, QGIS reports the number of copied records for:

- **Copied SoilBodies**;
- **Copied geometries**;
- **Copied Derived Profile Presence rows**;
- **Copied isbasedonsoilbody rows**.

A value of zero is not necessarily an error. For example:

- the selected Soil Bodies may have no optional relationships;
- their related Soil Profiles may not yet exist in the target;
- their related Soil Derived Objects may not yet exist in the target;
- some SoilBody records may already be present in the target.

Records that already exist in the target are ignored rather than duplicated. Therefore, the reported number represents newly inserted records, not the total number available after the export.

## Checks Recommended After the Export

After the tool finishes:

1. Open the target GeoPackage in QGIS.
2. Check that the geometry layer is present.
3. Verify that the expected number of geometries has been copied.
4. Open the `soilbody` table and check the transferred records.
5. Check any expected `derivedprofilepresenceinsoilbody` rows.
6. Check any expected `isbasedonsoilbody` rows.
7. Save the QGIS project if the target layers have been added to it.

It is also advisable to open a few copied geometries and confirm that their `guid_soilbody` values refer to valid Soil Bodies in the target.

If an expected optional relationship is missing, check whether the referenced Soil Profile or Soil Derived Object is already present in the target GeoPackage.

## Warnings

### Do Not Use the Same File as Source and Target

The source and target GeoPackages must be different files.

The tool rejects an operation in which the same GeoPackage is selected for both parameters.

### Use the Exact Geometry Table Name

The value entered in **Geometry table name** must match the table stored in the source GeoPackage.

If the table does not exist, the tool stops without copying records.

### The Geometry Layer Must Be MultiPolygon

The SoilBody geometry layer must be MultiPolygon.

A Point, LineString, Polygon or other geometry type is rejected before the target GeoPackage is modified.

### Existing Geometry Layers Are Overwritten

If the target GeoPackage already contains a geometry layer with the selected name, that layer is replaced.

Create a backup if the existing layer contains information that must be retained.

### The Target Must Already Contain the SoilWise Schema

The tool does not create missing non-spatial SoilWise tables.

If a destination table is absent, its records cannot be copied. Use a target GeoPackage created from the correct SoilWise database structure.

### Existing Records Are Not Updated

The tool uses the existing identifiers to prevent duplicate insertion.

If a record is already present in the target, it is ignored rather than updated with values from the source. The tool is therefore intended for selective transfer, not for synchronising modifications between two existing copies.

### Optional Relationships Depend on Existing Target Records

Rows from `derivedprofilepresenceinsoilbody` and `isbasedonsoilbody` are copied only when their required related records already exist in the target.

A relationship may therefore be omitted even though it exists in the source.

### Do Not Interrupt the Process

Avoid closing QGIS, disconnecting the storage device or modifying either GeoPackage while the tool is running.

Wait until Processing reports that the operation has completed.

## Summary

The **Export SoilBody by geometry layer** tool performs a selective export from one SoilWise GeoPackage to another using a MultiPolygon geometry layer as the selection filter.

In a single operation, it:

- identifies Soil Bodies from valid geometries;
- copies the related `soilbody` records;
- verifies that all required Soil Bodies exist in the target;
- copies valid `derivedprofilepresenceinsoilbody` rows;
- copies valid `isbasedonsoilbody` rows;
- recreates or overwrites the selected geometry layer in the target;
- excludes the `fid` attribute;
- writes geometries in EPSG:3035;
- reports the number of newly copied records.

The tool is designed for controlled transfer between two different GeoPackages. The target must already contain the SoilWise schema, and a backup is recommended before overwriting an existing geometry layer.
