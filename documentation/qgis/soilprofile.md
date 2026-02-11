
# Soil Profile Form

## Open
To open the Soil Profile custom forms, go to the Layers panel, right‑click the `soilprofile` layer, and choose **Open Attribute Table** from the context menu.

> [!TIP]
> For further information on the custom forms, consult the documents [Customized Attribute Forms in QGIS](./custom_form.md) and [Navigating GeoPackage Tables via Forms](./navigating_via_form.md)  

## Edit

A Soil Profile does not contain any geometry. To enable editing through forms, follow the instructions provided in the document [Editing Records Through a QGIS Form](./edit_form.md).

### REQUIRED fields
- `id`: primary key (auto-incrementing)
- `Begin Lifespan version`: DATETIME (default: today)
- `Valid From`: DATETIME (default: today)
- `isderived`: BOOLEAN (default: 0)
- `isoriginalclassification`: BOOLEAN (default: 1)

### ID Group
<p>
  <img src="../assets/group_id.webp"
       alt="Fig.1" align="left" width="500">
</p>
<br clear="all"><br>

#### Fields

- **`id`** - Primary **AUTO_INCREMENT INTEGER PRIMARY KEY**; it’s the required identifier for GeoPackage tables and is assigned automatically on insert.

- **`guid`** - **Global identifier** in UUID format, stored as text. The field is **optional**. It's **automatically managed via triggers**.

> [!IMPORTANT]
> On opening, the **ID** group is collapsed: there is no need for manual editing, as **both fields are system‑managed** (the `id` by the SQLite engine and the `guid` by triggers), reducing errors and ensuring identifier consistency over time.

### Dynamic Form Behaviour Based on `Is Derived`

The **Soil Profile** data-entry form must adapt dynamically according to the value of the field `soilprofile.isderived`, which determines whether the profile is **Observed** (`0`) or **Derived** (`1`). This behaviour follows the INSPIRE Soil data model and is strictly enforced in the GeoPackage through database triggers and CHECK constraints.

#### `Is Derived = 0` → **Observed Soil Profile**

<p>
  <img src="../assets/observed.webp"
       alt="Fig.1" align="left" width="500">
When users select <strong>Observed</strong> -  <strong>The checkbox is unchecked</strong> ①

The form automatically shows the elements related to point‑location and to the contextual relations available for observed profiles.
The <strong>Soil Plot section</strong> becomes visible, ② allowing the selection of the associated soilplot.guid. 

The <strong>Derived Soil Profile section</strong> (representing the isderivedfrom relation where the observed profile may appear as guid_related) also appears,③ along with the <strong>SoilDerived Object</strong> relation area ④.
Attempts to store a NULL or invalid location are blocked by database triggers, ensuring data consistency. 

Interface elements specific to derived profiles—such as derived‑only constraints or soil‑body‑percentage definitions—do not appear while the profile is in Observed mode.
Database logic prevents these derived‑only relations from being created for an observed profile.
</p>
<br clear="all"><br>

#### `Is Derived = 1` → **Derived Soil Profile**

<p>
  <img src="../assets/derived.webp"
       alt="Fig.1" align="left" width="500">
When users select <strong>Derived</strong> - <strong>The checkbox is checked</strong> ①

The form adjusts by hiding all location‑related components, including the Soil Plot section, since derived profiles are not point‑located. Any non‑NULL value in the location field is rejected by database triggers.

In this mode, the form exposes only the relational structures applicable to derived profiles:

the <strong>Is Derived From section</strong>, ② which represents the association to observed profiles through the isderivedfrom table (where the derived profile appears as guid_base)
the Derived Presence in <strong>Soil Body section</strong>,③ corresponding to the derivedprofilepresenceinsoilbody table, used to express the percentage‑based presence of the derived profile within a Soil Body

These interface elements reflect the constraints, type‑coherence checks, and consistency rules enforced at database level.
</p>
<br clear="all"><br>


### INSPIRE ID Group
<p>
  <img src="../assets/group_inspireid.webp"
       alt="Fig.1" align="left" width="500">
</p>
<br clear="all"><br>

>An **INSPIRE ID** is the **external unique identifier** assigned to each spatial object in INSPIRE datasets; it ensures **uniqueness** and **persistence** and allows external applications to reliably reference the same object over time.
The identifier **must not be changed** during the object’s life cycle; it can also be published as a **URI** to facilitate web-based referencing. [^1]

[^1]: Creating INSPIRE external unique object identifiers in the scope of the END reporting.
https://epanet.eea.europa.eu/Eionet/reportnet/docs/noise/guidelines/inspire_identifiers_doc.pdf 

#### Fields

- **`Local id`** — Local identifier assigned by the data provider; **unique** within its namespace.
- **`Namespace`** — Namespace that uniquely identifies the data source/domain of the spatial object. 
- **`Version id`** — Identifier of the specific **version** of the object; **optional (voidable)** and used to distinguish different versions of the same object. 

> [!IMPORTANT]
> These fields are not mandatory, but **filling them out is strongly recommended**: they help uniquely identify the record in forms and across data exchanges.  
> In particular, `localid` + `namespace` form a stable identifier; `versionid` helps track changes over time.

### Editing Child Elements in QGIS Forms
Editing child elements directly within a parent form improves data quality and speed: it keeps users in context, guarantees referential integrity through predefined relations, and reduces errors by enforcing database rules at the moment of entry. In your GeoPackage, several relationships are validated by triggers and codelist checks, so capturing child data where it belongs (inside the parent form) closely aligns UI behavior with DB constraints.

### Sub Form 
<p>
  <img src="../assets/spr_sub_wrb.webp"
       alt="Fig.1" align="left" width="500">
</p>
<br clear="all"><br>

Use the following buttons to manage child layers during data editing.
  
**Toggle editing mode for child layer** ① enables editing on the related (child) layer embedded in the form; once active, you can add/modify/delete child records directly from the parent record’s view.

**Save child layer edit** ② commits the pending edits for the child layer to the GeoPackage. Use this to persist changes without leaving the parent form.

**Add child feature** ③ creates a new child record pre‑linked to the current parent (relation fields are auto‑populated by the form’s relation widget), ensuring correct foreign keys and preventing orphan rows.

### Sub Form WRB Qualifier
**Purpose**: attaches WRB qualifier groups to a soil profile with an explicit qualifierposition (ordering).

**DB enforcement**: triggers align wrbversion between profile and qualifier group, and ensure qualifierposition is unique per qualifierplace within the same profile.

**Form hint**: expose as a child list on the Soil Profile form; use Add child feature to add qualifiers in order, and Save child layer edit to commit.

### Sub Form Other Soil Name
**Purpose**: stores additional soil names and related classification flags for a profile.

**DB enforcement**: codelist checks on othersoilname_type; GUID immutability; cascading rules via FK to soilprofile.

**Form hint**: present as a child panel in the Soil Profile form for quick insertion of multiple alternative names; Save child layer edit to persist

### Sub Form Derived Soil Profile (*Visible if `isderived` = TRUE*) - Is Derived From (*Visible if `isderived` = FALSE*)
**Purpose**: records the association Derived Soil Profile (guid_base) → Observed Soil Profile (guid_related).

**DB enforcement**: triggers guarantee that guid_base points to a derived profile and guid_related to an observed one; duplicates of the same pair are prevented.

**Form hint**: embed this as a child table under Derived Soil Profile; use Add child feature to append observed sources and Save child layer edit to persist.

### Constraints
- **CHECK**: `validfrom <= validto` (BEFORE INSERT/UPDATE).
- **CHECK**: `beginlifespanversion < endlifespanversion` (BEFORE INSERT).
- **Observed/Derived location rule**: if `isderived=1` then `location` must be NULL; if `isderived=0` then `location` must be NOT NULL (BEFORE INSERT/UPDATE).
- **WRB version membership**: `wrbversion ∈ codelist(id)` where `collection='wrbversion'` when not NULL (BEFORE INSERT/UPDATE).
- **WRB RSG + version coherence**: if `wrbversion` targets a specific year, `wrbreferencesoilgroup` must belong to the corresponding year collection (BEFORE INSERT/UPDATE).
- **GUID immutability** and **versioning refresh** on UPDATE.

### Attribute Reference
For an  overview of the **attributes used in the custom form**, refer to the soilsite table  [documentation](../tables/soilplot.md). It provides the key definitions and data types needed to correctly interpret the fields and configure the form within the data model.

## Datastream Relationship
Since this table is relationally linked to a SensorThings **Datastream**, you can add further domain‑specific properties in an STA2‑compatible way by correctly populating the [Datastream](./datastream.md) and [Observation](./observation.md) tables. 

In practice, Datastream holds the series‑level context of the time series, while each Observation record carries the measurement‑level details, reflecting the SensorThings data model in which every observation is organized under a datastream.

## Save

For a more comprehensive overview of form‑saving workflows, refer to the detailed documentation in [Saving Edits in QGIS Forms](./save_form.md).
