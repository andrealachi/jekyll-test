# SO GeoPackage (SQLite3) — Tables & Schema Overview

This folder documents the **database tables** of the *SoilWise INSPIRE Soil (SO) GeoPackage*, delivered as an **OGC GeoPackage** implemented on top of **SQLite3**. [3](https://github.com/soilwise-he/Geopackage-so/blob/main/documentation/index.md)[1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
The schema combines:

- an INSPIRE Soil–oriented relational core (Site → Plot → Profile → Profile Elements), and [4](https://github.com/soilwise-he/Geopackage-so/blob/main/documentation/data_loading.md)[1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- a **SensorThings-like** observation layer (Datastream/Observation) to publish soil observations as interoperable time series (aligned with the repository scope and STA 2.0 draft direction). [2](https://github.com/soilwise-he/Geopackage-so)[1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)

This page provides a **technical entry point**: how the schema is organised, which entities are central, and which integrity constraints and triggers are enforced.

---

## Quick index (main tables)

> Links are **relative** to the current directory (`documentation/tables/`).  
> If your filenames differ (e.g., `soilsite/index.md`), update the link targets only.

### Code lists / domains
- **[codelist](codelist.md)** — controlled values repository used to validate domains (INSPIRE codelists + local “Category” collections). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)

### INSPIRE Soil core (Site → Plot → Profile → Element)
- **[soilsite](soilsite.md)** — investigation site geometry (POLYGON) and purpose. [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- **[soilplot](soilplot.md)** — plot / sampling point (POINT), located on a Soil Site. [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- **[soilprofile](soilprofile.md)** — soil profile (Observed vs Derived), linked to a plot (Observed only). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- **[profileelement](profileelement.md)** — vertical elements (Horizon/Layer) belonging to a profile, with depth constraints and domain checks. [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

### Observations (SensorThings-like)
- **[datastream](datastream.md)** — observation “channel”: what is observed (ObservedProperty), how (Procedure/Sensor), and on which Feature of Interest (FOI). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)[2](https://github.com/soilwise-he/Geopackage-so)  
- **[observation](observation.md)** — time-stamped results, constrained by `datastream.type` (Quantity/Category/Boolean/Count/Text). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

---

## 1) GeoPackage as a technical container (SQLite + OGC metadata)

A `.gpkg` file is an SQLite database that includes **OGC GeoPackage metadata tables** (e.g., `gpkg_contents`, `gpkg_geometry_columns`) to make layers directly usable in GIS software. [3](https://github.com/soilwise-he/Geopackage-so/blob/main/documentation/index.md)[1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

In this implementation:
- feature layers are registered in `gpkg_contents` (per-table records), [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- geometry types and spatial reference metadata are registered in `gpkg_geometry_columns`, [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- geometry columns are indexed (e.g., `idx_soilsite_geom`, `idx_soilplot_geom`). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

**Spatial reference:** the schema registers and uses **ETRS89 / LAEA Europe (EPSG:3035)** for the main geometry layers (e.g., Soil Site polygons, Soil Plot points, SoilBody geometry). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

---

## 2) Cross-cutting conventions (GUIDs, temporal validity, domains)

### 2.1 GUIDs: autogenerated and immutable
Most business tables expose a `guid` column (unique). If `guid` is NULL on INSERT, triggers generate a UUID-like lowercase value. Updates of `guid` are blocked by dedicated triggers. [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

### 2.2 Validity and lifecycle timestamps
Where required by the conceptual model, tables include:
- `validfrom` / `validto` with checks enforcing `validfrom <= validto`, [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- `beginlifespanversion` / `endlifespanversion` with consistency checks and automatic refresh of `beginlifespanversion` upon relevant updates. [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

### 2.3 Domain enforcement via `codelist`
Many domain attributes are not free text: triggers validate that the stored value exists in `codelist.id` for a given `codelist.collection` (e.g., `SoilInvestigationPurposeValue`, `SoilPlotTypeValue`, WRB/FAO codelists, and the local `Category` collection for categorical observations). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
**Implication:** load the necessary `codelist` collections before inserting domain records. [4](https://github.com/soilwise-he/Geopackage-so/blob/main/documentation/data_loading.md)[1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

---

## 3) INSPIRE Soil core backbone (Site → Plot → Profile → ProfileElement)

The schema follows the operational flow widely used for soil field/legacy datasets:

### 3.1 Soil Site (`soilsite`)
Represents an investigation area:
- geometry: **POLYGON** (`geometry`), [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- mandatory `soilinvestigationpurpose` validated against codelists. [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

### 3.2 Soil Plot (`soilplot`)
Represents a plot / sampling location:
- geometry: **POINT** (`soilplotlocation`), [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- `locatedon` is a FK to `soilsite(guid)` (plot belongs to a site). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

### 3.3 Soil Profile (`soilprofile`)
Represents a soil profile, explicitly distinguishing:
- **Observed profile** (`isderived = 0`): must have `location` NOT NULL, [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- **Derived profile** (`isderived = 1`): must have `location` NULL. [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

Observed profiles link to plots via `location → soilplot(guid)` (with cascade delete). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
WRB classification fields are guarded by consistency checks and codelist membership rules (including version-dependent collections). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

### 3.4 Profile Element (`profileelement`)
Represents vertical subdivisions within a profile:
- belongs to a profile via `ispartof → soilprofile(guid)` (cascade delete), [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- `profileelementtype` encodes:
  - `0` = **Horizon**
  - `1` = **Layer** [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)[4](https://github.com/soilwise-he/Geopackage-so/blob/main/documentation/data_loading.md)  
- depth constraints enforce coherent upper/lower limits and non-null depth information, [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- additional triggers enforce field compatibility (e.g., layer-only vs horizon-only attributes; “geogenic” constraints). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

---

## 4) Observations layer (SensorThings-like): Datastream + Observation

To manage soil measurements and indicators as time series, the schema implements:

### 4.1 Datastream (`datastream`)
Defines an observation stream:
- descriptive metadata (`name`, `definition`, `description`), [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- `type` constrained to `{Quantity, Category, Boolean, Count, Text}` with strict allowed combinations (UoM vs codespace vs bounds). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- FK block: links to `sensor`, `observedproperty`, optional `observingprocedure` (plus optional `thing`). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- FOI block: only **one** among SoilSite / SoilProfile / ProfileElement / SoilDerivedObject can be populated (enforced via CHECK). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- if `guid_observingprocedure` is present, the pair (procedure, observedProperty) must exist in `obsprocedure_obsdproperty` (enforced by triggers). [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

### 4.2 Observation (`observation`)
Stores individual results for a datastream:
- each observation references `guid_datastream` and is unique per `(phenomenontime_start, guid_datastream)`. [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
- triggers enforce result “shape” and validity **based on the linked datastream type**:
  - **Quantity:** `result_real` NOT NULL; bounds enforced if `value_min/value_max` set, [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
  - **Category:** `result_text` NOT NULL and must exist in `codelist` where `collection = datastream.codespace`, [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
  - **Boolean:** `result_boolean` ∈ {0,1}; other result fields NULL, [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
  - **Count:** `result_real` integer-like (numerically integral) plus bounds (if set), [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
  - **Text:** `result_text` NOT NULL; other result fields NULL. [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

Additionally, observation triggers keep `datastream.phenomenontime_start/end` synchronised to the MIN/MAX of linked observations. [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

### 4.3 Convenience view: `view_observation`
A dedicated view joins observations to their FOI context (site/plot/profile/element), and standardises output fields such as property name, procedure, UoM symbol, and type-specific numeric values. This is intended for reporting and extraction without re-implementing complex joins. [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  

---

## 5) Minimal SQL examples

