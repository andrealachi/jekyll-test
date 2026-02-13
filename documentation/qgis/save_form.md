# Saving Edits in QGIS Forms

## Edit Buffer
When a layer is [put into editing mode](./edit_form.md) , all operations performed through the form—such as inserting, updating, or deleting records—are **not written directly** to the data source.  
QGIS uses an **edit buffer**, an in‑memory structure that:

- records every modification made by the user,  
- allows local undo/redo during the editing session,  
- isolates changes from the physical dataset until they are explicitly committed.

> [!WARNING]
>The contents of the buffer are temporary: if the editing session ends without saving, the buffer is discarded.



## Committing Changes (Save Layer Edits)
Saving occurs when the user clicks **Save Layer Edits** <img src="../assets/save.webp">. At this stage, QGIS performs a **commit** of the edit buffer to the underlying data source (GeoPackage, Shapefile, PostGIS, etc.).

During the commit:

- the buffer is processed and its operations are converted into write actions,  
- QGIS performs **data validation** (constraints, required fields, data types, domains, unique values, parent–child relationships),  
- any validation errors are reported before the commit, preventing inconsistent data from being written.

If validation succeeds, the buffer is cleared and the dataset is updated.

> [!CAUTION]
>**Errors are displayed in a banner within the map canvas**. However, the banner may not always be visible when some panels are in a floating (undocked) state, which can partially cover or obscure the map canvas.
Make sure to check the map canvas if error messages are not immediately visible.

> [!TIP]
> In case of an error, to understand its cause it is necessary to locate the corresponding message directly within the **trigger section** of the table associated with the form where the error occurred.



## Rolling Back Changes
When editing mode is disabled <img src="../assets/pencil.webp">, QGIS prompts the user to either:

- **save** pending changes, or  
- **discard** them.

If the user chooses not to save, QGIS performs a **rollback**:

- the edit buffer is removed,  
- all unsaved modifications are discarded,  
- the layer returns to its original on‑disk state.

This process is similar to rolling back a transaction in a relational database.



## Handling Parent–Child Relationships
When a layer is part of a relational structure (e.g., a GeoPackage with related tables), forms may include sub‑forms for editing child records.

Technically:

- each involved layer maintains its **own edit buffer**,  
- modifications to child records are not saved automatically when the parent layer is saved,  
 

## Persistence and Disk Writing
At the end of a successful commit:

- QGIS writes all changes to the GeoPackage (SQLite) or the target database,  
- indexes and internal metadata structures are updated,  
- any database‑level triggers or constraints are executed.

The dataset now reflects the final state of the editing session.




