
# SOILWISE Soil GeoPackage Template — Data Loading & Modelling Guide

**Goal.** Provide a complete, deterministic **loading order** and **data entry rules** to populate the database that implements the INSPIRE Soil application schema. The guide covers the logical flow **Site → Plot → Profile → Elements**, the **Observed vs Derived** distinction, **Horizon vs Layer** behaviour, **classifications**, **associations**, and **STA2 (datastream/observation)** integration. References to the INSPIRE Technical Guidelines and Feature Concept Dictionary are provided as footnotes.[^2][^3][^4][^5]



## 1) Concepts at a glance

### Observed vs Derived Soil Profile
- **Observed Soil Profile**: a soil profile **found at a specific location**, described from a trial pit or a borehole. 
- **Derived Soil Profile**: a **non point‑located** reference profile for a soil type in a wider area.

(See official INSPIRE definitions.)[^3]

### Profile Element → Horizon vs Layer
- **Profile Element** is an abstract type grouping the vertical slices that compose a soil profile. It specialises into:
  - **Soil Horizon**: pedogenetically formed, relatively homogeneous layer, approximately parallel to the surface, recognised by morphological/analytical characteristics; 
  - **Soil Layer**: an **arbitrary** (often depth‑based) slice or a **grouping of horizons** for operational purposes, not necessarily matching pedogenic boundaries.  

(See official INSPIRE definitions.)[^4][^5]



## 2) Conventions & prerequisites

- **Authoritative source** for table structures, constraints, and triggers: the DDL shipped with this repository.[^1]
- **Code lists.** Values are validated via the `codelist` table. You **must preload** every collection you plan to use *before* you insert domain records (e.g., `SoilInvestigationPurposeValue`, `SoilPlotTypeValue`, `LayerTypeValue`, `EventProcessValue`, `EventEnvironmentValue`, `LayerGenesisProcessStateValue`, `LithologyValue`, FAO/WRB codelists, `wrbversion`, `Category`).
- **Temporal rules.** Many tables implement: `beginlifespanversion = now()` on insert; `beginlifespanversion` auto‑update upon relevant changes; checks on `validfrom ≤ validto` and `beginlifespanversion < endlifespanversion`.
- **GUIDs.** If `guid` is NULL on insert, the DB generates a UUIDv4‑like lowercase value; updates to `guid` are blocked by triggers.



## 3) Recommended loading order (deterministic)

1. **Lookups:** `codelist`, `unitofmeasure`  
2. **Spatial features:** `soilsite`  
3. **`soilplot`** (FK → `soilsite`)  
4. **`soilprofile`** — choose type via `isderived` (Observed or Derived; see §4.3)  
5. **`isderivedfrom`** — link Derived ↔ Observed profiles  
6. **`soilbody`**, then **`soilbody_geom`**  
7. **`derivedprofilepresenceinsoilbody`** — assign Derived profiles with shares (≤ 100 per soilbody)  
8. **`profileelement`** (then, for Horizons only: `faohorizonnotationtype`, `otherhorizonnotationtype` + `otherhorizon_profileelement`)  
9. **WRB qualifiers:** `wrbqualifiergrouptype`, then `wrbqualifiergroup_profile`  
10. **STA2 master data:** `observedproperty`, `sensor`, `observingprocedure`, `obsprocedure_obsdproperty`, `obsprocedure_sensor`  
11. **`datastream`** — set the FOI; triggers resolve and/or create the target `thing`  
12. **`observation`** — value‑level checks per `datastream.type`  
13. **(Optional)** `soilderivedobject` and its relations: `isbasedonobservedsoilprofile`, `isbasedonsoilbody`, `isbasedonsoilderivedobject`

> **Why this order?** Each step unlocks the next (FKs and triggers). For instance: Plots need Sites; Observed Profiles need Plots; Derived–Observed links need both sides; Elements need Profiles; Notations and WRB need both codelists and target types; Datastreams require master data and FOI; Observations require Datastreams and, for Category, a codespace collection.



## 4) Table‑specific rules (summary)

### 4.1 `soilsite` (POLYGON)
- **Required**: `geometry`, `soilinvestigationpurpose` ∈ `SoilInvestigationPurposeValue`.
- **Temporal checks**: `validfrom ≤ validto`; `beginlifespanversion < endlifespanversion`.

### 4.2 `soilplot` (POINT)
- **Required**: `soilplotlocation`, `soilplottype` ∈ `SoilPlotTypeValue`.
- **FK**: `locatedon` → `soilsite.guid` (Site must exist first).

### 4.3 `soilprofile` (Observed vs Derived)
- **Type switch**: `isderived` → `0 = Observed`, `1 = Derived`.[^3]
- **Observed (`isderived=0`)**: `location` **NOT NULL** and references **`soilplot.guid`**; `location` is **UNIQUE** (one observed profile per plot).
- **Derived (`isderived=1`)**: `location` **MUST be NULL** (non point‑located profile).
- **WRB pairing**: `wrbversion` ∈ `wrbversion` collection; `wrbreferencesoilgroup` must match the year‑specific collection coherent with `wrbversion` (INSPIRE/2014/2022).
- **Temporal checks** as in §4.1.

### 4.4 `isderivedfrom` (Derived → Observed)
- `guid_base` → `soilprofile.guid` with `isderived=1`; `guid_related` → `soilprofile.guid` with `isderived=0`. Triggers enforce types and uniqueness.

### 4.5 `soilbody` & `soilbody_geom`
- Create `soilbody` first, then `soilbody_geom` (MULTIPOLYGON) referencing it.

### 4.6 `derivedprofilepresenceinsoilbody`
- `guid_soilprofile` must be **Derived**. 
- Sum of `lowervalue` for the same `guid_soilbody` must **not exceed 100.00** (trigger).

### 4.7 `profileelement` → Horizon vs Layer
- **Type switch**: `profileelementtype` → `0 = HORIZON`, `1 = LAYER`. (See INSPIRE definitions.)[^4][^5]
- **Depth integrity**: at least one of `…_uppervalue` / `…_lowervalue` must be set; if both are set, `uppervalue < lowervalue` (depth increases downward).
- **Parent**: `ispartof` → `soilprofile.guid`.

**Horizon (`profileelementtype=0`)**
- Layer‑specific fields **must be NULL**: `layertype`, `layerrocktype`, `layergenesisprocess`, `layergenesisenviroment`, `layergenesisprocessstate`.
- **Horizon notation enabled**: may link to `faohorizonnotationtype` (1:1) and to `otherhorizonnotationtype` via `otherhorizon_profileelement` (N:M). Triggers enforce that the target is a Horizon.

**Layer (`profileelementtype=1`)**
- Domain validations: 
  - `layertype` ∈ `LayerTypeValue`; `layerrocktype` ∈ `LithologyValue`; 
  - `layergenesisprocess` ∈ `EventProcessValue`; `layergenesisenviroment` ∈ `EventEnvironmentValue`; 
  - `layergenesisprocessstate` ∈ `LayerGenesisProcessStateValue`.
- **Geogenic guardrails**: when `layertype` ≠ geogenic, genesis‑related fields **must be NULL**. 
- **Horizon notation is not applicable** (triggers block it).

### 4.8 Horizon notation
- **`faohorizonnotationtype`** (1:1 to Horizon via `guid_profileelement`): FAO masters/subordinates/prime from their codelists; trigger ensures the PE is a Horizon.
- **`otherhorizonnotationtype`** + **`otherhorizon_profileelement`** (N:M): `horizonnotation` ∈ `OtherHorizonNotationTypeValue`; `diagnostichorizon` must belong to the collection indicated by `horizonnotation`.

### 4.9 WRB qualifiers
- **`wrbqualifiergrouptype`** (qualifier group library): 
  - `wrbversion` ∈ `wrbversion`; `qualifierplace` ∈ `WRBQualifierPlaceValue`; 
  - `wrbqualifier` and `wrbspecifier_1/_2` must come from year‑specific collections coherent with `wrbversion`.
  - Uniqueness: `(wrbversion, qualifierplace, wrbqualifier, wrbspecifier_1, wrbspecifier_2)`.
  - Logic: `_1` required if `_2` is set; `_1` and `_2` must differ.
- **`wrbqualifiergroup_profile`** (attach groups to profiles): version **match** (profile vs group) enforced; `qualifierposition` **unique** per `(soilprofile, qualifierplace)`.

### 4.10 Soilderivedobject and linkages
- **`soilderivedobject`** (POLYGON) can represent thematic outputs (derived maps/envelopes).
- **Associations**: 
  - `isbasedonobservedsoilprofile` (link to **Observed** profiles); 
  - `isbasedonsoilbody`; 
  - `isbasedonsoilderivedobject` (hierarchies of derived objects).

### 4.11 STA2 master data
- **`observedproperty`**: `name`; `definition` must be a well‑formed URI (checked). Optional JSON `properties`.
- **`sensor`**: `name`; if `metadata` is set, `encodingtype` must be `PDF` or `HTML`; otherwise both NULL.
- **`observingprocedure`**: `name`; optional `definition` URI.
- **N:M registries**: 
  - `obsprocedure_obsdproperty` (procedure ↔ property) — the pair is **required** if a procedure is set on a `datastream`;
  - `obsprocedure_sensor` (procedure ↔ sensor) — optional.

### 4.12 `datastream` (series definition & FOI)
- **Type & combinations**: `type ∈ {'Quantity','Category','Boolean','Count','Text'}` with table‑level checks:
  - **Quantity**: requires `code_unitofmeasure` and forbids `codespace` and categorical bounds;
  - **Category**: requires `codespace` (well‑formed URI and a `codelist.collection`), forbids units and numeric bounds;
  - **Boolean/Count/Text**: shape and bounds rules apply at `observation` level (see below).
- **FOI multiplicity**: exactly **1** among `guid_soilsite`, `guid_soilprofile`, `guid_profileelement`, `guid_soilderivedobject`.
- **Auto‑resolve to `thing`** (triggered):
  - FOI = **SoilSite** → `guid_thing = SoilSite.guid`;
  - FOI = **SoilProfile (Derived)** → `guid_thing = SoilProfile.guid`;
  - FOI = **SoilProfile (Observed)** → resolve to **SoilSite** via `SoilPlot` (Profile→Plot→Site);
  - FOI = **ProfileElement** → if parent profile is **Derived**, use the **profile**; if **Observed**, use the **site** via Plot;
  - FOI = **SoilDerivedObject** → `guid_thing = Soilderivedobject.guid`.
  Missing `thing` rows are created **on demand** with reasonable defaults.

### 4.13 `observation` (values)
- **Uniqueness**: `(phenomenontime_start, guid_datastream)`.
- **Shape by `datastream.type`** (enforced by triggers):
  - **Quantity**: `result_real` only; apply numeric bounds if defined; 
  - **Category**: `result_text` only; must belong to `codelist.id` within the `codespace` collection;
  - **Boolean**: `result_boolean ∈ {0,1}` only; 
  - **Count**: `result_real` must be **integer in value**; obey numeric bounds if defined; 
  - **Text**: `result_text` only.
- **Datastream range maintenance**: `datastream.phenomenontime_*` is recalculated from member observations on insert/update/delete.



## 5) Troubleshooting checklist

- **Code list errors** → Preload the exact `collection` and `id` values referenced by the target column.
- **`soilprofile` rejects insert** → Check `isderived` vs `location` rule; verify WRB version/group pairing.
- **`derivedprofilepresenceinsoilbody` fails** → The sum of `lowervalue` for the same `soilbody` would exceed **100**.
- **`profileelement` fails** → Depth range order (`upper < lower`), Horizon/Layer field constraints, geogenic rule.
- **`wrbqualifiergroup_profile` fails** → `wrbversion` mismatch or duplicate `qualifierposition` for the same `qualifierplace`.
- **`datastream` fails** → FOI multiplicity (must be 0 or 1); type/combination checks; required `(procedure, property)` pair.
- **`observation` fails** → Shape/type mismatch, bounds violation, or Category value not in the `codespace` collection.
- **Unexpected FOI → thing mapping** → Review the resolution rules in §4.12 (Observed Profile & some ProfileElements resolve to **SoilSite**).



## References

[^1]: **GeoPackage Template DDL (tables, constraints, triggers)**.  
./DDL_SO_14.sql

[^2]: **INSPIRE Technical Guidelines — Soil (2024, entry page)**.  
https://inspire-mif.github.io/technical-guidelines/data/so/

[^3]: **Observed & Derived Soil Profile — Feature Concept Dictionary**.  
Observed: https://inspire.ec.europa.eu/featureconcept/ObservedSoilProfile  
Derived: https://inspire.ec.europa.eu/featureconcept/DerivedSoilProfile

[^4]: **Horizon/Layer/ProfileElement — Feature Concept Dictionary**.  
Soil Horizon: https://inspire.ec.europa.eu/featureconcept/SoilHorizon  
Soil Layer: https://inspire.ec.europa.eu/featureconcept/SoilLayer  
Profile Element: https://inspire.ec.europa.eu/featureconcept/ProfileElement:1

[^5]: **INSPIRE Soil — Overview slide deck (Site–Plot–Profile–Horizon/Layer)**.  
https://zenodo.org/records/13970777/files/II3-INSPIRE-Soil.pdf
