# Covid_Death_Data
- A brief look into Covid Death data using SQL on BigQuery.
- Data was provided from public dataset here: https://ourworldindata.org/covid-deaths
- This data cleaning and exploration project was guided by "Alex The Analyst" on Youtube. Project was done using his stated permission.

### BigQuery SQL 


-- Select data that we are going to be using

- SELECT Location, date, total_cases, new_cases, total_deaths, population
- FROM covid-project-agd.Covid.Deaths
- ORDER BY 1,2 -- Location and Date 

-- Looking at Total Cases vs Total Deaths

- SELECT Location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
- FROM covid-project-agd.Covid.Deaths
- ORDER BY 1,2 -- Location and Date 

-- Look at Total Cases vs Population

- SELECT Location, date, total_cases, population, (total_cases/population)*100 as CasesPercentage
- FROM covid-project-agd.Covid.Deaths
- ORDER BY 1,2 -- Location and Date 

-- Look at Countries with highest infection rate per population

- SELECT Location, population, MAX(total_cases) AS TopInfectionCount, MAX((total_cases/population))*100 as PercentPopulationInfected
- FROM covid-project-agd.Covid.Deaths
- GROUP BY Location, population
- ORDER BY PercentPopulationInfected desc  

-- Look at Countries with Highest Death count per population

- SELECT location, MAX(total_deaths) AS TotalDeathCount
- FROM covid-project-agd.Covid.Deaths
- WHERE continent is not null -- Removes Continet data to only show Countries
- GROUP BY location
- order by TotalDeathCount desc

-- Continents Death count sorted from highest to lowest

- SELECT location, MAX(total_deaths) AS TotalDeathCount
- FROM covid-project-agd.Covid.Deaths
- WHERE continent is null -- Shows only Continet data
- GROUP BY location
- order by TotalDeathCount desc

-- Looking at Global Numbers

- SELECT SUM(new_cases) as total_cases, SUM(new_deaths) as total_deaths, (SUM(new_deaths)/SUM(new_cases))*100 as TotalDeathPercentage
- FROM covid-project-agd.Covid.Deaths
- where continent is not null
- order by 1,2

-- Join tables in order to compare population vs vaccine rate

- SELECT *
- From covid-project-agd.Covid.Deaths dea
- JOIN covid-project-agd.Covid.Vaccinations vac
- ON dea.location = vac.location
- and dea.date = vac.date -- Tables are joined

--Calculate the percent of population that has recieved at least one vaccine

-- First calculation shows how many people were vaccinated per day, and a rolling total.

- SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    - SUM(vac.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS TotalPeopleVaccinated
- FROM covid-project-agd.Covid.Deaths dea
- JOIN covid-project-agd.Covid.Vaccinations vac
    - ON dea.location = vac.location
    - AND dea.date = vac.date
- WHERE dea.continent is not null 
- order by 2,3

-- Create new table to showcase Percent of population vaccinated


- Create Temp Table PercentPopulationVaccinated
- (Continent STRING,
Location STRING,
Date datetime,
Population INT64,
New_vaccinations INT64,
TotalPeopleVaccinated INT64);

-- Now that Temp table is created, we add data to the table

- Insert into PercentPopulationVaccinated
- Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as TotalPeopleVaccinated
- From covid-project-agd.Covid.Deaths
- Join covid-project-agd.Covid.Vaccinations
	On dea.location = vac.location
	and dea.date = vac.date
- where dea.continent is not null 
- order by 2,3

-- Take a look at the new filled in Temp Table

- Select *, (RollingPeopleVaccinated/Population)*100
- From PercentPopulationVaccinated


-- Creating a view to use later for pssible visualizations.

- Create View PercentPopulationVaccinated as
- Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
- From covid-project-agd.Covid.Deaths
- Join covid-project-agd.Covid.Vaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
- where dea.continent is not null 
