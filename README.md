# SoilWise INSPIRE‑SOIL GeoPackage (GPKG)

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Zenodo DOI](https://zenodo.org/badge/doi/10.5281/zenodo.18246824.svg)](https://doi.org/10.5281/zenodo.18246824)
![QGIS](https://img.shields.io/badge/QGIS-3.44%2B-589632?logo=qgis&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite3-3-003B57?logo=sqlite&logoColor=white)
![Python](https://img.shields.io/badge/python-3.8%2B-blue)
![R](https://img.shields.io/badge/R-4.0%2B-276DC3?logo=r&logoColor=white)


## Abstract
This repository delivers a streamlined **INSPIRE‑aligned soil database** in **GeoPackage (SQLite)** format, designed to improve interoperability and usability of soil datasets across Europe. Building on the EJP SOIL INSPIRE‑SO template, SoilWise provides an updated logical model that remains faithful to the INSPIRE Soil conceptual model while being practical for GIS workflows. A key novelty is the integration pathway toward **OGC SensorThings API 2.0 (STA2)** to complement “data‑at‑rest” soil datasets with “data‑in‑motion” observation time series, enabling near‑real‑time interoperability for soil monitoring networks and services. Overall, the combined approach supports **FAIR** soil information sharing by reducing provider burden and improving consumer usability in support of EU soil policies and monitoring needs.

---

## Knowledge Sources
This work is based on the following primary normative and technical resources:

- INSPIRE Soil theme: conceptual model (UML), feature catalogue, and implementation guidance (INSPIRE Soil Technical Guidelines).
- INSPIRE Good Practice: GeoPackage encoding for INSPIRE datasets.
- OGC GeoPackage standard.
- OGC SensorThings API 2.0 (STA2, draft) for observation/time‑series exposure via HTTP/MQTT.
- OMS / ISO 19156:2023 alignment for observation semantics.
- EJP SOIL INSPIRE‑SO GeoPackage template as baseline.

---

## Conceptual Model
The GeoPackage schema is a **relational transposition** of the INSPIRE Soil conceptual model (UML), preserving the core “soil investigation chain” and relationships:

- **SoilSite** → context/area of investigation  
- **SoilPlot** → investigation point/portion within a SoilSite  
- **SoilProfile** → vertical profile at a plot  
- **ProfileElement** → horizon or layer within a profile  
- **SoilBody** → mapping unit concept, linked to representative profiles

In addition, the model includes an **observational component** aligned with **STA2** concepts (e.g., Things, Datastreams, Observations, Sensors, ObservedProperties) to support time‑series exposure via standards‑based services.

<p>
  <img src="documentation/assets/db_structure.webp"
     style="width: 100%; height: auto; display: block;">
</p>

---

## Overview of the INSPIRE‑SOIL GeoPackage
The SoilWise GeoPackage is designed to be:

- **GIS‑native**: editable and viewable directly in QGIS and other GeoPackage‑capable GIS tools.
- **Relational and traceable**: UML classes map to tables; UML associations map to foreign keys and link tables.
- **Semantically controlled**: code lists and controlled vocabularies are materialized as reference tables.
- **Interoperable across static + dynamic flows**:
  - **GeoPackage** provides structured “data‑at‑rest” soil datasets.
  - **STA2 alignment** provides a pathway to “data‑in‑motion” observation streams.

---
## Installation / Access
No installation is required. A GeoPackage is a single, portable file (.gpkg) that you simply download and open. GeoPackage is an SQLite database container, so its content can be accessed and updated directly without intermediate format conversions.

### Recommended usage (GIS)
The GeoPackage can be used in any GIS that supports the GeoPackage format. We recommend QGIS as the reference GIS, and this repository provides custom QGIS forms and styles to facilitate data entry and visualization. 

### Programmatic access (R / Python)
Because a GeoPackage is an SQLite database file, it can also be accessed programmatically using standard SQLite tooling in R or Python (e.g., via SQLite drivers / bindings). 
In R, GeoPackage workflows are commonly supported via GDAL-backed packages and SQLite interfaces (e.g., terra/RSQLite-based tooling).

### Recreating the GeoPackage from scratch (optional)
It is also possible to rebuild your own GeoPackage from zero by starting from an empty GeoPackage and executing the SQL scripts shipped with this repository (DDL/DML and metadata population). 

Open the empty GeoPackage model: http://www.geopackage.org/data/empty.gpkg (e.g., with a DB manager like DBeaver).
Execute the SQL instructions using the provided SQL files (located in geopackage_ddl/):

- DDL_SO.sql — creates the full SoilWise database structure (tables + relationships).
- META_SO.sql — populates GeoPackage metadata (non‑INSPIRE format) to support read/write operations.
- DML_SO.sql — populates the codelist support table required for correct functionality.
- DML_SO_PPU_Glosis.sql — imports GLOSIS-compliant soil properties/procedures and related units of measure.

> [!NOTE]
> The SQL scripts required to recreate the GeoPackage are available in this repository under the geopackage_ddl/ folder

---

## Repository Contents
```
.
├── README.md
├── Gemfile
├── documentation/            # Living technical documentation (model, guides, table reference)
├── geopackage/               # Example output .gpkg files for testing/validation
├── geopackage_ddl/           # SQL scripts: DDL/DML for schema creation and codelist/metadata population
└── .github/workflows/        # Automation (if applicable)
```

Key documentation entry point:
- `documentation/index.md` — overview, modelling rationale, loading guide, QGIS manual, and database tables reference.

---

## Soilwise-he project
This work has been initiated as part of the [Soilwise-he](https://soilwise-he.eu) project. The project receives
funding from the European Union’s HORIZON Innovation Actions 2022 under grant agreement No.
101112838. Views and opinions expressed are however those of the author(s) only and do not necessarily
reflect those of the European Union or Research Executive Agency. Neither the European Union nor the
granting authority can be held responsible for them.

---
---
---

## 🛠️ Pipeline (From model → GeoPackage → services)
Instead of a single “build pipeline”, this repository provides **repeatable assets** to implement and maintain the database consistently:

1. **Schema definition (DDL)** — SQL scripts to create the GeoPackage schema and supporting tables.
2. **Codelist and metadata population (DML)** — SQL scripts to populate codelist tables and field‑level metadata.
3. **Templates & example outputs** — sample `.gpkg` files for testing and validation.
4. **QGIS user experience layer** — custom forms and styles to support editing and reduce errors.
5. **STA2 exposure (implementation pathway)** — structural alignment enabling transfer between GeoPackage tables and STA2 entities.

---

## 📊 Quality, Validation & Compliance
This repository aims to support:

- INSPIRE‑faithful modelling derived from the INSPIRE conceptual model and feature catalogue.
- INSPIRE GeoPackage publishing good practice.
- STA2 readiness for observations/time series.
- Operational usability in GIS through QGIS forms.

> **QGIS requirement**: QGIS **3.44.0 “Solothurn”** (or higher) is required to fully use the provided custom forms and workflows.

---



---

## 🚀 Quick Start

### 1) Clone the repository
```bash
git clone https://github.com/soilwise-he/Geopackage-so.git
cd Geopackage-so
```

### 2) Open a sample GeoPackage in QGIS
- Start QGIS (>= 3.44.0)
- **Layer → Add Layer → Add Vector Layer…**
- Browse to `geopackage/` and open a `.gpkg` file

### 3) Create a fresh empty database (optional)
- Create a new empty `.gpkg` in your GIS or with SQLite tooling
- Run the schema scripts from `geopackage_ddl/`:
  - **DDL**: create tables/constraints
  - **DML**: populate codelists and field metadata (where provided)

### 4) Enable the QGIS forms and styles
- Import/apply assets in `qgis_style/`
- Use the guided forms for consistent data entry (dropdowns, validation rules, grouped tabs)

### 5) Follow the loading order and constraints
If you need to populate or update a database, start from the **Data Loading & Modelling Guide** referenced in `documentation/index.md`.

---

## 🔗 Resource Availability
- Repository: https://github.com/soilwise-he/Geopackage-so
- Model diagram (dbdiagram): https://dbdiagram.io/d/SoilWise_Geopackage-69399847e877c6307451317a
- SoilWise data & knowledge hub (context): https://repository.soilwise-he.eu/

---

## 🔗 Imported Schemas & Standards
- INSPIRE Soil conceptual model (UML) and feature catalogue
- INSPIRE GeoPackage encoding good practice
- OGC GeoPackage
- OGC SensorThings API 2.0 (STA2, draft)
- OMS / ISO 19156:2023

---

## 🔗 Linked Vocabularies & Codelists
The GeoPackage includes (or is designed to include) reference tables supporting controlled vocabularies such as INSPIRE code lists, horizon/layer notations, WRB qualifiers, and units of measure.

---

## 💡 Use Cases
- INSPIRE‑aligned soil dataset exchange in a GIS‑friendly container (GeoPackage/SQLite).
- Operational data entry and curation in QGIS using guided forms and styles.
- Soil monitoring workflows where structured datasets (“data‑at‑rest”) are complemented with STA2‑aligned observation time series (“data‑in‑motion”).
- Testing and validation of harmonized soil datasets against a consistent relational schema.

---

## 🗣️ Feedback
- Please open a GitHub Issue for bugs, enhancements, or missing elements.

---

## 🧾 How to Cite
> SoilWise HE (2026). *INSPIRE‑SOIL GeoPackage (GPKG) with STA2 alignment*. GitHub repository: https://github.com/soilwise-he/Geopackage-so

---

## 🙏 Acknowledgements
This work was developed in the context of the **SoilWise** Horizon Europe project (grant agreement **101112838**).

---

## 🧾 To‑do
See **Issues** for planned tasks and enhancements.

---

## 📄 License (recommended)
A practical dual‑license approach (commonly used for mixed repositories) is recommended:

- **Code / scripts** (SQL, automation, helper scripts): **MIT License**
- **Data / schema / documentation / GeoPackage templates & examples**: **CC BY 4.0**

Suggested setup:
- Add `LICENSE` (MIT)
- Add `LICENSE-CC-BY-4.0` (CC BY 4.0)
- Keep this section in the README to clarify which assets fall under which license.

---

## How to add the diagram image (recommended for GitHub)
GitHub README pages do **not** render interactive embeds (e.g., iframes). Instead:

1. Export the diagram from dbdiagram as **PNG** (or **PDF** / **SVG** if available).
2. Commit it to this repository, e.g. `documentation/assets/SoilWise_Geopackage.png`
3. Embed it in the README with:

```markdown
[![SoilWise GeoPackage data model](documentation/assets/SoilWise_Geopackage.png)](https://dbdiagram.io/d/SoilWise_Geopackage-69399847e877c6307451317a)
```


