# Soil Site


## Definition

> “*A **Soil Site** is considered as the surrounding of a soil profile, and/or the larger piece of land that is directly linked to and described by all soil investigations on one or more spots, called soil plots.*”[^1]
> [^1]: European Commission – Joint Research Centre (JRC),  
**INSPIRE Data Specification on Soil – Technical Guidelines**,  
D2.8.III.3.  
https://inspire-mif.github.io/technical-guidelines/data/so/dataspecification_so.pdf

<p>&nbsp;</p>

<p>
  <img src="../assets/soilsite.svg"
     alt="TABELLA SOILSITE EXP"
     align="left"
     width="420">
 <!--
  <strong>TABELLA SOILSITE EXP</strong><br>
  Testo a destra dell’immagine. Lorem Ipsum è un testo segnaposto utilizzato nel settore della tipografia e della stampa. Lorem Ipsum è considerato il testo segnaposto standard sin dal sedicesimo secolo, quando un anonimo tipografo prese una cassetta di caratteri e li assemblò per preparare un testo campione. È sopravvissuto non solo a più di cinque secoli, ma anche al passaggio alla videoimpaginazione, pervenendoci sostanzialmente inalterato. Fu reso popolare, negli anni ’60, con la diffusione dei fogli di caratteri trasferibili “Letraset”, che contenevano passaggi del Lorem Ipsum, e più recentemente da software di impaginazione come Aldus PageMaker, che includeva versioni del Lorem Ipsum.
 -->
</p>

<br clear="all">
<p>&nbsp;</p>

## Table: `soilsite`

### Columns

| Name | Type | Constraints | Description |
|------|------|-------------|-------------|
| `id` | `INTEGER` | PRIMARY KEY | Primary Key of the Table. |
| `guid` | `TEXT` |  | Universally unique identifier |
| `geometry` | `POLYGON` | NOT NULL | Geometry. |
| `inspireid_localid` | `TEXT` |  | A local identifier, assigned by the data provider. The local identifier is unique within the namespace, that is no other spatial object carries the same unique identifier. |
| `inspireid_namespace` | `TEXT` |  | Namespace uniquely identifying the data source of the spatial object. |
| `inspireid_versionid` | `TEXT` |  | The identifier of the particular version of the spatial object, with a maximum length of 25 characters. If the specification of a spatial object type with an external object identifier includes life-cycle information, the version identifier is used to distinguish between the different versions of a spatial object. Within the set of all versions of a spatial object, the version identifier is unique. |
| `soilinvestigationpurpose` | `TEXT` | NOT NULL | Indication why a survey was conducted. |
| `validfrom` | `DATETIME` | NOT NULL, DEFAULT strftime('%Y-%m-%dT%H:%M:%fZ', 'now') | The time when the phenomenon started to exist in the real world. |
| `validto` | `DATETIME` |  | The time from which the phenomenon no longer exists in the real world. |
| `beginlifespanversion` | `DATETIME` | NOT NULL, DEFAULT strftime('%Y-%m-%dT%H:%M:%fZ', 'now') | Date and time at which this version of the spatial object was inserted or changed in the spatial data set. |
| `endlifespanversion` | `DATETIME` |  | Date and time at which this version of the spatial object was superseded or retired in the spatial data set. |

### Table Identifiers
In this table, the primary key is the *id* field (integer, auto-incrementing).  
There is also a text field named **GUID**, which stores a *UUID* (Universally Unique Identifier) compliant with RFC 4122.

Although GUID is not mandatory at the schema level (it is not declared NOT NULL), its functional requirement is enforced by two triggers:
- **soilsiteguid (INSERT)** trigger: if GUID is missing or empty, a correctly formatted UUID is generated and inserted automatically.
- **soilsiteguidupdate (UPDATE)** trigger: prevents any modification of GUID after insertion, making it immutable (effectively behaving as a stable key).  

Any foreign keys (FK) from other tables reference this table’s GUID field rather than the id field, ensuring stable and interoperable references across datasets and database instances.

> [!NOTE]
> **GUID management** is handled by database triggers, which ensure their automatic generation at the time of record insertion, **without any user involvement**.

### Coded Fields

The **`soilinves­tigationpurpose`** field is a **coded field** (*codelist-based attribute*), meaning that it can only contain values belonging to a predefined **codelist**, in accordance with the INSPIRE specifications.


> [!WARNING]
> Any attempt to insert a value that is not included in the corresponding codelist is considered **invalid** by the system and will result in the **failure of the data insertion operation**.

#### Codelist Definition

The complete list of allowed codes is stored in the **codelist table**.  
The associated [documentation](codelist.md), provides a detailed description of:

- which codes are available (codelist URL),
- the database tables to which each codelist applies,
- the fields for which each code is valid,

in accordance with the adopted conceptual model.

#### Validation and Data Entry

The semantic and syntactic validation of the inserted values is enforced at the database level through dedicated **control triggers** (i_soilinvestigationpurpose/u_soilinvestigationpurpose), ensuring compliance with the defined codelists.

> [!IMPORTANT]
>During data entry via the **QGIS interface**, users are supported by **dropdown menus** that display only the valid codes for the selected field.

> [!NOTE]
>This mechanism **reduces the risk of data entry errors** and guarantees alignment with the constraints imposed by the INSPIRE codelists.

### Relationships (as child)
- None

### Referenced by (as parent)
- `datastream.guid_soilsite` → `soilsite.guid` (**ON UPDATE** CASCADE, **ON DELETE** CASCADE)
- `soilplot.locatedon` → `soilsite.guid` (**ON UPDATE** CASCADE, **ON DELETE** NO ACTION)

### Indexes

| Name | Unique | Columns | Origin | Partial |
|------|--------|---------|--------|---------|
| `idx_soilsite_geom` | No | `geometry` | `c` | No |
| `sqlite_autoindex_soilsite_1` | Yes | `guid` | `u` | No |

### Triggers

For every trigger you will find:

- **When it runs** (BEFORE/AFTER, INSERT/UPDATE/DELETE)
- **What it reads and compares** (columns, lookups in other tables)
- **What happens on success** (the statement proceeds, optional updates)
- **What happens on failure** (the exact error text raised)

 


#### `soilsiteguid`
**When it runs:** AFTER INSERT (per row)

**What it does:** It examines the newly inserted row. If `NEW.guid` is `NULL` or absent, it generates a fresh **UUID v4** in lowercase canonical format and prepares to set it.

**If the check passes:** It updates the just-inserted row, setting `guid` to the generated UUID. No error is raised.

**If the check fails:** No failure path — it only assigns when `guid` is missing.

#### `soilsiteguidupdate`
**When it runs:** AFTER UPDATE OF `guid`

**What it does:** It compares `NEW.guid` with `OLD.guid` to detect any attempt to change the GUID.

**If the check passes:** If the GUID is unchanged, the update proceeds.

**If the check fails:** If `NEW.guid <> OLD.guid`, the statement is aborted with: `Cannot update guid column.`

#### `i_ceckvalidperiodsoilsite`
**When it runs:** BEFORE INSERT

**What it does:** Reads `validfrom` and `validto` from the new row and checks that the **from** date is **not later** than the **to** date.

**If the check passes:** Insert continues.

**If the check fails:** Aborts with: `Table soilsite: validto must be less than validfrom`.

#### `u_ceckvalidperiodsoilsite`
**When it runs:** BEFORE UPDATE

**What it does:** Performs the same period consistency check as above using `NEW.validfrom` and `NEW.validto`.

**If the check passes:** Update continues.

**If the check fails:** Aborts with: `Table soilsite: validto must be less than validfrom`.

#### `i_ceckvalidversionsoilsite`
**When it runs:** BEFORE INSERT

**What it does:** Compares lifecycle fields `beginlifespanversion` and `endlifespanversion`; begin must be **earlier** than end.

**If the check passes:** Insert proceeds.

**If the check fails:** Aborts with: `Table soilsite: beginlifespanversion must be less than endlifespanversion`.

##### `i_soilinvestigationpurpose`
**When it runs:** BEFORE INSERT

**What it does:** Looks up `soilinvestigationpurpose` in the `codelist` table, restricting to `collection = 'SoilInvestigationPurposeValue'`.

**If the check passes:** Insert proceeds.

**If the check fails:** Aborts with: `Table soilsite: Invalid value for soilinvestigationpurpose. Must be present in id of soilinvestigationpurposevalue codelist.`

#### `u_soilinvestigationpurpose`
**When it runs:** BEFORE UPDATE

**What it does:** Performs the same codelist membership check as above before updating.

**If the check passes:** Update proceeds.

**If the check fails:** Same abort message as the insert variant.

#### `u_begin_today_soilsite`
**When it runs:** AFTER UPDATE of selected columns

**What it does:** If `endlifespanversion` is in the **future** or **NULL** (record still active), it refreshes `beginlifespanversion` to the **current timestamp**.

**If the check passes:** The row is updated with the refreshed begin-lifespan timestamp.

**If the check fails:** No explicit failure; this trigger only updates the timestamp.

#### `u_begin_today_soilsite_error`
**When it runs:** AFTER UPDATE of the same columns

**What it does:** If `endlifespanversion` is **before now**, it blocks the edit.

**If the check passes:** When end is in the future or NULL, the update continues.

**If the check fails:** Aborts with: `If you change record endlifespanversion must be greater than today`.



