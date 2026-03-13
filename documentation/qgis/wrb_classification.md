# WRB Classification in the Soilwise GeoPackage

The **World Reference Base for Soil Resources (WRB)** is today’s main international standard for soil classification, widely adopted in both scientific and operational contexts to ensure a common language among researchers, technicians, and institutions. Within the **Soilwise GeoPackage**, WRB is integrated through a structured set of domains, lookup tables, and controlled fields designed to guide the user in the consistent compilation of soil profile descriptive data.

This chapter explains how to correctly use the tools provided by the GeoPackage to classify a soil profile according to the WRB system — from identifying diagnostic features to assigning the **Reference Soil Group (RSG)** and the corresponding **qualifiers**.  
The goal is to provide a simple and reliable workflow that helps the user avoid common mistakes, maintain consistency with the official definitions, and produce a classification fully compliant with the WRB standard.

## Introduction

<p>
  <a href="../assets/wrb_01.webp" target="_blank">
    <img src="../assets/wrb_01.webp"
         alt="Fig.1" align="left" width="500">
  </a>
Among the attributes of a "Soil Profile", whether it is of type <strong>"Observed"</strong> or <strong>"Derived"</strong>, there is the option to insert the <strong>"World Reference Base"</strong> classification ①. <br>
<br>
In the <strong>"Soil Profile"</strong> form, you can select the classification year ② and the <strong>"Reference Soil Groups"</strong>. ③  <br>
</p>
<br clear="all"><br>

<p>
  <a href="../assets/wrb_00.webp" target="_blank">
    <img src="../assets/wrb_00.webp"
         alt="Fig.1" align="left" width="500">
  </a>
As an example, to follow the workflow, let's take the following classification from the 2006 World Reference Base: <br>
<br>
<br>
- Indicate the <strong>"Reference Soil Groups"</strong>, which has already been described in the <strong>"Soil Profile"</strong>. <br>
<br>
- Specify the <strong>"Qualifier"</strong>, and where applicable, the <strong>"Specifier"</strong>. <br>
<br>
- Specify the <strong>"Qualifier Place"</strong>, whether it is a prefix or a suffix. <br>
<br>
- Finally, define the <strong>"Position"</strong> of the qualifier within the classification.
</p>
<br clear="all"><br>

## Detailing the Classification from the Soil Profile Form


<p>
  <a href="../assets/wrb_02.webp" target="_blank">
    <img src="../assets/wrb_02.webp"
         alt="Fig.1" align="left" width="500">
  </a>
In this case as well, a related table is edited directly from the form containing it. Right-click on the <strong>"Layers"</strong> panel on the <strong>"soilprofile"</strong>entry ① and from the menu, select "Open Attribute Table" ②.
</p>
<br clear="all"><br>

<p>
  <a href="../assets/wrb_03.webp" target="_blank">
    <img src="../assets/wrb_03.webp"
         alt="Fig.1" align="left" width="500">
  </a>
Select a "Soil Profile" ③.
Navigate to the "WRB Qualifier" tab, ④ click the pencil icon <strong>"Toggle editing mode for child layer"</strong>⑤ to make the tab editable, and then click the <strong>"Add Child Feature"</strong> button. ⑥ <br>
<br>
A window will open ⑦ allowing you to insert both prefixes and suffixes and their position within the classification.<br>
<br>
The <strong>"Soil Prile" field</strong> ⑧ will already contain the name of the selected "Soil Profile".<br>
<br>
Define the position ⑨ of the prefix or suffix within the classification.<br>
<br>
Search for the relevant prefix or suffix, ⑩ paying attention to the WRB year as well.<br>
</p>
<br clear="all"><br>

<p>
  <a href="../assets/wrb_04.webp" target="_blank">
    <img src="../assets/wrb_04.webp"
         alt="Fig.1" align="left" width="500">
  </a>
It is possible to filter the list values by typing within the field. The letter <strong>"P"</strong> indicates a <strong>"Prefix"</strong>, and <strong>"S"</strong> indicates a <strong>"Suffix"</strong>.<br>
<br>
Close the window by clicking the <strong>"OK"</strong> button.<br>
<br>
Continue until the classification is completed by inserting all necessary prefixes and suffixes, starting again from step ⑥.
</p>
<br clear="all"><br>

<p>
  <a href="../assets/wrb_05.webp" target="_blank">
    <img src="../assets/wrb_05.webp"
         alt="Fig.1" align="left" width="500">
  </a>
Click the <strong>"Save child layer edits"</strong> button ⑪ to save the edits, and then the <strong>"Toggle editing mode for child layer"</strong> button ⑫ to stop editing.
</p>
<br clear="all"><br>


## Creating a new WRB Qualifier Group Type

We could not complete the classification in the example because the **"dropdown menu"** in the **"Qualifier"** field did not contain **"WRB 2006 S Skeletic"**, so we need to create it.

<p>
  <a href="../assets/wrb_06.webp" target="_blank">
    <img src="../assets/wrb_06.webp"
         alt="Fig.1" align="left" width="500">
  </a>

Right-click on the <strong>"Layers"</strong> panel on the <strong>"wrbqualifiergrouptype"</strong> entry ① and from the menu, select <strong>"Open Attribute Table"</strong>. ②
</p>
<br clear="all"><br>

<p>
  <a href="../assets/wrb_07.webp" target="_blank">
    <img src="../assets/wrb_07.webp"
         alt="Fig.1" align="left" width="500">
  </a>
In the <strong>"WRB Qualifier Group Type"</strong> window, click the first icon on the left named <strong>"Toggle Edit Mode"</strong>,③  and then click the <strong>"Add Feature"</strong> icon. ④<br>
<br>
Fill in the fields.<br>
<br>
It is also possible to define up to two specifiers.<br>
<br>
Click the <strong>"Save Layer Edits"</strong> button ⑤ to save the edits, and then the <strong>"Toggle Editing"</strong> button ③ to stop editing.
</p>
<br clear="all"><br>


> [!NOTE]
> Selecting the WRB year will modify the values in the underlying selectors, providing the relevant options for the chosen classification type.

<p>
  <a href="../assets/wrb_08.webp" target="_blank">
    <img src="../assets/wrb_08.webp"
         alt="Fig.1" align="left" width="500">
  </a>
You can now complete the classification by adding the newly created suffix ⑥ by returning to the <strong>"Soil Profile"</strong> form.
</p>
<br clear="all"><br>








