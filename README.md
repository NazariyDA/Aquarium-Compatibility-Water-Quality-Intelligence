# 🐠 Aquarium Compatibility & Water Quality Intelligence
Dive into the world of environmental monitoring and hydrochemistry with an interactive Power BI dashboard that transforms thousands of complex water chemical tests into intuitive analytical stories. This project uncovers critical insights into species survivability, environmental parameter synergy, and optimal bioload distribution across aquatic ecosystems.

## 🎯 Objectives of this project
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

## 📊 Tech Stack & Architecture

The project follows a classic corporate analytical solution architecture (DWH/BI). All heavy data transformation, initial data cleansing, and mathematical range comparisons were performed at the **SQL** level. This approach ensured maximum performance and a highly optimized, lightweight data model within **Power BI**.

### 🛠️ SQL Stage: Cleansing, Transformation, and Logical Matching
The raw flat database was normalized and cleansed using optimized SQL queries:

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Filling Data Gaps (Mean Imputation):** Missing cells (`NULL` and empty text strings `''`) in critical columns (`ph`, `Sulfate`, `Trihalomethanes`) were programmatically detected using `NULLIF(TRIM(), '')` structures and forced to be replaced with the exact arithmetic mean values across the entire dataset.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Metric Normalization:** The total water hardness indicator, `Hardness` (originally provided in ppm/mg/L), was mathematically converted into German degrees of hardness (**dGH**) using the formula `Hardness / 17.84` and rounded to 2 decimal places to fully align with the aquarium directory.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Generating a Many-to-Many Fact Table:** Since a single water sample can suit multiple species, and a single species can live in thousands of different samples, a unique `water_id` field was generated in SQL to create a **Compatibility Fact Table `(fact_compatibility)`** containing 107k+ rows. The merge was performed using non-equi JOIN logic: `water_ph BETWEEN pH_Min AND pH_Max AND water_dgh BETWEEN gh_Min AND gh_Max.` This shifted the heavy computation of overlapping ranges onto the database backend.

### 📐 Power BI Stage: Data Modeling and Presentation Layer

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Data Model:** A canonical **Star Schema** data model was established in the modeling view. Two dimension tables (`dim_fish_directory` and `dim_water_parameters`) manage the central fact table (`fact_compatibility`) through single-directional 1-to-many (`1 to *`) relationships.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Interface Optimization (DAX Modeling):** To resolve the circular dependency bug and set the correct experience-based sorting logic within the slicers, an independent lookup table called `Difficulty_Lookup` was generated using DAX. The text tiles were forced into their true biological order: *Beginner ➡️ Intermediate ➡️ Advanced*.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **Advanced DAX:** All analytical indicators are computed using dynamic measures grouped inside an isolated `_Measures` folder. The `CALCULATE` function was utilized to enforce cross-filtering across the Many-to-Many context, allowing for the calculation of exact average toxin levels specifically for the compatible water. Conditional logic was written for dynamic text alerts (`Chloramines Warning`).

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/58c263c3-4d25-4ef9-b06e-951fea48eba1" /> **UI/UX & Cross-Page Filtering:** Synced slicers were configured to pass filters between pages. The **Drill-through** feature was integrated: right-clicking on a specific species navigates the user to its corresponding chemical environment analysis, while right-clicking on an anomalous water sample on the Scatter Chart returns the user to the first page, filtering the matrix to show only the species capable of surviving in that specific water sample.
















