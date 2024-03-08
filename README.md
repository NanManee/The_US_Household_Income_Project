# US_Household_Income

![Screenshot 2024-02-02 174945](https://github.com/NanManee/The_US_Household_Income_Project/assets/156528525/15b4f219-1d71-4282-9665-076f5ed73b8b)


### Data Source

www.analystbuilder.com/courses/mysql-for-data-analytics

This dataset does not include year information, so the instructor may focus more on data wrangling for learning purposes than on current statistics.



```SQL
#Check and clean data

SELECT * 
FROM us_project.us_household_income;

SELECT * 
FROM us_project.us_household_income_statistics;

ALTER TABLE us_project.us_household_income_statistics RENAME COLUMN `ï»¿id` TO `Id`;

#Identify duplicates

SELECT COUNT(id) 
FROM us_project.us_household_income;

SELECT COUNT(id) 
FROM us_project.us_household_income_statistics;

#Identify duplicates

SELECT id, COUNT(id) 
FROM us_project.us_household_income
GROUP BY id
HAVING COUNT(id) > 1;

SELECT *
FROM (SELECT row_id, id, ROW_NUMBER() OVER(PARTITION BY id ORDER BY id) AS row_num
	FROM us_project.us_household_income) AS duplicates
WHERE row_num > 1;

#Delete the duplicates

DELETE FROM us_household_income
WHERE row_id IN
        (SELECT row_id
        FROM (SELECT row_id,
        id,
        ROW_NUMBER() OVER(PARTITION BY id ORDER BY id) AS row_num
		    FROM us_project.us_household_income) AS duplicates
	      WHERE row_num > 1);

#To double-check if there are no duplicates

SELECT id, COUNT(id) 
FROM us_project.us_household_income
GROUP BY id
HAVING COUNT(id) > 1;

SELECT id, COUNT(id) 
FROM us_project.us_household_income_statistics
GROUP BY id
HAVING COUNT(id) > 1;

#Correcting a spelling error in a state name

SELECT State_Name, COUNT(State_Name)
FROM us_project.us_household_income
GROUP BY State_Name;

SELECT DISTINCT State_Name
FROM us_project.us_household_income;

UPDATE us_project.us_household_income
SET state_name = 'Alabama'
WHERE state_name = 'alabama';

UPDATE us_project.us_household_income
SET state_name = 'Georgia'
WHERE state_name = 'georia';

SELECT * 
FROM us_project.us_household_income;

SELECT DISTINCT State_ab
FROM us_project.us_household_income
ORDER BY 1;

#Populate missing value

SELECT * 
FROM us_project.us_household_income
WHERE place = '';

SELECT * 
FROM us_project.us_household_income
WHERE place = 'Autauga County';

UPDATE us_project.us_household_income
SET place = 'Autaugaville'
WHERE county = 'Autauga County'
AND city = 'Vinemont';

#Check for any errors or duplications

SELECT type, COUNT(type) 
FROM us_project.us_household_income
GROUP BY type;

UPDATE us_project.us_household_income
SET type = 'Borough'
WHERE type = 'Boroughs';

SELECT DISTINCT AWater, ALand
FROM us_project.us_household_income
WHERE AWater = 0 OR AWater = '' OR AWater IS NULL;

SELECT DISTINCT ALand, AWater
FROM us_project.us_household_income
WHERE ALand = 0 OR ALand = '' OR ALand IS NULL;

#Exploratory Data Analysis

SELECT State_Name, county, city, ALand, AWater
FROM us_project.us_household_income;

#Which state has the most land and water(lake and stream)

SELECT State_Name, SUM(ALand), SUM(AWater)
FROM us_project.us_household_income
GROUP BY State_Name
ORDER BY 2 DESC;

SELECT State_Name, SUM(ALand), SUM(AWater)
FROM us_project.us_household_income
GROUP BY State_Name
ORDER BY 3 DESC;

#Top 10 larger state by land and water

SELECT State_Name, SUM(ALand), SUM(AWater)
FROM us_project.us_household_income
GROUP BY State_Name
ORDER BY 2 DESC
LIMIT 10;

SELECT State_Name, SUM(ALand), SUM(AWater)
FROM us_project.us_household_income
GROUP BY State_Name
ORDER BY 3 DESC
LIMIT 10;

#Join tables

SELECT * 
FROM us_project.us_household_income u
JOIN us_project.us_household_income_statistics us
	ON u.id = us.id;
    
SELECT * 
FROM us_project.us_household_income u
JOIN us_project.us_household_income_statistics us
	ON u.id = us.id
WHERE Mean <> 0;

SELECT u.State_Name, county, type, `Primary`, mean, median
FROM us_project.us_household_income u
JOIN us_project.us_household_income_statistics us
	ON u.id = us.id
WHERE Mean <> 0;

#Top 10 States with the Lowest Mean Income

SELECT u.State_Name, ROUND(AVG(mean),1), ROUND(AVG(median),1)
FROM us_project.us_household_income u
JOIN us_project.us_household_income_statistics us
	ON u.id = us.id
WHERE mean <> 0
GROUP BY u.State_Name
ORDER BY ROUND(AVG(mean))
LIMIT 10;

#Top 15 States with the Highest Mean Income

SELECT u.State_Name, ROUND(AVG(mean),1), ROUND(AVG(median),1)
FROM us_project.us_household_income u
JOIN us_project.us_household_income_statistics us
	ON u.id = us.id
WHERE mean <> 0
GROUP BY u.State_Name
ORDER BY ROUND(AVG(mean))DESC
LIMIT 15;

#Top 10 States with the Lowest Median Income

SELECT u.State_Name, ROUND(AVG(mean),1), ROUND(AVG(median),1)
FROM us_project.us_household_income u
JOIN us_project.us_household_income_statistics us
	ON u.id = us.id
WHERE mean <> 0
GROUP BY u.State_Name
ORDER BY ROUND(AVG(median))
LIMIT 10;

#Top 10 States with the Highest Median Income

SELECT u.State_Name, ROUND(AVG(mean),1), ROUND(AVG(median),1)
FROM us_project.us_household_income u
JOIN us_project.us_household_income_statistics us
	ON u.id = us.id
WHERE mean <> 0
GROUP BY u.State_Name
ORDER BY 3 DESC
LIMIT 10;

#The region with the highest mean household income

SELECT Type, COUNT(type), ROUND(AVG(mean),1), ROUND(AVG(median),1)
FROM us_project.us_household_income u
JOIN us_project.us_household_income_statistics us
	ON u.id = us.id
WHERE mean <> 0
GROUP BY type
ORDER BY 3 DESC
LIMIT 15;

#The region with the highest median household income

SELECT Type, COUNT(type), ROUND(AVG(mean),1), ROUND(AVG(median),1)
FROM us_project.us_household_income u
JOIN us_project.us_household_income_statistics us
	ON u.id = us.id
WHERE mean <> 0
GROUP BY type
ORDER BY 4 DESC
LIMIT 15;

#What states that have community

SELECT *
FROM us_project.us_household_income
WHERE type = 'Community';

# Puerto Rico has community
   
#Top 15 cities with the highest median household income

SELECT u.State_Name, city, ROUND(AVG(median),1)
FROM us_project.us_household_income u
JOIN us_project.us_household_income_statistics us
	ON u.id = us.id
GROUP BY u.State_Name, city
ORDER BY 2 DESC
LIMIT 15;

#Top 15 cities with the highest mean and median household income

SELECT u.State_Name, city, ROUND(AVG(mean),1), ROUND(AVG(median),1)
FROM us_project.us_household_income u
JOIN us_project.us_household_income_statistics us
	ON u.id = us.id
GROUP BY u.State_Name, city
ORDER BY ROUND(AVG(mean),1) DESC
LIMIT 15;
```

## Tableau Visualization: 

https://public.tableau.com/app/profile/nanthawan.maneethong/viz/HouseholdIncomeinTheU_S_/Dashboard1

### Limitations
- This project is part of my Data Analysis course from Analyst Builder. The dataset lacks yearly information, preventing the presentation of trends over the past decade.
- This dataset does not include information on urban, suburban, and rural areas, so we cannot examine how household income distribution varies across these areas.

### Insights

### How does household income vary by geographic region in the United States, and what factors contribute to these differences?
 
### Eight Southeastern states account for the majority of states with an average median household income below $70,000, followed by one state each from the Northeast and the Southwest:

- Southeastern States: West Virginia, Kentucky, Tennessee, Arkansas, Louisiana, Mississippi, Alabama, and South Carolina.
- Northeastern State: Maine.
- Southwestern State: Oklahoma.

### Seven Northeastern states account for the majority of states with an average median household income above $100,000, followed by six states from the West and one from the Southeast:

- Northeastern States: New York, New Hampshire, New Jersey, Massachusetts, Rhode Island, Connecticut, and Maryland.
- Western States: California, Washington, Utah, Colorado, Wyoming, and Alaska.
- Southeastern State: Virginia.

### Twelve Midwestern states account for the majority of states with an average median household income between $70,000 and $100,000, followed by five states from the West, three from the Northeast, three from the South, and three from the Southwest.
- Midwestern States: North Dakota, South Dakota, Nebraska, Kansas, Missouri, Iowa, Minnesota, Wisconsin, Illinois, Indiana, Michigan, and Ohio.
- Western States: Montana, Idaho, Oregon, Nevada, and Hawaii.
- Northeastern States: Pennsylvania, Vermont, and Delaware.
- Southeastern States: Florida, Georgia, and North Carolina.
- Southwestern States: Texas, New Mexico, and Arizona.

### In summary: 
the United States exhibits a diverse economic landscape, with the Southeast hosting the majority of lower-income states, the Northeast and certain Western states presenting higher median household incomes, and the Midwest, along with a mix of states from other regions, reflecting middle-income levels. This distribution highlights the varying economic conditions and living standards across the country.
