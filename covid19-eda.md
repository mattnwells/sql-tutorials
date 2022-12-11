```sql
/*
Covid 19 Data Exploration in SQL

Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Data Type Conversions

Tools used: Excel, SQL, BigQuery
*/

SELECT *
FROM `molten-hall-332415.covid.CovidDeaths`
WHERE continent IS NOT NULL
ORDER BY 3,4;


-- Select the data we are going to start with

SELECT location, date, total_cases, new_cases, total_deaths, population
FROM `molten-hall-332415.covid.CovidDeaths`
WHERE continent IS NOT NULL
ORDER BY 1,2;


-- Total Cases vs. Total Deaths
-- Shows likelihood of dying if you contract Covid in your country

SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS death_percentage
FROM `molten-hall-332415.covid.CovidDeaths`
WHERE location = 'United States'
AND continent IS NOT NULL
ORDER BY 1,2;


-- Total Cases vs. Population
-- Shows what percentage of the population contracted Covid

SELECT location, date, population, total_cases, (total_deaths/population)*100 AS death_percentage
FROM `molten-hall-332415.covid.CovidDeaths`
--WHERE location = 'United States'
ORDER BY 1,2;


-- Countries with the Highest Infection Rate compared to Populations

SELECT location, population, MAX(total_cases) AS higest_infection_count, MAX((total_cases/population))*100 AS percent_population_infected
FROM `molten-hall-332415.covid.CovidDeaths`
--WHERE location = 'United States'
GROUP BY location, population
ORDER BY percent_population_infected desc;


-- Countires with Highest Death Count by Population
SELECT location, MAX(total_deaths) AS total_death_count
FROM `molten-hall-332415.covid.CovidDeaths`
--WHERE location = 'United States'
WHERE continent IS NOT NULL
GROUP BY location
ORDER BY total_death_count desc; 


-- BREAK DOWN BY CONTINENT --

-- Showing contintents with the highest death count by population

SELECT continent, MAX(total_deaths) AS total_death_count
FROM `molten-hall-332415.covid.CovidDeaths`
--WHERE location = 'United States'
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY total_death_count desc;


-- GLOBAL NUMBERS --

SELECT SUM(new_cases) AS total_cases, SUM(new_deaths) AS total_deaths, SUM(new_deaths)/SUM(new_cases)*100 AS death_percentage
FROM `molten-hall-332415.covid.CovidDeaths`
--WHERE location = 'United States'
WHERE continent IS NOT NULL
--GROUP BY date
ORDER BY 1,2;

-- Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_individuals_vaccinated
--, (rolling_individuals_vaccinated/population)*100
FROM `molten-hall-332415.covid.CovidDeaths` AS dea
JOIN `molten-hall-332415.covid.CovidVaccinations` AS vac
  ON dea.location = vac.location
  AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
ORDER BY 2,3;


-- Using CTE to perform Calculation on PARTITION BY in previous query

WITH pop_vs_vac AS 
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
 SUM(vac.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_individuals_vaccinated
--, (rolling_individuals_vaccinated/population)*100
FROM `molten-hall-332415.covid.CovidDeaths` dea
JOIN `molten-hall-332415.covid.CovidVaccinations` vac
  ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
--order by 2,3
)
SELECT *, (rolling_individuals_vaccinated/population)*100
FROM pop_vs_vac;


-- Using Temp Table to perform Calculation on PARTITION BY in previous query

DROP TABLE IF EXISTS `molten-hall-332415.covid.percent_population_vaccinated`;

CREATE TABLE `molten-hall-332415.covid.percent_population_vaccinated`
(
continent string,
location string,
date datetime, 
population numeric,
new_vaccinations numeric, 
rolling_individiuals_vaccinated numeric  
);

INSERT INTO `molten-hall-332415.covid.percent_population_vaccinated`
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_individuals_vaccinated
--, (rolling_individuals_vaccinated/population)*100
FROM `molten-hall-332415.covid.CovidDeaths` dea
JOIN `molten-hall-332415.covid.CovidVaccinations` vac
	ON dea.location = vac.location AND dea.date = vac.date;
--WHERE dea.continent IS NOT NULL 
--order by 2,3

SELECT *, (rolling_individuals_vaccinated/population)*100
FROM `molten-hall-332415.covid.percent_population_vaccinated`;


-- Creating view to store for later visualizations 

CREATE VIEW `molten-hall-332415.covid.percent_population_vaccinated` AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
  , SUM(vac.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_individuals_vaccinated
--, (rolling_individuals_vaccinated/population)*100
FROM `molten-hall-332415.covid.CovidDeaths` dea
JOIN `molten-hall-332415.covid.CovidVaccinations` vac
	ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;
```
