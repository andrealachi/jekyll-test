# Creating a Geometry Layer Linked to SoilBody

## Purpose of the Tool

The **Create a geometry layer linked to SoilBody** tool creates a new polygon layer directly in the GeoPackage used by the project and links it to the existing `soilbody` layer.

The new layer can be used to associate one or more geometries with each Soil Body stored in the database. The tool automatically performs the main required operations:

- creates the new layer in the GeoPackage;
- prepares the identifier fields;
- links the new layer to the `soilbody` layer;
- configures the data entry forms;
- adds the new layer to the QGIS project;
- adds the list of linked geometries to the SoilBody form;
- saves the project changes whenever possible.

The tool is designed specifically for the QGIS project associated with the SoilWise GeoPackage. It must not be used with projects or GeoPackages that have a different structure.

## Adding the Script to the QGIS Toolbox

### Save the Script on Your Computer

Copy the following file:

`create_soilbody_geometry_layer.py`

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
4. Locate the `create_soilbody_geometry_layer.py` file.
5. Select the file and confirm.

Once added, the tool should appear in the following Toolbox group:

**SoilWise Geopackage Tools**

with the name:

**Create a geometry layer linked to SoilBody**

To find it quickly, enter one of the following terms in the Toolbox search bar:

- `SoilBody`
- `geometry layer`

## Running the Tool

### Start the Tool

In the Processing Toolbox:

1. Expand the **SoilWise Geopackage Tools** group.
2. Double-click **Create a geometry layer linked to SoilBody**.

The tool dialog will open.

### Enter the Name of the New Layer

In the **New geometry name** field, enter the name of the new geometry layer.

The default value is:

`soilbody_newname`

Replace it with a meaningful name for the type of geometry you want to create.

The name entered becomes the actual name of the new table in the GeoPackage. Choose a clear and final name, preferably using lowercase letters and no spaces.

### Run the Process

After entering the name:

1. Click **Run**.
2. Wait for the process to finish.
3. Check that no error messages are displayed.
4. Close the tool dialog when processing is complete.

## Processing Result

### Creation of the New Layer

The tool creates a new **MULTIPOLYGON** geometry layer in the GeoPackage.

This geometry type allows a single feature to consist of:

- one polygon;
- several separate polygons belonging to the same feature.

The coordinate reference system assigned to the layer is:

**ETRS89-extended / LAEA Europe – EPSG:3035**

The geometries must therefore be consistent with the coordinate reference system used by the SoilWise project.

### Link to SoilBody

The tool automatically creates a relation between:

- the `guid_soilbody` field in the new layer;
- the `guid` field in the `soilbody` layer.

Each geometry must be linked to an existing Soil Body.

This link is mandatory. A geometry cannot be added without an associated Soil Body.

### Loading the Layer in the Layers Panel

When processing is complete, the new layer is automatically loaded into the project.

If the Layers panel contains a group named:

**Soil Body**

the tool attempts to place the new layer inside that group.

The displayed layer name corresponds to the value entered in the **New geometry name** field.

## Automatic Form Configuration

### Form of the New Layer

The tool creates a custom form for the new layer.

The form contains:

- an **ID** group containing the technical `id` and `guid` fields;
- a **Soil Body** field used to select the linked `soilbody` record.

The **ID** group is visible in QGIS Desktop and is collapsed by default because users do not normally need to edit its technical values.

The group is also configured to remain hidden in QField, making the fieldwork form simpler.

### SoilBody Layer Form

The tool also modifies the form of the `soilbody` layer.

A subform showing the geometries linked to the current Soil Body is added to the following section:

**GEOMETRY**

From the form of a Soil Body, users can therefore:

- view the linked geometries;
- add a new child geometry;
- open and edit an existing geometry;
- check the association between the Soil Body and its geometries.

If the **GEOMETRY** section is not present in the form, the tool adds the subform to the main section of the form.

## Saving the Project

When processing is complete, the tool attempts to save the QGIS project automatically.

However, it is **strongly recommended** that you also save it manually by selecting:

**Project → Save**

## Warnings

### Do Not Rename the Table Manually

Do not rename the table directly in the GeoPackage after it has been created.

The relation and project settings use the name assigned during creation. Renaming the table manually may break the link to the `soilbody` layer.

### Do Not Delete the `guid_soilbody` Field

The `guid_soilbody` field is required for the link to SoilBody. Deleting it makes the relation unusable.

### Do Not Change the Structure During Fieldwork

If the GeoPackage has already been transferred to a device running QField, avoid making simultaneous changes on the computer to:

- the table structure;
- field names;
- relations;
- the geometry type.

Before making structural changes, transfer all field data back to the computer and create a backup copy.

## Summary

The **Create a geometry layer linked to SoilBody** tool automates the creation of a multipolygon layer in the SoilWise GeoPackage.

In a single operation, it:

- creates the new geometry table;
- assigns the EPSG:3035 coordinate reference system;
- generates the required identifiers;
- links the layer to `soilbody`;
- configures the **Soil Body** field;
- prepares the form of the new layer;
- adds the subform to the **GEOMETRY** section of SoilBody;
- loads the layer into the project;
- attempts to save the project configuration.

To preserve the result correctly, use the intended SoilWise project, choose a valid name for the new layer, and save the QGIS project after running the tool.
