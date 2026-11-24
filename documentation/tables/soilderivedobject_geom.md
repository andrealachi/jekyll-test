# Soil Derived Object Geom

## Definition

> *The `soilderivedobject_geom` table is used to store the geometric representation of Soil Derived Objects. Geometry management is separated from the `soilderivedobject` table, allowing a single Soil Derived Object to be associated with multiple geometries.*

## Table: `soilderivedobject_geom`

### Columns
| Name | Type | Constraints | Description |
|---|---|---|---|
| `id` | `INTEGER` | PRIMARY KEY | Primary Key of the Table. |
| `guid` | `TEXT` | UNIQUE | Universally unique identifier. |
| `geometry` | `MULTIPOLYGON` | NOT NULL | Geometry. |
| `guid_soilderivedobject` | `TEXT` | NOT NULL | Foreign key to the SoilDerivedObject table, guid field. |

### Table Identifiers
In this table, the primary key is the *id* field (integer, auto-incrementing). There is also a text field named **GUID**, which stores a UUID. If GUID is missing or empty, it is generated automatically; later changes are prevented by dedicated triggers.

### Relationships (as child)
- `soilderivedobject_geom.guid_soilderivedobject` → `soilderivedobject.guid` (**ON UPDATE** CASCADE, **ON DELETE** CASCADE)
- *Note:* delete on `soilderivedobject` cascades to `soilderivedobject_geom`.

### Referenced by (as parent)
- None

### Indexes
| Name | Unique | Columns | Origin | Partial |
|---|---|---|---|---|
| `idx_soilderivedobject_geom_guid_soilderivedobject` | No | `guid_soilderivedobject` | `c` | No |

### Triggers
#### `soilderivedobject_geomguid` / `soilderivedobject_geomguidupdate`
**When they run:** AFTER INSERT / AFTER UPDATE OF `guid`  
**What they do:** Assign GUID at insert when missing; prevent changes later.  
**If the check passes:** Insert writes GUID; unchanged updates proceed.  
**If the check fails:** On change, abort with: `Cannot update guid column.`
```
