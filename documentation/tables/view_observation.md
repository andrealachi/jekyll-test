# View Observation

## Why a view

The view was created to present a **wide** format to end users (i.e., multiple thematic columns on a single row), which is easier to read than the **long** format (also known as *tidy/long format*) commonly found in normalized models where observed values are spread across multiple rows. This improves human-friendly browsing, export, and reporting for non-technical audiences.

## What the query does (short version)

The `view_observation` definition starts from the `observation` table and **enriches** each record with metadata and key relationships, producing a single “wide” row per observation. In particular: [creagov-my...epoint.com]

- **Parent keys**: resolves the local identifiers of **Soil Site**, **Soil Profile**, and **Profile Element** using a chain of `LEFT JOIN` and `COALESCE`, climbing—when needed—through `soilplot` up to `soilsite`:  
  `ssLocalid` (soilsite), `spLocalid` (soilprofile), `peLocalid` (profileelement). [creagov-my...epoint.com] [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)

- **Profile type (Observed/Derived)**: sets the `isderived` label based on the FOI linked by `datastream` (profile element or soil profile) and the `isderived` flag on the profile. [creagov-my...epoint.com] [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)

- **FOI type (feature of interest)**: computes `FOIType` among *Profile Element*, *Soil Profile*, *Soil Site*, *Soil Derived Object*, or *None* depending on which `guid_*` is populated in `datastream`. [creagov-my...epoint.com] [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)

- **Depth**: exposes `upperLimit` and `lowerLimit` (only when the FOI is a `profileelement`). [creagov-my...epoint.com] [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)

- **Observation info**: [creagov-my...epoint.com]  
  - `time` = `observation.phenomenontime_start`. [creagov-my...epoint.com] [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
  - `property` taken from `observedproperty.name`. [creagov-my...epoint.com] [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
  - `uom` (unit of measure) from `unitofmeasure.symbol` when `datastream.type = 'Quantity'`. [creagov-my...epoint.com] [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
  - `procedure` from `observingprocedure.name` when present. [creagov-my...epoint.com] [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)

- **Result values**: [creagov-my...epoint.com]  
  - `category_value` (text) and `boolean_value` (boolean), when applicable. [creagov-my...epoint.com] [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)  
  - `quantity_value` populated only for `Quantity` series; `count_value` only for `Count` series (both derived from `observation.result_real`). [creagov-my...epoint.com] [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)

**In summary**, the view **aligns and flattens** registry attributes (site/profile/element), context (FOI and lineage), measurement metadata (property, unit, procedure), and the **observation value** on a single row—so users don’t need to perform manual joins or reshaping. [creagov-my...epoint.com] [1](https://creagov-my.sharepoint.com/personal/andrea_lachi_crea_gov_it/Documents/File%20chat%20di%20Microsoft%20Copilot/DDL_SO_20.sql)


### View

| Name             | Alias                          | Type       | Constraints  | Description |
|------------------|--------------------------------|------------|--------------|-------------|
| `ssLocalid`      | SoilSite LocalId               | `TEXT`     | nullable     | Local identifier of the Soil Site associated with the observation. |
| `spLocalid`      | SoilProfile LocalId            | `TEXT`     | nullable     | Local identifier of the Soil Profile associated with the observation. |
| `peLocalid`      | ProfileElement LocalId         | `TEXT`     | nullable     | Local identifier of the Profile Element where the observation was taken. |
| `isderived`      | —                              | `TEXT`     | nullable     | Human‑readable label (“Derived”/“Observed”) computed from the linked FOI and the profile’s `isderived` flag. |
| `FOIType`        | Feature-of-Interest Type       | `TEXT`     | **NOT NULL** | Indicates the type of Feature of Interest (Soil Site, Soil Profile, Profile Element, or Soil Derived Object). |
| `upperLimit`     | Upper Depth                    | `INTEGER`  | nullable     | Upper depth limit of the profile element (cm). |
| `lowerLimit`     | Lower Depth                    | `INTEGER`  | nullable     | Lower depth limit of the profile element (cm). |
| `time`           | Observation Time               | `DATETIME` | **NOT NULL** | Timestamp when the observed phenomenon occurred. |
| `property`       | Observed Property              | `TEXT`     | **NOT NULL** | Name of the observed property associated with the datastream. |
| `uom`            | —                              | `TEXT`     | nullable     | Unit symbol exposed by the view for `Quantity` series (taken from `unitofmeasure.symbol`). |
| `procedure`      | Procedure                      | `TEXT`     | nullable     | Name of the observing procedure used to obtain the observation. |
| `category_value` | Category Result                | `TEXT`     | nullable     | Categorical result of the observation (for Category‑type datastreams). |
| `boolean_value`  | Boolean Result                 | `BOOLEAN`  | nullable     | Boolean result of the observation (for Boolean‑type datastreams). |
| `quantity_value` | Quantity Result                | `REAL`     | nullable     | Numeric result of the observation (for Quantity‑type datastreams). |
| `count_value`    | Count Result                   | `REAL`     | nullable     | Integer‑valued result of the observation (for Count‑type datastreams). |
