# Export to Excle

## Introduction

QGIS supports exporting layers to a wide range of formats to help you share, analyze, and report your data beyond the GIS environment. This short guide focuses specifically on **exporting to Microsoft Excel (XLSX)** using the project view **`view_observation`** as the working example. The same workflow applies—often with only minor differences—to **most vector layers and database views** you might have in your project.

> [!TIP]
> For further information on the teh View Observation, consult the documents [Customized Attribute Forms in QGIS](./custom_form.md).
## Export the View Observation in Excel


<p>
  <img src="../assets/exp_exc_01.webp"
       alt="Fig.1" align="left" width="500">
  
① **Select the object to export** (in this case, your view) and **right‑click** to open the **context menu**.  
  
② Choose **Export**.  

③ Click **Save Features As…** to open the **Export** dialog. 

</p>
<br clear="all"><br>

<p>
  <img src="../assets/exp_exc_02.webp"
       alt="Fig.1" align="left" width="500">
  
  ④ In the **Export** dialog, select the **output format** you need (for Excel: `MS Office Open XML spreadsheet [XLSX]`).  
</p>
<br clear="all"><br>


<p>
  <img src="../assets/exp_exc_03.webp"
       alt="Fig.1" align="left" width="500">
  
⑤ Set the **output file name** and **location**. If needed, choose **which fields to export** ⑥ (by default, **all fields** are exported).  

⑦ **Use aliases for exported name**  
Exports column headers using the **field aliases** (or the **Export name** set in the field mapping) instead of the raw database field names.  
Useful when the file is intended for non‑technical users or when you need translated headers.  
If this option is unchecked, the **original field names** will be used.

⑧  **Replace all selected raw field values by displayed values**  
Exports the **values as displayed in QGIS** (the “displayed/formatted values”) instead of the raw database values:  
e.g., **Value Relation** labels instead of keys, date/time formatted values, boolean values as “Yes/No”, numeric formatting, etc.  
Enable this when you want a more **human‑readable** output; leave it unchecked if you need the **original codes** for further data processing.

⑨ **(Optional)** Tick the option to **add the exported file back to the current project** after export (e.g., *Add saved file to map*), then click **OK** ⑩ to finish.

</p>
<br clear="all"><br>








