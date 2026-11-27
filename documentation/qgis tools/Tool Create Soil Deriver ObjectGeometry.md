# Creating a Geometry Layer Linked to SoilDerivedObject

## Purpose of the Tool

The **Create SDO Geometry Layer** tool creates a new geometry layer directly in the GeoPackage used by the project and links it to the existing `soilderivedobject` layer.

The new layer can be used to associate one or more geometries with each Soil Derived Object stored in the database. The tool automatically performs the main required operations:

- creates the new geometry layer in the GeoPackage;
- allows the user to select the geometry type;
- prepares the identifier fields;
- links the new layer to the `soilderivedobject` layer;
- configures the data entry forms;
- adds the new layer to the QGIS project;
- adds the list of linked geometries to the SoilDerivedObject form;
- saves the project changes whenever possible.

The tool is designed specifically for the QGIS project associated with the SoilWise GeoPackage. It must not be used with projects or GeoPackages that have a different structure.

## Adding the Script to the QGIS Toolbox

### Save the Script on Your Computer

Copy the following file:

`create_sdo_geometry_layer.py`

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
4. Locate the `create_sdo_geometry_layer.py` file.
5. Select the file and confirm.

Once added, the tool should appear in the following Toolbox group:

**SoilWise Geopackage Tools**

with the name:

**Create SDO Geometry Layer**

To find it quickly, enter one of the following terms in the Toolbox search bar:

- `SDO`
- `geometry layer`

## Running the Tool

### Start the Tool

In the Processing Toolbox:

1. Expand the **SoilWise Geopackage Tools** group.
2. Double-click **Create SDO Geometry Layer**.

The tool dialog will open.

### Enter the Name of the New Layer

In the **New SDO geometry table name** field, enter the name of the new geometry layer.

The default value is:

`soilderivedobject_geometry`

Replace it with a meaningful name for the type of geometry you want to create.

The name entered becomes the actual name of the new table in the GeoPackage. Choose a clear and final name, preferably using lowercase letters and no spaces.

The table name may contain:

- letters;
- numbers;
- underscores (`_`).

The first character must be a letter or an underscore. Spaces, hyphens and other special characters are not allowed.

### Select the Geometry Type

Use the **Geometry type** field to select the type of features that the new layer will contain.

The available options are:

- **Point** for individual point features;
- **MultiPoint** for features made up of multiple points;
- **LineString** for individual lines;
- **MultiLineString** for features made up of multiple lines;
- **Polygon** for individual polygons;
- **MultiPolygon** for features made up of one or more polygons.

Choose the geometry type carefully because it defines the structure of the layer and cannot be changed by this tool after the table has been created.

### Run the Process

After entering the table name and selecting the geometry type:

1. Click **Run**.
2. Wait for the process to finish.
3. Check that no error messages are displayed.
4. Close the tool dialog when processing is complete.

## Processing Result

### Creation of the New Layer

The tool creates a new geometry layer in the same GeoPackage used by the `soilderivedobject` layer.

The geometry column is named:

`geometry`

The geometry type corresponds to the option selected when the tool was run.

The coordinate reference system assigned to the layer is:

**ETRS89-extended / LAEA Europe – EPSG:3035**

The geometries must therefore be consistent with the coordinate reference system used by the SoilWise project.

### Link to SoilDerivedObject

The tool automatically creates a relation between:

- the `guid_soilderivedobject` field in the new layer;
- the `guid` field in the `soilderivedobject` layer.

Each geometry must be linked to an existing Soil Derived Object.

This link is mandatory. A geometry cannot be added without an associated Soil Derived Object.

If the linked Soil Derived Object is deleted, the associated geometry records are also deleted. This ensures that child geometries are not left without a valid parent record.

### Automatic Identifiers

The new layer contains the following identifying fields:

- `id`, the internal numeric identifier;
- `guid`, the unique identifier generated automatically for each record;
- `guid_soilderivedobject`, the field that links the geometry to its Soil Derived Object.

The `guid` value is generated automatically when a new feature is created and should not be changed manually afterwards.

### Loading the Layer in the Layers Panel

When processing is complete, the new layer is automatically loaded into the project.

If the Layers panel contains a group named:

**Soil Derived Object**

the tool attempts to place the new layer inside that group.

The displayed layer name corresponds to the value entered in the **New SDO geometry table name** field.

## Automatic Form Configuration

### Form of the New Layer

The tool creates a custom form for the new layer.

The form contains:

- an **ID** group containing the technical `id` and `guid` fields;
- a **Soil Derived Object** section containing the field used to select the linked `soilderivedobject` record.

The **ID** group is visible in QGIS Desktop and is collapsed by default because users do not normally need to edit its technical values.

The group is configured to remain hidden in QField, making the fieldwork form simpler.

The form gives more space to the **Soil Derived Object** section so that the relation field is easier to view and use.

### Selecting the Soil Derived Object

The `guid_soilderivedobject` field is configured as a relation reference field.

Instead of entering a GUID manually, the user can select the related Soil Derived Object from the available records.

The form also enables the **Open Form** button, allowing the user to open the form of the selected Soil Derived Object directly from the child geometry form.

To provide a readable description in feature lists and forms, the layer displays the `inspireid_localid` value of the related Soil Derived Object whenever it is available.

### SoilDerivedObject Layer Form

The tool also modifies the form of the `soilderivedobject` layer.

A subform showing the geometries linked to the current Soil Derived Object is added to the following section:

**GEOMETRY**

From the form of a Soil Derived Object, users can therefore:

- view the linked geometries;
- add a new child geometry;
- open and edit an existing geometry;
- check the association between the Soil Derived Object and its geometries.

If the **GEOMETRY** section is not present in the form, the tool adds the subform to the main section of the form.

## Saving the Project

When processing is complete, the tool attempts to save the QGIS project automatically.

However, it is **strongly recommended** that you also save it manually by selecting:

**Project → Save**

Saving the project preserves:

- the relation between the two layers;
- the relation reference field;
- the form layout;
- the position of the new layer in the Layers panel;
- the geometry subform added to SoilDerivedObject.

If the relation field does not update immediately, save the project, close it and reopen it in QGIS.

## Warnings

### Use the Correct SoilWise Project

The active QGIS project must contain a valid `soilderivedobject` vector layer with a `guid` field.

The `soilderivedobject` layer must come from a valid GeoPackage. If these requirements are not met, the tool stops without creating the new table.

### Do Not Rename the Table Manually

Do not rename the table directly in the GeoPackage after it has been created.

The relation and project settings use the name assigned during creation. Renaming the table manually may break the link to the `soilderivedobject` layer.

### Do Not Delete the `guid_soilderivedobject` Field

The `guid_soilderivedobject` field is required for the link to SoilDerivedObject. Deleting it makes the relation unusable.

### Do Not Edit the `guid` Field Manually

The `guid` value is generated automatically and uniquely identifies the geometry record.

Once assigned, it must not be changed manually.

### Do Not Change the Geometry Column

The geometry column is named `geometry` and is registered in the GeoPackage metadata.

Do not rename or delete this column because doing so would prevent QGIS and QField from correctly recognising the layer as a spatial table.

### Do Not Change the Structure During Fieldwork

If the GeoPackage has already been transferred to a device running QField, avoid making simultaneous changes on the computer to:

- the table structure;
- field names;
- relations;
- the geometry type.

Before making structural changes, transfer all field data back to the computer and create a backup copy.

## Summary

The **Create SDO Geometry Layer** tool automates the creation of a geometry layer linked to SoilDerivedObject in the SoilWise GeoPackage.

In a single operation, it:

- creates a new spatial table;
- allows the user to choose among six geometry types;
- names the geometry column `geometry`;
- assigns the EPSG:3035 coordinate reference system;
- generates the required identifiers;
- links the layer to `soilderivedobject`;
- configures the **Soil Derived Object** relation field;
- enables the button for opening the related parent form;
- prepares the form of the new layer;
- adds the child subform to the **GEOMETRY** section of SoilDerivedObject;
- loads the layer into the **Soil Derived Object** group;
- attempts to save the project configuration.

To preserve the result correctly, use the intended SoilWise project, select the appropriate geometry type, choose a valid name for the new layer, and save the QGIS project after running the tool.
