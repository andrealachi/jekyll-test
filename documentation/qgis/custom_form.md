# Customized Attribute Forms in QGIS


In QGIS, customized attribute forms allow full control over the layout and behavior of the user interface used for viewing, inserting, and editing attribute data.  
Instead of relying on automatically generated forms, a clearer and more structured layout has been defined, enabling precise control over which fields are displayed, how they are grouped, and under which conditions they are visible.

Each field has been associated with a specific **widget**, i.e., a control designed to facilitate data entry and reduce user errors.


## Available Widgets

The main widget types used include:

- `Text Edit`– free‑text input  
- `Checkbox`– boolean values  
- `Date/Time`– date and time selection  
- `Value Map / Value Relation` – predefined lists of values (fixed or derived from another table)  
- `Relation Reference / Relation Editor`– handling one‑to‑many or many‑to‑many table relationships  
- `HTML / Text / Spacer`– support elements for notes, layout structure, or separators  

These widgets allow the form to guide the user and enforce the data‑entry rules defined in the underlying data model.

>[!IMPORTANT]
>When the **Relation Reference** widget contains a large number of entries, it is possible to quickly **filter the list by typing one or more letters** into the text input field.
>The list is automatically filtered in real time based on the entered characters, allowing faster and easier selection of the desired item.



## Purpose of Customized Forms

Customized forms are used to:

- **guide the user** through data entry with a clear and structured layout  
- **apply constraints**, conditional visibility, and default values  
- **ensure consistency** with the data model  
- **display enriched information** (images, documents, related records)

>[!IMPORTANT]
>The goal is to provide a functional interface, more efficient and clearer than the default one, particularly suitable for complex datasets.


# Interface Elements Related to Forms


<p>
  <img src="../assets/form_01.webp"
       alt="Fig.1" align="left" width="500">
On the left side of the interface, a panel ① lists all the records belonging to the layer.  
Selecting a record from this list displays its attribute content on the right-hand side ②, using the associated customized form.

The attribute toolbar provides two main viewing modes ③:
<strong>Form View</strong>  Displays the attribute form for a single record at a time.
<strong>Table View</strong> Shows the attributes in tabular mode, useful for multi-record inspection and quick selection operations.

Selection tools ④ ⑤ determine which records can be opened or edited through the form.  
</p>
<br clear="all"><br> 


>[!NOTE]
>Selection is an essential part of the editing workflow, as it controls which records are loaded into the form.



# Tooltip

<p>
  <img src="../assets/Tooltip.webp"
       alt="Fig.1" align="left" width="500">
These tooltips provide concise, context‑specific descriptions ① of the attributes when the user <strong>hovers over their labels</strong>. They offer additional clarification on the fields’ purpose and meaning within the form, helping users correctly interpret the data without leaving the editing interface.
</p>
<br clear="all"><br> 



>[!TIP]
>For more details, refer to the official QGIS [documentation](https://docs.qgis.org/3.44/en/docs/user_manual/working_with_vector/attribute_table.html)




