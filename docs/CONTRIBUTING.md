# Project Onboarding: Vision, Data & Power BI Setup

Welcome to the team, everyone! Before we dive into building dashboards, here is a quick overview of what our project is about, where our data lives, and how we are going to connect to it without breaking the Power BI file.

---

## 1. The Project Vision & Our Data

Our project analyzes the global transition towards a modern, sustainable future. We are exploring the intersection of three main pillars: **Macroeconomics**, **Digital Infrastructure**, and **Green Finance**. To do this, we have collected a massive, high-quality database from 4 top-tier global institutions:

* **The World Bank (WB):** Macro indicators (GDP, Population), Green House Gas emissions, and Innovation (Patents, High-Tech Exports).
* **International Monetary Fund (IMF):** Financial flows, including Foreign Direct Investment (FDI), Green Bonds, and Government Environmental Expenses.
* **Eurostat:** Regional data on e-waste, IT specialists, and electric vehicle infrastructure.
* **Int. Telecommunication Union (ITU):** Global digital connectivity (Broadband, Mobile subs, Telecom investments).

---

## 2. The OneDrive Architecture

If you check our shared OneDrive folder, you will see exactly 15 CSV files.

* **1 Dimension Table:** `Dict_Countries.csv` (Our master list of countries, regions, and income groups).
* **14 Fact Tables:** The raw data files mentioned above.

**Why are they in OneDrive?** OneDrive acts as our "Data Lake" (Single Source of Truth). Instead of everyone downloading random files to their desktop, we all sync this one folder. If we need to update a dataset later, we just replace the file in OneDrive, and everyone's dashboards will update automatically.

---

## 3. How to Connect Your Power BI (The Magic Step)

I have already built the initial architecture. **Do NOT import the CSV files manually.** Instead, I have set up a dynamic parameter. Here is how you connect your local Power BI to our shared data in 3 steps:

### **Step 1: Sync the Folder**
Make sure you have the shared project folder synced to your local computer via the OneDrive app. Open your normal Windows File Explorer, find the folder, click the address bar at the top, and copy the folder path (e.g., `C:\Users\Your Name\OneDrive - novaims.unl.pt\PowerBI Project`).

### **Step 2: Open Power BI & Find the Parameter**
* Open our `.pbix` project file.
* Go to the **Home** tab and click **Transform Data** (this opens Power Query).
* On the left panel, you will see a parameter named `FolderPath`. Click on it.

### **Step 3: Update and Refresh**
* In the main window, paste your copied folder path into the **Current Value** box.  
  *(Important: Make sure there is no backslash `\` at the very end of your path)*.
* Hit **Enter**, then click the **Refresh Preview** button at the top.
* Click **Close & Apply** (top left).

> **Boom!** All 15 datasets will instantly connect and load into your computer.

---

## Team Work Distribution (Power BI Project)

> **Project Architecture Note:** The `FolderPath` parameter is already set up. Each team member will receive the `.pbix` file, update the `FolderPath` parameter to their local OneDrive directory, and click "Refresh". All datasets will connect automatically.

### **Role 1: Team Lead & Data Architect**
* **Workload:** Lighter on raw dataset cleaning; focused on project structure, data modeling, and macro-indicators.
* **Assigned Datasets (3):** `Dict_Countries` (Dimension table), `WB_GDP`, `WB_Population`.
* **Key Responsibilities:**
  * **ETL:** Clean the macro datasets (Unpivot, standardizing columns). Ensure the `Dict_Countries` table is perfectly formatted (Country Name, Country Code, Region, Income Group).
  * **Data Modeling:** Build the "Star Schema" (Схема Звезда). Connect all 14 Fact tables from the team to the central `Dict_Countries` Dimension table using a 1-to-Many relationship.
  * **Report Architecture:** Set up the global theme, colors, and navigation buttons for the Power BI report.
  * **Assembly:** Merge everyone's dashboard pages into the final file.

### **Role 2: Financial & Environmental Analyst**
* **Workload:** Standard. Focused on green economy, state spending, and direct investments.
* **Assigned Datasets (4):** `IMF_FDI`, `IMF_GreenBonds`, `IMF_Gov_Expenses (COFOG)`, `WB_GHG_Emissions`.
* **Key Responsibilities:**
  * **ETL:** Remove top rows (for WB), promote headers, and perform "Unpivot Other Columns" on all 4 datasets to create Year and Value columns. Ensure data types are correct (Whole Number/Decimal).
  * **DAX:** Create measures for Financial KPIs (e.g., Total Green Bonds issued, YoY growth in FDI, Gov Expenses as a % of GDP).
  * **Visualization:** Design and build the "Green Economy & Finance" dashboard page.

### **Role 3: Innovation & Tech Infrastructure Analyst**
* **Workload:** Standard. Focused on technological progress and advanced infrastructure.
* **Assigned Datasets (4):** `WB_Patents`, `WB_HighTech_Exports`, `Eurostat_ICT_Specialists`, `Eurostat_EV_Charging`.
* **Key Responsibilities:**
  * **ETL:** Standard clean-up (Remove rows, promote headers, Unpivot). Pay attention to Eurostat datasets, which might require filtering out empty rows.
  * **DAX:** Calculate innovation metrics (e.g., Patents per 1M population, High-Tech Exports as a % of total GDP).
  * **Visualization:** Design and build the "Innovation & Infrastructure" dashboard page.

### **Role 4: Digital Connectivity Analyst**
* **Workload:** Standard. Focused on telecommunications and the digital footprint.
* **Assigned Datasets (4):** `ITU_Telecom_Investments`, `ITU_Broadband_Subs`, `ITU_Mobile_Subs`, `Eurostat_EWaste`.
* **Key Responsibilities:**
  * **ETL:** Standard clean-up (Unpivot). **Crucial Task:** The ITU datasets have values in scientific notation with dollar signs (e.g., `$1.99e+006`). You must clean this column (Replace values to remove `$`, then convert to Decimal Number) so Power BI recognizes them as numbers.
  * **DAX:** Calculate connectivity metrics (e.g., Mobile Subs per 100 people, Total E-Waste generated vs. Telecom Investments).
  * **Visualization:** Design and build the "Digital Society & Connectivity" dashboard page.

---

## Project Timeline & Deadlines

To make sure we hit our final submission without any all-nighters, and to leave enough time to assemble and debug the final architecture, here is our internal schedule:

### **Phase 1: Data Cleaning (ETL) & DAX Measures**
* **Deadline:** This Sunday (April 26), End of Day.
* **Your Task:** Open your personal `.pbix` file, connect it to our OneDrive folder (using the `FolderPath` parameter), clean your assigned datasets (Unpivot, fix data types), and write your basic DAX formulas.
* **Next step:** Save your file as `P3_I_YourName.pbix` and upload it to the `Team_Drafts` folder on OneDrive.
* *(I will use Mon/Tue to review the data, fix any bugs, and build the central Star Schema).*

### **Phase 2: Dashboards & Visualizations**
* **Deadline:** Next Sunday (May 3), End of Day.
* **Your Task:** Once the Star Schema is ready, you will build your assigned dashboard page (charts, colors, slicers).
* **Next step:** Re-upload your updated file to the `Team_Drafts` folder.
* *(I will use Mon/Tue to merge all your pages into one final app and set up cross-filtering).*

### **Phase 3: Final Review**
* **Timeline:** May 6-7.
* **Our Task:** We will do a quick team sync to click through the final dashboard, ensure no filters are broken, and prep our methodology notes for the professors.

---

## 💡 Tip for Role 4: Data Cleaning Guide (ITU Datasets)

**To the Digital Connectivity Analyst:** The ITU datasets have a specific formatting issue. The values are currently stored as text with a dollar sign and scientific notation (e.g., `$1.99667e+006`). If we leave them like this, Power BI will not be able to sum them up or build charts.

Here is how to clean this column in 5 quick steps (do this after you have performed the **Unpivot** step):

1. Right-click on the header of the **Value** column (where the messy numbers are).
2. Select **Replace Values...** from the dropdown menu.
3. In the **Value to find** box, type a single dollar sign: `$`
4. Leave the **Replace with** box completely empty and click **OK**. *(This removes the dollar signs)*.
5. Finally, click on the data type icon (it probably says `ABC`) on the left side of the **Value** column header and select **Decimal Number**.

> Power BI will instantly convert the scientific notation into standard numbers (e.g., `$1.99667e+006` becomes `1996670`). Click **Close & Apply** when you are done!