
# QGIS Model Designer — Quick Start (Open & Load Models)

This short guide explains what the **Model Designer** (a.k.a. Graphical Modeler) is in QGIS 3.x and how to **open** it and **load** existing models (`.model3`) for editing or execution. It is intended for technical documentation on GitHub.

---

## What is the Model Designer?
The Model Designer lets you **chain multiple Processing algorithms** into a single reusable workflow (a *model*). No matter how many steps are involved, a model can be **executed as one algorithm**, which improves repeatability and saves time.  

---

## Open the Model Designer
- In QGIS, go to **`Processing → Model Designer`** (the UI may also label it *Graphical Modeler* in some docs).  
- The window presents a **canvas** (to design the workflow), an **Inputs** panel (to define model parameters), and an **Algorithms** panel (to add tools to the chain).  

> Tip: The Model Designer window also offers commands like **Validate**, **Run**, **Export**, and **Help**, available from the **Model** menu and toolbar.  

---

## Load an existing model
You can work with saved QGIS models stored as **`.model3`** files:

1. **Open for edit or execution**  
   In Model Designer, choose **`Model → Open Model…`** (or press **Ctrl+O**) and select your `.model3` file to edit or run it.  

2. **Add a model to the Processing Toolbox**  
   In the **Processing Toolbox**, use **Add Model to Toolbox** so the model appears under **Algorithms → Models** and can be called like any other algorithm (including as a step inside other models). citeturn49search52

3. **(Optional) Embed the model in the project**  
   Use **`Model → Save Model in project`** to embed it in the current `.qgz`, making the model available when sharing the project file.  



