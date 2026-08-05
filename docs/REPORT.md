# World Bank Report: SDG Goals 2026
## Project Technical Documentation & Written Report

| Role | Name & ID |
| :--- | :--- |
| **Group Name** | P3_I |
| **Team Leader** | Amir Namatov - 20251793 |
| **Member (Green Economy)** | Maksim Blazevic - 20251816 |
| **Member (Digital Society)** | Joaquim Inacio - 20241815 |
| **Member (Innovation)** | Tanjina Sumaiya - 20241404 |

---

## 1. Project Context and Objectives

The objective of this project is to provide the World Bank with a comprehensive, interactive Business Intelligence solution capable of analysing global trends across 12 distinct analytical areas. These areas are strictly aligned with the United Nations Sustainable Development Goals (SDGs) for 2026, encompassing macroeconomic stability, environmental mitigation, digital society expansion, and technological innovation. The project synthesizes 14 distinct datasets from four primary international organizations (World Bank, IMF, Eurostat, and ITU) into a unified semantic model, allowing policymakers to derive insights across previously isolated domains.

---

## 2. Options Taken and Technologies Used

To ensure a robust, scalable, and replicable solution, the following technological decisions were made:

* **Power BI Desktop:** Selected as the primary analytical engine due to its advanced semantic modelling capabilities and native support for the Star Schema architecture, which is critical for handling disparate temporal and geographical granularities.
* **Power Query (M):** Utilized as the primary Extract, Transform, Load (ETL) tool. It provides a highly reproducible transformation pipeline, which is essential for updating the report with future datasets without manual intervention.
* **Data Analysis Expressions (DAX):** Chosen for developing explicit, dynamic measures. Implicit aggregations were strictly avoided to maintain analytical precision and to enable complex context transitions (e.g., historical YoY comparisons).

---

## 3. Data Modelling and Architectural Diagram

The foundation of the report is a highly optimized Star Schema. This architectural choice prevents many-to-many relationship ambiguities and ensures that cross filtering propagates correctly across all 14 fact tables.

* **Central Dimension Tables:** The model relies on two primary dictionaries. `Dict_Countries` standardizes geographical entities (Country Code, Country Name, Region, Income Group). During data preparation, non-sovereign macro-aggregates (e.g., "World", "IDA total") were explicitly filtered out of this dictionary to prevent double counting in global totals. `Dict_Years` serves as a continuous date table, enabling time-series synchronization across all domains.
* **Fact Tables:** Each of the 14 operational datasets (e.g., `WB_GDP`, `IMF_GreenBonds`, `Eurostat_EWaste`) acts as a fact table connected to the dimension tables via active, single-direction, 1-to-Many relationships.

---

## 4. ETL Processes: M Queries Explanation

To ensure the project is fully replicable with updated datasets, specific transformation strategies were implemented in Power Query (M):

### 4.1. Directory Parameterization

To prevent hard-coded path failures when sharing the project, a parameter named `SourcePath` was established. All M queries point to this parameter, allowing the entire data source location to be updated centrally.

```powerquery
let
    Source = Excel.Workbook(File.Contents(SourcePath & "\WB_Population.xlsx"), null, true)
...
```

### 4.2. Data Cleansing and Anomaly Handling

Raw datasets, particularly from the World Bank, frequently contain string placeholders (such as `".."` ) for missing data. Attempting to convert these columns to numeric types directly causes calculation errors in DAX. During the ETL process in Power Query Editor, explicit data cleansing steps were applied to find and replace these text anomalies with null values. Only after this sanitization were the columns successfully converted to 'Whole Number' types, ensuring the integrity of the Star Schema.

### 4.3. Structural Normalization (Unpivoting)

Multiple source files provided time-series data horizontally (years as columns). To conform to the Star Schema and enable relational filtering, the `Table.UnpivotOtherColumns` function was utilized extensively to transform wide tables into a normalized "Year" and "Value" format.

---

## 5. Analytical Engine: DAX Measurement Explanation

Explicit DAX measures were engineered to support the 12 analytical areas. The logic is categorized below by design pattern to illustrate the analytical depth required for the report:

### 5.1. Base Aggregations

These measures serve as the foundational numerical calculations for specific indicators across our 14 fact tables.

```dax
Total Broadband Subs = SUM (ITU_Broadband_Subs [Value])

Total E-Waste = SUM (Eurostat_EWaste [E-Waste Amount (kg per capita)])

Total EV Charging = SUM(Eurostat_EV_Charging [EV_Charging_Points])

Total FDI Inward (USD Millions) = SUM(IMF_FDI[FDI_Inward_USD_Millions])

Total GDP = SUM(WB_GDP[Value_GDP])

Total GHG Emissions (MtCO2e) = SUM(WB_GHG_Emissions[GHG_Emissions_MtCO2e])

Total Gov Env Expenditure (USD) = SUM(IMF_Gov_Expenses[Gov_Env_Expenditure_USD])

Total Green Bonds Issued (USD) = SUM(IMF_GreenBonds[GreenBonds_Issued_USD])

Total High-Tech Exports = SUM(WB_HighTech_Exports[HighTech_Exports_USD])

Total Mobile Subs = SUM(ITU_Mobile_Subs[Value])

Total Patents = SUM(WB_Patents[Total_Patents])

Total Population = SUM(WB_Population[Value_Pop])

Total Telecom Investments = SUM(ITU_Telecom_Investments[Value (millions of local currency)])
```

### 5.2. Time Intelligence and Variable-Based Calculations

Instead of relying on basic built-in time intelligence, advanced DAX utilizing variables (`VAR`) and context manipulation (`SELECTEDVALUE`) was employed to calculate dynamic Year-over-Year (YoY) trajectories. This approach optimizes performance and handles custom integer year columns flawlessly.

**FDI YoY Growth %:**
```dax
FDI YoY Growth % = 
VAR CurrentYear = SELECTEDVALUE(IMF_FDI[Year])
VAR CurrentValue = [Total FDI Inward (USD Millions)]
VAR PreviousValue = CALCULATE(
    [Total FDI Inward (USD Millions)],
    IMF_FDI[Year] = CurrentYear - 1
)
RETURN DIVIDE(CurrentValue - PreviousValue, PreviousValue)
```

**Green Bonds YoY Growth %:**
```dax
Green Bonds YoY Growth % = 
VAR CurrentYear = SELECTEDVALUE(IMF_GreenBonds[Year])
VAR CurrentValue = [Total Green Bonds Issued (USD)]
VAR PreviousValue = CALCULATE(
    [Total Green Bonds Issued (USD)],
    IMF_GreenBonds[Year] = CurrentYear - 1
)
RETURN DIVIDE(CurrentValue - PreviousValue, PreviousValue)
```

**GHG YoY Change (MtCO2e):**
```dax
GHG YoY Change (MtCO2e) = 
VAR CurrentYear = SELECTEDVALUE(WB_GHG_Emissions[Year])
VAR CurrentValue = [Total GHG Emissions (MtCO2e)]
VAR PreviousValue = CALCULATE(
    [Total GHG Emissions (MtCO2e)],
    WB_GHG_Emissions[Year] = CurrentYear - 1
)
RETURN CurrentValue - PreviousValue
```

### 5.3. Cross-Domain Comparative Ratios

To align with SDG reporting, several measures synthesize data across entirely different fact tables to provide multi-dimensional insights.

```dax
GDP per Capita = DIVIDE([Total GDP], [Total Population], 0)

Mobile Subs per 100 = DIVIDE([Total Mobile Subs], [Total Population], 0) * 100

Gov Expenditure vs Green Bonds Ratio = DIVIDE([Total Gov Env Expenditure (USD)], [Total Green Bonds Issued (USD)])

Subs per Local Currency Invested = DIVIDE(SUM('ITU_Mobile_Subs'[Value]), SUM('ITU_Telecom_Investments'[Value (millions of local currency)]))

E-Waste to Investment Ratio = DIVIDE(SUM('Eurostat_EWaste'[E-Waste Amount (kg per capita)]), SUM('ITU_Telecom_Investments'[Value (millions of local currency)]))
```

*(Cross-domain measure combining three different fact datasets):*
```dax
E-Waste per Subscription = DIVIDE(
    SUM('Eurostat_EWaste'[E-Waste Amount (kg per capita)]),
    SUM('ITU_Mobile_Subs'[Value]) + SUM('ITU_Broadband_Subs'[Value])
)
```

### 5.4. Advanced Context Filtering and Statistical Functions

Specific analytical requirements dictated the use of context-altering functions (`CALCULATE`, `ALL`) and averages.

* **Top Emitting Country:** *(Dynamically identifies the highest polluter by explicitly overriding the visual context)*
```dax
Top Emitting Country = CALCULATE(
    SELECTEDVALUE(WB_GHG_Emissions[Country Name]),
    TOPN(1, ALL(WB_GHG_Emissions[Country Name]), [Total GHG Emissions (MtCO2e)])
)
```

* **Average ICT Specialists:** *(Utilizes an average rather than a sum to accurately represent the specific statistical methodology of the dataset)*
```dax
Average ICT Specialists (000s) = AVERAGE(Eurostat_ICT_Specialists[ICT_Specialists_Pct])
```

---

## 6. Conclusion

The delivered semantic model successfully bridges the gap between raw, disconnected institutional data and actionable global insights. The rigorous application of Star Schema principles, coupled with parameterized ETL routines and explicitly coded analytical measures, ensures that the World Bank possesses a scalable, fully replicable tool for monitoring SDG progress towards 2026.

---

## 7. References

* **Eurostat.** (2024). *Information society, science and technology datasets*. European Commission. Retrieved from https://ec.europa.eu/eurostat/data/database 
* **International Monetary Fund (IMF).** (2024). *Climate Change Indicators Dashboard*. Retrieved from https://climatedata.imf.org/ 
* **International Telecommunication Union (ITU).** (2024). *World Telecommunication/ICT Indicators Database*. Retrieved from https://www.itu.int/en/ITU-D/Statistics/ 
* **World Bank.** (2024). *World Development Indicators Open Data*. The World Bank Group. Retrieved from https://data.worldbank.org/
