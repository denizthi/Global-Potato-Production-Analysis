# SQL Queries - Thiel, Deniz

## Capstone Project: The Global Journey of Potatoes





## Schema Reference



&#x20; **Fact Table: fact\_production**



&#x20;   Area\_ID: Foreign key to dim\_country (the country or area)

&#x20;   Item\_ID: Foreign key to dim\_item (the food item being measured)

&#x20;   Element\_ID: Foreign key to dim\_element (the type of measurement)

&#x20;   Year: The year of measurement (e.g., 2005, 2010)

&#x20;   Value: The actual recorded value (in tonnes, kg/ha, etc. depending on the element)



&#x20;Dimension Table: dim\_country



&#x20;   Country\_ID: Unique ID for the country or area

&#x20;   Country: Human-readable country name (e.g., "India", "Brazil")



&#x20;Dimension Table: dim\_item



&#x20;   Item\_ID: Unique ID for the item

&#x20;   Item: Name of the food item (e.g., "Wheat", "Maize (corn)")



&#x20;Dimension Table: dim\_element



&#x20;   Element\_ID: Unique ID for the measurement type

&#x20;   Element: The measurement name. Examples:

&#x20;       'Production' – total amount produced

&#x20;       'Yield' – efficiency of production (e.g., kg per hectare)

&#x20;       'Area harvested' – size of land used

&#x20;       'Stocks' – amount stored or held in reserve

&#x20;   Unit: The unit of measurement (e.g., tonnes, kg/ha, 1000 head, etc.)



## Q1. Mapping the Giants of Production

Which countries lead the charge in growing *Potatoes* on a global scale?

```sql
SELECT dc.Country, SUM(fp.Value) AS Total\_Production, de.Unit
FROM fact\_production AS fp
LEFT JOIN dim\_country AS dc USING(Country\_ID)
LEFT JOIN dim\_item AS di USING(Item\_ID)
LEFT JOIN dim\_element AS de USING(Element\_ID)
WHERE di.Item='Potatoes' 
  AND de.Element ='Production'
  AND dc.Country NOT LIKE 'China,%'
  AND year >= 2000
GROUP BY dc.Country, de.Unit
ORDER BY Total\_Production DESC
LIMIT 10;
```

> Filters applied to limit the results for the last 25 years in order to exclude dissolved countries (USSR etc.) and to prevent duplicated counts of China data. Production amount of country “China” is equal to the sum of “China, mainland” and “China, Taiwan Province of”. The source code of China production can be found at Appendix.
> 

## Q2. Yield Champions: Efficiency Over Volume

Which countries extract the most from each hectare?

```sql
WITH yield\_table AS (
  SELECT dc.Country, 
    ROUND(AVG(fp.Value),0) AS Avg\_Yield, 
    de.Unit, 
    Avg\_Yield/1000 AS Ton\_Avg\_Yield,
    't/ha' AS Unit\_t
  FROM fact\_production AS fp
  LEFT JOIN dim\_country AS dc USING(Country\_ID)
  LEFT JOIN dim\_item AS di USING(Item\_ID)
  LEFT JOIN dim\_element AS de USING(Element\_ID)
  WHERE di.Item='Potatoes' 
    AND de.Element ='Yield'
    AND year >= 2000
  GROUP BY dc.Country, de.Unit
),
area\_table AS (
  SELECT dc.Country,
    ROUND(AVG(fp.Value)) AS Avg\_Area,
    de.Unit
  FROM fact\_production AS fp
  LEFT JOIN dim\_country AS dc USING(Country\_ID)
  LEFT JOIN dim\_item AS di USING(Item\_ID)
  LEFT JOIN dim\_element AS de USING(Element\_ID)
  WHERE di.Item='Potatoes' 
    AND de.Element ='Area harvested'
    AND year >= 2000
  GROUP BY dc.Country, de.Unit
)
SELECT  yield\_table.Country,
        yield\_table.Avg\_Yield,
        yield\_table.Unit,
        yield\_table.Ton\_Avg\_Yield,
        yield\_table.Unit\_t,
        area\_table.Avg\_Area,
        area\_table.Unit
FROM yield\_table
LEFT JOIN area\_table USING(Country)
ORDER BY Ton\_Avg\_Yield DESC
LIMIT 10;
```

> Filter applied to limit the results for the last 25 years in order to exclude dissolved countries (USSR etc.).
> 

## Q3. How Has Global Production Shifted Through Time?

Did wars, climate patterns, or economic booms affect production volumes?

```sql
SELECT fp.Year AS Year, SUM(fp.Value) AS Global\_Production, AVG(fp.Value) As Avg\_Production, de.Unit
FROM fact\_production AS fp
LEFT JOIN dim\_country AS dc USING(Country\_ID)
LEFT JOIN dim\_item AS di USING(Item\_ID)
LEFT JOIN dim\_element AS de USING(Element\_ID)
WHERE di.Item='Potatoes' 
  AND de.Element ='Production'
  AND dc.Country NOT LIKE 'China,%' 
GROUP BY Year, de.Unit
ORDER BY Year;
```

> Filters applied to prevent duplicated counts of China data. Production amount of country “China” is equal to the sum of “China, mainland” and “China, Taiwan Province of”. The source code of China production can be found at Appendix.
> 

## Q4. Yield Volatility: Who Plays the Risky Game?

Which countries show erratic yield swings?

```sql
SELECT  dc.Country, 
        AVG(fp.Value)/1000 AS Ton\_Avg\_Yield,  --Unit:ton
        stddev(fp.Value)/1000 AS Std\_Dev,   --Unit:ton
        CASE WHEN AVG(fp.Value) > 0 THEN stddev(fp.Value)/AVG(fp.Value) 
          ELSE NULL END AS CV
FROM fact\_production AS fp
LEFT JOIN dim\_country AS dc USING(Country\_ID)
LEFT JOIN dim\_item AS di USING(Item\_ID)
LEFT JOIN dim\_element AS de USING(Element\_ID)
WHERE di.Item='Potatoes' 
  AND de.Element ='Yield'
  AND dc.Country NOT LIKE 'China,%'
  AND year >= 2000
GROUP BY dc.Country
ORDER BY CV DESC
LIMIT 10;
```

> Filters applied to prevent duplicated counts of China data. Production amount of country “China” is equal to the sum of “China, mainland” and “China, Taiwan Province of”. The source code of China production can be found at Appendix.
> 

## Q5. A Tale of Inequality: Distribution of Global Output

Is production concentrated or shared globally?

```sql
---Calculation of HHI ---
WITH market\_sh AS (
  SELECT  fp.Year, 
          dc.Country, 
          SUM(fp.Value) AS Country\_Production, 
          SUM(fp.Value) OVER(PARTITION BY fp.Year) AS Global\_Production,
          ROUND(Country\_Production\*100/Global\_Production,0) AS Market\_Share\_Perc,
  FROM fact\_production AS fp
  LEFT JOIN dim\_country AS dc USING(Country\_ID)
  LEFT JOIN dim\_item AS di USING(Item\_ID)
  LEFT JOIN dim\_element AS de USING(Element\_ID)
  WHERE di.Item='Potatoes' 
    AND de.Element ='Production'
    AND dc.Country NOT LIKE 'China,%' 
  GROUP BY fp.Year, dc.Country, fp.Value
  ORDER BY fp.Year, dc.Country)

SELECT  market\_sh.Year, 
        SUM(POWER(market\_sh.Country\_Production/Global\_Production,2)) AS HHI
FROM market\_sh
GROUP BY market\_sh.Year
ORDER BY year;
```

> Yearly HHI index is calculated. Filters applied to prevent duplicated counts of China data. Production amount of country “China” is equal to the sum of “China, mainland” and “China, Taiwan Province of”. The source code of China production can be found at Appendix.
> 

```sql
---Extended Code with Market Share Percentage Calculation---
WITH market\_sh AS (
  SELECT  fp.Year, 
          dc.Country, 
          SUM(fp.Value) AS Country\_Production, 
          SUM(fp.Value) OVER(PARTITION BY fp.Year) AS Global\_Production\_p\_Year,
          ROUND(Country\_Production\*100/Global\_Production\_p\_year,0) AS Market\_Share\_Perc,
  FROM fact\_production AS fp
  LEFT JOIN dim\_country AS dc USING(Country\_ID)
  LEFT JOIN dim\_item AS di USING(Item\_ID)
  LEFT JOIN dim\_element AS de USING(Element\_ID)
  WHERE di.Item='Potatoes' 
    AND de.Element ='Production'
    AND dc.Country NOT LIKE 'China,%' 
  GROUP BY fp.Year, dc.Country, fp.Value
  ORDER BY fp.Year, dc.Country)

SELECT  market\_sh.Year, 
        market\_sh.Country, 
        market\_sh.Country\_Production,
        market\_sh.Global\_Production\_p\_year,
        market\_sh.Market\_share\_Perc,
        SUM(POWER(market\_sh.Country\_Production/Global\_Production\_p\_Year,2)) OVER(PARTITION BY market\_sh.Year) AS HHI
FROM market\_sh
WHERE market\_sh.Market\_share\_Perc >= 5
ORDER BY market\_sh.Year, market\_sh.Market\_share\_Perc DESC;
```

> Beside yearly HHI index, Market Share Percentage of the countries for every year is calculated. Filters applied to prevent duplicated counts of China data. Production amount of country “China” is equal to the sum of “China, mainland” and “China, Taiwan Province of”. The source code of China production can be found at Appendix.
> 

## Q6. Spotlight: Germany’s Journey Over the Years

How has Germany's production changed over time?

```sql
SELECT  fp.Year, 
        dc.Country,
        fp.Value AS Production, 
        de.Unit, 
        Production-LAG(Production) OVER (ORDER BY year) AS YOY, 
        Unit, 
        ROUND(((Production/LAG(Production) OVER (ORDER BY year))-1)\*100,0) AS YOY\_Perc
FROM fact\_production AS fp
  LEFT JOIN dim\_country AS dc
  USING(Country\_ID)
  LEFT JOIN dim\_item AS di
  USING(Item\_ID)
  LEFT JOIN dim\_element AS de
  USING(Element\_ID)
WHERE di.Item = 'Potatoes' 
    AND de.Element = 'Production' 
    AND dc.Country = 'United States of America'
ORDER BY Year;
```

## Q7. What Do We Actually Track About *Potatoes*?

Which kinds of measurements exist for *Potatoes*?

```sql
SELECT dc.Country, di.Item, de.Element, de.Unit
FROM fact\_production AS fp
LEFT JOIN dim\_country AS dc USING(Country\_ID)
LEFT JOIN dim\_item AS di USING(Item\_ID)
LEFT JOIN dim\_element AS de USING(Element\_ID)
WHERE di.Item='Potatoes' 
  AND dc.Country ='United States of America'
GROUP BY dc.Country, di.Item, de.Element, de.Unit;
```

## Q8. Breakout Growth Stories: Year-on-Year Jumps

Where did production suddenly explode?

```sql
---YOY calculation with Countries with Production Volume higher than Median---
WITH yoy\_table AS (
  SELECT  dc.Country,
          fp.Year,
          fp.Value AS Production, 
          de.Unit,
          LAG(Production) OVER (PARTITION BY dc.Country ORDER BY year) AS Prev\_Year\_Production,
          Production-LAG(Production) OVER (PARTITION BY dc.Country ORDER BY year) AS YOY, 
          de.Unit, 
          CASE WHEN LAG(Production) OVER(PARTITION BY dc.Country ORDER BY year) > 0 THEN ROUND(((Production/LAG(Production) OVER (PARTITION BY dc.Country ORDER BY year))-1)\*100,0)
            ELSE NULL END AS YOY\_Perc
  FROM fact\_production AS fp
    LEFT JOIN dim\_country AS dc
    USING(Country\_ID)
    LEFT JOIN dim\_item AS di
    USING(Item\_ID)
    LEFT JOIN dim\_element AS de
    USING(Element\_ID)
  WHERE di.Item = 'Potatoes' 
    AND de.Element = 'Production' 
    AND dc.Country NOT LIKE 'China,%' 
  ORDER BY Country, Year),

median\_table AS (
  SELECT fp.Year, MEDIAN(fp.Value) AS Median
  FROM fact\_production AS fp
  LEFT JOIN dim\_country AS dc USING(Country\_ID)
  LEFT JOIN dim\_item AS di USING(Item\_ID)
  LEFT JOIN dim\_element AS de USING(Element\_ID)
  WHERE di.Item='Potatoes' 
    AND de.Element = 'Production' 
    AND dc.Country NOT LIKE 'China,%' 
  GROUP BY Year
  ORDER BY Year)
  
SELECT Country, Year-1 AS Prev\_Year, Prev\_Year\_Production, Year, Production, YOY, YOY\_Perc
FROM yoy\_table 
LEFT JOIN median\_table USING(Year)
WHERE Production > Median
  AND Year > 2000
ORDER BY YOY\_Perc DESC
LIMIT 10;
```

> YOY is calculated excluding the countries that has less production amount than yearly production median in order to have a better view on data without small production amounts. Filters applied to prevent duplicated counts of China data. Production amount of country “China” is equal to the sum of “China, mainland” and “China, Taiwan Province of”. The source code of China production can be found at Appendix.
> 

## Q9. Share of the Global Pie

How much does Germany contribute to the world’s total *Potatoes* production each year?

```sql
WITH potato\_production AS (
  SELECT  fp.Year, 
          dc.Country, 
          SUM(fp.Value) AS Country\_Production,
          SUM(fp.Value) OVER(PARTITION BY fp.Year) AS Global\_Production,
          de.Unit
  FROM fact\_production AS fp
  LEFT JOIN dim\_country AS dc USING(Country\_ID)
  LEFT JOIN dim\_item AS di USING(Item\_ID)
  LEFT JOIN dim\_element AS de USING(Element\_ID)
  WHERE di.Item='Potatoes' 
    AND de.Element ='Production'
    AND dc.Country NOT LIKE 'China,%' 
  GROUP BY fp.Year, dc.Country, fp.Value, de.Unit
  ORDER BY fp.Year, dc.Country)

SELECT  Year, 
        Country\_Production AS USA, 
        Global\_Production,
        Unit,
        ROUND(Country\_Production\*100/Global\_Production,0) AS USA\_Cont\_Perc,
FROM potato\_production
WHERE Country = 'United States of America'
  AND Year >= 2000;
```

> Filters applied to prevent duplicated counts of China data. Production amount of country “China” is equal to the sum of “China, mainland” and “China, Taiwan Province of”. The source code of China production can be found at Appendix.
> 

## Q10. Smoothing the Signal: 3-Year Rolling Averages

Has production grown steadily or in spurts?

```sql
SELECT  fp.Year, 
        SUM(fp.Value) AS Global\_Production, 
        AVG(SUM(fp.Value)) OVER (ORDER BY fp.Year ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS Three\_Year\_Rolling\_AVG
FROM fact\_production AS fp
LEFT JOIN dim\_country AS dc
USING(Country\_ID)
LEFT JOIN dim\_item AS di
USING(Item\_ID)
LEFT JOIN dim\_element AS de
USING(Element\_ID)
WHERE de.Element = 'Production'
  AND di.Item = 'Potatoes' 
  AND dc.Country NOT LIKE 'China,%' 
GROUP BY fp.Year
ORDER BY fp.Year;
```

> Filters applied to prevent duplicated counts of China data. Production amount of country “China” is equal to the sum of “China, mainland” and “China, Taiwan Province of”. The source code of China production can be found at Appendix.
> 

## Q11. Yearly Champions: Who Led Each Year?

Who were the yearly champions of production?

```sql
WITH pot\_prod AS (
  SELECT  fp.Year, 
          dc.Country, 
          SUM(fp.Value) AS Champion\_Production, 
          RANK() OVER(PARTITION BY fp.Year ORDER BY Champion\_Production DESC) AS Rank
  FROM fact\_production AS fp
  LEFT JOIN dim\_country AS dc USING(Country\_ID)
  LEFT JOIN dim\_item AS di USING(Item\_ID)
  LEFT JOIN dim\_element AS de USING(Element\_ID)
  WHERE di.Item='Potatoes' 
    AND de.Element ='Production'
    AND dc.Country NOT LIKE 'China,%' 
  GROUP BY fp.Year, dc.Country
  ORDER BY Rank, fp.Year, dc.Country)

SELECT Year, Country, Champion\_Production
FROM pot\_prod
WHERE Rank = 1
ORDER BY Year;

```

> Filters applied to prevent duplicated counts of China data. Production amount of country “China” is equal to the sum of “China, mainland” and “China, Taiwan Province of”. The source code of China production can be found at Appendix.
> 

## Q12. Who Improved the Most Over 5 Years?

Who made the biggest leap in the last 5 years?

```sql
WITH l AS (
  SELECT dc.Country, ROUND(AVG(fp.Value)) AS Last\_5, de.Unit
  FROM fact\_production AS fp
  LEFT JOIN dim\_country AS dc USING(Country\_ID)
  LEFT JOIN dim\_item AS di USING(Item\_ID)
  LEFT JOIN dim\_element AS de USING(Element\_ID)
  WHERE di.Item='Potatoes' 
    AND de.Element ='Production'
    AND dc.Country NOT LIKE 'China,%'
    AND Year BETWEEN 2019 AND 2023 
  GROUP BY Country, Unit),

p AS (
  SELECT dc.Country, ROUND(AVG(fp.Value)) AS Prev\_5, de.Unit
  FROM fact\_production AS fp
  LEFT JOIN dim\_country AS dc USING(Country\_ID)
  LEFT JOIN dim\_item AS di USING(Item\_ID)
  LEFT JOIN dim\_element AS de USING(Element\_ID)
  WHERE di.Item='Potatoes' 
    AND de.Element ='Production'
    AND dc.Country NOT LIKE 'China,%'
    AND Year BETWEEN 2014 AND 2018
  GROUP BY Country, Unit),

improvement AS (
  SELECT l.Country, l.Last\_5 - p.Prev\_5 AS improvement\_absolute 
  FROM l
  LEFT JOIN p USING(Country)),

avg\_improvement AS(
  SELECT ROUND(AVG(improvement\_absolute),0) AS avg\_impr
  FROM improvement)

SELECT  
        l.Country, 
        l.Last\_5, 
        p.Prev\_5, 
        l.Unit,
        improvement\_absolute,
        avg\_impr,
        CASE  WHEN p.Prev\_5 > 0 
                THEN ROUND(((l.Last\_5/p.Prev\_5)-1)\*100,0)
              ELSE NULL END AS Improvement\_Perc,
        RANK() OVER(ORDER BY Improvement\_Perc DESC) AS Rank
FROM l
LEFT JOIN p USING(Country)
LEFT JOIN improvement USING(Country)
CROSS JOIN avg\_improvement
WHERE improvement\_absolute > avg\_impr
ORDER BY Rank
LIMIT 10;
```

> Filters applied to prevent duplicated counts of China data. Production amount of country “China” is equal to the sum of “China, mainland” and “China, Taiwan Province of”. The source code of China production can be found at Appendix.
> 

## Appendix

```sql
SELECT dc.Country, ROUND(SUM(fp.Value),0) AS Total\_Production, de.Unit
FROM fact\_production AS fp
LEFT JOIN dim\_country AS dc USING(Country\_ID)
LEFT JOIN dim\_item AS di USING(Item\_ID)
LEFT JOIN dim\_element AS de USING(Element\_ID)
WHERE di.Item='Potatoes' 
  AND de.Element ='Production'
  AND year > 2000
  AND dc.Country LIKE 'China%'
GROUP BY dc.Country, de.Unit
ORDER BY Total\_Production DESC;
```

> Total Potato Production of China and China Provinces. With this data it can be calculated that production volume of country “China” is equal to the sum of “China, mainland” and “China, Taiwan Province of”.
>

