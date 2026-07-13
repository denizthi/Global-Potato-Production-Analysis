# Capstone Project, Global Potato Production Trends

This project analyzes global potato production trends using FAO data and highlights the key producers and patterns that shape agricultural supply chains. The findings can support decision making for stakeholders such as policy makers, agribusiness firms, and investors.

\---

## Project overview

Potatoes are a staple crop with strategic importance for food security, trade, and industry. This capstone project explores how global potato production has evolved over time, which countries drive the trends, and how production is distributed and concentrated across the world.

The analysis combines SQL for data preparation and Microsoft Power BI and Excel for visualization, to answer a series of concrete analytical questions about global potato production.

\---

## Data and scope

* **Data source**, potato production data from the Food and Agriculture Organization of the United Nations (FAO); Provided by FactSet
* **Geographic scope**, Global coverage at country level
* **Time frame**, Multiple years, as provided in the FAO dataset
* **Unit of analysis**

  * Production levels per country per year
  * Yields and volatility metrics
  * Shares of global production and growth dynamics

The original FAO database is queried and cleaned using SQL, including consolidation of duplicated entities such as China and related territories to ensure consistency and avoid double counting.

\---

## Tools and methods

### SQL

Used to

* Connect to and query the FAO database
* Clean and standardize country level data, especially duplicated or overlapping entities
* Apply filters for time periods, crops, and countries
* Aggregate production and yield data to answer each analytical question
* Export the results as structured tables for further visualization

The repository includes the SQL logic in a dedicated file

* `SQL Queries.md`

and pre prepared query result tables as CSV files for each question such as

* `Q1 Mapping the Giants of Production.csv`
* `Q2 Yield Champions Efficiency Over Volume.csv`
* `Q3 How Has Global Production Shifted Through Time.csv`
* `Q4 Yield Volatility Who Plays the Risky Game.csv`
* `Q5 A Tale of Inequality Distribution of Global Output.csv`
* `Q5 A Tale of Inequality Distribution of Global Output all.csv`
* `Q6 Spotlight A Country’s Journey Over the Years.csv`
* `Q7 What Do We Actually Track About Potatoes.csv`
* `Q8 Breakout Growth Stories Year-on-Year Jumps.csv`
* `Q9 Share of the Global Pie.csv`
* `Q10 Smoothing the Signal 3-Year Rolling Averages.csv`
* `Q11 Yearly Champions Who Led Each Year .csv`
* `Q12 Who Improved the Most Over 5 Years.csv`

### Power BI

Used to

* Import the cleaned SQL outputs
* Build interactive dashboards that explore production levels, yields, volatility, and country trajectories
* Visualize global and regional patterns and highlight top and bottom performers

If you have the Power BI report file (`.pbix`), you can open it locally in Microsoft Power BI Desktop to explore the interactive views.

### Microsoft Excel

Used to

* Create basic charts from SQL outputs where lightweight exploration is sufficient
* Perform quick checks on data transformations
* Support simple visual comparisons and summaries

\---

## Analytical questions covered

The analysis is organized around a set of guiding questions. Each question has a corresponding SQL query and an output dataset, and in many cases a visualization.

Examples include

* **Q1, Mapping the giants of production**, Which countries dominate global potato production
* **Q2, Yield champions**, Which countries achieve the highest yields relative to volume
* **Q3, Shifts in global production over time**, How has the geographic distribution of production changed
* **Q4, Yield volatility**, Which producers have the most volatile yields over time
* **Q5, Inequality in global output**, How concentrated is global potato production
* **Q6, Country spotlight**, How has a selected country’s production evolved
* **Q7, What do we actually track**, Which dimensions and indicators are available in the FAO data
* **Q8, Breakout growth stories**, Which countries show standout year on year jumps
* **Q9, Share of the global pie**, How each country’s share of global production changes
* **Q10, Smoothing the signal**, Trends using three year rolling averages
* **Q11, Yearly champions**, Which country led global production in each year
* **Q12, Who improved the most**, Which producers improved most over a five year horizon

Each CSV file in the repository corresponds to one of these questions and contains the result set produced by the SQL queries.

\---


## How to use this repository

This repository is intended for

* Reviewing the SQL logic used to query and clean FAO potato data
* Inspecting prepared datasets for each analytical question
* Reusing the data in your own BI tools or analyses
* Understanding the structure of the Power BI and Excel based visualizations

You can

1. Download the CSV files from the `data` folder and load them into your own tools, such as Power BI, Excel, R, or Python.
2. Use `SQL Queries.md` as a reference for how the FAO database was filtered, cleaned, and aggregated.
3. Refer to the PNG files in `figures` to quickly see the key visual findings without opening the BI tools.

\---

## Reproducibility notes

* The original FAO data is not bundled in this repository, but cleaned data. To fully reproduce the pipeline, download the relevant potato production data from FAO.
* Country naming, such as different entries for China and territories, has been harmonized in SQL. If you replicate the work, apply similar standardization.
* Date ranges and filters should match those used in the SQL queries described in `SQL Queries.md`.

\---

## Acknowledgements

The data sets and original task description come from Kiron / Thrive 2.0 collaboration with FactSet for this SQL Project work. The project structure and wording have been adapted for public sharing as a portfolio piece.

\---

## Author

**Deniz**

For questions you can reach me at `denizyi@gmail.com`.

