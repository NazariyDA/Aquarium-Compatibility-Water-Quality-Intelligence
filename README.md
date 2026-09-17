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





