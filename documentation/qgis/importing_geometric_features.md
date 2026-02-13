# Importing Geometric Features (QGIS‑only)

This guide describes two **QGIS‑only** ways to load geometry into the *soilwise* GeoPackage:  
1) a **native “out‑of‑the‑box” workflow** using standard QGIS tools, and  
2) a **simplified workflow via a dedicated QGIS plugin** (wizard‑style).  

## Geometry-bearing features in _soilwise_

- **Soil Site** — *polygon*
- **Soil Plot** — *point*
- **Soil Body** — *geometry as defined by the official schema* (e.g., polygon/multipolygon; confirm against the model)
- **Soil Derived Object** — *polygon*
- **Datastream** — *polygon (optional)*

> [!NOTE]
> Before importing, confirm that each source layer’s **geometry type** (including multipart vs singlepart) matches the _soilwise_ model expectation.

> [!WARNING]
>The target GeoPackage expects layers in **ETRS89 / LAEA Europe (EPSG:3035)** and enforces geometry types and integrity via constraints/triggers defined in the schema.

## Prerequisites & Best Practices (apply to all methods)

- **CRS**: use the project-wide, agreed coordinate reference system and reproject sources **before** import when needed.
- **Valid geometries**: ensure geometry validity/topology (closed polygons, no self‑intersections, points within expected extent). QGIS provides native tools for validation and repair.
- **Attribute schema**: align field names, data types, and constraints (PK/UK, domains/codelists) prior to import.
- **IDs/keys**: guarantee primary key uniqueness; avoid duplicates.
- **Multipart vs singlepart**: respect the target model.
- **Logging & traceability**: record source, date, version, and procedure used (SQL, QGIS, plugin) for reproducibility and audits.

## Method 1 — QGIS: copy/paste features into the GeoPackage


The import of geometries into an existing table of a GeoPackage with QGIS is structured in three main steps:

1. **Importing** geometries as a layer in QGIS  
2. **Copying** the imported geometries  
3. **Pasting** the geometries into the GeoPackage layer  

Let’s look at each step in detail.


### 1) Importing geometries

QGIS allows the import of geometries from various formats, such as CSV, Shapefile, or other GeoPackages. In this example, we will import data from a CSV file.

<p>
  <img src="../assets/geom_01.webp"
       alt="Fig.1" align="left" width="500">
Click the <strong>Open Data Source Manager</strong> button ① in the QGIS toolbar.
</p>
<br clear="all"><br>

<p>
  <img src="../assets/geom_02.webp"
       alt="Fig.1" align="left" width="500">
  In the window that opens, you can choose from various data sources to import.
  In our example, select <strong>CSV</strong> ② as the source format and proceed with importing the desired file ③.
   
  Check the <strong>geometry type</strong> (e.g., WKT or coordinates separated into latitude/longitude) ④.
  
  Set the correct <strong>Coordinate Reference System (CRS)</strong> ⑤.
  
  Click <strong>Add</strong> ⑥ to create the layer (in this case, a point layer) in the project.
</p>
<br clear="all"><br>


> [!WARNING]
> For the copy–paste operation to work correctly, the **source layer** (from which geometries are copied) must have the **same fields** (name and data type) as the **destination layer**, or at least match the required fields.  
> This check can be done during the import phase, later using QGIS tools, or by using an RDBMS to modify or remove unnecessary fields.  
> In this example, since the check was not performed during import, a temporary support layer named **“copy”** was created and used for preprocessing.



### 2) Copying geometries


<p>
  <img src="../assets/geom_03.webp"
       alt="Fig.1" align="left" width="500">
Import the newly created layer (if it is not already present in the project).
   
Right‑click the layer name ⑦ and, from the context menu, select <strong>Open Attribute Table</strong> ⑧ to view its data.
</p>
<br clear="all"><br>


<p>
  <img src="../assets/geom_04.webp"
       alt="Fig.1" align="left" width="500">
<strong>Select all</strong> geometries ⑨
   
<strong>Copy</strong> the geometries ⑩.
</p>
<br clear="all"><br>

### 3) Pasting geometries

<p>
  <img src="../assets/geom_05.webp"
       alt="Fig.1" align="left" width="500">
Enable <strong>editing mode</strong> on the destination GeoPackage layer using the <strong>Toggle Editing</strong> button ⑪.  
   
<strong>Paste</strong> the geometries. ⑫

<strong>Save</strong> the changes.
</p>
<br clear="all"><br>


## Method 2 — QGIS: import via Plugin (simplified workflow)


### QGIS plugin support

To support this procedure, there is also a QGIS plugin that simplifies and extends the described steps.  
It is called **AppendFeaturesToLayer** and is available at:  
[github.com/gacarrillor/AppendFeaturesToLayer](https://github.com/gacarrillor/AppendFeaturesToLayer)

The plugin includes two geoprocessing tools:

- **ETL_LOAD** — allows easy matching of the source table with the destination geometry table.

  <p>
  <img src="../assets/geom_06.webp"
       alt="Fig.1" align="left" width="500">

</p>

<br clear="all"><br>


- **Append Feature To Layer** — handles the copy–paste operation with additional advanced options.

  <p>
  <img src="../assets/geom_07.webp"
       alt="Fig.1" align="left" width="500">

</p>

<br clear="all"><br>



## Alternative ETL tools (optional)

For complex mappings, schema reconciliation, and automation, consider:

- **HALE Studio** — Open‑source ETL focused on harmonization toward standards (OGC/INSPIRE), with integrated validation and broad format support; helpful to define and document repeatable transformations. 
  <https://wetransform.to/halestudio/>

- **KNIME** — Visual data/ETL platform with hundreds of connectors/nodes; suitable for reusable and scheduled pipelines, DB integrations, and more. 
  <https://www.knime.com/>

