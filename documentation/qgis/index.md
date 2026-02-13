# Introduction to the Custom Forms for Geopackage Data Entry in QGIS

**QGIS** is a fully **Open Source** Geographic Information System designed for the visualization, editing, and analysis of both spatial and non‑spatial data. One of its most relevant capabilities is the ability to display **geometries and attribute tables together**, leveraging **customizable attribute forms** that can be tailored to project‑specific data‑entry workflows.

These features make QGIS an excellent choice as the **primary user-facing environment**  for consulting and updating the contents of the Geopackage. Custom forms provide a controlled, guided interface for data entry, ensuring consistency with the underlying data model and validation rules.

## Alternative Access and Data Processing Tools

While QGIS serves as the main interface for interactive data inspection and editing, it is **not the only tool** capable of interacting with the Geopackage. The database structure can also be accessed, queried, or transformed through:

- **RDBMS systems** that support the Geopackage standard  
- **R**, for statistical analysis and data manipulation  
- **Python**, which enables full **CRUD (Create, Read, Update, Delete)** workflows and automated data-processing pipelines  
- **ETL (Extract, Transform, Load) tools**, such as **HALE Studio**, **KNIME**, and other similar platforms, which provide controlled transformation logic, conceptual schema mapping, validation workflows, and repeatable integration pipelines

These complementary tools support more complex data‑processing scenarios such as bulk imports, schema transformations, QC procedures, and integration with external systems.

For a complete understanding of the database structure, its constraints, relationships, and the required loading logic, users should refer to the [**Data Loading & Modelling Guide**](../data_loading.md) , which defines the conceptual and structural foundations of the Geopackage.



# Documentation Index

Below is the index of the technical documentation available in this repository:

[Open the Soilwise Geopackage in QGIS](./open_geopackage.md)  
[Renaming the Soilwise GeoPackage](./rename_geopackage.md)  
[Customized Attribute Forms in QGIS](./custom_form.md)  
[Navigating GeoPackage Tables via Forms](./navigating_via_form.md)  
[Importing Geometric Features](./importing_geometric_features.md)  
[Editing Records Through a QGIS Form](./edit_form.md)  
[Saving Edits in QGIS Forms](./save_form.md)  
[Soil Site Form](./soilsite.md)  
[Soil Plot Form](./soilplot.md)  
[Soil Profile Form](./soilprofile.md)  
[Profile Element Form](./profileelement.md)  
[Datastream Form](./datastream.md)  
[Observation Form](./observation.md)  
