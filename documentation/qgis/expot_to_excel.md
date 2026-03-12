# Exporting a View from QGIS (e.g., to Excel)

This guide explains how to export a **view** from QGIS and what the two key export options do when saving to spreadsheet formats.

## Steps

1) **Select the object to export** (in this case, your view) and **right‑click** to open the **context menu**.  
2) Choose **Export**.  
3) Click **Save Features As…** to open the **Export** dialog.  
4) In the **Export** dialog, select the **output format** you need (for Excel: `MS Office Open XML spreadsheet [XLSX]`).  
5) Set the **output file name** and **location**. If needed, choose **which fields to export** (by default, **all fields** are exported).  

6) **Use aliases for exported name**  
Exports column headers using the **field aliases** (or the **Export name** set in the field mapping) instead of the raw database field names.  
Useful when the file is intended for non‑technical users or when you need translated headers.  
If this option is unchecked, the **original field names** will be used.

7) **Replace all selected raw field values by displayed values**  
Exports the **values as displayed in QGIS** (the “displayed/formatted values”) instead of the raw database values:  
e.g., **Value Relation** labels instead of keys, date/time formatted values, boolean values as “Yes/No”, numeric formatting, etc.  
Enable this when you want a more **human‑readable** output; leave it unchecked if you need the **original codes** for further data processing.

8) *(Optional)* Tick the option to **add the exported file back to the current project** after export (e.g., *Add saved file to map*), then click **OK** to finish.
