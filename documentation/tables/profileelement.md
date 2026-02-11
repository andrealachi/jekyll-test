# Profile Element 

## Definition

> "*An abstract spatial object type grouping soil layers and / or horizons for functional/
operational aims.*"[^1]


[^1]: European Commission – Joint Research Centre (JRC),  
**INSPIRE Data Specification on Soil – Technical Guidelines**,  
D2.8.III.3.  
https://inspire-mif.github.io/technical-guidelines/data/so/dataspecification_so.pdf
<p>&nbsp;</p>


## Horizon vs Layer (*profileelement*)

In the INSPIRE *Soil* model, *Profile Element* is an abstract type that groups the vertical slices composing a soil profile; it specialises into **Soil Horizon** and **Soil Layer**. A **Soil Horizon** is a pedogenetically formed, relatively homogeneous layer (roughly parallel to the surface) identified by morphological/analytical characteristics. A **Soil Layer** is an arbitrary (often depth‑based) slice or a grouping of horizons, not necessarily reflecting pedogenic boundaries.[^2][^3][^4][^1]


## Field that determines the element type

The boolean field **`profileelement.profileelementtype`** determines the specialisation:

- `0` → **Horizon**  
- `1` → **Layer**

This behaviour is enforced by database triggers. See the DDL for details.[^6]

## Behavior of the `profileelement` table per type

> **Depth range & validity (applies to both types).** At least one of `profileelementdepthrange_uppervalue` or `profileelementdepthrange_lowervalue` must be non‑null; if both are provided, `uppervalue` must be **less than** `lowervalue` (i.e., depth increases downward).[^6]

### Horizon (`profileelementtype = 0`)
- **Layer‑specific fields must be NULL**: the attributes intended for *Layer* (e.g., `layertype`, `layerrocktype`, `layergenesisprocess`, `layergenesisenviroment`, `layergenesisprocessstate`) are **forbidden** on Horizons; triggers reject inserts/updates that set them.[^6]  
- **Horizon notation is enabled**: only Horizons can be linked to horizon‑notation records (e.g., **FAO** horizon notation via `faohorizonnotationtype`, and other horizon notation via `otherhorizon_profileelement`); triggers enforce that the associated `profileelement` is a Horizon.[^6]  
- **Belongs to a SoilProfile**: every `profileelement` (Horizon or Layer) references its parent profile via **`ispartof` → `soilprofile.guid`**.[^1][^6]

### Layer (`profileelementtype = 1`)
- **Layer attributes allowed + domain rules**:  
  - `layertype` must come from the **LayerTypeValue** codelist;  
  - `layerrocktype` from **LithologyValue**;  
  - `layergenesisprocess` from **EventProcessValue**;  
  - `layergenesisenviroment` from **EventEnvironmentValue**;  
  - `layergenesisprocessstate` from **LayerGenesisProcessStateValue**.  
  Triggers validate all assignments against the corresponding codelists.[^6]  
- **Geogenic guardrails**: when `layertype` is **not** “geogenic”, the genesis‑related fields (`layerrocktype`, `layergenesis*`) **must be NULL**; dedicated triggers enforce this consistency.[^6]  
- **Horizon notation not applicable**: links to FAO/other horizon notation are **blocked** for Layers by triggers that require `profileelementtype = 0` on the target element.[^6]  
- **Belongs to a SoilProfile**: like Horizons, Layers are related to their parent profile through **`ispartof`**.[^1][^6]

>[!WARNING]
> Records that do not comply with the defined constraints **shall be rejected by the system and shall not be persisted** in the GeoPackage.



[^2]: **Soil Horizon — INSPIRE Feature Concept Dictionary**.  
https://inspire.ec.europa.eu/featureconcept/SoilHorizon

[^3]: **Soil Layer — INSPIRE Feature Concept Dictionary**.  
https://inspire.ec.europa.eu/featureconcept/SoilLayer

[^4]: **Profile Element — INSPIRE Feature Concept Dictionary**.  
https://inspire.ec.europa.eu/featureconcept/ProfileElement:1

[^5]: **INSPIRE Soil — Overview slide deck (Site–Plot–Profile–Horizon/Layer)**.  
https://zenodo.org/records/13970777/files/II3-INSPIRE-Soil.pdf

[^6]: **Database schema (DDL) — tables, constraints & triggers for `profileelement`**.  
./DDL_SO_14.sql


<p>
  <img src="../assets/profileelement.svg"
     alt="TABELLA SOILSITE EXP"
     align="left"
     width="420">
 <!--
  <strong>TABELLA SOILSITE EXP</strong><br>
   Lorem Ipsum.
 -->
</p>

<br clear="all">
<p>&nbsp;</p>


## Table: `profileelement`

### Columns

| Name | Type | Constraints | Description |
|------|------|-------------|-------------|
| `id` | `INTEGER` | PRIMARY KEY | Primary Key of the Table. |
| `guid` | `TEXT` |  | Universally unique identifier. |
| `inspireid_localid` | `TEXT` |  | A local identifier, assigned by the data provider. The local identifier is unique within the namespace, that is no other spatial object carries the same unique identifier. |
| `inspireid_namespace` | `TEXT` |  | Namespace uniquely identifying the data source of the spatial object. |
| `inspireid_versionid` | `TEXT` |  | The identifier of the particular version of the spatial object, with a maximum length of 25 characters. If the specification of a spatial object type with an external object identifier includes life-cycle information, the version identifier is used to distinguish between the different versions of a spatial object. Within the set of all versions of a spatial object, the version identifier is unique. |
| `profileelementdepthrange_uppervalue` | `INTEGER` |  | Upper depth of the profile element (layer or horizon) measured from the surface of a soil profile (in cm). |
| `profileelementdepthrange_lowervalue` | `INTEGER` |  | Lower depth of the profile element (layer or horizon) measured from the surface of a soil profile (in cm). |
| `beginlifespanversion` | `DATETIME` | NOT NULL, DEFAULT strftime('%Y-%m-%dT%H:%M:%fZ', 'now') | Date and time at which this version of the spatial object was inserted or changed in the spatial data set. |
| `endlifespanversion` | `DATETIME` |  | Date and time at which this version of the spatial object was superseded or retired in the spatial data set. |
| `layertype` | `TEXT` |  | Assignation of a layer according to the concept that fits its kind. |
| `layerrocktype` | `TEXT` |  | Type of the material in which the layer developed. |
| `layergenesisprocess` | `TEXT` |  | Last non-pedogenic process (geologic or anthropogenic) that coined the material composition and internal structure of the layer. |
| `layergenesisenviroment` | `TEXT` |  | Setting in which the last non-pedogenic process (geologic or anthropogenic) that coined the material composition and internal structure of the layer took place. |
| `layergenesisprocessstate` | `TEXT` |  | Indication whether the process specified in layerGenesisProcess is on-going or seized in the past. |
| `profileelementtype` | `BOOLEAN` | NOT NULL, DEFAULT 0 | Boolean value to indicate whether the record is of Horizon or Layer type. |
| `ispartof` | `TEXT` | NOT NULL | Foreign key to the SoilProfile table, guid field. |

### Table Identifiers
In this table, the primary key is the *id* field (integer, auto-incrementing).  
There is also a text field named **GUID**, which stores a *UUID* (Universally Unique Identifier) compliant with RFC 4122.

Although GUID is not mandatory at the schema level (it is not declared NOT NULL), its functional requirement is enforced by two triggers:
- **profileelementguid (INSERT)** trigger: if GUID is missing or empty, a correctly formatted UUID is generated and inserted automatically.
- **profileelementguidupdate (UPDATE)** trigger: prevents any modification of GUID after insertion, making it immutable (effectively behaving as a stable key).  

Any foreign keys (FK) from other tables reference this table’s GUID field rather than the id field, ensuring stable and interoperable references across datasets and database instances.
> [!NOTE]
> **GUID management** is handled by database triggers, which ensure their automatic generation at the time of record insertion, **without any user involvement**.

### Coded Fields

The **`layertype`**,**`layerrocktype`**, **`layergenesisprocess`**, **`layergenesisenviroment`** and **`layergenesisprocessstate`** fields are **coded fields**(*codelist-based attribute*), meaning that they can only contain values belonging to a predefined **codelists**, in accordance with the INSPIRE specifications.


> [!WARNING]
> Any attempt to insert a value that is not included in the corresponding codelist is considered **invalid** by the system and will result in the **failure of the data insertion operation**.



#### Codelist Definition

The complete list of allowed codes is stored in the **codelist table**.  
The associated [documentation](codelist.md), provides a detailed description of:

- which codes are available (INSPIRE codelist URL),
- the database tables to which each codelist applies,
- the fields for which each code is valid,

in accordance with the adopted conceptual model.

#### Validation and Data Entry

The semantic and syntactic validation of the inserted values is enforced at the database level through dedicated **control triggers** (i_layertype/u_layertype/i_layerrocktype/u_layerrocktype/i_layergenesisprocess/u_layergenesisprocess/i_layergenesisenviroment/u_layergenesisenviroment/i_layergenesisprocessstate/ u_layergenesisprocessstate), ensuring compliance with the defined codelists.

> [!IMPORTANT]
>During data entry via the **QGIS interface**, users are supported by **dropdown menus** that display only the valid codes for the selected field.

> [!NOTE]
>This mechanism **reduces the risk of data entry errors** and guarantees alignment with the constraints imposed by the INSPIRE codelists.

### Relationships (as child)
- `profileelement.ispartof` → `soilprofile.guid` (**ON UPDATE** CASCADE, **ON DELETE** CASCADE)
  - *Note:* delete on `soilprofile` cascades to `profileelement`.

### Referenced by (as parent)
- `datastream.guid_profileelement` → `profileelement.guid` (**ON UPDATE** CASCADE, **ON DELETE** CASCADE)
- `faohorizonnotationtype.guid_profileelement` → `profileelement.guid` (**ON UPDATE** CASCADE, **ON DELETE** CASCADE)
- `otherhorizon_profileelement.guid_profileelement` → `profileelement.guid` (**ON UPDATE** CASCADE, **ON DELETE** CASCADE)

### Indexes

| Name | Unique | Columns | Origin | Partial |
|------|--------|---------|--------|---------|
| `sqlite_autoindex_profileelement_1` | Yes | `guid` | `u` | No |

### Triggers
For every trigger you will find:

- **When it runs** (BEFORE/AFTER, INSERT/UPDATE/DELETE)
- **What it reads and compares** (columns, lookups in other tables)
- **What happens on success** (the statement proceeds, optional updates)
- **What happens on failure** (the exact error text raised)

 


#### `profileelementguid` / `profileelementguidupdate`
**When they run:** AFTER INSERT / AFTER UPDATE OF `guid`

**What they do:** Generate a GUID when missing; prevent changing it later.

**If the check passes:** Insert writes GUID; unchanged updates proceed.

**If the check fails:** On change, abort with: `Cannot update guid column.`

#### `i_ceckvalidversionprofileelement`
**When it runs:** BEFORE INSERT

**What it does:** Require `beginlifespanversion < endlifespanversion`.

**If the check passes:** Insert proceeds.

**If the check fails:** Aborts with: `Table profileelement: beginlifespanversion must be less than endlifespanversion`.

#### `i_ceckvaliddeepprofileelement` / `u_ceckvaliddeepprofileelement`
**When they run:** BEFORE INSERT / BEFORE UPDATE

**What they do:** Enforce depth-range ordering: `uppervalue` must be **less** than `lowervalue`.

**If the check passes:** Statement proceeds.

**If the check fails:** Aborts with: `Table profileelement: profileelementdepthrange_uppervalue must be less than profileelementdepthrange_lowervalue`.

#### `i_checkgeogenicfieldsnull` / `u_checkgeogenicfieldsnotnull`
**When they run:** BEFORE INSERT / BEFORE UPDATE

**What they do:** When `layertype` is **not** geogenic (`http://inspire.ec.europa.eu/codelist/LayerTypeValue/geogenic`), the fields `layerrocktype`, `layergenesisprocess`, `layergenesisenviroment`, `layergenesisprocessstate` must all be **NULL**.

**If the check passes:** Statement proceeds.

**If the check fails:** Aborts with a specific message pointing to the offending field (e.g., `layerrocktype must be NULL when LayerTypeValue is not "geogenic".`).

#### `i_ceckhorizonfields` / `u_ceckhorizonfields`
**When they run:** BEFORE INSERT / BEFORE UPDATE

**What they do:** For horizon-type elements (`profileelementtype = 0`), all layer-specific attributes must be **NULL**.

**If the check passes:** Statement proceeds.

**If the check fails:** Aborts with messages such as `layertype must be NULL when profilelement is "HORIZON".`

#### `i_layertype` / `u_layertype`
**When they run:** BEFORE INSERT / BEFORE UPDATE

**What they do:** Validate `layertype` membership in the `LayerTypeValue` codelist.

**If the check passes:** Statement proceeds.

**If the check fails:** Aborts with: `Table profileelement: Invalid value for layertype. Must be present in id of layertypevalue codelist.`

#### `i_layergenesisenviroment` / `u_layergenesisenviroment`
**When they run:** BEFORE INSERT / BEFORE UPDATE

**What they do:** If provided, `layergenesisenviroment` must exist in `EventEnvironmentValue`.

**If the check passes:** Statement proceeds.

**If the check fails:** Aborts with: `Table profileelement: Invalid value for layergenesisenviroment. Must be present in id of eventenvironmentvalue codelist.`

#### `i_layergenesisprocess` / `u_layergenesisprocess`
**When they run:** BEFORE INSERT / BEFORE UPDATE

**What they do:** If provided, `layergenesisprocess` must exist in `EventProcessValue`.

**If the check passes:** Statement proceeds.

**If the check fails:** Aborts with: `Table profileelement: Invalid value for layergenesisprocess. Must be present in id of eventprocessvalue codelist.`

#### `i_layergenesisprocessstate` / `u_layergenesisprocessstate`
**When they run:** BEFORE INSERT / BEFORE UPDATE

**What they do:** If provided, `layergenesisprocessstate` must exist in `LayerGenesisProcessStateValue`.

**If the check passes:** Statement proceeds.

**If the check fails:** Aborts with: `Table profileelement: Invalid value for layergenesisprocessstate. Must be present in id of layergenesisprocessstatevalue codelist.`

#### `i_layerrocktype` / `u_layerrocktype`
**When they run:** BEFORE INSERT / BEFORE UPDATE

**What they do:** If provided, `layerrocktype` must exist in `LithologyValue`.

**If the check passes:** Statement proceeds.

**If the check fails:** Aborts with: `Table profileelement: Invalid value for layerrocktype. Must be present in id of lithologyvalue codelist .`

#### `i_check_depth_range` / `u_check_depth_range`
**When they run:** BEFORE INSERT / BEFORE UPDATE

**What they do:** Require that at least one of `profileelementdepthrange_uppervalue` or `profileelementdepthrange_lowervalue` is **not NULL** (i.e., some depth info is present).

**If the check passes:** Statement proceeds.

**If the check fails:** Aborts with: `At least one of profileelementdepthrange_uppervalue and profileelementdepthrange_lowervalue must not be null`.

#### `u_begin_today_profileelement` / `u_begin_today_profileelement_error`
**When they run:** AFTER UPDATE

**What they do:** Refresh `beginlifespanversion` when still active; block edits if the end is **past**.

**If the check passes:** Timestamp refreshed / update continues.

**If the check fails:** Aborts with: `If you change record endlifespanversion must be greater than today`.

