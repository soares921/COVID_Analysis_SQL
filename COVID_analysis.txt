# Choosing the data to explore
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM covid.covid_deaths
ORDER BY 1,2

# Looking at total cases vs. total deaths
# Shows the likelihood of dying when contracting covid in country
SELECT location, date, total_cases, total_deaths,(total_deaths/total_cases)*100 AS DeathPercentage
FROM covid.covid_deaths
WHERE location = 'United States'
ORDER BY 1,2

# Looking at total cases vs. population
# Shows percentage of population that got covid
SELECT location, date, total_cases, population,(total_cases/population)*100 AS InfectionPercentage
FROM covid.covid_deaths
WHERE location = 'United States'
ORDER BY 2

# Looking at countries with highest infection rate
SELECT location, population,MAX(total_cases) AS HighestInfectionCount, MAX((total_cases/population))*100 AS InfectionPercentage
FROM covid.covid_deaths
GROUP BY location, population
ORDER BY InfectionPercentage DESC

# Looking at countries with highest death count per population
SELECT location, MAX(total_deaths) AS TotalDeathCount
FROM covid.covid_deaths
WHERE continent IS NOT null
GROUP BY location
ORDER BY TotalDeathCount DESC

# Breaking it down by continent
# Showing the continents with highest death count per population
SELECT continent, MAX(total_deaths) AS TotalDeathCount
FROM covid.covid_deaths
WHERE continent IS NOT null
GROUP BY continent
ORDER BY TotalDeathCount DESC

# Global numbers
# Shows total cases and deaths at global level per day
SELECT date, SUM(new_cases) AS TotalCases, SUM(new_deaths) AS TotalDeaths,SUM(new_deaths)/ NULLIF(SUM(new_cases),0)*100 AS DeathPercentage
FROM covid.covid_deaths
WHERE continent IS NOT null
GROUP BY date
ORDER BY 1,2

# Looking at total vaccination vs. population
SELECT death.continent, death.location, death.date, death.population, vax.new_vaccinations,
  SUM(vax.new_vaccinations) OVER (PARTITION BY death.location ORDER BY death.location, death.date) AS VaccinatedRolling
FROM covid.covid_deaths AS death
JOIN covid.covid_vaccinations AS vax
  ON death.location = vax.location
  AND death.date = vax.date
WHERE death.continent IS NOT null
ORDER BY 2,3

# Using CTE to find percentage of the rolling vaccinated population
WITH PopvsVax AS (
  SELECT death.continent, death.location, death.date, death.population, vax.new_vaccinations,
  SUM(vax.new_vaccinations) OVER (PARTITION BY death.location ORDER BY death.location, death.date) AS VaccinatedRolling
FROM covid.covid_deaths AS death
JOIN covid.covid_vaccinations AS vax
  ON death.location = vax.location
  AND death.date = vax.date
WHERE death.continent IS NOT null
)

SELECT *, (VaccinatedRolling/population)*100 AS PercentVaccinated
FROM PopvsVax


#Creating View to store data for later visualization
CREATE VIEW covid.PercentPopulationVaccinated AS
SELECT death.continent, death.location, death.date, death.population, vax.new_vaccinations,
  SUM(vax.new_vaccinations) OVER (PARTITION BY death.location ORDER BY death.location, death.date) AS VaccinatedRolling
FROM covid.covid_deaths AS death
JOIN covid.covid_vaccinations AS vax
  ON death.location = vax.location
  AND death.date = vax.date
WHERE death.continent IS NOT null

