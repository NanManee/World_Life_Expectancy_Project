# World_Life_Expectancy 

![Screenshot 2024-02-02 162935](https://github.com/NanManee/World_Life_Expectancy_Project/assets/156528525/f2b54f7a-cc6a-4a62-9b6c-43137865907d)

### Introduction

In this project, we embark through the data on world life expectancy from 2007 to 2022. My goal is to show the average lifespan of people globally, highlight the countries with the highest and lowest life expectancies, and the contrasts between developed and developing nations. Utilizing SQL for the analysis, This project aim to uncover trends and patterns in life expectancy, offering insights into the factors that influence how long people live around the world. This exploration seeks to provide a clear understanding of global health dynamics over the past fifteen years.

### Project Task

- To understanding the current average life expectancy across different countries.
- Investigate variations in life expectancy between developed and developing countries.
- Analyze historical data to understand trends in life expectancy and explore factors contributing to changes over time.

### Questions to answers

- What is the latest global life expectancy?
- Does life expectancy vary between different countries?
- Identifying which countries have the lowest and highest life expectancy?
- What trends have been observed in life expectancy over times?

### Data Source

Data Source: www.analystbuilder.com/courses/mysql-for-data-analytics
Year: 2007-2022

### Tools

MySQL - Data Cleaning and Data Analysis
Tableau - Data visualization

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

### Tableau Visualization:

https://public.tableau.com/app/profile/nanthawan.maneethong/viz/world1_17097682941850/Dashboard1

### Insights

- As of 2022, the latest year for which data is available, the average global life expectancy stands at 72 years.

- Trends observed in life expectancy over time indicate that the average global life expectancy has mostly increased each year over the past 15 years, except for 2020. During the pandemic caused by COVID-19, life expectancy worldwide decreased from 71 years in 2019 to 68 years in 2020. However, there was a rebound to 72 years in 2021.

- The data also highlight the significant gap in life expectancy between developing countries, which have an average of 67 years, and developed countries, with an average of 79 years. The difference may be based on factors such as:

	- Healthcare Access and Quality: Countries with well-developed healthcare systems typically have higher life expectancies. Access to preventive services, early and effective treatment of diseases, and the availability of advanced medical technology can 		  significantly impact overall health outcomes.
	- Economic Factors: Conversely, poverty is often linked with malnutrition, limited access to clean water, inadequate housing, and a lack of healthcare, all of which can contribute to lower life expectancy.
	- Education: Higher levels of education can lead to better health outcomes. Educated individuals are more likely to understand health-related information.
	- Lifestyle Factors: Smoking, alcohol consumption, physical inactivity, and unhealthy eating habits are major risk factors for chronic diseases such as heart disease, cancer, and diabetes, which can lower life expectancy.
	- Infectious Diseases: Countries with high rates of HIV/AIDS, tuberculosis, malaria, and other infectious diseases often have lower life expectancies. The impact of pandemics, such as COVID-19, has also significantly affected life expectancy rates in 2020.
	  Understanding the reasons behind variations in life expectancy across different countries is crucial for developing targeted interventions aimed at improving health outcomes and closing the gap in life expectancy.
   
- Japan was the only country from Asia to have the highest life expectancy in 2022, at 84 years, followed by Switzerland, Spain, Australia, and Italy, each with 83 years. It appears that many European countries have a high average.

- Sierra Leone had the lowest life expectancy in 2022, at only 46 years. Following closely are the Central African Republic and Lesotho with 49 years, and Angola and Chad with 50 years. These countries are all located in Africa.

