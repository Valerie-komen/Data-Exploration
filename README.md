# Data-Exploration
Using Covid 19 Dataset

### Skills used: 
1. Joins
2. CTE's
3. Temp Tables
4. Windows Functions
5. Aggregate Functions
6. Creating Views
7. Converting Data Types

### Tools
Mysql server


```sql
SELECT *
FROM PortfolioProject..CovidDeaths
WHERE continent is not null 
ORDER BY 3,4
```

#### Select Data that we are going to be starting with
```sql
SELECT Location, date, total_cases, new_cases, total_deaths, population
FROM PortfolioProject..CovidDeaths
WHERE continent is not null 
ORDER BY 1,2
```

#### Total Cases vs Total Deaths
##### Shows likelihood of dying if you contract covid in your country
```sql
SELECT Location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE location like '%states%'
AND continent is not null 
ORDER BY 1,2
```

#### Total Cases vs Population
##### Shows what percentage of population infected with Covid
```sql
SELECT Location, date, Population, total_cases,  (total_cases/population)*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
WHERE location like '%states%'
ORDER BY 1,2
```

#### Countries with Highest Infection Rate compared to Population
```sql
SELECT Location, Population, MAX(total_cases) AS HighestInfectionCount,  Max((total_cases/population))*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
WHERE location like '%states%'
GROUP BY Location, Population
ORDER BY PercentPopulationInfected desc
```

#### Countries with Highest Death Count per Population
```sql
SELECT Location, MAX(cast(Total_deaths as int)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE location like '%states%'
AND continent is not null 
GROUP BY Location
ORDER BY TotalDeathCount desc
```


#### BREAKING THINGS DOWN BY CONTINENT

##### Showing contintents with the highest death count per population
```sql
SELECT continent, MAX(cast(Total_deaths as int)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE location like '%states%'
WHERE continent is not null 
GROUP BY continent
ORDER BY TotalDeathCount desc
```


#### GLOBAL NUMBERS
```sql
SELECT SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) AS total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE location like '%states%'
WHERE continent is not null 
GROUP BY date
ORDER BY 1,2
```


#### Total Population vs Vaccinations
##### Shows Percentage of Population that has received at least one Covid Vaccine
```sql
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) AS RollingPeopleVaccinated, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
	ON dea.location = vac.location
	AND dea.date = vac.date
WHERE dea.continent is not null 
ORDER BY 2,3
```

#### Using CTE to perform Calculation on Partition By in previous query
```sql
With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
AS
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated, (RollingPeopleVaccinated/population)*100
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
	ON dea.location = vac.location
	AND dea.date = vac.date
WHERE dea.continent is not null 
ORDER BY 2,3
)
SELECT*, (RollingPeopleVaccinated/Population)*100
FROM PopvsVac
```


#### Using Temp Table to perform Calculation on Partition By in previous query
```sql
DROP Table if exists #PercentPopulationVaccinated
CREATE TABLE #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) AS RollingPeopleVaccinated, (RollingPeopleVaccinated/population)*100
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
	ON dea.location = vac.location
	AND dea.date = vac.date
WHERE dea.continent is not null 
ORDER BY 2,3

SELECT*, (RollingPeopleVaccinated/Population)*100
FROM #PercentPopulationVaccinated
```



#### Creating View to store data for later visualizations
```sql
CREATE VIEW PercentPopulationVaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) AS RollingPeopleVaccinated, (RollingPeopleVaccinated/population)*100
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
	ON dea.location = vac.location
	AND dea.date = vac.date
WHERE dea.continent is not null 
```
