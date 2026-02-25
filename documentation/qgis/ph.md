
# Mean Profile pH — QGIS Model

Download the [model file](../../models/ph/SoilWise_SP_mean_pH.model3) and its corresponding [style](../../models/ph/pH_USDA.qml)

[model file](https://github.com/andrealachi/jekyll-test/blob/main/models/ph/SoilWise_SP_mean_pH.model3)

[scarica](https://raw.githubusercontent.com/andrealachi/jekyll-test/refs/heads/main/models/ph/SoilWise_SP_mean_pH.model3)

>[!WARNING]
>**Model and style must be co‑located**.
>
>The model file and its corresponding style file must reside in the same folder. If either file is moved or the directory structure changes, relative references will break and the style will not be applied at >runtime/build time.


## 1) Objective
This document describes a QGIS model that computes the **mean pH per soil plot/profile** from *Observations* by selecting **pH** datastreams measured with **H₂O** procedure, normalizing multiple observations per datastream to a group mean, and finally aggregating per plot.

> [!TIP]
> For further information on model creation and execution, consult the **official QGIS documentation** for the [Model Designer](https://docs.qgis.org/3.40/en/docs/user_manual/processing/modeler.html).


## 2) Expected Inputs
The model works **exclusively on the SoilWise GeoPackage** and uses the following tables/layers: `profileelement`, `soilprofile`, `datastream`, `observation`, `soilplot`. Expected relations: `profileelement.ispartof → soilprofile.guid`; `datastream.guid_profileelement → profileelement.guid`; `observation.guid_datastream → datastream.guid`; `soilprofile.location → soilplot.guid`.

## 3) Parameters / Constants
- **Datastream filter (pH in H₂O):**
  - `guid_observedproperty = '2422dfa9-82a2-4e73-9f63-630d055dcd43'`  - [pH](http://w3id.org/glosis/model/codelists/physioChemicalPropertyCode-pH) 
  - `guid_observingprocedure = 'a6cbc33c-872b-45bb-85ba-18b35fbf6813'`  - [pHH2O_ratio1-5](http://w3id.org/glosis/model/procedure/pHProcedure-pHH2O_ratio1-5) 


## 4) Workflow (condensed)
1. **Select datastreams** by observed property & observing procedure (pH + H₂O).
2. **Join** `soilprofile ↔ profileelement` (inner join, discard non‑matching).
3. **Join** with selected `datastream` (inner) and **retain** the essential fields.
4. **Join** with `observation` to bring `result_real` and timestamp.
5. **Compute `Ph_Obs` (Field Calculator):**
   - If a `guid_3` (datastream) is **unique** → `Ph_Obs = result_real`.
   - If **repeated** → assign the **mean(result_real)** of the group to all rows.
6. **De‑duplicate** by `guid_3` (one row per datastream).
7. **Statistics by categories** (`guid;location`) over `Ph_Obs`.
   - Rename the field **`median` → `Average_pH`**.
8. **Join** the statistic back to `soilplot`.
9. **Apply** the style `pH_USDA.qml`.


## 5) Output
- Final layer: **`soilplot`** with attribute **`Average_pH`** (plot‑level pH) and the **`pH_USDA.qml`** style applied.

