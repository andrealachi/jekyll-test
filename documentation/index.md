<h1 align="center">SoilWise GeoPackage</h1>


![Soilvise poster](assets/soilvise_poster.webp)

# Introduction to the **SoilWise** GeoPackage (SQLite)

The **Soilwise** GeoPackage is the relational (SQLite‑based) container adopted by the **SoilWise** project (Horizon Europe – Mission Soil) to enable **exchange, storage, and GIS‑native use** of soil data, with the explicit goal of making them **FAIR** and reusable across European policies, research, and land‑management workflows.[^1][^2] SoilWise provides an **integrated, actionable entry point** to scattered soil data and knowledge, recognising existing workflows and repositories and connecting to them through a **modular, scalable** architecture designed to last beyond the project horizon and co‑created with stakeholders.[^1][^2]

## Why GeoPackage/SQLite

**GeoPackage** is an **OGC** open, portable, self‑contained standard for geospatial data. Being an SQLite container, it allows **direct** use of vector features, rasters/tiles and attribute data in a **single file**, without intermediate format translations. This makes it ideal for GIS environments and for constrained connectivity scenarios.[^3][^4]  
Within the INSPIRE ecosystem, an **endorsed Good Practice** describes how to design and publish **INSPIRE datasets encoded as GeoPackage**, while preserving legal and technical compliance with the Implementing Rules (with technical conformity demonstrable through transformation to the default encoding, **GML**). The Good Practice promotes logical models **optimised for GIS usability**, with explicit mapping rules from the INSPIRE conceptual model.[^5][^6]

## Continuity with INSPIRE: from EJP SOIL to **Soilwise**

The **Soilwise** database builds upon—and updates—the work carried out around INSPIRE, including the **EJP SOIL** GeoPackage template for the **Soil (SO)** theme. That template focused on **semantic harmonisation**, **code‑list management**, and **repeatable transformations**, and is a relevant baseline for Soilwise’s relational modelling approach.[^7] This direction aligns with community guidance on publishing INSPIRE data as a **relational database** (GeoPackage as a specialisation of SQLite), including recipes and patterns for harmonisation and publication.[^8]

## Data architecture and governance

In line with the **INSPIRE Good Practice**, each GeoPackage logical schema should be accompanied by:
- a **model description** (including benefits and limitations compared to the default encoding);
- an **empty GeoPackage template** for distribution;
- an **executable specification** for model transformation (**UML→GPKG** or **GML→GPKG**), aligned with the generic INSPIRE transformation rules.[^5]

This approach ensures **traceable compliance**, supports reuse across thematic communities, and allows **use‑case‑specific** logical models while preserving a coherent methodological and semantic framework.[^5]

## Integration with **OGC SensorThings API 2.0 (STA2)**

Within SoilWise, an implementation of **OGC SensorThings API 2.0 (STA2)** is used to expose **observations** and **metadata** as **interoperable time series** via HTTP and MQTT, consistent with **O&M / ISO 19156:2023**. The **STA2** draft (OGC document **23‑019**) updates the data model and bindings (e.g., OData 4.01 alignment), strengthening the API’s role as a standards‑based **“data‑in‑motion”** complement to the GeoPackage’s **“data‑at‑rest”** layer.[^9][^10]  
The growing adoption of SensorThings for observational data (well beyond narrow IoT use) is documented by OGC/WMO community materials, which highlight alignment with the renewed **OMS/ISO 19156:2023** standard and integration with GIS tools.[^11][^12]

## QGIS usage and custom forms

The **Soilwise** GeoPackage is **natively supported by QGIS**, enabling editing, styling, and map production with no format conversion. To improve data‑entry quality and speed, we recommend configuring **custom attribute forms** (widgets, value map/relation, constraints and expressions) and adopting a **design of tabs and groups** aligned with project code lists and validation processes.[^13][^14]


# Relational Structure of the GeoPackage (INSPIRE UML + STA2 Transposition)

![structure poster](assets/db_structure.webp)

> [!TIP]
> Click the image to see it full‑size for better viewing.

## Purpose and framing

This GeoPackage implements a **relational schema** that is a **faithful transposition** of the **INSPIRE Soil conceptual model (UML)** and its classes/associations, as described in the **INSPIRE Soil Technical Guidelines** and Feature Catalogue. It also integrates the **OGC SensorThings API 2.0 (STA2)** model for the management and exposure of **observations** (time‑series and observation metadata).[^5][^9][^10]

> [!NOTE]
> The Soil Technical Guidelines provide the authoritative description of the Soil theme, including the Feature Catalogue and the UML‑based relationships among elements (which underpin any encoding, such as GML or GeoPackage).[^5]



## Sources of the model (normative and technical)

- **INSPIRE Soil – Technical Guidelines**: define the content, the Feature Catalogue and the UML elements of the Soil theme, plus implementation recommendations.[^5][^15]  
- **INSPIRE Good Practice – GeoPackage encoding**: describes how to publish INSPIRE datasets using the OGC GeoPackage standard, while preserving legal and technical compliance; it promotes GIS‑oriented logical models and executable mappings from the conceptual model.[^5][^6][^15]  
- **OGC SensorThings API 2.0 (STA2)**: provides the data model and API for **Things, Datastreams, Observations, Sensors, ObservedProperties**, including HTTP/MQTT bindings and alignment with **O&M / ISO 19156:2023**.[^9][^10]



## From UML to a relational GeoPackage (SQLite)

The translation into GeoPackage follows widely adopted relational design rules, consistent with INSPIRE/OGC guidance:

1. **UML Classes → Tables**  
   Each INSPIRE *Feature Type* (e.g., *SoilSite*, *SoilPlot*, *SoilProfile*, *ProfileElement*) becomes a table; **attributes** map to typed columns; spatial attributes are stored in `geometry` columns as per GeoPackage.[^3][^4][^5][^15]  

2. **UML Associations → Foreign Keys**  
   UML cardinalities are enforced through **foreign keys** and (where required) **link tables** for *1:N* / *N:M* relationships, preserving the Soil chain *Site → Plot → Profile → ProfileElement* defined at the conceptual level.[^5][^15]  

3. **Code lists → Reference tables**  
   Controlled vocabularies (INSPIRE and other allowed authorities) are materialized as **code‑list tables**, typically keeping **URI / notation / label / authority / version** to ensure **semantic interoperability**.[^5][^15]  

4. **Observational component (STA2)**  
   STA2 entities **Thing / Datastream / Observation / Sensor / ObservedProperty** are mapped to dedicated tables. **Observation** rows reference their **Datastream**, keep temporal attributes (`phenomenonTime`, `resultTime`), and a **result** (numeric or textual) according to STA2. This design makes the observational layer complementary to the core INSPIRE Soil features, and ready for standards‑based exposure via the STA2 API.[^9][^10]

> [!NOTE]
> GeoPackage is an OGC open, portable and self‑contained standard (an SQLite container) that supports **direct use** of vector/raster/attribute data in a **single file**, without intermediate format translations—ideal for GIS tools and constrained‑connectivity scenarios.[^3][^4]  
> The INSPIRE Good Practice recognises GeoPackage as **additional/alternative encoding** to the default, with conformity demonstrable via transformation to GML.[^5]



## Key INSPIRE Soil features (concise overview)

> [!NOTE]
> The short summaries below are aligned with the Soil Feature Catalogue; full definitions, attributes, multiplicities and constraints are given in the official Technical Guidelines.[^5]

- **SoilSite** — *Context/area of investigation*: an area (often polygonal) providing the context within which one or more specific investigations are carried out; it acts as a logical container to **combine or compare** investigations performed at nearby points/times.[^5]

- **SoilPlot** — *Investigation point/portion*: a point or area where a specific investigation is performed (e.g., pit, borehole, sample location); it is **contained** in a *SoilSite* and is the operational focus of many descriptions and measurements.[^5]

- **SoilProfile** — *Vertical cross‑section*: the section from ground surface down to material not modified by pedogenesis; it is the basis for describing **horizons** or **layers** (modelled as *ProfileElement*).[^5]

- **ProfileElement** — *Horizon or layer*: either a **SoilHorizon** (a pedogenetic horizon homogeneous by physical/chemical/biological properties) or a **SoilLayer** (a depth‑defined layer, not necessarily equal to a horizon).[^5]

- **SoilBody** — *Mapping unit*: an association of co‑occurring soils in an area (conceptually similar to a **Soil Mapping Unit**), typically linked to one or more representative **SoilProfile(s)**.[^5]



[^1]: **SoilWise – project website** (Horizon Europe – Mission Soil).  
https://soilwise-he.eu/

[^2]: **Mission Soil Platform (EC) – SoilWise project page**.  
https://mission-soil-platform.ec.europa.eu/project-hub/soilwise

[^3]: **GeoPackage.org** – Overview and resources.  
https://www.geopackage.org/

[^4]: **OGC – GeoPackage Standard** (publications page).  
https://www.ogc.org/standards/geopackage/

[^5]: **INSPIRE Knowledge Base** – *GeoPackage encoding of INSPIRE datasets* (Good Practice).  
https://knowledge-base.inspire.ec.europa.eu/geopackage-encoding-inspire-datasets_en

[^6]: **INSPIRE‑MIF – gp‑geopackage‑encodings** (Good Practice repository).  
https://github.com/INSPIRE-MIF/gp-geopackage-encodings

[^7]: **EJP SOIL – INSPIRE Soil GeoPackage template** (GitHub).  
https://github.com/ejpsoil/inspire_soil_gpkg_template

[^8]: **Soil data guidance** – *INSPIRE Soil in a relational database* (recipe & workflow).  
https://ejpsoil.github.io/soildata-assimilation-guidance/cookbook/glosis-db.html

[^9]: **OGC – SensorThings API 2.0 (draft 23‑019)**.  
https://hylkevds.github.io/23-019/23-019.html

[^10]: **OGC Publications – SensorThings API** (Part 1/1.1 Sensing, Part 2 Tasking Core, STAplus).  
https://www.ogc.org/standards/sensorthings/

[^11]: **WMO/OGC** – *Integrating new OGC Standards* (slides).  
https://wmo.int/sites/default/files/2024-02/Integration%20of%20new%20OGC%20Standards%20e.g%20SensorThings%20API.pdf

[^12]: **BRGM HAL** – *SensorThings API and WaterML2.0* (slides).  
https://brgm.hal.science/hal-04456540v2/file/SensorThings%20API%20and%20%20WaterML2.0%20.pdf

[^13]: **QGIS Training Manual** – *Forms / Attribute Forms*.  
https://docs.qgis.org/latest/en/docs/training_manual/create_vector_data/forms.html

[^14]: **QField Documentation** – *Simple attribute form configuration*.  
https://docs.qfield.org/how-to/project-setup/attributes-form/

[^15]: **INSPIRE UML models — Approved (HTML view).**  
https://inspire-mif.github.io/uml-models/approved/

## Data Loading & Modelling Guide
[Data Loading & Modelling Guide](./data_loading.md) document  is the practical guide for **populating (loading)** the GeoPackage.  
It explains, clearly and in sequence:
- the **order** in which datasets must be loaded and their **dependencies**;
- the **constraints** to respect (e.g., required fields, minimum cross-table consistency);
- what to **verify before/after loading** to ensure a correct result.

>[!IMPORTANT]
> In short: if you need to **populate or update** the GeoPackage, **start here**.

## QGIS Manual

[A practical guide](./qgis/README.md) to using QGIS to **read from and write to** the database (GeoPackage), supported by **custom Forms** that simplify data entry and updates.

### Custom Forms
The Forms are designed to guide users in filling fields correctly:
- Guided fields with **drop-down menus**, **default values**, and **contextual help**.
- **Validation checks** to flag incomplete forms or inconsistent values.
- **Sections** that group the most used fields and hide technical fields.
- **Actions** (where available) for automatic completion or quick checks.

> [!WARNING]
> QGIS version **3.44.0 – Solothurn** or higher is required.


## Database Tables

This chapter provides the **detailed description of all database tables**.  
For each table, you will find its **purpose**, **key fields**, any **geometry**, the most relevant **relationships**, and **usage notes**.

- [codelist](./tables/codelist.md)
- [datastream](./tables/datastream.md)
- [derivedprofilepresenceinsoilbody](./tables/derivedprofilepresenceinsoilbody.md)
- [faohorizonnotationtype](./tables/faohorizonnotationtype.md)
- [isbasedonobservedsoilprofile](./tables/isbasedonobservedsoilprofile.md)
- [isbasedonsoilbody](./tables/isbasedonsoilbody.md)
- [isbasedonsoilderivedobject](./tables/isbasedonsoilderivedobject.md)
- [isderivedfrom](./tables/isderivedfrom.md)
- [observation](./tables/observation.md)
- [observedproperty](./tables/observedproperty.md)
- [observingprocedure](./tables/observingprocedure.md)
- [obsprocedure_obsdproperty](./tables/obsprocedure_obsdproperty.md)
- [obsprocedure_sensor](./tables/obsprocedure_sensor.md)
- [otherhorizon_profileelement](./tables/otherhorizon_profileelement.md)
- [otherhorizonnotationtype](./tables/otherhorizonnotationtype.md)
- [othersoilnametype](./tables/othersoilnametype.md)
- [profileelement](./tables/profileelement.md)
- [sensor](./tables/sensor.md)
- [soilbody](./tables/soilbody.md)
- [soilbody_geom](./tables/soilbody_geom.md)
- [soilderivedobject](./tables/soilderivedobject.md)
- [soilplot](./tables/soilplot.md)
- [soilprofile](./tables/soilprofile.md)
- [soilsite](./tables/soilsite.md)
- [thing](./tables/thing.md)
- [unitofmeasure](./tables/unitofmeasure.md)
- [wrbqualifiergroup_profile](./tables/wrbqualifiergroup_profile.md)
- [wrbqualifiergrouptype](./tables/wrbqualifiergrouptype.md)




## Relationship Summary 
[This document](./relationships_summary.md) helps you understand **how tables are connected**.  
It highlights the most relevant links and suggests when tables should be used together—useful to navigate between **core entities** and **supporting tables**.


## Cascade Summary 
[This document](./cascade_summary.md) explains how the system manages the **consequences** of updates and deletions across related tables.  
It clarifies what happens to **related records** and how to keep data **consistent** when modifying “parent” or “child” data.


