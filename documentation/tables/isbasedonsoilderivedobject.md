# Is Based On Soil Derived Object

## Definition

>*The **`isbasedonsoilderivedobject`** table is designed to represent **derivation relationships between soil-derived objects**, allowing a soil-derived object to be based on another soil-derived object.
This table implements a **self-referencing (recursive) relationship** on the `soilderivedobject` entity, enabling the explicit modeling of derivation chains, dependencies, or transformations within the set of soil-derived objects. Through this mechanism, it is possible to describe cases where a derived object results from the refinement, aggregation, reinterpretation, or further processing of an existing derived object, rather than directly from a primary soil entity.*

The use of a dedicated relationship table ensures that derivation links are:

- **explicitly defined and traceable**  
- **independent from the internal structure** of the `soilderivedobject` table  
- **capable of supporting one-to-one, one-to-many, and many-to-many derivation scenarios**

By allowing soil-derived objects to reference other soil-derived objects as their source, this table supports the representation of **multi-level derivation workflows**, which are common in soil data production processes (e.g. successive interpretations, modelling steps, or product generalizations).

This approach improves both:

- **data lineage transparency**, by documenting how derived products are generated  
- **data integrity**, by enforcing consistent relationships between derived entities within the GeoPackage
  
<p>&nbsp;</p>

<p>
  <img src="../assets/isbasedonsoilderivedobject.svg"
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

## Table: `isbasedonsoilderivedobject`

### Columns

| Name | Type | Constraints | Description |
|------|------|-------------|-------------|
| `id` | `INTEGER` | PRIMARY KEY | Primary Key of the Table. |
| `guid_base` | `TEXT` | NOT NULL | Foreign key to the SoilDerivedObject table, guid field. - Base SoilDerivedObject. |
| `guid_related` | `TEXT` | NOT NULL | Foreign key to the SoilDerivedObject table, guid field. - Derived SoilDerivedObject. |

### Relationships (as child)
- `isbasedonsoilderivedobject.guid_related` → `soilderivedobject.guid` (**ON UPDATE** CASCADE, **ON DELETE** CASCADE)
  - *Note:* delete on `soilderivedobject` cascades to `isbasedonsoilderivedobject`.
- `isbasedonsoilderivedobject.guid_base` → `soilderivedobject.guid` (**ON UPDATE** CASCADE, **ON DELETE** CASCADE)
  - *Note:* delete on `soilderivedobject` cascades to `isbasedonsoilderivedobject`.

### Referenced by (as parent)
- None

### Indexes

| Name | Unique | Columns | Origin | Partial |
|------|--------|---------|--------|---------|
| `sqlite_autoindex_isbasedonsoilderivedobject_1` | Yes | `guid_base`, `guid_related` | `u` | No |

### Triggers
- None

