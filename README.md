# <img width="35" height="35" alt="aquarium32" src="https://github.com/user-attachments/assets/15805268-2174-4764-9f56-2912cba62a5b" /> Aquarium Compatibility & Water Quality Intelligence
Dive into the world of environmental monitoring and hydrochemistry with an interactive Power BI dashboard that transforms thousands of complex water chemical tests into intuitive analytical stories. This project uncovers critical insights into species survivability, environmental parameter synergy, and optimal bioload distribution across aquatic ecosystems.

## <img width="30" height="30" alt="fish orange" src="https://github.com/user-attachments/assets/42d4a742-61f8-487d-863c-00de840268fa" /> Objectives of this project
**Exploring Environmental Patterns:** Analyzing the relationship between water chemical composition (pH acidity, dGH hardness) and the specific biological requirements of over 70 species of aquarium inhabitants to automate the eco-matching process.

**Developing an End-to-End Solution:** Demonstrating the full data lifecycle (ETL/ELT) — from importing raw, flat CSV files, through data cleaning and transformation in SQL, to designing a relational data model and building an interactive dashboard.

**Analytical Approach to Fishkeeping:** Identifying the most resilient and delicate species, uncovering hidden hydrochemical paradoxes (such as the mismatch between human drinking water and fish requirements), and assessing the risks of toxic substances (sulfates and chloramines) on ecosystem bioloads.

## 🗒 Data Overview

**Data Volume & Source Databases:** The project is built on the combination of two independent data sources:
* **Water Quality Database:** A database consisting of **3,276 unique environmental water samples (tests)**, each containing 9 unique chemical and physical parameters.

* **Aquarium Directory:** A database containing **75 different species of aquarium inhabitants** along with their individual environmental requirements.

**Compatibility Scale:** Through analytical data modeling in SQL, **107,739 unique instances of successful compatibility** were generated and analyzed between the water source parameters and the species' tolerance ranges.

**Key Performance Indicators (KPIs) Displayed on the Dashboard:**
* **Total Species:** The total number of fish and invertebrate species (such as snails and shrimps) available in our database.

* **Compatible Sources:** A dynamic count of water sources that perfectly match the biological requirements of the selected segment or a specific species.

* **Human Potability** Rate: The percentage of water sources officially classified as safe for human consumption (39.01% in this database).

* **Avg Safe pH & dGH:** The average water acidity and hardness levels, calculated strictly within the safe tolerance zones for the selected ecosystems.

<img width="818" height="456" alt="Page 1" src="https://github.com/user-attachments/assets/1a799c19-63b6-4edf-b413-2141468e8d27" />


<img width="818" height="456" alt="page 2" src="https://github.com/user-attachments/assets/0887e19a-ec84-4da3-a7b8-1de0c85c7532" />


## 📊 Tech Stack & Architecture

The project follows a classic corporate analytical solution architecture (DWH/BI). All heavy data transformation, initial data cleansing, and mathematical range comparisons were performed at the **SQL** level. This approach ensured maximum performance and a highly optimized, lightweight data model within **Power BI**.

### 🛠️ SQL Stage: Cleansing, Transformation, and Logical Matching
The raw flat database was normalized and cleansed using optimized SQL queries:

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Filling Data Gaps (Mean Imputation):** Missing cells (`NULL` and empty text strings `''`) in critical columns (`ph`, `Sulfate`, `Trihalomethanes`) were programmatically detected using `NULLIF(TRIM(), '')` structures and forced to be replaced with the exact arithmetic mean values across the entire dataset.

<details>
  <summary>📄 SQL Query (Click to expand)</summary>
  
  ```sql
WITH prepared_water AS (
    SELECT 
        NULLIF(TRIM(CAST(ph AS VARCHAR)), '') AS ph_clean,
        NULLIF(TRIM(CAST(Sulfate AS VARCHAR)), '') AS sulfate_clean,
        NULLIF(TRIM(CAST(Trihalomethanes AS VARCHAR)), '') AS trihalomethanes_clean,
        Hardness, Solids, Chloramines, Conductivity, Organic_carbon, Turbidity, Potability
    FROM water_potability
),
averages AS (
    SELECT 
        AVG(CAST(ph_clean AS DOUBLE PRECISION)) AS avg_ph,
        AVG(CAST(sulfate_clean AS DOUBLE PRECISION)) AS avg_sulfate,
        AVG(CAST(trihalomethanes_clean AS DOUBLE PRECISION)) AS avg_trihalomethanes
    FROM prepared_water
)
SELECT 
    ROW_NUMBER() OVER () AS water_id,
    ROUND(
        COALESCE(CAST(p.ph_clean AS DOUBLE PRECISION), a.avg_ph), 
        2
    ) AS water_ph,
    ROUND((CAST(p.Hardness AS DOUBLE PRECISION) / 17.84), 2) AS water_dgh,
    ROUND(
        COALESCE(CAST(p.sulfate_clean AS DOUBLE PRECISION), a.avg_sulfate), 
        2
    ) AS Sulfate,
    ROUND(
        COALESCE(CAST(p.trihalomethanes_clean AS DOUBLE PRECISION), a.avg_trihalomethanes), 
        2
    ) AS Trihalomethanes,
    p.Solids, 
    p.Chloramines, 
    p.Conductivity, 
    p.Organic_carbon, 
    p.Turbidity, 
    p.Potability
FROM prepared_water p
CROSS JOIN averages a;
```

</details>


<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Metric Normalization:** The total water hardness indicator, `Hardness` (originally provided in ppm/mg/L), was mathematically converted into German degrees of hardness (**dGH**) using the formula `Hardness / 17.84` and rounded to 2 decimal places to fully align with the aquarium directory.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Generating a Many-to-Many Fact Table:** Since a single water sample can suit multiple species, and a single species can live in thousands of different samples, a unique `water_id` field was generated in SQL to create a **Compatibility Fact Table `(fact_compatibility)`** containing 107k+ rows. The merge was performed using non-equi JOIN logic: `water_ph BETWEEN pH_Min AND pH_Max AND water_dgh BETWEEN gh_Min AND gh_Max.` This shifted the heavy computation of overlapping ranges onto the database backend.

<details>
  <summary>📄 SQL Query (Click to expand)</summary>
  
  ```sql
WITH cleaned_water AS (
    SELECT 
        ROW_NUMBER() OVER () AS water_id,
        ROUND(COALESCE(CAST(p.ph_clean AS REAL), a.avg_ph), 2) AS water_ph,
        ROUND((CAST(p.Hardness AS REAL) / 17.84), 2) AS water_dgh
    FROM (
        SELECT 
            NULLIF(TRIM(CAST(ph AS TEXT)), '') AS ph_clean,
            Hardness
        FROM water_potability
    ) p
    CROSS JOIN (
        SELECT AVG(CAST(NULLIF(TRIM(CAST(ph AS TEXT)), '') AS REAL)) AS avg_ph 
        FROM water_potability
    ) a
)
SELECT 
    w.water_id,
    f.Slug AS fish_slug
FROM cleaned_water w
JOIN fish_directory f 
  ON w.water_ph BETWEEN CAST(f."pH Min" AS REAL) AND CAST(f."pH Max" AS REAL)
 AND w.water_dgh BETWEEN CAST(f."gh Min (dGH)" AS REAL) AND CAST(f."gh Max (dGH)" AS REAL);
```

</details>





### <img width="25" height="25" alt="water-splash" src="https://github.com/user-attachments/assets/b48396f0-9c54-4b73-a7a8-62e8cdff2eb3" /> Power BI Stage: Data Modeling and Presentation Layer

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Data Model:** A canonical **Star Schema** data model was established in the modeling view. Two dimension tables (`dim_fish_directory` and `dim_water_parameters`) manage the central fact table (`fact_compatibility`) through single-directional 1-to-many (`1 to *`) relationships.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Interface Optimization (DAX Modeling):** To resolve the circular dependency bug and set the correct experience-based sorting logic within the slicers, an independent lookup table called `Difficulty_Lookup` was generated using DAX. The text tiles were forced into their true biological order: *Beginner ➡️ Intermediate ➡️ Advanced*.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Advanced DAX:** All analytical indicators are computed using dynamic measures grouped inside an isolated `_Measures` folder. The `CALCULATE` function was utilized to enforce cross-filtering across the Many-to-Many context, allowing for the calculation of exact average toxin levels specifically for the compatible water. Conditional logic was written for dynamic text alerts (`Chloramines Warning`).

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **UI/UX & Cross-Page Filtering:** Synced slicers were configured to pass filters between pages. The **Drill-through** feature was integrated: right-clicking on a specific species navigates the user to its corresponding chemical environment analysis, while right-clicking on an anomalous water sample on the Scatter Chart returns the user to the first page, filtering the matrix to show only the species capable of surviving in that specific water sample.

## <img width="30" height="30" alt="star" src="https://github.com/user-attachments/assets/6c69c7cf-8630-4caf-815c-d8bf097caa1f" /> **Insights & Analysis**

* ### Water Quality & Fish Species Compatibility Directory:

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> The analysis clearly separates aquarium inhabitants into two categories: resilient and sensitive.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> `Kribensis` and `Clown loach` proved to be the most resilient species, capable of thriving in nearly **70%** of all analyzed water sources.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> `Discus` and `Cardinal tetra` are the most delicate species, matching with less than **10%** of water sources due to their strict acidity requirements.

* ### Species Distribution by Aquarium Zone:

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> This donut chart helps users distribute fish properly and evenly throughout the aquarium.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> Strictly bottom-dwelling inhabitants (`Bottom`) make up the largest share of the database **(nearly 25%)**, while mid-water fish (`Midwater`) account for **28%**.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> Thanks to dynamic filtering, selecting a specific water type immediately displays the percentage ratio of inhabitants for each water layer, preventing overcrowding at the bottom or surface.

* ### Water Source Chemical Distribution (pH vs. dGH):

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> This scatter plot maps out the complete hydrochemical profile, where each of the **3,276 points** represents an individual water sample.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> Safe human drinking water (`Drinking Water`, highlighted in light blue) forms a dense core in the center of the chart (neutral pH 6.0–8.0).

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> The visual uncovers a major ecological paradox: more than half of the water safe for humans is completely unsuitable for soft-water Amazonian fish.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> The clear vertical line of points down the center demonstrates the high quality of data cleansing — showcasing where missing values were successfully replaced in SQL using the mean pH of 7.08.

* ### Top 10 Species by Sulfate Exposure:

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> This chart identifies the Top 10 inhabitants that, due to their broad tolerance ranges, most frequently end up in water with high chemical loading.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> `Cardinal tetra` tops the ranking, with the average sulfate concentration in its compatible water being the highest at **347.98**.

* ### Top 10 Species with Lowest Chloramine Tolerance:

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> By isolating only the Top 10 critical cases, we can clearly pinpoint the inhabitants exposed to the highest average toxicity levels.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> For the anti-rating leaders — `Betta fish` and `Flying fox` — the average chloramine level exceeds safe thresholds, which automatically triggers the **`⚠️ Dechlorinator Required!`** alert in the main ledger to signal the mandatory use of water conditioners.

* ### Water Clarity & Conductivity Profile by Aquarium Zone:

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> This combo chart compares two vital environmental markers: water turbidity (line) and dissolved solids/mineralization (columns).

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> The analysis confirmed a key biological insight: bottom-dwelling inhabitants (`Bottom`) live in environments with higher natural turbidity (`Turbidity`) compared to top-layer fish.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> Electrical conductivity (`Conductivity`) remains perfectly stable across all zones (around 424–425 µS/cm), indicating highly consistent mineral levels throughout this specific dataset.

## <img width="25" height="25" alt="shell" src="https://github.com/user-attachments/assets/678af2f0-fd12-4141-96fd-86de33d96437" /> **Business Value & Recommendations**
Based on the built dashboard, the following actionable solutions have been developed for both users and businesses (such as aquarium retail stores):

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Smart Matching & Optimization:** Instead of manually studying dozens of reference guides, the dashboard’s algorithm allows users to instantly match the perfect combination of fish to a customer's specific tap water parameters with just a single click.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Minimizing Biological Risks:** The aquarium zone distribution donut chart provides a clear, visual guide for proper, multi-layered fish stocking (surface, midwater, bottom). This reduces territorial stress and aggression among aquarium inhabitants.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Automating Sales (Upselling Strategies):** Thanks to the automated `Chloramines Warning` trigger, aquarium stores can implement data-driven product recommendations. When a customer purchases high-exposure species like `Betta fish` or `Flying fox`, the system prompts the staff to recommend dechlorinators and water conditioners, effectively increasing the average order value.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> Environmental Risk Audit: The end-to-end cross-page Drill-through filter enables ecologists to instantly isolate chemical anomalies on the Scatter Chart and visually identify which aquatic species would be the first to face extinction threats in a specific region.

### Thank you for your interest in this project <img width="25" height="25" alt="fish" src="https://github.com/user-attachments/assets/f9aee3a9-af8b-4492-9651-a75cdcc00b61" />








