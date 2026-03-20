# QField — Quick Guide (Project Loading & Navigation)
**Opening in QField**
1. Start **QField**.
2. **Open project** → navigate to the folder → select **`.qgs/.qgz`**.
3. Wait for layers and styles to load.
> Tip: if the project doesn't open or some layers are missing, check in QGIS that the **paths** are **relative** (*Project → Properties → Paths*).
---
## 2) Interface Navigation
### Map
- **Pan/Zoom** (drag, pinch), **Rotation** (two fingers), **reset**.
- **GPS**: target button to center on your position.
### Toolbar *(icons may vary)*
- **🔍 Identify**: tap an object → **attribute/form panel**.
- **✏️ Edit**: enable editing on the active layer.
- **➕ Add feature**: create **point/line/polygon** (choose layer, fill the form).
- **📏 Measure**: distances/areas on the map.
- **🔎 Search**: attribute search (if available).
- **🧭 GPS/Tracking**: capture geometries from position.
### Layer Panel
- **Visibility** on/off; select the **editing layer** before editing.
- (If configured) **Map themes** for predefined layer sets.
### Forms & Attachments
- When selecting/creating, a **form** opens: required fields, pick‑lists, dates, numbers.
- **📷 Photos/Attachments**: take or attach files → saved in the project *media* folder.
- **✔ Save** / **✖ Cancel**.
---
## 3) Quick Workflows
**A) Add a point**
1. **✏️ Edit** → choose **layer**.
2. **➕ Point** → tap the map (or use GPS).
3. Fill the **form** → **✔ Save**.
**B) Edit attributes**
1. **🔍 Identify** → **Edit**.
2. Update fields → **✔ Save**.
## 4) Saving & Back to Office
- Changes are written directly into the data (**GeoPackage**).
- With **QFieldCloud**: **Sync**.
- Without cloud: **copy the project folder** (or only the `.gpkg`) from the device to the PC and open in QGIS.
---
## 5) Mini‑troubleshooting
- **Inaccurate GPS** → wait for fix, enable high precision (settings), check location provider.
- **Slow map** → simplify symbology, reduce raster size, use lighter tiles.
---