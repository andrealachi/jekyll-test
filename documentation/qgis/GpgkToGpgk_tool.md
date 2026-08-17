# Installing and Using the GpkgToGpkg Model in QGIS

## Introduction

**GpkgToGpkg** is a QGIS Processing model designed to transfer data from a source GeoPackage, referred to as the **SOURCE**, to a destination GeoPackage, referred to as the **TARGET**.

During execution, the model compares the tables contained in the two GeoPackages and identifies records that are available in the SOURCE but are not yet present in the TARGET. Only the missing records are appended to the destination GeoPackage.

The comparison is primarily based on the `guid` field, which is used as a unique identifier. For some junction tables, the comparison is instead based on a combination of multiple fields. This approach allows the model to be run repeatedly on the same datasets while reducing the risk of creating duplicate records.

The model therefore performs an **incremental synchronization**:

- it appends missing records to the TARGET;
- it does not overwrite existing tables;
- it does not update records that already have the same key;
- it does not delete records from the TARGET;
- it does not modify the source GeoPackage.

The model has a modular structure. In addition to the required parameters, users can select which data groups should be transferred, choosing among Soil Sites, observed Soil Profiles, derived Soil Profiles, Soil Bodies, and Soil Derived Objects.

This selection is managed through QGIS **Conditional Branch** algorithms. Based on the enabled parameters, the model runs only the required workflow sections, including the associated descriptive tables, relationships, datastreams, and observations.

> [!IMPORTANT]
> The TARGET GeoPackage must already exist and must use a structure that is compatible with the SOURCE. In particular, it must contain the tables expected by the model.



## Available Methods for Running the Model

The model can be used in QGIS in two ways:

1. by installing it in the **Processing Toolbox**, so that it remains permanently available;
2. by opening it directly in the **Model Designer**, where it can be run, inspected, or modified.

Installing the model in the Processing Toolbox is the recommended approach for routine use. The Model Designer is more appropriate when the internal structure of the model needs to be examined or when changes must be made to the processing workflow.



# Part I. Installing the Model in the Processing Toolbox

## Downloading the Model

Download the model file from the GitHub repository. The file uses the following extension:

```text
.model3
```

Keep the original file extension unchanged.



## Locating the QGIS Models Folder

To avoid loading the `.model3` file manually every time the tool is required, copy it to the folder used by QGIS to store Processing models.

Follow these steps:

<p>
  <img src="../assets/form_01.webp"
       alt="Fig.1" align="left" width="500">
Start QGIS. <br>
① Open the <strong>Settings</strong> menu. <br>
② Select <strong>Options</strong>.<br>
In the <strong>Options</strong> dialog, select ③ <strong>Processing</strong> from the panel on the left.<br>
On the right-hand side of the dialog, expand the <strong>Models</strong> section.<br>
Locate the ④ <strong>Models folder</strong> setting.<br>
The path displayed under <strong>Models folder</strong> identifies the directory in which QGIS searches for user-installed Processing models.
</p>
<br clear="all"><br> 





Copy the downloaded `GpkgToGpkg.model3` file into this folder.

> [!NOTE]
> The folder path may vary depending on the operating system, QGIS version, and active QGIS user profile. For this reason, use the path displayed in the QGIS options instead of relying on a predefined system path.


## Displaying the Model in the Processing Toolbox

After copying the file to the models folder, return to the main QGIS window.

If the **Processing Toolbox** is not visible, open it from:

**Processing → Toolbox**

In the Processing Toolbox, expand:

⑤ **Models → SoilWise Geopackage Tools**

The following model should be available in this subgroup:

⑥ **GpkgToGpkg**

If the model does not appear immediately, refresh the Processing Toolbox or restart QGIS.


## Opening the Model Execution Dialog

Double-click **GpkgToGpkg** in the Processing Toolbox.

QGIS opens the model execution dialog. In this dialog, specify:

- the source GeoPackage;
- the target GeoPackage;
- the optional log folder;
- the data groups to be copied.

The right-hand side of the dialog also contains the model help panel. This panel provides a brief description of the tool and explains the purpose of the available parameters.

---

# Part II. Configuring the Parameters

## Main Parameters

### Source GeoPackage

The **Source GeoPackage** parameter identifies the source dataset.

This is the database from which the model reads the records that must be checked and, when necessary, copied.

The SOURCE:

- is accessed for reading;
- contains the data to be transferred;
- must include the tables expected by the model;
- is not modified by the append operations.

To set this parameter, select the source file with the `.gpkg` extension.

```text
Example: database_source.gpkg
```

### Target GeoPackage

The **Target GeoPackage** parameter identifies the destination dataset.

The model compares the SOURCE tables with the corresponding TARGET tables and appends only the records that are missing from the TARGET.

The TARGET must:

- exist before the model is run;
- have a structure consistent with the SOURCE;
- contain the tables required by the model;
- be writable;
- not be locked by another application.

If a record with the same key is already present in the TARGET, it is not copied again.

```text
Example: database_target.gpkg
```

> [!WARNING]
> The model does not create a complete copy of the SOURCE file and does not automatically create the full TARGET schema. Its purpose is to append missing records to a target GeoPackage that has already been prepared.

### Log Folder

The **Log Folder** parameter identifies the directory in which the execution log will be saved.

The log can be used to:

- verify the outcome of the processing operation;
- investigate errors;
- review which algorithms were executed;
- retain an audit trail of the operation;
- document processing runs involving large datasets.

If a valid folder is selected, the model saves the QGIS Processing log in that folder.

If the parameter is left empty:

- data transfer can still be performed;
- the log export workflow is not enabled;
- no log file is created in a user-selected folder.

Log export is controlled by the **LOG** conditional branch, which checks that the path is not null and is not an empty string.

---

## Data Selection Parameters

The following parameters allow the transfer to be restricted to specific data groups.

They can be enabled individually or in combination. The selected combination determines which sections of the model are executed.

### Copy Soil Site

The **Copy Soil Site** parameter controls the transfer of data related to Soil Sites.

#### When the parameter is enabled

The model copies:

- records from the `soilsite` table that are not already present in the TARGET;
- datastreams associated with the selected Soil Sites;
- observations linked to those datastreams.

The workflow therefore includes the Soil Site entity and its associated observational data.

#### When the parameter is disabled

The model does not copy:

- `soilsite` records;
- datastreams linked to Soil Sites;
- observations associated with those datastreams.

Disabling this parameter does not prevent other selected data groups from being processed.

### Copy Observed Soil Profile

The **Copy Observed Soil Profile** parameter controls the transfer of observed Soil Profiles.

Observed profiles are identified by the following condition:

```text
isderived = 0
```

#### When the parameter is enabled

The model copies:

- missing observed Soil Profiles;
- associated Soil Plots;
- related Profile Elements;
- associated descriptive and classification tables;
- datastreams related to profiles and profile elements;
- observations linked to those datastreams;
- relationships with Soil Derived Objects, when **Copy Soil Derived Object** is also enabled.

#### When the parameter is disabled

Profiles where `isderived = 0` are not appended to the TARGET.

Workflow sections that specifically depend on observed profiles, such as Soil Plots and some relationships with Soil Derived Objects, are also skipped.

### Copy Derived Soil Profile

The **Copy Derived Soil Profile** parameter controls the transfer of derived Soil Profiles.

Derived profiles are identified by the following condition:

```text
isderived = 1
```

#### When the parameter is enabled

The model copies:

- missing derived Soil Profiles;
- associated Profile Elements;
- related descriptive and classification tables;
- datastreams related to profiles and profile elements;
- linked observations;
- the relationship with Soil Bodies, when **Copy Soil Body** is also enabled.

#### When the parameter is disabled

Profiles where `isderived = 1` are not appended to the TARGET.

Relationships that require derived profiles are also not copied.

### Relationship Between Observed and Derived Profiles

When both of the following parameters are enabled:

- **Copy Observed Soil Profile**;
- **Copy Derived Soil Profile**;

the model also enables the transfer of the `isderivedfrom` table.

This table represents the relationship between a derived profile and the observed profile from which it was derived.

The relationship is copied only when both profile groups are selected. This prevents the TARGET from receiving a relationship that points to a profile that has not been included.

The possible combinations are therefore:

- **observed profiles only:** `isderivedfrom` is not copied;
- **derived profiles only:** `isderivedfrom` is not copied;
- **observed and derived profiles:** both groups and `isderivedfrom` are copied;
- **neither group selected:** the Soil Profile workflow is not executed.

### Copy Soil Body

The **Copy Soil Body** parameter controls the transfer of data related to Soil Bodies.

#### When the parameter is enabled

The model copies:

- records from the `soilbody` table;
- geometries stored in `soilbody_geom`;
- `derivedprofilepresenceinsoilbody`, when derived Soil Profiles are also enabled;
- `isbasedonsoilbody`, when Soil Derived Objects are also enabled.

#### When the parameter is disabled

The model does not copy:

- Soil Body records;
- their geometries;
- relationships that depend on the presence of Soil Bodies.

This parameter can be combined with **Copy Derived Soil Profile** and **Copy Soil Derived Object** to preserve relationships between the corresponding domains.

### Copy Soil Derived Object

The **Copy Soil Derived Object** parameter controls the transfer of Soil Derived Objects.

#### When the parameter is enabled

The model copies:

- records from the `soilderivedobject` table;
- associated geometries;
- records from the `isbasedonsoilderivedobject` relationship table;
- datastreams linked to Soil Derived Objects;
- observations associated with those datastreams;
- `isbasedonobservedsoilprofile`, when observed Soil Profiles are also enabled;
- `isbasedonsoilbody`, when Soil Bodies are also enabled.

#### When the parameter is disabled

The model does not copy:

- Soil Derived Objects;
- their related datastreams;
- linked observations;
- relationships that depend on the presence of Soil Derived Objects.


## Selecting Parameter Combinations

The parameters can be used independently, but some combinations preserve a more complete set of relationships.

### Soil Sites Only

Enable:

- **Copy Soil Site**

The model copies Soil Sites, related datastreams, and linked observations.

### Observed Soil Profiles Only

Enable:

- **Copy Observed Soil Profile**

The model copies profiles where `isderived = 0`, Soil Plots, and the associated descriptive and observational data.

### Derived Soil Profiles Only

Enable:

- **Copy Derived Soil Profile**

The model copies profiles where `isderived = 1` and the related data.

### Observed and Derived Soil Profiles

Enable:

- **Copy Observed Soil Profile**;
- **Copy Derived Soil Profile**.

The model copies both groups and the `isderivedfrom` relationship.

### Soil Bodies and Derived Soil Profiles

Enable:

- **Copy Soil Body**;
- **Copy Derived Soil Profile**.

The model can also copy the `derivedprofilepresenceinsoilbody` relationship.

### Soil Derived Objects and Observed Soil Profiles

Enable:

- **Copy Soil Derived Object**;
- **Copy Observed Soil Profile**.

The model can also copy the `isbasedonobservedsoilprofile` relationship.

### Soil Derived Objects and Soil Bodies

Enable:

- **Copy Soil Derived Object**;
- **Copy Soil Body**.

The model can also copy the `isbasedonsoilbody` relationship.

> [!TIP]
> To preserve all relationships between two domains, enable both relevant parameters. Otherwise, the model may omit the relationship table to avoid references to entities that are not present in the TARGET.


## Running the Model from the Processing Toolbox

After configuring all parameters:

1. verify that the correct SOURCE and TARGET have been selected;
2. confirm that the selected TARGET is the GeoPackage that should be modified;
3. select a log folder if a saved execution log is required;
4. enable the data groups to be copied;
5. click **Run**.

During execution, QGIS displays information about the algorithms being run in the Processing log panel.

After the model has finished, it is good practice to:

- verify that processing completed without errors;
- review the saved log, if requested;
- check the number of records in the TARGET tables;
- confirm that relationships between tables have been preserved;
- check for orphaned references.

> [!TIP]
> Before running the model on production data for the first time, create a backup copy of the TARGET GeoPackage.


# Part III. Opening and Running the Model in the Model Designer

## When to Use the Model Designer

The model can be opened directly in the Model Designer without first installing it in the models folder.

This method is useful when you need to:

- run the model occasionally;
- inspect its structure;
- review dependencies between algorithms;
- inspect the SQL expressions;
- modify parameters;
- modify Conditional Branch algorithms;
- add or remove workflow steps;
- create a customized version of the model.

The Model Designer represents a sequence of processing algorithms as a single workflow that can later be executed with different input data.


## Opening the Model

After downloading the `.model3` file:

1. start QGIS;
2. open the **Processing** menu;
3. select **Model Designer**;
4. in the Model Designer window, select **Open Model**;
5. use the file browser to locate the downloaded model;
6. select `GpkgToGpkg.model3`;
7. confirm the selection.

The model is displayed in the Model Designer workspace, where the following elements are visible:

- input parameters;
- comparison algorithms;
- join operations;
- field selection operations;
- copy operations;
- dependencies between algorithms;
- Conditional Branch algorithms.


## Running the Model from the Model Designer

To run the model from the Model Designer:

1. select **Run Model** or click the **Run** button;
2. wait for the parameter dialog to open;
3. set the Source GeoPackage, Target GeoPackage, and Log Folder parameters;
4. select the data groups to be copied;
5. click **Run**.

The parameter dialog is the same as the one displayed when the model is launched from the Processing Toolbox.

The following keyboard shortcuts are also available:

- **F5** runs the entire model;
- **Shift + F5** runs only the selected model steps.

Running selected steps is particularly useful when developing, testing, or troubleshooting the model.


## Modifying the Model

Opening the model in the Model Designer allows any of its components to be modified, including:

- the SQL queries used to identify missing records;
- the keys used for comparisons;
- the tables included in the workflow;
- the fields retained in intermediate outputs;
- the conditions that enable individual workflow sections;
- the GDAL options used for append operations;
- dependencies between algorithms;
- parameters displayed to the user;
- the model's internal help documentation.

Before saving a modified version, it is advisable to:

1. use a different file name;
2. retain a copy of the original model;
3. validate the model;
4. test it on copies of the GeoPackages;
5. verify the referential integrity of the output.

> [!WARNING]
> Changes to SQL queries, comparison keys, or conditional dependencies may affect duplicate prevention or produce incomplete relationships. Refer to the model's technical documentation for a detailed explanation of its internal structure and workflow logic.


## Conclusion

The **GpkgToGpkg** model can be integrated into the QGIS Processing Toolbox so that it remains permanently available and can be accessed as a standard Processing tool.

The execution dialog allows the user to specify the source and target GeoPackages and to select only the required data groups. Conditional Branch algorithms interpret these selections and enable the relevant sections of the model.

For routine use, installing the `.model3` file in the QGIS models folder is the recommended approach. The Model Designer is the more appropriate environment for inspecting, validating, or modifying the structure of the model.


## References

- [QGIS User Guide: The Model Designer](https://docs.qgis.org/latest/en/docs/user_manual/processing/modeler.html)
- [QGIS User Guide: Processing Framework](https://docs.qgis.org/latest/en/docs/user_manual/processing/index.html)
