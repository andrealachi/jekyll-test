
# QField – Operational Guide (QGIS ⇆ QField)
This short guide briefly describes:
- how to **install the QField plugin (QFieldSync) from QGIS**;
- how to **identify the projects** inside the provided **GeoPackage**;
- how to **export/package the QGIS project** for use with QField (Android, iOS, Windows);
- how to **transfer the data to the device** for use with QField (Android, iOS, Windows).

> [!IMPORTANT]
> This is an introductory guide; for more details on using QField and its plugin, please refer to the official documentation.

## 1) Install the QField plugin from QGIS

### Step‑by‑step procedure
1. **Open QGIS** (Desktop).
2. Go to **Plugins → Manage and Install Plugins…**
3. In the search field, type **“QFieldSync”** (in some installations it is shown as **“QField”**).
4. Select **QFieldSync** and click **Install Plugin**.
5. (Optional) Verify it is enabled: **Installed** tab → check **QFieldSync**.

### What the plugin enables
- **Layer configuration** for offline use (copy to GeoPackage, attachment handling).
- **Export/Packaging** of the project for QField.
- Conversion to **relative paths**, copying of resources (icons, media), inclusion of styles and forms.

> **Tip:** Before exporting, always save the QGIS project and ensure **paths are relative** (**Project → Properties → Paths**) to avoid broken links on the device.

## 2) Identifying the project in the GeoPackage
The provided GeoPackage contains **two different QGIS projects** that point to the **same data source** (the same layers/tables in the same `.gpkg`):
- **Project `_PRJ_SO` for QGIS** – Optimized for **desktop** use (full symbology, views, and tools for office work).
- **Project `_PRJ_SO_QField` for QField** – Optimized for **mobile devices** with **custom forms**, pre‑filled fields, simplified widgets, and constraints for field data capture.

Both projects use **the same GeoPackage** as their data source; this ensures consistency and alignment between office work and field work.

> **Operational note:**
> - Open `_PRJ_SO.qgs/.qgz` in **QGIS** when working on desktop.
> - Open `_PRJ_SO_QField.qgs/.qgz` in **QField** (or use it as the source for export via QFieldSync).

---

## 3) Exporting/Packaging the QGIS project for QField

### A. Configure layers for offline use
1. Open the **`_PRJ_SO_QField`** project in QGIS.
2. In the **QFieldSync toolbar**, open **Project Configuration** (or **Project → QFieldSync…**).
3. For each **layer**, you can set the mode:
   - **Copy to GeoPackage** *(recommended)*: creates an offline copy for field editing.
   - **Keep original path**: keeps the reference to the external file (less portable).
   - **Read‑only**: if it must not be edited.

**NOTE:** Unless there are specific needs, it is discouraged to change the project configuration. The full export of the GeoPackage can be performed without altering the configuration.

### B. Export the project
1. From the **QFieldSync** menu or toolbar, choose **Export/Package for QField**.
2. Select the **destination folder**.
3. Start the export: the plugin will copy data, styles, and resources, converting paths to **relative** ones.

### Typical resulting structure:
```text
/Project_QField/
├─ project.qgs            # QField project
├─ data/                  # GeoPackage and copied layers
├─ media/                 # Photos/icons/attachments
└─ styles/                # Any styles and resources
```

> **Output alternatives:**
> - **Folder** (most common): ready to copy via USB/Cloud.
> - **Compressed archive (.zip)**: convenient for e‑mail or cloud platforms.
> - **`.qfield` file** (if supported by your version): a single package that can be opened directly by QField.

## 4). Transfer to device
- **Android/iOS:** copy the folder/zip via **USB** or **cloud** (OneDrive/Drive/Nextcloud). On iOS, use the **Files** app or share directly with QField.
- **Windows:** copy the folder/zip and open the project with **QField for Windows**.

> Warning
> **Write errors:** make sure layers are not read‑only and that the device allows writing to the project folder.  
> **Heavy rasters:** consider **resampling** or using lighter tiles for better performance on mobile.
