
# Soil Organic Carbon (SOC) 0–30 cm — QGIS Model (SoilWise)

## 1) Objective
This document describes a QGIS model that computes **Soil Organic Carbon (SOC)** as a **depth‑weighted value in the top 0–30 cm**. The workflow integrates datastream selection, 0–30 cm layer filtering, hierarchical joins across profile tables, weighted value computation per datastream, and final aggregation per soil plot.

## 2) Expected Inputs
The model works **exclusively on the SoilWise GeoPackage** and uses the following tables/layers: `profileelement`, `soilprofile`, `datastream`, `observation`, `soilplot`. Expected relations: `profileelement.ispartof → soilprofile.guid`; `datastream.guid_profileelement → profileelement.guid`; `observation.guid_datastream → datastream.guid`; `soilprofile.location → soilplot.guid`.

## 3) Parameters / Constants
- **Depth window:** `[0, 30]` cm.
- **Datastream filter:**
  - `guid_observedproperty = '6022fde8-d281-4d85-92ae-2279f6415245'`
  - `guid_observingprocedure = '9c6f5f47-352e-463b-9c4e-6320a154040a'`.

---

## 4) Workflow (condensed)
1. **Select datastreams** by observed property & observing procedure.
2. **Filter profile elements** intersecting 0–30 cm: `upper < 30` and `lower > 0`.
3. **Join** `soilprofile ↔ profileelement` (filtered).
4. **Join** with selected `datastream` and **retain** only needed fields.
5. **Join** with `observation` to bring `result_real` and time.
6. **Compute `oc_0_30` (Field Calculator)**:
   - Per row, compute a weighted contribution:  

$$
w = result\cdot\frac{\max(0,\min(30,low)-\max(0,up))}{low-up}
$$

   - If a `guid_3` (datastream) is **unique** → `oc_0_30 = w`; if **repeated** → see 5.2.
7. **De‑duplicate** by `guid_3` (one row per datastream).
8. **Statistics by soilplot** (`location`): use **sum** and **rename** to `soc_0_30`.
9. **Join** result back to `soilplot` and apply the `CO_0_30.qml` style.

---

## 5) Key Expressions
### 5.1 Row weight (depth‑weighted contribution)

$$
w = result\cdot\frac{\max(0,\min(30,low)-\max(0,up))}{low-up}
$$

### 5.2 Rule for the final per‑datastream value (no restricted macros)

- If `count(guid_3) = 1`:

$$
oc_{0-30} = w
$$

- If `count(guid_3) > 1`:

$$
oc_{0-30} = \frac{1}{n}\sum_{i=1}^{n} w_i
$$



## 6) Output
- Final layer: **`soilplot`** with attribute **`soc_0_30`** (SOC 0–30 cm) and the **`CO_0_30.qml`** style applied.

---

## 7) Operational Notes
- On some GitHub instances, certain LaTeX macros (e.g. `\operatorname`) are blocked. The formulas above avoid these macros by design.
- **NULL/decimals:** `result_real` decimals are normalized; consider excluding NULLs upstream if required.
- **Depth edge cases:** to include layers that touch 0, change the filter to `lower >= 0`.
- **Soilplot aggregation:** currently **sum**; switch to **mean** in the statistics step if needed.

*Last updated:* 2026-02-23
