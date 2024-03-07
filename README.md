# World_Life_Expectancy 2007 - 2022

```SQL
#Explore the data for missing field, duplicates, identify the issues and data cleaning

SELECT * 
FROM world_life_expectancy;

#Identify if there's any duplicates and how many they are
#Combine and create unique column

SELECT country, year, CONCAT(country, year)
FROM world_life_expectancy;


#Identify the duplicats in Country, Year and Row_ID

SELECT country, year, CONCAT(country, year), COUNT(CONCAT(country, year))
FROM world_life_expectancy
GROUP BY country, year, CONCAT(country, year)
HAVING COUNT(CONCAT(country, year)) > 1;


#Which Row_ID has duplicates

SELECT *
FROM (SELECT Row_ID,
	CONCAT(country, year),
	ROW_NUMBER() OVER(PARTITION BY CONCAT(country, year) ORDER BY CONCAT(country, year)) AS row_num
	FROM world_life_expectancy) AS row_table
WHERE row_num > 1;


#Delete the duplicates Row_ID

DELETE FROM world_life_expectancy
WHERE Row_ID IN 
	(SELECT Row_ID
	FROM (SELECT Row_ID,
	CONCAT(country, year),
	ROW_NUMBER() OVER(PARTITION BY CONCAT(country, year) ORDER BY CONCAT(country, year)) AS row_num
	FROM world_life_expectancy) AS row_table
	WHERE row_num > 1);


#Check Status column for blanks, nulls then populate data

SELECT * 
FROM world_life_expectancy;

SELECT *
FROM world_life_expectancy
WHERE status = '';

SELECT DISTINCT(status)
FROM world_life_expectancy
WHERE status <> '';

SELECT DISTINCT(country)
FROM world_life_expectancy
WHERE status = 'Developing';


#Update dataset, self join to fill country's status that blank

UPDATE world_life_expectancy t1
JOIN world_life_expectancy t2
	ON t1.country = t2.country
SET t1.status = 'Developing'
WHERE t1.status = ''
AND t2.status <> ''
AND t2.status = 'Developing';

SELECT *
FROM world_life_expectancy
WHERE status = '';

UPDATE world_life_expectancy t1
JOIN world_life_expectancy t2
	ON t1.country = t2.country
SET t1.status = 'Developed'
WHERE t1.status = ''
AND t2.status <> ''
AND t2.status = 'Developed';

SELECT *
FROM world_life_expectancy
WHERE country = 'United States of America';

SELECT *
FROM world_life_expectancy
WHERE status IS NULL;


#Check Life Expectancy column for blanks, nulls then populate data

SELECT * 
FROM world_life_expectancy
WHERE `Life expectancy` = '';


#Self join to find the average life expectancy between the year before and the year after to populate blanks

SELECT t1.country, t1.year, t1.`Life expectancy`,
	   t2.country, t2.year, t2.`Life expectancy`,
       t3.country, t2.year, t3.`Life expectancy`,
      ROUND((t2.`Life expectancy` + t3.`Life expectancy`)/2,1)
FROM world_life_expectancy t1
JOIN world_life_expectancy t2
	ON t1.country = t2.country
    AND t1.year = t2.year - 1
JOIN world_life_expectancy t3
	ON t1.country = t3.country
    AND t1.year = t3.year + 1
WHERE t1.`Life expectancy` = '';

UPDATE world_life_expectancy t1
JOIN world_life_expectancy t2
	ON t1.country = t2.country
    AND t1.year = t2.year - 1
JOIN world_life_expectancy t3
	ON t1.country = t3.country
    AND t1.year = t3.year + 1
SET t1.`Life expectancy` = ROUND((t2.`Life expectancy` + t3.`Life expectancy`)/2,1)
WHERE t1.`Life expectancy` = '';

SELECT * 
FROM world_life_expectancy
WHERE `Life expectancy` = '';


##Exploratory data analysis
SELECT * 
FROM world_life_expectancy;


#Which country have the higher and the lower life expectancy and highest increase in Life expectancy
#Also filter out some country that has 0 values

SELECT Country, MIN(`Life expectancy`), MAX(`Life expectancy`),
ROUND (MAX(`Life expectancy`) - MIN(`Life expectancy`),1) AS Life_Increase_Over_15_Years
FROM world_life_expectancy
GROUP BY Country
HAVING MIN(`Life expectancy`) <> 0
AND MIN(`Life expectancy`) <> 0
ORDER BY Life_Increase_Over_15_Years DESC;


##The world average life expectancy

SELECT Year, ROUND(AVG(`Life expectancy`),2) AS Average_Life_Expectancy
FROM world_life_expectancy
WHERE `Life expectancy`<> 0
AND `Life expectancy`<> 0
GROUP BY Year
ORDER BY Year;


##Which country has the highest average of Life expectancy?

SELECT country, Year, ROUND(AVG(`Life expectancy`),2) AS Average_Life_Expectancy
FROM world_life_expectancy
WHERE `Life expectancy`<> 0
AND `Life expectancy`<> 0
GROUP BY country, Year
ORDER BY Year DESC, Average_Life_Expectancy DESC;


#The correlation between life expectancy and GDP(Gross Domestic Product)
#The average of the life expectancy and GDP of each country

SELECT country, ROUND(AVG(`Life expectancy`),1) AS Average_Life_Expectancy,
	ROUND(AVG(GDP),1) AS Average_GDP
FROM world_life_expectancy
GROUP BY country
HAVING Average_Life_Expectancy > 0
AND Average_GDP > 0
ORDER BY Average_GDP ASC;

##The lower the GDPs are correlated with lower life expectancy, it can be that their healthcare infrastucture is now as well built


## The positive correlation, the higher the GDPs, is also the higher the life expectancy

SELECT country, ROUND(AVG(`Life expectancy`),1) AS Average_Life_Expectancy,
	ROUND(AVG(GDP),1) AS Average_GDP
FROM world_life_expectancy
GROUP BY country
HAVING Average_Life_Expectancy > 0
AND Average_GDP > 0
ORDER BY Average_GDP DESC;

SELECT * 
FROM world_life_expectancy
ORDER BY GDP;

SELECT 
SUM(CASE 
	WHEN GDP >= 1500 THEN 1 
	ELSE 0
END) High_GDP_Count
FROM world_life_expectancy;

#1326 rows that GDP >= 1500


SELECT 
SUM(CASE WHEN GDP >= 1500 THEN 1 ELSE 0 END) High_GDP_Count,
AVG(CASE WHEN GDP >= 1500 THEN `Life expectancy`ELSE NULL END) High_GDP_Life_expectancy
FROM world_life_expectancy;

#1326 rows that GDP >= 1500 have the average of High_GDP_Life_expectancy at 74.02 years

#A high correlations from a high GDPs to a high life expectancy VS a low GDPs to a low life expectancy

SELECT 
SUM(CASE WHEN GDP >= 1500 THEN 1 ELSE 0 END) High_GDP_Count,
AVG(CASE WHEN GDP >= 1500 THEN `Life expectancy`ELSE NULL END) High_GDP_Life_expectancy,
SUM(CASE WHEN GDP <= 1500 THEN 1 ELSE 0 END) Low_GDP_Count,
AVG(CASE WHEN GDP <= 1500 THEN `Life expectancy`ELSE NULL END) Low_GDP_Life_expectancy
FROM world_life_expectancy;

#1326 rows that GDP >= 1500, have the average of High_GDP_Life_expectancy at 74.02 years
#1612 rows that GDP <= 1500, have the average of Low_GDP_Life_expectancy at 64.70 years


#The difference in life expectancy among countries's status

SELECT * 
FROM world_life_expectancy;

#The Developing countries have the average of life expectancy at 66.8 years
#The Developed countries have the average of life expectancy at 79.2 years

SELECT status, ROUND(AVG(`Life expectancy`),1)
FROM world_life_expectancy
GROUP BY status;


#How many Developed and Developing country? 32 & 161

SELECT status, COUNT(DISTINCT country), ROUND(AVG(`Life expectancy`),1)
FROM world_life_expectancy
GROUP BY status;

SELECT country, ROUND(AVG(`Life expectancy`),1) AS Average_Life_Expectancy,
	ROUND(AVG(BMI),1) AS average_BMI
FROM world_life_expectancy
GROUP BY country
HAVING Average_Life_Expectancy > 0
AND Average_BMI > 0
ORDER BY Average_BMI DESC;

#Lower BMI, lower life expectancy


SELECT country, ROUND(AVG(GDP),1) AS Average_GDP,
	ROUND(AVG(BMI),1) AS average_BMI
FROM world_life_expectancy
GROUP BY country
HAVING Average_GDP > 0
AND Average_BMI > 0
ORDER BY Average_GDP DESC;

#Lower GDP - lower BMI, Higher GDP - higher BMI


#How many people are dying each year in each country
SELECT country, year, status, `Life expectancy`, `Adult Mortality`,
SUM(`Adult Mortality`) OVER(PARTITION BY country ORDER BY Year) AS Rolling_Total
FROM world_life_expectancy
ORDER BY `Adult Mortality` DESC;

#The developing country has much lower life expectancy and higher adult mortality 


SELECT country, year, status, `Life expectancy`, `Adult Mortality`,
SUM(`Adult Mortality`) OVER(PARTITION BY country ORDER BY Year) AS Rolling_Total
FROM world_life_expectancy
WHERE country LIKE '%United States%';

# The adult mortality rate in the US has significantly decreased since 2014
```

Tableau Visualization: https://public.tableau.com/app/profile/nanthawan.maneethong/viz/world1_17097682941850/Dashboard1

