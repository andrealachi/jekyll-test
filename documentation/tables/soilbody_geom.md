# Soil Body Geom

## Definition

> *The **`soilbody_geom`** table is used to store the **geometric representation of soil bodies**. Geometry management is intentionally separated from the `soilbody` table in order to provide greater flexibility in spatial representation.
By decoupling geometry from soil body attributes, this design allows a single soil body to be associated with **multiple geometries at different spatial scales**. This approach supports the creation of maps at various resolutions and cartographic scales within the same GeoPackage, without duplicating soil body attribute data.*

The `soilbody_geom` table therefore enables multi-scale spatial visualization and analysis, facilitating the management of generalized and detailed geometries while maintaining consistency across representations.

<p>&nbsp;</p>

<p>
  <img src="../assets/soilbody_geom.svg"
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

## Table: `soilbody_geom`

### Columns

| Name | Type | Constraints | Description |
|------|------|-------------|-------------|
| `id` | `INTEGER` | PRIMARY KEY | Primary Key of the Table. |
| `guid` | `TEXT` |  | Universally unique identifier. |
| `geom` | `MULTIPOLYGON` | NOT NULL | Geometry. |
| `guid_soilbody` | `TEXT` | NOT NULL | Foreign key to the SoilBody table. |

### Table Identifiers
In this table, the primary key is the *id* field (integer, auto-incrementing).  
There is also a text field named **GUID**, which stores a *UUID* (Universally Unique Identifier) compliant with RFC 4122.

Although GUID is not mandatory at the schema level (it is not declared NOT NULL), its functional requirement is enforced by two triggers:
- **soilbody_geomguid (INSERT)** trigger: if GUID is missing or empty, a correctly formatted UUID is generated and inserted automatically.
- **soilbody_geomguidupdate (UPDATE)** trigger: prevents any modification of GUID after insertion, making it immutable (effectively behaving as a stable key).  

Any foreign keys (FK) from other tables reference this table’s GUID field rather than the id field, ensuring stable and interoperable references across datasets and database instances.

> [!NOTE]
> **GUID management** is handled by database triggers, which ensure their automatic generation at the time of record insertion, **without any user involvement**.


### Relationships (as child)
- `soilbody_geom.guid_soilbody` → `soilbody.guid` (**ON UPDATE** CASCADE, **ON DELETE** CASCADE)
  - *Note:* delete on `soilbody` cascades to `soilbody_geom`.

### Referenced by (as parent)
- None

### Indexes

| Name | Unique | Columns | Origin | Partial |
|------|--------|---------|--------|---------|
| `idx_soiBody_geom` | No | `geom` | `c` | No |
| `sqlite_autoindex_soilbody_geom_1` | Yes | `guid` | `u` | No |

### Triggers
For every trigger you will find:

- **When it runs** (BEFORE/AFTER, INSERT/UPDATE/DELETE)
- **What it reads and compares** (columns, lookups in other tables)
- **What happens on success** (the statement proceeds, optional updates)
- **What happens on failure** (the exact error text raised)

 


#### `soilbody_geomguid` / `soilbody_geomguidupdate`
**When they run:** AFTER INSERT / AFTER UPDATE OF `guid`

**What they do:** Assign GUID when missing at insert; prevent changes later.

**If the check passes:** Insert writes GUID; unchanged updates proceed.

**If the check fails:** On change, abort with: `Cannot update guid column.`


