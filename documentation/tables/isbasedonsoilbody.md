# Is Based On Soil Body

## Definition

> *The **`isbasedonsoilbody`** table defines a **many-to-many relationship** between soil-derived objects and soil bodies, linking the `soilderivedobject` and `soilbody` tables.
This relationship allows soil-derived objects to be based on one or more soil bodies and, conversely, enables a single soil body to contribute to multiple derived objects.*

<p>&nbsp;</p>

<p>
  <img src="../assets/isbasedonsoilbody.svg"
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

## Table: `isbasedonsoilbody`

### Columns

| Name | Type | Constraints | Description |
|------|------|-------------|-------------|
| `id` | `INTEGER` | PRIMARY KEY | Primary Key of the Table. |
| `guid_soilderivedobject` | `TEXT` | NOT NULL | Foreign key to the SoilDerivedObject table, guid field. |
| `guid_soilbody` | `TEXT` | NOT NULL | Foreign key to the SoilBody table, guid field. |

### Relationships (as child)
- `isbasedonsoilbody.guid_soilbody` → `soilbody.guid` (**ON UPDATE** CASCADE, **ON DELETE** CASCADE)
  - *Note:* delete on `soilbody` cascades to `isbasedonsoilbody`.
- `isbasedonsoilbody.guid_soilderivedobject` → `soilderivedobject.guid` (**ON UPDATE** CASCADE, **ON DELETE** CASCADE)
  - *Note:* delete on `soilderivedobject` cascades to `isbasedonsoilbody`.

### Referenced by (as parent)
- None

### Indexes

| Name | Unique | Columns | Origin | Partial |
|------|--------|---------|--------|---------|
| `sqlite_autoindex_isbasedonsoilbody_1` | Yes | `guid_soilderivedobject`, `guid_soilbody` | `u` | No |

### Triggers
- None
